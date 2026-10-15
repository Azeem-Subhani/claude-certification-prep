# 02 · Multi-Agent Orchestration (Coordinator–Subagent Pattern)

**Exam link:** Task Statement 1.2: *Orchestrate multi-agent systems with coordinator-subagent patterns.*
**Prerequisite:** [01 · Agentic Loops](01-agentic-loops.md).
**Scope note:** This file covers the **architecture** (who does what, and why). The **configuration** (how to define and start subagents in the Agent SDK) is in [03](03-subagent-configuration.md).

---

## 1. Why use more than one agent?

One agent with many tools works well for small tasks. For big tasks, a single agent runs into problems:

- **Context overload:** Its history fills with search results, documents, and notes. Attention dilution grows (see [00](00-foundations-and-glossary.md)).
- **Mixed roles:** One prompt must cover searching, analyzing, and writing. Each job gets less focus.
- **No parallel work:** It does one thing at a time.

A **multi-agent system** splits the work across several agents, each with a focused job and its own clean context.

**When *not* to use multiple agents:** if the task is small or the steps are tightly linked, extra agents give no benefit. They only add cost, latency, and complexity.

---

## 2. Hub-and-spoke architecture

The pattern the exam focuses on is **hub-and-spoke**:

```
                    ┌─────────────────┐
         user ─────▶│   COORDINATOR   │─────▶ final answer
                    │     (hub)       │
                    └──┬─────┬─────┬──┘
                       │     │     │
             ┌─────────┘     │     └─────────┐
             ▼               ▼               ▼
      ┌────────────┐  ┌────────────┐  ┌────────────┐
      │ Web search │  │  Document  │  │ Synthesis  │
      │  subagent  │  │  analysis  │  │  subagent  │
      │  (spoke)   │  │  subagent  │  │  (spoke)   │
      └────────────┘  └────────────┘  └────────────┘
```

- The **coordinator** (also called the orchestrator or lead agent) is the hub.
- **Subagents** are the spokes. Each one does a single focused job.
- **All communication goes through the coordinator.** Subagents never talk to each other directly.

### Why route everything through the hub?

| Benefit | What it means in practice |
|---|---|
| **Observability** | One place to log every request and result, so debugging is easy |
| **Consistent error handling** | One place decides what to do when a subagent fails (retry, skip, re-delegate, report) |
| **Controlled information flow** | The coordinator chooses exactly what each subagent receives, so no leakage of irrelevant or sensitive data |

If subagents talked directly to each other ("mesh" design), you would lose all three. Errors could spread silently, and nobody would have the full picture.

> ⚠️ **Product note:** the exam treats direct subagent-to-subagent talk as always wrong. Current Claude Code lets a subagent start its own subagents, up to three layers deep, and this can be turned off. This does not change the hub-and-spoke exam answer. (See the Sources checked table in the [README](README.md), row C3.)

---

## 3. Subagents have isolated context

This is one of the most tested facts in the domain:

> **A subagent does not inherit the coordinator's conversation history.** It starts with a fresh, empty context. It only sees what the coordinator puts in its prompt.

**Consequences:**

