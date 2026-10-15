# 00 · Foundations and Glossary (Domain 4)

> **Read this first.** This file explains the quality ideas behind Domain 4: precision, false positives, and the difference between output that is *well-formed* and output that is *correct*.
> Earlier domains define agents ([Domain 1](../domain-1-agentic-architecture-orchestration/00-foundations-and-glossary.md)), tools and `tool_choice` ([Domain 2](../domain-2-tool-design-mcp-integration/00-foundations-and-glossary.md)), and Claude Code CI usage ([Domain 3](../domain-3-claude-code-configuration-workflows/06-ci-cd-integration.md)).

**Exam link:** Background for all of Domain 4 (20% of the exam).

---

## 1. What this domain is about

Domain 4 is about getting **reliable, usable output** from Claude in production systems, especially two kinds of systems:

- **Review systems**, e.g. automated code review in CI: the output is a list of **findings**.
- **Extraction systems**, e.g. pulling fields from invoices, contracts, or research papers: the output is **structured data**.

Both share the same goals:

| Goal | Techniques (file) |
|---|---|
| Report the **right** things, not noise | Explicit criteria ([01](01-explicit-criteria-precision.md)), few-shot examples ([02](02-few-shot-prompting.md)) |
| Output in a **consistent, machine-readable** shape | Few-shot ([02](02-few-shot-prompting.md)), tool use + JSON schemas ([03](03-structured-output-tool-use-schemas.md)) |
| Catch and fix **mistakes** | Validation and retry ([04](04-validation-retry-feedback-loops.md)), independent review ([06](06-multi-instance-multi-pass-review.md)) |
| Run at **scale and low cost** | Batch processing ([05](05-batch-processing.md)) |

---

## 2. Precision, recall, and false positives

When a system flags issues, each flag is either right or wrong:

| | Issue really exists | Issue does not exist |
|---|---|---|
| **System flags it** | ✅ True positive | ❌ **False positive** (noise) |
| **System doesn't flag it** | ❌ False negative (missed) | ✅ True negative |

- **Precision** = of everything flagged, how much is real? (High precision = few false positives.)
- **Recall** = of all real issues, how many were flagged? (High recall = few misses.)

**Example:** A code reviewer posts 20 comments; 5 are real bugs.
→ Precision = 5/20 = **25%**. Developers will quickly learn to ignore it.

### Why false positives matter so much

Developers have limited attention. If many findings are wrong, they **stop trusting all findings**, including the correct ones. One noisy category (e.g. style nitpicks) can make people ignore the accurate security findings sitting next to them.

> **Domain 4 mindset:** In review systems, **trust is the product**. Improving precision often matters more than catching one more minor issue.

---

## 3. Well-formed vs correct output

Two different kinds of errors appear in structured output:

| Error type | Example | Fixed by |
|---|---|---|
| **Syntax / schema error** | Invalid JSON, missing required field, wrong type (`"total": "abc"`) | **Tool use / structured outputs with a JSON schema** ([03](03-structured-output-tool-use-schemas.md)) |
| **Semantic error** | Valid JSON, but line items don't add up to the total; the due date is placed in the invoice-date field; a value is invented | **Validation logic, retries with feedback, review** ([04](04-validation-retry-feedback-loops.md)) |

> A schema guarantees the **shape** of the data, not its **truth**.

---

## 4. Prompt building blocks used in this domain

| Block | Purpose | Example |
|---|---|---|
| **Task + context** | What to do and why | "Review this diff for issues that could cause incidents in production." |
| **Explicit criteria** | Exactly what counts and what doesn't | "Report: SQL injection, missing auth checks. Skip: naming, formatting." |
| **Few-shot examples** | Show desired behavior and format | 2–4 input → output pairs |
| **Output format / schema** | Make output machine-readable | A tool with a JSON schema |
| **Normalization rules** | Make values consistent | "Dates as YYYY-MM-DD; currency as ISO code." |

---

## 5. The six Domain 4 topics

| File | Task statement | One-line summary |
|---|---|---|
| [01](01-explicit-criteria-precision.md) | 4.1 Explicit criteria | Say exactly what to report and what to skip |
| [02](02-few-shot-prompting.md) | 4.2 Few-shot prompting | Show 2–4 examples for format and hard cases |
| [03](03-structured-output-tool-use-schemas.md) | 4.3 Structured output | Use tools/schemas; design fields so the model never has to invent |
| [04](04-validation-retry-feedback-loops.md) | 4.4 Validation and retry | Check meaning, retry with specific errors, learn from dismissals |
| [05](05-batch-processing.md) | 4.5 Batch processing | Use the Batches API for cheap, non-urgent volume work |
| [06](06-multi-instance-multi-pass-review.md) | 4.6 Review architectures | Independent reviewers and focused passes |
| [MCQ bank](../mcq-bank/04-prompt-engineering-structured-output.md) | All | 48 exam-level practice questions |

---

## 6. Glossary

| Term | Meaning |
|---|---|
| **Batch (Message Batches API)** | Asynchronous API that processes many requests at 50% cost, within up to 24 hours |
| **Calibration** | How well a model's stated confidence matches how often it is actually right |
| **`custom_id`** | Your identifier on each batch request, used to match results to requests |
| **`detected_pattern`** | A field in a finding that records which code construct triggered it, for later analysis |
| **Enum** | A schema field limited to a fixed list of values (e.g. `low`, `medium`, `high`) |
| **Explicit criteria** | Concrete, categorical rules about what to report or skip |
| **Extraction** | Pulling structured fields out of unstructured text or documents |
| **False positive** | A flagged issue that isn't real |
| **Few-shot prompting** | Including a few worked examples in the prompt |
| **Hallucination** | The model stating or inventing information not supported by the input |
| **Multi-pass review** | Splitting a review into several focused passes (e.g. per file, then cross-file) |
| **Nullable field** | A schema field allowed to be `null` when the information isn't available |
| **Precision / recall** | Share of flags that are real / share of real issues that are flagged |
| **Retry with error feedback** | Re-asking with the specific validation errors included |
| **Self-review** | The same session/instance reviewing its own output |
| **Semantic error** | Output with the right shape but wrong meaning |
| **SLA** | Service level agreement: a promised maximum time to deliver results |
| **Synchronous API** | Normal request → immediate response (Messages API) |

---

## Common Q&A

**Q: Is precision or recall more important?**
A: It depends on the use case. For developer-facing review tools, **precision** is usually critical, because noise destroys trust. For safety-critical screening, recall may matter more. The exam's review scenarios focus on reducing false positives.

**Q: If I use a JSON schema, is my extracted data correct?**
A: No. A schema guarantees valid structure. You still need **semantic validation** (totals, dates, field placement).

**Q: What's the difference between few-shot examples and instructions?**
A: Instructions **describe** what you want; few-shot examples **show** it. Showing is usually clearer, especially for format and ambiguous judgment.

---

## Key Takeaways

- Domain 4 = **reliable output** for **review** and **extraction** systems.
- **Precision** (few false positives) is key to developer **trust**.
- **Schemas fix syntax; validation fixes meaning.**
- Prompt building blocks: task, **explicit criteria**, **examples**, **schema**, **normalization rules**.
