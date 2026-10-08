# 04 · Validation, Retry, and Feedback Loops for Extraction Quality

**Exam link:** Task Statement 4.4: *Implement validation, retry, and feedback loops for extraction quality.*
**Prerequisite:** [03 · Structured Output](03-structured-output-tool-use-schemas.md) (schemas fix syntax, not semantics).

---

## 1. The quality loop

After [03](03-structured-output-tool-use-schemas.md), every extraction has the **right shape**. Now we need the **right meaning**. A robust pipeline looks like this:

```
document ─▶ extract (tool use + schema) ─▶ validate (code) ──┬─ valid ─▶ save
                                                            │
                                                            └─ invalid ─▶ fixable? ──┬─ yes ─▶ retry with error feedback
                                                                                     │
                                                                                     └─ no ──▶ flag for human / mark missing
```

Three ideas: **validate semantics in code**, **retry with specific feedback**, and **know when retry can't help**.

---

## 2. Semantic validation in code

Write checks for what the schema can't express:

```python
def validate_invoice(data: dict) -> list[str]:
    """Return human-readable semantic errors; an empty list means valid."""
    errors = []
    line_sum = round(sum(item["amount"] for item in data["line_items"]), 2)
    if abs(line_sum - data["stated_total"]) > 0.01:
        errors.append(f"line_items sum to {line_sum} but stated_total is {data['stated_total']}")
    if data["po_number"] and data["po_number"] == data["invoice_number"]:
        errors.append("po_number equals invoice_number; likely copied from the wrong field")
    if data["payment_terms"] == "other" and not data["payment_terms_detail"]:
        errors.append("payment_terms is 'other' but payment_terms_detail is empty")
    return errors
```

---

## 3. Retry with error feedback

When validation fails, **don't just re-run the same request** and hope. Send a **follow-up** that includes:

1. the **original document**,
2. the **failed extraction**,
3. the **specific validation errors**.

```python
def extract_with_retry(document_text: str, max_attempts: int = 3) -> dict:
    messages = [{"role": "user", "content": f"Extract the invoice fields.\n<document>\n{document_text}\n</document>"}]
    for attempt in range(max_attempts):
        response = client.messages.create(model=MODEL, max_tokens=4096,
                                          tools=[extract_invoice], messages=messages)
        tool_use = next((b for b in response.content if b.type == "tool_use"), None)
        if tool_use is None:
            raise ExtractionFailed([f"no extraction tool call (stop_reason={response.stop_reason})"])
        errors = validate_invoice(tool_use.input)
        if not errors:
            return tool_use.input

        # Feed back the exact problems so the model can self-correct
        messages.append({"role": "assistant", "content": response.content})
        messages.append({"role": "user", "content": [{
            "type": "tool_result",
            "tool_use_id": tool_use.id,
            "is_error": True,
            "content": "Validation failed:\n- " + "\n- ".join(errors) +
                       "\nRe-read the document and call extract_invoice again with corrected values. "
                       "If a value is genuinely not in the document, use null.",
        }]})
    raise ExtractionFailed(errors)  # route to human review
```

**Why it works:** "Try again" gives no direction. "Line items sum to 1,240 but stated_total is 1,420" tells the model **exactly where to look**: maybe it missed a line item, or swapped digits.

---

## 4. When retries can't help

Retries fix problems that come from **how the model read or formatted** the document. They **cannot** create information that **isn't there**.

| Problem | Will retry help? | Why |
|---|---|---|
| Date in the wrong format | ✅ Yes | The value exists; just reformat |
| Value placed in the wrong field | ✅ Yes | The value exists; move it |
| Missed a line item | ✅ Usually | The model can re-read and find it |
| Output structure wrong (nested incorrectly) | ✅ Yes | Structural fix |
| **Field is not in the document at all** | ❌ No | No amount of retrying can find it |
| **Info is in a different document** (e.g. "terms as per master agreement") that wasn't provided | ❌ No | Must fetch the other document, or mark missing |

> **Exam pattern:** "Retries keep failing for one field in some documents." Check whether the information **exists in the source**. If it's in an **external document not provided**, retries are wasted. Provide the document, accept `null`, or route to a human.

**Design rule:** Cap retries (e.g. 2–3). After that, **escalate** instead of looping.

---

## 5. Self-checking fields in the schema

