# 01 · Preserving Critical Information Across Long Interactions

**Exam link:** Task Statement 5.1: *Manage conversation context to preserve critical information across long interactions.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (context as a budget).

---

## 1. Progressive summarization loses the details that matter

In long conversations, a common way to save space is to **summarize older turns**. If you summarize repeatedly (**progressive summarization**), details fade with each round:

| Original | After summary 1 | After summary 2 |
|---|---|---|
| "Refund of **$1,247.83** for order **#A-55120**, charged twice on **March 3**; customer expects it **by Friday**." | "Customer was double-charged on order A-55120 and wants a refund soon." | "Customer has a billing issue." |

What disappears first are exactly the things a support agent needs:

- **numbers and amounts** ($1,247.83 → "a refund"),
- **percentages** ("a 15% discount" → "a discount"),
- **dates and deadlines** ("by Friday" → "soon"),
- **customer expectations and promises** ("we promised a callback at 3 pm" → gone).

---

## 2. The fix: a persistent "case facts" block

Extract **transactional facts** into a **structured block** that is **included in every prompt** and **never summarized**:

```text
<case_facts>
customer_id: C-20931 (verified)
order_id: A-55120
issue: duplicate charge
charged_amount: 1247.83 USD (charged twice on 2026-03-03)
refund_status: approved, pending processing
promised_to_customer: refund confirmation email by 2026-03-07 (Friday)
</case_facts>

<conversation_summary>
Customer contacted support about a duplicate charge; identity verified; refund approved...
</conversation_summary>

<recent_messages>
... last few turns in full ...
</recent_messages>
```

- The **summary** can be compressed further over time.
- The **case facts** stay **exact**, because the harness (your code) updates them from **tool results**, not from model prose.

### Multi-issue sessions: separate structured layer

When one conversation covers **several issues**, keep a **structured issue list** in its own layer:

```json
{
  "issues": [
    {"id": 1, "type": "duplicate_charge", "order_id": "A-55120", "amount": 1247.83, "status": "refund_approved"},
    {"id": 2, "type": "damaged_item", "order_id": "A-55188", "item": "desk lamp", "status": "replacement_shipped", "tracking": "1Z999..."},
    {"id": 3, "type": "address_change", "status": "open", "new_address_verified": false}
  ]
}
```

Each issue's IDs, amounts, and status stay precise, even as the free-text conversation gets summarized.

---

## 3. Trim verbose tool outputs before they pile up

Tool results are added to the history and **re-sent on every later request**. Large results use **tokens out of proportion to their value**.

**Example:** `lookup_order` returns **40+ fields** (warehouse codes, internal flags, audit timestamps…). For a return request, **5** matter:

```python
RETURN_RELEVANT_FIELDS = ["order_id", "items", "delivered_at", "return_window_ends", "payment_method"]

def trim_for_returns(order: dict) -> dict:
    """Keep only the fields the return workflow needs before adding to context."""
    return {k: order[k] for k in RETURN_RELEVANT_FIELDS if k in order}
```

Do this **before** the result enters the conversation (in your tool wrapper, or with a `PostToolUse` hook, see Domain 1 · [05](../domain-1-agentic-architecture-orchestration/05-agent-sdk-hooks.md)). After 10 lookups, the difference is 50 fields of context instead of 400+.

---

## 4. Position effects: "lost in the middle"

With long inputs, models reliably use information at the **beginning** and **end**, but may **miss findings in the middle**.

**Mitigation for aggregated inputs** (e.g. a synthesis agent receiving results from 8 subagents):

1. Put a **key findings summary at the beginning**.
2. Organize the details under **explicit section headers**.
3. Optionally repeat the critical instruction or question at the **end**.

```text
## KEY FINDINGS (read first)
1. Adoption of AI tools in music production rose sharply in 2024 (sources: A, C).
2. Film industry data is limited; only one 2025 survey found (source: F).
3. Two credible sources disagree on publishing-industry job impact (sources: D, E).

## SECTION 1: Music industry (from web-researcher-1)
...
## SECTION 2: Film and video (from web-researcher-2)
...
## SECTION 3: Publishing (from doc-analyst)
...

## TASK
Write the synthesis. Make sure every KEY FINDING above is covered.
```

