# Domain 1 Anti-patterns: Agentic Architecture & Orchestration

**Scope:** wrong designs for agent loops, multi-agent systems, enforcement, hooks, decomposition, and sessions (task statements 1.1 to 1.7).

---

## Anti-patterns

| # | Wrong approach | Why it fails | Do this instead | Topic |
|---|---|---|---|---|
| 1 | Stop the loop when Claude writes "done" or "task complete". | The wording changes from reply to reply. Claude may also say "done" while it is still working. | Stop only when `stop_reason` is `"end_turn"`. | [01](01-agentic-loops.md) |
| 2 | Use an iteration cap (for example, 10 rounds) as the main way to stop. | It cuts off long tasks or wastes calls on short ones. It hides the real stop signal. | Use `stop_reason`. Keep a cap only as a safety net, with a warning log. | [01](01-agentic-loops.md) |
| 3 | End the loop when the reply contains a text block. | Claude often writes text and calls a tool in the same reply. The agent then stops halfway. | Check `stop_reason`, not the content type. | [01](01-agentic-loops.md) |
| 4 | Run the tool but do not add its result to the history. | The API is stateless, so Claude never sees the result. It asks for the same data again. | Append the assistant reply and every `tool_result` before the next call. | [01](01-agentic-loops.md) |
| 5 | Set `tool_choice: "any"` so the agent "keeps working". | Claude must call a tool on every turn. It can never reach `end_turn`, so the loop never ends. | Use `"auto"` in a normal agent loop. | [01](01-agentic-loops.md) |
| 6 | Let a tool exception crash the loop, or drop the failed result. | The user gets no answer, or Claude does not know the tool failed. | Return a `tool_result` with `is_error: true` and a clear message. | [01](01-agentic-loops.md) |
| 7 | Let subagents talk to each other directly. | You lose one place for logging, error handling, and control of what each agent sees. | Use hub-and-spoke: all messages go through the coordinator. | [02](02-multi-agent-orchestration.md) |
| 8 | Expect a subagent to know the coordinator's history or its own earlier calls. | Each subagent call starts with an empty context. Missing facts lead to wrong or generic work. | Put everything the subagent needs into its prompt. | [02](02-multi-agent-orchestration.md), [03](03-subagent-configuration.md) |
| 9 | Run the full pipeline for every question. | Simple questions waste time and money on agents they do not need. | Let the coordinator pick only the subagents the question needs. | [02](02-multi-agent-orchestration.md) |
| 10 | Add more subagents when the report misses whole topics. | The coordinator split the topic too narrowly. New agents get the same narrow split. | Fix the coordinator's decomposition. | [02](02-multi-agent-orchestration.md) |
| 11 | Pass findings to the synthesis agent as plain summaries. | Source URLs, document names, and pages are lost, so the report has no citations. | Pass structured data that keeps content and metadata apart. | [03](03-subagent-configuration.md) |
| 12 | Give the synthesis agent search tools to fix missing citations. | It starts searching instead of synthesizing. The real cause (missing metadata upstream) stays. | Fix what the coordinator passes to the synthesis agent. | [03](03-subagent-configuration.md) |
| 13 | Start independent subagents one at a time, in separate turns. | The work runs in sequence, so it takes much longer. | Ask for several Task (Agent) calls in one coordinator response. | [03](03-subagent-configuration.md) |
| 14 | Give subagents step-by-step scripts. | The subagent cannot adapt to what it finds. | Give goals and quality criteria. | [03](03-subagent-configuration.md) |
| 15 | Rely on a stronger system prompt for money, security, or compliance rules. | Prompts lower the failure rate, but never to zero. One miss can be a wrong refund or a data leak. | Use code: a prerequisite gate or a `PreToolUse` hook. | [04](04-workflow-enforcement-and-handoffs.md) |
| 16 | Use few-shot examples or a routing classifier to fix skipped steps. | Examples improve consistency but guarantee nothing. A classifier only routes requests; it does not enforce order inside one agent. | Add a gate that blocks the later tool until the earlier step is done. | [04](04-workflow-enforcement-and-handoffs.md) |
| 17 | Write a human handoff that says "see the chat above". | The human agent cannot see the transcript. They must start again. | Send a standalone summary: customer ID, issue, root cause, amount, recommended action. | [04](04-workflow-enforcement-and-handoffs.md) |
| 18 | Answer only the first issue when a customer raises several. | The other issues stay open and the customer must ask again. | Split the issues, check them with shared context, and give one combined answer. | [04](04-workflow-enforcement-and-handoffs.md) |
| 19 | Use a `PostToolUse` hook to block a risky action. | `PostToolUse` runs after the tool. The action has already happened. | Use `PreToolUse` to block or redirect before the tool runs. | [05](05-agent-sdk-hooks.md) |
| 20 | Let the model convert mixed data formats (Unix time, ISO dates, status codes) itself. | The model is not consistent, so the same value can be read in different ways. | Normalize the data in a `PostToolUse` hook before Claude reads it. | [05](05-agent-sdk-hooks.md) |
| 21 | Review many files in one pass. | Attention dilution: early files get detail, later files get little, and results contradict each other. | One pass per file, plus one cross-file integration pass. | [06](06-task-decomposition.md) |
| 22 | Use a fixed pipeline for open-ended work (legacy code, research, unknown bugs). | The steps are not known in advance, so the plan misses what is found later. | Use dynamic adaptive decomposition: map, prioritize, and change the plan as you learn. | [06](06-task-decomposition.md) |
| 23 | Resume a session with a lot of stale history after big changes. | Old tool results stay in the history. The model trusts them and works on the wrong code. | Start fresh and inject a structured summary. (For a few known changes, see conflict R1 in [07](07-session-state-resumption-forking.md).) | [07](07-session-state-resumption-forking.md) |
| 24 | Re-read the whole codebase when only a few files changed. | It wastes time and fills the context. | Re-analyze only the changed files. | [07](07-session-state-resumption-forking.md) |
| 25 | Use `fork_session` to fix stale context. | A fork copies the same stale history into the new branch. | Fork only to compare approaches. Fix stale context with a fresh session and summary. | [07](07-session-state-resumption-forking.md) |

