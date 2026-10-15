# Verification Log

**Purpose:** record every technical claim used in the study notes and MCQ bank, the official page it was checked against, and the result.
**Checked on:** 2026-10-15.
**Owner:** Azeem Subhani (repo owner). Human review is needed before anyone relies on this for client or production work.

## How this log was made

- Pages were fetched from official sources only: `platform.claude.com` (the Claude API docs, formerly docs.claude.com), `code.claude.com` (Claude Code and the Agent SDK), and `modelcontextprotocol.io` (the MCP specification).
- Most pages were read through a summarizing fetch tool. So the paraphrases below are close but not word-for-word. Before you quote any wording exactly, open the page.
- Exception: the `tool_choice` row (A5) was read in full from the raw page.
- The official exam guide (`Claude Certified Architect Foundations Exam Guide.pdf`, version 1.0, July 2026) is the authority for **what the exam tests**. The docs are the authority for **how the product works today**. When they differ, the notes teach the exam answer and add a ⚠️ product note. Every difference is also listed in [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).

Verdicts: **CONFIRMED** = docs agree. **PARTIAL** = docs agree with limits. **CONFLICT** = docs say something different. **NOT FOUND** = docs do not say this.

## A. Claude API (Messages API)

| ID | Claim | Verdict | What the docs say | Source |
|---|---|---|---|---|
| A1 | The API is stateless. Each request must include the full history. | CONFIRMED | You always send the full conversation history. | platform.claude.com/docs/en/build-with-claude/working-with-messages |
| A2 | `stop_reason` values | CONFIRMED | Seven values: `end_turn`, `max_tokens`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal`, `model_context_window_exceeded`. | platform.claude.com/docs/en/build-with-claude/handling-stop-reasons |
| A2b | What to do on `pause_turn` | CONFIRMED | A server-side tool loop hit its iteration limit. Send the assistant response back as it is, and make another request. Your own (client) tools never cause `pause_turn`. | same |
| A2c | `refusal` | CONFIRMED | The model declined. The API still returns HTTP 200. | same |
| A3a | `tool_result` blocks must come first in the user message | CONFIRMED | Any text must come after all tool results, or the API returns a 400 error. | platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls |
| A3b | One response can contain text and `tool_use` blocks together | CONFIRMED | Example responses show a text block followed by a `tool_use` block. | define-tools; handle-tool-calls |
| A3c | One response can contain several `tool_use` blocks (parallel tool use) | CONFIRMED | Every `tool_use` block needs a matching `tool_result` in the next user message. | platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use |
| A3d | `disable_parallel_tool_use` | CONFIRMED | It is set inside `tool_choice`, not at the top level. | same |
| A4 | `tool_result` has an `is_error` field | CONFIRMED | Optional boolean. Set it to true when the tool fails while running. | handle-tool-calls |
| A5 | `tool_choice` options | CONFIRMED | `auto` (default when tools exist), `any`, `tool` (named), `none` (default when no tools). | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| A5b | Forced tool use works on all models | **CONFLICT** | Manual extended thinking does not support `any` or `tool`. **Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1 return a 400 error for `any` and `tool`.** The docs say to use `auto` with `strict: true` or structured outputs instead. On models that accept it, a forced call skips thinking. | same |
| A5c | With `any`/`tool`, Claude writes no text before the tool call | CONFIRMED | The API prefills the response to force the tool call. | same |
| A6 | Tool definitions have `name`, `description`, and `input_schema` | CONFIRMED | Optional `input_examples` also exists. The description is "by far the most important factor". Write at least 3 to 4 sentences. | same |
| A6b | Tool design advice | PARTIAL | The docs also advise grouping related actions into one tool with an `action` parameter, and returning only high-signal fields. | same |
| A7a | `strict: true` on tools | CONFIRMED | Uses grammar-constrained sampling, so tool inputs match the schema. | platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use |
| A7b | JSON output parameter is `output_config.format` | CONFIRMED (summarized) | `output_config.format` with type `json_schema`. The older `output_format` is deprecated. **VERIFY the exact wording.** | platform.claude.com/docs/en/build-with-claude/structured-outputs |
| A7c | Structured outputs always match the schema | PARTIAL | Exceptions exist: refusals, `max_tokens` cut-offs, and some edge cases. Keep validation in code. | same |
| A8a | Batch API costs 50% less | CONFIRMED | All usage is charged at 50% of standard prices. | platform.claude.com/docs/en/build-with-claude/batch-processing |
| A8b | Batch timing | CONFIRMED | Most batches finish within 1 hour. Requests not finished after 24 hours expire. | same |
| A8c | `custom_id` | CONFIRMED | Results can come back in any order. Match them by `custom_id`. | same |
| A8d | Results kept for 29 days; limit of 100,000 requests or 256 MB per batch | CONFIRMED | | same |
| A8e | Result types | CONFIRMED | `succeeded`, `errored`, `canceled`, `expired`. Requests that do not succeed are not billed. | same |
| A8f | Batches cannot use tools | **CONFLICT (partial)** | Tool use, including server tools, **is** supported in batch requests. But you cannot run your own client-side tool loop inside one batch request: the round trip needs a new request. | same |
| A9 | Prompt caching exists | CONFIRMED | Uses `cache_control` breakpoints. The default cache lifetime is 5 minutes, with an optional 1-hour TTL (time to live). Cache reads cost much less than normal input. (The exam tests only that caching exists.) | platform.claude.com/docs/en/build-with-claude/prompt-caching |
| A10 | SDK Tool Runner | CONFIRMED (beta) | `client.beta.messages.tool_runner` (Python) runs the loop for you. For human approval steps, the docs suggest a manual loop. | platform.claude.com/docs/en/agents-and-tools/tool-use/tool-runner |

## B. MCP specification

| ID | Claim | Verdict | What the spec says | Source |
|---|---|---|---|---|
| B1 | Tool execution errors use `isError: true` in the result | CONFIRMED | Examples: API failures, input validation errors, business logic errors. The model can see them and self-correct. | modelcontextprotocol.io/specification/2025-11-25/server/tools (same in 2026-07-28) |
| B2 | Protocol errors are JSON-RPC errors | CONFIRMED | Examples: unknown tool, malformed request, server error. | same |
| B2b | "Bad arguments" are protocol errors (cheat sheet 2.2) | **CONFLICT** | The spec lists **input validation errors** as **tool execution errors** (`isError: true`), not protocol errors. | same |
| B3 | `isRetryable` and `errorCategory` are not in the MCP spec | CONFIRMED (absent) | They do not appear on the tools pages. They are an app-level convention that the exam guide uses. | same |
| B4 | Primitives | CONFIRMED | Tools are model-controlled. Resources are application-driven and identified by URI. Prompts are user-controlled. | .../server/resources, .../server/prompts |
| B5 | Transports | CONFIRMED | stdio and Streamable HTTP. The older HTTP+SSE transport is deprecated. | .../basic/transports; 2026-07-28/deprecated |

## C. Claude Code and the Agent SDK

| ID | Claim | Verdict | What the docs say | Source |
|---|---|---|---|---|
| C1a | The subagent tool is "Task" | PARTIAL | It was renamed **"Agent"** (v2.1.63). "Task" still works as an alias. | code.claude.com/docs/en/sub-agents; .../agent-sdk/subagents |
| C1b | "Task"/"Agent" must be in `allowedTools` for the coordinator to spawn | PARTIAL | Every SDK example lists "Agent" in `allowedTools` (to auto-approve). The docs do not state it as a hard rule. The exam guide does. | .../agent-sdk/subagents |
| C2 | `AgentDefinition` fields | CONFIRMED | `description` and `prompt` are required. Optional: `tools`, `disallowedTools`, `model`, `maxTurns`, `skills`, `mcpServers`, and more. | same |
| C3 | Subagents cannot spawn subagents | **CONFLICT** | By default, a subagent **can** spawn its own subagents, up to three layers deep. This can be turned off. | .../sub-agents |
| C4 | Resume and fork | CONFIRMED | CLI: `-r/--resume` (by ID or name), `-c/--continue` (latest session in this folder), `--fork-session`. SDK: `resume`, `fork_session` (Python) / `forkSession` (TypeScript). | .../cli-reference; .../agent-sdk/sessions |
| C4b | Named sessions | CONFIRMED | `--name`/`-n` at start, `/rename` during a session, `claude -r <name>` to resume. | .../sessions |
| C5a | Hook events | CONFIRMED | Includes PreToolUse, PostToolUse, PostToolUseFailure, PreCompact, PostCompact, Stop, SubagentStart, SubagentStop, SessionStart, SessionEnd, UserPromptSubmit, and others. | .../agent-sdk/hooks |
| C5b | PreToolUse can block or change a call | CONFIRMED | `permissionDecision` (allow / deny / ask / defer) and `updatedInput`. | same |
| C5c | PostToolUse runs after the tool has run, and can change what Claude sees | CONFIRMED | `updatedToolOutput` replaces the result. `additionalContext` adds to it. | same |
| C6a | CLAUDE.md locations | CONFIRMED | User: `~/.claude/CLAUDE.md`. Project: `./CLAUDE.md` or `./.claude/CLAUDE.md`. Subfolder files load when Claude reads files there. | code.claude.com/docs/en/memory |
| C6b | `CLAUDE.local.md` | CONFIRMED | Supported. Personal project notes, kept out of git. | same |
| C6c | Imports use `@path` | CONFIRMED | Paths resolve relative to the importing file. Up to four levels of nested imports. | same |
| C6d | "More specific CLAUDE.md wins" | NOT FOUND | Files load from broad to specific. Conflicting instructions may be followed unpredictably. | same |
| C7 | `.claude/rules/` with `paths:` frontmatter | CONFIRMED | The rule loads only when Claude reads or edits a matching file. Rules without `paths` load at start. | same |
| C8 | `/memory` | CONFIRMED | Lists the memory files and opens them for editing. `/context` shows context use, including loaded memory files. | .../commands; .../memory |
| C9a | Skills: `.claude/skills/<name>/SKILL.md` | CONFIRMED | `.claude/commands/<name>.md` still works. | code.claude.com/docs/en/skills |
| C9b | Skill frontmatter | CONFIRMED | Includes `name`, `description`, `argument-hint`, `allowed-tools`, `disallowed-tools`, `context: fork`, `agent`, `model`, `disable-model-invocation`, `paths`, and more. | same |
| C9c | `allowed-tools` **restricts** tools | **CONFLICT** | It **pre-approves** the listed tools. Other tools can still be called. Use `disallowed-tools` to remove tools. | same |
| C10 | Plan mode | CONFIRMED | Shift+Tab cycles modes (default → acceptEdits → plan). `--permission-mode plan`. `/plan`. | .../permission-modes; .../commands |
| C10b | Explore subagent is read-only | CONFIRMED | Write and Edit are denied. | .../sub-agents |
| C11 | `/compact` and auto-compaction | CONFIRMED | `/compact [instructions]` summarizes. Auto-compaction runs at a threshold. | .../commands; .../context-window |
| C12 | CLI flags | CONFIRMED | `-p/--print`, `--output-format text|json|stream-json`, `--json-schema` (result in `structured_output`), `--append-system-prompt`, `--system-prompt`, `--max-turns` (print mode), `--allowedTools`, `--disallowedTools`, `--bare`, `--permission-mode`. | .../cli-reference; .../headless |
| C13 | GitHub Actions | CONFIRMED | `anthropics/claude-code-action@v1`, with CLI flags passed in `claude_args`. | .../github-actions |
| C14 | Edit needs a unique match | CONFIRMED | `old_string` must appear exactly once. Otherwise, add more surrounding text or set `replace_all: true`. | .../tools-reference |
| C15a | MCP scopes | CONFIRMED | Three scopes: local (default) and user are stored in `~/.claude.json`. Project scope is stored in `.mcp.json` and is meant for git. | code.claude.com/docs/en/mcp |
| C15b | Remote vs local server config | CONFIRMED | `"type": "http"` + `"url"`. stdio servers use `command` + `args`. | same |
| C15c | `${VAR}` and `${VAR:-default}` | CONFIRMED | Works in `command`, `args`, `env`, `url`, `headers`. | same |
| C15d | All servers' tools are available at once | PARTIAL | All connected servers' tools are offered together. But by default, tool search loads full schemas only when they are needed. | same |
| C17 | Settings files | CONFIRMED | `~/.claude/settings.json` (you, all projects), `.claude/settings.json` (shared, committed), `.claude/settings.local.json` (you, this project, gitignored), managed settings (organization). Precedence, highest first: managed, command line, project local, shared project, user. | code.claude.com/docs/en/settings |
| C18 | Permission rules | CONFIRMED | `permissions.allow` / `deny` lists, for example `"Bash(npm run test *)"` and `"Read(./.env)"`. The hooks page says to use the permission system, not a hook, to enforce a hard allow or deny. | code.claude.com/docs/en/settings; code.claude.com/docs/en/hooks |
| C19 | Claude Code hooks | CONFIRMED | Hooks are set in settings files. A PreToolUse command hook blocks a call by exiting with code 2, or by returning `permissionDecision: "deny"`. A hook that times out does not block by default. The docs call CLAUDE.md the place for static conventions. | code.claude.com/docs/en/hooks |
| C16 | Git worktrees | CONFIRMED | `--worktree` / `-w` starts Claude in its own worktree. | .../cli-reference |

## D. Exam facts (from the official exam guide PDF only)

| Fact | Value | Guide section |
|---|---|---|
| Exam code | CCAR-F | §3 |
| Number of items | 60 | §3 |
| Item format | Multiple-choice **and multiple-response** (each item says how many answers to pick) | §3 |
| Structure | 4 scenarios, picked at random from a bank of 6 | §3, §5 |
| Time limit | 120 minutes | §3 |
| Passing score | Scaled score of 720 (scale 100 to 1,000) | §3, §10 |
| Fee | $125 USD | §3 |
| Validity | 12 months | §3 |
| Domain weights | D1 27%, D2 18%, D3 20%, D4 20%, D5 15% | §4 |
| Guide version | 1.0, effective July 2026 | cover, §18 |

Anything about the exam that is **not** in this table is not confirmed. Mark it **VERIFY**.
