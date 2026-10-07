# 01 · CLAUDE.md Hierarchy, Scoping, and Modular Organization

**Exam link:** Task Statement 3.1: *Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (scopes).
**Scope note:** This file covers where CLAUDE.md files live, `@import`, and splitting into `.claude/rules/`. **Conditional** (path-scoped) rules are covered in [03](03-path-specific-rules.md).

---

## 1. What is CLAUDE.md?

`CLAUDE.md` is a Markdown file of **instructions that Claude Code loads into context** automatically. It's the place for things Claude should always know about the project:

```markdown
# Project: Payments Service

## Commands
- Build: `make build`
- Test: `pytest -q`
- Lint: `ruff check .`

## Conventions
- Python 3.12, type hints required on public functions
- Money is always stored as integer cents, never floats
- All new endpoints need an integration test in tests/api/

## Architecture
- services/ holds business logic; api/ only does request/response mapping
```

Keep it **specific and concise**. Vague text ("write good code") doesn't help. Very long files reduce how well each rule is followed.

---

## 2. The hierarchy: three levels the exam tests

| Level | Location | Who it applies to | Shared via git? |
|---|---|---|---|
| **User** | `~/.claude/CLAUDE.md` | **Only you**, in every project | ❌ No |
| **Project** | `CLAUDE.md` in the repo root **or** `.claude/CLAUDE.md` | **Everyone** working on the repo | ✅ Yes |
| **Directory** | `CLAUDE.md` in a subdirectory (e.g. `packages/billing/CLAUDE.md`) | Everyone, when working in that part of the repo | ✅ Yes |

Other locations exist in current docs: an organization-wide **managed policy** CLAUDE.md (set by IT) and a personal, gitignored **`CLAUDE.local.md`** per project.

### How they combine

- Files are **combined (concatenated)**, not replaced. A project file doesn't erase your user file; both are in context.
- Broader files load first; more specific files appear later.
- **Subdirectory CLAUDE.md files load on demand**, when Claude reads or edits files in that subdirectory.
- If two files **contradict** each other, Claude may follow either one. Keep them consistent.

---

## 3. The classic problem: instructions in the wrong scope

> **Scenario:** A senior developer has carefully written team conventions. Everything works for them. A new team member clones the repo, and Claude ignores all the conventions.

**Diagnosis:** The conventions are in the senior developer's **`~/.claude/CLAUDE.md`** (user level). User-level files live on one person's machine. They're **never** shared through version control.

**Fix:** Move team conventions into the **project** `CLAUDE.md` (or `.claude/CLAUDE.md`) and commit it. Keep only personal preferences (e.g. "explain things briefly") in the user file.

| Instruction | Correct level |
|---|---|
| "Use `pnpm`, not `npm`, in this repo" | Project |
| "All API responses use the `{data, error}` envelope" | Project |
| "I prefer short explanations" | User |
| "My local test DB is at localhost:5433" | Local (`CLAUDE.local.md`) or user |
| "Billing package uses decimal money library X" | Directory (`packages/billing/CLAUDE.md`) or a rule file |

---

## 4. `@import`: keep CLAUDE.md modular

A CLAUDE.md can **import** other files with `@path/to/file`:

```markdown
# packages/billing/CLAUDE.md

Billing-specific notes:
- Money uses the decimal library, never floats.

@../../docs/standards/python-style.md
@../../docs/standards/pci-handling.md
```

- Paths can be **relative** (to the file containing the import) or **absolute**.
- Imported files can import others (up to a limited depth).
- To mention a path *without* importing it, put it in backticks: `` `@README` ``.

### Use case: selective imports per package

In a monorepo, different packages need different standards:

```
packages/
├── billing/CLAUDE.md     → @standards/python.md  @standards/pci.md
├── web/CLAUDE.md         → @standards/typescript.md  @standards/accessibility.md
└── infra/CLAUDE.md       → @standards/terraform.md  @standards/security.md
```

Each package **imports only the standards relevant to it**, based on what that package's maintainers know matters. Standards are written **once** and reused.

