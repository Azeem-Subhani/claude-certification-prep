# Domain 5 Anti-patterns: Context Management & Reliability

**Scope:** wrong choices for long conversations, escalation, error propagation, long codebase sessions, human review, and source tracking (task statements 5.1 to 5.6).

---

## Anti-patterns

| # | Wrong approach | Why it fails | Do this instead | Topic |
|---|---|---|---|---|
| 1 | Summarize old turns that hold amounts, dates, and order IDs. | Summaries turn exact values into vague phrases like "a refund was discussed". | Keep exact facts in a case facts block that is sent with every prompt and never summarized. | [01](01-conversation-context-preservation.md) |
| 2 | Keep full tool results "in case they are needed later". | A 40-field result fills the context when you need 5 fields. | Trim results to the relevant fields before they enter the history. | [01](01-conversation-context-preservation.md) |
| 3 | Tell Claude to "pay attention to the middle sections". | "Lost in the middle" is a structure problem. Instructions do not change how position affects reading. | Put a key findings summary at the top and use clear section headers. | [01](01-conversation-context-preservation.md) |
| 4 | Let upstream agents send long reasoning chains. | Downstream agents run out of context for the real work. | Ask upstream agents for structured data: key facts, citations, relevance, and dates. | [01](01-conversation-context-preservation.md) |
| 5 | Cut random parts of the history to save tokens. | The API is stateless. Claude loses track and repeats or contradicts itself. | Control what you send: facts block, trimmed results, and summaries of non-critical turns. | [01](01-conversation-context-preservation.md) |
| 6 | Investigate first when the customer asks for a human. | The customer's explicit request is ignored. | Escalate at once, with a structured handoff. | [02](02-escalation-ambiguity-resolution.md) |
| 7 | Escalate because the customer sounds angry. | Mood does not measure how complex the case is. Simple cases get escalated. | Show understanding and offer the fix. Escalate if the customer asks for a human again. | [02](02-escalation-ambiguity-resolution.md) |
| 8 | Escalate based on Claude's self-reported confidence. | Uncalibrated confidence is not a reliable signal. | Use explicit escalation criteria, with few-shot examples in the system prompt. | [02](02-escalation-ambiguity-resolution.md) |
| 9 | Make up a rule when the policy says nothing about the case. | The agent invents policy that the company never approved. | Treat a policy gap as an escalation trigger. | [02](02-escalation-ambiguity-resolution.md) |
| 10 | Pick the most recent or most active record when several customers match. | It may be the wrong person, which can expose someone else's data. | Ask for another identifier: email, phone, or order number. | [02](02-escalation-ambiguity-resolution.md) |
| 11 | Return empty results marked as success after a timeout. | The coordinator believes there is no data. This is the worst error pattern. | Return structured error context: failure type, what was tried, partial results, and options. | [03](03-error-propagation-multi-agent.md) |
| 12 | Stop the whole pipeline because one subagent failed. | Good results from the other subagents are lost. | Continue with partial results and add coverage notes. | [03](03-error-propagation-multi-agent.md) |
| 13 | Send a generic "search unavailable" message after retries. | The coordinator cannot decide what to do next. | Report what failed, what was tried, and what partial results exist. | [03](03-error-propagation-multi-agent.md) |
| 14 | Retry a query that ran and returned nothing. | An empty result is a valid answer. A retry gives the same result. | Accept it. Retry only access failures such as timeouts. | [03](03-error-propagation-multi-agent.md) |
| 15 | Use a bigger context window to fix vague answers late in a session. | Degradation is about context quality, not size. | Use scratchpad files, subagents, phase summaries, and `/compact`. | [04](04-large-codebase-context.md) |
| 16 | Think subagents are only for running work in parallel. | You miss their main benefit for exploration. | Use subagents to keep verbose discovery out of the main context. | [04](04-large-codebase-context.md) |
| 17 | Restart a long session without saving notes first. | Confirmed findings are lost with the old context. | Write findings to a scratchpad or state manifest, then restart. | [04](04-large-codebase-context.md) |
| 18 | Run a long, crash-prone job with no saved state. | A crash loses all progress. | Each agent exports state to a manifest that the coordinator loads on resume. | [04](04-large-codebase-context.md) |
| 19 | Use overall accuracy (for example, 97%) to justify automation. | A high average can hide weak document types or fields. | Measure accuracy by document type and by field first. | [05](05-human-review-confidence-calibration.md) |
| 20 | Route by raw confidence scores. | You do not know what a score really means until you check it. | Calibrate field-level scores on a labeled validation set, then set thresholds. | [05](05-human-review-confidence-calibration.md) |
| 21 | Sample only low-confidence items. | You never measure errors in the items you automate. | Use stratified random sampling that includes high-confidence items. | [05](05-human-review-confidence-calibration.md) |
| 22 | Spread reviewer time evenly across all items. | Limited reviewer time goes to easy cases. | Send the most uncertain items first. | [05](05-human-review-confidence-calibration.md) |
| 23 | Let the synthesis agent paraphrase without source links. | Attribution is lost at the synthesis step. | Require every claim to keep its claim-source mapping. | [06](06-provenance-and-uncertainty.md) |
| 24 | Pick the newer source, average the values, or trust the bigger publisher. | The real conflict is hidden, and the chosen value may be wrong. | Show both values with their sources and dates. | [06](06-provenance-and-uncertainty.md) |
| 25 | Treat different numbers as a contradiction without checking dates. | The numbers may show a trend over time. | Check publication or collection dates first. | [06](06-provenance-and-uncertainty.md) |
| 26 | Put all content in one format. | Financial data, news, and technical findings read best in different forms. | Use tables for financial data, prose for news, and structured lists for technical findings. | [06](06-provenance-and-uncertainty.md) |

---

## Common exam traps

1. **"Summarize the old conversation to save tokens."** Summaries lose exact numbers and dates. Keep them in a case facts block.
2. **"Investigate first, then escalate if still needed."** An explicit request for a human must be honored at once.
3. **"Escalate because the customer is frustrated."** Sentiment is not complexity. Offer the fix first.
4. **"Escalate when the model's confidence drops below a threshold."** Self-reported confidence is not a reliable trigger.
5. **"Choose the most recently active matching account."** Guessing can expose another person's data. Ask for another identifier.
6. **"Return an empty result so the report still completes."** Silent empty success is the worst error pattern.
7. **"Stop the job because one source failed."** Continue, and add coverage notes for the gap.
8. **"Answers are vague late in the session, so the token limit was hit."** This is context degradation, not a token-limit problem.
9. **"97% overall accuracy, so automate everything."** High overall accuracy can hide bad segments.
10. **"Sample only the items with low confidence."** You must also sample high-confidence items to measure real error rates.
11. **"Use the newer figure and drop the older one."** Never pick one of two conflicting values. Show both, with dates.
12. **"Tell the model to read the middle sections carefully."** Fix the structure instead: key findings first, clear headers.

> ⚠️ See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R2 (when to run `/compact`).
