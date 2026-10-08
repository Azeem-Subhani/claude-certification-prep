# 06 · Multi-Instance and Multi-Pass Review Architectures

**Exam link:** Task Statement 4.6: *Design multi-instance and multi-pass review architectures.*
**Prerequisites:** Domain 1 [06 · Task Decomposition](../domain-1-agentic-architecture-orchestration/06-task-decomposition.md) (per-file + cross-file passes), Domain 3 [06 · CI/CD](../domain-3-claude-code-configuration-workflows/06-ci-cd-integration.md) (independent reviewer in CI).
**Scope note:** Those files introduced per-file passes and the independent CI reviewer. This file explains **why self-review fails** (and why extended thinking doesn't fix it), how to **combine** instances and passes into a review architecture, and how to use **confidence for routing**.

---

## 1. Why self-review is weak

When a model generates code (or an extraction, or a plan), its **reasoning stays in the context**. If you then ask the **same session** to "review your work":

- It **reads its own reasoning** and finds it convincing, since it produced it.
- It tends to **confirm** its earlier decisions instead of questioning them.
- Mistakes that came from a **wrong assumption** are invisible, because the reviewer shares the same assumption.

This is like asking an author to proofread their own essay immediately after writing it: they read what they *meant*, not what they *wrote*.

### Why "review carefully" or extended thinking doesn't solve it

| Attempt | Why it falls short |
|---|---|
| "Now carefully review your code for bugs" | Same context, same assumptions, same blind spots |
| Enabling **extended thinking** for the self-review | More reasoning **inside the same context**, still anchored to the original reasoning |
| Asking it to "pretend to be a different reviewer" | Still sees and trusts its own earlier reasoning |

> **Exam point:** **Independent review instances**, which have **no prior reasoning context**, are **more effective** at catching subtle issues than self-review instructions or extended thinking.

---

## 2. Independent review instances

An **independent instance** is a **new** Claude call/session that receives only:

- the **artifact** to review (code diff, extraction result, document),
- the **requirements / criteria** (from [01](01-explicit-criteria-precision.md)),
- **relevant context** (e.g. related files, spec),

…but **not** the generator's conversation or reasoning.

```
Generator instance                         Reviewer instance (fresh)
──────────────────                         ─────────────────────────
spec + codebase ──▶ writes code ──▶ diff ──▶ diff + spec + review criteria ──▶ findings
     (reasoning stays here)                 (no access to generator reasoning)
```

Because it judges the result **on its own merits**, it notices when the code doesn't match the spec, even when the generator's reasoning "explained" why it should.

**Same idea for extraction:** a second instance can verify an extraction against the source document without seeing how the first instance reasoned.

---

## 3. Multi-pass review for large changes

(The per-file + cross-file split is explained in Domain 1 · [06](../domain-1-agentic-architecture-orchestration/06-task-decomposition.md). Here it becomes part of a review architecture.)

For a large multi-file change, combine **focused passes**:

| Pass | Input | Looks for |
|---|---|---|
| **Per-file local passes** (one per file, can run in parallel) | One file's diff + its immediate context | Bugs, security, and logic issues **inside** that file |
| **Cross-file integration pass(es)** | Interface changes, data flow between modules, summaries of local findings | Broken contracts, **data-flow** problems across files, inconsistent handling |

**Why:** One giant pass suffers **attention dilution** (shallow review of later files) and produces **contradictory findings** (approving a pattern in one file, flagging it in another). Focused passes give even depth and consistent results.

---

## 4. Confidence calibration for review routing

Not every finding needs the same handling. A **verification pass** can attach a **self-reported confidence** to each finding, and you can use it to **route** findings:

```json
{
  "location": "src/payments/refund.py:73",
  "issue": "Refund amount not checked against original charge; over-refund possible",
  "severity": "critical",
  "confidence": 0.92,
  "evidence": "Line 73 passes request.amount directly to gateway.refund()"
}
```

| Confidence | Routing |
|---|---|
| High (e.g. ≥ 0.85) | Post directly as a PR comment |
| Medium | Send to a human reviewer queue, or a second independent verification |
| Low | Log only, for analysis |

### Important: calibrate before you trust it

Self-reported confidence is only useful if it is **calibrated**: when the model says 0.9, it should be right about 90% of the time. Before using thresholds:

1. Collect findings with confidence scores on a **labeled sample**.
2. Measure actual accuracy per confidence band.
3. **Set thresholds from the data**, not from intuition.
4. Re-check periodically.

> **How this fits with [01](01-explicit-criteria-precision.md):** "Only report high-confidence findings" in the prompt does **not** fix precision by itself. Confidence works as a **routing signal** inside a designed pipeline, **after** explicit criteria define what to look for, and once it has been **calibrated** against real outcomes.

---

## 5. Putting it together: a review architecture

```
                  ┌─ per-file pass (file 1) ─┐
PR diff ─────────▶├─ per-file pass (file 2) ─┤──▶ cross-file integration pass ──▶ candidate findings
(independent      └─ per-file pass (file n) ─┘
 instances, no
 generator context)
                                                          │
                                                          ▼
                                          verification pass (independent instance):
                                          re-check each finding + confidence + evidence
                                                          │
                                   ┌──────────────────────┼─────────────────────┐
                                   ▼                      ▼                     ▼
                              high: post            medium: human          low: log only
```

Every stage uses **explicit criteria** ([01](01-explicit-criteria-precision.md)), **few-shot examples** for format ([02](02-few-shot-prompting.md)), and **structured output** ([03](03-structured-output-tool-use-schemas.md)).

---

## 6. Real-world case study

**Situation:** A team used Claude to generate data-migration scripts and then asked the **same session** to review them. Reviews almost never found problems, yet two migrations failed in staging (one dropped rows with NULL keys; one double-counted records).

**Attempt:** Enabled extended thinking for the self-review. Reviews became longer, but still missed both bug types.

**Fix:**
1. Review by a **separate instance** that sees only the migration script, the schema, and the criteria. It flagged the NULL-key filter on the first run.
2. For large migrations touching many tables: **per-table passes** + an **integration pass** for foreign-key and data-flow consistency.
3. A **verification pass** adds confidence; after calibrating on 300 labeled findings, ≥ 0.85 is auto-posted, 0.5–0.85 goes to a DBA, and below that is logged.

---

## Common Q&A

**Q: Why is self-review in the same session ineffective?**
A: The model keeps its generation reasoning in context, so it tends to confirm its earlier decisions instead of questioning them.

**Q: Doesn't extended thinking fix that?**
A: Not reliably. More thinking happens in the same context, anchored to the same assumptions. An **independent instance** is more effective.

**Q: What should an independent reviewer receive?**
A: The artifact, the requirements/criteria, and relevant context, but **not** the generator's reasoning or conversation.

**Q: Why split a big review into passes?**
A: To avoid attention dilution and contradictory findings: focused per-file passes for local issues, plus integration passes for cross-file data flow.

**Q: How should confidence scores be used?**
A: As a **routing** signal (post / human review / log), **after** calibrating thresholds against labeled outcomes. Not as a substitute for explicit criteria.

---

## Key Takeaways

- **Self-review is limited**: the same context keeps the same assumptions.
- **Independent instances** (no generator reasoning) catch subtle issues better than self-review **or** extended thinking.
- **Multi-pass**: per-file local passes + cross-file integration passes.
- **Verification passes with confidence** enable **routing**, but only after **calibration** on real data.
- Build the architecture on **explicit criteria, few-shot format, and structured output**.
