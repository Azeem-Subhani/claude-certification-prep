# 05 · Human Review Workflows and Confidence Calibration

**Exam link:** Task Statement 5.5: *Design human review workflows and confidence calibration.*
**Prerequisites:** Domain 4 [04 · Validation and Retry](../domain-4-prompt-engineering-structured-output/04-validation-retry-feedback-loops.md), Domain 4 [06 · Confidence Routing](../domain-4-prompt-engineering-structured-output/06-multi-instance-multi-pass-review.md).
**Scope note:** Domain 4 introduced calibrating confidence before routing **review findings**. This file focuses on **extraction pipelines**: how to decide **where humans are still needed**, how to **measure** quality safely, and **field-level** confidence.

---

## 1. The goal: use human reviewers where they matter

Human review is accurate but **slow and expensive**. Teams want to **reduce** it by automating "high-confidence" extractions. The danger is automating too much, too soon, and letting errors flow into downstream systems unnoticed.

A good human review workflow answers:
1. **Where** is the model reliable enough to automate?
2. **How do we keep measuring** quality after automation?
3. **Which items** should humans see, given **limited capacity**?

---

## 2. Aggregate accuracy can hide weak segments

"The model is **97% accurate** overall" sounds ready to automate. But look by **segment**:

| Document type | Share of volume | Accuracy |
|---|---|---|
| Typed invoices | 80% | 99.5% |
| Scanned receipts | 15% | 93% |
| Handwritten delivery notes | 5% | **71%** |
| **Overall** | 100% | **~97%** |

And by **field**:

| Field | Accuracy |
|---|---|
| `invoice_number` | 99.8% |
| `total` | 99.1% |
| `tax_id` | **84%** |

If you automate everything because of the 97% average, **handwritten notes** and **`tax_id`** errors go straight into accounting.

> **Rule:** Before reducing human review, **analyze accuracy by document type and by field**. Automate only segments that meet the bar **on their own**.

---

## 3. Field-level confidence, calibrated

Ask the model to output a **confidence score per field**, not just one per document:

```json
{
  "invoice_number": {"value": "INV-88123", "confidence": 0.98},
  "total":          {"value": 1420.00,     "confidence": 0.95},
  "tax_id":         {"value": "DE81234567", "confidence": 0.62},
  "due_date":       {"value": null,        "confidence": 0.90, "note": "not present in document"}
}
```

Then **calibrate** with a **labeled validation set** (documents where the correct answers are known):

1. Run the model on the validation set.
2. Group fields by confidence band (e.g. 0.9–1.0, 0.8–0.9, …).
3. Measure **actual accuracy** in each band, per field (and per document type).
4. Set **review thresholds** from the data.

| `tax_id` confidence band | Actual accuracy | Decision |
|---|---|---|
| ≥ 0.95 | 99.2% | Auto-accept |
| 0.80–0.95 | 92% | Human review |
| < 0.80 | 61% | Human review (priority) |

Different fields may need **different thresholds**: a 0.9 on `invoice_number` might be safe, while a 0.9 on `tax_id` isn't.

---

## 4. Routing: prioritize limited reviewer capacity

Send to humans:

| Route to review | Why |
|---|---|
| **Low-confidence fields** (below calibrated threshold) | Most likely to be wrong |
| **Ambiguous source documents** (unreadable, partial scans) | Model can't be reliable |
| **Contradictory sources** (`conflict_detected: true`, see Domain 4 · [04](../domain-4-prompt-engineering-structured-output/04-validation-retry-feedback-loops.md)) | Needs human judgment |
| **Weak segments** (e.g. handwritten notes) until they improve | Segment accuracy below the bar |

When reviewer capacity is limited, **prioritize** by risk: high-value amounts, low confidence, and regulated fields first.

---

## 5. Keep measuring: stratified random sampling

After automation, high-confidence extractions are **not reviewed** by default, so how do you know they stay correct?

**Stratified random sampling:** regularly take a **random sample** from **each segment** (document type × field × confidence band), including **high-confidence, auto-accepted** items, and have humans check them.

| Stratum | Sample per week |
|---|---|
| Typed invoices, high confidence | 50 |
| Scanned receipts, high confidence | 50 |
| Handwritten notes, high confidence | 50 |
| … | … |

**Why stratified (not just random)?** A plain random sample is mostly typed invoices (80% of volume), so it would rarely include the small, risky segments. Stratifying guarantees **every segment is measured**.

**What it gives you:**
- **Ongoing error rates** per segment, even for auto-accepted items.
- **Detection of novel error patterns**: e.g. a supplier changes its invoice layout and errors appear in a segment that used to be safe.

---

## 6. Real-world case study

**Situation:** An insurer automated claim-form extraction after measuring **96% overall accuracy**. Three months later, auditors found many wrong **policy numbers** in claims from one regional office.

**Root cause:**
- That office used an older **handwritten** form; accuracy on it was ~70%, hidden inside the 96% average.
- High-confidence items were never checked after automation, so the problem went unnoticed.

**New workflow:**
1. Accuracy measured **by form type and field** before any automation decision.
2. **Field-level confidence**, calibrated on 2,000 labeled forms; per-field thresholds.
3. Handwritten forms **always** go to human review until their accuracy improves.
4. Weekly **stratified random sampling** of auto-accepted items in every segment.
5. Reviewer queue prioritized by claim value and confidence.

---

## Common Q&A

**Q: Our model is 97% accurate overall. Can we remove human review?**
A: Not yet. Check accuracy **by document type and field** first. The average can hide weak segments.

**Q: Why field-level confidence instead of document-level?**
A: Some fields are much harder than others. Field-level scores let you review only the risky fields.

**Q: How do I choose confidence thresholds?**
A: Calibrate on a **labeled validation set**: measure actual accuracy per confidence band (per field), then set thresholds from that data.

**Q: Once high-confidence items are auto-accepted, how do I know they stay accurate?**
A: **Stratified random sampling** of auto-accepted items in every segment, reviewed by humans on a regular schedule.

**Q: What should go to human reviewers first when capacity is limited?**
A: Low-confidence fields, ambiguous or contradictory documents, weak segments, and high-risk/high-value items.

---

## Key Takeaways

- **Aggregate accuracy hides weak segments**. Analyze **by document type and field** before automating.
- Use **field-level confidence**, **calibrated** on **labeled validation sets**, with per-field thresholds.
- Route **low-confidence, ambiguous, and contradictory** items to humans; **prioritize** limited capacity.
- Keep measuring with **stratified random sampling**, including **high-confidence** items, to catch **novel error patterns**.
