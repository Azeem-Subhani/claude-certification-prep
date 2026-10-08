# 07 · Scenario Practice Questions (Domain 5)

**Exam link:** All of Domain 5 (Task Statements 5.1–5.6).
**How to use:** Answer each question before you open the explanation. Each answer links to the file that teaches the concept. These are **original practice questions** written for study. They are not taken from the official exam or practice test.

**Where Domain 5 appears:** It is a primary domain in **Customer Support Resolution Agent**, **Code Generation with Claude Code**, **Multi-Agent Research System**, and **Structured Data Extraction**.

---

## How to approach Domain 5 questions

1. **Details lost over time?** → **Case facts block**, trimmed tool outputs, scratchpad files.
2. **Escalation question?** → Explicit triggers (human requested, policy gap, no progress), not sentiment or confidence.
3. **Error handling across agents?** → Structured context to the coordinator; no silent suppression, no abort-everything.
4. **"Can we automate?"** → Check accuracy **by segment**; calibrate; keep **stratified sampling**.
5. **Sources and conflicts?** → Keep claim-source mappings and dates; **annotate** conflicts.

---

## Scenario A: Customer Support Resolution Agent

### A1
In long support chats, history is summarized every 10 turns. The agent later says "your refund is being processed" but can't state the amount, and once quoted the wrong date. What is the best fix?

- A. Summarize less often.
- B. Keep a persistent "case facts" block (amounts, dates, order IDs, promises) in every prompt, outside the summarized history.
- C. Ask the customer to repeat details.
- D. Increase `max_tokens`.

<details><summary>Answer</summary>

**B.** Progressive summarization blurs exact facts; a persistent structured block keeps them precise. → [01](01-conversation-context-preservation.md)
</details>

### A2
Each `lookup_order` call returns 45 fields, but returns processing needs 5. After several lookups, context is dominated by order data. What should you do?

- A. Use a bigger context window.
- B. Trim tool outputs to the relevant fields before they enter the conversation.
- C. Call `lookup_order` less often.
- D. Summarize the order data after every turn.

<details><summary>Answer</summary>

**B.** Trim verbose tool outputs early; they're re-sent on every later turn. → [01](01-conversation-context-preservation.md)
</details>

### A3
A customer writes: "I don't want a bot. Put me through to a person." What should the agent do?

- A. Look up their account first so the human has context, then escalate.
- B. Explain that the agent can probably help and try to resolve the issue.
- C. Escalate immediately.
- D. Ask why they want a human.

<details><summary>Answer</summary>

**C.** Honor explicit requests for a human immediately, without investigating first. → [02](02-escalation-ambiguity-resolution.md)
</details>

### A4
A customer writes angrily: "My headphones arrived broken, again! Photo attached." Policy covers free replacement for damaged items with photos. What is best?

- A. Escalate because the customer is angry.
- B. Acknowledge the frustration and process the replacement; escalate only if the customer asks for a human.
- C. Escalate because the model's confidence is below 0.8.
- D. Ask the customer to calm down.

<details><summary>Answer</summary>

**B.** Sentiment is not a reliable escalation trigger; this case is within the agent's capability. → [02](02-escalation-ambiguity-resolution.md)
</details>

### A5
A customer asks the agent to match a competitor's lower price. Policy only covers price adjustments when your own site's price drops. What should the agent do?

- A. Give the credit, since it's close to the existing policy.
- B. Refuse, saying it's not allowed.
- C. Escalate, because policy is silent on competitor matching (a policy gap).
- D. Offer half the difference as a compromise.

<details><summary>Answer</summary>

**C.** Policy gaps go to humans; the agent shouldn't invent policy. → [02](02-escalation-ambiguity-resolution.md)
</details>

### A6
`get_customer(name="Maria Garcia")` returns three accounts. What should the agent do?

- A. Use the most recently active account.
- B. Use the account with the most orders.
- C. Ask the customer for an additional identifier (e.g. email or phone digits).
- D. Show all three accounts to the customer.

<details><summary>Answer</summary>

**C.** Multiple matches require clarification, not heuristic selection. D would expose other customers' data. → [02](02-escalation-ambiguity-resolution.md)
</details>

### A7
The agent escalates many simple, policy-covered cases but tries to handle policy exceptions itself. What is the most effective fix?

- A. Escalate whenever confidence is low.
- B. Add explicit escalation criteria with few-shot examples of resolve vs escalate to the system prompt.
- C. Escalate whenever the customer uses negative words.
- D. Remove the escalation tool.

