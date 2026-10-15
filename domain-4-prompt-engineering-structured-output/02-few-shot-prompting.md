# 02 · Few-Shot Prompting for Consistency and Quality

**Exam link:** Task Statement 4.2: *Apply few-shot prompting to improve output consistency and quality.*
**Prerequisite:** [01 · Explicit Criteria](01-explicit-criteria-precision.md).
**Scope note:** Domain 3 · [05](../domain-3-claude-code-configuration-workflows/05-iterative-refinement.md) covered input/output examples as a way to *clarify a transformation while you work interactively*. This file covers few-shot examples as a **permanent part of a production prompt**: for format, ambiguous judgment, false-positive reduction, and extraction.

---

## 1. What is few-shot prompting?

**Few-shot prompting** means including a **small number of worked examples** in the prompt. Each example shows an input and the exact output (and sometimes the reasoning) you want.

- **Zero-shot**: instructions only.
- **Few-shot**: instructions + 2–4 examples.

> When detailed instructions alone still give **inconsistent** results, few-shot examples are the **most effective** technique for consistent, well-formatted, actionable output.

**Why it works:** Instructions are open to interpretation. Examples are concrete: they show the exact format, level of detail, and judgment you expect.

---

## 2. Use 1: Lock in the output format

**Problem:** Review findings come back in different shapes: sometimes a paragraph, sometimes bullets, sometimes without line numbers or a fix.

**Fix:** show the exact format in examples:

```text
Report each finding in exactly this format.

Example 1
Location: src/billing/invoice.py:42
Issue: Discount is applied after tax, so customers are overcharged on discounted orders.
Severity: major
Suggested fix: Apply `apply_discount()` before `add_tax()`.

Example 2
Location: src/api/users.py:88
Issue: User ID from the URL is used in the query without checking it belongs to the caller.
Severity: critical
Suggested fix: Add `require_owner(request.user, user_id)` before the query.
```

Every finding now has **location, issue, severity, suggested fix**, which is what makes it actionable.

---

## 3. Use 2: Show judgment in ambiguous cases

Some decisions are hard to describe in rules. Examples can show **how to decide**, including **why** one option beats plausible alternatives.

### Example: tool selection for ambiguous requests (support agent)

```text
Example (ambiguous request)
Customer: "I want to cancel."
Context: The customer has an active subscription AND an open order placed 1 hour ago.
Correct action: Ask which one they mean before calling any tool.
Why: Calling cancel_subscription or cancel_order would both be plausible, but each is
hard to undo. A one-line clarifying question is cheaper than a wrong cancellation.

Example (not ambiguous)
Customer: "Cancel order 88123, it's a duplicate."
Correct action: Call cancel_order(order_id="88123").
Why: The order is named explicitly; asking again would just slow the customer down.
```

### Example: branch-level test coverage gaps (CI test review)

```text
Example
Code: if user.is_admin or user.id == resource.owner_id: allow()
Existing tests: test_admin_can_access
Finding: Missing test for the non-admin owner branch (user.id == resource.owner_id).
Why: Line coverage is 100% because the line runs, but one of the two conditions is never
the deciding one. Branch-level gaps like this hide real permission bugs.
```

**Key point:** The examples include **reasoning**. That teaches the model a **way of thinking**, so it can **generalize to new cases** rather than only matching the exact patterns in the examples.

---

## 4. Use 3: Separate acceptable patterns from real issues

False positives often come from code that **looks** suspicious but is fine. Pair examples: one acceptable, one genuine issue.

```text
Example A (acceptable: do NOT report)
    except FileNotFoundError:
        return default_config()   # intentional: missing config file means "use defaults"
Why not an issue: Specific exception, deliberate fallback, documented.

Example B (genuine issue: REPORT)
    except Exception:
        pass
Why an issue: Swallows every error, including bugs and outages, with no logging.
```

This **reduces false positives**. The model learns "catching an exception" isn't automatically bad, while it still catches real problems. It also **generalizes** to similar patterns not shown.

