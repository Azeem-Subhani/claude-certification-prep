# 07 · Scenario Practice Questions (Domain 3)

**Exam link:** All of Domain 3 (Task Statements 3.1–3.6).
**How to use:** Answer each question before you open the explanation. Each answer links to the file that teaches the concept. These are **original practice questions** written for study. They are not taken from the official exam or practice test.

**Where Domain 3 appears:** It is a primary domain in **Code Generation with Claude Code**, **Developer Productivity with Claude**, and **Claude Code for Continuous Integration**.

---

## How to approach Domain 3 questions

1. **"Teammate doesn't get it"** → check **scope**: user (`~/.claude/`) vs project (repo, git).
2. **"Where should this instruction live?"** → always-on (CLAUDE.md), file-type-specific (path rule), on-demand workflow (skill/command).
3. **"Plan or just do it?"** → judge **scope and uncertainty**.
4. **"Claude keeps misunderstanding"** → give **examples or tests**, not more prose.
5. **CI questions** → `-p`, structured output, CLAUDE.md context, independent reviewer, no duplicates.

---

## Scenario A: Code Generation with Claude Code

*Your team uses Claude Code daily for code generation, refactoring, debugging, and documentation.*

### A1
A senior developer's Claude follows the team's API conventions perfectly. A new hire's Claude ignores them. The conventions are in the senior developer's `~/.claude/CLAUDE.md`. What is the best fix?

- A. Ask the new hire to copy the file into their own `~/.claude/CLAUDE.md`.
- B. Move the conventions into the project `CLAUDE.md` (or `.claude/CLAUDE.md`) and commit it.
- C. Add the conventions to every prompt.
- D. Create a skill named `/conventions`.

<details><summary>Answer</summary>

**B.** User-level files aren't shared via version control. Team rules belong in project scope. A works for one person but drifts and doesn't scale; C and D are not always-on. → [01](01-claude-md-hierarchy.md)
</details>

### A2
Your root CLAUDE.md is 800 lines and causes frequent merge conflicts. Different leads own testing, API, and deployment guidance. What is the best restructuring?

- A. Move everything into `~/.claude/CLAUDE.md`.
- B. Split it into topic files in `.claude/rules/` (`testing.md`, `api-conventions.md`, `deployment.md`).
- C. Delete half the content.
- D. Convert every section into a slash command.

<details><summary>Answer</summary>

**B.** Topic-specific rule files are modular, easier to own and review. A removes sharing; D makes always-needed standards on-demand only. → [01](01-claude-md-hierarchy.md)
</details>

### A3
In a monorepo, the `billing` package needs Python and PCI standards; the `web` package needs TypeScript and accessibility standards. The standards already exist as files in `docs/standards/`. What is the most maintainable approach?

- A. Copy the standards text into each package's CLAUDE.md.
- B. In each package's CLAUDE.md, `@import` only the relevant standards files.
- C. Put all standards in the root CLAUDE.md.
- D. Tell developers to paste standards when needed.

<details><summary>Answer</summary>

**B.** `@import` reuses shared standards selectively, with no duplication. → [01](01-claude-md-hierarchy.md)
</details>

### A4
Claude follows a rule in some sessions and not others. What should you do first to diagnose it?

- A. Rewrite the rule in capital letters.
- B. Use `/memory` (and `/context`) to check which memory files are actually loaded.
- C. Reinstall Claude Code.
- D. Switch to plan mode.

<details><summary>Answer</summary>

**B.** Check which files load before changing content. Often the rule is in another scope or a subdirectory file that wasn't loaded. → [01](01-claude-md-hierarchy.md)
</details>

### A5
You want a `/review` command, running the team's standard checklist, that every developer gets automatically when they clone or pull. Where should it go?

- A. `~/.claude/commands/review.md`
- B. `.claude/commands/review.md` in the repository
- C. The root `CLAUDE.md`
- D. `.mcp.json`

<details><summary>Answer</summary>

**B.** Project-scoped commands are shared via version control. (A `.claude/skills/review/SKILL.md` would also work in current Claude Code.) → [02](02-slash-commands-and-skills.md)
</details>

### A6
Your `/analyze-codebase` skill reads dozens of files and outputs thousands of lines. Afterward, the rest of the session is cluttered and less accurate. What frontmatter change helps most?

- A. `argument-hint: [path]`
- B. `context: fork`
- C. `allowed-tools: Read`
- D. Remove the description.

<details><summary>Answer</summary>

**B.** Forked context runs the skill in an isolated subagent and returns only the result. → [02](02-slash-commands-and-skills.md)
</details>

### A7
A documentation skill once ran a destructive git command. It only needs to read source files and write docs. What should you add?

- A. `context: fork`
- B. `allowed-tools` limited to read and file-write tools (no general Bash).
- C. A warning in the skill's description.
- D. `argument-hint`.

