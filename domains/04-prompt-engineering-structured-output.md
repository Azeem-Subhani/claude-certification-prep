# Domain 4: Prompt Engineering & Structured Output

> **Exam weight:** 20% of scored items (official exam guide v1.0, §4).
> **Task statements:** 4.1 to 4.6 (see [docs/domain-map.md](../docs/domain-map.md)).
> **Deep dives:** [domain-4-prompt-engineering-structured-output/](../domain-4-prompt-engineering-structured-output/README.md)
> **Facts checked against:** [docs/verification-log.md](../docs/verification-log.md). ⚠️ marks a place where the exam answer and today's product differ.

---

## Exam focus

- **Explicit criteria (4.1).** Clear categories of what to report beat vague words like "be conservative".
- **Few-shot examples (4.2).** When instructions alone give inconsistent results, add 2 to 4 targeted examples.
- **Schemas (4.3, 4.4).** Tool use with a JSON schema removes **syntax** errors. It does not remove **meaning** errors. Those need validation and retries.
- **Batch processing (4.5).** The Message Batches API is cheaper but slow, with no speed guarantee. Use it only for work that nobody waits for.
- **Review architecture (4.6).** Use an independent instance to review, and per-file passes plus a cross-file pass. Calibrate confidence scores before you route on them.

---

## Key concepts

### 1. Words you need first

A **false positive** is a finding that is reported but is not a real problem. Too many false positives make developers ignore **all** findings.

**Precision** is the share of reported findings that are real. This domain is mostly about raising precision without missing real bugs.

A **schema** is a formal description of the data shape: fields, types, and which fields are required. **JSON Schema** is the standard used by Claude tools.

### 2. Explicit criteria (Task 4.1)

Vague instructions do not work. "Be conservative" or "only report high-confidence findings" does not tell Claude **what** to report.

Write **categorical criteria** instead:

```text
REPORT:
- Bugs that change behavior (wrong condition, missing null check, off-by-one).
- Security issues (injection, secrets in code, missing auth check).
- A comment that says the code does X when the code does Y.

DO NOT REPORT:
- Style and naming preferences.
- Patterns that are normal in this codebase.
```

**Severity levels** should each have a short **code example**, not just a prose definition. Then Claude classifies findings the same way each time.

**Noisy category?** One category may produce many false positives and make developers ignore everything. Then:
1. Turn that category **off** for now.
2. Fix its prompt with criteria and examples.
3. Turn it back on.

### 3. Few-shot prompting (Task 4.2)

**Few-shot examples** are 2 to 4 worked examples placed in the prompt. They are the most effective fix when detailed instructions still give inconsistent output.

Use them to:
- **Fix the output format.** Show the exact structure: location, issue, severity, suggested fix.
- **Show judgment on unclear cases.** Include the reasoning: why one option was chosen over a plausible alternative. Then Claude can **generalize** to new cases, not just copy the examples.
- **Separate real issues from acceptable patterns.** Show one "flag this" and one "this is fine" example.
- **Handle varied documents.** Show extraction from a table and from narrative text. Show inline citations and a bibliography. This stops empty fields when the data is present but in an unusual place.

Choose examples that **target the failing cases**. Ten random examples are worse than three targeted ones.

### 4. Structured output with tool use (Task 4.3)

Asking for "JSON in your reply" can give broken JSON. The reliable approach is **tool use with a JSON schema**. You define an extraction tool. Claude "calls" it, and the tool input is your structured data.

```json
{
  "name": "record_invoice",
  "description": "Record the fields extracted from one supplier invoice.",
  "input_schema": {
    "type": "object",
    "properties": {
      "invoice_number": {"type": "string"},
      "po_number": {"type": ["string", "null"], "description": "null if the invoice shows no PO number"},
      "category": {"type": "string", "enum": ["goods", "services", "other", "unclear"]},
      "category_detail": {"type": ["string", "null"]},
      "stated_total": {"type": "number"},
      "calculated_total": {"type": "number"},
      "conflict_detected": {"type": "boolean"}
    },
    "required": ["invoice_number", "category", "stated_total", "calculated_total", "conflict_detected"]
  }
}
```

**Schema design rules:**
- **Optional or nullable fields** for data that may be missing. If every field is required, Claude may **invent** a value to fill it.
- **An `"unclear"` enum value** for cases that really are ambiguous.
- **An `"other"` value plus a detail field** for categories you did not list.
- **Format rules in the prompt** next to the schema. Example: "Dates as YYYY-MM-DD. Amounts as plain numbers, no currency signs."

