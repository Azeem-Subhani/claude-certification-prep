# 03 · Tool Distribution Across Agents and `tool_choice`

**Exam link:** Task Statement 2.3: *Distribute tools appropriately across agents and configure tool choice.*
**Prerequisites:** [01](01-tool-interface-design.md), and Domain 1 [03 · Subagent Configuration](../domain-1-agentic-architecture-orchestration/03-subagent-configuration.md) (`tools` in `AgentDefinition`).
**Scope note:** Domain 1 showed *how* to restrict a subagent's tools. This file explains *which* tools to give each agent and *why*, and how `tool_choice` controls whether a tool call happens at all.

---

## 1. More tools ≠ better agent

Each time Claude responds, it must consider **every tool it has** and pick one (or none). More tools means:

- **More decision complexity**: more options to compare, more chances of a wrong pick.
- **More overlap**: tools with similar purposes are easier to confuse (see [01](01-tool-interface-design.md)).
- **More context used**: every tool definition takes space in the prompt.

> **Exam principle:** An agent with **18 tools** selects less reliably than an agent with **4–5 focused tools**.

---

## 2. Agents misuse tools outside their specialty

If an agent has a tool that isn't part of its job, it **will** eventually use it, often badly.

**Example:** A **synthesis agent** (job: combine findings into a report) also has `web_search`. During writing, it notices a small gap and starts its own web searches. Problems:

- Its searches are unplanned and unchecked; sources may be low quality.
- It duplicates the search agent's work.
- Findings bypass the coordinator, so attribution and quality control are lost.

The fix is to **remove `web_search`** from the synthesis agent.

---

## 3. Scoped tool access

**Scoped tool access** = give each agent **only the tools its role needs**.

| Agent | Role | Tools |
|---|---|---|
| Coordinator | Plan, delegate, review | Spawning tool (`Task`), maybe `Read` |
| Web researcher | Find sources | `web_search`, `load_document` |
| Document analyst | Read user files | `Read`, `Grep`, `extract_data_points` |
| Synthesizer | Write the report | `verify_fact` (scoped cross-role tool, see §5) |

**Benefits:** better selection accuracy, less misuse, and **least privilege** (smaller blast radius if something goes wrong). Least privilege is also a security principle: an agent can't misuse a dangerous tool it doesn't have.

---

## 4. Replace generic tools with constrained ones

A **generic** tool can do almost anything. That's flexible but risky.

**Example:** `fetch_url(url)` can fetch *any* URL: internal admin pages, huge binary files, unrelated sites.

**Constrained alternative:** `load_document(url)`:

- accepts only URLs of supported document types (PDF, HTML article),
- **validates** the URL (allowed domains, no internal network addresses),
- returns cleaned text with metadata (title, date, source).

```python
from urllib.parse import urlparse

ALLOWED_SCHEMES = {"https"}
BLOCKED_HOST_SUFFIXES = (".internal", ".local")

def load_document(url: str) -> dict:
    """Constrained replacement for a generic fetch_url: validates before fetching."""
    parsed = urlparse(url)
    if parsed.scheme not in ALLOWED_SCHEMES or parsed.hostname is None:
        return {"isError": True, "errorCategory": "validation", "isRetryable": False,
                "message": "Only https:// document URLs are allowed"}
    if parsed.hostname.endswith(BLOCKED_HOST_SUFFIXES) or is_private_ip(parsed.hostname):
        return {"isError": True, "errorCategory": "permission", "isRetryable": False,
                "message": "Internal addresses are not allowed"}
    return fetch_and_clean_document(url)  # returns {"title", "text", "published", "source_url"}
```

**Why it's better:** more predictable outputs, safer (helps prevent requests to internal systems), and a clearer description for the model.

---

## 5. Scoped cross-role tools for frequent needs

Sometimes an agent **frequently** needs a small piece of another role's capability. Sending every such need back through the coordinator is slow.

**Example:** The synthesis agent often needs to check single facts ("Was this report published in 2024?"). Without a tool, each check means:

```
synthesizer → coordinator → web researcher → coordinator → synthesizer   (many turns, slow)
```

**Solution:** give the synthesis agent a **narrow, scoped** tool, `verify_fact(claim, source_url)`, that only checks a specific claim against a specific source. It is **not** a general web search.

**And keep the boundary:** complex cases (a claim needs new research, sources conflict) still go **through the coordinator**.

| Need | Route |
|---|---|
| Quick check of one claim against a known source | `verify_fact` directly (scoped cross-role tool) |
| New research or conflicting sources | Back to the coordinator → web researcher |

---

## 6. `tool_choice`: controlling whether a tool must be called

`tool_choice` is a Messages API parameter that controls **whether** Claude must call a tool, and **which** one.