<details><summary>Answer</summary>

**B.** Restrict tools during skill execution to prevent destructive actions. → [02](02-slash-commands-and-skills.md)
</details>

### A8
You want a stricter personal version of the team's `/review` skill without changing what teammates get. What should you do?

- A. Edit `.claude/skills/review/SKILL.md` and commit it.
- B. Create `~/.claude/skills/review-strict/SKILL.md` with a different name.
- C. Create `~/.claude/skills/review/SKILL.md` with the same name.
- D. Add your rules to the project CLAUDE.md.

<details><summary>Answer</summary>

**B.** Personal variants go in user scope with a **different name**. A and D change the team's behavior; C creates a confusing name clash. → [02](02-slash-commands-and-skills.md)
</details>

### A9
Developers often run `/release-notes` without the version range it needs, and the output is wrong. What helps most?

- A. `context: fork`
- B. `argument-hint: [from-tag] [to-tag]`
- C. Move the instructions into CLAUDE.md.
- D. `allowed-tools: Bash`

<details><summary>Answer</summary>

**B.** `argument-hint` prompts users for required parameters. → [02](02-slash-commands-and-skills.md)
</details>

### A10
A 70-line step-by-step "how to write a database migration" procedure is in CLAUDE.md. Migrations are written about twice a month. What is the better home for it?

- A. Keep it in CLAUDE.md.
- B. A skill (on-demand), e.g. `/db-migration`.
- C. `~/.claude/CLAUDE.md`.
- D. A comment in the migrations folder.

<details><summary>Answer</summary>

**B.** Task-specific workflows belong in skills (loaded on demand). CLAUDE.md is for always-relevant, universal standards. → [02](02-slash-commands-and-skills.md)
</details>

---

## Scenario B: Developer Productivity with Claude

*Engineers use Claude Code to explore, refactor, and extend a large codebase.*

### B1
Test files are spread across 40 directories and must all follow the same testing conventions. What is the most maintainable approach?

- A. Put a CLAUDE.md in each of the 40 directories.
- B. A `.claude/rules/tests.md` file with `paths: ["**/*.test.ts", "**/*.test.tsx"]`.
- C. Put the conventions in the root CLAUDE.md.
- D. A `/tests` slash command.

<details><summary>Answer</summary>

**B.** Glob-based path rules apply by file type regardless of location, in one file. A duplicates and drifts; C loads everywhere; D depends on someone remembering to run it. → [03](03-path-specific-rules.md)
</details>

### B2
Terraform conventions currently live in the root CLAUDE.md and load even during frontend work. What is the best change?

- A. Delete them.
- B. Move them to `.claude/rules/terraform.md` with `paths: ["terraform/**/*"]`.
- C. Move them to the user CLAUDE.md.
- D. Put them in `.mcp.json`.

<details><summary>Answer</summary>

**B.** Path-scoped rules load only when editing matching files, which saves tokens and avoids irrelevant context. → [03](03-path-specific-rules.md)
</details>

### B3
Which situation is a better fit for a subdirectory CLAUDE.md than for a path-scoped rule?

- A. Conventions for all `*.stories.tsx` files across the repo.
- B. Conventions specific to the `packages/payments/` package only.
- C. Conventions for all SQL migration files in many services.
- D. Conventions for every Dockerfile in the repo.

<details><summary>Answer</summary>

**B.** One area of the repo. A, C, and D are file types spread across directories, which suit path rules. → [03](03-path-specific-rules.md)
</details>

### B4
You must restructure a monolith into microservices, which involves service boundaries and module dependencies across dozens of files. Which approach is best?

- A. Direct execution, starting with the largest module.
- B. Plan mode to explore and design boundaries first, then execute the approved plan.
- C. Ask Claude to do it all in one prompt.
- D. Use `-p` mode.

<details><summary>Answer</summary>

**B.** Architectural, multi-file, multiple valid approaches → plan mode first, then execute. → [04](04-plan-mode-vs-direct-execution.md)
</details>

### B5
A single-file bug comes with a clear stack trace pointing to an off-by-one error. Which approach is best?

- A. Plan mode.
- B. Direct execution.
- C. Explore subagent first.
- D. Interview pattern.

<details><summary>Answer</summary>

**B.** Small, clear scope → direct execution. → [04](04-plan-mode-vs-direct-execution.md)
</details>

### B6
During a large multi-phase refactor, the main session fills with search results from discovery, and Claude starts losing track of earlier decisions. What helps most?

- A. Restart without any context.
- B. Use the Explore subagent for discovery so only summaries return to the main session.
- C. Paste all files into the prompt.
- D. Switch to direct execution.

<details><summary>Answer</summary>

**B.** The Explore subagent isolates verbose discovery and prevents context exhaustion. → [04](04-plan-mode-vs-direct-execution.md)
</details>