**`tool_choice` in extraction:**
- `"auto"` lets Claude answer in text instead of calling the tool.
- `"any"` makes Claude call one of your tools. Use it when you have several schemas and the document type is unknown.
- **Forced** (`{"type": "tool", "name": "extract_metadata"}`) makes one specific step run first. Use `"auto"` again for the next steps.

> ⚠️ **Product note:** on Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1, `"any"` and forced choice return a 400 error. On those models, use `"auto"` with `strict: true` tools, or **structured outputs** (`output_config.format`). Both use **constrained decoding**: the model can only produce tokens that fit the schema. Outputs can still break on a refusal or a `max_tokens` cut-off, so keep validating (verification log A5b, A7a to A7c).

**Schemas fix syntax, not meaning.** A schema guarantees the shape. It cannot catch:
- line items that do not add up to the total
- a value placed in the wrong field
- an invented value

These need extra validation.

### 5. Validation, retry, and feedback loops (Task 4.4)

**Retry with error feedback.** When validation fails, send a follow-up request with:
1. the original document
2. the failed extraction
3. the **specific** validation error, for example: "Line items sum to 1,240.00 but stated_total is 1,420.00."

Claude can then correct itself.

**When retries help:** format mismatches, structure errors, misplaced values, math errors.
**When retries do not help:** the information is **not in the document**, or it is only in another document you did not provide. No retry can create missing data. Return null or send the item to human review.

**Self-checking fields:**
- `stated_total` plus `calculated_total` lets code spot a mismatch automatically.
- `conflict_detected: true` marks a document that contradicts itself. Extract both values. Do not pick one.

**Feedback loops for reviews.** Add a `detected_pattern` field to each finding. Track which patterns developers dismiss. Then improve the prompt for those patterns.

**Pydantic** is a Python library for data validation. Use its validators for rules that compare fields. Feed its error messages into the retry. Keep it even when you use tool use, because the schema checks only shape.

### 6. Batch processing (Task 4.5)

The **Message Batches API** runs many requests in the background.

| Fact | Value (confirmed in docs) |
|---|---|
| Cost | 50% of the standard price |
| Time | Most batches finish within 1 hour, but it can take up to 24 hours. Requests not finished by then **expire**. |
| Speed guarantee | None. There is no latency SLA (service-level agreement). |
| Matching results | Results can come back in **any order**. Match them with your `custom_id`. |
| Limits | 100,000 requests or 256 MB per batch. Results are kept for 29 days. |
| Result types | `succeeded`, `errored`, `canceled`, `expired` |

**When to use it:**

| Work | API |
|---|---|
| Someone waits for the result (pre-merge check, live chat) | Synchronous (real-time) |
| Results are used later (nightly tests, weekly audit, overnight report, bulk extraction) | Batch |

**No client tool loop inside one batch request.** A batch request cannot stop, wait for your code to run a tool, and continue. If a step needs your tools during processing, use the synchronous API.

> ⚠️ **Product note:** batch requests **may** include tools, and server tools work in batches (verification log A8f). What is not possible is your own multi-turn tool loop inside one request. That is the exam point.

**SLA math.** Suppose each document must be processed within 30 hours of arrival.
1. A batch can take up to 24 hours. So a document may wait at most 30 − 24 = 6 hours before it is submitted.
2. If you submit every N hours, a document can wait up to N hours. So N must be at most 6.
3. Exactly 6 leaves no safety margin, so submit every 4 hours. This matches the guide's example.

**Failures.** Find the failed items by `custom_id`. Fix them (for example, split documents that were too long). Resubmit **only** those items.

**Test first.** Refine the prompt on a small sample (for example, 5 to 10 documents) before you send thousands.

### 7. Multi-instance and multi-pass review (Task 4.6)

**Self-review is weak.** The session that wrote the output keeps its own reasoning. So it tends to agree with itself. Asking it to "review carefully", or to use extended thinking, does not remove this bias. Use a **second, independent instance** without the generation history.

**Multi-pass review** for large changes:
1. One **per-file pass** for local issues.
2. One **cross-file integration pass** for data flow between files.

