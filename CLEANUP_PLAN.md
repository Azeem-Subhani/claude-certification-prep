# Cleanup Plan (Step 1: audit and plan)

**Status:** waiting for approval. Nothing has been deleted or moved yet.
**Date:** 2026-10-15
**Branch:** `exam-prep-overhaul`
**Goal:** one canonical folder per exam domain (`domain-N-*`), plus the MCQ bank and the cheat sheet. Nothing duplicated, no empty folders, no broken links.

---

## 0. Name check: what your folder names point to

Your request uses some names that differ from what is in the repo. I matched them like this:

| You wrote | What exists in the repo | Note |
|---|---|---|
| `domain` | `domains/` (5 files) | This is the folder that duplicates the `domain-N-*` folders. |
| `lab` | `labs/` | Only an empty `.gitkeep` file. |
| `notes` | `notes/` | Only an empty `.gitkeep` file. |
| `practice-questions` | `practice-questions/` | Only an empty `.gitkeep` file. Not the same as `mcq-bank/`. |
| `review-needed` | **No such folder.** There is a root **file**, `REVIEW_NEEDED.md`. | Decision needed (see §6, item A). |
| `docs` | `docs/` (4 files) | Holds important content. Decision needed (see §6, item B). |

---

## 1. Every folder and file

The PDF in the root is **not tracked** in git. Everything else listed here is tracked.

### Root

| Path | One-line summary |
|---|---|
| `.gitignore` | Ignores secrets, `.env` files, OS/editor files, and Python/Node build output. |
| `README.md` | Entry point: exam facts, how to study, study order, repo map, glossary. |
| `REVIEW_NEEDED.md` | Open items: 14 conflicts between sources, 6 claims to verify, 6 owner decisions. |
| `claude-architect-cheat-sheet.md` | Your cheat sheet converted from PDF, with 16 ⚠️ verify notes. |
| `Claude Certified Architect Foundations Exam Guide.pdf` | Official exam guide. **Untracked.** It will not be touched. |

### `domains/` (the duplicate folder, 5 files)

| Path | One-line summary |
|---|---|
| `domains/01-agentic-architecture.md` | Domain 1 summary in six sections. Has the agentic loop diagram and the loop failure table. |
| `domains/02-tool-design-mcp.md` | Domain 2 summary in six sections: descriptions, errors, `tool_choice`, MCP config, built-in tools. |
| `domains/03-claude-code-configuration.md` | Domain 3 summary in six sections: CLAUDE.md, settings/permissions, skills, rules, plan mode, CI. |
| `domains/04-prompt-engineering-structured-output.md` | Domain 4 summary in six sections: criteria, few-shot, schemas, retry, batch, review. |
| `domains/05-context-management-reliability.md` | Domain 5 summary in six sections: case facts, escalation, errors, degradation, calibration, provenance. |

### `docs/` (4 files)

| Path | One-line summary |
|---|---|
| `docs/audit.md` | One-time audit of the repo before the last update: file list and gaps. |
| `docs/domain-map.md` | How the 5 domains were derived, the 30 task statements, the 6 exam scenarios, and the out-of-scope list. |
| `docs/verification-log.md` | Every technical claim, checked against official docs, with the page URL and a verdict. |
| `docs/verification-report.md` | One-time quality report for the MCQ bank and domain files (Step 5 of the last update). |

### `mcq-bank/` (6 files; stays)

| Path | One-line summary |
|---|---|
| `mcq-bank/README.md` | Index, question counts, how to use, and quality-check results. |
| `mcq-bank/01-agentic-architecture.md` | 65 questions with answer key. |
| `mcq-bank/02-tool-design-mcp.md` | 43 questions with answer key. |
| `mcq-bank/03-claude-code-configuration.md` | 48 questions with answer key. |
| `mcq-bank/04-prompt-engineering-structured-output.md` | 48 questions with answer key. |
| `mcq-bank/05-context-management-reliability.md` | 36 questions with answer key. |

### Empty folders