- ✅ Good: each subagent's context stays small and focused, so less attention dilution.
- ⚠️ Risk: if the coordinator forgets to pass key information (e.g. the user's original question, a date range, earlier findings), the subagent works blind.

**Example:** The user tells the coordinator, *"Only use sources from 2024 onward."* The coordinator sends the search subagent *"Research AI in healthcare."* The subagent returns 2019 papers. The subagent did nothing wrong: it never saw the constraint. The fix is in the coordinator's delegation prompt. (How to pass context well: [03](03-subagent-configuration.md).)

---

## 4. The coordinator's four jobs

### 4.1 Task decomposition

Break the user's request into subtasks. *"Write a report on the impact of AI on creative industries"* might become:

- Music industry impact
- Visual arts and design impact
- Film and video impact
- Writing and publishing impact
- Labor-market and legal issues across all of them

### 4.2 Delegation, with dynamic selection

Choose **which** subagents to call, based on the query. Not every query needs every subagent.

| Query | Sensible delegation |
|---|---|
| "What is the capital of Australia?" | No subagents; answer directly |
| "Summarize this attached PDF" | Document analysis only |
| "Compare current research on battery recycling" | Web search + document analysis + synthesis |

> **Exam pattern:** "Always run every request through the full pipeline" is a **wrong** design. It wastes time and money on simple queries. A good coordinator **analyzes query requirements and selects subagents dynamically**.

### 4.3 Result aggregation

Combine subagent outputs into one result: remove duplicates, resolve conflicts, keep source attribution.

### 4.4 Quality control through iterative refinement

The coordinator checks the combined result for **gaps** and fixes them:

```
1. Delegate research → 2. Synthesize → 3. Coordinator reviews:
      "Is anything missing?"
          │
          ├── Yes → send targeted queries to search/analysis subagents
          │         → re-run synthesis → review again
          │
          └── No  → deliver final report
```

This is an **iterative refinement loop**. It stops when coverage is sufficient (with a safety limit on rounds, like any loop).

---

## 5. The biggest coordinator risk: overly narrow decomposition

If the coordinator splits a broad topic too narrowly, the final result **looks complete but has blind spots**. The subagents did their jobs perfectly. The *plan* was the problem.

**Example (similar to exam scenarios):** Topic: *"Impact of AI on creative industries."* The coordinator creates only three subtasks: *AI in digital art*, *AI in graphic design*, *AI in photography*. The final report is excellent, but it covers only visual arts. Music, writing, film, and gaming are missing.

**How to diagnose it:** check the coordinator's decomposition (its list of subtasks), not the subagents' outputs. If every subagent did its assigned work well and coverage is still missing, the root cause is the decomposition.

**How to prevent it:**

- In the coordinator prompt, ask it to first **list the major dimensions** of the topic before assigning work.
- Add the gap-check step from §4.4.
- Check coverage against the original question, not against the subtask list.

---

## 6. Partitioning scope to avoid duplication

The opposite problem is **overlap**: two subagents research the same thing. This wastes tokens, and the synthesis step must remove duplicates.

**Partition by clear, separate boundaries:**

| Partition by | Example |
|---|---|
| **Subtopic** | Agent A: music; Agent B: film; Agent C: publishing |
| **Source type** | Agent A: academic papers; Agent B: news; Agent C: industry reports |
| **Time period** | Agent A: 2020–2022; Agent B: 2023–present |
| **Region** | Agent A: EU regulation; Agent B: US regulation |

Each subagent's prompt should state its boundary *and* what is out of scope ("Do not cover film; another agent handles that").

---

## 7. Real-world case study: research system

**Situation:** A consulting firm builds a research assistant. Version 1 is a single agent. On broad topics, it gives shallow answers, and its context fills with raw search results.

**Version 2 (hub-and-spoke):**

1. **Coordinator** reads the question and lists 4–6 dimensions.
2. It starts **web search subagents in parallel**, one per dimension, with clear boundaries.
3. A **document analysis subagent** reads the PDFs the user attached.
4. A **synthesis subagent** gets all findings (with sources) and writes a draft.
5. The coordinator **reviews the draft for gaps**, sends 1–2 targeted follow-up searches, and re-runs synthesis.

**Results:** better coverage and faster run time (parallel searches). All communication is logged in one place, so the team can see exactly which subagent produced each claim.

---

## Common Q&A

**Q: Can subagents call each other directly to save time?**
A: In the hub-and-spoke pattern tested on the exam, no. All routing goes through the coordinator for observability, consistent error handling, and control over information flow.

**Q: A subagent's output ignored a constraint the user gave. Whose fault is it?**
A: Usually the coordinator's. Subagents don't inherit context; the coordinator must include the constraint in the delegation.

**Q: The final report is well written but misses whole areas of the topic. What do you check first?**
A: The coordinator's **task decomposition**. If each subagent covered its assigned area well, the plan was too narrow.

**Q: Should the coordinator always use all subagents for thoroughness?**
A: No. It should choose subagents based on the query. Simple queries may need none.

**Q: How does the coordinator know when to stop refining?**
A: It evaluates the synthesis against the original question and quality criteria. When no meaningful gaps remain, it stops. A round limit acts as a safety net.

**Q: Is multi-agent always better than single-agent?**
A: No. It adds cost and complexity. Use it when tasks are broad, parallel, or need separate focused contexts.

---

## Key Takeaways

- **Hub-and-spoke:** one coordinator, many focused subagents, and **all communication through the hub**.
- Routing through the hub gives **observability, consistent error handling, and controlled information flow**.
- Subagents have **isolated context**; they know only what the coordinator gives them.
- The coordinator **decomposes, delegates dynamically, aggregates, and refines iteratively**.
- **Overly narrow decomposition** causes hidden coverage gaps; check the plan first.
- **Partition scope** with clear boundaries to avoid duplicate work.
