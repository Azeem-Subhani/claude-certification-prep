# 05 · Efficient Batch Processing Strategies

**Exam link:** Task Statement 4.5: *Design efficient batch processing strategies.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (SLA, synchronous API).

---

## 1. Two ways to call Claude

| | **Synchronous Messages API** | **Message Batches API** |
|---|---|---|
| How | Send a request → get the response right away | Submit many requests at once → collect results later |
| Latency | Seconds | Most batches finish within 1 hour, but it can take **up to 24 hours**, with **no guaranteed latency** |
| Cost | Standard price | **50% cheaper** |
| Good for | Interactive and **blocking** workflows | **Non-blocking, latency-tolerant** bulk work |

---

## 2. Key facts about the Message Batches API

- **50% cost savings** compared with standard API prices.
- Processing can take **up to 24 hours**. Requests not processed within 24 hours **expire**.
- **No latency guarantee**: you can't promise "results in 10 minutes".
- Each request has a **`custom_id`** that you choose. Results may come back **in any order**, so always match them by `custom_id`, never by position.
- Each result has a type: `succeeded`, `errored`, `canceled`, or `expired`.
- Limits (current docs): up to 100,000 requests or 256 MB per batch; results available for 29 days.

### No multi-turn tool calling inside a request

A batch request is processed **once** and returns **one response**. If Claude decides to call **your** tool (a client tool), the batch can't run that tool and continue the conversation within the same request. You'd get a response with `stop_reason: "tool_use"` and would need **another request** (e.g. another batch) for the next turn.

So batches suit **single-shot** tasks: "extract these fields", "summarize this file", "generate tests for this module". They don't suit agentic loops that need several tool round-trips.

---

## 3. Choosing the right API

**Rule:** If someone or something is **waiting** for the result, use the **synchronous** API. If the result is needed **later**, use the **batch** API.

| Workflow | Blocking? | API |
|---|---|---|
| **Pre-merge check** that must pass before a developer can merge | ✅ Yes, a developer waits | **Synchronous** |
| Real-time customer support reply | ✅ Yes | **Synchronous** |
| **Overnight** technical-debt report | ❌ No | **Batch** |
| **Weekly** compliance audit of all repos | ❌ No | **Batch** |
| **Nightly** test generation for changed modules | ❌ No | **Batch** |
| Backfilling extraction for 2 million archived documents | ❌ No | **Batch** |

> **Exam pattern:** "Reduce API costs for (1) a blocking pre-merge check and (2) an overnight report." → **Batch only the overnight report.** Moving the pre-merge check to batch would block developers for up to 24 hours.

---

## 4. Calculating submission frequency from an SLA

If you promise results within an **SLA** (e.g. 30 hours), and batch processing can take **up to 24 hours**, you must **submit often enough** that the waiting time before submission plus processing time stays within the SLA.

**Formula:**

```
max wait before submission + max batch processing time ≤ SLA
```

**Example:** SLA = **30 hours**, batch processing up to **24 hours**.

```
max wait before submission ≤ 30 − 24 = 6 hours
```

So submit at least **every 6 hours**. To leave a safety margin (time to collect results, retry failures, post-process), submit every **4 hours**:

```
worst case = 4 h (document waits for next submission) + 24 h (processing) = 28 h  ≤  30 h ✅
```

| Submission interval | Worst case | Meets 30 h SLA? |
|---|---|---|
| Every 4 h | 28 h | ✅ with 2 h margin |
| Every 6 h | 30 h | ⚠️ no margin |
| Every 12 h | 36 h | ❌ |

---

## 5. Handling failures: resubmit only what failed

In a batch of 10,000 documents, some will fail. **Don't resubmit the whole batch.**

1. Read results and group by type.
2. Use **`custom_id`** to identify exactly which requests failed or expired.
3. Look at **why** they failed and **fix** before resubmitting.
4. Resubmit **only those** in a new batch.

