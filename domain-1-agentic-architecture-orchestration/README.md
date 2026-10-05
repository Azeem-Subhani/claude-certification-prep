# Domain 1: Agentic Architecture & Orchestration

**Exam weight:** 27% (largest domain)
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

## Exam scenarios where this domain is primary

- Scenario 1: Customer Support Resolution Agent
- Scenario 3: Multi-Agent Research System
- Scenario 4: Developer Productivity with Claude (secondary)

## Related guide material

- Preparation Exercise 1: Build a Multi-Tool Agent with Escalation Logic
- Preparation Exercise 4: Design and Debug a Multi-Agent Research Pipeline
- Sample Questions 1–3 (customer support) and 7–9 (multi-agent research)

## Study notes (read in order)

| File | Covers |
| --- | --- |
| [00 · Foundations and Glossary](00-foundations-and-glossary.md) | Core terms, workflow vs agent, deterministic vs probabilistic |
| [01 · Agentic Loops](01-agentic-loops.md) | 1.1: `stop_reason`, tool results, loop anti-patterns |
| [02 · Multi-Agent Orchestration](02-multi-agent-orchestration.md) | 1.2: hub-and-spoke, coordinator jobs, decomposition risks |
| [03 · Subagent Configuration](03-subagent-configuration.md) | 1.3: `Task` tool, `AgentDefinition`, context passing, parallel spawning |
| [04 · Enforcement and Handoffs](04-workflow-enforcement-and-handoffs.md) | 1.4: prerequisite gates, multi-concern requests, structured handoffs |
| [05 · Agent SDK Hooks](05-agent-sdk-hooks.md) | 1.5: `PreToolUse` interception, `PostToolUse` normalization |
| [06 · Task Decomposition](06-task-decomposition.md) | 1.6: prompt chaining vs adaptive decomposition |
| [07 · Sessions](07-session-state-resumption-forking.md) | 1.7: resume, fork, stale context |
| [08 · Practice Questions](08-scenario-practice-questions.md) | 25 original scenario questions with explanations |

