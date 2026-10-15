# Domain 2: Tool Design & MCP Integration

**Exam weight:** 18%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

## Exam focus

- **Tool descriptions (2.1).** Claude picks tools mainly from their descriptions. When Claude picks the wrong tool, the first fix is usually a better description.
- **Error responses (2.2).** You must tell four error types apart, and a failure apart from a valid empty result.
- **Tool distribution and `tool_choice` (2.3).** Each agent should get only the 4 to 5 tools its role needs. You must know what `"auto"`, `"any"`, and forced selection do.
- **MCP configuration (2.4).** You must know where team and personal servers go. You must also know how to keep secrets out of git, and when to use resources.
- **Built-in tools (2.5).** Use Grep to search file contents and Glob to find files by name. Use Edit for small changes, with Read + Write as the fallback.

## Exam scenarios where this domain is primary

- Scenario 1: Customer Support Resolution Agent
- Scenario 3: Multi-Agent Research System
- Scenario 4: Developer Productivity with Claude

## Related guide material

- Preparation Exercise 1: Build a Multi-Tool Agent with Escalation Logic (tool descriptions, structured errors)
- Preparation Exercise 2: Configure Claude Code for a Team Development Workflow (`.mcp.json`, `~/.claude.json`)
- Preparation Exercise 4: Design and Debug a Multi-Agent Research Pipeline (error propagation)
- Sample Questions 2 (tool descriptions), 8 (error propagation), 9 (scoped cross-role tool)

## Study notes (read in order)

| File | Covers |
| --- | --- |
| [00 · Foundations and Glossary](00-foundations-and-glossary.md) | Tool definition anatomy, MCP architecture, tools vs resources vs prompts |
| [01 · Tool Interface Design](01-tool-interface-design.md) | 2.1: descriptions, naming, splitting tools, system-prompt keyword traps |
| [02 · Structured Error Responses](02-structured-error-responses.md) | 2.2: `isError`, error categories, retryable flags, local recovery, empty vs failed |
| [03 · Tool Distribution and `tool_choice`](03-tool-distribution-and-tool-choice.md) | 2.3: scoped tool sets, constrained tools, `auto` / `any` / forced selection |
| [04 · MCP Server Integration](04-mcp-server-integration.md) | 2.4: `.mcp.json` vs `~/.claude.json`, env var expansion, resources, community servers |
| [05 · Built-in Tools](05-built-in-tools.md) | 2.5: Read, Write, Edit, Bash, Grep, Glob and incremental exploration |
| [Anti-patterns](anti-patterns.md) | Wrong approaches, why they fail, what to do instead, and exam traps |
| [MCQ bank: Domain 2](../mcq-bank/02-tool-design-mcp.md) | 43 exam-level practice questions with answer key and explanations |

