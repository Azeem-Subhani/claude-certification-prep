# 06 · Task Decomposition Strategies

**Exam link:** Task Statement 1.6: *Design task decomposition strategies for complex workflows.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (workflow vs agent, attention dilution), [02](02-multi-agent-orchestration.md).
**Scope note:** [02](02-multi-agent-orchestration.md) covers *how a coordinator splits work across agents*. This file covers *which decomposition pattern to choose* for a task, whether one agent or many does the work.

---

## 1. What is task decomposition?

**Task decomposition** means splitting a large task into smaller steps that are easier to do well. A model given one huge instruction ("review this 40-file pull request") often:

- misses details (attention dilution),
- gives uneven quality (deep on the first files, shallow on the last),
- mixes different kinds of thinking (local bugs vs system design) in one pass.

Smaller, focused steps fix this.

There are two main strategies. Choosing the right one is what the exam tests.

---

## 2. Strategy A: Fixed sequential pipeline (prompt chaining)

**Prompt chaining** is a fixed series of model calls. Each step has one job, and its output feeds the next step. The **code** decides the steps in advance.

```
input ─▶ Step 1 ─▶ Step 2 ─▶ Step 3 ─▶ output
         (extract)  (analyze)  (format)
```

**Use it when:**

- The task is **predictable**: you know the steps before you start.
- The same steps apply to every input.
- You want easy testing: you can check each step on its own.

**Examples:**

- Contract review: extract clauses → compare each clause with policy → write a risk summary.
- Content pipeline: draft → fact-check → edit for style → translate.
- Multi-aspect code review (see §4).

**Strengths:** predictable, cheap, easy to debug, and you can add validation between steps ("gates").
**Weakness:** cannot adapt. If step 2 discovers something unexpected, the pipeline does not change.

---

## 3. Strategy B: Dynamic adaptive decomposition

In **dynamic decomposition**, the subtasks are **generated as you go**, based on what each step discovers. The plan changes during the work.

```
explore ─▶ discover X ─▶ new subtask for X ─▶ discover Y ─▶ new subtask for Y ─▶ …
```

**Use it when:**

- The task is **open-ended**: you can't know the steps in advance.
- What you find early changes what you should do next.

**Examples:**

- "Find out why our checkout page got slow last week."
- "Add comprehensive tests to this legacy codebase."
- "Investigate this security alert."

**Strengths:** handles the unknown; effort goes where it matters.
**Weakness:** less predictable cost and time; needs good stopping criteria.

---

## 4. Choosing the pattern

| Question | If yes → |
|---|---|
| Do I know all the steps before I start? | Prompt chaining |
| Are the same steps used for every input? | Prompt chaining |
| Will early findings change later steps? | Dynamic decomposition |
| Is the goal "investigate" / "explore" / "figure out"? | Dynamic decomposition |

Many real systems **combine** them: a fixed outer pipeline with one adaptive step inside, or an adaptive plan whose individual steps are fixed chains.

---

## 5. Pattern: per-file passes plus a cross-file integration pass

This is a classic exam example of prompt chaining for **large code reviews**.

**Problem:** A pull request changes 14 files. A single review prompt with all 14 files gives:

- detailed comments on the first few files and shallow ones on the rest,
- missed bugs,
- sometimes contradictory comments (flagging a pattern in one file but approving it in another).

**Solution: split by type of thinking.**

```
           ┌─ Pass 1a: review file 1 (local issues) ─┐
           ├─ Pass 1b: review file 2 (local issues) ─┤
 PR ──────▶├─ ...                                    ├──▶ Pass 2: cross-file integration ──▶ report
           └─ Pass 1n: review file 14 (local issues) ┘     (interfaces, data flow,
                (can run in parallel)                       consistency between files)
```

1. **Local passes:** one focused pass per file. Looks for bugs, style, and logic errors *inside* that file.
2. **Integration pass:** one separate pass that looks at how files work *together*: changed function signatures, data passed between modules, consistent error handling, missing updates in callers.

