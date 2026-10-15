# Claude Architect Exam Cheat Sheet (Complete Version)

> **Source:** converted from the owner's personal PDF `claude-architect-cheat-sheet.md.pdf` (17 pages), extracted with macOS PDFKit (the PDF had a text layer, so OCR was not needed).
> **Conversion rules:** content and order are unchanged. Only broken formatting was fixed: words split across lines ("A gentic" → "Agentic", "verif y" → "verify", "itsel f" → "itself"), `**` glob characters lost in the PDF (`/*.test.tsx` → `**/*.test.tsx`), and one Batch API line whose words were scrambled in the PDF (marked below).
> **Verify notes:** lines marked **⚠️ Verify (docs):** were added during conversion. They show where official Anthropic docs say something different or more. The original `[verify]` tags are the cheat sheet author's own. All open items are collected in [REVIEW_NEEDED.md](REVIEW_NEEDED.md).

## How to read

Each line is one decision. "If" = the situation in the question. "choose" = the correct answer. "Don't" = a wrong answer the exam uses as a trap. **[verify]** = the guide says the exam answer and the current product are different, or the topic is outside the exam.

## Key terms

- **Agentic loop:** code that calls Claude again and again until the task is done.
- **stop_reason:** a field in each reply. "tool_use" = Claude wants a tool, continue. "end_turn" = finished.
- **Coordinator:** the main agent. It splits the task, calls helper agents, and joins the results.
- **Subagent:** a helper agent. It sees only what the coordinator puts in its prompt.
- **Hook:** code that runs at a fixed point. PreToolUse = before a tool runs. PostToolUse = after a tool runs.
- **Prerequisite gate:** code that blocks a tool until an earlier step is done.
- **MCP (Model Context Protocol):** a standard way to connect Claude to outside tools and data.
- **tool_choice:** "auto" = tool is optional. "any" = must call some tool. Forced = must call one named tool.
  - **⚠️ Verify (docs):** the API also has `"none"`. On Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1, `"any"` and forced `"tool"` return a 400 error. See [docs/verification-log.md](docs/verification-log.md) row A5b.
- **isRetryable:** true only if sending the exact same call again can work.
- **CLAUDE.md:** a rules file Claude Code loads in every session.
- **Skill:** a saved workflow (like /review) that loads only when used.
- **Few-shot examples:** 2 to 4 worked examples inside the prompt.
- **Attention dilution:** too many items in one pass, so some items get less care.
- **Stale context:** old tool results in history that no longer match the real files.
- **Context degradation:** in a long session, Claude forgets exact details and speaks in general terms.
- **Case facts block:** a fixed block of key numbers, dates, and IDs added to every prompt.
- **Calibration:** comparing confidence scores with known correct answers to learn what they really mean.
- **Provenance:** where a fact came from (source, document, excerpt, date).
- **Policy gap:** the policy says nothing about this case. **Policy violation:** the policy clearly says no.

---

## Domain 1: Agentic Architecture and Orchestration

### 1.1 Agentic loops

- If stop_reason is "tool_use", choose: run the tools, add the results to the history, and call Claude again.
- If stop_reason is "end_turn", choose: stop the loop and show the final answer.
- If the agent stops in the middle of complex tasks, choose: check stop_reason instead of the content type.
- If you need protection from a loop that never ends, choose: a loop limit as a safety net only, with a warning log.
- If business rules need a fixed order (money, security, law), choose: code enforcement. Otherwise, let the model pick tools.
- If you see other stop_reason values (pause_turn, max_tokens, refusal), choose: treat them as "not finished, check why" [verify: the exam uses only tool_use and end_turn].
  - **⚠️ Verify (docs):** docs list seven values: end_turn, max_tokens, stop_sequence, tool_use, pause_turn, refusal, model_context_window_exceeded (row A2). The guide names only tool_use and end_turn.
- Don't: look for words like "I'm done" in the text. This is an anti-pattern.
- Don't: use a loop limit (for example, 10 loops) as the main way to stop. This is an anti-pattern.
- Don't: check if the first content block is text to decide the task is finished. Text can come with a tool call. This is an anti-pattern.
- Don't: force tool_choice "any" to stop text replies. It creates an endless loop. This is an anti-pattern.
- Don't: forget to add tool results to the history. Claude cannot use what it never sees. This is an anti-pattern.

### 1.2 Multi-agent orchestration

