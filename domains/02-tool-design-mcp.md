# Domain 2: Tool Design & MCP Integration

> **Exam weight:** 18% of scored items (official exam guide v1.0, §4).
> **Task statements:** 2.1 to 2.5 (see [docs/domain-map.md](../docs/domain-map.md)).
> **Deep dives:** [domain-2-tool-design-mcp-integration/](../domain-2-tool-design-mcp-integration/README.md)
> **Facts checked against:** [docs/verification-log.md](../docs/verification-log.md). ⚠️ marks a place where the exam answer and today's product differ.

---

## Exam focus

- **Tool descriptions (2.1).** Claude picks tools mainly from their descriptions. When Claude picks the wrong tool, the first fix is usually a better description.
- **Error responses (2.2).** You must tell four error types apart, and a failure apart from a valid empty result.
- **Tool distribution and `tool_choice` (2.3).** Each agent should get only the 4 to 5 tools its role needs. You must know what `"auto"`, `"any"`, and forced selection do.
- **MCP configuration (2.4).** You must know where team and personal servers go. You must also know how to keep secrets out of git, and when to use resources.
- **Built-in tools (2.5).** Use Grep to search file contents and Glob to find files by name. Use Edit for small changes, with Read + Write as the fallback.

---

## Key concepts

### 1. Words you need first

A **tool definition** has three parts:
- a `name`
- a `description` in plain text
- an `input_schema`, which is a JSON Schema for the inputs

**MCP (Model Context Protocol)** is an open standard for connecting AI apps to outside tools and data. An **MCP server** is a program that offers tools, resources, and prompts. Claude Code or your agent is the **MCP client** that connects to it.

MCP has three kinds of things a server can offer:

| Primitive | Who controls it | Use it for |
|---|---|---|
| **Tools** | The model decides when to call them. | Actions: look up an order, create a ticket. |
| **Resources** | The application chooses what to show. Each has a URI (an address). | Content to read: an issue list, a document tree, a database schema. |
| **Prompts** | The user picks them, often as slash commands. | Reusable prompt templates. |

### 2. Tool interface design (Task 2.1)

The **description is the main signal** Claude uses to choose a tool. The official docs say it is "by far the most important factor" and suggest at least 3 to 4 sentences.

A good description covers five things:

1. What the tool does.
2. What inputs it needs, and in what format.
3. Example queries it is good for.
4. Limits and edge cases (what it does **not** return).
5. When to use a different tool instead.

**Overlap causes misrouting.** Suppose two tools have almost the same description, for example `analyze_content` and `analyze_document`. Then Claude guesses. Fix it by:
- **renaming** a tool so the name shows its purpose (for example, `extract_web_results`)
- **rewriting** descriptions so the boundary is clear
- **splitting** a generic tool into narrow tools with clear input and output contracts (for example, `extract_data_points`, `summarize_content`, `verify_claim_against_source`)

**Check the system prompt too.** A line like "For anything about refunds, check the customer record" can pull Claude to the wrong tool. This can happen even when the descriptions are good.

**The usual order of fixes** (lowest effort first):
1. Improve the descriptions.
2. Check the system prompt for misleading keywords.
3. Only then add few-shot examples, merge tools, or build a router.

> ⚠️ **Product note:** the API docs also suggest grouping closely related actions into one tool with an `action` parameter (verification log A6b). This fits the cheat sheet rule "many versions of one job → one tool with a type parameter". It does not conflict with splitting a vague, do-everything tool.

### 3. Structured error responses (Task 2.2)

In MCP, a tool that fails while running returns a normal result with **`isError: true`**. Claude can read the error and plan what to do next. This is part of the MCP spec.

