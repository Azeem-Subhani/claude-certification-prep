# 07 · Session State, Resumption, and Forking

**Exam link:** Task Statement 1.7: *Manage session state, resumption, and forking.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (stateless API, context), [01](01-agentic-loops.md) (tool results in history).

---

## 1. What is a session?

A **session** is a saved agent conversation: your prompts, every tool call, every tool result, and every response. Claude Code and the Agent SDK store sessions automatically (as local transcript files) and give each one a **session ID**.

Sessions solve a real problem. Agent work often lasts longer than one sitting: you investigate a bug today and continue tomorrow. Without sessions, you would have to start from zero and re-read all the files.

There are three ways to work with a past session:

| Operation | What it does | Original session |
|---|---|---|
| **Resume** | Continue the same conversation with full history | Grows with new messages |
| **Fork** | Create a **new** session that starts as a **copy** of the history, then goes its own way | Unchanged |
| **Start fresh** | New, empty session (optionally with a summary you inject) | Not used |

> Sessions save the **conversation**, not the **files on disk**. If an agent edits files, those changes are real for every session in that directory.

---

## 2. Resuming a session

### In the Claude Code CLI

| Command | Effect |
|---|---|
| `claude --continue` | Resume the most recent conversation in the current directory |
| `claude --resume` | Open a picker to choose a session |
| `claude --resume <session-name>` | Resume a **named** session directly |
| `claude --resume <session-id>` | Resume by ID |
| `claude -n auth-investigation` | Start a session with a name |
| `/rename auth-investigation` | Name the current session |

**Named sessions** help when you run several investigations in parallel:

```bash
claude -n payment-timeout-bug
# ... work, exit ...
claude --resume payment-timeout-bug   # next day: continue exactly where you left off
```

### In the Agent SDK (Python)

```python
from claude_agent_sdk import query, ClaudeAgentOptions, ResultMessage

# 1) First run: capture the session ID from the result message
session_id = None
async for message in query(prompt="Analyze the auth module", options=ClaudeAgentOptions(allowed_tools=["Read", "Grep"])):
    if isinstance(message, ResultMessage):
        session_id = message.session_id

# 2) Later: resume it. The agent has its earlier analysis in context.
async for message in query(
    prompt="Now implement the refactoring you suggested",
    options=ClaudeAgentOptions(resume=session_id, allowed_tools=["Read", "Edit", "Grep"]),
):
    ...
```

(`continue_conversation=True` resumes the most recent session without an ID.)

---

## 3. Forking a session

**Forking** (`fork_session`) creates an **independent branch** from a shared starting point. Use it when you want to **compare different approaches** without repeating the expensive shared work.

```
                     ┌──▶ Fork A: "refactor using the Strategy pattern"
 Shared analysis ────┤
 (read 40 files,     └──▶ Fork B: "refactor using plain functions"
  mapped the code)
```

Both forks start with the full analysis, so neither needs to re-read the 40 files. Each develops independently, and the original stays untouched.

### Agent SDK (Python)

```python
# Branch from the analysis session to try an alternative, keeping the original intact
async for message in query(
    prompt="Instead of JWT, outline how OAuth2 would work for the auth module",
    options=ClaudeAgentOptions(resume=session_id, fork_session=True),
):
    if isinstance(message, ResultMessage):
        oauth_fork_id = message.session_id   # a NEW ID, different from session_id
```

### CLI

```bash
claude --resume auth-analysis --fork-session
```

(Inside Claude Code, `/branch` does the same from an active session.)

### Typical fork uses

- Compare **two testing strategies** (unit-heavy vs integration-heavy) from the same codebase analysis.
- Compare **two refactoring approaches**.
- Try a risky idea while keeping a "safe" line of work.

> **Remember:** forks share *conversation history*, not *file state*. If both forks edit the same files in the same directory, they affect each other. Use separate working copies (e.g. git worktrees) when forks will change code.

---

## 4. The stale context problem

When you resume, the agent's history contains **old tool results**: file contents it read, test outputs, search results. If the world has changed since then, those results are **stale** (out of date).

**Example:**

1. Monday: the agent reads `payment.py` and finds the bug on line 120.
2. Monday evening: a teammate refactors `payment.py`. The function moves to line 80 and its logic changes.
3. Tuesday: you resume and say "fix the bug". The agent trusts Monday's content and edits the wrong code.