Sources: the exam guide PDF, plus Anthropic docs for SDK details: [Agent SDK hooks](https://code.claude.com/docs/en/agent-sdk/hooks), [Agent SDK sessions](https://code.claude.com/docs/en/agent-sdk/sessions), [Agent SDK subagents](https://code.claude.com/docs/en/agent-sdk/subagents), [Claude Code sessions](https://code.claude.com/docs/en/sessions), [Handling tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls). Where the guide and current docs differ (e.g. `Task` vs `Agent` tool name), the notes give both.

---

## 1.1 Design and implement agentic loops for autonomous task execution

**Know**
- [ ] Loop lifecycle: send request, inspect `stop_reason` (`"tool_use"` vs `"end_turn"`), execute requested tools, return results for the next iteration
- [ ] Tool results are appended to conversation history so the model can reason about its next action
- [ ] Model-driven decisions (Claude picks the next tool from context) vs pre-configured decision trees or fixed tool sequences

**Do**
- [ ] Continue the loop on `stop_reason == "tool_use"`; terminate on `"end_turn"`
- [ ] Add tool results to context between iterations
- [ ] Avoid anti-patterns: parsing natural-language signals to decide termination, arbitrary iteration caps as the *primary* stop mechanism, checking assistant text for a "done" indicator

## 1.2 Orchestrate multi-agent systems with coordinator-subagent patterns

**Know**
- [ ] Hub-and-spoke: a coordinator manages all inter-subagent communication, error handling, and information routing
- [ ] Subagents run with isolated context; they do not inherit the coordinator's conversation history
- [ ] Coordinator responsibilities: task decomposition, delegation, result aggregation, choosing which subagents to invoke based on query complexity
- [ ] Risk: overly narrow decomposition by the coordinator leads to incomplete coverage of broad topics

**Do**
- [ ] Have the coordinator analyze the query and dynamically select subagents instead of always running the full pipeline
- [ ] Partition scope across subagents to minimize duplication (distinct subtopics or source types per agent)
- [ ] Build iterative refinement: coordinator checks synthesis output for gaps, re-delegates targeted queries, re-runs synthesis until coverage is sufficient
- [ ] Route all subagent communication through the coordinator for observability, consistent error handling, and controlled information flow

## 1.3 Configure subagent invocation, context passing, and spawning

**Know**
- [ ] The Task tool spawns subagents; the coordinator's `allowedTools` must include `"Task"`
- [ ] Subagent context must be passed explicitly in the prompt; no automatic inheritance or shared memory between invocations
- [ ] `AgentDefinition` config: description, system prompt, and tool restrictions per subagent type
- [ ] Fork-based session management for exploring divergent approaches from a shared analysis baseline

**Do**
- [ ] Put complete prior findings directly in the subagent prompt (e.g., web search results and document analysis passed to the synthesis subagent)
- [ ] Use structured data formats that separate content from metadata (source URLs, document names, page numbers) to preserve attribution
- [ ] Spawn parallel subagents by emitting multiple Task calls in a *single* coordinator response, not across separate turns
- [ ] Write coordinator prompts that state research goals and quality criteria rather than step-by-step procedures, so subagents can adapt

## 1.4 Implement multi-step workflows with enforcement and handoff patterns

**Know**
- [ ] Programmatic enforcement (hooks, prerequisite gates) vs prompt-based guidance for workflow ordering
- [ ] When compliance must be deterministic (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate
- [ ] Structured handoff protocols for mid-process escalation: customer details, root cause analysis, recommended actions

**Do**
- [ ] Block downstream tool calls until prerequisites complete (e.g., block `process_refund` until `get_customer` returns a verified customer ID)
- [ ] Decompose multi-concern requests into distinct items, investigate each in parallel with shared context, then synthesize one resolution
- [ ] Compile structured handoff summaries (customer ID, root cause, refund amount, recommended action) for human agents who can't see the transcript

## 1.5 Apply Agent SDK hooks for tool call interception and data normalization

**Know**
- [ ] `PostToolUse` hooks intercept tool results for transformation before the model processes them
- [ ] Hooks that intercept outgoing tool calls can enforce compliance rules (e.g., block refunds above a threshold)
- [ ] Hooks give deterministic guarantees; prompt instructions give probabilistic compliance

**Do**
- [ ] Use `PostToolUse` to normalize mixed formats across MCP tools (Unix timestamps, ISO 8601, numeric status codes)
- [ ] Intercept policy-violating calls (e.g., refunds over $500) and redirect to an alternative workflow such as human escalation
- [ ] Choose hooks over prompts when a business rule needs guaranteed compliance

## 1.6 Design task decomposition strategies for complex workflows

**Know**
- [ ] Fixed sequential pipelines (prompt chaining) vs dynamic, adaptive decomposition driven by intermediate findings
- [ ] Prompt chaining for reviews: analyze each file individually, then run a cross-file integration pass
- [ ] Adaptive investigation plans that generate subtasks based on what each step discovers

**Do**
- [ ] Match the pattern to the work: prompt chaining for predictable multi-aspect reviews, dynamic decomposition for open-ended investigation
- [ ] Split large code reviews into per-file local passes plus a separate cross-file pass to avoid attention dilution
- [ ] For open-ended tasks (e.g., "add comprehensive tests to a legacy codebase"): map structure first, find high-impact areas, then build a prioritized plan that adapts as dependencies surface

## 1.7 Manage session state, resumption, and forking

**Know**
- [ ] `--resume <session-name>` continues a specific prior conversation
- [ ] `fork_session` creates independent branches from a shared analysis baseline to explore divergent approaches
- [ ] When resuming after code changes, tell the agent which previously analyzed files changed
- [ ] Starting a new session with a structured summary is more reliable than resuming with stale tool results

**Do**
- [ ] Use `--resume` with session names to continue named investigations across work sessions
- [ ] Use `fork_session` to compare parallel approaches (e.g., two testing strategies or refactoring approaches) from one codebase analysis
- [ ] Resume when prior context is mostly valid; start fresh with an injected summary when prior tool results are stale
- [ ] Tell a resumed session about specific file changes for targeted re-analysis instead of full re-exploration

---

## Key themes to remember

- **Deterministic beats probabilistic** for anything that must always happen (hooks and gates over prompt instructions).
- **Context is explicit**: subagents only know what you put in their prompt.
- **Coordinator owns the flow**: routing, error handling, aggregation, and gap-filling all go through it.
- **Loop control comes from `stop_reason`**, not from parsing text or iteration caps.

## My notes

<!-- Add study notes, lab links, and weak areas here -->
