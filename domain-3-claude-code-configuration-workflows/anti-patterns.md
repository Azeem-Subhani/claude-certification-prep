# Domain 3 Anti-patterns: Claude Code Configuration & Workflows

**Scope:** wrong choices for CLAUDE.md, settings, skills, path rules, plan mode, iterative refinement, and CI/CD (task statements 3.1 to 3.6).

---

## Anti-patterns

| # | Wrong approach | Why it fails | Do this instead | Topic |
|---|---|---|---|---|
| 1 | Put team rules in `~/.claude/CLAUDE.md`. | That file is personal. New teammates never get the rules. | Put team rules in the project `CLAUDE.md` (root or `.claude/`) and commit it. | [01](01-claude-md-hierarchy.md) |
| 2 | Expect the more specific CLAUDE.md to win a conflict. | No level is sure to win. Conflicting instructions may be followed in an unpredictable way. | Remove the conflict. For a rule that must hold, use settings or a hook. | [01](01-claude-md-hierarchy.md) |
| 3 | Use CLAUDE.md to block a tool or protect a file. | CLAUDE.md is guidance. Claude can still miss it. | Use a `deny` permission rule in `.claude/settings.json`. | [01](01-claude-md-hierarchy.md) |
| 4 | Keep one very long CLAUDE.md with every topic. | It is hard to maintain and loads everything in every session. | Split it with `@path` imports or `.claude/rules/` topic files. | [01](01-claude-md-hierarchy.md) |
| 5 | Use `/memory` to load a missing file. | `/memory` shows and opens memory files. It does not load them. | Use `/memory` to check which files are loaded, then fix the file's location. | [01](01-claude-md-hierarchy.md) |
| 6 | Put a team command in `~/.claude/skills/` or `~/.claude/commands/`. | User scope is personal, so teammates do not get it. | Put it in `.claude/skills/<name>/SKILL.md` (or `.claude/commands/`) and commit it. | [02](02-slash-commands-and-skills.md) |
| 7 | Put a flat `review.md` file directly in `.claude/skills/`. | A skill is a folder with a `SKILL.md` file inside. | Create `.claude/skills/review/SKILL.md`. | [02](02-slash-commands-and-skills.md) |
| 8 | Edit the team skill to suit your own taste. | It changes the workflow for everyone. | Create a personal skill in `~/.claude/skills/` with a different name. | [02](02-slash-commands-and-skills.md) |
| 9 | Run a noisy skill (codebase analysis, brainstorming) in the main context. | Its long output fills the conversation and pushes out useful context. | Add `context: fork` so it runs in a separate subagent context. | [02](02-slash-commands-and-skills.md) |
| 10 | Use a skill for an always-on standard. | Skills load only when someone calls them. | Put always-on standards in CLAUDE.md or `.claude/rules/`. | [02](02-slash-commands-and-skills.md) |
| 11 | Put a task workflow (a release checklist) in CLAUDE.md. | It loads in every session, even when nobody needs it. | Make it a skill that runs on demand. | [02](02-slash-commands-and-skills.md) |
| 12 | Copy a CLAUDE.md into each of 50 test folders. | The copies drift apart and are hard to update. | Use one `.claude/rules/` file with `paths: ["**/*.test.ts"]`. | [03](03-path-specific-rules.md) |
| 13 | Put file-type rules (Terraform, tests) in the root CLAUDE.md. | They load in every session and waste tokens. | Use a `.claude/rules/` file with a `paths` glob. | [03](03-path-specific-rules.md) |
| 14 | Use a folder CLAUDE.md for rules that span many folders. | It applies only inside that one folder. | Use a path rule with a glob that matches every folder. | [03](03-path-specific-rules.md) |
| 15 | Use direct execution for a multi-file architecture change. | Claude starts editing before the approach is agreed. Rework is costly. | Use plan mode first, then direct execution to apply the plan. | [04](04-plan-mode-vs-direct-execution.md) |
| 16 | Use plan mode for a clear one-file fix. | It adds a step with no benefit. | Use direct execution. | [04](04-plan-mode-vs-direct-execution.md) |
| 17 | Start direct and switch to plan mode only when problems appear. | If the task is known to be complex, the early edits are wasted. | Plan first when the scope is unclear or large. | [04](04-plan-mode-vs-direct-execution.md) |
| 18 | Do long discovery work in the main conversation. | Verbose output fills the main context. | Use the Explore subagent and keep only its summary. | [04](04-plan-mode-vs-direct-execution.md) |
| 19 | Rewrite a prose description "more carefully" when Claude misreads it. | Prose is the problem, so a new version is misread too. | Give 2 to 3 concrete input/output examples. | [05](05-iterative-refinement.md) |
| 20 | Use the interview pattern when you already know the exact change. | It adds questions you do not need. | Give examples or tests. Use the interview pattern for unfamiliar domains. | [05](05-iterative-refinement.md) |
| 21 | Send issues that affect each other one at a time. | Each fix can undo another. | Send interacting issues together in one message. Send independent issues one at a time. | [05](05-iterative-refinement.md) |
| 22 | Fix a CI hang with `CLAUDE_HEADLESS=true`, `--batch`, or stdin from `/dev/null`. | None of these stops Claude from waiting for input. | Run with `-p` (`--print`). | [06](06-ci-cd-integration.md) |
| 23 | Review code in the same session that wrote it. | The session keeps its own reasoning, so it agrees with itself. | Run a separate, independent Claude instance for the review. | [06](06-ci-cd-integration.md) |
| 24 | Re-run reviews without the earlier findings. | The same comments appear again on every push. | Include past findings. Ask only for new or still-open issues. | [06](06-ci-cd-integration.md) |
| 25 | Use the Batch API for a pre-merge check. | Batch can take up to 24 hours, with no speed guarantee. Developers would be blocked. | Use real-time calls for blocking checks. Use batch for overnight jobs. | [06](06-ci-cd-integration.md) |

---

## Common exam traps

1. **"Put the team rules in your user-level CLAUDE.md."** User-level files are not shared. Teammates must get project-level files from git.
2. **"The more specific CLAUDE.md always wins."** There is no such guarantee. Must-hold rules belong in settings or hooks.
3. **"Use `/memory` to load the file."** `/memory` only shows and opens memory files.
4. **"Add a CLAUDE.md to every test folder."** One path rule with `**/*.test.*` does the same job without copies.
5. **"Make the coding standard a skill."** Skills are on-demand. Always-on standards go in CLAUDE.md or rules.
6. **"Store the team's command in `~/.claude/`."** That is personal scope. Team commands go in `.claude/` in the repo.
7. **"Use plan mode because the task is hard."** Difficulty does not decide. Unclear scope, many files, and architecture choices do.
8. **"Use direct execution because the change is simple, even though nobody knows which files are affected."** Simple but unclear needs a plan.
9. **"Rewrite the prose description."** Concrete input/output examples beat more prose.
10. **"Set `CLAUDE_HEADLESS=true` to stop the CI hang."** Only `-p` runs Claude non-interactively.
11. **"Ask the authoring session to review its own code carefully."** Self-review keeps the same blind spots. Use an independent instance.
12. **"Move the pre-merge check to the Batch API to save 50%."** Blocking work cannot wait up to 24 hours.

> ⚠️ For the exam, `allowed-tools` "restricts" a skill's tools. In current Claude Code it pre-approves them, and `disallowed-tools` removes them. See [02](02-slash-commands-and-skills.md) and [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R4.