<details><summary>Answer</summary>

**B.** Explicit criteria + examples calibrate escalation behavior. → [02](02-escalation-ambiguity-resolution.md)
</details>

---

## Scenario B: Multi-Agent Research System

### B1
A web-search subagent times out and returns `{"findings": []}` with status "success". The final report omits that topic without comment. What is the problem?

- A. The coordinator's prompt is too short.
- B. Silent error suppression: an access failure was reported as a valid empty result.
- C. The synthesis agent dropped findings.
- D. The topic doesn't exist.

<details><summary>Answer</summary>

**B.** → [03](03-error-propagation-multi-agent.md)
</details>

### B2
After one of six subagents fails, an engineer changes the system to abort the entire job on any error. What is wrong with this?

- A. Nothing; it's the safest option.
- B. It throws away successful partial work; the better design continues with partial results and annotates the gap.
- C. It should retry the whole job instead.
- D. It should hide the error.

<details><summary>Answer</summary>

**B.** Terminating whole workflows on a single failure is an anti-pattern. → [03](03-error-propagation-multi-agent.md)
</details>

### B3
A subagent can't complete its search after local retries. What should it send to the coordinator?

- A. `"search unavailable"`
- B. Failure type, what was attempted, partial results, and potential alternatives.
- C. Nothing; the coordinator will notice.
- D. The full raw logs.

<details><summary>Answer</summary>

**B.** Structured error context enables intelligent coordinator recovery. → [03](03-error-propagation-multi-agent.md)
</details>

### B4
Some sources were unavailable during research. How should the final report reflect this?

- A. Leave it out to keep the report clean.
- B. Add coverage annotations showing which findings are well supported and which areas have gaps due to unavailable sources.
- C. Fill the gaps with general knowledge.
- D. Delay the report until all sources are available.

<details><summary>Answer</summary>

**B.** → [03](03-error-propagation-multi-agent.md)
</details>

### B5
The synthesis agent receives long outputs from eight subagents and often misses findings from the middle sections. What helps most?

- A. Put a key-findings summary at the beginning and organize details under explicit section headers.
- B. Shuffle the order of sections.
- C. Remove the section headers.
- D. Add more subagents.

<details><summary>Answer</summary>

**A.** Mitigates the "lost in the middle" effect. → [01](01-conversation-context-preservation.md)
</details>

### B6
Subagents return pages of reasoning, and the synthesis agent's context fills up. What should change upstream?

- A. Ask subagents to "be brief".
- B. Have subagents return structured data (key facts, citations, relevance scores, dates, methodology) instead of verbose reasoning.
- C. Remove citations to save space.
- D. Give the synthesis agent fewer subagents' results.

<details><summary>Answer</summary>

**B.** → [01](01-conversation-context-preservation.md)
</details>

### B7
Two credible sources report different job-impact statistics. The synthesis picked one and presented it as fact. What is the correct handling?

- A. Use the larger number.
- B. Use the more recent source only.
- C. Include both with attribution and annotate the conflict (and methodology differences).
- D. Leave the topic out.

<details><summary>Answer</summary>

**C.** Annotate conflicts with attribution; don't select arbitrarily. → [06](06-provenance-and-uncertainty.md)
</details>

### B8
A report labels "20% adoption" and "55% adoption" as conflicting estimates. One is from 2022 data, the other from 2025. What should have prevented this?

- A. Removing older data.
- B. Requiring publication/data collection dates in subagents' structured outputs so time differences are interpreted correctly.
- C. Averaging the two numbers.
- D. Using only one source per topic.

<details><summary>Answer</summary>

**B.** Temporal metadata prevents false contradictions and reveals trends. → [06](06-provenance-and-uncertainty.md)
</details>

### B9
Final reports contain claims with no sources, even though subagents found good sources. Where is attribution most likely lost?

- A. In the web search tool.
- B. During summarization steps that compress findings without claim-source mappings.
- C. In the user's question.
- D. In the coordinator's tool list.

<details><summary>Answer</summary>

**B.** → [06](06-provenance-and-uncertainty.md)
</details>

### B10
A document-analysis subagent finds two different values for the same statistic in two documents. What should it do?

- A. Pick the value from the more famous source.
- B. Average them.
- C. Complete the analysis with both values included and the conflict flagged, so the coordinator can decide how to reconcile.
- D. Drop the statistic.

<details><summary>Answer</summary>

