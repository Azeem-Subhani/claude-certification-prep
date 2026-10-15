# Domain 3: Claude Code Configuration & Workflows

**Exam weight:** 20%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

## Exam focus

- **Scope (3.1, 3.2).** The question is usually "who should get this?" Project files are shared through git. User files (`~/.claude/`) are personal.
- **Always-on vs on-demand (3.2, 3.3).**
  - Use CLAUDE.md for rules that always apply.
  - Use `.claude/rules/` with `paths` for rules about certain file types.
  - Use skills for workflows you run when needed.
- **Plan mode vs direct execution (3.4).** Choose by how clear the scope is, not by how hard the task is.
- **Iterative refinement (3.5).** Use examples when prose is misread, tests for edge cases, and the interview pattern for unfamiliar domains.
- **CI/CD (3.6).** Use `-p` for jobs that hang and JSON output with a schema for machines. Use an independent reviewer. Include prior findings when you review again.

## Exam scenarios where this domain is primary

- Scenario 2: Code Generation with Claude Code
- Scenario 4: Developer Productivity with Claude
- Scenario 5: Claude Code for Continuous Integration

## Related guide material

- Preparation Exercise 2: Configure Claude Code for a Team Development Workflow
- Sample Questions 4 (shared slash command), 5 (plan mode), 6 (path-specific rules), 10 (`-p` in CI), 12 (large PR review)

## Study notes (read in order)

| File | Covers |
| --- | --- |
| [00 · Foundations and Glossary](00-foundations-and-glossary.md) | Map of Claude Code configuration: scopes, the `.claude/` folder, what loads when |
| [01 · CLAUDE.md Hierarchy](01-claude-md-hierarchy.md) | 3.1: user/project/directory levels, `@import`, `.claude/rules/`, `/memory` |
| [02 · Slash Commands and Skills](02-slash-commands-and-skills.md) | 3.2: commands vs skills, `context: fork`, `allowed-tools`, `argument-hint` |
| [03 · Path-Specific Rules](03-path-specific-rules.md) | 3.3: `paths` frontmatter, glob patterns, rules vs subdirectory CLAUDE.md |
| [04 · Plan Mode vs Direct Execution](04-plan-mode-vs-direct-execution.md) | 3.4: when to plan, when to just do it, the Explore subagent |
| [05 · Iterative Refinement](05-iterative-refinement.md) | 3.5: input/output examples, test-driven iteration, interview pattern, batching feedback |
| [06 · CI/CD Integration](06-ci-cd-integration.md) | 3.6: `-p`, `--output-format json`, `--json-schema`, independent review, re-review hygiene |
| [Anti-patterns](anti-patterns.md) | Wrong approaches, why they fail, what to do instead, and exam traps |
| [MCQ bank: Domain 3](../mcq-bank/03-claude-code-configuration.md) | 48 exam-level practice questions with answer key and explanations |

