# 07 · Scenario Practice Questions (Domain 4)

**Exam link:** All of Domain 4 (Task Statements 4.1–4.6).
**How to use:** Answer each question before you open the explanation. Each answer links to the file that teaches the concept. These are **original practice questions** written for study. They are not taken from the official exam or practice test.

**Where Domain 4 appears:** It is a primary domain in **Claude Code for Continuous Integration** and **Structured Data Extraction**.

---

## How to approach Domain 4 questions

1. **Too many false positives?** → **Explicit categorical criteria**, not "be conservative". Disable noisy categories.
2. **Inconsistent format or judgment?** → **Few-shot examples** (2–4, targeted, with reasoning).
3. **Broken JSON?** → **Tool use + schema**. **Wrong values?** → **Semantic validation + retry with errors**.
4. **Invented values?** → **Nullable fields** + examples showing `null`.
5. **Cost/latency question?** → **Batch only non-blocking work**; do the SLA math.
6. **Reviewer misses issues in generated code?** → **Independent instance**, not self-review or more thinking.

---

## Scenario A: Claude Code for Continuous Integration

*Claude reviews every pull request in CI and posts findings as comments.*

### A1
The CI reviewer flags many comment-related issues, mostly wording nitpicks. You want it to flag only comments that are actually misleading. Which instruction is best?

- A. "Check that comments are accurate."
- B. "Be conservative when flagging comments."
- C. "Flag a comment only when the behavior it claims contradicts what the code actually does."
- D. "Only report comment issues you are highly confident about."

<details><summary>Answer</summary>

**C.** Explicit, categorical criteria define exactly what counts. A is vague; B and D don't define the category. → [01](01-explicit-criteria-precision.md)
</details>

### A2
Developers dismiss 70% of findings. Analysis shows the "performance" category has 85% dismissals, while "security" findings are 92% accurate. Developers have started ignoring security findings too. What is the best immediate action?

- A. Add "only report high-confidence findings" to the prompt.
- B. Temporarily disable the performance category while improving its prompt, keeping security enabled.
- C. Disable the bot entirely.
- D. Increase the number of findings so more real issues get through.

<details><summary>Answer</summary>

**B.** Noisy categories undermine trust in accurate ones. Disable temporarily, fix, then re-enable. → [01](01-explicit-criteria-precision.md)
</details>

### A3
The same type of bug is labeled "critical" on one PR and "minor" on another. What improves consistency most?

- A. Ask the model to think harder about severity.
- B. Define each severity level with concrete code examples.
- C. Remove severity from findings.
- D. Use a larger model.

<details><summary>Answer</summary>

**B.** Concrete examples anchor what each level means. → [01](01-explicit-criteria-precision.md)
</details>

### A4
Detailed instructions describe the finding format, but outputs still vary: some lack line numbers, some lack suggested fixes. What is the most effective fix?

- A. Repeat the format instructions three times.
- B. Add 2–3 few-shot examples showing the exact format (location, issue, severity, suggested fix).
- C. Lower the temperature.
- D. Post-process with regex.

<details><summary>Answer</summary>

**B.** Few-shot examples are the most effective way to get consistent format when instructions alone aren't enough. → [02](02-few-shot-prompting.md)
</details>

### A5
The reviewer flags every `try/except` block, including deliberate, specific fallbacks. You want it to still catch broad `except: pass` handlers. What is the best approach?

- A. Tell it to ignore exception handling.
- B. Add few-shot examples contrasting an acceptable specific fallback with a genuine broad swallow-all handler, with brief reasoning.
- C. List every acceptable exception type in the prompt.
- D. Disable the bugs category.

<details><summary>Answer</summary>

**B.** Paired examples teach the distinction and generalize to new cases. A loses real issues; C won't scale; D is too broad. → [02](02-few-shot-prompting.md)
</details>

### A6
You want to learn which kinds of findings developers dismiss most, so you can fix the prompts. What should you add to each finding?

- A. A timestamp.
- B. A `detected_pattern` field identifying the code construct that triggered it.
- C. The model's full reasoning.
- D. The developer's name.

<details><summary>Answer</summary>

**B.** `detected_pattern` enables analysis of dismissal rates by pattern. → [04](04-validation-retry-feedback-loops.md)
</details>

### A7
Your team uses Claude in two workflows: (1) a blocking pre-merge check developers wait on, and (2) a nightly technical-debt report. You want to cut API costs. What should you do?

- A. Move both to the Message Batches API.
- B. Move only the nightly report to the Message Batches API; keep the pre-merge check synchronous.
- C. Move only the pre-merge check to batch.
- D. Keep both synchronous and reduce `max_tokens`.

<details><summary>Answer</summary>

**B.** Batch is 50% cheaper but can take up to 24 hours with no latency guarantee, so it fits only non-blocking work. → [05](05-batch-processing.md)
</details>

### A8
Claude generates a code change in one session. You then ask that same session, with extended thinking enabled, to review the change. It misses a subtle bug. What is the most effective improvement?

