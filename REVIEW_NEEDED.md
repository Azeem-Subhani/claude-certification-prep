# Review Needed

**Purpose:** every conflict, unverified claim, and open decision found while building this study set. Nothing here was silently resolved.
**Owner for decisions:** Azeem Subhani.
**Created:** 2026-10-15.
**How to use:** work through the items. Decide each one, then update the linked files and remove the item.

**Item types:**
- **CONFLICT:** two sources disagree.
- **VERIFY:** a claim that is not yet confirmed word for word.
- **DECISION:** a choice that only you can make.

**Sources:**
- **Guide:** official exam guide PDF v1.0 (July 2026).
- **Cheat sheet:** [claude-architect-cheat-sheet.md](claude-architect-cheat-sheet.md).
- **Docs:** official Anthropic docs and the MCP spec, as logged in the "Sources checked" table of each domain folder's `README.md` (row IDs like A5b or C3).

**How the study set handles conflicts now:**
- The notes teach the **exam guide's answer** and add a ⚠️ product note.
- The MCQ bank **avoids** questions whose answer depends on a contested point.

---

## A. Conflicts between sources

| ID | Type | Topic | Source A says | Source B says | Current handling | Your decision |
|---|---|---|---|---|---|---|
| R1 | CONFLICT (guide vs cheat sheet) | Resuming after files changed | Cheat sheet 1.7: "Don't use --resume after files changed" and "Don't resume and just ask Claude to re-read the changed files". | Guide 1.7 skills: "Informing a resumed session about specific file changes for targeted re-analysis" is a correct technique. A fresh start with a summary is for **stale** tool results. | Domain 1 notes give the guide's view and flag this. MCQs test only the clear cases. | Which rule should the notes teach for "a few known files changed"? |
| R2 | CONFLICT (guide vs cheat sheet) | When to run `/compact` | Cheat sheet 5.4: "/compact early and often"; "Don't use /compact only when context is full". | Guide 5.4: use `/compact` to reduce context "when context fills with verbose discovery output". | Domain 5 notes flag it. No MCQ depends on it. | Keep, soften, or remove the cheat sheet anti-pattern line? |
| R3 | CONFLICT (exam vs docs) | `tool_choice: "any"` and forced tool | Guide 2.3 / 4.3: `"any"` guarantees a tool call. Forced selection makes a step run first. | Docs (read in full): **Claude Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1 return 400** for `"any"` and `"tool"`. Manual extended thinking also blocks them. Docs suggest `"auto"` + `strict: true`, or structured outputs. | Exam answer taught, with ⚠️ notes in Domains 2 and 4. MCQ stems that need `"any"`/forced state that the model supports forced tool use. | Confirm this approach. **High impact** for real projects. |
| R4 | CONFLICT (exam vs docs) | Skill `allowed-tools` | Guide 3.2: it **restricts** tool access during the skill. | Docs: it **pre-approves** the listed tools. Other tools stay callable. `disallowed-tools` removes tools. | Domain 3 notes give both. MCQs do not hinge on it. Old file `domain-3-.../02-slash-commands-and-skills.md` contradicts itself (lines about "restricts" vs "pre-approves"). | Fix the old file? (Not edited, to keep changes focused.) |
| R5 | CONFLICT (exam vs docs) | Nested subagents | Exam framing: subagents never talk directly; hub-and-spoke only. | Docs: by default a subagent can spawn subagents, up to 3 layers deep. | ⚠️ note in Domain 1. MCQs test hub-and-spoke only. | None needed unless the exam changes. |
| R6 | CONFLICT (cheat sheet vs MCP spec) | "Bad arguments" | Cheat sheet 2.2: unknown tool **and bad arguments** are protocol errors. | MCP spec (2025-11-25 and 2026-07-28): **input validation errors** are tool execution errors (`isError: true`). Unknown tool and malformed requests are protocol errors. | ⚠️ note in Domain 2 and the cheat sheet. MCQs avoid it. | Update the cheat sheet line? |
| R7 | CONFLICT (exam vs docs) | MCP config scopes | Guide: two levels (project `.mcp.json`, user `~/.claude.json`). | Docs: three scopes (local = default, project, user). Local and user both live in `~/.claude.json`. | ⚠️ note in Domain 2. | None needed. |
| R8 | PARTIAL (exam vs docs) | Subagent tool name and `allowedTools` | Guide: "Task" tool; must be in `allowedTools`. | Docs: renamed "Agent" (Task is an alias). Examples list it in `allowedTools`, but the docs never state this as a hard rule. | ⚠️ note in Domain 1. | None needed. |
| R9 | PARTIAL (cheat sheet vs docs) | Tools in batches | Cheat sheet 4.5: tool calls mid-processing → use the synchronous API. | Docs: batch requests may include tools, including server tools. A **client-side** multi-turn tool loop still cannot run inside one request. | Domain 4 explains the difference. | None needed. |
| R10 | PARTIAL (exam vs docs) | "All MCP tools available at once" | Guide 2.4: tools from all servers are available simultaneously. | Docs: all are offered, but tool search loads full schemas on demand by default. | ⚠️ note in the cheat sheet. | None needed. |
| R11 | PARTIAL (exam vs docs) | Edit fails on a non-unique match | Guide 2.5: fallback is Read + Write. | Docs: add surrounding context, or set `replace_all: true`. | Exam answer taught. MCQs avoid making these compete. | None needed. |
| R12 | PARTIAL (exam vs docs) | Hooks vs permission rules | Guide 1.5 / cheat sheet: use hooks for guaranteed compliance. | Claude Code hooks page: use the **permission system**, not a hook, for a hard allow/deny. A timed-out hook does not block by default. | Domain 3 shows permission rules for hard blocks. Domain 1 keeps Agent SDK hooks as the exam answer. | Confirm. |
| R13 | PARTIAL (exam vs docs) | Tool consolidation | Guide 2.1: split generic tools into purpose-specific tools. | API docs: group related operations into one tool with an `action` parameter. | Domain 2 explains both (vague tool → split; many variants of one job → one tool with a type parameter). | None needed. |
| R14 | PARTIAL (exam vs docs) | `stop_reason` values | Guide: only `tool_use` and `end_turn`. | Docs: seven values. | Domain 1 table lists all seven. Exam answers use the two. | None needed. |

