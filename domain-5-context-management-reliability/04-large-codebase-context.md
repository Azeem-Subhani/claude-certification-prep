# 04 · Managing Context in Large Codebase Exploration

**Exam link:** Task Statement 5.4: *Manage context effectively in large codebase exploration.*
**Prerequisites:** Domain 3 [04 · Plan Mode and the Explore Subagent](../domain-3-claude-code-configuration-workflows/04-plan-mode-vs-direct-execution.md), Domain 1 [07 · Sessions](../domain-1-agentic-architecture-orchestration/07-session-state-resumption-forking.md).
**Scope note:** Domain 3 introduced the Explore subagent; Domain 1 covered session resume and fork. This file covers **context degradation** in long explorations and the tools to fight it: **scratchpad files**, **phase summaries**, **manifests for crash recovery**, and **`/compact`**.

---

## 1. Context degradation in long sessions

Exploring a large codebase can take hours and hundreds of tool calls. As the session fills with file contents and search output, quality drops. Typical signs of **context degradation**:

- **Inconsistent answers**: the agent says the opposite of what it found an hour ago.
- **Generic instead of specific**: it talks about "the typical repository pattern" instead of the **specific class** it found earlier (`OrderRepository` in `src/orders/repo.py`).
- **Re-doing work**: it searches again for things it already found.

**Cause:** earlier findings are buried under lots of newer content (and may be compressed or summarized away when context fills).

---

## 2. Delegate investigations to subagents

Keep the **main agent** as a **coordinator** with a clean, high-level view. Send **specific questions** to subagents, which do the verbose reading in their **own** context and return compact answers:

| Subagent question | Returns |
|---|---|
| "Find all test files and their frameworks" | List of paths + framework per folder |
| "Trace the refund flow's dependencies" | Call chain: `api/refunds.py → RefundService.issue() → PaymentGateway.refund()` |
| "List every place `customer.email` is written" | 6 locations with file:line |

The main context receives **conclusions**, not thousands of lines of raw output.

---

## 3. Scratchpad files: write findings down

A **scratchpad file** is a file where the agent **records key findings** as it goes, and **reads them back** when it needs them.

```markdown
# scratchpad/refund-investigation.md

## Confirmed facts
- Refund entry point: `api/refunds.py::create_refund` (line 42)
- Business logic: `services/refund_service.py::RefundService.issue`
- Gateway call: `PaymentGateway.refund(amount_cents, charge_id)`; amounts in **cents**
- Idempotency: NOT implemented (no idempotency key passed) → likely cause of double refunds

## Open questions
- Does the webhook handler (`webhooks/stripe.py`) also create refunds?

## Decisions
- Fix idempotency in RefundService, not in the API layer (affects batch jobs too)
```

**Why it works:**
- Findings live **outside the context window**, so they survive context limits, compaction, and new sessions.
- Later questions can **refer to exact facts** ("see scratchpad: amounts in cents") instead of vague memory.
- It counteracts degradation: the agent re-reads **precise** notes, not compressed summaries.

> In a prompt or CLAUDE.md: *"Record confirmed findings, open questions, and decisions in `scratchpad/<task>.md`. Before answering a question about earlier findings, re-read the scratchpad."*

---

## 4. Phase summaries for multi-phase exploration

Big investigations run in **phases** (map → trace → analyze → plan). Between phases:

1. **Summarize** the key findings of the finished phase.
2. **Inject** that summary into the **initial context** of the next phase's subagents.

```
Phase 1: map structure        → summary: modules, entry points, owners
          │
          ▼ (summary injected)
Phase 2: trace refund flow    → summary: call chain, data flow, missing idempotency
          │
          ▼ (summary injected)
Phase 3: plan the fix         → plan
```

Each phase starts with **what matters** from earlier phases, not the full raw history.

---

## 5. Crash recovery with manifests

Long multi-agent runs can **crash** (process killed, machine restarted, rate limits). Without saved state, all progress is lost.

**Design:** each agent **exports its state** to a **known location** in a structured format; the coordinator writes a **manifest** of overall progress. On restart, the coordinator **loads the manifest** and **injects** it into agent prompts.

```jsonc
// state/manifest.json
{
  "task": "Map payment-related code for PCI audit",
  "updated_at": "2026-10-08T14:22:00Z",
  "phases": {
    "map_structure": {"status": "done", "summary_file": "state/phase1_summary.md"},
    "trace_flows":   {"status": "in_progress",
                      "done": ["refunds", "charges"],
                      "remaining": ["payouts", "disputes"]},
    "report":        {"status": "pending"}
  },
  "agents": {
    "tracer-refunds": {"status": "done", "output": "state/tracer-refunds.json"},
    "tracer-payouts": {"status": "failed", "error": "timeout", "attempts": 2}
  }
}
```

**On resume:** the coordinator reads the manifest, **skips** completed work, **re-runs** failed or remaining parts, and gives each agent the relevant summaries. (Compare session resume in Domain 1 · [07](../domain-1-agentic-architecture-orchestration/07-session-state-resumption-forking.md): a manifest is **your own structured state**, which works even when a session transcript is lost or stale.)

---

## 6. `/compact` in Claude Code

When an interactive Claude Code session fills with verbose discovery output, **`/compact`** replaces the conversation history with a **summary** (you can add instructions about what to keep), freeing context so the session can continue.

```text
/compact Keep the confirmed refund-flow call chain, the idempotency finding, and open questions.
```

**Use it with care:** compaction is a summary, so details can be lost (see progressive summarization in [01](01-conversation-context-preservation.md)). That's why you should **write important facts to a scratchpad first**, then compact.

---

## 7. Real-world case study

**Situation:** An engineer used Claude Code for a two-day exploration of a 400k-line legacy codebase to plan a payments refactor.

**Problems:**
- By mid-afternoon, Claude described "a typical service layer" instead of the specific `LegacyBillingFacade` it had found that morning.
- The laptop restarted overnight; a long multi-agent mapping run was lost.

**Changes:**
1. Specific questions delegated to **subagents** (Explore) so the main session stayed high-level.
2. A **scratchpad file** for confirmed facts, open questions, and decisions; Claude re-reads it before answering.
3. **Phase summaries** injected into each new phase's subagents.
4. A **manifest** in `state/` for the mapping run; after a crash, the coordinator resumes from it.
5. **`/compact`** used after big discovery bursts, after saving key facts to the scratchpad.

**Result:** Consistent, specific answers across both days, and crashes cost minutes instead of hours.

---

## Common Q&A

**Q: What are signs of context degradation?**
A: Inconsistent answers, generic "typical pattern" answers instead of specific classes found earlier, and repeated searches.

**Q: What is a scratchpad file for?**
A: Persisting key findings outside the context window so the agent can refer back to exact facts later.

**Q: How do subagents help with context?**
A: They do verbose exploration in their own context and return compact answers, keeping the main agent's context clean.

**Q: How do I make a long multi-agent run survive a crash?**
A: Have agents export state to a known location and the coordinator maintain a **manifest**; on resume, load it and inject it into prompts.

**Q: When should I use `/compact`?**
A: When an extended session fills with verbose discovery output. Save key facts (e.g. in a scratchpad) first, because compaction summarizes.

---

## Key Takeaways

- Long sessions suffer **context degradation**: inconsistent, generic answers.
- **Delegate** specific investigations to **subagents**; keep the main agent high-level.
- Keep **scratchpad files** of confirmed facts and re-read them.
- **Summarize each phase** and inject the summary into the next phase.
- Use **manifests** (structured state exports) for **crash recovery**.
- Use **`/compact`** to free context, after saving important facts.