---

## 5. Upstream agents should return structured, compact data

In multi-agent systems, a downstream agent (e.g. the synthesizer) has a **limited context budget**. If each upstream agent returns **long prose and full reasoning chains**, the downstream agent fills up fast and starts missing things.

**Change upstream outputs** to **structured data**:

| ❌ Verbose upstream output | ✅ Structured upstream output |
|---|---|
| 3 pages of "First I searched for…, then I noticed…, which suggests…" | `{"key_facts": [...], "citations": [...], "relevance_score": 0.8}` |

And **require metadata** in those outputs so downstream work stays accurate:

```json
{
  "claim": "68% of surveyed indie musicians used AI mastering tools in 2024.",
  "source_url": "https://example.org/indie-music-survey-2025",
  "source_title": "Indie Music Tech Survey",
  "publication_date": "2025-01-15",
  "data_collection_period": "2024-09 to 2024-11",
  "methodology": "online survey, n=1,200, self-selected",
  "relevance_score": 0.9
}
```

Dates, source locations, and methodology let the synthesizer weigh and cite findings correctly (more in [06](06-provenance-and-uncertainty.md)).

---

## 6. Always send complete history (and why that makes trimming important)

The API is stateless. To keep the conversation **coherent**, each request must include the **conversation history** the model needs: earlier messages, its own tool calls, and their results. If you drop turns carelessly, the model may repeat questions, contradict earlier answers, or lose track of what was done.

This is why the techniques above matter. You **keep sending history**, so you **control what goes into it**. Use trimmed tool results, case facts, and structured issue lists instead of cutting history randomly.

### Prompt caching (know that it exists)

**Prompt caching** lets the API reuse a long prompt prefix that you send again and again, such as a long system prompt or document. The repeated part then costs less and returns faster. The exam tests only that it exists. Implementation details are out of scope (exam guide §17; Sources checked table in the [README](README.md), row A9).

---

## 7. Real-world case study

**Situation:** A telecom support agent handled long chats (30+ turns). To control cost, the team summarized history every 10 turns.

**Symptoms:**
- The agent told customers "your refund is being processed" without the amount, and once quoted the wrong amount.
- A promised callback time disappeared from context; the customer was never called.
- Every `get_account` call added ~60 fields; by turn 20, most of the context was account data.

**Fixes:**
1. A **case facts block** (account ID, amounts, dates, promises), updated from tool results and included in every request.
2. A **structured issue list** for multi-issue chats.
3. Tool outputs **trimmed** to 6–8 relevant fields.
4. For supervisor summaries built from many chats: **key findings first**, then sections with headers.

**Result:** Correct amounts and promises kept across long chats, and context use per turn dropped significantly.

---

## Common Q&A

**Q: Why not just summarize the whole history?**
A: Summaries blur exact facts (amounts, dates, promises). Keep those in a **case facts block** that's never summarized.

**Q: Who updates the case facts block?**
A: Preferably your **code**, from tool results, so values are exact and not paraphrased by the model.

**Q: Why trim tool outputs?**
A: Every result is re-sent on later turns. Irrelevant fields waste tokens and attention.

**Q: What is "lost in the middle" and how do I reduce it?**
A: Models may miss content in the middle of long inputs. Put a **key findings summary first**, use **section headers**, and restate the key task at the end.

**Q: What should upstream subagents return?**
A: **Structured data**: key facts, citations, relevance scores, dates, source locations, methodology. Not long reasoning chains.

---

## Key Takeaways

- **Progressive summarization** loses numbers, percentages, dates, and promises.
- Keep a persistent **case facts block** (and a **structured issue layer** for multi-issue sessions) outside summarized history.
- **Trim tool outputs** to relevant fields **before** they accumulate.
- Fight **lost in the middle**: key findings **first**, **section headers**, key task at the end.
- Upstream agents return **structured, compact data with metadata**, not verbose reasoning.
- Always send the history the model needs, and control what goes into it.