## B. Claims to verify word for word

The verification fetches used a summarizing tool. These items matter enough to open the page and confirm the exact wording:

| ID | Type | Claim | Where used | Page |
|---|---|---|---|---|
| V1 | VERIFY | Structured outputs use `output_config.format`, and `output_format` is deprecated. | Domain 4; cheat sheet 4.3 note | platform.claude.com/docs/en/build-with-claude/structured-outputs |
| V2 | VERIFY | Tool Runner is beta (`client.beta.messages.tool_runner`). | Domain 1 Sources checked, row A10 (not taught) | platform.claude.com/docs/en/agents-and-tools/tool-use/tool-runner |
| V3 | VERIFY | PostToolUse `updatedToolOutput` replaces the result for any tool (`updatedMCPToolOutput` is older). | Domain 1 Sources checked, row C5c | code.claude.com/docs/en/agent-sdk/hooks |
| V4 | VERIFY | Skills use `$ARGUMENTS` in the body to insert user input. | Domain 3 skill example | code.claude.com/docs/en/skills |
| V5 | VERIFY | Model names (Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1) in ⚠️ notes. These will go out of date. | Domains 2 and 4 (topic files and Sources checked) | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools |
| V6 | VERIFY | Cheat sheet 4.5 first line: the words were scrambled in the PDF and were put back in order. Check it against your original. | Cheat sheet 4.5 | your source of the cheat sheet |

## C. Decisions only you can make

| ID | Type | Question |
|---|---|---|
| D1 | DECISION | **Exam guide PDF.** It is in the repo folder but **not tracked** in git. Guide §14 covers confidentiality of exam materials. Should it be committed, kept local, or added to `.gitignore`? (Left untracked, untouched.) |
| D2 | DECISION | **Multiple-response items.** The real exam uses multiple-choice **and multiple-response** items (guide §3). Your spec asked for single-answer MCQs only, so the bank has none. Add a set of "choose two" items? |
| D3 | DECISION | **Old practice questions** (125 in `domain-*/0X-scenario-practice-questions.md`). They are easier, and about 85% have answer "B". They were kept as warm-ups and linked as such. Keep, rewrite, or remove? (Not deleted: your rule says ask first.) |
| D4 | DECISION | **Difficulty mix.** Your spec said "30% easy, 40% medium, 60% hard" (sums to 130%) and then "roughly 25/40/35". The bank uses exactly 25/40/35 per domain. Confirm. |
| D5 | DECISION | **Rule 1 wording.** It read "Copy text, questions... from the cyberskill site or claudecertificationguide.com". This was treated as "**Do not** copy". The cyberskill practice site was **not visited**, so no copying from it is possible. The guide page was read for ideas only and rewritten. Confirm this reading. |
| D6 | DECISION | **Question distribution.** 240 questions were split by the official weights: 65 / 43 / 48 / 48 / 36 (27/18/20/20/15%). Confirm. |

## D. Human review before use

- The MCQ bank was written with AI help and checked by scripts plus a blind re-answer pass (see [mcq-bank/README.md](mcq-bank/README.md)). A human should still spot-check hard questions before sharing the bank with others.
- No client data is used anywhere. All companies and IDs in the scenarios are invented.