- If several agents work together, choose: hub-and-spoke. All messages go through the coordinator.
- If a subagent needs information, choose: write all of it into the subagent's prompt. There is no shared memory.
- If the same subagent is called twice, choose: treat each call as new. It remembers nothing from the last call.
- If a question is simple, choose: call only the subagents it needs, not the full pipeline.
- If several subagents search, choose: give each one a different subtopic or source type.
- If the combined result has gaps, choose: the coordinator sends targeted new queries and runs synthesis again.
- If the report misses whole categories (scope, not depth), choose: fix the coordinator's task split.
- Don't: blame the subagents when the coordinator split the topic too narrowly. This is an anti-pattern.
- Don't: let subagents talk to each other directly, even to save time. This is an anti-pattern.
- Don't: add more subagents to fix a narrow task split. This is an anti-pattern.
- Note: current Claude Code lets a subagent start its own subagents [verify]. On the exam, direct subagent-to-subagent talk is always wrong.
  - **⚠️ Verify (docs):** confirmed. By default a subagent can start its own subagents, up to three layers deep (row C3).

### 1.3 Subagent invocation and context passing

- If the coordinator must start subagents, choose: include "Task" in its allowedTools (current name "Agent") [verify].
  - **⚠️ Verify (docs):** confirmed rename. The tool is "Agent"; "Task" still works as an alias (row C1a). The docs do not state the allowedTools rule as a hard requirement (row C1b).
- If you define a subagent, choose: an AgentDefinition with a description, a system prompt, and tool limits.
- If a later agent needs earlier results, choose: pass the complete findings in its prompt.
- If you pass findings, choose: structured data that keeps content and metadata apart (claim, source URL, document name, page).
- If you write the coordinator's instructions to subagents, choose: goals and quality criteria, not step-by-step procedures.
- If subagent tasks are independent, choose: several Task calls in one coordinator response (parallel).
- If you want to try different approaches from the same analysis, choose: fork_session (a new branch).
- If you want to continue the same work, choose: --resume.
- Don't: blame the synthesis agent's prompt when its report has no sources. It never got the metadata. This is an anti-pattern.
- Don't: give the synthesis agent search tools to fix missing citations. This is an anti-pattern.
- Don't: start independent subagents one at a time in separate turns. This is an anti-pattern.
- Don't: mix up fork_session (branch) and --resume (continue). This is an anti-pattern.

### 1.4 Workflow enforcement and handoff

- If one failure can lose money (refunds, transfers, payments), choose: programmatic enforcement.
- If one failure can break security (identity check, access control), choose: programmatic enforcement.
- If one failure can break compliance (AML checks, regulations), choose: programmatic enforcement.
- If it is low risk (format, style, output order), choose: prompt instructions.
- If a refund must wait for an identity check, choose: a gate. process_refund returns an error until get_customer gives a verified ID.
- If a prompt already says the right order but fails 8% of the time, choose: a prerequisite gate.
- If a customer raises several issues at once, choose: split them, check them in parallel with shared context, and give one combined answer.
- If the agent hands off to a human, choose: a complete summary with customer ID, conversation summary, root cause, refund amount, and recommended action.
- Don't: fix high-risk failures with stronger system prompts. Failure gets lower but never zero. This is an anti-pattern.
- Don't: rely on few-shot examples for guaranteed compliance. This is an anti-pattern.
- Don't: use a routing classifier to fix a step-order problem inside one agent. Classifiers only route. This is an anti-pattern.
- Don't: write a handoff that needs the chat history. The human cannot see the transcript. This is an anti-pattern.
- Don't: handle several issues one by one in separate chats, or answer only the first. This is an anti-pattern.

### 1.5 Agent SDK hooks