Many teams also add their own fields. The exam guide uses `errorCategory` and `isRetryable`. **These are an app convention, not part of the MCP spec** (verification log B3).

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "{\"errorCategory\": \"business\", \"isRetryable\": false, \"message\": \"Refunds over $500 need manager approval.\", \"customerMessage\": \"I need to pass this to a specialist.\"}"
  }]
}
```

**Four error types:**

| Type | Example | `isRetryable` | What the agent should do |
|---|---|---|---|
| **Transient** (temporary) | Timeout, service down | `true` | Wait, then retry the same call. |
| **Validation** | Wrong date format, missing field | `false` | Fix the input, then make a new call. |
| **Business** | Over a policy limit | `false` | Take another path or escalate. Explain it kindly to the customer. |
| **Permission** | Access denied | `false` | Escalate or use other credentials. |

`isRetryable: false` does **not** always mean "give up". A validation error can be fixed and sent again as a new, corrected call.

**Failure vs empty result.** A query that ran and found nothing is a **success** with zero results (`isError: false`). Do not retry it. A query that could not reach the data (timeout) is a **failure**. Never return an empty list marked as success after a timeout. The agent would then believe "there is no data".

**Local recovery first.** A subagent should retry a transient error itself, with **backoff** (a longer wait after each try). If it still fails, it reports to the coordinator with what it tried and any partial results.

> ⚠️ **Spec note:** the cheat sheet puts "bad arguments" under protocol errors. The MCP spec lists **input validation errors** as tool execution errors (`isError: true`). Unknown tools and malformed requests are **protocol errors**: JSON-RPC errors that the host handles. (JSON-RPC is the message format that MCP uses.) See verification log B2b.

### 4. Tool distribution and `tool_choice` (Task 2.3)

**More tools make selection worse.** An agent with 18 tools makes more wrong choices than an agent with 4 to 5. Give each agent only the tools its role needs.

Agents also **misuse tools outside their role**. For example, a synthesis agent with a web search tool starts searching instead of synthesizing.

**Scoped tools:**
- Replace a generic tool with a constrained one. For example, use `load_document`, which only accepts approved document URLs, instead of an open `fetch_url`.
- If an agent often needs a simple check from another role, give it one small, scoped tool, for example `verify_fact`. Send complex cases through the coordinator.

**`tool_choice`** controls whether Claude must call a tool:

| Setting | Meaning | Typical use |
|---|---|---|
| `"auto"` (default) | Claude may call a tool or answer in text. | Normal agent loops |
| `"any"` | Claude must call a tool, but it chooses which one. | You need structured output, and you have several extraction schemas for unknown document types. |
| `{"type": "tool", "name": "..."}` (forced) | Claude must call this exact tool. | One step must always run first (for example, `extract_metadata`). Switch back to `"auto"` for later steps. |
| `"none"` | Claude may not call tools. | Not tested on the exam |

> ⚠️ **Product note (important):** on **Claude Opus 5.5, Sonnet 5.5, Fable 5.1, and Mythos 5.1**, `"any"` and forced tool choice return a **400 error**. The docs suggest `"auto"` with `strict: true`, or structured outputs, instead. Manual extended thinking also blocks `"any"` and forced choice. The exam guide (July 2026) still tests `"any"` and forced selection as correct answers. Learn both. See verification log A5b and [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item R3.

### 5. MCP servers in Claude Code (Task 2.4)

**Where to configure a server:**

| Need | Place | Shared? |
|---|---|---|
| The whole team needs it | `.mcp.json` in the project root | Yes, committed to git |
| Only you need it, or you are testing it | `~/.claude.json` (user scope) | No |

> ⚠️ **Product note:** the docs show three scopes: **local** (the default; only you, only this project), **project**, and **user**. Local and user scopes are both stored in `~/.claude.json`. The exam guide uses two levels (verification log C15a).

**Keep secrets out of git.** Use environment variable expansion in `.mcp.json`:

```json
{
  "mcpServers": {
    "tickets": {
      "type": "http",
      "url": "https://tickets.example.com/mcp",
      "headers": { "Authorization": "Bearer ${TICKETS_TOKEN}" }
    },
    "local-search": {
      "command": "node",
      "args": ["./tools/search-server.js"],
      "env": { "INDEX_PATH": "${INDEX_PATH:-./index}" }
    }
  }
}
```

`${VAR}` reads a variable. `${VAR:-default}` uses a default value when the variable is not set.

**All servers work together.** When several servers are configured, their tools are offered to the agent together. Moving tools to a second server does not reduce the choice problem.

**Resources reduce exploring.** Sometimes the agent wastes calls just to learn what data exists. Then expose a **resource**, such as an issue list or a database schema.

**Community before custom.** For standard systems (Jira, GitHub, Slack), use an existing community server. Build a custom server only for a workflow that is unique to your team.

**Description quality matters for MCP tools too.** Claude may keep using the built-in Grep instead of your better MCP search tool. Then write a richer description for the MCP tool.

### 6. Built-in tools (Task 2.5)

| Tool | Use it to | Example |
|---|---|---|
| **Grep** | Search **inside** files for text or patterns | Find all callers of `calculate_tax`. Find an error message. |
| **Glob** | Find files by **name or path pattern** | `**/*.test.tsx` |
| **Read** | Load a whole file | Read a config file |
| **Write** | Write a whole file | Create a new file |
| **Edit** | Change part of a file by matching unique text | Change one function |
| **Bash** | Run shell commands | Run the tests |

**Edit needs a unique match.** If the text appears more than once, Edit fails. The exam's fallback is **Read the full file, then Write** the new version.

**Explore step by step.** Do not read every file first. Instead:
1. Grep for the entry points.
2. Read those files and follow the imports.
3. Grep again for new names you find.

**Re-exports.** A function may be re-exported under another name, for example in an `__init__.py` file. Read the wrapper file, list every exported name, and then Grep for each name.

---

## Common traps

| Tempting wrong answer | Why it is wrong |
|---|---|
| "Add few-shot examples so Claude picks the right tool." | This costs tokens and misses the cause. Fix the short or overlapping descriptions first. |
| "Build a routing classifier." | Too much work for a first step. Descriptions are cheaper and fix the cause. |
| "Merge the two similar tools." | Possible later, but more costly than clear descriptions and names. |
| "Return `Operation failed` and let Claude figure it out." | Claude cannot choose between retry, fix, or escalate without a category. |
| "Retry the business-rule error." | The same rule will fail again. Take another path or escalate. |
| "Mark validation errors as retryable." | Sending the same bad input again fails again. The input must change. |
| "Retry the empty result." | The query worked. Zero results is the answer. |
| "Return an empty list after a timeout so the pipeline continues." | This hides a failure as a success. It is the worst error pattern. |
| "Give every agent all 18 tools so they are flexible." | Selection gets worse, and agents misuse tools outside their role. |
| "Use `\"auto\"` when you must always get structured output." | With `"auto"`, Claude may answer in text. (On the exam, use `"any"` or forced.) |
| "Put the team's server in `~/.claude.json`." | That file is personal. Teammates will not get it. |
| "Commit the API token in `.mcp.json`." | Secrets leak into git. Use `${VAR}`. |
| "Build a custom Jira server." | A community server already exists. Build custom servers only for unique workflows. |
| "Use Glob to find who calls a function." | Glob matches file names, not file contents. Use Grep. |
| "Read every file first to understand the codebase." | This fills the context. Explore step by step. |

---

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

---

## Quick recap

- Descriptions are the main tool-selection signal. Fix them first, then check the system prompt for misleading keywords.
- Split vague tools, rename overlapping ones, and give each agent 4 to 5 role-specific tools.
- `isError: true` is in the MCP spec. `errorCategory` and `isRetryable` are an app convention.
- Retry transient errors. Fix validation errors. Do not retry business or permission errors. Never retry a valid empty result.
- `"auto"` = optional, `"any"` = must call some tool, forced = must call a named tool. ⚠️ The newest models reject `"any"` and forced choice.
- Put team servers in `.mcp.json` with `${VAR}` secrets and personal servers in `~/.claude.json`. Use resources for catalogs, and community servers for standard systems.
- Use Grep for contents and Glob for file names. When Edit fails on a non-unique match, use Read + Write.

---

## Practice questions

Go to the Domain 2 question bank: **[mcq-bank/02-tool-design-mcp.md](../mcq-bank/02-tool-design-mcp.md)** (43 questions).
Older, easier warm-up questions: [domain-2-.../06-scenario-practice-questions.md](../domain-2-tool-design-mcp-integration/06-scenario-practice-questions.md).
