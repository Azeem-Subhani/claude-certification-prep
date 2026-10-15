# 02 · Custom Slash Commands and Skills

**Exam link:** Task Statement 3.2: *Create and configure custom slash commands and skills.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (scopes), [01](01-claude-md-hierarchy.md) (CLAUDE.md).

---

## 1. Why commands and skills?

Teams repeat the same multi-step instructions: "review this PR against our checklist", "write release notes", "analyze this module". Typing them every time is slow and inconsistent.

**Custom slash commands** and **skills** package these instructions so anyone can run them with `/name`.

| | **Slash command** | **Skill** |
|---|---|---|
| File | `.claude/commands/review.md` | `.claude/skills/review/SKILL.md` (a folder) |
| Invoked by | You, typing `/review` | You (`/review`) **or Claude automatically** when relevant |
| Extra files | No | Yes, the folder can hold scripts, templates, examples |
| Frontmatter | Yes | Yes, plus a few more options |

> **Current product note:** Claude Code docs now say custom commands have been **merged into skills**. A file at `.claude/commands/review.md` and a skill at `.claude/skills/review/SKILL.md` both create `/review`. The old commands folder still works. If both exist with the same name, the **skill** wins. The exam still treats `.claude/commands/` as the place for slash commands.

---

## 2. Project vs user scope

| Location | Scope | Use for |
|---|---|---|
| **`.claude/commands/`** | **Project**, shared via version control | Team-wide workflows everyone should use |
| `~/.claude/commands/` | **User**, only you | Personal shortcuts |
| **`.claude/skills/<name>/SKILL.md`** | **Project**, shared | Team skills |
| `~/.claude/skills/<name>/SKILL.md` | **User**, only you | Personal skills and personal variants |

