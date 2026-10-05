# 03 · Subagent Invocation, Context Passing, and Spawning

**Exam link:** Task Statement 1.3: *Configure subagent invocation, context passing, and spawning.*
**Prerequisite:** [02 · Multi-Agent Orchestration](02-multi-agent-orchestration.md) (coordinator, isolated context).
**Scope note:** [02](02-multi-agent-orchestration.md) explains *why* the coordinator pattern works. This file is the *how*: Agent SDK configuration, writing delegation prompts, and parallel spawning. Session forking is covered in [07](07-session-state-resumption-forking.md).

---

## 1. How a subagent is started

In the Claude Agent SDK, the main agent starts a subagent by **calling a special built-in tool**. The model writes a `tool_use` block for it, just like any other tool (see [01](01-agentic-loops.md)). The SDK then:

1. creates a new agent with a **fresh context**,
2. gives it the subagent's own system prompt and tools,
3. sends it the prompt the coordinator wrote,
4. runs it to completion, and
5. returns its final answer to the coordinator as the **tool result**.

### Tool name: "Task" vs "Agent"

| Source | Name used |
|---|---|
| **Exam guide** | **`Task`** tool: "the coordinator's `allowedTools` must include `"Task"`" |
| **Current Agent SDK docs** | **`Agent`** tool (appears as `Task` in some older versions and in the session init tool list) |

**For the exam, answer with `Task`.** In real code, check the docs for your SDK version. The concept is the same: **the coordinator can only spawn subagents if this tool is in its allowed tools.**

> **Exam pattern:** "The coordinator never delegates; it tries to do everything itself." A likely root cause: the spawning tool (`Task`) is **missing from `allowedTools`**.

---

## 2. `AgentDefinition`: defining subagent types

You define each subagent type with an **`AgentDefinition`**. The main fields:

| Field | Purpose | Example |
|---|---|---|
| `description` | Tells the coordinator **when** to use this subagent. The model matches tasks to this text, so make it specific | "Searches the public web for recent sources on a topic. Use for finding new information, not for reading attached files." |
| `prompt` | The subagent's **system prompt**: its role, rules, and output format | "You are a research specialist. Return findings as JSON with claim, source_url, date…" |
| `tools` | **Tool restrictions**: the only tools this subagent may use | `["WebSearch", "WebFetch"]` |
| `model` | Optional: which model this subagent uses (e.g. a smaller, cheaper one for simple jobs) | `"haiku"` |

### Python example

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions, AgentDefinition, ResultMessage

options = ClaudeAgentOptions(
    system_prompt=(
        "You coordinate research. Break the question into distinct subtopics, "
        "delegate to subagents, check the synthesis for gaps, and re-delegate if needed."
    ),
    # The spawning tool must be allowed. Exam guide name: "Task"; current SDK name: "Agent".
    allowed_tools=["Agent", "Read"],
    agents={
        "web-researcher": AgentDefinition(
            description="Finds recent public sources on ONE assigned subtopic. Use for new information from the web.",
            prompt=(
                "You research a single assigned subtopic. Return a JSON list of findings. "
                "Each finding has: claim, source_url, source_title, publication_date."
            ),
            tools=["WebSearch", "WebFetch"],      # least privilege: no file writes, no shell
        ),
        "doc-analyst": AgentDefinition(
            description="Reads and extracts key points from documents the user provided.",
            prompt="Extract key claims with document name and page number for each.",
            tools=["Read", "Grep"],
        ),
        "synthesizer": AgentDefinition(
            description="Writes the final cited report from findings passed in the prompt.",
            prompt="Write a report using ONLY the findings given. Keep every citation. Flag conflicting sources.",
            tools=[],                             # needs no tools: all input comes in the prompt
            model="sonnet",
        ),
    },
)


async def main():
    async for message in query(prompt="Research the impact of AI on creative industries", options=options):
        if isinstance(message, ResultMessage) and message.subtype == "success":
            print(message.result)

asyncio.run(main())
```

**Design points:**

- **Least privilege:** each subagent gets only the tools it needs. The synthesizer has none, because everything it needs comes in its prompt.
- **Clear descriptions** stop the coordinator from choosing the wrong subagent (e.g. using the web researcher to read an attached PDF).

---

## 3. Context must be passed explicitly in the prompt

From [02](02-multi-agent-orchestration.md): subagents do **not** inherit the coordinator's history. Here is what that means for configuration:

> **Subagents don't automatically inherit parent context, and they don't share memory between invocations.** Each call starts fresh. The **prompt the coordinator writes** is the subagent's only window into the task.

### What goes in a good delegation prompt

| Include | Why |
|---|---|
| The **user's original goal** | So the subagent understands the purpose |
| **Constraints** (dates, regions, sources to avoid, format) | The subagent never saw them |
| **Complete prior findings** it needs to build on | No access to earlier subagent results otherwise |
| **Scope boundaries** (what to cover, what not to) | Avoids overlap with other subagents |
| **Output format** | So the coordinator can parse and combine results |

### Example: passing findings to the synthesis subagent

❌ **Wrong:** "Write the report based on the research we did."
The synthesizer has no idea what research was done.

✅ **Right:** include the **actual findings** in the prompt:

```text
Write a report on: "Impact of AI on creative industries".
Audience: business executives. Length: about 1,500 words.

Use ONLY these findings. Keep every citation.

<web_findings>
[{"claim": "...", "source_url": "https://...", "source_title": "...", "date": "2025-03-14"}, ...]
</web_findings>

