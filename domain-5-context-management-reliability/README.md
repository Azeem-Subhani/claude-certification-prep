# Domain 5: Context Management & Reliability

**Exam weight:** 15%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

## Exam focus

- **Keeping exact facts (5.1).** Summaries lose numbers and dates. Keep them in a "case facts" block. Trim long tool outputs. Put key findings at the top.
- **Escalation (5.2).** Know the real triggers and the false ones. When a search returns several possible matches, ask for more details.
- **Error propagation (5.3).** Send structured error context to the coordinator. Never hide a failure as an empty success.
- **Long codebase sessions (5.4).** Recognize context degradation. Fix it with scratchpads, subagents, phase summaries, and manifests.
- **Human review (5.5) and provenance (5.6).** Measure accuracy by segment and calibrate confidence scores. Keep a source attached to every claim.

## Exam scenarios where this domain is primary

- Scenario 1: Customer Support Resolution Agent
- Scenario 2: Code Generation with Claude Code
- Scenario 3: Multi-Agent Research System
- Scenario 6: Structured Data Extraction

## Related guide material

- Preparation Exercise 1: Build a Multi-Tool Agent with Escalation Logic
- Preparation Exercise 4: Design and Debug a Multi-Agent Research Pipeline (error propagation, conflicting sources, provenance)
- Sample Questions 3 (escalation calibration), 7–9 (multi-agent research reliability)

## Study notes (read in order)

| File | Covers |
| --- | --- |
| [00 · Foundations and Glossary](00-foundations-and-glossary.md) | Context window as a budget, the four reliability risks, key terms |
| [01 · Preserving Critical Information](01-conversation-context-preservation.md) | 5.1: case facts blocks, trimming tool output, position effects, structured handoffs between agents |
| [02 · Escalation and Ambiguity Resolution](02-escalation-ambiguity-resolution.md) | 5.2: escalation triggers, honoring human requests, policy gaps, multiple matches |
| [03 · Error Propagation in Multi-Agent Systems](03-error-propagation-multi-agent.md) | 5.3: coordinator recovery decisions, anti-patterns, coverage annotations |
| [04 · Context in Large Codebase Exploration](04-large-codebase-context.md) | 5.4: context degradation, scratchpad files, manifests for crash recovery, `/compact` |
| [05 · Human Review and Confidence Calibration](05-human-review-confidence-calibration.md) | 5.5: hidden weak segments, stratified sampling, field-level confidence, review routing |
| [06 · Provenance and Uncertainty](06-provenance-and-uncertainty.md) | 5.6: claim-source mappings, conflicting sources, temporal data, rendering by content type |
| [Anti-patterns](anti-patterns.md) | Wrong approaches, why they fail, what to do instead, and exam traps |
| [07 · Practice Questions](07-scenario-practice-questions.md) | 25 original scenario questions with explanations |

