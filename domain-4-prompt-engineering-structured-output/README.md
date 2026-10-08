# Domain 4: Prompt Engineering & Structured Output

**Exam weight:** 20%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

## Exam scenarios where this domain is primary

- Scenario 5: Claude Code for Continuous Integration
- Scenario 6: Structured Data Extraction

## Related guide material

- Preparation Exercise 3: Build a Structured Data Extraction Pipeline (schemas, validation-retry, batch strategy)
- Sample Questions 11 (batch vs synchronous API), 12 (multi-pass review of a large PR)

## Study notes (read in order)

| File | Covers |
| --- | --- |
| [00 · Foundations and Glossary](00-foundations-and-glossary.md) | Precision vs recall, false positives, syntax vs semantic errors, key terms |
| [01 · Explicit Criteria for Precision](01-explicit-criteria-precision.md) | 4.1: categorical criteria, disabling noisy categories, severity definitions |
| [02 · Few-Shot Prompting](02-few-shot-prompting.md) | 4.2: 2–4 targeted examples, ambiguous cases, format, extraction from varied documents |
| [03 · Structured Output with Tools and Schemas](03-structured-output-tool-use-schemas.md) | 4.3: extraction tools, `tool_choice` for extraction, nullable fields, enums with "other" |
| [04 · Validation, Retry, and Feedback Loops](04-validation-retry-feedback-loops.md) | 4.4: retry with error feedback, when retries can't help, `detected_pattern`, self-checking fields |
| [05 · Batch Processing](05-batch-processing.md) | 4.5: Message Batches API, latency fit, SLA math, `custom_id` resubmission |
| [06 · Multi-Instance and Multi-Pass Review](06-multi-instance-multi-pass-review.md) | 4.6: self-review limits, independent reviewers, confidence calibration for routing |
| [07 · Practice Questions](07-scenario-practice-questions.md) | 25 original scenario questions with explanations |

Sources: the exam guide PDF, plus Anthropic docs: [Batch processing](https://platform.claude.com/docs/en/build-with-claude/batch-processing) and [Structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs). Where the guide and current API behavior differ (forced `tool_choice` on the newest models), the notes give both.

---

## 4.1 Design prompts with explicit criteria to improve precision and reduce false positives

**Know**
- [ ] Explicit criteria beat vague instructions ("flag comments only when claimed behavior contradicts actual code" vs "check comments are accurate")
- [ ] "Be conservative" / "only high-confidence findings" don't improve precision as well as specific categorical criteria
- [ ] High false-positive categories destroy developer trust in accurate categories too

**Do**
- [ ] Define which issues to report (bugs, security) and which to skip (minor style, local patterns), instead of confidence filtering
- [ ] Temporarily disable high false-positive categories to restore trust while improving their prompts
- [ ] Define severity levels with concrete code examples for consistent classification

## 4.2 Apply few-shot prompting to improve output consistency and quality

**Know**
- [ ] Few-shot examples are the most effective way to get consistent, actionable output when instructions alone are inconsistent
- [ ] Examples show how to handle ambiguous cases (tool choice for ambiguous requests, branch-level coverage gaps)
- [ ] Examples help the model generalize judgment to new patterns, not just match listed cases
- [ ] Examples reduce hallucination in extraction (informal measurements, varied document structures)

**Do**
- [ ] Write 2–4 targeted examples for ambiguous scenarios, showing why one action beats plausible alternatives
- [ ] Use examples to fix output format (location, issue, severity, suggested fix)
- [ ] Use examples that separate acceptable patterns from real issues
- [ ] Show correct handling of varied document structures (inline citations vs bibliographies, etc.)
- [ ] Show correct extraction from varied formats, including empty/null required fields

## 4.3 Enforce structured output using tool use and JSON schemas

**Know**
- [ ] Tool use with a JSON schema is the most reliable way to get schema-compliant output, eliminating JSON syntax errors
- [ ] `tool_choice`: `"auto"` (may return text), `"any"` (must call some tool), forced (must call a named tool)
- [ ] Strict schemas remove syntax errors but **not** semantic errors (totals that don't add up, values in the wrong field)
- [ ] Schema design: required vs optional, enums with `"other"` + detail string for extensible categories

**Do**
- [ ] Define extraction tools with JSON schemas as input parameters; read the data from `tool_use`
- [ ] Use `tool_choice: "any"` when several extraction schemas exist and the document type is unknown
- [ ] Force a specific tool (e.g. `extract_metadata`) to run before enrichment steps
- [ ] Make fields optional/nullable when the source may not contain the data, so the model doesn't invent values
- [ ] Add enum values like `"unclear"` and `"other"` + detail fields
- [ ] Include format normalization rules in prompts alongside strict schemas

## 4.4 Implement validation, retry, and feedback loops for extraction quality

**Know**
- [ ] Retry with error feedback: add the specific validation errors to the retry prompt
- [ ] Retries don't help when the information isn't in the source document (vs format/structure errors)
- [ ] Track which code constructs trigger findings (`detected_pattern`) to analyze dismissals
- [ ] Semantic validation errors differ from schema syntax errors (which tool use eliminates)

**Do**
- [ ] On retry, send the original document, the failed extraction, and the specific errors
- [ ] Recognize when retries won't work (info is in an external document) vs when they will (format mismatch, structural errors)
- [ ] Add `detected_pattern` to findings to analyze false-positive patterns
- [ ] Design self-checks: extract `calculated_total` alongside `stated_total`; add `conflict_detected` booleans

## 4.5 Design efficient batch processing strategies

**Know**
- [ ] Message Batches API: 50% cost savings, up to 24-hour processing, no guaranteed latency SLA
- [ ] Good for non-blocking, latency-tolerant work (overnight reports, weekly audits, nightly test generation); bad for blocking work (pre-merge checks)
- [ ] A batch request can't run your tools mid-request and continue (no multi-turn tool calling inside one request)
- [ ] `custom_id` links each request to its result

**Do**
- [ ] Synchronous API for blocking pre-merge checks; batch API for overnight/weekly analysis
- [ ] Calculate submission frequency from SLA (e.g. submit every 4 hours to guarantee a 30-hour SLA with 24-hour processing)
- [ ] Resubmit only failed items (by `custom_id`) with fixes (e.g. chunk documents that were too long)
- [ ] Refine prompts on a sample before batch-processing large volumes

## 4.6 Design multi-instance and multi-pass review architectures

**Know**
- [ ] Self-review is limited: the model keeps its generation reasoning and is less likely to question itself
- [ ] Independent review instances catch subtle issues better than self-review instructions or extended thinking
- [ ] Multi-pass review: per-file local passes + cross-file integration passes avoid attention dilution and contradictions

**Do**
- [ ] Use a second, independent instance (without the generator's reasoning) to review generated code
- [ ] Split large multi-file reviews into per-file passes + separate integration passes for cross-file data flow
- [ ] Run verification passes where the model reports confidence per finding, for calibrated review routing

---

## Key themes to remember

- **Specific beats general:** categorical criteria, not "be careful".
- **Show beats tell:** few-shot examples for format and ambiguous judgment.
- **Schemas fix syntax, not truth:** validate semantics separately.
- **Retry only fixes what's fixable:** format yes, missing information no.
- **Batch = cheap and slow:** never for blocking workflows.
- **Fresh eyes:** independent instances and focused passes beat self-review.

## My notes

<!-- Add study notes, lab links, and weak areas here -->
