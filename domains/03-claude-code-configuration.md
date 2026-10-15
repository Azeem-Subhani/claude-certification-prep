# Domain 3: Claude Code Configuration & Workflows

> **Exam weight:** 20% of scored items (official exam guide v1.0, §4).
> **Task statements:** 3.1 to 3.6 (see [docs/domain-map.md](../docs/domain-map.md)).
> **Deep dives:** [domain-3-claude-code-configuration-workflows/](../domain-3-claude-code-configuration-workflows/README.md)
> **Facts checked against:** [docs/verification-log.md](../docs/verification-log.md). ⚠️ marks a place where the exam answer and today's product differ.

---

## Exam focus

- **Scope (3.1, 3.2).** The question is usually "who should get this?" Project files are shared through git. User files (`~/.claude/`) are personal.
- **Always-on vs on-demand (3.2, 3.3).**
  - Use CLAUDE.md for rules that always apply.
  - Use `.claude/rules/` with `paths` for rules about certain file types.
  - Use skills for workflows you run when needed.
- **Plan mode vs direct execution (3.4).** Choose by how clear the scope is, not by how hard the task is.
- **Iterative refinement (3.5).** Use examples when prose is misread, tests for edge cases, and the interview pattern for unfamiliar domains.
- **CI/CD (3.6).** Use `-p` for jobs that hang and JSON output with a schema for machines. Use an independent reviewer. Include prior findings when you review again.

---

## Key concepts

### 1. Words you need first

**Claude Code** is Anthropic's coding agent. It runs in the terminal, in IDEs, in a desktop app, and in CI.

**CLAUDE.md** is a Markdown file of instructions that Claude Code loads into context. It is **guidance**: Claude reads and follows it, but nothing forces it.

**Settings files** (`settings.json`) control behavior in code, for example permissions and hooks. They are **enforced** by Claude Code itself.

### 2. The CLAUDE.md hierarchy (Task 3.1)

| Level | File | Who gets it |
|---|---|---|
| **User** | `~/.claude/CLAUDE.md` | Only you, in all your projects. Not shared. |
| **Project** | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Everyone who clones the repo (committed to git). |
| **Directory** | `packages/api/CLAUDE.md` | Everyone. It loads when Claude works with files in that folder. |
| **Personal per repo** | `./CLAUDE.local.md` | Only you, in this repo. Keep it out of git. |

**The classic bug:** a new teammate does not follow the team's rules. The rules sit in someone's **user-level** file. Move them to the **project** file.

**Imports.** A CLAUDE.md file can include other files with `@path` lines, for example `@docs/testing-standards.md`. The exam calls this "@import". (There is no `@import` keyword; you write `@` plus the path.) Imports keep a long file modular. Each package can import only the standards it needs.

**`.claude/rules/`** holds topic files such as `testing.md` or `api-conventions.md`. They are an alternative to one huge CLAUDE.md.

**`/memory`** shows which memory files are loaded and opens them for editing. Use it to check why Claude follows a rule in one session and not in another. It does not load files.

**Conflicts.** No level is guaranteed to win when two CLAUDE.md files disagree. The docs say conflicting instructions may be followed unpredictably (verification log C6d). If a rule must hold every time, do not rely on CLAUDE.md. Use settings (see below).

### 3. When a rule must always hold: settings, permissions, and hooks

CLAUDE.md asks. Settings enforce.

| Settings file | Who gets it |
|---|---|
| `~/.claude/settings.json` | You, in every project |
| `.claude/settings.json` | Everyone (committed to git) |
| `.claude/settings.local.json` | You, in this project (gitignored) |

**Permission rules** allow or block tools:

```json
{
  "permissions": {
    "allow": ["Bash(npm run test *)"],
    "deny": ["Read(./.env)", "Read(./.env.*)"]
  }
}
```

**Hooks** in settings run your command at fixed events. For example, a `PreToolUse` hook can check a command and block it. The docs say to use **permission rules** for a hard allow or deny (verification log C18, C19).

### 4. Custom slash commands and skills (Task 3.2)

A **skill** is a saved workflow that loads only when it is used. You run it as a slash command, for example `/review`.

| Scope | Place | Shared? |
|---|---|---|
| Team | `.claude/skills/<name>/SKILL.md` (or the older `.claude/commands/<name>.md`) | Yes, through git |
| Personal | `~/.claude/skills/<name>/SKILL.md` (or `~/.claude/commands/`) | No |

A skill is a **folder** with a `SKILL.md` file inside. A flat `.md` file put directly in `.claude/skills/` is not a skill.

To make your own version of a team skill, create a personal skill with a **different name**. Then teammates are not affected.

**Frontmatter** is the YAML block at the top of `SKILL.md`. The exam tests three fields:

```markdown
---
description: Review the current diff against team standards
argument-hint: "[branch-name]"
context: fork
allowed-tools: Read, Grep, Glob
---
Review the changes on $ARGUMENTS ...
```

- **`context: fork`** runs the skill in a separate subagent context. Long, noisy output (analysis, brainstorming) then stays out of your main conversation.
- **`argument-hint`** tells the user what input the skill expects.
- **`allowed-tools`** lists tools for the skill.

> ⚠️ **Product note:** the exam guide says `allowed-tools` **restricts** tools during the skill. Current docs say it **pre-approves** the listed tools, so Claude can use them without asking. It does not block other tools. Use `disallowed-tools` to remove tools (verification log C9c, [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R4).

**Skill or CLAUDE.md?** Use CLAUDE.md for standards that always apply. Use a skill for a task you run on demand.

### 5. Path-specific rules (Task 3.3)

A rule file in `.claude/rules/` can have a `paths` field. It then loads **only** when Claude reads or edits a matching file. This saves tokens and keeps context relevant.

```markdown
---
paths:
  - "**/*.test.ts"
  - "**/*.test.tsx"
---
# Test conventions
- Use the shared fixtures in test/fixtures.
- One behavior per test.
```

**Glob** patterns match file paths:
- `*` matches any characters in one folder level.
- `**` matches any number of folders.
- So `**/*.test.tsx` means "every `.test.tsx` file anywhere".

**Choosing the right place:**

| The rule applies to... | Put it in... |
|---|---|
| All code | Root CLAUDE.md |
| One package folder | That folder's CLAUDE.md |
| One **file type** spread over many folders | `.claude/rules/` file with `paths` |
| A workflow you run when needed | A skill |

### 6. Plan mode vs direct execution (Task 3.4)

**Plan mode** lets Claude explore and design a plan **without changing files**. You review the plan before any edits. **Direct execution** means Claude makes the changes straight away.

**Use plan mode when:**
- the change is large (monolith to microservices)
- there are several valid approaches
- architecture decisions are needed
- many files will change, for example a library migration across 45 files

**Use direct execution when:**
- the change is small and clear, for example a one-file bug fix with a clear stack trace
- you are adding one validation check

**The key rule:** unclear scope decides, not difficulty. A hard but clear task can run directly. A simple but unclear task needs a plan.

**Plan, then execute.** A common pattern: plan the migration in plan mode, then switch to direct execution to apply the agreed plan.

**How to enter plan mode:** press Shift+Tab to cycle modes, start with `--permission-mode plan`, or type `/plan`.

**The Explore subagent** is a built-in read-only subagent. It does the noisy discovery work and returns a summary. Your main context stays clean.

### 7. Iterative refinement (Task 3.5)

| Problem | Technique |
|---|---|
| Claude reads your prose description in different ways | Give **2 to 3 concrete input/output examples**. |
| A complex change with many edge cases | **Test-driven iteration:** write the tests first, then share the failing test output. |
| You do not know the domain well | **Interview pattern:** ask Claude to ask you questions before it builds. |
| Examples work, but one edge case fails | Add a **specific test case** for that edge case (for example, a null value). |
| Several issues affect each other | Send them **together** in one detailed message. |
| Issues are independent | Fix them **one at a time**. |

### 8. Claude Code in CI/CD (Task 3.6)

**CI/CD** (continuous integration / continuous delivery) runs automated checks on every change.

| Need | Flag or practice |
|---|---|
| The job hangs, waiting for input | `-p` (or `--print`): non-interactive mode. Claude runs once and exits. |
| Machines must read the output | `--output-format json` plus `--json-schema`. The validated data is in the `structured_output` field. |
| Add rules but keep the default behavior | `--append-system-prompt` |
| Replace the whole system prompt | `--system-prompt` |
| Limit the number of turns | `--max-turns` |
| Pre-approve or remove tools | `--allowedTools` / `--disallowedTools` |

```bash
claude -p "Review this PR for bugs and security issues. Report only new findings." \
  --output-format json \
  --json-schema "$(cat review-schema.json)"
```

**Project context for CI** goes in CLAUDE.md: testing standards, available fixtures, and review criteria. The CI run loads it like any session.

**Use an independent reviewer.** The session that wrote the code keeps its own reasoning. So it is less likely to question its own choices. Run a separate Claude instance for the review.

**Re-reviews.** After new commits, include the earlier findings. Ask Claude to report only new or still-open issues. This avoids duplicate PR comments.

**Test generation.** Give Claude the existing tests. Then it does not suggest tests that already exist.

**Real-time or batch?**
- A check that blocks a merge needs the real-time API.
- An overnight report can use the Batch API (see Domain 4).

---

## Common traps

| Tempting wrong answer | Why it is wrong |
|---|---|
| "Put the team rules in `~/.claude/CLAUDE.md`." | That file is personal. Teammates never get it. |
| "The more specific CLAUDE.md always wins." | No such guarantee. Use settings or hooks for rules that must hold. |
| "Use `/memory` to load the missing file." | `/memory` shows and opens memory files. It does not load them. |
| "Add a CLAUDE.md to each of the 50 test folders." | Duplication. Use one `.claude/rules/` file with `paths: ["**/*.test.*"]`. |
| "Put Terraform rules in the root CLAUDE.md." | They load in every session and waste tokens. Use a path rule. |
| "Make the coding standard a skill." | Skills load only when called. Always-on standards belong in CLAUDE.md or rules. |
| "Put the team's `/deploy-check` command in `~/.claude/skills/`." | That is personal scope. Use `.claude/skills/` in the repo. |
| "Put `SKILL.md` directly in `.claude/skills/`." | A skill is a folder: `.claude/skills/<name>/SKILL.md`. |
| "Use plan mode for a clear one-line fix." | It adds a step with no benefit. Use direct execution. |
| "Start direct and switch to plan mode if problems appear." | If the task is already known to be complex, plan first. |
| "Rewrite the prose description more carefully." | Prose is the problem. Give input/output examples. |
| "Set `CLAUDE_HEADLESS=true` or redirect stdin from `/dev/null`." | These do not stop the hang. Use `-p`. |
| "Ask the same session to review its own code carefully." | It keeps its reasoning. Use an independent instance. |
| "Use the Batch API for the pre-merge check." | Batch has no speed guarantee. Blocking checks need real-time calls. |

---

## Worked scenario

**Setting:** A team of 12 developers uses Claude Code on a TypeScript monorepo. It has 6 packages and test files next to the source files in about 60 folders.

**Step 1: Fix missing rules for new hires.**
- Two new developers say Claude ignores the team's error-handling rules.
- `/memory` shows the rules only in a senior developer's `~/.claude/CLAUDE.md`.
- **Fix:** move them into the project's `CLAUDE.md` and commit it.

**Step 2: Shrink the 900-line CLAUDE.md.**
- The team splits it into `.claude/rules/api-conventions.md`, `.claude/rules/deployment.md`, and `.claude/rules/testing.md`.
- `testing.md` gets `paths: ["**/*.test.ts", "**/*.test.tsx"]`. So it loads only when Claude works on test files.

**Step 3: Make a must-hold rule.**
- Claude must never read the `.env` files.
- A CLAUDE.md line is not enough, so the team adds `"deny": ["Read(./.env)", "Read(./.env.*)"]` to `.claude/settings.json`.

**Step 4: Share a review workflow.**
- The team creates `.claude/skills/pr-review/SKILL.md` with `context: fork` and `argument-hint: "[PR number]"`.
- One developer wants a stricter version. They create `~/.claude/skills/pr-review-strict/` with a different name.

**Step 5: Pick a mode.**
- Task A: "Move the date library from moment to date-fns across 45 files." Many files change, so: plan mode, then direct execution.
- Task B: "Fix the null check in `parseInvoice`; the stack trace points to line 88." The scope is clear, so: direct execution.

**Step 6: Add CI review.**
- The CI job runs `claude -p` with `--output-format json --json-schema review-schema.json`.
- A script posts each finding as an inline PR comment.
- On new commits, the job passes the earlier findings and asks only for new or unresolved issues.

---

## Quick recap

- User level (`~/.claude/`) is personal. Project level (`.claude/`, root `CLAUDE.md`) is shared through git.
- CLAUDE.md is guidance. Settings permissions and hooks enforce rules that must always hold.
- Use `@path` imports to keep CLAUDE.md modular. Use `.claude/rules/` with `paths` for file-type rules across many folders.
- A skill is `.claude/skills/<name>/SKILL.md`. Use `context: fork` for noisy output and `argument-hint` for inputs.
- Plan mode is for unclear scope, many files, and architecture choices. Direct execution is for clear, small changes.
- Use examples for misread prose, tests for edge cases, the interview pattern for unfamiliar domains, and batched feedback for issues that affect each other.
- In CI: use `-p`, JSON output with `--json-schema`, an independent reviewer, and prior findings on re-review.

---

## Practice questions

Go to the Domain 3 question bank: **[mcq-bank/03-claude-code-configuration.md](../mcq-bank/03-claude-code-configuration.md)** (48 questions).
Older, easier warm-up questions: [domain-3-.../07-scenario-practice-questions.md](../domain-3-claude-code-configuration-workflows/07-scenario-practice-questions.md).