Sources: the exam guide PDF, plus [Claude Code sessions docs](https://code.claude.com/docs/en/sessions) (`/compact`) and the Agent SDK docs used in Domain 1.

---

## 5.1 Manage conversation context to preserve critical information across long interactions

**Know**
- [ ] Progressive summarization risks: numbers, percentages, dates, and customer expectations become vague
- [ ] "Lost in the middle": models handle the start and end of long inputs well but may miss the middle
- [ ] Tool results pile up in context and use tokens out of proportion to their relevance (40+ fields when 5 matter)
- [ ] Full conversation history must be sent in later API requests to keep the conversation coherent

**Do**
- [ ] Extract transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block, outside summarized history
- [ ] Keep structured issue data in a separate context layer for multi-issue sessions
- [ ] Trim verbose tool outputs to relevant fields before they accumulate
- [ ] Put key-findings summaries at the start of aggregated inputs; organize details with explicit section headers
- [ ] Require subagents to include metadata (dates, source locations, methodological context) in structured outputs
- [ ] Have upstream agents return structured data (key facts, citations, relevance scores) instead of verbose reasoning when downstream context is limited

## 5.2 Design effective escalation and ambiguity resolution patterns

**Know**
- [ ] Good escalation triggers: customer asks for a human, policy exceptions/gaps, no meaningful progress possible
- [ ] Escalate immediately when the customer explicitly demands a human; offer to resolve when the issue is simple
- [ ] Sentiment and self-reported confidence are unreliable proxies for case complexity
- [ ] Multiple customer matches → ask for more identifiers, don't pick by heuristic

**Do**
- [ ] Add explicit escalation criteria with few-shot examples to the system prompt
- [ ] Honor explicit requests for a human immediately, without investigating first
- [ ] Acknowledge frustration while offering a fix when within capability; escalate if the customer repeats the request
- [ ] Escalate when policy is ambiguous or silent on the specific request (e.g. competitor price matching when policy covers only own-site adjustments)
- [ ] Ask for additional identifiers when a lookup returns multiple matches

## 5.3 Implement error propagation strategies across multi-agent systems

**Know**
- [ ] Structured error context (failure type, attempted query, partial results, alternatives) enables coordinator recovery
- [ ] Access failures (timeouts) differ from valid empty results (no matches)
- [ ] Generic statuses ("search unavailable") hide useful context
- [ ] Silently suppressing errors and terminating whole workflows on one failure are both anti-patterns

**Do**
- [ ] Return structured error context: failure type, what was attempted, partial results, alternatives
- [ ] Distinguish access failures from valid empty results
- [ ] Recover locally from transient failures in subagents; propagate only unresolved errors with attempts and partial results
- [ ] Annotate synthesis output with coverage: which findings are well-supported, which areas have gaps due to unavailable sources

## 5.4 Manage context effectively in large codebase exploration

**Know**
- [ ] Context degradation in long sessions: inconsistent answers, "typical patterns" instead of the specific classes found earlier
- [ ] Scratchpad files persist key findings across context boundaries
- [ ] Subagent delegation isolates verbose exploration; the main agent keeps high-level coordination
- [ ] Structured state persistence for crash recovery: agents export state; the coordinator loads a manifest on resume

**Do**
- [ ] Spawn subagents for specific questions ("find all test files", "trace refund flow dependencies")
- [ ] Maintain scratchpad files of key findings and refer back to them
- [ ] Summarize one phase's findings and inject them into the next phase's subagents
- [ ] Design crash recovery with structured state exports (manifests) loaded on resume
- [ ] Use `/compact` to reduce context during long exploration sessions

## 5.5 Design human review workflows and confidence calibration

**Know**
- [ ] Aggregate accuracy (e.g. 97% overall) can hide poor performance on specific document types or fields
- [ ] Stratified random sampling measures error rates in high-confidence extractions and finds new error patterns
- [ ] Field-level confidence scores, calibrated on labeled validation sets, route review attention
- [ ] Validate accuracy by document type and field before automating high-confidence extractions

**Do**
- [ ] Run stratified random sampling of high-confidence extractions for ongoing error measurement
- [ ] Analyze accuracy by document type and field before reducing human review
- [ ] Output field-level confidence; calibrate thresholds with labeled validation sets
- [ ] Route low-confidence or ambiguous/contradictory extractions to human review, prioritizing limited reviewer capacity

## 5.6 Preserve information provenance and handle uncertainty in multi-source synthesis

**Know**
- [ ] Attribution is lost when findings are summarized without claim-source mappings
- [ ] Synthesis must preserve and merge structured claim-source mappings
- [ ] Conflicting statistics from credible sources: annotate the conflict with attribution, don't pick one arbitrarily
- [ ] Temporal data: require publication/collection dates so time differences aren't read as contradictions

**Do**
- [ ] Require subagents to output claim-source mappings (URLs, document names, excerpts) that survive synthesis
- [ ] Structure reports into well-established vs contested findings, preserving source characterizations and methodology
- [ ] Pass conflicting values through, annotated, and let the coordinator decide how to reconcile before synthesis
- [ ] Require publication/collection dates in structured outputs
- [ ] Render content appropriately: financial data as tables, news as prose, technical findings as structured lists

---

## Key themes to remember

- **Facts don't belong in summaries.** Keep exact values in a persistent structured block.
- **Trim early, structure always.** Only relevant fields enter context.
- **Escalate on clear criteria**, not on mood or self-reported confidence.
- **Errors carry context**: what failed, what was tried, what partial results exist.
- **Write it down**: scratchpads and manifests survive context limits and crashes.
- **Averages hide failures**: measure by segment, sample high-confidence output.
- **Every claim keeps its source and date**; conflicts are shown, not hidden.
- **Escalate at once** on an explicit request for a human, and on a policy gap. When several records match, ask for another identifier.
- **Never abort everything** because one subagent failed. Continue and add coverage notes.
- **Match the format to the content:** tables for financial data, prose for news, structured lists for technical findings.

## Worked scenario

**Setting:** A support agent for an online electronics store handles long chats. These often involve several orders.

**Step 1: Keep the facts.**
- A chat runs 45 turns about two orders.
- The agent's code keeps a case facts block with both order numbers, amounts, dates, and statuses.
- Older turns are summarized, but the facts block is sent unchanged in every request.

**Step 2: Trim the lookup.**
- `lookup_order` returns 52 fields.
- A `PostToolUse` hook keeps only 6 of them: order ID, items, amount, ship date, delivery status, and return eligibility.

**Step 3: Ambiguity.**
- The customer gives only the name "Sam Lee".
- `get_customer` returns three matches.
- The agent asks for the email address on the account. It does not guess.

**Step 4: Frustration.**
- The customer writes, "This is ridiculous, your service is terrible."
- The issue is a standard late-delivery credit, which is covered by policy.
- The agent apologizes and offers the credit.
- The customer replies, "I want to talk to a person." The agent escalates at once, with a structured handoff.

**Step 5: Policy gap.**
- Another customer asks the store to match a competitor's price.
- The policy covers only the store's own price changes.
- This is a gap, so the agent escalates instead of making up a rule.

**Step 6: Tool failure.**
- The returns system times out.
- The tool reports `failure_type: timeout` after two local retries.
- The agent tells the customer that it cannot check return status right now. It does not claim "no returns found".

**Step 7: Review quality.**
- The team wants to auto-approve refunds that the agent marks as "high confidence".
- Before doing so, they calibrate the confidence scores on 500 labeled past cases.
- They find poor accuracy for one product category, and keep human review for it.
- They keep random sampling of the auto-approved cases.

## Sources checked

Each technical claim in this folder was checked against official docs on 2026-10-15.
**Verdicts:** CONFIRMED = the docs agree. PARTIAL = they agree with limits. CONFLICT = the docs say something different. NOT FOUND = the docs do not say this.
Conflicts with the exam guide are listed in [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).

| ID | Claim | Verdict | What the docs say | Source |
|---|---|---|---|---|
| A1 | The API is stateless. Each request must include the full history. | CONFIRMED | You always send the full conversation history. | platform.claude.com/docs/en/build-with-claude/working-with-messages |
| A9 | Prompt caching exists | CONFIRMED | Uses `cache_control` breakpoints. The default cache lifetime is 5 minutes, with an optional 1-hour TTL (time to live). Cache reads cost much less than normal input. (The exam tests only that caching exists.) | platform.claude.com/docs/en/build-with-claude/prompt-caching |
| C11 | `/compact` and auto-compaction | CONFIRMED | `/compact [instructions]` summarizes. Auto-compaction runs at a threshold. | code.claude.com/docs/en/commands; code.claude.com/docs/en/context-window |

## My notes

<!-- Add study notes, lab links, and weak areas here -->