| Path | One-line summary |
|---|---|
| `labs/.gitkeep` | Empty placeholder file. |
| `notes/.gitkeep` | Empty placeholder file. |
| `practice-questions/.gitkeep` | Empty placeholder file. |

### `domain-1-agentic-architecture-orchestration/` (10 files; canonical)

| File | One-line summary |
|---|---|
| `README.md` | Task statements 1.1 to 1.7 as Know/Skills lists, scenarios, sources, key themes. |
| `00-foundations-and-glossary.md` | Agent vs workflow, stateless Messages API, harness, Agent SDK, MCP intro, glossary. |
| `01-agentic-loops.md` | The loop, a `stop_reason` table (6 values), a Python loop, 4 anti-patterns, an ASCII (plain-text) diagram. |
| `02-multi-agent-orchestration.md` | Hub-and-spoke, isolated context, the coordinator's four jobs, narrow decomposition. |
| `03-subagent-configuration.md` | Task/Agent tool, `AgentDefinition`, context passing, parallel spawning, goals not scripts. |
| `04-workflow-enforcement-and-handoffs.md` | Prompt vs code enforcement, prerequisite gates, multi-issue requests, human handoff. |
| `05-agent-sdk-hooks.md` | `PreToolUse` blocking, `PostToolUse` normalization, hooks vs prompts. |
| `06-task-decomposition.md` | Prompt chaining vs adaptive decomposition, per-file plus cross-file passes. |
| `07-session-state-resumption-forking.md` | Resume, fork, fresh start with summary, stale context. |
| `08-scenario-practice-questions.md` | 25 older, easier practice questions. |

### `domain-2-tool-design-mcp-integration/` (8 files; canonical)

| File | One-line summary |
|---|---|
| `README.md` | Task statements 2.1 to 2.5, scenarios, sources, key themes. |
| `00-foundations-and-glossary.md` | Tool definition parts, MCP roles, transports, tools vs resources vs prompts, glossary. |
| `01-tool-interface-design.md` | Descriptions as the selection signal, overlap, rename/split, system prompt keywords. |
| `02-structured-error-responses.md` | `isError`, protocol vs execution errors (matches the MCP spec), four categories, empty vs failed. |
| `03-tool-distribution-and-tool-choice.md` | 4 to 5 tools per agent, scoped tools, `tool_choice` with the newest-model 400 note. |
| `04-mcp-server-integration.md` | Local/project/user scopes, `${VAR}` expansion, community vs custom, resources. |
| `05-built-in-tools.md` | Grep vs Glob, Edit uniqueness, Read + Write fallback, step-by-step exploration. |
| `06-scenario-practice-questions.md` | 25 older, easier practice questions. |

### `domain-3-claude-code-configuration-workflows/` (9 files; canonical)

| File | One-line summary |
|---|---|
| `README.md` | Task statements 3.1 to 3.6, scenarios, sources, key themes. |
| `00-foundations-and-glossary.md` | Config from files, scopes, the `.claude/` folder tree, working modes, glossary. |
| `01-claude-md-hierarchy.md` | CLAUDE.md levels, `@import`, `.claude/rules/`, `/memory` and `/context`. |
| `02-slash-commands-and-skills.md` | Skills vs commands, scopes, frontmatter. **Says `allowed-tools` both "restricts" and "pre-approves"** (REVIEW_NEEDED R4). |
| `03-path-specific-rules.md` | `paths:` globs, glob cheat sheet, path rules vs folder CLAUDE.md. |
| `04-plan-mode-vs-direct-execution.md` | When to plan, when to execute directly, plan-then-execute, Explore subagent. |
| `05-iterative-refinement.md` | Input/output examples, test-driven iteration, interview pattern, batched vs sequential feedback. |
| `06-ci-cd-integration.md` | `-p`, JSON output with `--json-schema`, independent reviewer, re-reviews, GitHub Actions. |
| `07-scenario-practice-questions.md` | 25 older, easier practice questions. |

### `domain-4-prompt-engineering-structured-output/` (9 files; canonical)

