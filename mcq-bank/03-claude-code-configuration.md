# Domain 3 MCQ Bank: Claude Code Configuration & Workflows

> 48 questions (12 Easy, 19 Medium, 17 Hard) covering task statements 3.1-3.6 · [Study notes](../domain-3-claude-code-configuration-workflows/README.md) · [Anti-patterns](../domain-3-claude-code-configuration-workflows/anti-patterns.md) · [Bank index](README.md)

## Questions

### Q1 · Task 3.4 · Easy

A developer at Larkspur Logistics must change the API request timeout from 30 to 60 seconds. The value lives on one known line in config/http.yaml. No other code reads the value directly.

Which approach is BEST?

- A. Start in plan mode so Claude can map every caller before the edit.
- B. Use the Explore subagent to search for timeout usage, then decide.
- C. Use direct execution and ask Claude to change the value.
- D. Write a plan document first, then switch to direct execution.

### Q2 · Task 3.1 · Medium

The project CLAUDE.md at Brightpath Insurance says: "Never run `terraform apply`." During a long session, Claude still tried to run it once. The team wants this rule to hold every time, for every developer.

What is the BEST change?

- A. Add a deny rule for `Bash(terraform apply *)` in .claude/settings.json.
- B. Repeat the instruction in CLAUDE.md in capitals and mark it as IMPORTANT.
- C. Move the instruction into a .claude/rules/ file with paths for *.tf files.
- D. Copy the instruction into each developer's ~/.claude/CLAUDE.md as well.

### Q3 · Task 3.6 · Hard

A CI pipeline at Quillmark runs `claude -p` to generate a database migration script for each PR. The review must catch subtle logic errors before merge. To save cost, an engineer proposes to resume the same session. The new prompt would be "Now review the code you just wrote". The budget allows one extra model run per PR.

What is the BEST way to run the review?

- A. Resume the generating session and ask it to review with extended thinking on.
- B. Resume the generating session, but add strict review criteria with --append-system-prompt.
- C. Review in the same session, then run a second pass on the files it flags.
- D. Start a new claude -p run that gets only the diff and the review criteria.

### Q4 · Task 3.2 · Easy

The team at Harbor Analytics wants a /release-notes skill. Everyone who clones the repository should get it automatically.

Where should the developer create it?

- A. .claude/skills/release-notes.md
- B. .claude/skills/release-notes/SKILL.md
- C. ~/.claude/skills/release-notes/SKILL.md
- D. A "Release notes" section in the project CLAUDE.md

### Q5 · Task 3.3 · Medium

In a React monorepo at Tidewell, custom hooks live in files such as cart.hook.ts and auth.hook.tsx. These files are spread across 30 feature folders, not one hooks folder. The team is writing .claude/rules/hooks.md for them.

Which `paths` value BEST matches only these files?

- A. ["src/hooks/**/*.ts", "src/hooks/**/*.tsx"]
- B. ["**/*.hook.ts", "**/*.hook.tsx"]
- C. ["**/*.ts", "**/*.tsx"]
- D. ["src/*.hook.ts", "src/*.hook.tsx"]

### Q6 · Task 3.5 · Hard

Claude built a CSV (comma-separated values) importer for billing exports at Pinecrest Clinics. There are three problems. First, the date parser reads 03/04 as March 4, but the files are day-first. Second, amounts are rounded before currency conversion, and the conversion uses an exchange rate looked up by the parsed date. Third, a progress log message has a typo.

What is the BEST way to give feedback?

- A. Send all three issues in one message so Claude sees the full picture.
- B. Send each issue in a separate message, starting with the log typo.
- C. Report the date issue first, and report amounts only if they still fail.
- D. Send the date and amount issues together, then the log typo on its own.

### Q7 · Task 3.1 · Easy

A developer at Cobalt Labs wants Claude to end each task with a three-line summary. They want this in every repository on their laptop. Their teammates should not be affected.

Where should this instruction go?

- A. ~/.claude/CLAUDE.md
- B. ./CLAUDE.md in each repository
- C. CLAUDE.local.md in the current repository
- D. .claude/CLAUDE.md in each repository

### Q8 · Task 3.2 · Medium

The team at Ridgeway Payments put a 14-step incident postmortem checklist in the project CLAUDE.md. They run it about twice a month. Since then, Claude sometimes starts postmortem steps during normal coding tasks. Every session also loads the long checklist.

What is the BEST fix?

- A. Add a line to CLAUDE.md: "Only use the postmortem steps when asked."
- B. Move the checklist to a .claude/rules/ file with no paths frontmatter.
- C. Move the checklist into a project skill in .claude/skills/postmortem/.
- D. Move the checklist into each on-call engineer's ~/.claude/CLAUDE.md file.

### Q9 · Task 3.4 · Hard

