# Domain 4 Anti-patterns: Prompt Engineering & Structured Output

**Scope:** wrong choices for review criteria, few-shot examples, schemas, validation and retry, batch processing, and review design (task statements 4.1 to 4.6).

---

## Anti-patterns

| # | Wrong approach | Why it fails | Do this instead | Topic |
|---|---|---|---|---|
| 1 | Tell the reviewer to "be conservative" or "only report high-confidence findings". | These words do not say **what** to report. Precision does not improve. | Write categorical criteria: what to report (bugs, security) and what to skip (style, local patterns). | [01](01-explicit-criteria-precision.md) |
| 2 | Use a confidence threshold to cut false positives. | Scores without clear criteria do not fix the cause. Real issues can be filtered out too. | Define explicit criteria first. Use confidence only after calibration. | [01](01-explicit-criteria-precision.md) |
| 3 | Keep a noisy category active while you fix it. | Developers keep seeing false alarms and start ignoring all findings. | Turn the noisy category off, fix it with criteria and examples, then turn it back on. | [01](01-explicit-criteria-precision.md) |
| 4 | Define severity levels in prose only. | Claude classifies the same issue in different ways. | Give a short code example for each severity level. | [01](01-explicit-criteria-precision.md) |
| 5 | Add more detailed instructions after instructions already failed. | More of the same input gives the same inconsistent output. | Add 2 to 4 targeted few-shot examples. | [02](02-few-shot-prompting.md) |
| 6 | Show only input and output pairs, without reasoning. | Claude copies the surface of the examples instead of learning the judgment. | Show why one option was chosen over a plausible alternative. | [02](02-few-shot-prompting.md) |
| 7 | Give examples from only one document layout. | Fields come back empty when the data sits in a different place, such as a paragraph. | Add examples from varied layouts: tables, narrative text, citations. | [02](02-few-shot-prompting.md) |
| 8 | Ask for JSON in the text reply and parse it. | The JSON can be broken or incomplete. | Use tool use with a JSON schema (or structured outputs). | [03](03-structured-output-tool-use-schemas.md) |
| 9 | Believe a schema stops all extraction errors. | A schema checks shape only. Totals can be wrong and values can sit in the wrong field. | Add semantic validation in code, plus retry. | [03](03-structured-output-tool-use-schemas.md) |
| 10 | Make every schema field required. | When the data is missing, Claude invents a value to fill the field. | Make fields optional or nullable when the source may not contain them. | [03](03-structured-output-tool-use-schemas.md) |
| 11 | Force every value into a fixed category list. | Ambiguous or new cases get a wrong label. | Add `"unclear"` for ambiguous cases, and `"other"` plus a detail field for new ones. | [03](03-structured-output-tool-use-schemas.md) |
| 12 | Mix up `tool_choice` `"auto"` and `"any"`. | `"auto"` lets Claude answer in text. `"any"` forces a tool call. | Use `"any"` (exam answer) when you must get structured output. ⚠️ Newest models reject it. | [03](03-structured-output-tool-use-schemas.md) |
| 13 | Retry with a generic "try again". | Claude does not know what was wrong, so it repeats the error. | Send the original document, the failed output, and the specific validation error. | [04](04-validation-retry-feedback-loops.md) |
| 14 | Retry when the information is not in the document. | No retry can create missing data. It only wastes calls or invites invented values. | Return null, or send the item to human review. | [04](04-validation-retry-feedback-loops.md) |
| 15 | Pick one value when a document contradicts itself. | The conflict is hidden, and the chosen value may be wrong. | Extract both values and set `conflict_detected: true`. | [04](04-validation-retry-feedback-loops.md) |
| 16 | Drop Pydantic because tool use already checks the schema. | Tool use checks shape, not cross-field rules like totals. | Keep validators for meaning, and feed their errors into the retry. | [04](04-validation-retry-feedback-loops.md) |
| 17 | Move all work to the Batch API to save 50%. | Batch can take up to 24 hours, with no speed guarantee. Blocking work would stall. | Use batch only for work nobody waits for (overnight, weekly). | [05](05-batch-processing.md) |
| 18 | Plan around batches "usually finishing in an hour". | There is no guarantee. Requests not done after 24 hours expire. | Plan for 24 hours, and compute the submit interval from the SLA. | [05](05-batch-processing.md) |
| 19 | Resubmit the whole batch after a few items fail. | It costs money and time for items that already worked. | Find the failed items by `custom_id`, fix them, and resubmit only those. | [05](05-batch-processing.md) |
| 20 | Use a batch request for a multi-turn tool loop. | A batch request cannot pause, wait for your tool, and continue. | Use the synchronous API for steps that need your tools mid-process. | [05](05-batch-processing.md) |
| 21 | Send thousands of documents before testing the prompt. | Prompt flaws repeat across the whole batch. | Refine the prompt on a small sample first. | [05](05-batch-processing.md) |
| 22 | Ask the same session to "review carefully". | It keeps its own reasoning and agrees with itself. | Use a second, independent instance without the generation history. | [06](06-multi-instance-multi-pass-review.md) |
| 23 | Review many files in one pass, or switch to a bigger context window. | Attention dilution: some files get little care, and findings contradict each other. | One pass per file, plus a cross-file integration pass. | [06](06-multi-instance-multi-pass-review.md) |
| 24 | Route findings by raw, uncalibrated confidence scores. | You do not know what a score of 0.9 really means. | Calibrate on a labeled set, then set thresholds. | [06](06-multi-instance-multi-pass-review.md) |

---

## Common exam traps

1. **"Tell Claude to be conservative."** Vague words do not define what to report. Explicit categories do.
2. **"Raise the confidence threshold to remove false positives."** This treats the symptom, not the cause. Fix the criteria first.
3. **"Add more detailed instructions."** Instructions already failed. Examples beat more instructions.
4. **"Tool use with a schema guarantees correct data."** It guarantees valid syntax and shape only. Wrong totals still need validation.
5. **"Make every field required for completeness."** Required fields push Claude to invent values. Use nullable fields.
6. **"Retry until the missing field appears."** Retries cannot create data that is not in the source.
7. **"Use `\"auto\"` so Claude can choose."** If you must always get structured output, `"auto"` is wrong because Claude may answer in text.
8. **"Move both the pre-merge check and the nightly report to batch."** Only the nightly report can wait. Blocking checks need real-time calls.
9. **"Batches usually finish in an hour, so submit hourly for a 2-hour SLA."** Batch can take up to 24 hours. Plan for the worst case.
10. **"Resubmit the full batch."** Resubmit only the failed `custom_id`s, with fixes.
11. **"Use extended thinking so the same session reviews itself better."** The bias comes from shared context. An independent instance is the fix.
12. **"Use a larger context window so one pass covers all files."** Attention dilution remains. Split into focused passes.

> ⚠️ See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) items R3 (`tool_choice` on the newest models) and R9 (tools in batch requests).