Sources: the exam guide PDF, plus [Claude Code MCP docs](https://code.claude.com/docs/en/mcp), [MCP specification: Tools](https://modelcontextprotocol.io/specification/2025-06-18/server/tools), and Anthropic's tool-use docs for `tool_choice`. Where the guide and current product behavior differ (e.g. forced `tool_choice` on the newest models), the notes give both.

---

## 2.1 Design effective tool interfaces with clear descriptions and boundaries

**Know**
- [ ] Tool descriptions are the main way the model chooses a tool; minimal descriptions cause unreliable selection between similar tools
- [ ] Good descriptions include input formats, example queries, edge cases, and boundaries
- [ ] Ambiguous or overlapping descriptions cause misrouting (e.g. `analyze_content` vs `analyze_document`)
- [ ] System prompt wording can create unintended keyword-based tool associations

**Do**
- [ ] Write descriptions that state purpose, inputs, outputs, and when to use this tool vs similar ones
- [ ] Rename tools and rewrite descriptions to remove overlap (e.g. `analyze_content` → `extract_web_results`)
- [ ] Split generic tools into purpose-specific tools with clear input/output contracts
- [ ] Review system prompts for keyword-sensitive instructions that override good tool descriptions

## 2.2 Implement structured error responses for MCP tools

**Know**
- [ ] The MCP `isError` flag pattern for reporting tool failures to the agent
- [ ] Error categories: transient, validation, business, permission
- [ ] Generic errors ("Operation failed") prevent good recovery decisions
- [ ] Retryable vs non-retryable errors; metadata prevents wasted retries

**Do**
- [ ] Return structured metadata: `errorCategory`, `isRetryable`, human-readable description
- [ ] For business-rule violations, return `isRetryable: false` plus a customer-friendly explanation
- [ ] Recover locally inside subagents for transient failures; escalate only unresolved errors, with partial results and what was tried
- [ ] Distinguish access failures (need a retry decision) from valid empty results (query succeeded, no matches)

## 2.3 Distribute tools appropriately across agents and configure tool choice

**Know**
- [ ] Too many tools (e.g. 18 instead of 4–5) makes tool selection less reliable
- [ ] Agents misuse tools outside their specialization (e.g. a synthesis agent doing web searches)
- [ ] Scoped access: only role-relevant tools, plus limited cross-role tools for frequent needs
- [ ] `tool_choice` options: `"auto"`, `"any"`, forced `{"type": "tool", "name": "..."}`

**Do**
- [ ] Restrict each subagent's tools to its role
- [ ] Replace generic tools with constrained ones (e.g. `fetch_url` → `load_document` that validates URLs)
- [ ] Give scoped cross-role tools for frequent needs (e.g. `verify_fact` for the synthesis agent); route complex cases via the coordinator
- [ ] Use forced selection to make a specific tool run first (e.g. `extract_metadata`), then continue in later turns
- [ ] Use `tool_choice: "any"` to guarantee a tool call instead of a text reply

## 2.4 Integrate MCP servers into Claude Code and agent workflows

**Know**
- [ ] Scoping: project-level `.mcp.json` (shared team tooling) vs user-level `~/.claude.json` (personal/experimental)
- [ ] Environment variable expansion in `.mcp.json` (e.g. `${GITHUB_TOKEN}`) keeps secrets out of the repo
- [ ] Tools from all configured MCP servers are discovered at connection time and available at the same time
- [ ] MCP resources expose content catalogs (issue summaries, docs hierarchies, DB schemas) to reduce exploratory tool calls

**Do**
- [ ] Configure shared servers in `.mcp.json` with env var expansion for tokens
- [ ] Configure personal/experimental servers in `~/.claude.json`
- [ ] Improve MCP tool descriptions so the agent doesn't prefer weaker built-in tools (like Grep)
- [ ] Prefer existing community MCP servers for standard integrations (e.g. Jira); build custom servers only for team-specific workflows
- [ ] Expose content catalogs as MCP resources

## 2.5 Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively

**Know**
- [ ] Grep = search file **contents** (function names, error messages, imports)
- [ ] Glob = match file **paths** (names, extensions)
- [ ] Read/Write = full-file operations; Edit = targeted change using a unique text match
- [ ] If Edit fails on a non-unique match, fall back to Read + Write

**Do**
- [ ] Use Grep to find all callers of a function or locate an error message
- [ ] Use Glob for patterns like `**/*.test.tsx`
- [ ] Read the full file, then Write, when Edit can't find a unique anchor
- [ ] Build understanding incrementally: Grep for entry points, then Read to follow imports, not read everything upfront
- [ ] Trace usage through wrapper modules: find all exported names first, then search for each

---

## Key themes to remember

- **The description is the interface.** The model chooses tools by reading descriptions, so make them specific and non-overlapping.
- **Errors need structure.** Category + retryable flag + clear message lets the agent recover correctly.
- **Fewer, focused tools per agent** beat one agent with everything.
- **Shared config in `.mcp.json`, secrets in env vars, personal config in `~/.claude.json`.**
- **Grep for content, Glob for paths, Edit for unique targeted changes, Read + Write as fallback.**
- **Fix descriptions first**, then check the system prompt for keywords that pull Claude to the wrong tool.
- **`isError: true` is in the MCP spec.** `errorCategory` and `isRetryable` are an app convention.
- **Retry transient errors. Fix validation errors.** Do not retry business or permission errors. Never retry a valid empty result.
- **`"auto"` = optional, `"any"` = must call some tool, forced = must call a named tool.** ⚠️ The newest models reject `"any"` and forced choice.
- **Use resources for catalogs** and community servers for standard systems.

## Worked scenario

**Setting:** A multi-agent research system has a coordinator and three subagents: web search, document analysis, and synthesis.

**Problem 1: wrong tool choice.**
- The document analysis agent often calls `analyze_content` when it should call `analyze_document`.
- Both descriptions are one line long and nearly the same.
- **Fix:** rename `analyze_content` to `extract_web_results`. Rewrite both descriptions to cover all five parts of a good description.

**Problem 2: too many tools.**
- The synthesis agent has 14 tools, including web search.
- It searches when it should be writing.
- **Fix:** give it only its synthesis tools, plus one scoped `verify_fact` tool for quick checks. Send complex checks to the coordinator.

**Problem 3: silent failures.**
- The web search agent hits a timeout.
- It returns `[]`, marked as success.
- The final report says "no sources found" for a topic that has many sources.
- **Fix:**
  - The search tool returns `isError: true` with `errorCategory: "transient"` and `isRetryable: true`.
  - The subagent retries twice with backoff.
  - It then reports to the coordinator with the query, the attempts, and any partial results.
  - The report adds a coverage note for the gap.

**Problem 4: configuration.**
- The team adds a shared citation-database server to `.mcp.json`.
- The token is set as `${CITATION_DB_TOKEN}`.
- A developer tries an experimental PDF server in their own `~/.claude.json`, so teammates are not affected.

**Problem 5: exploring the data.**
- The analysis agent makes 6 calls just to learn which documents exist.
- **Fix:** the server exposes a document catalog as an MCP **resource**, so the agent can see the list without calling tools.

## Sources checked

Each technical claim in this folder was checked against official docs on 2026-10-15.
**Verdicts:** CONFIRMED = the docs agree. PARTIAL = they agree with limits. CONFLICT = the docs say something different. NOT FOUND = the docs do not say this.
Conflicts with the exam guide are listed in [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).

| ID | Claim | Verdict | What the docs say | Source |
|---|---|---|---|---|
| A5 | `tool_choice` options | CONFIRMED | `auto` (default when tools exist), `any`, `tool` (named), `none` (default when no tools). | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| A5b | Forced tool use works on all models | **CONFLICT** | Manual extended thinking does not support `any` or `tool`. **Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1 return a 400 error for `any` and `tool`.** The docs say to use `auto` with `strict: true` or structured outputs instead. On models that accept it, a forced call skips thinking. | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| A5c | With `any`/`tool`, Claude writes no text before the tool call | CONFIRMED | The API prefills the response to force the tool call. | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| A6 | Tool definitions have `name`, `description`, and `input_schema` | CONFIRMED | Optional `input_examples` also exists. The description is "by far the most important factor". Write at least 3 to 4 sentences. | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| A6b | Tool design advice | PARTIAL | The docs also advise grouping related actions into one tool with an `action` parameter, and returning only high-signal fields. | platform.claude.com/docs/en/agents-and-tools/tool-use/define-tools (read in full) |
| B1 | Tool execution errors use `isError: true` in the result | CONFIRMED | Examples: API failures, input validation errors, business logic errors. The model can see them and self-correct. | modelcontextprotocol.io/specification/2025-11-25/server/tools (same in 2026-07-28) |
| B2 | Protocol errors are JSON-RPC errors | CONFIRMED | Examples: unknown tool, malformed request, server error. | modelcontextprotocol.io/specification/2025-11-25/server/tools (same in 2026-07-28) |
| B2b | "Bad arguments" are protocol errors (cheat sheet 2.2) | **CONFLICT** | The spec lists **input validation errors** as **tool execution errors** (`isError: true`), not protocol errors. | modelcontextprotocol.io/specification/2025-11-25/server/tools (same in 2026-07-28) |
| B3 | `isRetryable` and `errorCategory` are not in the MCP spec | CONFIRMED (absent) | They do not appear on the tools pages. They are an app-level convention that the exam guide uses. | modelcontextprotocol.io/specification/2025-11-25/server/tools (same in 2026-07-28) |
| B4 | Primitives | CONFIRMED | Tools are model-controlled. Resources are application-driven and identified by URI. Prompts are user-controlled. | modelcontextprotocol.io/specification/2025-11-25/server/resources, .../server/prompts |
| B5 | Transports | CONFIRMED | stdio and Streamable HTTP. The older HTTP+SSE transport is deprecated. | modelcontextprotocol.io/specification/2025-11-25/basic/transports; modelcontextprotocol.io/specification/2026-07-28/deprecated |
| C14 | Edit needs a unique match | CONFIRMED | `old_string` must appear exactly once. Otherwise, add more surrounding text or set `replace_all: true`. | code.claude.com/docs/en/tools-reference |
| C15a | MCP scopes | CONFIRMED | Three scopes: local (default) and user are stored in `~/.claude.json`. Project scope is stored in `.mcp.json` and is meant for git. | code.claude.com/docs/en/mcp |
| C15b | Remote vs local server config | CONFIRMED | `"type": "http"` + `"url"`. stdio servers use `command` + `args`. | code.claude.com/docs/en/mcp |
| C15c | `${VAR}` and `${VAR:-default}` | CONFIRMED | Works in `command`, `args`, `env`, `url`, `headers`. | code.claude.com/docs/en/mcp |
| C15d | All servers' tools are available at once | PARTIAL | All connected servers' tools are offered together. But by default, tool search loads full schemas only when they are needed. | code.claude.com/docs/en/mcp |