At Sagebrook Retail, the lead must move session storage for four services from an in-memory cache to a shared database. The team already knows the change touches login, cart, and checkout, and needs a data migration. The lead proposes starting in direct execution for speed and switching to plan mode if problems appear. The deadline is two weeks.

What should the architect recommend?

- A. Keep the lead's approach, but commit after each file so mistakes are easy to revert.
- B. Use plan mode only for the first service, then repeat that pattern directly for the rest.
- C. Use direct execution with the Explore subagent so discovery does not fill the context.
- D. Start in plan mode, agree on the design across all four services, then execute.

### Q10 · Task 3.3 · Hard

The root CLAUDE.md at Velo Health grew to 1,100 lines. Most of it covers database migrations, GraphQL schemas, and Kotlin mobile code. The team split it into five files and added @path lines in CLAUDE.md. Context use at session start did not drop, even for developers who only touch web code.

What change will MOST reduce startup context while keeping the guidance?

- A. Move each file-type section into .claude/rules/ files with matching paths globs.
- B. Nest the @path imports one level deeper so they load only when needed.
- C. Turn each file-type section into a skill that developers run before editing.
- D. Move the file-type sections into CLAUDE.local.md so each person loads their own.

### Q11 · Task 3.6 · Medium

CI at Juniper Logistics runs `claude -p` to rate the risk of each PR. A script reads the text output with a regular expression. The script breaks when Claude adds a short intro sentence. The engineer added --output-format json, but the risk value still sits inside free text.

What is the BEST next change?

- A. Add --append-system-prompt telling Claude to reply with JSON only.
- B. Switch to --output-format stream-json and parse each event.
- C. Add --json-schema with the risk fields and read structured_output.
- D. Keep the regex but make it skip any lines before the first brace.

### Q12 · Task 3.5 · Easy

A developer reviews Claude's work on a settings page. They find three unrelated problems: a wrong button color token, a typo in a tooltip, and a missing aria-label on a separate icon. None of the fixes affect each other.

Which feedback approach BEST fits?

- A. Batch all three in one message, because together they give more context.
- B. Give the issues one at a time, and check each fix before the next.
- C. Use the interview pattern so Claude asks about each issue first.
- D. Write tests for all three issues before giving any feedback.

### Q13 · Task 3.1 · Hard

The project CLAUDE.md at Kestrel Bank says to use 2-space indentation. One developer's ~/.claude/CLAUDE.md still says to use tabs, from an older job. For that developer, Claude mixes both styles, while other developers see no problem. The team wants correct indentation for every developer, every time.

What is the BEST long-term fix?

- A. Run the project formatter through a hook in the shared project settings.
- B. Rely on the project file, since project-level CLAUDE.md overrides user-level files.
- C. Add "This file overrides all other CLAUDE.md files" to the project CLAUDE.md.
- D. Copy the indentation rule into a CLAUDE.md inside every source folder.

### Q14 · Task 3.2 · Medium

A developer at Fernhill Media wants a /standup command that summarizes their own commits from the last day. They use it in all eight repositories they work on. Their teammates have other habits and do not want it.

Where should the developer create it?

- A. .claude/skills/standup/SKILL.md in each of the eight repositories
- B. A "Standup" section in each repository's CLAUDE.local.md file
- C. .claude/commands/standup.md in the team's shared template repository
- D. ~/.claude/skills/standup/SKILL.md in their home directory

### Q15 · Task 3.4 · Medium

An engineer at Orchard Pay must add retry with exponential backoff to one HTTP client function. The ticket gives exact rules: five tries at most, a 200-millisecond base delay, and retries only on 503 and timeouts. The engineer finds the math tricky and plans to use plan mode.

Which approach is BEST?

- A. Plan mode, because the backoff math makes the task complex.
- B. Plan mode with the Explore subagent to find every HTTP caller.
- C. Direct execution, because the scope and requirements are already clear.
- D. Direct execution, but only after an interview about retry needs.

### Q16 · Task 3.3 · Easy

The team at Alder Systems adds .claude/rules/migrations.md. Its frontmatter is `paths: ["db/migrations/**/*.sql"]`.

When does Claude load this rule?

- A. At the start of every session, along with CLAUDE.md.
- B. When Claude reads or edits a file that matches the glob.
- C. Only when a developer runs /memory and selects the file.
- D. Only when a developer types /migrations as a command.

### Q17 · Task 3.5 · Medium

Claude must convert fixed-width bank statement files into ledger entries for Copperline Accounting. There are many edge cases: reversals, split payments, multi-currency lines, and blank memo fields. After two rounds of feedback, each fix breaks a case that worked before.

What is the MOST effective approach?

- A. Write a longer prose spec that lists every edge case in detail.
- B. Give two input/output examples of normal lines and let Claude infer the rest.
- C. Write edge-case tests first, then share the failing output each round.
- D. Ask Claude to interview you about the statement format before coding.

### Q18 · Task 3.2 · Hard