- A. Increase the extended thinking budget.
- B. Ask it to "review as if you were a different engineer".
- C. Use a second, independent Claude instance without the generator's reasoning context to review the change.
- D. Ask it to review twice.

<details><summary>Answer</summary>

**C.** Independent instances catch subtle issues more effectively than self-review or extended thinking in the same context. → [06](06-multi-instance-multi-pass-review.md)
</details>

### A9
A PR changes 22 files. A single review pass gives deep comments on early files, shallow ones on later files, and contradictory findings about the same pattern. What architecture fixes this?

- A. One pass with a bigger context window.
- B. Per-file local passes plus a separate cross-file integration pass for data flow.
- C. Review only the first 10 files.
- D. Ask the model to be consistent.

<details><summary>Answer</summary>

**B.** Focused passes avoid attention dilution and contradictions; the integration pass catches cross-file issues. → [06](06-multi-instance-multi-pass-review.md)
</details>

### A10
You want to auto-post some findings and send others to a human. You plan to use self-reported confidence. What must you do for this to work well?

- A. Nothing; model confidence is always accurate.
- B. Calibrate thresholds against labeled outcomes, then route by confidence (e.g. high → post, medium → human, low → log).
- C. Post everything above 0.5.
- D. Ask the model to only output high-confidence findings.

<details><summary>Answer</summary>

**B.** Confidence is useful for routing only once it's calibrated against real accuracy. → [06](06-multi-instance-multi-pass-review.md)
</details>

---

## Scenario B: Structured Data Extraction

*You extract structured data from invoices, contracts, and research papers, validate it with JSON schemas, and send it to downstream systems.*

### B1
Asking for "JSON output" in the prompt causes parse failures in about 2% of responses. What is the most reliable fix?

- A. Add "make sure the JSON is valid" to the prompt.
- B. Define an extraction tool whose input schema is the desired structure, and read the data from the `tool_use` block.
- C. Retry every failed parse.
- D. Use regex to repair JSON.

<details><summary>Answer</summary>

**B.** Tool use with a JSON schema eliminates syntax errors. → [03](03-structured-output-tool-use-schemas.md)
</details>

### B2
After switching to tool use with a strict schema, all output parses, but some invoices have line items that don't sum to the total. Why?

- A. The schema is too loose.
- B. Strict schemas prevent syntax errors, not semantic errors; you need semantic validation.
- C. The model is broken.
- D. Tool use doesn't work for invoices.

<details><summary>Answer</summary>

**B.** Schemas guarantee structure, not correctness. → [03](03-structured-output-tool-use-schemas.md), [04](04-validation-retry-feedback-loops.md)
</details>

### B3
Many supplier invoices have no PO number, but every extracted record contains one, often copied from the invoice number. The schema makes `po_number` a required string. What is the best fix?

- A. Add "do not hallucinate" to the prompt.
- B. Make `po_number` nullable, instruct the model to use null when absent, and show an example.
- C. Remove `po_number` from the schema.
- D. Retry until the PO number differs from the invoice number.

<details><summary>Answer</summary>

**B.** Required non-nullable fields push the model to fabricate. Nullable fields + an example fix it. → [03](03-structured-output-tool-use-schemas.md)
</details>

### B4
Incoming documents may be invoices, receipts, or credit notes, each with its own extraction tool. The type is unknown in advance, and sometimes the model replies with text instead of calling a tool. Per the exam guide, what `tool_choice` should you use?

- A. `"auto"`
- B. `"any"`
- C. `{"type": "tool", "name": "extract_invoice"}`
- D. `"none"`

<details><summary>Answer</summary>

**B.** `"any"` guarantees a tool call while letting the model choose the right schema. (On the newest models, use `auto` + strict tools + instruction, because `"any"` returns a 400.) → [03](03-structured-output-tool-use-schemas.md)
</details>

### B5
Contract types change over time, and you want to classify them without forcing new types into wrong categories. What schema design is best?

- A. A free-text `contract_type` field.
- B. An enum with known types plus `"unclear"` and `"other"`, with a `contract_type_detail` string.
- C. An enum with only known types.
- D. A boolean per type.

<details><summary>Answer</summary>

**B.** Extensible categorization: consistent known values, honest handling of ambiguity, and a record of new types. → [03](03-structured-output-tool-use-schemas.md)
</details>

### B6
Dates arrive as "03/04/2025", "4 March 2025", and "2025-03-04". The schema field is a string. What else is needed for consistent output?

- A. Nothing; the schema handles it.
- B. Format normalization rules in the prompt (e.g. YYYY-MM-DD, how to handle ambiguous day/month).
- C. A separate model call per date.
- D. Convert dates in the database later.

<details><summary>Answer</summary>

**B.** Schemas control type and structure; normalization rules control formatting inside values. → [03](03-structured-output-tool-use-schemas.md)
</details>

### B7
An extraction fails semantic validation. What should the retry request contain?

