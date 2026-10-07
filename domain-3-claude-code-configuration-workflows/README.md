# Domain 3: Claude Code Configuration & Workflows

**Exam weight:** 20%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

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
| [07 · Practice Questions](07-scenario-practice-questions.md) | 25 original scenario questions with explanations |

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

## My notes

<!-- Add study notes, lab links, and weak areas here -->