The team at Wrenfield is building a /new-service skill with 40 lines of instructions. It also needs a 600-line service template. Claude needs the template only in the last step, and many runs stop earlier. The lead wants the template shared in git and kept out of context until it is needed.

Where should the template go?

- A. In the skill folder next to SKILL.md, referenced from the instructions.
- B. Pasted at the end of SKILL.md so all skill content stays in one file.
- C. In the project CLAUDE.md, pulled in with an @path line.
- D. In ~/.claude/skills/new-service/ so the large template does not bloat the repository.

### Q19 · Task 3.6 · Easy

The CI review job at Rowan Tech needs two extra rules: flag hardcoded URLs and flag missing error handling. The team wants to keep all of Claude Code's default behavior and tools.

Which option BEST fits?

- A. --system-prompt with the new rules
- B. --output-format json with the rules in a schema
- C. --permission-mode plan with the rules in the prompt
- D. --append-system-prompt with the new rules

### Q20 · Task 3.1 · Medium

In the Lumen Robotics monorepo, packages/firmware/CLAUDE.md contains the line `@docs/firmware-style.md`. The team meant the file at the repository root, docs/firmware-style.md. Claude never seems to follow the style guide while working in firmware.

What is the MOST likely cause?

- A. @path lines only work in the root CLAUDE.md, so imports in subfolder files are ignored.
- B. The path resolves from packages/firmware/, so it points to a missing file.
- C. Imports need the @import keyword, so the line is treated as plain text.
- D. Subfolder CLAUDE.md files never load until a developer runs /memory.

### Q21 · Task 3.4 · Hard

A ticket at Bluefin Travel has two parts. First, a pagination off-by-one bug, where the stack trace points to one line in one function. Second, "improve search caching," with several possible designs that affect three services. The developer has one afternoon.

What is the BEST way to handle the ticket?

- A. Fix the pagination bug directly, then use plan mode for the caching work.
- B. Use plan mode for both parts so they share one reviewed plan.
- C. Use direct execution for both parts, starting with the caching work.
- D. Fix both in one direct session, and ask Claude to explain each design choice it makes.

### Q22 · Task 3.3 · Medium

At Mossgate Software, someone copied the same testing CLAUDE.md into 18 test folders. Over time the copies drifted, and now they give different mocking advice. Test files sit in many folders, and all end in .spec.ts.

What is the BEST way to consolidate?

- A. Keep the copies and add a CI check that they stay identical.
- B. Move the testing rules into the root CLAUDE.md for every session.
- C. Create one .claude/rules/testing.md with paths: ["**/*.spec.ts"].
- D. Create a /testing-rules skill that developers run before writing tests.

### Q23 · Task 3.1 · Hard

Early in a long session at Marlow Freight, a developer gave Claude a rule in chat. The rule was "Always use our logger wrapper, never console.log". It worked for an hour. After /compact, Claude started using console.log again. This is a team standard, and new teammates hit the same problem.

What is the LOWEST-EFFORT fix that lasts?

- A. Repeat the instruction in chat after each /compact.
- B. Run /memory after each /compact so the instruction reloads.
- C. Add the instruction to each developer's CLAUDE.local.md file.
- D. Add the instruction to the project CLAUDE.md and commit it.

### Q24 · Task 3.5 · Medium

Claude is building an address formatter for shipping labels at Elmstone Shipping. The developer gave three input/output examples of street addresses. Output is now right for those cases. But post office box (PO box) addresses get a wrong street-number field.

What is the BEST next step?

- A. Rewrite the prose description to explain PO boxes in more detail.
- B. Add an input/output example that shows a PO box address.
- C. Replace the examples with a full interview about address formats.
- D. Ask Claude to be more careful with unusual address types.

### Q25 · Task 3.2 · Medium

The team skill /schema-diff at Hollis Data compares database schemas. Its frontmatter description is just "Helper." When developers ask "what changed in the orders schema?", Claude never uses the skill. It works when typed as /schema-diff.

What is the BEST fix?

- A. Add an argument-hint so Claude knows what input to pass.
- B. Set context: fork so the skill runs in its own context.
- C. Move the skill to ~/.claude/skills so Claude finds it sooner.
- D. Write a description that says what the skill does and when to use it.

### Q26 · Task 3.6 · Medium

CI at Cedar Grove runs `claude -p` to review PRs and post comments. Security requires that this job never changes files in the checkout. Someone proposes a CLAUDE.md line: "In CI, do not edit files."

What is the BEST way to meet the requirement?

- A. Pass --disallowedTools to remove Edit, Write, and Bash in the CI command.
- B. Keep the CLAUDE.md line and add the same text with --append-system-prompt.
- C. Use --system-prompt to replace the defaults with a review-only identity.
- D. Add --max-turns 1 so Claude stops before it can reach any file edit step.

### Q27 · Task 3.1 · Easy