**C.** → [06](06-provenance-and-uncertainty.md)
</details>

### B11
A report contains revenue figures, recent news events, and technical specifications. How should they be presented?

- A. Everything as bullet points.
- B. Everything as tables.
- C. Revenue as tables, news as prose, technical findings as structured lists.
- D. Everything as prose.

<details><summary>Answer</summary>

**C.** Render each content type appropriately. → [06](06-provenance-and-uncertainty.md)
</details>

---

## Scenario C: Code Generation with Claude Code / Large Codebase Exploration

### C1
Several hours into exploring a large codebase, Claude starts describing "a typical service layer" instead of the specific `LegacyBillingFacade` it found earlier, and gives inconsistent answers. What is happening and what helps?

- A. The model is broken; restart without notes.
- B. Context degradation; use a scratchpad file of key findings, delegate specific questions to subagents, and re-read the scratchpad.
- C. Use plan mode.
- D. Increase `max_tokens`.

<details><summary>Answer</summary>

**B.** → [04](04-large-codebase-context.md)
</details>

### C2
A multi-agent codebase-mapping run lost all progress when the machine restarted. What design prevents this?

- A. Run everything in one agent.
- B. Agents export structured state to a known location; the coordinator maintains a manifest and loads it on resume.
- C. Avoid long runs.
- D. Keep everything in the conversation history.

<details><summary>Answer</summary>

**B.** → [04](04-large-codebase-context.md)
</details>

### C3
An interactive exploration session is filling up with verbose discovery output. You want to continue in the same session. What should you do?

- A. Use `/compact` (after saving key facts to a scratchpad) to reduce context usage.
- B. Keep going until it fails.
- C. Paste the output into a new chat manually.
- D. Disable tools.

<details><summary>Answer</summary>

**A.** → [04](04-large-codebase-context.md)
</details>

### C4
You're moving from a "map the codebase" phase to a "trace the refund flow" phase, using new subagents. How should the new subagents get the earlier results?

- A. They automatically inherit the earlier context.
- B. Summarize the key findings from phase 1 and inject the summary into the phase 2 subagents' initial context.
- C. Give them the full raw transcript of phase 1.
- D. They don't need earlier findings.

<details><summary>Answer</summary>

**B.** → [04](04-large-codebase-context.md)
</details>

---

## Scenario D: Structured Data Extraction

### D1
An extraction model shows 97% overall accuracy. The team wants to stop human review entirely. What should happen first?

- A. Stop review; 97% is excellent.
- B. Analyze accuracy by document type and field to check for weak segments.
- C. Increase the confidence threshold for everything.
- D. Switch models.

<details><summary>Answer</summary>

**B.** Aggregate accuracy can hide poor performance in specific segments. → [05](05-human-review-confidence-calibration.md)
</details>

### D2
High-confidence extractions are auto-accepted. How do you keep measuring their error rate and catch new error patterns?

- A. Trust the confidence scores.
- B. Stratified random sampling of high-confidence extractions across segments, reviewed by humans.
- C. Review only low-confidence items.
- D. Check a few documents when someone complains.

<details><summary>Answer</summary>

**B.** → [05](05-human-review-confidence-calibration.md)
</details>

### D3
You want to route only risky fields to reviewers. What approach is best?

- A. One document-level confidence score with a fixed 0.9 threshold.
- B. Field-level confidence scores with thresholds calibrated on a labeled validation set.
- C. Review every field of every document.
- D. Random routing.

<details><summary>Answer</summary>

**B.** → [05](05-human-review-confidence-calibration.md)
</details>

---

## Score yourself

| Score | Next step |
|---|---|
| 22–25 | Ready for Domain 5. Revisit any you guessed. |
| 17–21 | Re-read the files linked from your wrong answers. |
| < 17 | Re-read files 00–06 in order, then try again in a few days. |

---

## Key Takeaways

- **Preserve facts**: case facts block, trimmed tool outputs, key findings first, structured upstream outputs.
- **Escalate on criteria**: human requested (immediately), policy gap, no progress; **ask** when ambiguous.
- **Propagate errors with context**; avoid silent suppression and abort-everything; annotate coverage.
- **Long explorations**: subagents, scratchpads, phase summaries, manifests, `/compact`.
- **Human review**: segment-level accuracy, calibrated field confidence, stratified sampling, prioritized routing.
- **Provenance**: claim-source mappings, annotated conflicts, dates, content-appropriate rendering.