The agent **does not know** the file changed unless you tell it, or unless it re-reads the file.

### Fix 1: Tell the resumed session what changed (targeted re-analysis)

If **most** prior context is still valid, resume and **name the specific changes**:

```text
Since our last session, payment.py and retry_policy.py were modified by a teammate
(refactor of process_payment). Everything else is unchanged.
Re-read those two files before continuing. Your other findings are still valid.
```

This is **targeted re-analysis**: the agent re-reads only what changed instead of exploring the whole codebase again. It's faster and cheaper.

> ⚠️ **Review item (R1):** the exam guide (task 1.7) supports this fix. The owner's cheat sheet says "never resume after files changed". Both agree on the heavy case: many changed files or a messy history means a fresh start with a summary. See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R1.

### Fix 2: Start fresh with a structured summary

If **much** of the prior context is stale (big refactor, many files changed, old test results no longer true), resuming is risky. The old tool results will mislead the model. Instead:

1. Write (or ask the agent to write, *before* things change) a **structured summary** of the important conclusions.
2. Start a **new session** and inject the summary.

```text
Context from previous investigation (summary, verified 2025-06-02):
- Goal: fix intermittent payment timeouts
- Confirmed: timeouts happen only when retry_count > 3
- Ruled out: database connection pool (load-tested, fine)
- Open question: does the new retry policy still apply exponential backoff?
- Note: payment.py was refactored since; re-read it before trusting any line numbers
```

> **Why is a fresh start with a summary more reliable than resuming with stale tool results?** The summary contains **conclusions** that are still true. The old transcript contains **raw outputs** that may be false now, and the model tends to trust what's already in its context.

---

## 5. Choosing: resume, fork, or start fresh

| Situation | Best choice |
|---|---|
| Continue yesterday's work; nothing important changed | **Resume** |
| Continue; a few known files changed | **Resume + tell it which files changed** |
| Many files changed / old results unreliable | **Start fresh + inject a structured summary** |
| Compare two approaches from the same analysis | **Fork** |
| Totally new, unrelated task | **Start fresh** (no summary needed) |

---

## 6. Real-world case study

**Situation:** A platform team uses Claude Code to plan a database migration. The investigation takes three days.

- **Day 1:** Named session `db-migration-analysis`. The agent maps 60 files that touch the database.
- **Day 2:** `claude --resume db-migration-analysis`. Two files changed overnight; the engineer tells the agent which two. The agent re-reads only those and continues.
- **Day 2 (afternoon):** The team can't choose between "dual-write" and "big-bang cutover". They **fork** the session twice. Each fork develops a full plan from the same 60-file analysis. They compare the two plans side by side.
- **Day 3:** A large unrelated refactor lands and touches 25 of the 60 files. Instead of resuming, the engineer asks for a **structured summary** of the decisions, starts a **fresh session**, injects the summary, and lets the agent re-map the changed files.

**Result:** No repeated full exploration, a clear comparison of the two approaches, and no decisions based on stale file contents.

---

## Common Q&A

**Q: What's the difference between resume and fork?**
A: Resume **continues** the same session (it grows). Fork creates a **new** session that starts as a copy; the original stays unchanged.

**Q: Does forking a session also copy the files?**
A: No. Forking copies the conversation history only. File changes on disk are shared by everything working in that directory.

**Q: I resumed a session after editing code, and the agent refers to old line numbers. Why?**
A: Its history contains **stale tool results** from before your edits. Tell it which files changed so it re-reads them, or start fresh with a summary.

**Q: When is starting fresh better than resuming?**
A: When a large share of the earlier tool results is out of date. Old raw outputs in context can mislead the model; a short, accurate summary is more reliable.

**Q: Why name sessions?**
A: So you can resume the right one directly (`--resume <name>`), especially when several investigations run in parallel.

**Q: Do I need to re-explore the whole codebase after any change?**
A: No. If you know what changed, tell the resumed session about those specific files for **targeted re-analysis**.

---

## Key Takeaways

- A **session** stores the conversation (not files). You can **resume**, **fork**, or **start fresh**.
- **`--resume <session-name>`** continues a named session; SDK uses `resume=<session_id>`.
- **`fork_session`** creates independent branches from a shared analysis to **compare approaches**.
- **Stale tool results** are the main risk of resuming after changes.
- Few changes → **resume and name the changed files**. Many changes → **start fresh with a structured summary**.