In one repository, a developer at Glenbrook runs a local mock server on port 4100. They also want Claude to run only the fast unit-test target. These preferences are just for them and just for this repository.

Where should these notes go?

- A. ~/.claude/CLAUDE.md
- B. CLAUDE.local.md in the repository root
- C. ./CLAUDE.md in the repository root
- D. .claude/rules/local.md with no paths

### Q28 · Task 3.3 · Hard

The root CLAUDE.md at Thornbury Insurance has three parts. Part one holds naming and error-handling standards for all code. Part two is a quarterly release checklist run on demand. Part three holds rules for *.proto files in 20 service folders. Sessions start with heavy context, and Claude sometimes follows the checklist unprompted.

Which restructure is BEST?

- A. Keep part one in CLAUDE.md; move parts two and three into path-scoped rules.
- B. Keep part one in CLAUDE.md; move parts two and three into skills.
- C. Keep part one in CLAUDE.md; make part two a skill and part three a path rule.
- D. Split all three parts into separate files and pull them in with @path lines.

### Q29 · Task 3.4 · Medium

A product manager at Kiteline asks Claude to "make the reports dashboard faster". No one knows yet if the slow part is the queries, the API, or the front end. The change could be small, or it could span many files.

How should the developer start?

- A. Plan mode, to investigate causes and compare options before edits.
- B. Direct execution, since speed fixes are usually small changes.
- C. Direct execution, adding a database index on the largest table first.
- D. Plan mode, then let Claude apply each fix as soon as it finds one.

### Q30 · Task 3.5 · Hard

A developer at Saltmarsh Freight must build a freight-rate calculator. They do not know the domain: dimensional weight, fuel surcharges, and zone tables. Delivery is in three days, so they want to start with Claude now. They cannot write tests or examples yet, because they do not know the right outputs.

What is the BEST first step?

- A. Give Claude a best-guess spec and correct the code as problems show up.
- B. Write a detailed prose spec from online research before involving Claude.
- C. Ask Claude to generate tests from its own knowledge, then code to them.
- D. Ask Claude to interview you first, to surface points you had not considered.

### Q31 · Task 3.2 · Easy

A skill author at Palisade writes a skill that uses `Bash(git log:*)` and `Read`. They want the skill file itself to list these tools.

Which frontmatter field holds this list?

- A. argument-hint
- B. allowed-tools
- C. context
- D. tools-required

### Q32 · Task 3.1 · Medium

Ashford Media keeps its API style rules in docs/api-style.md, which the platform team maintains. The project CLAUDE.md should include that file without copying text, so updates flow through automatically.

What should the developer add to the project CLAUDE.md?

- A. The line `@import docs/api-style.md`
- B. A Markdown link to docs/api-style.md
- C. The line `@docs/api-style.md`
- D. A note telling Claude to read the doc before each task

### Q33 · Task 3.6 · Hard

CI at Birchway reviews each push to a PR with a fresh `claude -p` run. Developers say issues fixed two pushes ago get flagged again. The bot also cannot say which problems are resolved. Someone suggests reviewing only the latest commit's diff to cut noise and cost.

What is the BEST change?

- A. Review only the latest commit's diff, without earlier review results.
- B. Resume the session that generated the code so it remembers past issues.
- C. Pass the earlier findings in the prompt and report only new or open issues.
- D. Raise --max-turns so Claude can re-check the full history of the PR.

### Q34 · Task 3.4 · Easy

A developer at Fairhaven wants to start a Claude Code session in plan mode from the terminal.

Which command does this?

- A. claude --permission-mode plan
- B. claude --plan-only
- C. claude -p plan
- D. claude --mode readonly

### Q35 · Task 3.2 · Hard

The team skill /dead-code at Larchmont scans the whole repository. It lists hundreds of unused symbols, with reasoning for each one. After it runs, the main session is full of scan output, and later answers get worse. Developers still want to discuss the final list in the main chat.

What is the BEST change to the skill?

- A. Keep it in the main context and run /compact right after each scan.
- B. Move the scan instructions into CLAUDE.md so Claude runs them automatically.
- C. Split the skill into ten smaller skills, one per top-level folder.
- D. Add context: fork and have the skill return a short ranked summary.

### Q36 · Task 3.3 · Easy

A developer creates .claude/rules/security.md but forgets to add `paths` frontmatter.

What happens?

- A. The rule never loads until a paths field is added to its frontmatter.
- B. The rule loads at session start, like CLAUDE.md content.
- C. The rule loads only when Claude edits a security file.
- D. Claude Code rejects the file and shows an error.

### Q37 · Task 3.5 · Medium

A developer at Quarry Labs knows the exact output they want. Log lines like "2024-05-01T10:00Z ERROR payment id=88" must become a JSON object with four fields. Claude's attempts from a prose description use different field names each time. A teammate suggests the interview pattern.

What is the BEST approach?