| File | One-line summary |
|---|---|
| `README.md` | Task statements 4.1 to 4.6, scenarios, sources, key themes. |
| `00-foundations-and-glossary.md` | Precision/recall, false positives, well-formed vs correct, glossary. |
| `01-explicit-criteria-precision.md` | REPORT / DO NOT REPORT criteria, severity examples, disabling noisy categories. |
| `02-few-shot-prompting.md` | Few-shot for format, judgment, false positives, and varied documents. |
| `03-structured-output-tool-use-schemas.md` | Extraction tools, `strict`/`output_config.format`, `tool_choice`, nullable fields, enums. |
| `04-validation-retry-feedback-loops.md` | Semantic validation, retry with error, limits of retry, `detected_pattern`. |
| `05-batch-processing.md` | Batch facts and limits, SLA math, resubmit failures, test on a sample. |
| `06-multi-instance-multi-pass-review.md` | Self-review limits, independent instances, multi-pass, calibrated routing. |
| `07-scenario-practice-questions.md` | 25 older, easier practice questions. |

### `domain-5-context-management-reliability/` (9 files; canonical)

| File | One-line summary |
|---|---|
| `README.md` | Task statements 5.1 to 5.6, scenarios, sources, key themes. |
| `00-foundations-and-glossary.md` | Context as a budget, reliability risks, glossary. |
| `01-conversation-context-preservation.md` | Case facts, trimming tool output, lost in the middle, structured upstream output. |
| `02-escalation-ambiguity-resolution.md` | Escalation triggers, honoring human requests, policy gaps, multiple matches. |
| `03-error-propagation-multi-agent.md` | Structured error context, silent suppression vs abort, coverage notes. |
| `04-large-codebase-context.md` | Degradation, subagents, scratchpads, phase summaries, manifests, `/compact`. |
| `05-human-review-confidence-calibration.md` | Segment accuracy, calibrated confidence, routing, stratified sampling. |
| `06-provenance-and-uncertainty.md` | Claim-source mappings, conflicts, dates, report structure, format per content type. |
| `07-scenario-practice-questions.md` | 25 older, easier practice questions. |

---

## 2. Mapping `domains/` content to `domain-N-*` folders

**How to read the tables:**
- **Already there** = the domain-N folder already covers it. Nothing to move.
- **NEW** = only `domains/` has it. It will be merged into the named file.
- **Traps** = it goes into the new `anti-patterns.md`.

### `domains/01-agentic-architecture.md` → `domain-1-agentic-architecture-orchestration/`

| Section in `domains/01` | Destination | Status |
|---|---|---|
| Exam focus (5 bullets) | `README.md`, new "Exam focus" section at the top | NEW |
| Words you need first | `00-foundations-and-glossary.md` | Already there |
| Agentic loop: five steps, why `stop_reason` | `01-agentic-loops.md` | Already there |
| Mermaid + plain-text loop diagram (with error path and other stop reasons) | `01-agentic-loops.md`, **replaces** its weaker ASCII diagram (that one shows only 2 stop reasons) | NEW |
| `stop_reason` table, 7 values (adds `model_context_window_exceeded`) | `01-agentic-loops.md`, extends its 6-value table | NEW (one row) |
| Error handling: `tool_result` must come before text; API errors are retried with backoff | `01-agentic-loops.md` | NEW |
| "How loops fail" table (6 failure modes, including "never ends" from `tool_choice: "any"`) | `01-agentic-loops.md` | NEW |
| Coordinator and subagents | `02-multi-agent-orchestration.md` | Already there |
| Task → Agent rename note | `03-subagent-configuration.md` | Already there |
| Nested subagents (up to 3 layers) product note | `02-multi-agent-orchestration.md` | NEW |
| Enforcement, gates, handoff | `04-workflow-enforcement-and-handoffs.md` | Already there |
| Hooks | `05-agent-sdk-hooks.md` | Already there |
| Decomposition | `06-task-decomposition.md` | Already there |
| Sessions + R1 conflict note | `07-session-state-resumption-forking.md`, add the R1 note | NEW (note only) |
| Common traps (14 rows) | `anti-patterns.md` | Traps |
| Worked scenario (support agent, 7 steps) | `README.md`, new "Worked scenario" section | NEW |
| Quick recap (7 bullets) | `README.md`, merged into "Key themes to remember" | NEW (merge) |