Sources: the exam guide PDF, plus Claude Code docs: [Memory / CLAUDE.md](https://code.claude.com/docs/en/memory), [Skills](https://code.claude.com/docs/en/skills), [Run Claude Code programmatically](https://code.claude.com/docs/en/headless). Where the guide and current product behavior differ (e.g. commands now merged into skills, `--bare` skipping CLAUDE.md), the notes give both.

---

## 3.1 Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization

**Know**
- [ ] Hierarchy: user (`~/.claude/CLAUDE.md`), project (`.claude/CLAUDE.md` or root `CLAUDE.md`), directory (subdirectory `CLAUDE.md`)
- [ ] User-level instructions apply only to that user; they are not shared through version control
- [ ] `@import` syntax keeps CLAUDE.md modular (e.g. each package imports only the standards it needs)
- [ ] `.claude/rules/` holds topic-specific rule files as an alternative to one huge CLAUDE.md

**Do**
- [ ] Diagnose hierarchy problems (e.g. a new teammate missing instructions that live in someone's user-level file)
- [ ] Use `@import` to include relevant standards files per package
- [ ] Split a large CLAUDE.md into `.claude/rules/` files (`testing.md`, `api-conventions.md`, `deployment.md`)
- [ ] Use `/memory` to see which memory files are loaded and diagnose inconsistent behavior

## 3.2 Create and configure custom slash commands and skills

**Know**
- [ ] Project commands in `.claude/commands/` (shared via git) vs user commands in `~/.claude/commands/` (personal)
- [ ] Skills in `.claude/skills/` with `SKILL.md`; frontmatter supports `context: fork`, `allowed-tools`, `argument-hint`
- [ ] `context: fork` runs a skill in an isolated subagent so its output doesn't pollute the main conversation
- [ ] Personal variants go in `~/.claude/skills/` with different names so teammates aren't affected

**Do**
- [ ] Create team-wide commands in `.claude/commands/`
- [ ] Use `context: fork` for verbose (codebase analysis) or exploratory (brainstorming) skills
- [ ] Restrict tools with `allowed-tools` (e.g. limit to file writes, no destructive actions)
- [ ] Use `argument-hint` to prompt for required parameters
- [ ] Choose skills (on-demand, task-specific) vs CLAUDE.md (always loaded, universal standards)

## 3.3 Apply path-specific rules for conditional convention loading

**Know**
- [ ] `.claude/rules/` files with YAML `paths` frontmatter (glob patterns) activate conditionally
- [ ] Path-scoped rules load only when editing matching files, which saves context and tokens
- [ ] Glob rules beat directory-level CLAUDE.md for conventions spread across many directories (e.g. tests everywhere)

**Do**
- [ ] Create path-scoped rules (e.g. `paths: ["terraform/**/*"]`)
- [ ] Apply conventions by file type regardless of location (e.g. `**/*.test.tsx`)
- [ ] Choose path rules over subdirectory CLAUDE.md when files are spread across the codebase

## 3.4 Determine when to use plan mode vs direct execution

**Know**
- [ ] Plan mode: complex tasks, large-scale changes, several valid approaches, architectural decisions, multi-file changes
- [ ] Direct execution: simple, well-scoped changes (e.g. one validation check in one function)
- [ ] Plan mode allows safe exploration and design before changes, which prevents costly rework
- [ ] The Explore subagent isolates verbose discovery and returns summaries, preserving main context

**Do**
- [ ] Choose plan mode for architectural work (microservice restructuring, 45+ file library migration, choosing between integration approaches)
- [ ] Choose direct execution for clear-scope changes (single-file bug fix with a stack trace, adding a date check)
- [ ] Use the Explore subagent for verbose discovery to avoid context exhaustion
- [ ] Combine: plan mode to investigate, then direct execution to implement the plan

## 3.5 Apply iterative refinement techniques for progressive improvement

**Know**
- [ ] Concrete input/output examples communicate transformations best when prose is interpreted inconsistently
- [ ] Test-driven iteration: write tests first, then iterate by sharing failures
- [ ] Interview pattern: Claude asks questions to surface considerations before implementing
- [ ] Interacting issues → one message; independent issues → fix sequentially

**Do**
- [ ] Give 2–3 concrete input/output examples when prose gives inconsistent results
- [ ] Write tests (expected behavior, edge cases, performance) before implementation, then share failures
- [ ] Use the interview pattern in unfamiliar domains (cache invalidation, failure modes)
- [ ] Give specific test cases with example input and expected output for edge cases (e.g. nulls in migrations)
- [ ] Send interacting issues together in one detailed message; fix independent issues one at a time

## 3.6 Integrate Claude Code into CI/CD pipelines

**Know**
- [ ] `-p` / `--print` runs Claude Code non-interactively
- [ ] `--output-format json` and `--json-schema` enforce structured output in CI
- [ ] CLAUDE.md provides project context (testing standards, fixtures, review criteria) to CI runs
- [ ] The session that wrote code is worse at reviewing it than an independent review instance

**Do**
- [ ] Run Claude Code in CI with `-p` to avoid interactive hangs
- [ ] Use `--output-format json` + `--json-schema` for machine-parseable findings (e.g. inline PR comments)
- [ ] On re-review after new commits, include prior findings and report only new or unaddressed issues
- [ ] Provide existing test files so generated tests don't duplicate covered scenarios
- [ ] Document testing standards, valuable test criteria, and fixtures in CLAUDE.md

---

## Key themes to remember

- **Shared → project files in git. Personal → `~/.claude/`.** Most "teammate doesn't get it" bugs are scope bugs.
- **Always-on → CLAUDE.md. On-demand → skills. File-type-specific → path rules.**
- **Plan for big or uncertain changes; execute directly for small, clear ones.**
- **Show, don't tell:** examples and tests beat prose descriptions.
- **CI = `-p` + structured output + fresh reviewer + no duplicate comments.**
- **CLAUDE.md is guidance.** Settings permission rules and hooks enforce rules that must always hold.
- **Use `@path` imports** to keep CLAUDE.md modular.
- **A skill is a folder:** `.claude/skills/<name>/SKILL.md`. Use `context: fork` for noisy output and `argument-hint` for inputs.

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

## Sources checked

Each technical claim in this folder was checked against official docs on 2026-10-15.
**Verdicts:** CONFIRMED = the docs agree. PARTIAL = they agree with limits. CONFLICT = the docs say something different. NOT FOUND = the docs do not say this.
Conflicts with the exam guide are listed in [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).

| ID | Claim | Verdict | What the docs say | Source |
|---|---|---|---|---|
| C6a | CLAUDE.md locations | CONFIRMED | User: `~/.claude/CLAUDE.md`. Project: `./CLAUDE.md` or `./.claude/CLAUDE.md`. Subfolder files load when Claude reads files there. | code.claude.com/docs/en/memory |
| C6b | `CLAUDE.local.md` | CONFIRMED | Supported. Personal project notes, kept out of git. | code.claude.com/docs/en/memory |
| C6c | Imports use `@path` | CONFIRMED | Paths resolve relative to the importing file. Up to four levels of nested imports. | code.claude.com/docs/en/memory |
| C6d | "More specific CLAUDE.md wins" | NOT FOUND | Files load from broad to specific. Conflicting instructions may be followed unpredictably. | code.claude.com/docs/en/memory |
| C7 | `.claude/rules/` with `paths:` frontmatter | CONFIRMED | The rule loads only when Claude reads or edits a matching file. Rules without `paths` load at start. | code.claude.com/docs/en/memory |
| C8 | `/memory` | CONFIRMED | Lists the memory files and opens them for editing. `/context` shows context use, including loaded memory files. | code.claude.com/docs/en/commands; code.claude.com/docs/en/memory |
| C9a | Skills: `.claude/skills/<name>/SKILL.md` | CONFIRMED | `.claude/commands/<name>.md` still works. | code.claude.com/docs/en/skills |
| C9b | Skill frontmatter | CONFIRMED | Includes `name`, `description`, `argument-hint`, `allowed-tools`, `disallowed-tools`, `context: fork`, `agent`, `model`, `disable-model-invocation`, `paths`, and more. | code.claude.com/docs/en/skills |
| C9c | `allowed-tools` **restricts** tools | **CONFLICT** | It **pre-approves** the listed tools. Other tools can still be called. Use `disallowed-tools` to remove tools. | code.claude.com/docs/en/skills |
| C10 | Plan mode | CONFIRMED | Shift+Tab cycles modes (default → acceptEdits → plan). `--permission-mode plan`. `/plan`. | code.claude.com/docs/en/permission-modes; code.claude.com/docs/en/commands |
| C10b | Explore subagent is read-only | CONFIRMED | Write and Edit are denied. | code.claude.com/docs/en/sub-agents |
| C11 | `/compact` and auto-compaction | CONFIRMED | `/compact [instructions]` summarizes. Auto-compaction runs at a threshold. | code.claude.com/docs/en/commands; code.claude.com/docs/en/context-window |
| C12 | CLI flags | CONFIRMED | `-p/--print`, `--output-format text | json | stream-json`, `--json-schema` (result in `structured_output`), `--append-system-prompt`, `--system-prompt`, `--max-turns` (print mode), `--allowedTools`, `--disallowedTools`, `--bare`, `--permission-mode`. | code.claude.com/docs/en/cli-reference; code.claude.com/docs/en/headless |
| C13 | GitHub Actions | CONFIRMED | `anthropics/claude-code-action@v1`, with CLI flags passed in `claude_args`. | code.claude.com/docs/en/github-actions |
| C16 | Git worktrees | CONFIRMED | `--worktree` / `-w` starts Claude in its own worktree. | code.claude.com/docs/en/cli-reference |
| C17 | Settings files | CONFIRMED | `~/.claude/settings.json` (you, all projects), `.claude/settings.json` (shared, committed), `.claude/settings.local.json` (you, this project, gitignored), managed settings (organization). Precedence, highest first: managed, command line, project local, shared project, user. | code.claude.com/docs/en/settings |
| C18 | Permission rules | CONFIRMED | `permissions.allow` / `deny` lists, for example `"Bash(npm run test *)"` and `"Read(./.env)"`. The hooks page says to use the permission system, not a hook, to enforce a hard allow or deny. | code.claude.com/docs/en/settings; code.claude.com/docs/en/hooks |
| C19 | Claude Code hooks | CONFIRMED | Hooks are set in settings files. A PreToolUse command hook blocks a call by exiting with code 2, or by returning `permissionDecision: "deny"`. A hook that times out does not block by default. The docs call CLAUDE.md the place for static conventions. | code.claude.com/docs/en/hooks |