- A. Give two or three concrete input/output examples of the conversion.
- B. Use the interview pattern so Claude asks about the log format.
- C. Rewrite the prose description more precisely and run it again.
- D. Ask Claude to explain its field choices before each attempt.

### Q38 · Task 3.1 · Hard

At Pembrook Health Billing, a senior developer's Claude always uses the team's error-response format. Teammates' Claude does not, even though they cloned the same repository. The senior keeps that rule in CLAUDE.local.md, next to personal notes. The lead suspects the problem is that teammates use different code editors.

What is the BEST fix?

- A. Ask teammates to copy the rule into their own ~/.claude/CLAUDE.md.
- B. Remove CLAUDE.local.md from .gitignore and commit it, so everyone gets the whole file.
- C. Ask teammates to run /memory so the senior's file is loaded for them.
- D. Move the format rule into the project CLAUDE.md; keep personal notes local.

### Q39 · Task 3.6 · Medium

Dovetail has three CI pipelines: review, test generation, and docs checks. Each one passes the same 60 lines of project conventions with --append-system-prompt. The copies drift, and local developer sessions do not get the conventions at all.

What is the BEST change?

- A. Move the conventions into one shared --system-prompt file for all pipelines.
- B. Move the conventions into the project CLAUDE.md, which CI runs also load.
- C. Keep the flags, and add a CI step that checks the three copies match.
- D. Put the conventions in each developer's ~/.claude/CLAUDE.md file.

### Q40 · Task 3.4 · Hard

The architect at Elkhorn must find every use of a deprecated payments client across 12 services. Discovery will read hundreds of files. A developer proposes: "Let the Explore subagent find and fix each call in one pass to save time."

What is the BEST plan?

- A. Let the Explore subagent find and edit the calls, then review the diff.
- B. Skip subagents and run all discovery in the main session in plan mode.
- C. Use Explore to find and summarize calls, agree a plan, then edit in main.
- D. Use direct execution across all services, fixing calls as Claude finds them.

### Q41 · Task 3.1 · Easy

A new developer at Vantage runs /memory. Then they say: "Good, now my CLAUDE.md files are loaded."

Which statement is accurate?

- A. /memory loads every CLAUDE.md file in the repository at once.
- B. /memory lists the memory files and opens them for editing.
- C. /memory reloads the CLAUDE.md files that /compact removed.
- D. /memory saves the current chat as a new CLAUDE.md file.

### Q42 · Task 3.2 · Medium

The Grayhawk repository has eight files in .claude/commands/, such as deploy-check.md. A new hire says these files are dead because skills replaced commands. They want to delete them.

Which statement is accurate?

- A. They still work; skills add a folder format with SKILL.md and extra files.
- B. They stopped working; each one must move to a flat .claude/skills/<name>.md file.
- C. They still work, but only from ~/.claude/commands, not the project.
- D. They work only if CLAUDE.md lists each one with an @path line.

### Q43 · Task 3.3 · Hard

Fairpoint has .claude/rules/api.md with `paths: ["services/api/**/*.ts"]`. A developer opens a fresh session and asks Claude to draft a plan for three new endpoints. The plan ignores the rule's naming scheme. Later, when Claude edits services/api/routes.ts, it follows the rule.

What is the MOST likely cause?

- A. The glob is wrong, because ** does not match nested folders.
- B. The root CLAUDE.md overrode the rule because it loads first.
- C. Rules in .claude/rules/ need an @path line in CLAUDE.md.
- D. The rule loads only after Claude reads or edits a matching file.

### Q44 · Task 3.6 · Easy

A nightly CI job at Ironleaf runs `claude -p` to fix lint errors. Sometimes Claude loops through many tool calls, and the job runs for an hour.

Which flag sets a limit on agent turns?

- A. --output-format json
- B. --append-system-prompt
- C. --max-turns
- D. --disallowedTools

### Q45 · Task 3.5 · Hard

Claude wrote a tax-rounding module for Brookline Retail, and 6 of 40 tests fail. The developer is about to paste the full 3,000-line test log with "tests fail, please fix". A teammate suggests asking Claude to rewrite the module from scratch instead.

What is the BEST next message?

- A. The full 3,000-line log, so Claude has every detail.
- B. The six failing test names with expected and actual values.
- C. A request to rewrite the module from scratch, with more care.
- D. A longer prose spec that restates the rounding rules.

### Q46 · Task 3.1 · Medium

Stonebridge keeps its team rules in .claude/CLAUDE.md, committed to git. There is no CLAUDE.md at the repository root. A developer says Claude cannot see the rules and wants to move the file.

Which statement is accurate?

- A. .claude/CLAUDE.md is a valid project location, so a move is not needed.
- B. Only a root CLAUDE.md loads, so the file must move to the root.
- C. .claude/CLAUDE.md loads only when Claude works on files inside the .claude folder.
- D. .claude/CLAUDE.md is the user-level location and is not shared.

