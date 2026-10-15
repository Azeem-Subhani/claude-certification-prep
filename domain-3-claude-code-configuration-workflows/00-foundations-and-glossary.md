# 00 · Foundations and Glossary (Domain 3)

> **Read this first.** This file is a map of how Claude Code is configured. Later files go deep on each part.
> Agent basics are in [Domain 1 · 00](../domain-1-agentic-architecture-orchestration/00-foundations-and-glossary.md). Tools, MCP, and `.mcp.json` are in [Domain 2](../domain-2-tool-design-mcp-integration/README.md).

**Exam link:** Background for all of Domain 3 (20% of the exam).

---

## 1. What is Claude Code?

**Claude Code** is Anthropic's coding agent. It reads your codebase, edits files, runs commands, and works through tasks with you. You can use it:

- **Interactively**: in a terminal, an IDE, or the desktop app, chatting and approving actions.
- **Non-interactively**: in scripts and CI pipelines (`claude -p "..."`), with no human at the keyboard.

Domain 3 is about **configuring** Claude Code so it behaves well for a **team**, and **using** it with good workflows.

---

## 2. The big idea: context is loaded from files

Every Claude Code session starts with a **fresh context window**. Claude knows nothing about your project's rules unless they are loaded from somewhere. Configuration files are how you give Claude **persistent knowledge**:

| What you want Claude to know or do | Where it goes | Loaded when | File |
|---|---|---|---|
| Universal project standards (build commands, style, architecture) | `CLAUDE.md` | Every session | [01](01-claude-md-hierarchy.md) |
| Topic-specific rules | `.claude/rules/*.md` | Every session (or only for matching files) | [01](01-claude-md-hierarchy.md), [03](03-path-specific-rules.md) |
| Conventions for certain file types | `.claude/rules/*.md` with `paths:` | Only when working on matching files | [03](03-path-specific-rules.md) |
| Repeatable task workflows | Skills / slash commands | Only when invoked | [02](02-slash-commands-and-skills.md) |
| External tools and systems | MCP servers (`.mcp.json`) | Session start | [Domain 2 · 04](../domain-2-tool-design-mcp-integration/04-mcp-server-integration.md) |

> **Context costs tokens and attention.** Everything loaded "every session" uses space in every request. Good configuration loads **only what is needed, when it is needed**.

---

## 3. Scopes: who gets the configuration?

The same idea appears everywhere in Claude Code: **user scope** vs **project scope**.

| Scope | Location | Shared with team? | Use for |
|---|---|---|---|
| **User** | `~/.claude/…` (your home folder) | ❌ No, only you, in all your projects | Personal preferences, experiments |
| **Project** | `.claude/…` or files in the repo root, **committed to git** | ✅ Yes, everyone who clones the repo | Team standards and shared workflows |
| **Local** (personal, per project) | e.g. `CLAUDE.local.md` (gitignored) | ❌ No | Your private notes for one project |
| **Managed** (organization) | Set by IT/admins | ✅ Everyone in the org | Company-wide policies |

> **The #1 Domain 3 diagnosis:** "A teammate doesn't get the behavior I get." → Check whether the config lives in **user** scope (only you) instead of **project** scope (everyone).

---

## 4. The `.claude/` folder at a glance

```
my-project/
├── CLAUDE.md                     # project instructions (or .claude/CLAUDE.md)
├── CLAUDE.local.md               # personal project notes (gitignored)
├── .mcp.json                     # shared MCP servers (Domain 2)
├── packages/
│   └── billing/
│       └── CLAUDE.md             # directory-level instructions
└── .claude/
    ├── rules/
    │   ├── testing.md            # topic rule (always loaded)
    │   └── terraform.md          # path-scoped rule (paths: ["terraform/**/*"])
    ├── commands/
    │   └── review.md             # project slash command → /review
    ├── skills/
    │   └── analyze-codebase/
    │       └── SKILL.md          # project skill
    ├── agents/                   # custom subagents
    └── settings.json             # permissions, hooks, etc.

~/.claude/                        # USER scope (your machine only)
├── CLAUDE.md
├── rules/
├── commands/
└── skills/
```

---

## 5. Working modes