- If you must block, change, or redirect a tool call before it runs, choose: a PreToolUse hook.
- If you must change tool results before Claude reads them, choose: a PostToolUse hook.
- If tools return mixed formats (Unix time, ISO dates, number codes, letter codes), choose: PostToolUse to convert all of them to one format.
- If refunds above $500 need a human, choose: PreToolUse blocks the call and sends it to escalation.
- If a transfer needs an AML check first, choose: PreToolUse blocks transfer_funds until aml_check passes.
- If discounts above 20% need a manager, choose: PreToolUse sends the call to an approval queue.
- If a rule must work 100% of the time, choose: a hook. If small mistakes are OK, choose: a prompt.
- If you want to save the full transcript before /compact, choose: a PreCompact hook [verify: outside the guide's scope].
- Don't: use PostToolUse to block an action. The tool already ran. This is an anti-pattern.
- Don't: use prompt instructions for 100% compliance. This is an anti-pattern.
- Don't: let the model convert mixed data formats itself. It is not consistent. This is an anti-pattern.
- Don't: mix up the direction. Pre = before, Post = after. This is an anti-pattern.

### 1.6 Task decomposition

- If the steps are known in advance (code review, document extraction, compliance checks), choose: a fixed sequential pipeline (prompt chaining).
- If the scope is unknown (legacy code, security audit, research, debugging an unknown system), choose: dynamic adaptive decomposition.
- If you review many files, choose: a fixed pipeline. One pass per file, then one cross-file pass, then a report.
- If you add tests to a legacy codebase, choose: dynamic decomposition. Map, prioritize, and change the plan as you learn.
- If early files get detailed feedback and later files get little, choose: one pass per file plus one cross-file integration pass.
- If the same code is flagged in one file but approved in another, choose: the same multi-pass fix (it is attention dilution).
- Don't: use a bigger model or a bigger context window to fix attention dilution. This is an anti-pattern.
- Don't: keep one pass and just improve the prompt. This is an anti-pattern.
- Don't: use a fixed pipeline for open-ended investigation. This is an anti-pattern.
- Don't: split files into batches without a cross-file pass. Cross-batch issues are missed. This is an anti-pattern.
- Don't: pick a pattern because it sounds more advanced. Match it to the task. This is an anti-pattern.

### 1.7 Session state and resumption

- If you continue work and no files changed, choose: --resume.
- If you compare two approaches from the same analysis, choose: fork_session.
- If files changed since the last session, choose: a fresh session, a structured summary, and a list of the changed files.
  - **⚠️ Verify (exam guide):** guide task 1.7 lists "informing a resumed session about specific file changes for targeted re-analysis" as a correct skill. This conflicts with the "Don't: use --resume after files changed" lines below. See REVIEW_NEEDED.md item R1.
- If only 3 of 50 files changed, choose: targeted re-analysis of those 3 files only.
- If a long session has a messy history, choose: a fresh session with a summary.
- If dependencies were updated, choose: a fresh session with a summary.
- Don't: re-read the whole codebase when only a few files changed. This is an anti-pattern.
- Don't: use --resume after files changed. Old tool results stay in history. This is an anti-pattern.
- Don't: resume and just ask Claude to re-read the changed files. The old results still affect answers. This is an anti-pattern.
- Don't: use fork_session to fix stale context. The fork copies the stale history. This is an anti-pattern.
- Note: name a session with --name, and -c continues the latest one [verify].
  - **⚠️ Verify (docs):** confirmed. `--name`/`-n` names a session, `/rename` renames it, `-c` continues the latest session in the folder (row C4b).

**Exam traps (Domain 1):** Loop limits do not fix early stops. Subagents get only what you pass them. Scope gaps start at the coordinator. Money, security, and compliance always need code, not prompts.

---

## Domain 2: Tool Design and MCP Integration

### 2.1 Tool interface design

- If you write a tool description, choose: five parts. What it does, inputs and formats, example queries, limits, and when to use a different tool.
- If Claude picks the wrong tool and descriptions are short, choose: expand the descriptions first.
- If one generic tool does many jobs (analyze_document), choose: split it into narrow tools with clear inputs and outputs.
- If two tool names are confusing, choose: rename one so its purpose is clear.
- If you updated the descriptions, choose: check the system prompt for keywords that pull Claude to the wrong tool.
- If an agent has far more than 4 to 5 tools, choose: fix the toolset (see 2.3), not the descriptions.
- Don't: add few-shot examples first. They cost tokens and miss the cause. This is an anti-pattern.
- Don't: build a routing classifier as the first fix. It is too much work. This is an anti-pattern.
- Don't: merge similar tools as the first fix. It costs more than better descriptions. This is an anti-pattern.
- Don't: ignore the system prompt after you fix the descriptions. This is an anti-pattern.

### 2.2 Structured error responses

- If the request is malformed (unknown tool, bad arguments), choose: a protocol error handled by the host. Fix the schema or client.
  - **⚠️ Verify (docs):** the MCP spec lists input validation errors as tool execution errors (`isError: true`), not protocol errors. Unknown tools and malformed requests are protocol errors (row B2b).
- If the tool ran and failed inside, choose: a normal result with isError: true, so Claude can plan recovery.
- If the error is transient (timeout, service down, rate limit), choose: isRetryable true. Retry after a short wait.
- If the error is validation (bad format, missing field), choose: isRetryable false. Fix the input and make a new call [verify: the guide does not set this value].
- If the error is business (over a policy limit), choose: isRetryable false. Take another path or escalate.
- If the error is permission (access denied), choose: isRetryable false. Escalate or use other credentials.
- If the query worked but found nothing, choose: isError false with zero results. Do not retry.
- If a subagent hits a short failure, choose: retry inside the subagent first.
- If the subagent cannot fix it, choose: report up with what it tried and any partial results.
- Note: errorCategory and isRetryable are an app convention, not part of the MCP spec.
- Don't: retry an empty result from a successful query. This is an anti-pattern.
- Don't: return a general message like "Operation failed". This is an anti-pattern.
- Don't: retry business errors. The same rule fails every time. This is an anti-pattern.
- Don't: mark validation errors isRetryable true. This is an anti-pattern.
- Don't: read isRetryable false as "give up". Validation errors can still be fixed. This is an anti-pattern.
- Don't: hide subagent errors by returning empty results as success. This is an anti-pattern.

### 2.3 Tool distribution and tool_choice

- If you give tools to an agent, choose: 4 to 5 tools that fit its role only.
- If tools are few but two look alike, choose: sharper descriptions.
- If tools do different jobs (query, transform, export), choose: split them by role, 4 to 5 per agent.
- If many tools are versions of one job, choose: one tool with a type parameter (for example, transform_data).
- If a tool can do more than the agent needs, choose: a narrower tool (load_document instead of fetch_url).
- If you plan to move tools to a second MCP server, choose: don't. Claude still sees one flat list.
- If Claude may answer in text or call a tool, choose: tool_choice "auto".
- If you need structured output and the document type is unknown, choose: tool_choice "any".
- If one step must always run first (extract_metadata), choose: forced tool_choice, then "auto" for later steps.
  - **⚠️ Verify (docs):** on Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1, `"any"` and forced tool choice return a 400 error. Docs suggest `"auto"` with `strict: true`, or structured outputs (row A5b). The exam guide still uses `"any"`/forced.
- If an agent often needs simple checks from another role (85% simple), choose: a small scoped tool on that agent (verify_fact).
- If the check is complex, choose: route it through the coordinator.
- Don't: send every simple check through the coordinator. It adds round trips and latency. This is an anti-pattern.
- Don't: use "auto" when structured output is required. This is an anti-pattern.
- Don't: give one agent 18 tools. This is an anti-pattern.
- Don't: give a subagent a generic fetch_url when load_document is enough. This is an anti-pattern.

### 2.4 MCP server integration

- If the whole team needs a server, choose: .mcp.json in the project root (shared in git).
- If the server is personal or a test, choose: ~/.claude.json (not shared).
- Note: the exam uses two levels; current docs show three scopes (local, project, user) [verify].
- If the server is remote, choose: "type": "http" plus a url. If it is local, choose: command plus args.
- If the config needs a secret, choose: ${VAR_NAME}. For a fallback value, choose: ${VAR:-default}.
- If several servers are configured, choose: expect all their tools to be available at once.
  - **⚠️ Verify (docs):** partial. Tools from all servers are offered together, but tool search loads full tool schemas only when needed by default (row C15d).
- If the agent wastes calls to learn what data exists, choose: MCP resources (issue lists, doc trees, database schemas).
- If you need a standard integration (Jira, GitHub, Slack), choose: a community server first.
- If your workflow or system is unique, choose: a custom server.
- If Claude prefers Grep over a better MCP tool, choose: a richer MCP tool description.
- Don't: build a custom server for Jira. This is an anti-pattern.
- Don't: put team servers in ~/.claude.json. This is an anti-pattern.
- Don't: commit secrets in .mcp.json. This is an anti-pattern.
- Don't: leave MCP tool descriptions short. This is an anti-pattern.

### 2.5 Built-in tools

- If you search text inside files (callers, error messages, imports), choose: Grep.
- If you find files by name, extension, or folder, choose: Glob.
- If you change part of a file, choose: Edit.
- If Edit fails because the text is not unique, choose (exam): Read + Write. In real work: a longer match or replace_all [verify].
- If you explore a codebase, choose: Grep for entry points, Read to follow imports, Grep for new names, Read only what you need.
- If a function is re-exported under another name, choose: Read the wrapper file, then Grep for each new name.
- If you need callers of a deprecated function and their tests, choose: Grep for the function, Glob for test files, Grep for wrapper names.
- Don't: use Glob to find function callers. This is an anti-pattern.
- Don't: use Grep to find files by name pattern. This is an anti-pattern.
- Don't: read all files before you know what matters. This is an anti-pattern.
- Don't: use Read + Write for every change. This is an anti-pattern.
- Don't: answer "widen the match" on the exam when Edit fails. This is an anti-pattern.

**Exam traps (Domain 2):** Choose the low-effort fix first. Count the tools before you choose a fix. An empty result is not a failure. Community servers come before custom ones.

---

## Domain 3: Claude Code Configuration and Workflows

### 3.1 CLAUDE.md hierarchy

- If it is a personal preference, choose: ~/.claude/CLAUDE.md (not shared).
- If it is a team standard, choose: .claude/CLAUDE.md or root CLAUDE.md (shared in git).
- If it is only for one package, choose: a CLAUDE.md in that subfolder.
- If it is your personal note for one repo, choose: CLAUDE.local.md (usually in .gitignore).
- If a new teammate does not get the rules, choose: move them from user level to project level.
- If two CLAUDE.md files conflict, choose: no level is sure to win. Move the rule to settings.json or a hook.
- If a rule must hold every time (blocked tool, required formatter, permissions), choose: settings.json or a hook.
- If one CLAUDE.md is too long, choose: split it with @path lines (there is no @import keyword).
- If you want to load less context per session, choose: .claude/rules/ with paths, not @path imports.
- If you want to see which files loaded, choose (exam): /memory. In current Claude Code: /context [verify].
  - **⚠️ Verify (docs):** `/memory` lists and opens memory files; `/context` shows context use including loaded memory files (row C8).
- If an instruction disappears after /compact, choose: it was a nested CLAUDE.md or path rule (loads again later), or it was only in chat.
- Don't: think /memory loads files. It only shows them. This is an anti-pattern.
- Don't: use a folder CLAUDE.md for rules that span many folders. This is an anti-pattern.
- Don't: believe "the more specific file wins" for CLAUDE.md. This is an anti-pattern.

### 3.2 Custom slash commands and skills

- If the team needs a command, choose: .claude/skills/name/SKILL.md (or .claude/commands/name.md).
- If the command is only for you, choose: ~/.claude/skills/ or ~/.claude/commands/.
- If you want your own version of a team skill, choose: a personal skill with a different name.
- If skill output is long and noisy (analysis, brainstorming), choose: context: fork.
- If a skill should use only some tools, choose (exam): allowed-tools "restricts" tools [verify: now it pre-approves them; disallowed-tools removes them].
  - **⚠️ Verify (docs):** confirmed. `allowed-tools` pre-approves tools; it does not block others (row C9c).
- If users need to know what input to give, choose: argument-hint.
- If you want Claude to pick the skill by itself, choose: a good description field [verify: outside the exam's three fields].
- If the rule must apply all the time, choose: CLAUDE.md or .claude/rules/. If it is an occasional workflow, choose: a skill.
- Don't: put a flat .md file directly in .claude/skills/. A skill is a folder with SKILL.md. This is an anti-pattern.
- Don't: put team commands in ~/.claude/. This is an anti-pattern.
- Don't: use skills for always-on rules. This is an anti-pattern.
- Don't: run noisy skills without context: fork. This is an anti-pattern.
- Don't: put task workflows in CLAUDE.md. This is an anti-pattern.

### 3.3 Path-specific rules

- If a standard applies to all code, choose: root CLAUDE.md.
- If a standard applies to one package folder, choose: that folder's CLAUDE.md.
- If a standard applies to one file type in many folders, choose: a .claude/rules/ file with paths globs.
- If it is a workflow you run on demand, choose: a skill in .claude/skills/.
- If test files sit next to source files in 50+ folders, choose: paths `["**/*.test.tsx", "**/*.test.ts"]`.
- If Terraform rules should load only for Terraform files, choose: paths `["terraform/**/*", "**/*.tf"]`.
- Don't: copy a CLAUDE.md into every test folder. This is an anti-pattern.
- Don't: put file-type rules in root CLAUDE.md. They waste tokens in every session. This is an anti-pattern.
- Don't: use a skill for automatic file-type rules. This is an anti-pattern.

### 3.4 Plan mode vs direct execution

- If the change is large (monolith to microservices), choose: plan mode.
- If there are several valid approaches, choose: plan mode.
- If architecture decisions are needed, choose: plan mode.
- If many files change (library migration across 30 or 45 files), choose: plan mode, then direct execution.
- If you must explore the codebase first, choose: plan mode with the Explore subagent.
- If it is a one-file bug fix with a clear stack trace, choose: direct execution.
- If you add one validation check or change one config value, choose: direct execution.
- If the task is hard but clear, choose: direct. If it is simple but unclear, choose: plan. Unclear scope decides, not difficulty.
- If discovery output is long, choose: the Explore subagent so the main chat stays clean.
- If you need plan mode, choose: --permission-mode plan, Shift+Tab, or start the prompt with /plan.
- Don't: use direct execution for multi-file architecture changes. This is an anti-pattern.
- Don't: use plan mode for a clear one-file fix. This is an anti-pattern.
- Don't: forget the plan-then-execute pattern. This is an anti-pattern.
- Don't: start direct and switch to plan only when problems appear, if the task is already known to be complex. This is an anti-pattern.

### 3.5 Iterative refinement

- If Claude reads your prose description differently each time, choose: 2 to 3 concrete input/output examples.
- If the transformation is complex with many edge cases, choose: test-driven iteration. Write tests first and share the failures.
- If you work in a domain you do not know well, choose: the interview pattern. Claude asks questions before it builds.
- If several issues affect each other, choose: batch feedback in one message.
- If the issues are independent, choose: sequential feedback, one at a time.
- If examples work for normal cases but miss edge cases, choose: add an edge-case example.
- Don't: rewrite the prose more carefully when Claude reads it differently. This is an anti-pattern.
- Don't: mix up batch and sequential feedback. This is an anti-pattern.
- Don't: use the interview pattern when you already know the exact change. Use examples. This is an anti-pattern.

### 3.6 CI/CD integration

- If a CI job hangs waiting for input, choose: the -p flag (--print).
- If machines must read the output, choose: --output-format json and --json-schema (data is in structured_output).
- If you review code that Claude generated, choose: a separate, independent Claude run.
- If each push repeats the same review comments, choose: include past findings and report only new or still-open issues.
- If CI needs project context (test rules, fixtures, review criteria), choose: put it in CLAUDE.md.
- If CI generates tests, choose: include existing tests so Claude does not repeat them.
- If a check blocks merging, choose: the real-time API.
- If a job runs overnight or weekly (tech debt report, audit, nightly tests), choose: the Batch API.
- If you want to add rules but keep Claude Code's default behavior, choose: --append-system-prompt.
- If you need a different identity or permission model, choose: --system-prompt (replaces everything).
- If you need a turn limit, choose: --max-turns. To pre-approve tools, choose: --allowedTools. To remove tools, choose: --disallowedTools.
- If you want a fast, minimal scripted run without project config, choose: --bare.
- If you run Claude in GitHub Actions, choose: anthropics/claude-code-action@v1, with flags in claude_args [verify: outside the guide].
- If you run several sessions in parallel on one repo, choose: git worktrees [verify: outside the guide].
- Don't: use CLAUDE_HEADLESS=true, --batch, or stdin from /dev/null. They do not fix the hang. This is an anti-pattern.
- Don't: review code in the same session that wrote it. This is an anti-pattern.
- Don't: use the Batch API for pre-merge checks. This is an anti-pattern.
- Don't: run reviews without past findings. This is an anti-pattern.

**Exam traps (Domain 3):** Project (.claude/) = shared. User (~/.claude/) = personal. Unclear scope, not difficulty, decides plan mode. /memory only shows files. The -p flag is the answer for hanging CI jobs.

---

## Domain 4: Prompt Engineering and Structured Output

### 4.1 System prompts with explicit criteria

- If a review has too many false findings, choose: clear categories. Report bugs and security issues; skip style and local patterns.
- If comments are checked, choose: flag them only when the comment says something the code does not do.
- If you define severity levels, choose: a code example for each level, not a prose description.
- If one category is often wrong and developers ignore all findings, choose: turn that category off, fix it with examples, then turn it on again.
- If you use confidence scores, choose: use them for routing only after clear criteria exist.
- Don't: write "be conservative" or "only report high-confidence findings". This is an anti-pattern.
- Don't: use confidence thresholds to fix false positives. This is an anti-pattern.
- Don't: keep all categories active while you fix a noisy one. This is an anti-pattern.

### 4.2 Few-shot prompting

- If detailed instructions still give inconsistent format, choose: few-shot examples.
- If Claude makes different judgments on unclear cases, choose: few-shot examples that show the reasoning.
- If fields come back empty but the data is in the document (for example, in paragraphs), choose: examples from varied layouts.
- If you build examples, choose: 2 to 4 targeted examples that cover the failing cases.
- If you want fewer false positives in review, choose: examples of what to flag and what to accept.
- If output format is inconsistent, choose: few-shot examples.
- If JSON is malformed, choose: tool_use with a JSON schema.
- If Claude invents values for missing fields, choose: optional or nullable schema fields.
- If Claude picks the wrong tool, choose: better tool descriptions first, then few-shot.
- If Claude misses data in narrative text, choose: few-shot examples of narrative extraction.
- If line items do not add up to the total, choose: a validation-retry loop.
- Don't: add more detailed instructions when instructions already failed. This is an anti-pattern.
- Don't: show only input-output pairs without reasoning. This is an anti-pattern.
- Don't: use confidence thresholds to fix inconsistent judgments. This is an anti-pattern.

### 4.3 Structured output with tool_use

- If JSON must always be valid, choose: tool_use with a JSON schema, not JSON in a text reply.
- If Claude may answer with text instead of a tool, choose: tool_choice "auto".
- If you must get structured output and the document type is unknown, choose: tool_choice "any".
- If one extraction step must always run, choose: forced tool_choice. Send the next request with "auto".
  - **⚠️ Verify (docs):** `"any"` and forced tool choice return 400 on the newest models (row A5b). Exam answer unchanged.
- If the source may not have some data, choose: optional or nullable fields.
- If the source is truly unclear, choose: an "unclear" enum value.
- If a category may not fit the list, choose: an "other" value plus a detail text field.
- If you need the same format each time (ISO dates, plain numbers), choose: format rules in the prompt next to the schema.
- If totals are wrong, values are in the wrong field, or values are invented, choose: extra validation (schemas cannot catch these).
- Note: the API also offers strict: true and output_config.format [verify: the exam uses the two-level answer].
  - **⚠️ Verify (docs):** `strict: true` and `output_config.format` confirmed (rows A7a, A7b). Structured outputs can still fail on refusals or `max_tokens` cut-offs, so keep validation (row A7c).
- Don't: believe tool_use stops all extraction errors. It stops syntax errors only. This is an anti-pattern.
- Don't: mix up "auto" and "any". This is an anti-pattern.
- Don't: make every field required. Claude will invent values. This is an anti-pattern.

### 4.4 Validation, retry, and feedback loops

- If validation fails, choose: retry with the original document, the failed output, and the exact error.
- If the error is a format mismatch, structure error, misplaced value, or math error, choose: retry.
- If the data is not in the source, or only in a document you did not give, choose: human review or null, not retry.
- If you want to catch total mismatches automatically, choose: calculated_total and stated_total fields.
- If the document contradicts itself, choose: extract both values and set conflict_detected true.
- If you want to improve review prompts over time, choose: a detected_pattern field and track which patterns developers dismiss.
- If the problem is a syntax error, choose: tool_use. If it is a meaning error, choose: validation logic plus retry.
- If you build the pipeline in Python, choose: Pydantic validators for cross-field rules; feed their errors into the retry.
- Don't: assume retries fix every failure. This is an anti-pattern.
- Don't: retry without the specific error message. This is an anti-pattern.
- Don't: rely on schema checks alone. This is an anti-pattern.
- Don't: drop Pydantic because tool_use already checks the schema. This is an anti-pattern.

### 4.5 Batch processing

- If you need to remember the facts, choose: 50% cheaper, up to 24 hours, no speed guarantee, custom_id to match results. *(Conversion note: the words of this line were scrambled in the PDF; this is the reconstructed order with no words added or removed.)*
- If someone waits for the result (pre-merge check, live review), choose: the synchronous API.
- If results are used later (overnight report, weekly audit, nightly tests, bulk extraction), choose: the Batch API.
- If the SLA is 30 hours, choose: 30 minus 24 leaves 6 hours of buffer, so submit every 4 hours.
- If a batch comes back expired, choose: resubmit it.
- If some items fail, choose: find them by custom_id and resend only those, with fixes (split big documents, simpler prompts, add examples).
- If you plan a large batch, choose: test and refine the prompt on 5 to 10 sample documents first.
- If a step needs tool calls in the middle of processing, choose: the synchronous API [verify: server tools now work in batch].
  - **⚠️ Verify (docs):** confirmed. Batch requests may include tools, including server tools. Your own client-side tool loop still needs a new request per round trip, so it cannot finish inside one batch request (row A8f).
- Don't: move all work to batch to save money. This is an anti-pattern.
- Don't: design around batches that usually finish fast. Plan for 24 hours. This is an anti-pattern.
- Don't: use batch for multi-turn tool calling. This is an anti-pattern.
- Don't: resend the whole batch when only some items failed. This is an anti-pattern.

### 4.6 Multi-instance and multi-pass review

- If Claude must review its own output, choose: a new, independent instance without the generation history.
- If a review covers many files, choose: one pass per file, then one cross-file integration pass.
- If findings have confidence scores, choose: high confidence goes to developers; low confidence goes to human review.
- If you set confidence thresholds, choose: calibrate them on a labeled set first.
- If review quality affects production (CI/CD, finance, compliance), choose: the full multi-pass setup, even if it costs more.
- Don't: ask the same session to "review carefully". This is an anti-pattern.
- Don't: review many files in one pass. This is an anti-pattern.
- Don't: switch to a larger context window to fix uneven review. This is an anti-pattern.
- Don't: route findings with uncalibrated confidence scores. This is an anti-pattern.

**Exam traps (Domain 4):** Clear criteria beat vague words. Examples beat more instructions. Schemas stop syntax errors, not wrong values. Retries cannot create missing data. Batch is never for blocking work.

---

## Domain 5: Context Management and Reliability

### 5.1 Context window management

- If you summarize long chats but need exact facts, choose: a case facts block (amounts, dates, order IDs, statuses) in every prompt, never summarized.
- If a customer has several issues, choose: one separate entry per issue in the facts block.
- If key results are buried in a long input, choose: a key findings summary at the top and clear section headers.
- If a tool returns 40+ fields and you need 5, choose: trim the result in a PostToolUse hook or the tool, before it enters history.
- If you send a request, choose: the full conversation history. The API keeps no state.
- If upstream agents send long reasoning, choose: make them return structured data (facts, citations, relevance, dates).
- If the same long prompt is sent many times, choose: prompt caching (only its existence is tested).
- Don't: summarize transaction data. Numbers and dates get lost. This is an anti-pattern.
- Don't: fix "lost in the middle" by telling Claude to pay attention. Fix the structure. This is an anti-pattern.
- Don't: keep full tool results "in case they are needed later". This is an anti-pattern.
- Don't: cut parts of the history. Use a facts block and summaries instead. This is an anti-pattern.

### 5.2 Escalation and ambiguity resolution

- If the customer asks for a human, choose: escalate at once, with no investigation first.
- If the policy says nothing about the case (policy gap), choose: escalate.
- If the policy clearly says no (policy violation), choose: no escalation; answer with the policy.
- If the agent really tried and cannot make progress, choose: escalate.
- If the customer is upset but the issue is simple, choose: show understanding and offer the fix.
- If the customer asks for a human again after your offer, choose: escalate.
- If a search returns several customers with the same name, choose: ask for more identifiers (email, phone, order number).
- If the agent escalates easy cases and keeps hard ones, choose: clear escalation rules with few-shot examples in the system prompt.
- Don't: escalate based on sentiment or frustration. This is an anti-pattern.
- Don't: escalate based on Claude's self-reported confidence. This is an anti-pattern.
- Don't: try to solve the issue before honoring a request for a human. This is an anti-pattern.
- Don't: pick the most recent or most active customer record. This is an anti-pattern.

### 5.3 Error propagation in multi-agent systems

- If a subagent fails, choose: structured error context. Failure type, what was tried, partial results, and other options.
- If the coordinator gets this context, choose: decide to retry, try another option, continue with partial results, or escalate.
- If the tool could not reach the data (timeout, connection error), choose: consider a retry.
- If the query ran and found nothing, choose: accept it as the answer.
- If the failure is transient, choose: retry inside the subagent with backoff before reporting up.
- If a report has gaps from failed sources, choose: coverage notes that say which areas are limited and why.
- Don't: return empty results marked as success after a timeout. This is the worst anti-pattern.
- Don't: stop the whole pipeline because one subagent failed. This is an anti-pattern.
- Don't: return a general "search unavailable" message after retries. This is an anti-pattern.
- Don't: retry a valid empty result. This is an anti-pattern.

### 5.4 Codebase exploration and context degradation

- If Claude starts saying "typical patterns" instead of exact class names, choose: treat it as context degradation, not a token limit.
- If a long exploration starts, choose: scratchpad files for key findings, from the start.
- If exploration output is long, choose: subagents. The main benefit is a clean main context, not speed.
- If exploration has phases, choose: inject the Phase 1 summary into Phase 2 subagent prompts.
- If the session is long, choose: /compact early and often, not only at the limit.
  - **⚠️ Verify (exam guide):** guide task 5.4 says "Using /compact to reduce context usage ... when context fills with verbose discovery output". The "early and often" advice is not in the guide. See REVIEW_NEEDED.md item R2.
- If a crash could lose progress, choose: a state manifest file (explored paths, findings, phase, next steps) loaded on resume.
- Don't: use a bigger context window to fix degradation. This is an anti-pattern.
- Don't: think subagents are only for parallel work. This is an anti-pattern.
- Don't: restart a session without saving notes first. This is an anti-pattern.
- Don't: use /compact only when context is full. This is an anti-pattern.

### 5.5 Human review and confidence calibration

- If overall accuracy is high (for example, 97%), choose: check accuracy by document type and by field before automating.
- If you automate high-confidence items, choose: stratified random sampling that includes high-confidence items.
- If you route by confidence, choose: field-level scores calibrated on a labeled validation set.
- If a field is above the calibrated threshold, choose: automate it, with sampling.
- If a field is below the threshold, choose: human review. If it is in the unclear zone, choose: priority human review.
- If reviewers are limited, choose: send the most uncertain items first (low confidence, unclear or conflicting sources, weak document types).
- If items keep arriving, choose: reorder the review queue by uncertainty, not by time.
- If you plan to reduce human review, choose: this order. Measure by segment, calibrate, set thresholds, sample, then reduce.
- Don't: use overall accuracy to justify automation. This is an anti-pattern.
- Don't: sample only low-confidence items. This is an anti-pattern.
- Don't: use raw confidence scores without calibration. This is an anti-pattern.
- Don't: spread reviewer time evenly across all items. This is an anti-pattern.

### 5.6 Information provenance and multi-source synthesis

- If a subagent reports a finding, choose: claim, source URL, document name, excerpt, and publication date.
- If a synthesis agent combines findings, choose: a prompt that requires every claim to keep its source link.
- If two good sources give different numbers, choose: show both values with sources and dates; let the reader decide.
- If sources have different dates, choose: check if it is a trend over time, not a contradiction.
- If the data is financial, choose: a table. If it is news, choose: prose. If it is technical, choose: a structured list.
- If an analysis agent finds conflicting values, choose: finish the analysis with the conflict marked; the coordinator decides.
- If the report has strong and weak findings, choose: separate well-established findings from contested ones.
- If you ask where sources usually get lost, choose: the synthesis step.
- Don't: pick the newer source, average the values, or pick the bigger publisher. This is an anti-pattern.
- Don't: treat different numbers as contradictions without checking dates. This is an anti-pattern.
- Don't: let the synthesis agent paraphrase without keeping claim-source links. This is an anti-pattern.
- Don't: put all content in one format. This is an anti-pattern.

**Exam traps (Domain 5):** Summaries lose numbers, so keep a facts block. Mood and self-confidence are not escalation triggers. Silent empty "success" is the worst error pattern. Degradation is not a token-limit problem. High overall accuracy can hide bad segments. Never pick one of two conflicting values.