### `domains/02-tool-design-mcp.md` → `domain-2-tool-design-mcp-integration/`

| Section in `domains/02` | Destination | Status |
|---|---|---|
| Exam focus | `README.md` | NEW |
| Tool definition parts, MCP primitives table | `00-foundations-and-glossary.md` | Already there |
| Five parts of a good description, order of fixes | `01-tool-interface-design.md` | Already there |
| "Group related actions with an `action` parameter" product note (docs advice) | `01-tool-interface-design.md` | NEW |
| Four error types, `isRetryable` as an app convention, input validation = `isError` | `02-structured-error-responses.md` and `00-...glossary` | Already there (matches the MCP spec) |
| `tool_choice` table and the newest-model 400 note | `03-tool-distribution-and-tool-choice.md` | Already there |
| MCP scopes, `${VAR}`, resources, community vs custom | `04-mcp-server-integration.md` | Already there |
| Built-in tools | `05-built-in-tools.md` | Already there |
| Common traps (15 rows) | `anti-patterns.md` | Traps |
| Worked scenario (research system, 5 problems) | `README.md` | NEW |
| Quick recap | `README.md`, merged into "Key themes" | NEW (merge) |

### `domains/03-claude-code-configuration.md` → `domain-3-claude-code-configuration-workflows/`

| Section in `domains/03` | Destination | Status |
|---|---|---|
| Exam focus | `README.md` | NEW |
| CLAUDE.md levels, `@path`, rules, `/memory`, conflicts | `01-claude-md-hierarchy.md` | Already there |
| **Settings, permission rules, and hooks for must-hold rules** (3 settings files, `allow`/`deny` example, "use permissions for a hard allow/deny") | `01-claude-md-hierarchy.md`, new section "When a rule must always hold" | NEW (today only a glossary line mentions it) |
| Skills, frontmatter, and the `allowed-tools` exam-vs-product note | `02-slash-commands-and-skills.md`: fix the "restricts" vs "pre-approves" contradiction (exam answer + verified product fact, verification log C9c) | NEW (correction) |
| Path rules | `03-path-specific-rules.md` | Already there |
| Plan mode: add the `/plan` command as an entry method | `04-plan-mode-vs-direct-execution.md` | NEW (one line) |
| Iterative refinement | `05-iterative-refinement.md` | Already there |
| CI flags table | `06-ci-cd-integration.md` | Already there |
| Common traps (14 rows) | `anti-patterns.md` | Traps |
| Worked scenario (12-developer monorepo, 6 steps) | `README.md` | NEW |
| Quick recap | `README.md`, merged into "Key themes" | NEW (merge) |

### `domains/04-prompt-engineering-structured-output.md` → `domain-4-prompt-engineering-structured-output/`

| Section in `domains/04` | Destination | Status |
|---|---|---|
| Exam focus | `README.md` | NEW |
| Criteria, few-shot, schemas, `tool_choice` notes | `01`, `02`, `03` files | Already there |
| Pydantic for cross-field validation | `04-validation-retry-feedback-loops.md` | NEW (short paragraph) |
| Batch facts, SLA math, failures | `05-batch-processing.md` | Already there |
| Multi-instance review | `06-multi-instance-multi-pass-review.md` | Already there |
| Common traps (14 rows) | `anti-patterns.md` | Traps |
| Worked scenario (40,000 invoices, 7 steps) | `README.md` | NEW |
| Quick recap | `README.md`, merged into "Key themes" | NEW (merge) |

### `domains/05-context-management-reliability.md` → `domain-5-context-management-reliability/`

