# Domain 5: Context Management & Reliability

> **Exam weight:** 15% of scored items (official exam guide v1.0, §4).
> **Task statements:** 5.1 to 5.6 (see [docs/domain-map.md](../docs/domain-map.md)).
> **Deep dives:** [domain-5-context-management-reliability/](../domain-5-context-management-reliability/README.md)
> **Facts checked against:** [docs/verification-log.md](../docs/verification-log.md). ⚠️ marks a review item.

---

## Exam focus

- **Keeping exact facts (5.1).** Summaries lose numbers and dates. Keep them in a "case facts" block. Trim long tool outputs. Put key findings at the top.
- **Escalation (5.2).** Know the real triggers and the false ones. When a search returns several possible matches, ask for more details.
- **Error propagation (5.3).** Send structured error context to the coordinator. Never hide a failure as an empty success.
- **Long codebase sessions (5.4).** Recognize context degradation. Fix it with scratchpads, subagents, phase summaries, and manifests.
- **Human review (5.5) and provenance (5.6).** Measure accuracy by segment and calibrate confidence scores. Keep a source attached to every claim.

---

## Key concepts

### 1. Words you need first

The **context window** is everything Claude can see in one request: the system prompt, the history, the tool results, and the documents. It has a size limit, and every token in it costs money and attention.

**Reliability** here means the system gives correct, honest results even when parts fail or the session gets long.

### 2. Keeping critical information (Task 5.1)

**The API is stateless.** Each request must include the full conversation history. So everything you keep in the history is sent, paid for, and read again on every turn.

**Progressive summarization loses detail.** When you shorten an old conversation into a summary, exact values become vague. "$1,284.50 refund for order #A-7731 on 3 March" becomes "a refund was discussed".

**The fix: a case facts block.** Keep a structured block of exact facts and add it to every prompt. Do not summarize this block. If there are several issues, add one entry per issue.

```text
CASE FACTS (do not summarize)
- Customer ID: C-20931
- Issue 1: order #A-7731, damaged item, refund requested $1,284.50, status: pending photo
- Issue 2: order #A-7802, delivered late (promised 12 Mar, arrived 19 Mar), status: open
```

**Trim tool outputs early.** A lookup may return 40 or more fields when you need 5. Trim it in the tool, or in a `PostToolUse` hook, **before** it enters the history.

**"Lost in the middle."** Models read the start and end of long inputs well. They may miss details in the middle. Fix the **structure**:
- Put a short key-findings summary at the top.
- Use clear section headers.

Telling Claude to "pay attention" does not fix it.

**Structured upstream output.** If earlier agents send long reasoning, later agents run out of room. Ask earlier agents to return structured data instead: key facts, citations, relevance scores, and dates.

**Prompt caching** exists for long prompt prefixes that you send again and again. It lowers cost and response time for the repeated part. The exam tests only that it exists. Implementation details are out of scope (exam guide §17; verification log A9).

### 3. Escalation and ambiguity (Task 5.2)

**Escalate when:**
1. The customer **explicitly asks** for a human. Do it at once. Do not investigate first.
2. There is a **policy gap**: the policy says nothing about this case. Example: the policy covers price matching on your own site, and the customer asks you to match a competitor's price.
3. The agent has **really tried** and cannot make meaningful progress.

**Do not escalate when:**
- The policy clearly says no (a **policy violation**). Explain the policy instead.
- The customer is frustrated, but the issue is simple. Show understanding and offer the fix. If the customer **asks again** for a human, escalate.

**Bad triggers:**
- **Sentiment** (how upset the customer sounds) does not measure how complex the case is.
- **Claude's self-reported confidence** is not reliable without calibration.

**Fix a miscalibrated agent** with explicit escalation criteria plus few-shot examples in the system prompt. Show "escalate" cases and "resolve yourself" cases.

**Ambiguity.** A search may return several customers with the same name. Then ask for another identifier: email, phone, or order number. Never pick the most recent or most active record.

### 4. Error propagation in multi-agent systems (Task 5.3)

When a subagent fails, the coordinator needs **structured error context**:

```json
{
  "status": "failed",
  "failure_type": "timeout",
  "attempted": "search 'EU battery recycling rules 2025' (3 tries, backoff 2s/4s/8s)",
  "partial_results": [{"claim": "...", "source_url": "..."}],
  "alternatives": ["try the regulations database", "narrow to 2025 only"]
}
```

With this, the coordinator can choose to:
- retry
- try another source
- continue with partial results
- escalate

**Two opposite mistakes:**
1. **Silent suppression.** The subagent returns empty results marked as success after a timeout. The coordinator then believes there is no data. This is the **worst** pattern.
2. **Total abort.** The whole pipeline stops because one subagent failed. The other good results are lost.

**Access failure vs valid empty result:**
- A timeout means "we do not know". A retry may help.
- A query that ran and found nothing is a real answer. Do not retry it.

**Local recovery first.** The subagent retries transient errors itself, with backoff. Only errors it cannot fix go up to the coordinator, with what it tried and any partial results.

**Coverage notes.** The final report should say which areas are well supported and which have gaps because a source was unavailable.

### 5. Context in large codebase exploration (Task 5.4)

**Context degradation:** in a long session, Claude starts giving vague answers like "this follows typical patterns". It stops naming the exact classes it found earlier. This is a context-quality problem, not just a token limit.