**Why this works:** each pass has small, focused context, so there is no attention dilution. The integration pass catches problems that no single-file pass can see.

```python
def review_pull_request(files: dict[str, str]) -> str:
    # Pass 1: focused local review per file (independent, so they can run in parallel)
    local_findings = {path: review_single_file(path, code) for path, code in files.items()}

    # Pass 2: integration review gets summaries + diffs of interfaces, not the full raw context again
    return review_cross_file(
        file_summaries=local_findings,
        interface_changes=extract_interface_changes(files),
    )
```

> **Exam tip:** If an answer says "use a bigger context window" or "just write a stricter prompt" for this problem, it doesn't fix attention dilution. Splitting the work does.

---

## 6. Pattern: adaptive investigation plan for open-ended tasks

**Task:** *"Add comprehensive tests to this legacy codebase."* You can't list the subtasks in advance because you don't yet know the codebase.

**Approach:**

1. **Map the structure**: modules, entry points, dependencies, existing tests.
2. **Identify high-impact areas**: code that is critical, changes often, or has no tests (e.g. payment logic).
3. **Create a prioritized plan** from what you found.
4. **Adapt the plan as you discover things**: for example, "module A depends on an untested database helper, so test the helper first."

```
Map ──▶ "payments/ has 0 tests; billing.py imports legacy_db.py"
  │
  ▼
Plan v1: 1) payments  2) billing  3) reports
  │
  ▼ (while testing payments, discover it relies on legacy_db.py)
Plan v2: 1) legacy_db helper  2) payments  3) billing  4) reports
```

The plan is a **living document**, not a fixed list.

---

## 7. Real-world case study

**Situation:** An engineering team uses Claude to review pull requests in CI. Large PRs (10+ files) got poor reviews: later files were barely reviewed, and a renamed function broke three callers without anyone noticing.

**Change:**

- Small PRs (1–3 files): single pass (simple and cheap).
- Large PRs: **per-file local passes in parallel** + **one cross-file integration pass** that receives interface changes and per-file summaries.

**Result:** Review depth became even across files, and the integration pass now reliably catches signature and contract changes across modules.

**Second situation:** The same team asked Claude to "improve test coverage" across a legacy service. A fixed plan ("write tests for every file alphabetically") wasted effort on trivial files. Switching to an **adaptive plan** (map → prioritize by risk → adjust as dependencies appear) put effort on the most important code first.

---

## Common Q&A

**Q: Is dynamic decomposition always better because it's more flexible?**
A: No. For predictable tasks, prompt chaining is cheaper, more consistent, and easier to test. Use dynamic decomposition only when the steps truly depend on discoveries.

**Q: Why not review all files in one pass with a model that has a huge context window?**
A: A larger window lets you fit more text, but quality per item still drops when attention is spread thin. Focused passes give more consistent depth.

**Q: What does the integration pass need as input?**
A: Enough to reason across files: interface and signature changes, data flow, and summaries of local findings. It doesn't need every line of every file again.

**Q: How is task decomposition different from multi-agent orchestration?**
A: Decomposition is *how you split the work*. Orchestration is *who does each piece* (one agent in steps, or several agents). You can decompose a task and still run it with one agent.

**Q: In an adaptive plan, when do you stop?**
A: When the goal's success criteria are met (e.g. all high-priority areas have tests), with a budget or round limit as a safety net.

---

## Key Takeaways

- **Prompt chaining** = fixed steps, for **predictable** tasks. **Dynamic decomposition** = steps generated from findings, for **open-ended** tasks.
- Choose based on whether you **know the steps in advance**.
- For large reviews, use **per-file local passes + a separate cross-file integration pass** to avoid attention dilution.
- For open-ended tasks: **map → find high-impact areas → prioritized plan → adapt as you learn**.
- A bigger context window does **not** replace good decomposition.