You can make the model **show its working** so discrepancies are visible:

| Field | Purpose |
|---|---|
| `stated_total` | The total **as printed** in the document |
| `calculated_total` | The sum of extracted line items, **calculated** by the model |
| `conflict_detected` | `true` if the two don't match, or the source data is inconsistent |
| `conflict_detail` | Short explanation |

```json
{
  "stated_total": 1420.00,
  "calculated_total": 1240.00,
  "conflict_detected": true,
  "conflict_detail": "Printed total differs from sum of line items by 180.00; possible missing line or typo in source."
}
```

**Why this helps:**
- Some **source documents are genuinely wrong** (the supplier's total has a typo). Retrying won't "fix" that; it's a **real conflict** to flag, not an extraction error.
- It separates **"the model made a mistake"** from **"the document is inconsistent"**.
- Your code can route `conflict_detected: true` cases to humans.

---

## 6. Feedback loops for review findings: `detected_pattern`

For **review systems** (e.g. code review in CI), the feedback signal is **developers dismissing findings**. To learn from dismissals, you need to know **what triggered** each finding.

Add a **`detected_pattern`** field:

```json
{
  "location": "src/api/orders.py:57",
  "issue": "Broad exception handler may hide errors",
  "severity": "minor",
  "detected_pattern": "except_exception_with_logging",
  "suggested_fix": "Catch specific exceptions"
}
```

Then analyze dismissals by pattern:

| `detected_pattern` | Findings | Dismissed | Dismissal rate |
|---|---|---|---|
| `sql_string_concat` | 40 | 2 | 5% |
| `missing_auth_check` | 25 | 3 | 12% |
| `except_exception_with_logging` | 120 | 102 | **85%** |

**Action:** The `except_exception_with_logging` pattern is mostly noise (logging + re-raise is the team's accepted style). Update the criteria and add a few-shot "acceptable" example ([01](01-explicit-criteria-precision.md), [02](02-few-shot-prompting.md)), or disable that pattern.

Without `detected_pattern`, you'd only know "developers dismiss 40% of findings", with no idea **which** kinds.

---

## 7. Real-world case study

**Situation:** A logistics company extracts data from 20,000 bills of lading per day.

**Problems:**
1. 8% failed validation (weights didn't sum, dates in the wrong field). The team retried with the **same prompt**; most failed again.
2. One field, "insurance value", failed **every retry** on certain carriers' documents.

**Fixes:**
1. Retries now include the **document, failed output, and exact validation errors**. Most failures were fixed on the first retry.
2. Investigation showed those carriers put insurance value in a **separate certificate**, not in the bill of lading. Retrying was pointless. The field is now **nullable**, and those records are linked to the certificate in a later step.
3. Added `calculated_total_weight` and `conflict_detected`. Some source documents really were inconsistent, and they are now flagged for humans instead of being "fixed" by guesswork.

---

## Common Q&A

**Q: What should a retry request include?**
A: The original document, the failed extraction, and the **specific** validation errors.

**Q: When is retrying a waste?**
A: When the needed information **isn't in the provided source** (missing, or in another document). Retries fix format and structure, not absence.

**Q: What's the difference between schema errors and semantic errors?**
A: Schema errors (invalid JSON, wrong types) are eliminated by tool use with schemas. Semantic errors (wrong sums, wrong fields) need validation logic.

**Q: Why extract both `stated_total` and `calculated_total`?**
A: To expose discrepancies. If they differ, it's either an extraction error (retry) or a real problem in the source (flag with `conflict_detected`).

**Q: What is `detected_pattern` for?**
A: To record which code construct triggered a finding, so you can analyze which patterns developers dismiss and fix the false-positive sources.

**Q: How many retries?**
A: A small cap (2–3), then escalate to a human or mark the field missing.

---

## Key Takeaways

- Validate **semantics in code**: sums, field placement, consistency.
- **Retry with error feedback**: document + failed output + **specific errors**.
- Retries fix **format and structure**, not **missing information**; spot the cases where retries can't succeed.
- Use **self-check fields** (`calculated_total` vs `stated_total`, `conflict_detected`) to separate model mistakes from bad source data.
- Add **`detected_pattern`** to findings so you can analyze **dismissal patterns** and cut false positives.
- **Cap retries** and escalate.
