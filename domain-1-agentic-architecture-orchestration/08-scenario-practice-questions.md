# 08 · Scenario Practice Questions

**Exam link:** All of Domain 1 (Task Statements 1.1–1.7).
**How to use:** Answer each question before you read the explanation. Each answer links to the file that teaches the concept. These are **original practice questions** written for study. They are not taken from the official exam or practice test.

**Exam format reminder (from the exam guide):** multiple choice, scenario-based. The exam shows 4 of 6 scenarios at random. Domain 1 is primary in **Customer Support Resolution Agent** and **Multi-Agent Research System**, and also appears in **Developer Productivity with Claude**.

---

## How to approach Domain 1 questions

1. **Find the real requirement.** Is it *must always* (→ code), or *should usually* (→ prompt)?
2. **Find the root cause, not the symptom.** Bad output from a subagent? Check what the coordinator sent it.
3. **Eliminate "more prompt" answers when the rule is critical.** Capital letters, more examples, and "MANDATORY" are still probabilistic.
4. **Eliminate over-engineering.** A full pipeline for a simple query, or a bigger model for a decomposition problem, is usually wrong.
5. **Prefer structured signals.** `stop_reason` over text parsing; JSON with metadata over prose.

---

## Scenario A: Customer Support Resolution Agent

*You are building a support agent with the Claude Agent SDK. It uses MCP tools: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`. Goal: high first-contact resolution, with correct escalation.*

### A1
Your loop code ends the conversation when Claude's reply contains the phrase "Is there anything else I can help with?". Some customers report that their refund was promised but never processed. What is the best fix?

- A. Add the phrase "only say 'anything else' after all tools are done" to the system prompt.
- B. Continue the loop while `stop_reason == "tool_use"` and stop when it is `"end_turn"`.
- C. Lower the iteration cap so conversations end sooner.
- D. Check whether the response contains any text block and stop if it does.

<details><summary>Answer</summary>

**B.** The loop should be driven by `stop_reason`, the structured signal. A is still text-based and probabilistic. C makes early stops worse. D is a known anti-pattern: Claude often writes text *and* calls a tool in one response. → [01](01-agentic-loops.md)
</details>

### A2
Logs show that in 6% of conversations the agent calls `process_refund` before `get_customer` has returned a verified customer. What change most reliably prevents this?

- A. Add few-shot examples that always show `get_customer` first.
- B. Move the verification instruction to the top of the system prompt in capital letters.
- C. Add a programmatic prerequisite that blocks `process_refund` until a verified customer ID is recorded from `get_customer`'s output.
- D. Switch to a larger model that follows instructions better.

<details><summary>Answer</summary>

**C.** Identity verification before a financial action needs **deterministic** enforcement. A, B, and D reduce the failure rate but can't remove it. → [04](04-workflow-enforcement-and-handoffs.md)
</details>

### A3
Company policy: refunds over $500 need a manager. You want the agent to keep handling smaller refunds automatically. What is the best design?

- A. A `PreToolUse` hook on `process_refund` that denies amounts over $500 and tells the agent to call `escalate_to_human`.
- B. A `PostToolUse` hook that reverses refunds over $500 after they are processed.
- C. A system prompt rule: "Never refund more than $500."
- D. Remove `process_refund` from the agent and escalate every refund.

<details><summary>Answer</summary>

**A.** `PreToolUse` blocks the call *before* money moves and redirects to the alternative workflow. B acts too late. C is probabilistic. D destroys automation for the valid cases. → [05](05-agent-sdk-hooks.md)
</details>

### A4
`lookup_order` (legacy system) returns dates as Unix timestamps and status as numbers. `get_shipment` (new API) returns ISO 8601 dates and status words. The agent sometimes makes wrong date comparisons. What is the most robust fix?

- A. Add a table to the system prompt explaining every format and status code.
- B. A `PostToolUse` hook that converts all dates to ISO 8601 and status codes to words before the model sees results.
- C. Ask the model to double-check all date comparisons.
- D. Use only one of the two tools.

<details><summary>Answer</summary>

**B.** Normalize heterogeneous data **deterministically** in a hook. A and C keep the conversion in probabilistic model reasoning. D removes needed capability. → [05](05-agent-sdk-hooks.md)
</details>

### A5
A customer writes: "My order arrived broken, I was charged twice, and I need to update my address." What is the best agent behavior?

- A. Handle the first issue and ask the customer to open new tickets for the others.
- B. Escalate immediately because there are multiple issues.
- C. Split the message into three items, investigate each using the shared verified customer context, and reply with one unified resolution.
- D. Start three separate conversations so each issue has a clean context.

<details><summary>Answer</summary>

**C.** Decompose multi-concern requests, share context, and synthesize one answer. A hurts first-contact resolution; B over-escalates; D repeats verification and risks conflicting replies. → [04](04-workflow-enforcement-and-handoffs.md)
</details>

### A6
The agent escalates to a human with the note "Customer needs help with billing." Human agents complain they must re-interview every customer. What should change?

- A. Give human agents access to the full raw transcript.
- B. Make the agent compile a structured handoff: customer ID, issue summary, root cause, actions taken, amount, recommended action, escalation reason.
- C. Escalate less often.
- D. Ask the customer to summarize their issue before escalation.

<details><summary>Answer</summary>

**B.** Humans often don't have, or don't have time to read, the transcript. A structured handoff lets them act immediately. A shifts the reading burden to the human; C and D don't fix the handoff quality. → [04](04-workflow-enforcement-and-handoffs.md)
</details>

---

## Scenario B: Multi-Agent Research System

*A coordinator agent delegates to a web search subagent, a document analysis subagent, and a synthesis subagent, and produces cited reports.*

### B1
The topic is "The impact of remote work on cities." The final report is well written and accurate but discusses only commercial real estate. Every subagent completed its assigned work well. What is the most likely root cause?

- A. The web search subagent used poor search queries.
- B. The synthesis subagent dropped findings.
- C. The coordinator decomposed the topic too narrowly.
- D. The model's context window was too small.

<details><summary>Answer</summary>

**C.** If every subagent did its assigned job well and coverage is still missing, the **plan** was too narrow (transport, housing, tax revenue, retail, etc. were never assigned). → [02](02-multi-agent-orchestration.md)
</details>

### B2
The user said "use sources from 2024 onward only." The search subagent returns many 2019–2021 sources. What is the best fix?

- A. Add a `PostToolUse` hook to delete old sources from the synthesis output.
- B. Have the coordinator include the date constraint explicitly in the delegation prompt.
- C. Configure the search subagent to inherit the coordinator's conversation history.
- D. Ask the synthesis subagent to ignore old sources.

<details><summary>Answer</summary>

**B.** Subagents have **isolated context**; they never saw the user's constraint. C isn't how subagents work. A and D treat the symptom late in the pipeline. → [03](03-subagent-configuration.md)
</details>

### B3
The final report contains many claims without citations, even though the search subagent found good sources. What is the most likely cause and fix?

- A. The synthesis model is too small; use a larger model.
- B. Search results were passed as plain prose, which lost the URLs; pass structured findings with separate source fields.
- C. Too many sources; limit the search to three results.
- D. The coordinator should write the report itself.

<details><summary>Answer</summary>

**B.** Attribution is lost when content and metadata aren't separated. Structured formats preserve `source_url`, title, and date across handoffs. → [03](03-subagent-configuration.md)
</details>

### B4
Research on five independent subtopics takes five times as long as a single search. Logs show the coordinator delegates one subtopic per turn. What should change?

- A. Have the coordinator emit all five spawning (`Task`) calls in a single response.
- B. Merge all subtopics into one subagent call.
- C. Lower each subagent's `max_turns`.
- D. Use a faster model for the coordinator.

<details><summary>Answer</summary>

**A.** Parallel spawning = **multiple spawning calls in one coordinator response**. B loses focus and parallelism; C and D don't address sequential delegation. → [03](03-subagent-configuration.md)
</details>

### B5
Your coordinator never delegates. It tries to answer every research question itself. The `AgentDefinition`s look correct. What should you check first?

- A. Whether the subagents' prompts are long enough.
- B. Whether the spawning tool (`Task`) is included in the coordinator's allowed tools.
- C. Whether the synthesis subagent has web search access.
- D. Whether the coordinator uses the same model as the subagents.

<details><summary>Answer</summary>

**B.** The coordinator can only spawn subagents if the spawning tool is allowed. (Current SDK versions call it `Agent`; the exam guide uses `Task`.) → [03](03-subagent-configuration.md)
</details>

### B6
A user asks, "What does the acronym 'GDPR' stand for?" The system runs web search, document analysis, and synthesis, and takes 90 seconds. What design improvement is best?

- A. Add caching to the web search subagent.
- B. Have the coordinator analyze query complexity and select subagents dynamically, answering simple questions directly.
- C. Run the three subagents in parallel.
- D. Remove the document analysis subagent.

<details><summary>Answer</summary>

**B.** Always routing through the full pipeline is an anti-pattern. A and C speed up unnecessary work; D removes a needed capability for complex queries. → [02](02-multi-agent-orchestration.md)
</details>

### B7
Which design gives the best observability and the most consistent error handling?

- A. Subagents send results directly to each other in a chain.
- B. All subagent communication routes through the coordinator.
- C. Each subagent writes results to a shared file that others read.
- D. The synthesis subagent calls the search subagent directly when it needs more data.

<details><summary>Answer</summary>

**B.** Hub-and-spoke routing gives one place for logging, error handling, and controlled information flow. → [02](02-multi-agent-orchestration.md)
</details>

### B8
The coordinator's prompt to subagents reads: "1) Search 'X'. 2) Open the top 3 results. 3) Summarize each in two sentences." Results are often poor when the top results are irrelevant. What is a better approach?

- A. Increase to the top 10 results.
- B. State the research goal and quality criteria (source types, recency, number of findings) and let the subagent choose its own search strategy.
- C. Add a fourth step: "check if the results are relevant."
- D. Replace the subagent with a fixed search API call.

<details><summary>Answer</summary>

**B.** Goals + quality criteria let capable subagents adapt. Rigid procedures fail on unexpected results. → [03](03-subagent-configuration.md)
</details>

---

## Scenario C: Developer Productivity with Claude

*You use the Agent SDK and Claude Code to help engineers explore codebases, review code, and plan changes.*

### C1
A pull request changes 18 files. A single review pass gives detailed comments on the first files, shallow ones on the rest, and misses that a renamed function broke callers in other files. What is the best approach?

- A. Use a model with a larger context window.
- B. Per-file local review passes, plus a separate cross-file integration pass.
- C. Ask the developer to split the PR into 18 PRs.
- D. Add "review every file with equal care" to the prompt.

<details><summary>Answer</summary>

**B.** Focused local passes avoid attention dilution; the integration pass catches cross-file issues like broken callers. A and D don't fix dilution; C is unrealistic. → [06](06-task-decomposition.md)
</details>

### C2
Task: "Add comprehensive tests to this legacy service." Which decomposition is best?

- A. A fixed list: write tests for each file in alphabetical order.
- B. Map the structure, identify high-impact areas, create a prioritized plan, and adapt it as dependencies are discovered.
- C. One prompt: "Write all missing tests."
- D. Prompt chaining: generate tests → run tests → fix tests, applied to each file in order.

<details><summary>Answer</summary>

**B.** Open-ended tasks need **dynamic, adaptive decomposition**. A and D are fixed sequences that ignore what is discovered; C overloads a single pass. → [06](06-task-decomposition.md)
</details>

### C3
Your team does a monthly compliance check on every service: extract config → compare to policy → write a findings table. Which pattern fits best?

- A. Dynamic adaptive decomposition.
- B. Prompt chaining (fixed sequential pipeline).
- C. A multi-agent research system.
- D. A single prompt with all services at once.

<details><summary>Answer</summary>

**B.** The steps are known and identical every time: a predictable task → prompt chaining. → [06](06-task-decomposition.md)
</details>

### C4
Yesterday, an agent analyzed 50 files to plan a refactor in a named session. Overnight, a teammate changed two of those files. What is the most efficient reliable next step?

- A. Start a brand-new session and re-analyze all 50 files.
- B. Resume the named session and tell it exactly which two files changed so it re-reads them.
- C. Resume the session without mentioning the changes.
- D. Fork the session and hope the fork notices.

<details><summary>Answer</summary>

**B.** Most context is still valid → **resume + targeted re-analysis**. A wastes work; C risks stale tool results; D doesn't solve staleness. → [07](07-session-state-resumption-forking.md)
</details>

### C5
After a large merge, 30 of the 50 previously analyzed files changed significantly. What is best?

- A. Resume and continue as normal.
- B. Resume and tell the agent all 30 file names.
- C. Start a fresh session and inject a structured summary of the earlier conclusions.
- D. Fork the old session.

<details><summary>Answer</summary>

**C.** When much of the prior context is stale, a fresh session with an accurate summary is more reliable than a history full of outdated tool results. → [07](07-session-state-resumption-forking.md)
</details>

### C6
You've completed a codebase analysis and want to compare two refactoring strategies without repeating the analysis. What should you use?

- A. `--continue` twice in two terminals.
- B. `fork_session` (or `--fork-session`) to create two branches from the shared analysis.
- C. Two brand-new sessions.
- D. One session, trying strategy A and then undoing it before trying strategy B.

<details><summary>Answer</summary>

**B.** Forking creates independent branches from a shared baseline. A interleaves messages into one transcript; C repeats the analysis; D mixes both approaches in one history. → [07](07-session-state-resumption-forking.md)
</details>

---

## Mixed concept checks

### M1
Which statement about iteration caps in an agentic loop is correct?
- A. They should be the main way the loop ends.
- B. They are a safety net; the loop should normally end on `end_turn`.
- C. They are unnecessary if the prompt is good.
- D. They should be set to 1 for safety.

<details><summary>Answer</summary>**B.** → [01](01-agentic-loops.md)</details>

### M2
Claude returns three `tool_use` blocks in one response. How should you return the results?
- A. Three separate user messages, one per result.
- B. One user message containing three `tool_result` blocks with matching `tool_use_id`s.
- C. Only the result of the first tool.
- D. One assistant message with all three results.

<details><summary>Answer</summary>**B.** → [01](01-agentic-loops.md)</details>

### M3
Which requirement most clearly calls for a hook rather than a prompt instruction?
- A. "Use a friendly tone."
- B. "Never expose full payment card numbers to the model."
- C. "Prefer short answers."
- D. "Greet returning customers by name."

<details><summary>Answer</summary>**B.** It must always hold, and redacting in a `PostToolUse` hook ensures the model never sees the data. → [05](05-agent-sdk-hooks.md)</details>

### M4
A subagent is called twice in the same run. On the second call, what does it remember from the first?
- A. Everything.
- B. Only its own previous output.
- C. Nothing, unless the coordinator includes it in the new prompt.
- D. Whatever the coordinator's system prompt contains.

<details><summary>Answer</summary>**C.** → [03](03-subagent-configuration.md)</details>

### M5
When is a model-driven loop a *worse* choice than a fixed workflow?
- A. When the steps depend on what is discovered.
- B. When the task is predictable, identical every time, and must be easy to audit.
- C. When the user's request is ambiguous.
- D. When different customers need different tools.

<details><summary>Answer</summary>**B.** → [00](00-foundations-and-glossary.md), [06](06-task-decomposition.md)</details>

---

## Score yourself

| Score | Next step |
|---|---|
| 22–25 | Ready for Domain 1. Revisit any you guessed. |
| 17–21 | Re-read the files linked from your wrong answers. |
| < 17 | Re-read files 00–07 in order, then try again in a few days. |

---

## Key Takeaways

- Most Domain 1 questions test **one of five ideas**: `stop_reason` loop control, isolated subagent context, deterministic enforcement (gates and hooks), choosing the right decomposition, and handling stale session context.
- **Find the root cause.** Many wrong options fix a symptom downstream.
- **"Improve the prompt" is rarely right** when the scenario involves money, identity, or compliance.
- **"Do more" is rarely right** (full pipeline, bigger model, bigger context) when the real issue is design.