### B7
You describe a date-normalization transformation in a paragraph, and Claude's results vary between runs. What is the most effective next step?

- A. Write a longer paragraph.
- B. Provide 2–3 concrete input → output examples covering different cases.
- C. Switch to plan mode.
- D. Use a skill.

<details><summary>Answer</summary>

**B.** Concrete examples remove ambiguity that prose leaves. → [05](05-iterative-refinement.md)
</details>

### B8
You need to build a caching layer in a domain you don't know well. What should you do before implementation?

- A. Ask Claude to implement the simplest version immediately.
- B. Use the interview pattern: have Claude ask questions about invalidation, failure modes, and staleness first.
- C. Write the code yourself.
- D. Use `-p` mode.

<details><summary>Answer</summary>

**B.** The interview pattern surfaces considerations you might miss. → [05](05-iterative-refinement.md)
</details>

### B9
You found three bugs. Two involve the same timezone conversion logic and affect each other; the third is an unrelated typo. How should you send them?

- A. All three together.
- B. Each separately, in any order.
- C. The two related bugs together in one detailed message; the typo separately.
- D. Only the typo; the others will fix themselves.

<details><summary>Answer</summary>

**C.** Interacting issues together, independent issues separately. → [05](05-iterative-refinement.md)
</details>

### B10
Claude keeps mishandling NULL `middle_name` values in a migration script. What is the most effective feedback?

- A. "Please handle nulls better."
- B. A specific test case: input row with `middle_name: null` and the exact expected output.
- C. Switch models.
- D. Remove the column.

<details><summary>Answer</summary>

**B.** Specific test cases with input and expected output fix edge cases precisely. → [05](05-iterative-refinement.md)
</details>

---

## Scenario C: Claude Code for Continuous Integration

*You run Claude Code in CI to review pull requests and generate tests.*

### C1
Your pipeline runs `claude "Review this PR"` and the job hangs until timeout. Logs show it is waiting for input. What is the fix?

- A. Increase the CI timeout.
- B. Run with the `-p` flag.
- C. Use plan mode.
- D. Add `yes |` before the command.

<details><summary>Answer</summary>

**B.** `-p` runs non-interactively. → [06](06-ci-cd-integration.md)
</details>

### C2
A script must post each review finding as an inline PR comment, but parsing free-text reviews keeps breaking. What is the best approach?

- A. Better regex.
- B. `--output-format json` with `--json-schema` defining file, line, severity, and message.
- C. Ask Claude to "format nicely".
- D. Post the whole review as one comment.

<details><summary>Answer</summary>

**B.** Schema-enforced structured output is machine-parseable. → [06](06-ci-cd-integration.md)
</details>

### C3
After each new commit, the CI review reposts the same comments that were already addressed or already posted. What should you change?

- A. Run the review only once per PR.
- B. Include prior findings in context and instruct Claude to report only new or still-unaddressed issues.
- C. Delete all old comments before each run.
- D. Lower the severity threshold.

<details><summary>Answer</summary>

**B.** → [06](06-ci-cd-integration.md)
</details>

### C4
The same session that generated a feature is also asked to review it, and it misses obvious issues. What is the better design?

- A. Ask it to "review more carefully".
- B. Run the review in an independent Claude instance with fresh context.
- C. Skip the review.
- D. Use a longer prompt.

<details><summary>Answer</summary>

**B.** The authoring session is biased toward its own reasoning; an independent instance reviews better. → [06](06-ci-cd-integration.md)
</details>

### C5
CI test generation produces many low-value tests that duplicate existing ones and ignore the project's fixtures. Which combination helps most?

- A. Generate more tests and filter later.
- B. Provide existing test files in context, and document testing standards, valuable-test criteria, and available fixtures in CLAUDE.md.
- C. Use plan mode in CI.
- D. Limit output to 5 tests.

<details><summary>Answer</summary>

**B.** → [06](06-ci-cd-integration.md)
</details>

---

## Score yourself

| Score | Next step |
|---|---|
| 22–25 | Ready for Domain 3. Revisit any you guessed. |
| 17–21 | Re-read the files linked from your wrong answers. |
| < 17 | Re-read files 00–06 in order, then try again in a few days. |

---

## Key Takeaways

- Scope bugs: **team config must be in the project**, not `~/.claude/`.
- Placement: **CLAUDE.md (always)**, **path rules (by file type)**, **skills/commands (on demand)**.
- Skill frontmatter: **`context: fork`** (isolate), **`allowed-tools`** (restrict), **`argument-hint`** (prompt for args).
- **Plan** big or uncertain work; **execute** small, clear work; **Explore** to protect context.
- Refinement: **examples, tests, interviews**, and **batch interacting issues**.
- CI: **`-p`**, **JSON + schema**, **CLAUDE.md context**, **independent reviewer**, **no duplicate findings**, **existing tests in context**.
