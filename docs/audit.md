# Repository Audit (Step 1)

**Date:** 2026-10-15
**Scope:** every tracked file on the branch before this update (`domain-5-context-reliability`, commit `13e6ee8`).
**Method:** each file was read and compared with the exam guide's task statements (see [domain-map.md](domain-map.md)). The technical claims were then checked against the official docs (see [verification-log.md](verification-log.md)).

## 1. Files and what they cover

### Root

| File | Summary |
|---|---|
| `README.md` | Purpose and ground rules (no secrets, no client data). **Was out of date:** its folder table pointed to `notes/` and `practice-questions/`, but the content lives in the `domain-*` folders. Replaced in this update. |
| `.gitignore` | Ignores `.env` and secret files, OS/editor files, and Python/Node build output. |
| `labs/`, `notes/`, `practice-questions/` | Empty (only `.gitkeep`). |
| `Claude Certified Architect Foundations Exam Guide.pdf` | Official guide. **Not tracked in git.** It is left untracked on purpose, because guide §14 covers the confidentiality of exam materials (owner to decide, see REVIEW_NEEDED.md). |

### Domain 1: Agentic Architecture & Orchestration (`domain-1-agentic-architecture-orchestration/`)

| File | Summary |
|---|---|
| `00-foundations-and-glossary.md` | Agent vs chatbot, workflow vs agent, the stateless Messages API, the harness, the Agent SDK, an MCP intro, deterministic vs probabilistic control, glossary. |
| `01-agentic-loops.md` | The loop and the `stop_reason` table (6 of 7 values). Rules for `tool_use`/`tool_result`, `is_error`, a full Python loop, four anti-patterns. |
| `02-multi-agent-orchestration.md` | Hub-and-spoke, isolated subagent context, the coordinator's four jobs, decomposition that is too narrow, splitting scope between subagents. |
| `03-subagent-configuration.md` | Spawning with Task (exam) vs Agent (current), `AgentDefinition`, delegation prompts, structured attribution, parallel spawning. |
| `04-workflow-enforcement-and-handoffs.md` | Prompt vs code enforcement, a prerequisite-gate example, requests with several issues, structured human handoff. |
| `05-agent-sdk-hooks.md` | PreToolUse deny/allow/updatedInput, PostToolUse normalization, hook registration, hooks vs prompts. |
| `06-task-decomposition.md` | Prompt chaining vs adaptive decomposition, per-file plus cross-file passes, adaptive plan for legacy test coverage. |
| `07-session-state-resumption-forking.md` | Resume / fork / fresh start, CLI flags, SDK `resume`/`fork_session`, stale tool results. |
| `08-scenario-practice-questions.md` | 25 MCQs with answers in collapsible blocks. |
| `README.md` | Task statements 1.1 to 1.7 as checklists, sources, key themes. |

### Domain 2: Tool Design & MCP Integration (`domain-2-tool-design-mcp-integration/`)

| File | Summary |
|---|---|
| `00-foundations-and-glossary.md` | Parts of a tool definition, MCP host/client/server, stdio vs HTTP, tools vs resources vs prompts, glossary. |
| `01-tool-interface-design.md` | Descriptions as the selection signal, overlap causing misrouting, rename/split, keyword traps in system prompts. |
| `02-structured-error-responses.md` | MCP `isError`, protocol vs execution errors, four error categories, the `isRetryable` payload, failed vs empty results. |
| `03-tool-distribution-and-tool-choice.md` | Too many tools, scoped access, `load_document` vs `fetch_url`, cross-role tools, `tool_choice` with a current-model note. |
| `04-mcp-server-integration.md` | Project/user/local scopes, `${VAR}` expansion, all servers' tools loaded together, community vs custom servers, resources. |
| `05-built-in-tools.md` | Grep vs Glob, Edit uniqueness and the Read+Write fallback, Bash, step-by-step exploration, tracing re-exports. |
| `06-scenario-practice-questions.md` | 25 MCQs. |
| `README.md` | Task statements 2.1 to 2.5. |

### Domain 3: Claude Code Configuration & Workflows (`domain-3-claude-code-configuration-workflows/`)

| File | Summary |
|---|---|
| `00-foundations-and-glossary.md` | Config is loaded from files, the user/project/local/managed scopes, the `.claude/` folder tree, working modes, glossary. |
| `01-claude-md-hierarchy.md` | CLAUDE.md levels, how files combine, `@import`, `.claude/rules/`, `/memory` and `/context`. |
| `02-slash-commands-and-skills.md` | Commands vs skills, project vs user scope, frontmatter (`context: fork`, `allowed-tools`, `argument-hint`). |
| `03-path-specific-rules.md` | `paths:` globs, glob cheat sheet, path rules vs subfolder CLAUDE.md. |
| `04-plan-mode-vs-direct-execution.md` | When to use each mode, plan-then-execute, the Explore subagent. |
| `05-iterative-refinement.md` | Input/output examples, test-driven iteration, interview pattern, batched vs sequential feedback. |
| `06-ci-cd-integration.md` | `-p`, `--output-format json` + `--json-schema`, CLAUDE.md in CI, independent reviewer, re-review without duplicates, GitHub Actions. |
| `07-scenario-practice-questions.md` | 25 MCQs. |
| `README.md` | Task statements 3.1 to 3.6. |