**Confidence routing.** Claude can report a confidence score with each finding. Use the scores only after you **calibrate** them on a labeled set, where you know the right answers. Then:
- Send high-confidence findings to developers.
- Send low-confidence findings to human review.

---

## Common traps

| Tempting wrong answer | Why it is wrong |
|---|---|
| "Tell Claude to be conservative / only report high-confidence issues." | It does not define what to report. Precision does not improve. |
| "Raise the confidence threshold to cut false positives." | Scores without criteria and calibration do not fix the cause. |
| "Keep all categories on while you fix the noisy one." | Developers keep ignoring everything. Turn the noisy category off first. |
| "Add more detailed instructions." | Instructions already failed. Add targeted few-shot examples. |
| "Show only input and output pairs." | Without the reasoning, Claude copies examples instead of generalizing. |
| "Ask for JSON in the text reply and parse it." | It can break. Use tool use with a schema. |
| "Tool use with a schema guarantees correct data." | It guarantees the shape only. Totals and field placement need validation. |
| "Make every field required for completeness." | Claude invents values to fill required fields. Use nullable fields. |
| "Retry until the missing field appears." | If the data is not in the document, retries cannot create it. |
| "Retry with a generic 'try again'." | Claude needs the specific error to fix it. |
| "Move everything to the Batch API to save 50%." | Blocking work cannot wait up to 24 hours. |
| "Resubmit the whole batch after 3% failed." | Wasteful. Resubmit only the failed `custom_id`s, with fixes. |
| "Batches usually finish in an hour, so plan for one hour." | There is no guarantee. Plan for 24 hours. |
| "Use a bigger context window so one pass reviews all files well." | Attention dilution remains. Use per-file passes plus a cross-file pass. |

---

## Worked scenario

**Setting:** An accounts-payable team extracts data from 40,000 supplier invoices each month. Formats vary: PDF tables, scanned letters, and emails with the details in the text. Results go into the finance system the next morning.

**Step 1: Define the extraction tool.**
- The team defines `record_invoice` (see the schema above).
- `po_number` is nullable, because some invoices have no PO number.
- `category` has `"other"` (with a detail field) and `"unclear"`.

**Step 2: Make the call reliable.**
- The model in use supports forced tool use, so the team forces `record_invoice` for each request.
- (On a newer model that rejects forced choice, they would use `strict: true` with `"auto"`, or structured outputs.)

**Step 3: Add format rules.**
- The prompt says: "Dates as YYYY-MM-DD. Amounts as numbers without currency signs."

**Step 4: Add few-shot examples.**
- Fields come back empty for emails, where the amounts are in sentences.
- The team adds three examples: a table invoice, an email invoice, and an invoice with no PO number. Each shows the reasoning.

**Step 5: Validate in code.**
- A Pydantic model checks that `calculated_total` equals `stated_total`.
- On a mismatch, the pipeline retries with the document, the failed output, and the exact error.
- If the invoice itself has two different totals, `conflict_detected` is set to true and the item goes to human review.

**Step 6: Use batch.**
- Nobody waits for these results until the next morning.
- The team submits through the Message Batches API at 50% cost, with each invoice ID as the `custom_id`.
- Before the first full run, they test the prompt on 10 sample invoices.

**Step 7: Handle failures.**
- 120 requests fail because the documents are too long.
- The team splits those documents and resubmits only the 120 `custom_id`s.

---

## Quick recap

- Replace vague instructions with categorical REPORT / DO NOT REPORT criteria. Give each severity level a code example.
- When instructions are not enough, add 2 to 4 targeted few-shot examples that show the reasoning.
- Tool use with a schema stops syntax errors. Validation plus retry handles meaning errors.
- Use nullable fields, `"unclear"`, and `"other"` + detail so Claude does not invent values.
- Retry with the specific error. Do not retry when the data is not in the source.
- Batch = 50% cheaper, up to 24 hours, no speed guarantee, `custom_id`. Use it only for non-blocking work.
- Review with an independent instance and per-file plus cross-file passes. Calibrate confidence before you route on it.

---

## Practice questions

Go to the Domain 4 question bank: **[mcq-bank/04-prompt-engineering-structured-output.md](../mcq-bank/04-prompt-engineering-structured-output.md)** (48 questions).
Older, easier warm-up questions: [domain-4-.../07-scenario-practice-questions.md](../domain-4-prompt-engineering-structured-output/07-scenario-practice-questions.md).