| Section in `domains/05` | Destination | Status |
|---|---|---|
| Exam focus | `README.md` | NEW |
| Case facts, trimming, lost in the middle | `01-conversation-context-preservation.md` | Already there |
| **Prompt caching** (exists; exam tests only that) | `01-conversation-context-preservation.md` | NEW |
| Escalation, errors, calibration, provenance | `02`, `03`, `05`, `06` files | Already there |
| Degradation fixes + R2 `/compact` conflict note | `04-large-codebase-context.md`, add the R2 note | NEW (note only) |
| Common traps (14 rows) | `anti-patterns.md` | Traps |
| Worked scenario (electronics support, 7 steps) | `README.md` | NEW |
| Quick recap | `README.md`, merged into "Key themes" | NEW (merge) |

**Summary:**
- **Already in a domain-N folder:** about 75% of `domains/` (all the core concepts).
- **Not yet in a domain-N folder:**
  - the loop diagram and failure table
  - settings/permissions
  - prompt caching
  - Pydantic
  - a few product notes
  - the 5 worked scenarios
  - the 5 exam-focus and recap lists
  - the 5 common-traps tables

Every merged technical line is already confirmed in `docs/verification-log.md`. I will re-check each one against the official docs as I merge it.

---

## 3. Step 2: merge (after approval)

For each domain folder:

1. **Merge** the NEW rows above into the named existing files. No new topic files.
2. **Create `anti-patterns.md`**, with these parts:
   - title and one-line scope
   - an anti-pattern table (wrong approach / why it fails / do instead), built from the topic files' anti-pattern sections and the cheat sheet "Don't" lines
   - an "Exam traps" list (wrong answer / why it is wrong), built from the `domains/` Common traps tables
   - each item in 2 to 4 short sentences
3. **Contested items stay neutral.** Traps that touch a conflict say so and link to the conflict note. Affected items: R1 resume, R2 `/compact`, R3 `tool_choice`, R4 `allowed-tools`.
4. **Commit** once per domain, for example "Merge domain notes into domain-3 folder".

**Expected result:** each domain folder has `README.md`, `00`, the topic files, the practice-questions file (see decision C), and `anti-patterns.md`.

The new `anti-patterns.md` overlaps on purpose with the short anti-pattern sections inside the topic files. It is the one-page summary, and each row links back to its topic file. If you prefer, I can instead remove the anti-pattern sections from the topic files (see decision E).

---

## 4. Step 3: remove directories (after approval)

| Directory | Contents | Action | Why |
|---|---|---|---|
| `domains/` | 5 files | **Delete after Step 2** | All of its content is merged (§2). |
| `labs/` | `.gitkeep` only | Delete | Empty |
| `notes/` | `.gitkeep` only | Delete | Empty |
| `practice-questions/` | `.gitkeep` only | Delete | Empty. The real questions are in `mcq-bank/`. |
| `docs/` | 4 files | **Ask first** (decision B) | Has important content that is not in any domain folder. |
| `review-needed` | Does not exist | Nothing to do | See decision A for the `REVIEW_NEEDED.md` file. |

**Links that point into these folders and must be fixed:**

| File | Links to fix | Planned fix |
|---|---|---|
| `README.md` | 12 links to `domains/`, `docs/`, `REVIEW_NEEDED.md` | Rewrite in Step 4: domain folders, MCQ bank, cheat sheet only |
| `mcq-bank/0*.md` (5 files) | 1 each, to `../domains/0N-*.md` | Point to `../domain-N-*/README.md` |
| `mcq-bank/README.md` | 4 (to `domains/` and `REVIEW_NEEDED.md`) | Point to the domain folders; fix or drop the review links per decision A |
| `claude-architect-cheat-sheet.md` | 2 (`docs/verification-log.md`, `REVIEW_NEEDED.md`) | Update per decisions A and B |
| `REVIEW_NEEDED.md` | 1 (`docs/verification-log.md`) | Update per decision B |
| `docs/*` and `domains/*` | internal links | Deleted with their folders |
| `domain-N-*/README.md` | none | They only link to external doc URLs. No change needed. |

---