---

## Common exam traps

These are wrong answers the exam is likely to offer, and why each one is wrong.

1. **"Raise the iteration cap so long tasks finish."** A cap is only a safety net. It does not fix an agent that stops early. The real fix is to check `stop_reason`.
2. **"The reply has text, so the task is finished."** Text and a tool call can come in the same reply. Only `stop_reason == "end_turn"` means finished.
3. **"Write a stronger system prompt so the agent always verifies identity first."** The failure rate goes down but never to zero. Identity before refunds is a security rule, so it needs a gate or a hook.
4. **"Use `PostToolUse` to stop refunds over $500."** The refund has already run when `PostToolUse` fires. `PreToolUse` is the hook that can block it.
5. **"The subagent should remember its last call."** Subagents start empty on every call. Pass the needed context in the prompt.
6. **"The report has no citations, so improve the synthesis prompt."** The synthesis agent never received the sources. Fix what is passed to it.
7. **"Add more subagents to cover the missing topics."** Missing whole topics is a scope problem. Fix the coordinator's task split.
8. **"Let the two subagents message each other to save time."** This breaks hub-and-spoke, so you lose observability and error control. Route everything through the coordinator.
9. **"Use a bigger model or a bigger context window for uneven reviews."** Uneven reviews come from attention dilution, not from model size. Split the work into per-file passes plus a cross-file pass.
10. **"Pick the more advanced-sounding pattern."** The exam rewards matching the pattern to the task. Known steps need a pipeline; unknown scope needs adaptive decomposition.
11. **"Fork the session to get rid of stale results."** A fork copies the stale history too. Use a fresh session with a summary.
12. **"Use a routing classifier to stop the agent from skipping a step."** A classifier decides where a request goes. It cannot enforce the order of steps inside one agent.

> ⚠️ Some answers depend on exam framing that differs from today's product: for example, nested subagents (C3) and the Task/Agent tool name (C1a). See the Sources checked table in the [README](README.md) and [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).
