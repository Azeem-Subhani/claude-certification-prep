# Domain 2: Tool Design & MCP Integration

**Exam weight:** 18%
**Source:** *Claude Certified Architect – Foundations Exam Guide*, Section 6 (Detailed Objectives by Domain)

The objectives below are paraphrased from the guide for study purposes. Check exact wording against the PDF.

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
| [06 · Practice Questions](06-scenario-practice-questions.md) | 25 original scenario questions with explanations |

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

## My notes

<!-- Add study notes, lab links, and weak areas here -->