## 5. Step 4: reduce noise (after approval)

- **Remove placeholders:** the empty "## My notes" section with `<!-- Add study notes, lab links, and weak areas here -->` in all 5 domain READMEs. No "TODO" or "coming soon" text exists anywhere else. One option text in `mcq-bank/01` contains the words "fill in", but it is real question content and stays.
- **Duplicate and empty files:** after Step 3 there are no empty files. No two files share the same topic.
- **Rewrite `README.md`:** list only the 5 domain folders, the MCQ bank, and the cheat sheet, with the study order and the short exam facts. Keep the exam facts table from the official guide, because it is not stored anywhere else.
- **Link check:** run a script over every `.md` file and confirm 0 links point to missing files.
- **Commit.**

---

## 6. Decisions I need from you

| # | Question | My recommendation | Why |
|---|---|---|---|
| **A** | `REVIEW_NEEDED.md` is a **file**, not a folder. Delete it? | **Keep it** at the root until you have decided its items. Then delete it. | It holds 14 source conflicts, 6 unverified claims, and 6 open decisions. They are not stored anywhere else. Deleting it now would hide them. |
| **B** | `docs/` has important content: the **verification log** (evidence for every technical claim) and, in `domain-map.md`, the **6 exam scenarios** and the **out-of-scope list**. Delete it? | **Move, then delete.** Each domain's verified rows go into that domain `README.md` as a short "Sources checked" table. The exam scenarios, out-of-scope list, and exam facts go into the root `README.md`. Then delete `docs/audit.md` and `docs/verification-report.md` (one-time reports; still in git history). | This keeps the evidence and removes the folder. |
| **C** | Keep the 5 older `*-scenario-practice-questions.md` files (125 questions)? | **Delete them.** | They are not duplicates, but the 240-question bank replaces them. About 85% of their answers are "B", which teaches a bad guessing habit. You asked to be asked first, so I need your yes. |
| **D** | Keep `mcq-bank/` at the root? | **Yes.** | You listed it as one of the three things the README should show. |
| **E** | New `anti-patterns.md` vs the anti-pattern sections inside topic files | **Keep both**, with `anti-patterns.md` as the summary page linking back. | It is fewer edits and keeps each topic file complete. The other option is to strip those sections from the topic files. |
| **F** | Delete `CLEANUP_PLAN.md` at the end? | **Yes**, in the Step 4 commit. | It is only needed for this cleanup. It stays in git history. |

---

## 7. Final repo shape (if you accept my recommendations)

```text
.gitignore
README.md                          ← lists domains, MCQ bank, cheat sheet, study order
REVIEW_NEEDED.md                   ← kept until you resolve it (decision A)
claude-architect-cheat-sheet.md
mcq-bank/                          ← README + 5 question files
domain-1-agentic-architecture-orchestration/
  README.md  00..07 topic files  anti-patterns.md
domain-2-tool-design-mcp-integration/
  README.md  00..05 topic files  anti-patterns.md
domain-3-claude-code-configuration-workflows/
  README.md  00..06 topic files  anti-patterns.md
domain-4-prompt-engineering-structured-output/
  README.md  00..06 topic files  anti-patterns.md
domain-5-context-management-reliability/
  README.md  00..06 topic files  anti-patterns.md
```

**File count:** 67 tracked files now, 55 after the cleanup:
- 5 `domains/` files, 4 `docs/` files, 3 `.gitkeep` files, 5 old practice files, and `CLEANUP_PLAN.md` are removed.
- 5 `anti-patterns.md` files are added.

## 8. Conflicts to report (not resolved here)

These conflicts already exist. The cleanup carries them along unchanged and does not choose a side:
- **R1** resume vs fresh session after file changes
- **R2** when to run `/compact`
- **R3** `tool_choice` "any"/forced on the newest models
- **R4** `allowed-tools` restricts vs pre-approves

Full details are in `REVIEW_NEEDED.md`. The R4 fix in `02-slash-commands-and-skills.md` **keeps both views**. It does not drop the exam framing.