### Q47 · Task 3.2 · Hard

The lead at Ravenmoor wants two things for the whole team. First, a /changelog workflow that drafts release notes on demand. Second, Claude should use the conventional-commit format for every commit message during normal work.

Which setup is BEST?

- A. Both as project skills in .claude/skills/, shared through git.
- B. Both as sections in the project CLAUDE.md, shared through git.
- C. /changelog as a project skill; the commit format in project CLAUDE.md.
- D. /changelog in ~/.claude/skills/; the commit format in project CLAUDE.md.

### Q48 · Task 3.4 · Medium

Southport must move from an old date library to a new one in 35 files. The swap patterns are mostly known. But some files handle time zones in different ways.

Which approach is BEST?

- A. Direct execution, since the replacement patterns are already known.
- B. Plan mode for the whole job, including all the file edits.
- C. Separate direct sessions, one per file, with no shared plan.
- D. Plan mode to map the files and approach, then direct execution.

## Answer key

### Q1 · C

**Why C is correct:** The change is one value in one known file with no unclear scope. Direct execution fits small, well-defined edits. Plan mode adds overhead with no benefit here.

**Closest distractor (A):** Plan mode is for unclear scope or many valid approaches, and neither applies to a one-line config change.

### Q2 · A

**Why A is correct:** CLAUDE.md is guidance, so Claude can still miss it. A deny rule in settings.json permissions is enforced by Claude Code every time. Project settings also apply to every developer who clones the repository.

**Closest distractor (B):** Stronger wording is still only guidance and cannot guarantee the command is blocked.

### Q3 · D

**Why D is correct:** A session that wrote the code keeps its own reasoning and tends to agree with it. A separate run with only the diff and criteria reviews the code with fresh eyes. It uses the one extra run the budget allows.

**Closest distractor (B):** Better criteria help, but the resumed session still carries the generator's context and its blind spots.

### Q4 · B

**Why B is correct:** A project skill is a folder under .claude/skills/ with a SKILL.md file inside. It is committed to git, so everyone who clones gets it.

**Closest distractor (A):** A flat .md file placed directly in .claude/skills/ is not a valid skill; a skill must be a folder with SKILL.md.

### Q5 · B

**Why B is correct:** The files share a name pattern but live in many folders. A recursive glob on the *.hook.ts and *.hook.tsx suffix matches them wherever they are and nothing else.

**Closest distractor (A):** It assumes one hooks folder, so it misses the files spread across the 30 feature folders.

### Q6 · D

**Why D is correct:** The date and amount issues interact, because the conversion depends on the parsed date. Interacting issues should go together in one message. The independent log typo is best handled on its own.

**Closest distractor (A):** Batching is right for the linked issues, but adding the unrelated typo mixes independent feedback into the same change.

### Q7 · A

**Why A is correct:** A personal preference for all projects belongs in the user-level file. ~/.claude/CLAUDE.md applies on that machine only and is not shared through git.

**Closest distractor (C):** CLAUDE.local.md is personal, but it applies to one repository, not to every repository on the laptop.

### Q8 · C

**Why C is correct:** An occasional workflow belongs in a skill that runs on demand. That removes the checklist from every session's context. It also stops Claude from starting the steps during normal work.

**Closest distractor (B):** A rule file without paths loads at session start, so the checklist stays always-on.

### Q9 · D

**Why D is correct:** The task is already known to be complex, with cross-service design and a data migration. Plan mode should come first, not after problems appear. A plan across all four services keeps the design consistent.

**Closest distractor (B):** Planning only the first service misses cross-service decisions, such as how the migration and shared data work together.

### Q10 · A

**Why A is correct:** @path imports still load at session start, so splitting the file did not cut context. Rule files with paths globs load only when Claude reads or edits matching files. Web-only developers then stop paying for mobile and database rules.

**Closest distractor (B):** Nested imports still load with the parent file at startup, so context use stays the same.

### Q11 · C

**Why C is correct:** --output-format json alone wraps the reply, but the answer is still free text. Adding --json-schema makes Claude return data that matches the schema. The script then reads the structured_output field.

**Closest distractor (A):** A prompt instruction asks for JSON but does not guarantee a valid shape the script can trust.

### Q12 · B

**Why B is correct:** The three problems are independent. Sequential feedback lets the developer check each fix on its own. Batching is best kept for issues that affect each other.

**Closest distractor (A):** Batching fits interacting issues; for independent fixes it adds no value and makes each change harder to check.

### Q13 · A

**Why A is correct:** When CLAUDE.md files conflict, no level is sure to win. A formatter run by a hook enforces the style in code, for every developer, every time. It does not depend on which instructions Claude follows.

**Closest distractor (B):** There is no guaranteed precedence between CLAUDE.md levels; conflicting instructions can be followed unpredictably.

### Q14 · D