| Failure reason | Fix before resubmitting |
|---|---|
| Document **exceeded the context limit** | **Chunk** the document into parts (e.g. by section) and submit each part with its own `custom_id` (`doc-123-part-1`, …) |
| Invalid request (bad parameters) | Fix the request |
| Expired (24 h passed) | Resubmit as is (possibly at a quieter time) |
| Validation failed after success | Retry with error feedback ([04](04-validation-retry-feedback-loops.md)) |

```python
failed_ids = []
for result in client.messages.batches.results(batch_id):
    if result.result.type == "succeeded":
        save(result.custom_id, result.result.message)
    else:
        failed_ids.append(result.custom_id)       # errored / expired / canceled

retry_requests = [build_request(doc_id, chunk_if_too_long=True) for doc_id in failed_ids]
if retry_requests:
    client.messages.batches.create(requests=retry_requests)
```

---

## 6. Refine the prompt on a sample first

Batch jobs are cheap per request but **large**. A prompt bug can waste 100,000 requests, and then you pay again to resubmit.

**Best practice:**

1. Take a **representative sample** (e.g. 50–200 documents, including tricky ones).
2. Run it (synchronously or in a small batch) and **measure**: validation pass rate, accuracy, failure types.
3. **Refine the prompt** (criteria, few-shot examples, schema) until the sample results are good.
4. **Then** run the full batch.

This **maximizes first-pass success** and **reduces costly resubmissions**. (Anthropic's docs also recommend testing batch requests with the regular Messages API first.)

---

## 7. Real-world case study

**Situation:** A legal-tech company runs two Claude workflows:
- A **pre-merge** contract-template check in CI (developers wait for it).
- A **weekly** analysis of 80,000 archived contracts.

**First attempt:** To cut costs, they moved **both** to the Batches API. Developers were blocked for hours waiting for merge checks.

**Fix:**
1. Pre-merge check back on the **synchronous** API.
2. Weekly analysis stays on **batch** (50% savings).
3. Contract analysis has a **30-hour SLA**, so batches are submitted every **4 hours** as contracts arrive.
4. About 1% of contracts failed for exceeding context limits. Now only those `custom_id`s are **chunked and resubmitted**.
5. Before each prompt change, they test on a **200-contract sample**; first-pass success rose and resubmission costs fell.

---

## Common Q&A

**Q: What are the main trade-offs of the Message Batches API?**
A: **50% cheaper**, but processing can take **up to 24 hours** with **no latency guarantee**.

**Q: Can I use batch for a check developers wait on?**
A: No. Blocking workflows need the synchronous API.

**Q: How do I match batch results to my inputs?**
A: By **`custom_id`**. Results can come back in any order.

**Q: Can a batch request run an agent loop with my tools?**
A: Not within a single request. It can't execute your tools mid-request and continue. Each turn would need another request.

**Q: My SLA is 30 hours and batches can take 24. How often should I submit?**
A: At least every 6 hours; every **4 hours** gives a safety margin (worst case 28 h).

**Q: 300 of 10,000 requests failed. What do I resubmit?**
A: Only those 300, identified by `custom_id`, after fixing the cause (e.g. chunking oversized documents).

**Q: Why test on a sample before a large batch?**
A: To fix prompt problems cheaply and maximize first-pass success, avoiding expensive full resubmissions.

---

## Key Takeaways

- **Batch API = 50% cheaper, up to 24 h, no latency SLA.**
- Use batch for **non-blocking** work (overnight, weekly, nightly); use **synchronous** for **blocking** work (pre-merge checks).
- **No multi-turn tool calling** inside a single batch request.
- Match results by **`custom_id`**; results aren't in order.
- **Submission interval ≤ SLA − 24 h**, with a margin (30 h SLA → every 4 h).
- **Resubmit only failures**, fixed first (e.g. chunk oversized documents).
- **Refine prompts on a sample** before processing large volumes.