| Mode | What it means | File |
|---|---|---|
| **Direct execution** | Claude reads, edits, and runs things to complete the task | [04](04-plan-mode-vs-direct-execution.md) |
| **Plan mode** | Claude explores and proposes a plan **without changing files**, until you approve | [04](04-plan-mode-vs-direct-execution.md) |
| **Non-interactive (`-p`)** | One-shot run for scripts and CI; no prompts, no waiting for input | [06](06-ci-cd-integration.md) |

---

## 6. The six Domain 3 topics

| File | Task statement | One-line summary |
|---|---|---|
| [01](01-claude-md-hierarchy.md) | 3.1 CLAUDE.md | Put instructions at the right level and keep them modular |
| [02](02-slash-commands-and-skills.md) | 3.2 Commands and skills | Package repeatable workflows; isolate and restrict them |
| [03](03-path-specific-rules.md) | 3.3 Path-specific rules | Load conventions only for matching files |
| [04](04-plan-mode-vs-direct-execution.md) | 3.4 Plan vs execute | Plan big/uncertain work; execute small/clear work |
| [05](05-iterative-refinement.md) | 3.5 Iterative refinement | Examples, tests, interviews, and smart feedback batching |
| [06](06-ci-cd-integration.md) | 3.6 CI/CD | Run Claude Code safely and usefully in pipelines |
| [MCQ bank](../mcq-bank/03-claude-code-configuration.md) | All | 48 exam-level practice questions |

---

## 7. Glossary

| Term | Meaning |
|---|---|
| **`@import`** | A line like `@docs/testing.md` inside CLAUDE.md that pulls another file into context |
| **`allowed-tools`** | Skill/command frontmatter listing tools Claude may use while that skill runs |
| **`argument-hint`** | Skill/command frontmatter showing what arguments to type, e.g. `[pr-number]` |
| **CLAUDE.md** | Markdown file of persistent instructions that Claude Code loads into context |
| **`context: fork`** | Skill frontmatter that runs the skill in an isolated subagent |
| **Direct execution** | Letting Claude make changes immediately, without a separate planning step |
| **Explore subagent** | Built-in read-only subagent for searching and understanding code; returns a summary |
| **Frontmatter** | YAML settings between `---` lines at the top of a Markdown file |
| **Glob pattern** | File-matching pattern like `**/*.test.tsx` (`**` = any folders, `*` = any characters) |
| **Headless / non-interactive mode** | Running `claude -p` with no human interaction (scripts, CI) |
| **Interview pattern** | Asking Claude to question you about requirements before it implements |
| **`/memory`** | Command that lists memory files (CLAUDE.md etc.) and opens them for editing |
| **Path-scoped rule** | A `.claude/rules/` file with `paths:` that loads only for matching files |
| **Plan mode** | Mode where Claude explores and plans without editing until you approve |
| **Project scope** | Configuration committed to the repo and shared with the team |
| **Skill** | A packaged workflow (`SKILL.md`) loaded only when invoked or relevant |
| **Slash command** | A command typed as `/name`; custom ones come from commands or skills |
| **User scope** | Configuration in `~/.claude/` that applies only to you |

---

## Common Q&A

**Q: Does Claude Code remember my project between sessions?**
A: Not by itself. Each session starts fresh. CLAUDE.md files, rules, and (optionally) auto memory provide persistent knowledge.

**Q: What's the simplest way to give the whole team the same instructions?**
A: A project `CLAUDE.md` committed to version control.

**Q: Why not put everything in CLAUDE.md?**
A: It loads into **every** session. Large files waste tokens and reduce how well Claude follows each rule. Move task-specific workflows into skills and file-type conventions into path-scoped rules.

**Q: Are CLAUDE.md instructions enforced like code?**
A: No. They are **context**. Claude usually follows them, but they are not guaranteed. To block an action every time, use a hook or permission rule (Domain 1 · [05](../domain-1-agentic-architecture-orchestration/05-agent-sdk-hooks.md)).

---

## Key Takeaways

- Claude Code gets persistent knowledge from **files**: CLAUDE.md, rules, skills, MCP config.
- **User scope (`~/.claude/`) = only you. Project scope (repo, git) = the team.**
- Load **only what's needed**: always-on (CLAUDE.md), per file type (path rules), on demand (skills).
- Modes: **plan**, **direct execution**, and **non-interactive (`-p`)** for CI.
- CLAUDE.md is **guidance**, not enforcement.