---

## 5. Use 4: Reduce hallucination in extraction

In **extraction** tasks, models may **invent** values when the document is messy or a field is missing. Few-shot examples show what to do with hard inputs.

### Informal measurements

```text
Example
Text: "Patient reports drinking 'a couple of glasses' of wine most evenings."
Output: {"alcohol_quantity": null, "alcohol_description": "a couple of glasses, most evenings"}
Why: No exact number is given. Don't convert "a couple" into 2. Keep the original wording.
```

### Varied document structures

Research papers cite sources in different ways. Show each structure:

```text
Example 1: inline citations
Text: "...as shown previously (Garcia et al., 2021)."
→ citations: [{"author": "Garcia et al.", "year": 2021, "source_location": "inline"}]

Example 2: numbered bibliography
Text: "...as shown previously [14]." + References section: "14. Garcia, M. ... 2021"
→ citations: [{"author": "Garcia, M.", "year": 2021, "source_location": "bibliography"}]

Example 3: methodology details embedded in prose (no "Methods" heading)
Text: "We surveyed 412 nurses across 9 hospitals during spring 2023..."
→ {"sample_size": 412, "sites": 9, "methodology_section_present": false}
```

### Empty or null required fields

```text
Example
Invoice text has no PO number anywhere.
Output: {"po_number": null, ...}
Why: The field is required in our data model, but the document doesn't contain it.
Return null; never guess or copy another number (like the invoice number).
```

This connects to schema design in [03](03-structured-output-tool-use-schemas.md): the schema must **allow** `null`, and examples **show** when to use it.

---

## 6. How to write good few-shot examples

| Guideline | Why |
|---|---|
| Use **2–4** examples | Enough to show the pattern; more adds tokens and can over-fit |
| **Target the hard cases** (ambiguous, edge, commonly confused) | Easy cases don't need examples |
| **Vary** the examples | Three near-identical examples teach a too-narrow pattern |
| Include **brief reasoning** ("Why: …") | Helps the model generalize judgment |
| Show **both** positive and negative cases | Reduces false positives |
| Keep the **format identical** across examples | The model copies the format exactly |
| Use **realistic** data | Toy examples may not transfer to real inputs |

---

## 7. Real-world case study

**Situation:** A research-analytics firm extracts study details from 50,000 papers. With instructions only:
- Sample sizes were **invented** for papers that didn't state them.
- Citations in bibliographies were missed; only inline ones were found.
- Output format varied (some fields renamed, some nested differently).

**Change:** Added **four few-shot examples**. They were one inline-citation paper, one bibliography paper, and one with methodology embedded in prose. The last had a missing sample size (`null`, with "Why: not stated").

**Result:** Invented sample sizes dropped sharply on the validation set, bibliography citations were captured, and the format became consistent.

---

## Common Q&A

**Q: When should I add few-shot examples instead of more instructions?**
A: When detailed instructions still give inconsistent format or judgment. Examples are the most effective next step.

**Q: How many examples?**
A: Usually **2–4**, targeted at ambiguous or tricky cases.

**Q: Won't the model just copy my examples?**
A: It copies the **format**. If examples are varied and include reasoning, it **generalizes the judgment** to new cases rather than only matching listed patterns.

**Q: How do examples reduce false positives?**
A: By showing acceptable code next to genuinely problematic code, so the model learns where the line is.

**Q: How do examples reduce hallucination in extraction?**
A: By showing what to do when information is informal, oddly structured, or missing, including returning `null` instead of guessing.

---

## Key Takeaways

- Few-shot examples are the **most effective** fix when instructions alone give **inconsistent** output.
- Use **2–4 targeted** examples for **ambiguous** cases, with **reasoning** for why one choice beats alternatives.
- Examples lock in **format** (location, issue, severity, fix).
- Pair **acceptable vs problematic** examples to cut false positives while still generalizing.
- In extraction, examples handle **informal values, varied structures, and missing fields (null)**, which reduces hallucination.
