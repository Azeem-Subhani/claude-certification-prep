# 03 · Error Propagation Strategies Across Multi-Agent Systems

**Exam link:** Task Statement 5.3: *Implement error propagation strategies across multi-agent systems.*
**Prerequisites:** Domain 2 [02 · Structured Error Responses](../domain-2-tool-design-mcp-integration/02-structured-error-responses.md) (error categories, `isRetryable`, local recovery, failed vs empty), Domain 1 [02 · Multi-Agent Orchestration](../domain-1-agentic-architecture-orchestration/02-multi-agent-orchestration.md).
**Scope note:** Domain 2 covered how a **tool** reports errors and how a **subagent** recovers locally. This file is about the **system level**: what the **coordinator** needs to recover, the two opposite anti-patterns, and how errors become **coverage annotations** in the final output.

---

## 1. Errors are information for the coordinator

In a multi-agent system, the **coordinator** decides what to do when part of the work fails. It can only decide well if the error tells it enough.

**Structured error context** a subagent should send when it can't recover locally:

| Field | Why the coordinator needs it |
|---|---|
| **Failure type** (timeout, permission, not found, rate limit…) | Decide: retry later, re-route, or give up |
| **What was attempted** (queries, sources, retries) | Avoid repeating the same failed attempts |
| **Partial results** | Keep the useful work instead of throwing it away |
| **Potential alternatives** | Suggest other sources or approaches the coordinator can try |

```json
{
  "status": "partial",
  "failure_type": "timeout",
  "attempted": [
    "academic_search('AI film scoring adoption') → timeout ×3 (backoff 1s, 2s, 4s)",
    "academic_search('AI soundtrack composition study') → timeout"
  ],
  "partial_results": [
    {"claim": "Two major studios piloted AI temp scores in 2024", "source_url": "https://example.org/trade-news", "date": "2024-11-02"}
  ],
  "alternatives": ["industry trade publications via web_search", "retry academic_search after 15 minutes"],
  "coverage_impact": "No peer-reviewed sources yet for film scoring"
}
```

### Coordinator recovery options

With this context, the coordinator can choose sensibly:

| Situation | Coordinator decision |
|---|---|
| Transient failure, alternatives available | **Re-delegate** using an alternative source |
| Partial results are enough for the question | **Continue** with partial results and **annotate** the gap |
| Topic is critical and nothing worked | **Retry later** or **report** clearly that this part couldn't be completed |
| Permission failure | Don't retry; **report** and/or **escalate** |

---

## 2. Generic error statuses hide what matters

A subagent returning `"status": "search unavailable"` gives the coordinator **nothing to work with**:

- Was it one query or all of them?
- Was anything found before the failure?
- Is it temporary? Is there another way?

The coordinator is forced into a **blind choice**: retry everything (waste) or give up (lose coverage).

---

## 3. Two opposite anti-patterns

| ❌ Anti-pattern | What happens | Why it's bad |
|---|---|---|
| **Silent suppression**: returning empty results as success | Timeout → `{"findings": []}` with status OK | The coordinator thinks "nothing exists on this topic". The final report confidently omits it. **Hidden failure.** |
| **Fail-everything**: terminating the whole workflow on one failure | One of six subagents times out → whole research job aborts | **Throws away** five successful results; the user gets nothing |

The correct path is between them: **report the failure clearly, keep partial results, and continue where possible.**

### Access failure vs valid empty result (system view)

Domain 2 explained this at the tool level. At the system level, it decides what the **final report says**:

| Subagent result | Coordinator should conclude | Report should say |
|---|---|---|
| Access failure (timeout) | "We **don't know**" | "Sources for X were unavailable; coverage gap." |
| Valid empty result | "We searched and **found nothing**" | "No evidence found for X in sources A, B, C." |

Treating the first as the second turns **"we couldn't check"** into **"it doesn't exist"**, a serious reliability error.

---

## 4. Local recovery first, then propagate

(Pattern defined in Domain 2 · [02](../domain-2-tool-design-mcp-integration/02-structured-error-responses.md).) At the system level, the rule is:

```
Subagent: transient failure ──▶ retry locally (limited, with backoff) ──▶ success? ──▶ return results
                                                                           │
                                                                           └─ no ──▶ propagate structured
                                                                                    error context (§1)
```

The coordinator only sees errors the subagent **could not resolve**, and always **with** what was attempted and partial results.

---

## 5. Coverage annotations in the final output

When some sources failed, the **synthesis** must say so. Otherwise readers assume full coverage.

```markdown
## Coverage notes
| Topic | Coverage | Notes |
|---|---|---|
| Music production | ✅ Well supported | 7 sources, 2024–2025 |
| Visual arts | ✅ Well supported | 5 sources |
| Film scoring | ⚠️ Partial | Academic database unavailable (timeouts); based on 1 trade-press source |
| Publishing | ⚠️ Contested | Credible sources disagree (see section 4) |
| Gaming | ❌ Gap | No sources retrieved; search service failed after retries |
```

**Why:** Readers can judge how much to trust each section, and the team knows what to re-run later.

---

## 6. Real-world case study

**Situation:** A market-research system with 6 subagents produced polished reports. A client noticed the "Asia-Pacific" section was missing in one report, with no explanation.

**Investigation:**
- The APAC subagent had hit timeouts and returned `{"findings": []}` with status `success` (**silent suppression**).
- After a fix, an engineer made any subagent error **abort the whole job** (**fail-everything**). Clients then received nothing on bad-network days.

**Final design:**
1. Subagents retry transient errors locally (3 attempts with backoff).
2. Unresolved errors go to the coordinator as **structured context**: failure type, attempts, partial results, alternatives.
3. The coordinator **re-delegates** to alternative sources when possible, otherwise **continues**.
4. Reports include a **coverage table** marking well-supported, partial, and gap areas.

---

## Common Q&A

**Q: What should a subagent send when it can't recover from an error?**
A: Failure type, what was attempted, partial results, and possible alternatives.

**Q: What's wrong with returning an empty list when a search times out?**
A: It hides the failure. The coordinator and the final report will treat "couldn't search" as "nothing exists".

**Q: Should one failed subagent stop the entire workflow?**
A: No. That throws away good work. Continue with partial results and annotate the gap.

**Q: Why is "search unavailable" not enough?**
A: It hides which query failed, what was found, and what alternatives exist, so the coordinator can't make a smart recovery decision.

**Q: How should the final report reflect failures?**
A: With **coverage annotations**: which findings are well supported and which areas have gaps due to unavailable sources.

---

## Key Takeaways

- Propagate **structured error context**: **failure type, attempts, partial results, alternatives**.
- **Generic statuses** hide what the coordinator needs.
- Avoid both anti-patterns: **silent suppression** and **terminating everything on one failure**.
- Keep **access failure ≠ valid empty result** all the way to the final report.
- **Recover locally first**; propagate only unresolved errors.
- Add **coverage annotations** so readers know what is well supported and what is missing.