- A. The same original prompt only.
- B. The original document, the failed extraction, and the specific validation errors.
- C. Only the validation errors.
- D. A note saying "try again, be careful".

<details><summary>Answer</summary>

**B.** Specific error feedback guides self-correction. → [04](04-validation-retry-feedback-loops.md)
</details>

### B8
For some contracts, `governing_law` fails extraction on every retry. Investigation shows those contracts say "governed by the terms of the Master Agreement", which wasn't provided. What is the right conclusion?

- A. Increase retries to 10.
- B. Retries can't succeed because the information isn't in the provided document; provide the master agreement or record null and flag it.
- C. Make the model guess the most common governing law.
- D. Remove the field.

<details><summary>Answer</summary>

**B.** Retries fix format and structure, not absent information. → [04](04-validation-retry-feedback-loops.md)
</details>

### B9
Some invoices genuinely contain arithmetic errors from the supplier. You want to tell these apart from extraction mistakes. What schema design helps?

- A. Only extract `stated_total`.
- B. Extract `stated_total` and `calculated_total`, plus a `conflict_detected` boolean.
- C. Recalculate the total and overwrite the stated value.
- D. Reject all invoices with mismatches.

<details><summary>Answer</summary>

**B.** Self-check fields expose discrepancies and flag real source conflicts for humans. → [04](04-validation-retry-feedback-loops.md)
</details>

### B10
Extraction for some papers invents numeric sample sizes from phrases like "a few dozen participants", and misses citations in bibliographies. What is the best prompt improvement?

- A. Add "never hallucinate".
- B. Add few-shot examples covering informal quantities (returning null and keeping the original text), inline citations, and bibliography-style citations.
- C. Only process papers with a Methods section.
- D. Increase `max_tokens`.

<details><summary>Answer</summary>

**B.** Examples showing varied structures and informal values reduce hallucination and improve coverage. → [02](02-few-shot-prompting.md)
</details>

### B11
You must process 1 million archived documents with a 30-hour SLA from arrival to result. Batch processing can take up to 24 hours. How often should you submit batches?

- A. Once a day.
- B. Every 12 hours.
- C. Every 4 hours (worst case 4 + 24 = 28 hours).
- D. Once, at the end of the week.

<details><summary>Answer</summary>

**C.** Submission interval + max processing time must stay within the SLA, with margin. → [05](05-batch-processing.md)
</details>

### B12
In a batch of 10,000 documents, 150 failed because they exceeded the context limit. What should you do?

- A. Resubmit the full batch.
- B. Identify the 150 by `custom_id`, chunk those documents, and resubmit only them.
- C. Ignore them.
- D. Switch everything to the synchronous API.

<details><summary>Answer</summary>

**B.** Resubmit only failures, with the cause fixed. → [05](05-batch-processing.md)
</details>

### B13
You are about to run a new extraction prompt on 500,000 documents via batch. What should you do first?

- A. Submit everything right away; batch is cheap.
- B. Refine the prompt on a representative sample to maximize first-pass success.
- C. Split into 500,000 single-request batches.
- D. Remove the schema to save tokens.

<details><summary>Answer</summary>

**B.** Sample testing avoids expensive large-scale resubmissions. → [05](05-batch-processing.md)
</details>

### B14
Your extraction pipeline needs Claude to call an internal `lookup_vendor` tool and then continue extracting based on the result. Why is the Message Batches API a poor fit for this step?

- A. Batches don't support JSON.
- B. A batch request can't execute your tool mid-request and continue; multi-turn tool calling would require separate requests.
- C. Batches are more expensive.
- D. Batches don't support system prompts.

<details><summary>Answer</summary>

**B.** → [05](05-batch-processing.md)
</details>

### B15
To check extraction accuracy, you ask the same conversation that performed the extraction to verify it against the document. It almost always confirms its own output. What is better?

- A. Ask it to verify more strictly.
- B. Use an independent instance that receives only the source document, the extraction, and the verification criteria.
- C. Skip verification.
- D. Run the same verification twice.

<details><summary>Answer</summary>

**B.** Independent instances avoid confirmation of their own reasoning. → [06](06-multi-instance-multi-pass-review.md)
</details>

---

## Score yourself

| Score | Next step |
|---|---|
| 22–25 | Ready for Domain 4. Revisit any you guessed. |
| 17–21 | Re-read the files linked from your wrong answers. |
| < 17 | Re-read files 00–06 in order, then try again in a few days. |

---

## Key Takeaways

- **Precision**: explicit categories, skip lists, severity examples; disable noisy categories.
- **Consistency**: 2–4 targeted **few-shot** examples with reasoning.
- **Structure**: tool use + schema; **nullable** fields; enums with `unclear` / `other`; normalization rules.
- **Correctness**: semantic validation, **retry with specific errors**, know when retry can't help, self-check fields, `detected_pattern`.
- **Scale**: batch for **non-blocking** work; SLA math; resubmit failures by `custom_id`; test on a sample first.
- **Review**: **independent instances**, **multi-pass**, **calibrated confidence routing**.