**Why D is correct:** A personal command used across many repositories belongs in the user-level skills folder. ~/.claude/skills/ is available in every project on that machine. It is not shared with teammates.

**Closest distractor (A):** Project skills are committed to git, so teammates would get a command they do not want.

### Q15 · C

**Why C is correct:** The task is hard but its scope and rules are fully clear. Unclear scope decides plan mode, not difficulty. Direct execution is the efficient choice for a clear one-function change.

**Closest distractor (A):** It confuses difficulty with unclear scope; tricky math alone does not need a planning phase.

### Q16 · B

**Why B is correct:** A rule with paths frontmatter is path-scoped. Claude loads it only when it reads or edits a file that matches the glob. This keeps the rule out of unrelated sessions.

**Closest distractor (A):** That describes CLAUDE.md and rules without paths, not a path-scoped rule.

### Q17 · C

**Why C is correct:** Complex transformations with many edge cases suit test-driven iteration. Tests lock in each case, so a later fix cannot silently break an earlier one. Sharing the failing results guides each round.

**Closest distractor (B):** Examples of normal lines help with format, but they do not protect the many edge cases from regressions.

### Q18 · A

**Why A is correct:** A skill is a folder, so it can hold supporting files beside SKILL.md. The instructions can point to the template, and Claude reads it only in the last step. The folder is in the project, so git shares it.

**Closest distractor (B):** The whole SKILL.md loads when the skill runs, so the 600 lines enter context even in runs that stop early.

### Q19 · D

**Why D is correct:** --append-system-prompt adds the new rules on top of Claude Code's default system prompt. Default behavior and tools stay in place.

**Closest distractor (A):** --system-prompt replaces the whole default prompt, so default behavior would be lost.

### Q20 · B

**Why B is correct:** @path imports resolve relative to the file that contains them. From packages/firmware/, the line points to packages/firmware/docs/, which does not exist. The path needs to reach the root docs folder.

**Closest distractor (C):** There is no @import keyword; the @path form is correct, only the relative path is wrong.

### Q21 · A

**Why A is correct:** The pagination bug is a clear one-place fix, so direct execution fits. The caching work has several valid designs across three services, so it needs plan mode. Matching the mode to each part also fits the short time.

**Closest distractor (B):** Plan mode for a one-line bug with a clear stack trace adds overhead and delays the easy fix.

### Q22 · C

**Why C is correct:** One rule file replaces 18 drifting copies. The paths glob matches test files in any folder. The rule loads only when Claude works with those files.

**Closest distractor (A):** A sync check stops drift, but it keeps 18 copies to maintain instead of one source of truth.

### Q23 · D

**Why D is correct:** Instructions given only in chat can be lost when /compact summarizes the history. The project CLAUDE.md is loaded from disk again, so the rule persists. Committing it also helps new teammates.

**Closest distractor (C):** CLAUDE.local.md is personal and not in git, so every developer would need to copy the rule by hand.

### Q24 · B

**Why B is correct:** The examples already work for normal cases. Adding one targeted example for the missed edge case is the most direct fix. It shows Claude the exact expected output.

**Closest distractor (A):** Rewriting prose more carefully is the pattern that examples were meant to replace.

### Q25 · D

**Why D is correct:** Claude decides when to use a skill on its own based on its description. "Helper." gives no signal about schema comparisons. A clear what-and-when description lets Claude match related requests.

**Closest distractor (A):** argument-hint tells a user what to type after the command; it does not help Claude choose the skill.

### Q26 · A

**Why A is correct:** --disallowedTools removes the listed tools from the run. Without Edit, Write, and Bash, Claude has no tool that changes files, whatever the prompt says. A hard security rule needs that kind of enforcement.

**Closest distractor (B):** Prompt text and CLAUDE.md lines are guidance only, so they cannot guarantee that no file changes.

### Q27 · B

**Why B is correct:** CLAUDE.local.md holds personal notes for one repository. It is normally kept out of git, so teammates are not affected.

**Closest distractor (A):** ~/.claude/CLAUDE.md applies to every repository, not just this one.

### Q28 · C

**Why C is correct:** Each part goes to the mechanism that fits it. Global standards stay in CLAUDE.md. The on-demand checklist becomes a skill, and the *.proto rules become a path-scoped rule.

**Closest distractor (A):** A path rule fits the .proto rules, but a release checklist is an on-demand workflow, not file-type guidance.

### Q29 · A

**Why A is correct:** The request is simple to say but its scope is unclear. Plan mode lets Claude investigate and compare options before any edits. The developer then picks an approach with evidence.

**Closest distractor (C):** It guesses a cause before anyone knows where the slowness comes from.

### Q30 · D

**Why D is correct:** The developer does not know the domain well enough to write tests or examples. The interview pattern has Claude ask questions first. This surfaces design points, such as surcharge rules, before any code is built.

**Closest distractor (A):** Iterating on a guessed spec in an unknown domain spends the short deadline on rework.