> **Exam pattern:** "You want a `/review` command that every developer gets when they clone or pull the repo." → **`.claude/commands/review.md`** (project scope, committed). Not `~/.claude/commands/` (only you), and not CLAUDE.md (that's not a command).

### Personal variants without affecting teammates

You like the team's `/review` skill but want a stricter version for yourself. **Don't edit the team's file.** Create a **personal skill with a different name** in `~/.claude/skills/`, e.g. `~/.claude/skills/review-strict/SKILL.md`. The team keeps `/review`; you also have `/review-strict`.

---

## 3. A command/skill file

```markdown
---
description: Review the current branch against the team checklist
argument-hint: [pr-number]
allowed-tools: Read, Grep, Glob, Bash(git diff *)
---

Review pull request #$ARGUMENTS against our checklist:

1. Every new endpoint has input validation and an integration test.
2. No secrets, tokens, or PII in code or logs.
3. Money values use integer cents.
4. Public functions have type hints and docstrings.

Report findings grouped by severity (blocker / should-fix / nit) with file:line references.
```

- **`description`**: what it does. For skills, Claude uses this to decide when to **auto-invoke**.
- **`$ARGUMENTS`**: replaced with what the user types after the command (`/review 482`).
- The frontmatter options below control **how** it runs.

---

## 4. Key frontmatter options

### 4.1 `context: fork`: run in an isolated subagent

By default, a skill runs **in your main conversation**. Everything it reads and outputs stays in your context.

Some skills produce a **lot** of output:

- **Verbose analysis:** "analyze the whole codebase's dependency graph" reads dozens of files.
- **Exploration:** "brainstorm 5 alternative designs" produces long drafts you'll mostly discard.

Running these in the main session **pollutes** it, filling the context with material you don't need later.

`context: fork` runs the skill in an **isolated subagent**. The subagent does the heavy work in its own context and returns only a **result/summary** to your main conversation.

```markdown
---
description: Map the architecture of a module and summarize its dependencies
context: fork
---

Explore the module at $ARGUMENTS. Read the files you need, then return ONLY:
- a 10-line summary of its purpose,
- its public interface,
- its internal and external dependencies.
```

| Use `context: fork` for | Don't need it for |
|---|---|
| Codebase analysis that reads many files | A quick, short checklist |
| Brainstorming alternatives | Skills whose full output you need in the conversation |
| Any skill with large intermediate output | Skills that must see your current conversation |

> A forked skill **doesn't see your conversation history**. The skill's content is its prompt, so include everything it needs (Domain 1 · [03](../domain-1-agentic-architecture-orchestration/03-subagent-configuration.md)).

### 4.2 `allowed-tools`: list the tools for the skill

`allowed-tools` lists tools in the skill's frontmatter. The exam and the current product describe it differently:

- **Exam framing (guide 3.2):** `allowed-tools` **restricts** tool access while the skill runs. Use it to keep a skill safe and focused.
- **Current product:** `allowed-tools` **pre-approves** the listed tools, so they run without a permission prompt. It does not block other tools. To remove tools, use `disallowed-tools`, or a `deny` permission rule in settings.

```yaml
---
description: Generate a changelog file from commit messages
allowed-tools: Read, Write, Bash(git log *)
---
```

- **Exam view:** this skill is limited to reading files, writing the changelog, and running `git log`.
- **Product view:** these three run without prompts. Other tools still ask for permission. To truly block `rm` or `git push`, add them to `disallowed-tools` or to a `deny` rule.

> ⚠️ **Review item (R4):** for the exam, answer "use `allowed-tools` to limit what a skill can do". In real work, use `disallowed-tools` or `deny` rules to block tools. See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R4.

### 4.3 `argument-hint`: prompt for required parameters

```yaml
argument-hint: [pr-number] [severity-threshold]
```

When a developer types `/review`, autocomplete shows `[pr-number] [severity-threshold]`, so they know what to provide. This reduces "skill ran with no arguments and guessed" mistakes.

---

## 5. Skills vs CLAUDE.md: choosing correctly

| | **CLAUDE.md** | **Skill** |
|---|---|---|
| Loaded | **Always**, every session | **On demand**: when invoked or judged relevant (only the short description is always present) |
| Best for | **Universal standards**: build commands, code style, architecture rules | **Task-specific workflows**: release notes, PR review checklist, migration procedure |
| Token cost | Paid every session | Mostly paid only when used |

**Rule of thumb:**
- "Claude should **always** know this" → CLAUDE.md.
- "Claude needs this **when doing task X**" → skill.

**Example:** A 60-line "how to write a database migration" procedure in CLAUDE.md is loaded into every session, even when fixing CSS. As a `/db-migration` skill, it loads only when someone is writing a migration.

---

## 6. Real-world case study

**Situation:** A platform team had three problems:
1. A great `/review` command existed, but only in the tech lead's `~/.claude/commands/`.
2. A "codebase health report" command dumped thousands of lines into sessions, so later questions in the same session got worse answers.
3. A "generate docs" command once ran `git push` by mistake.

**Changes:**
1. Moved `/review` to **`.claude/commands/review.md`** and committed it. Everyone gets it on pull.
2. Rewrote the health report as a skill with **`context: fork`**; only the summary returns to the main session.
3. Added **`allowed-tools: Read, Glob, Grep, Write`** to the docs skill. (Exam view: this limits the skill. In current Claude Code, they would also need `disallowed-tools` to truly remove Bash.)
4. Added **`argument-hint: [module-path]`** to the docs skill so people stop running it without a target.
5. One developer wanted a stricter review: they created **`~/.claude/skills/review-strict/`** with a different name, leaving the team's `/review` unchanged.

---

## Common Q&A

**Q: Where do I put a command the whole team should have?**
A: `.claude/commands/` (or `.claude/skills/`) in the repo, committed to version control.

**Q: What does `context: fork` do?**
A: Runs the skill in an isolated subagent so its verbose or exploratory work doesn't fill your main conversation. Only the result comes back.

**Q: How do I stop a skill from running destructive commands?**
A: On the exam: limit its tools with `allowed-tools`, for example only read and write file tools. In current Claude Code, `allowed-tools` only pre-approves tools. Use `disallowed-tools` or a `deny` rule to block them.

**Q: What is `argument-hint` for?**
A: It shows the expected arguments when someone types the command, so they supply required parameters.

**Q: I want to customize a team skill just for me. Should I edit it?**
A: No. Create a personal copy in `~/.claude/skills/` with a **different name**, so teammates aren't affected.

**Q: Should our coding style guide be a skill?**
A: Usually no. Universal standards that apply to all work belong in CLAUDE.md (or rules). Skills are for on-demand workflows.

---

## Key Takeaways

- **Project** commands/skills (`.claude/commands/`, `.claude/skills/`) are shared via git; **user** ones (`~/.claude/...`) are personal.
- **`context: fork`** isolates verbose or exploratory skills in a subagent; only the summary returns.
- **`allowed-tools`** restricts what a skill can do, in the exam's framing. ⚠️ In current Claude Code it pre-approves tools, and `disallowed-tools` removes them.
- **`argument-hint`** tells users which parameters to pass.
- **Personal variants** → `~/.claude/skills/` with a **different name**.
- **Skills = on-demand, task-specific. CLAUDE.md = always-loaded, universal.**