> **Note:** Imports organize files, but imported content still **loads into context**. To load content only when relevant, use path-scoped rules ([03](03-path-specific-rules.md)) or skills ([02](02-slash-commands-and-skills.md)).

---

## 5. `.claude/rules/`: split a large CLAUDE.md by topic

When a CLAUDE.md grows to hundreds of lines covering everything, it becomes hard to maintain, and merge conflicts become common.

**Alternative:** put **topic-specific files** in `.claude/rules/`:

```
.claude/rules/
├── testing.md            # how to write and run tests
├── api-conventions.md    # endpoint naming, error format, versioning
└── deployment.md         # release steps, environment rules
```

**Benefits:**

- Each file has **one topic** and a clear owner (the testing lead owns `testing.md`).
- Smaller files are easier to review and update.
- Rules without `paths` frontmatter load every session (like CLAUDE.md). Add `paths` to load a rule only for matching files ([03](03-path-specific-rules.md)).

User-level rules can also go in `~/.claude/rules/`, applying to all your projects.

---

## 6. Diagnosing with `/memory`

When Claude behaves **inconsistently** (follows a rule in one session, ignores it in another, or behaves differently for two developers), check **which memory files are actually loaded**.

- **`/memory`** lists your CLAUDE.md (and related) files across user and project scopes and lets you open them for editing.
- Current docs also point to **`/context`** to see exactly which CLAUDE.md and rules files loaded into this session.

**Typical findings:**
- The rule is in a **user** file on one machine only.
- A **subdirectory** CLAUDE.md wasn't loaded because Claude hadn't touched files there yet.
- Two files **contradict** each other.

---

## 7. Real-world case study

**Situation:** A 30-person team had one 900-line CLAUDE.md. Problems:
1. New hires didn't get the "never log card numbers" rule. It turned out to be in the tech lead's `~/.claude/CLAUDE.md`.
2. Frontend developers complained Claude kept applying backend database rules to React code.
3. Every week, two or three PRs had merge conflicts in CLAUDE.md.

**Changes:**
1. Moved security rules into the **project** level, committed to git.
2. Split the big file into `.claude/rules/testing.md`, `api-conventions.md`, `deployment.md`, `security.md`, plus path-scoped frontend and database rules ([03](03-path-specific-rules.md)).
3. Package-level CLAUDE.md files use `@import` to pull only the standards each package needs.
4. Added a team habit: when behavior seems wrong, run `/memory` (and `/context`) first.

**Result:** Consistent behavior for all developers, fewer conflicts, and a root CLAUDE.md under 100 lines.

---

## Common Q&A

**Q: Is `~/.claude/CLAUDE.md` shared with my team?**
A: No. It's on your machine only and applies to you in every project. Team rules belong in the project CLAUDE.md, committed to git.

**Q: Root `CLAUDE.md` or `.claude/CLAUDE.md`?**
A: Either works for project level. Pick one convention for the team.

**Q: Does a subdirectory CLAUDE.md replace the root one?**
A: No. Files are combined. The subdirectory file adds instructions when Claude works in that directory.

**Q: When should I use `@import` vs `.claude/rules/`?**
A: `@import` reuses shared standards files from specific CLAUDE.md files (e.g. per package). `.claude/rules/` organizes project instructions by topic, with the option of path scoping.

**Q: Does `@import` reduce token usage?**
A: No. It organizes content, but imported files still load. Use path-scoped rules or skills to reduce what loads.

**Q: Claude follows a convention for me but not for my colleague. First step?**
A: Run `/memory` (and `/context`) on both machines and compare which files load. Usually the rule is in one person's user scope.

---

## Key Takeaways

- Hierarchy: **user** (`~/.claude/CLAUDE.md`, only you) → **project** (root or `.claude/CLAUDE.md`, shared) → **directory** (subfolder, loads on demand).
- Files are **combined**, not overridden. Avoid contradictions.
- **Team rules must be in project scope.** User-level files are never shared via git.
- Use **`@import`** to reuse standards files selectively (e.g. per package).
- Split big CLAUDE.md files into **topic files in `.claude/rules/`**.
- Use **`/memory`** (and `/context`) to see which files load and diagnose inconsistent behavior.