**Fixes:**
- **Scratchpad files.** From the start, write key findings to a file, for example `notes/refund-flow.md`. Read it back when needed.
- **Subagents for exploration.** A subagent investigates one question, such as "find all test files", and returns a summary. The main benefit is a **clean main context**, not speed.
- **Phase summaries.** At the end of phase 1, write a summary. Put it into the prompts of the phase-2 subagents.
- **State manifests for crash recovery.** Each agent saves its state to a known file: explored paths, findings, current phase, next steps. On restart, the coordinator loads it.
- **`/compact`** in Claude Code summarizes the conversation to free context during long sessions.

> ⚠️ **Review item:** the exam guide mentions `/compact` "when context fills". The cheat sheet says to compact early and often. See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R2.

A bigger context window does not fix degradation. Structure and persistence do.

### 6. Human review and confidence calibration (Task 5.5)

**Averages hide weak spots.** 97% overall accuracy can hide 70% accuracy on handwritten forms, or on one field like "expiry date". Before you automate, check accuracy **by document type and by field**.

**Field-level confidence, calibrated.** Have the model give a confidence score for each field. Then **calibrate** the scores: compare them with a **labeled validation set** (examples with known correct answers). This tells you what "0.9" really means. Set thresholds from that data.

**Routing:**
- Above the calibrated threshold: automate, but keep sampling.
- Below it: send to human review.
- Unclear or conflicting sources: send to **priority** human review.
- With limited reviewers, review the most uncertain items first. Do not spread their time evenly.

**Stratified random sampling.** Keep checking a random sample from **each** segment, **including** high-confidence items. This measures the real error rate and finds new error patterns. Sampling only low-confidence items misses errors that the model is confident about.

**Safe order for reducing human review:**
1. Measure by segment.
2. Calibrate.
3. Set thresholds.
4. Sample.
5. Reduce review.

### 7. Provenance and multi-source synthesis (Task 5.6)

**Provenance** is where a fact came from. Attribution is most often lost at the **synthesis step**, when findings are compressed.

**Keep claim-source mappings.** Each finding should carry:
- the claim
- the source URL
- the document name
- a short excerpt
- the publication date

The synthesis prompt must require every claim to keep its source.

**Conflicting numbers.** Two credible sources give different values. Do not pick one, average them, or trust the bigger publisher. Show **both** values with their sources and dates.

**Check the dates first.** "12% in 2022" and "19% in 2025" may be a trend, not a contradiction.

**Who decides?** An analysis subagent that finds a conflict should finish its work and **mark** the conflict. The coordinator decides how to reconcile it.

**Structure the report by certainty:**
- well-established findings
- contested findings
- gaps

**Match the format to the content:**
- financial data: a table
- news: prose
- technical findings: a structured list

---

## Common traps

| Tempting wrong answer | Why it is wrong |
|---|---|
| "Summarize the old conversation to save tokens." | Exact amounts, dates, and IDs get lost. Keep a case facts block. |
| "Keep the full tool results in case they are needed." | They crowd the context. Trim to the relevant fields first. |
| "Tell Claude to read the middle sections carefully." | Position effects are structural. Put key findings first and use headers. |
| "Investigate first, then escalate if still needed." | An explicit request for a human must be honored at once. |
| "Escalate because the customer sounds angry." | Sentiment is not complexity. Offer the fix. Escalate if they ask again. |
| "Escalate when the model's confidence is below 0.7." | Self-reported confidence is not calibrated. Use explicit criteria. |
| "Pick the most recently active matching account." | It may be the wrong person. Ask for another identifier. |
| "Return an empty result so one failure doesn't break the report." | A failure hidden as success misleads the coordinator. |
| "Stop the whole job when one source fails." | Good partial results are lost. Continue and add coverage notes. |
| "Use a model with a larger context window for long sessions." | Degradation is about context quality. Use scratchpads and subagents. |
| "Subagents are only useful to run work in parallel." | Their main benefit here is keeping the main context clean. |
| "97% accuracy overall, so automate everything." | Weak segments may be hidden. Check by document type and field. |
| "Sample only low-confidence items." | You never measure errors in the items you automate. |
| "Use the newer source and drop the older one." | Both may be right for their dates. Show both, with sources. |

---

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

---

## Quick recap

- Keep exact facts in a case facts block. Trim tool output before it enters history. Put key findings first.
- Escalate on an explicit request (at once), a policy gap, or no progress. Do not escalate on sentiment or self-reported confidence.
- When there are several possible matches, ask for another identifier.
- Send structured error context. Never hide failures as empty success, and never abort everything for one failure.
- Fight context degradation with scratchpads, subagents, phase summaries, manifests, and `/compact`.
- Check accuracy by segment. Calibrate field-level confidence on labeled data. Sample every segment, including high-confidence items.
- Keep claim-source links through synthesis. Show conflicting values with dates. Choose the format by content type.

---

## Practice questions

Go to the Domain 5 question bank: **[mcq-bank/05-context-management-reliability.md](../mcq-bank/05-context-management-reliability.md)** (36 questions).
Older, easier warm-up questions: [domain-5-.../07-scenario-practice-questions.md](../domain-5-context-management-reliability/07-scenario-practice-questions.md).