### Q31 · B

**Why B is correct:** allowed-tools is the skill frontmatter field that lists tools for the skill. The author can name tools such as specific Bash commands and Read there.

**Closest distractor (D):** tools-required is not a skill frontmatter field.

### Q32 · C

**Why C is correct:** CLAUDE.md pulls in another file with an @path line. The content loads from the source file, so platform team updates flow through.

**Closest distractor (A):** There is no @import keyword; the syntax is @ followed directly by the path.

### Q33 · C

**Why C is correct:** Each fresh run has no memory of earlier reviews. Passing the prior findings lets Claude tell which issues are fixed. It then reports only new or still-open problems.

**Closest distractor (A):** A diff-only review cuts noise, but it still cannot say which earlier issues are resolved.

### Q34 · A

**Why A is correct:** --permission-mode plan starts the session in plan mode. Shift+Tab and /plan are other ways to enter it during a session.

**Closest distractor (C):** -p runs a non-interactive print-mode query; here "plan" would just be the prompt text.

### Q35 · D

**Why D is correct:** context: fork runs the skill in a separate context. The long scan output stays there. Only the short summary returns to the main chat, where developers can discuss it.

**Closest distractor (A):** /compact cleans up after the damage and can also blur other useful context in the session.

### Q36 · B

**Why B is correct:** A rule file without paths frontmatter is not path-scoped. It loads at session start, like CLAUDE.md content.

**Closest distractor (A):** The paths field is optional; leaving it out makes the rule always-on rather than disabled.

### Q37 · A

**Why A is correct:** The developer already knows the exact target format. Two or three concrete input/output examples show it directly and remove the ambiguity. This is faster than any back-and-forth.

**Closest distractor (B):** The interview pattern is for unfamiliar domains, not for a change the developer can already show.

### Q38 · D

**Why D is correct:** CLAUDE.local.md is personal and not committed, so teammates never get it. A team standard belongs in the project CLAUDE.md. Personal notes can stay in the local file.

**Closest distractor (B):** Committing the local file shares the rule but also pushes the senior's personal notes to everyone.

### Q39 · B

**Why B is correct:** Claude Code runs in CI also load the project CLAUDE.md. Moving the conventions there gives one source of truth. Pipelines and local sessions then all see the same rules.

**Closest distractor (A):** --system-prompt replaces the default prompt and still does not reach local developer sessions.

### Q40 · C

**Why C is correct:** The Explore subagent is read-only, so it cannot edit files. It keeps the verbose discovery out of the main context and returns a summary. The main session then plans and makes the edits.

**Closest distractor (A):** The Explore subagent cannot write or edit files, so the proposed one-pass fix will not work.

### Q41 · B

**Why B is correct:** /memory shows which memory files are in use and opens them for editing. It does not load files. Loading happens through the normal CLAUDE.md rules.

**Closest distractor (C):** /memory does not reload anything; it only lists and opens memory files.

### Q42 · A

**Why A is correct:** Files in .claude/commands/ still work as commands. Skills add a folder format, with SKILL.md plus optional supporting files. The team can migrate later if it wants those features.

**Closest distractor (B):** They did not stop working, and a flat .md file in .claude/skills/ is not a valid skill anyway.

### Q43 · D

**Why D is correct:** A path-scoped rule loads only when Claude reads or edits a matching file. During planning, no API file had been opened yet. The rule loaded later, when Claude edited routes.ts.

**Closest distractor (A):** ** does match nested folders, and the later edit shows the glob works.

### Q44 · C

**Why C is correct:** --max-turns sets a cap on agent turns in print mode. It stops runaway loops in a CI job.

**Closest distractor (D):** --disallowedTools removes tools but does not limit how many turns Claude takes.

### Q45 · B

**Why B is correct:** Test-driven iteration works by sharing specific failures. The failing names with expected and actual values point Claude at the exact gaps. The 34 passing tests show most of the module is right.

**Closest distractor (A):** The full log buries six useful failures in thousands of lines of passing output.

### Q46 · A

**Why A is correct:** Project memory can live at ./CLAUDE.md or ./.claude/CLAUDE.md. Both are shared through git. Moving the file will not change whether it loads.

**Closest distractor (D):** The user-level file is ~/.claude/CLAUDE.md in the home directory, not .claude/CLAUDE.md in the repository.

### Q47 · C

**Why C is correct:** An on-demand team workflow fits a project skill. An always-on team standard fits the project CLAUDE.md. Both are committed, so the whole team gets them.

**Closest distractor (A):** A skill runs only when invoked, so it is the wrong tool for a rule that must apply during normal work.

### Q48 · D

**Why D is correct:** The migration spans many files and has open questions about time zones. Plan mode maps the files and settles the approach first. Direct execution then applies the agreed changes.

**Closest distractor (A):** Known patterns cover most files, but the time-zone differences need a plan before editing 35 files.