| Setting | Behavior | Use when |
|---|---|---|
| `{"type": "auto"}` (default) | Claude decides: call a tool or reply with text | Normal agent turns |
| `{"type": "any"}` | Claude **must call a tool**, but chooses which | You need a tool call, never a plain text reply |
| `{"type": "tool", "name": "X"}` | Claude **must call tool X** | A specific tool must run (e.g. first step) |
| `{"type": "none"}` | Claude must **not** call tools | You want only a text answer this turn |

### Pattern A: Force a specific first step

**Goal:** Every document must go through `extract_metadata` **before** any enrichment tools run.

```python
# Turn 1: force metadata extraction first
first = client.messages.create(
    model=MODEL,
    max_tokens=4096,
    tools=[extract_metadata_tool, lookup_author_tool, classify_topic_tool],
    tool_choice={"type": "tool", "name": "extract_metadata"},
    messages=messages,
)
# ... run extract_metadata, append its tool_result ...

# Turn 2+: back to auto so the model can choose enrichment tools based on the metadata
follow_up = client.messages.create(
    model=MODEL, max_tokens=4096,
    tools=[extract_metadata_tool, lookup_author_tool, classify_topic_tool],
    tool_choice={"type": "auto"},
    messages=messages,
)
```

The forced call handles **step 1**. Later steps are processed in **follow-up turns** with normal (`auto`) selection.

### Pattern B: Guarantee a tool call instead of chat

**Goal:** In a data-extraction pipeline, the model must **always** return data through a tool (for structured output), never a friendly sentence like "Here's what I found…".

```python
response = client.messages.create(
    model=MODEL, max_tokens=4096,
    tools=[record_invoice_tool, record_receipt_tool],
    tool_choice={"type": "any"},      # must call one of these; can't reply with plain text
    messages=messages,
)
```

### ⚠️ Current-model note (real code vs exam)

The exam guide tests `"any"` and forced `{"type": "tool"}` as shown above, so **answer exam questions using them**.

In current Anthropic docs, the newest models (**Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1**) **reject** `tool_choice` `"any"` and `"tool"` with a 400 error. On those models, get the same result with:

- `{"type": "auto"}` plus a clear instruction naming the tool,
- `strict: true` on the tool definition (arguments always match the schema), or
- **structured outputs** (`output_config.format`) when you just need JSON back.

Older models (for example Claude Opus 5, Sonnet 5, Haiku 4.5) still accept forced `tool_choice`. Always check the docs for the model you use.

---

## 7. Real-world case study

**Situation:** A legal-research company gave **one** agent all 18 tools: search, PDF reading, citation formatting, email, calendar, billing lookup, and more.

**Symptoms:**
- It sometimes emailed draft summaries to clients instead of saving them.
- It used `web_search` when it should have searched the internal case database.
- Selection accuracy on a test set was poor.

**Changes:**
1. Split into a coordinator + 3 subagents, each with **4–5 tools**.
2. Removed email entirely from research agents; only a final "delivery" step, with **human approval**, can send email.
3. Replaced `fetch_url` with `load_case_document`, which accepts only approved legal-database URLs.
4. Gave the drafting agent a scoped `check_citation(citation)` tool instead of full search access.
5. Forced `extract_case_metadata` as the first tool call for each new document.

**Result:** Much better tool selection on the test set and no more unintended emails.

---

## Common Q&A

**Q: Is it bad to give an agent many tools if they're all well described?**
A: Good descriptions help, but decision complexity still grows with each tool. Prefer **fewer, focused tools per agent** and split roles across agents.

**Q: The synthesis agent sometimes searches the web itself. What's the fix?**
A: Remove `web_search` from its tools. If it truly needs quick checks often, give it a **narrow** tool like `verify_fact`, and route complex research through the coordinator.

**Q: What's the difference between `"any"` and forced tool choice?**
A: `"any"` forces **a** tool call but lets Claude pick which. `{"type": "tool", "name": "X"}` forces **that specific** tool.

**Q: If I force a tool on turn 1, do I keep forcing it?**
A: No. Use forced choice for the step that must happen first, then switch back to `auto` for follow-up turns.

**Q: When should I use `"none"`?**
A: When you want a text answer only, for example a final summary turn where no more actions should happen.

**Q: Why replace `fetch_url` with `load_document`?**
A: A constrained tool validates inputs, gives predictable outputs, and limits risk (e.g. it blocks internal URLs).

---

## Key Takeaways

- **Too many tools reduce selection reliability.** Aim for a small, focused set per agent (about 4–5).
- Agents **misuse tools outside their role**. Remove them.
- Use **scoped access** (least privilege) and **constrained tools** instead of generic ones.
- Give **narrow cross-role tools** for frequent needs; route complex cases through the coordinator.
- `tool_choice`: **`auto`** (model decides), **`any`** (must call some tool), **forced** (must call a named tool), **`none`** (no tools).
- Exam: forced choice for a **required first step**, `any` to **guarantee a tool call**. Real code on the newest models: use `auto` + `strict` or structured outputs instead.
