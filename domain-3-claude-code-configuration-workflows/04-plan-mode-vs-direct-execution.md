# 04 · Plan Mode vs Direct Execution

**Exam link:** Task Statement 3.4: *Determine when to use plan mode vs direct execution.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (working modes), Domain 1 [06 · Task Decomposition](../domain-1-agentic-architecture-orchestration/06-task-decomposition.md).

---

## 1. Two ways to work

| | **Plan mode** | **Direct execution** |
|---|---|---|
| What Claude does | **Explores** the code and **writes a plan**. It does **not** edit files until you approve. | **Reads, edits, and runs** things straight away to finish the task. |
| Your role | Review and approve (or change) the plan | Review the result |
| Cost of a wrong direction | Low: only a plan is thrown away | High: changes must be undone (**rework**) |
| Speed for simple tasks | Slower (extra step) | Faster |

In Claude Code you can switch into plan mode during a session (for example with **Shift+Tab** to cycle permission modes) or start in it with `--permission-mode plan`.

---

## 2. When to use plan mode

Use plan mode when **getting the approach wrong is expensive**:

| Signal | Example |
|---|---|
| **Large-scale changes** | Migrating a logging library used in **45+ files** |
| **Architectural decisions** | Restructuring a monolith into **microservices**: where are the service boundaries? |
| **Multiple valid approaches** | Integrating payments via webhooks **or** polling, each with different infrastructure needs |
| **Many files / modules affected** | Changing an interface used across several packages |
| **Unfamiliar code** | First change in a legacy system nobody fully understands |

**Why it helps:** Plan mode lets Claude **safely explore** the codebase and **design** the change **before** committing to edits. You catch wrong assumptions (a missed dependency, a bad boundary) while they're cheap to fix, not after 30 files have been changed.

---

## 3. When to use direct execution

Use direct execution for **simple, well-scoped** changes where the right fix is clear:

| Example | Why direct is fine |
|---|---|
| Fix a bug in **one file** with a **clear stack trace** | Location and cause are known |
| Add **one validation check** to a function (e.g. "end date must be after start date") | Small, contained, obvious approach |
| Rename a local variable | Trivial |
| Update a dependency version and run the tests | Clear steps |

Using plan mode here adds an extra review step with little benefit.

> **Exam pattern:** Look at **scope** and **uncertainty**. Big scope or several valid designs → **plan mode**. Small scope and an obvious fix → **direct execution**.

---

## 4. Combine them: plan, then execute

The two modes work best **together** for big tasks:

```
1. Plan mode:        explore, compare approaches, write a step-by-step plan
2. Human review:     approve or adjust the plan
3. Direct execution: implement the approved plan, step by step, running tests
```

**Example: library migration**
1. **Plan mode:** Find all 47 usages of the old HTTP client, group them by pattern (simple GET, retries, streaming), propose a migration order and a compatibility wrapper.
2. **Review:** The team agrees to migrate the streaming cases last.
3. **Direct execution:** Implement batch by batch, running tests after each batch.

---

## 5. The Explore subagent: keep discovery out of your main context

Big tasks start with **discovery**: searching, reading many files, listing usages. This produces **verbose output** that fills the main conversation. In a multi-phase task, the context can run out **before you even start implementing**.

The **Explore subagent** is a built-in, read-only subagent for discovery:

- It searches and reads in **its own context**.
- It returns a **summary** to the main conversation.
- Your main context stays free for planning and implementation.

```
Main session: "Use the Explore agent to find every place we call the old HTTP client
               and summarize the usage patterns."
      │
      ▼
Explore subagent: reads 60 files (in ITS context)
      │
      ▼
Main session receives: "47 call sites in 3 patterns: (a) simple GET ×31, (b) retry wrapper ×12,
                        (c) streaming ×4. Files: …"
```

This is the same isolation idea as `context: fork` skills ([02](02-slash-commands-and-skills.md)) and subagents in Domain 1.

---

## 6. Real-world case study

**Situation A (too little planning):** A developer asked Claude to "switch our ORM from X to Y" with direct execution. Claude edited 38 files, then hit models with custom query builders that had no Y equivalent. Half the changes had to be reverted.

**Better:** Plan mode first. The exploration would have found the custom query builders, and the plan would have handled them first (or ruled out the migration).

**Situation B (too much planning):** Another developer used plan mode to fix an off-by-one error in a date function, with a clear failing test and stack trace. The plan step added time and gave no benefit.

**Better:** Direct execution.

**Situation C (context exhaustion):** During a large refactor, the main session filled up with search output from exploring 200 files, and Claude started losing track of earlier decisions.

**Better:** Delegate discovery to the **Explore subagent** and keep only the summary in the main session.

---

## Common Q&A

**Q: Does plan mode change files?**
A: No. Claude explores and proposes a plan; changes happen only after you approve and move to execution.

**Q: Should I always use plan mode to be safe?**
A: No. For small, clear changes it only adds overhead. Use it when scope is large or the approach is uncertain.

**Q: A single-file bug with a clear stack trace: plan or direct?**
A: **Direct execution.**

**Q: Restructuring a monolith into services: plan or direct?**
A: **Plan mode.** It involves architectural decisions and many files.

**Q: What problem does the Explore subagent solve?**
A: It keeps **verbose discovery output** out of the main context by doing the exploration separately and returning a summary, which prevents context exhaustion in long tasks.

**Q: Can I use both modes in one task?**
A: Yes, and you should for big tasks: plan mode to investigate and design, then direct execution to implement.

---

## Key Takeaways

- **Plan mode** = explore and design **before** changing anything. Use it for **large, architectural, multi-approach, multi-file** work.
- **Direct execution** = just do it. Use it for **small, well-scoped, obvious** changes.
- Plan mode **prevents costly rework** by catching wrong assumptions early.
- Use the **Explore subagent** for verbose discovery to **protect the main context**.
- Big tasks: **plan → review → execute**.