### Domain 4: Prompt Engineering & Structured Output (`domain-4-prompt-engineering-structured-output/`)

| File | Summary |
|---|---|
| `00-foundations-and-glossary.md` | Precision/recall, false positives and trust, well-formed vs correct output, glossary. |
| `01-explicit-criteria-precision.md` | REPORT / DO NOT REPORT criteria, why "be conservative" fails, disabling noisy categories, severity defined with code examples. |
| `02-few-shot-prompting.md` | Few-shot for format, ambiguous judgment, acceptable-vs-issue pairs, extraction from varied layouts. |
| `03-structured-output-tool-use-schemas.md` | Extraction tool + schema, `strict`/`output_config.format`, `tool_choice`, nullable fields, `unclear`/`other` enums. |
| `04-validation-retry-feedback-loops.md` | Semantic validation, retry with the error, when retry cannot help, self-check fields, `detected_pattern`. |
| `05-batch-processing.md` | Sync vs Message Batches (50%, 24 h, `custom_id`, limits), SLA math, resubmitting failures, testing on a sample first. |
| `06-multi-instance-multi-pass-review.md` | Why self-review fails, independent instances, multi-pass review, calibrated confidence routing. |
| `07-scenario-practice-questions.md` | 25 MCQs. |
| `README.md` | Task statements 4.1 to 4.6. |

### Domain 5: Context Management & Reliability (`domain-5-context-management-reliability/`)

| File | Summary |
|---|---|
| `00-foundations-and-glossary.md` | Context as a budget, four reliability risks, glossary. |
| `01-conversation-context-preservation.md` | Summaries that lose detail, case-facts block, trimming tool output, lost in the middle, structured upstream output. |
| `02-escalation-ambiguity-resolution.md` | Escalation triggers vs weak signals, honoring human requests, policy gaps, multiple matches. |
| `03-error-propagation-multi-agent.md` | Structured error context, recovery options, silent suppression vs aborting, coverage notes. |
| `04-large-codebase-context.md` | Context degradation, subagents, scratchpads, phase summaries, manifests, `/compact`. |
| `05-human-review-confidence-calibration.md` | Averages that hide weak segments, calibrated field-level confidence, routing, stratified sampling. |
| `06-provenance-and-uncertainty.md` | Claim-source mappings, conflicting sources, dates, sections by certainty, format per content type. |
| `07-scenario-practice-questions.md` | 25 MCQs. |
| `README.md` | Task statements 5.1 to 5.6. |

## 2. Overall finding

- **Coverage:** every one of the 30 task statements has a matching file. Accuracy is good. Most technical claims match current docs (see the verification log).
- **Structure:** there was no single short file per domain with the required six sections. The new `domains/` files fill this gap and link to the deep-dive folders.
- **Practice questions:** the old practice questions were too easy, and they are the biggest weakness.
  - There are 125 questions (25 per domain).
  - The correct answer is "B" in about 105 of the 125.
  - The longest option is usually correct.
  - Many distractors are obviously silly, for example "Reinstall Claude Code".
  - Several stems state the root cause, so the answer is given away.
  - Several answer keys give no reason.

## 3. Gaps found

| # | Gap | Where it is now fixed |
|---|---|---|
| G1 | Practice questions too easy, answer letter almost always B | New 240-question bank with balanced letters and reasoned distractors: [mcq-bank/](../mcq-bank/README.md) |
| G2 | No consistent one-file summary per domain, and no "Common traps" section | [domains/](../domains/) files with the six required sections |
| G3 | Loop failure modes are thin: no `model_context_window_exceeded`, no API error retry (429/529), no stuck-loop detection, no orphan `tool_use` without a `tool_result`; weak loop diagram | [domains/01-agentic-architecture.md](../domains/01-agentic-architecture.md) §Agentic loop, with a new Mermaid diagram |
| G4 | Prompt caching not mentioned anywhere | [domains/05-context-management-reliability.md](../domains/05-context-management-reliability.md), at the exam depth ("knowing it exists") |
| G5 | Claude Code hooks in `settings.json` and permission rules are missing from Domain 3, even though the cheat sheet points to them ("move the rule to settings.json or a hook") | [domains/03-claude-code-configuration.md](../domains/03-claude-code-configuration.md) |
| G6 | Exam-vs-product differences scattered or missing: nested subagents, `allowed-tools` semantics, forced `tool_choice` on the newest models, input validation errors in MCP | ⚠️ notes in the domain files + [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) |
| G7 | Exam format facts (60 items, multiple-response items, 4 of 6 scenarios, 720 scaled score) not stated anywhere | [README.md](../README.md) and the verification log §D |
| G8 | Some jargon used without explanation ("constrained decoding", "JSON-RPC", "backoff", "idempotency") | Explained at first use in the new domain files |
| G9 | Root README out of date | Rewritten |
| G10 | The `allowed-tools` explanation in `domain-3.../02` contradicts itself ("restricts" vs "pre-approves") | Separated into exam answer vs product fact in the new Domain 3 file. The old file is unchanged (flagged in REVIEW_NEEDED.md). |

## 4. Not changed

The existing `domain-*` folders were **not** edited or deleted. They stay as deep-dive reading. Their older practice questions stay as extra practice but are not part of the 240-question bank.