<document_findings>
[{"claim": "...", "document": "industry_survey.pdf", "page": 12}, ...]
</document_findings>

If two sources conflict, present both and say which is more recent.
```

---

## 4. Use structured formats to preserve attribution

When context passes between agents, **metadata gets lost easily**. If the web researcher returns plain prose ("Studies show AI music tools grew 40%…"), the synthesizer can't cite the source, because the URL is gone.

**Solution:** pass data in a **structured format** that keeps **content** separate from **metadata**:

```json
{
  "claim": "AI-assisted music production tools grew 40% in user adoption in 2024.",
  "source_url": "https://example.org/report",
  "source_title": "Music Tech Annual Report",
  "document_name": null,
  "page_number": null,
  "publication_date": "2025-02-01"
}
```

**Benefits:**

- The final report can **cite every claim**.
- The coordinator can **check** that sources exist and are recent.
- Conflicts are easier to spot (same topic, different sources, different numbers).

> **Exam pattern:** "The final report has claims with no citations, though the search agent found good sources." The root cause is usually that **attribution was lost when passing context**. The fix: a structured format with separate source fields.

---

## 5. Spawning subagents in parallel

If subtasks are **independent** (e.g. research on music, film, and publishing), run them **at the same time**.

**How:** the coordinator emits **multiple spawning tool calls in a single response**.

```
Coordinator response (ONE turn):
  tool_use: Task → web-researcher  "subtopic: music"
  tool_use: Task → web-researcher  "subtopic: film"
  tool_use: Task → web-researcher  "subtopic: publishing"
```

The three subagents run in parallel. All three results come back together.

**Compare: sequential (slow).**

```
Turn 1: Task → music      … wait …
Turn 2: Task → film       … wait …
Turn 3: Task → publishing … wait …
```

Total time ≈ the sum of all three, instead of about the longest one.

**When to stay sequential:** when one subtask **needs the output** of another (e.g. synthesis needs research results first).

**How to encourage parallel calls:** say it in the coordinator's system prompt, e.g. *"When subtasks are independent, delegate them all in the same response."*

---

## 6. Write goals and quality criteria, not step-by-step scripts

A coordinator can instruct subagents in two ways:

| Style | Example | Result |
|---|---|---|
| **Step-by-step procedure** | "1. Search 'AI music'. 2. Open the first 3 results. 3. Summarize each in 2 sentences." | Rigid. Fails if the first results are poor or irrelevant. The subagent can't adapt. |
| **Goals + quality criteria** | "Find how AI changed music production since 2023. Aim for 5–8 findings from different credible sources (industry reports, academic studies, major news). Prefer sources from 2024 onward. Note any disagreement between sources." | The subagent chooses its own searches and adapts to what it finds. |

> **Exam pattern:** Prefer prompts that state **research goals and quality criteria** rather than procedural steps. Subagents are capable agents. Give them the "what" and the "how good", and let them decide the "how".

---

## 7. Real-world case study

**Situation:** A market-intelligence team built a coordinator with three subagents. Problems in testing:

1. **Slow:** the coordinator delegated one subtopic per turn. A five-subtopic report took about 5× as long as one search.
2. **No citations:** the search subagent returned paragraphs of prose; URLs were lost before synthesis.
3. **Off-target research:** the user asked for "EU market only", but subagents returned US data.

**Fixes:**

1. Updated the coordinator prompt: *"Delegate independent subtopics in a single response."* Run time dropped to roughly the time of the slowest single search.
2. Changed the search subagent's output to **structured JSON** with `source_url`, `title`, and `date` fields. Every claim in the report became citable.
3. Delegation prompts now **include the user's constraints explicitly** ("EU market only, 2024 onward").

---

## Common Q&A

**Q: My subagent doesn't know what the user asked. Is the SDK broken?**
A: No. Subagents start with fresh context by design. The coordinator must include the goal and constraints in the delegation prompt.

**Q: Does a subagent remember what it did the last time it was called?**
A: No. Each invocation is independent; there is no shared memory between calls. Pass in anything it needs.

**Q: How do I make subagents run in parallel?**
A: The coordinator must emit **multiple spawning (`Task`) tool calls in one response**, not one per turn.

**Q: Why use structured JSON between agents instead of prose?**
A: To keep **metadata** (source URL, document name, page) separate from content, so attribution survives each handoff.

**Q: Should I give each subagent all tools "just in case"?**
A: No. Restrict tools per subagent (`tools` in `AgentDefinition`). It's safer (least privilege) and helps the subagent stay focused.

**Q: The coordinator picks the wrong subagent for a task. What do I check?**
A: The subagents' **`description`** fields. They should clearly say what each subagent does and when to use it.

**Q: The coordinator never spawns any subagents. What do I check first?**
A: That the spawning tool (`Task` per the exam guide; `Agent` in current SDK versions) is in the coordinator's **allowed tools**.

---

## Key Takeaways

- Subagents are spawned via the **`Task` tool** (now named `Agent` in current SDKs), which must be in the coordinator's **`allowedTools`**.
- **`AgentDefinition`** sets each subagent's **description** (when to use it), **prompt** (how to behave), and **tools** (restrictions).
- **Pass everything explicitly:** goal, constraints, prior findings, scope, format. No inheritance, no shared memory.
- Use **structured data** that separates content from **metadata** to preserve attribution.
- **Parallel = multiple spawning calls in one response.**
- Give subagents **goals and quality criteria**, not rigid step-by-step scripts.
