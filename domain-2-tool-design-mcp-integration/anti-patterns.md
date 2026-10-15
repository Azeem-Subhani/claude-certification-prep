# Domain 2 Anti-patterns: Tool Design & MCP Integration

**Scope:** wrong designs for tool descriptions, error responses, tool distribution, `tool_choice`, MCP configuration, and built-in tools (task statements 2.1 to 2.5).

---

## Anti-patterns

| # | Wrong approach | Why it fails | Do this instead | Topic |
|---|---|---|---|---|
| 1 | Write one-line tool descriptions. | Claude picks tools mainly from descriptions. Short or similar descriptions make it guess. | Describe what the tool does, inputs and formats, example queries, limits, and when to use another tool. | [01](01-tool-interface-design.md) |
| 2 | Add few-shot examples first when Claude picks the wrong tool. | Examples cost tokens and do not fix the cause, which is weak descriptions. | Improve the descriptions first. | [01](01-tool-interface-design.md) |
| 3 | Build a routing classifier, or merge tools, as the first fix. | Both take much more work than better descriptions. | Fix descriptions and names first. Use bigger changes only if that fails. | [01](01-tool-interface-design.md) |
| 4 | Keep two tools with near-identical names and descriptions. | Claude cannot tell them apart and misroutes requests. | Rename one so its purpose is clear, and rewrite both descriptions. | [01](01-tool-interface-design.md) |
| 5 | Keep one generic tool (for example, `analyze_document`) that does many jobs. | Its input and output are unclear, so Claude uses it in the wrong way. | Split it into narrow tools with clear input and output contracts. | [01](01-tool-interface-design.md) |
| 6 | Fix the descriptions and ignore the system prompt. | A keyword in the system prompt (for example, "for refunds, check the customer") can still pull Claude to the wrong tool. | Review the system prompt for keyword-sensitive instructions. | [01](01-tool-interface-design.md) |
| 7 | Return "Operation failed" for every error. | Claude cannot tell whether to retry, fix the input, or escalate. | Return a structured error: category, whether it can be retried, and a clear message. | [02](02-structured-error-responses.md) |
| 8 | Retry a business-rule error. | The same policy rule fails again every time. | Take another path or escalate. Explain the rule kindly to the customer. | [02](02-structured-error-responses.md) |
| 9 | Mark validation errors as retryable. | Sending the same bad input again fails again. | Mark it not retryable. Fix the input, then make a new call. | [02](02-structured-error-responses.md) |
| 10 | Read `isRetryable: false` as "give up". | Validation errors can still be fixed and sent again as a new call. | Fix the input. Escalate only business and permission errors. | [02](02-structured-error-responses.md) |
| 11 | Retry a query that worked but found nothing. | Zero results is a valid answer. Retrying wastes calls and gives the same result. | Return `isError: false` with zero results and accept it. | [02](02-structured-error-responses.md) |
| 12 | Return an empty list marked as success after a timeout. | The agent believes there is no data. This hides a failure as a success. | Return `isError: true` with a transient category, so the agent can retry or report it. | [02](02-structured-error-responses.md) |
| 13 | Give one agent 18 tools "for flexibility". | More choices make tool selection worse. Agents also misuse tools outside their role. | Give each agent the 4 to 5 tools its role needs. | [03](03-tool-distribution-and-tool-choice.md) |
| 14 | Give a subagent a generic `fetch_url` when it only needs documents. | It can fetch pages it should not, and it may wander off task. | Use a narrow tool such as `load_document` that accepts only valid document URLs. | [03](03-tool-distribution-and-tool-choice.md) |
| 15 | Send every simple check through the coordinator. | Each check adds round trips and delay. | Give the agent one small scoped tool (for example, `verify_fact`). Send only complex cases through the coordinator. | [03](03-tool-distribution-and-tool-choice.md) |
| 16 | Move tools to a second MCP server to "reduce choices". | Claude still sees all tools from all servers as one list. | Reduce or scope the tools per agent. | [03](03-tool-distribution-and-tool-choice.md) |
| 17 | Use `tool_choice: "auto"` when you must always get structured output. | With `"auto"`, Claude may answer in text instead. | On the exam, use `"any"` or a forced tool. ⚠️ The newest models reject these. Use `strict: true` with `"auto"`, or structured outputs. | [03](03-tool-distribution-and-tool-choice.md) |
| 18 | Put the team's MCP server in `~/.claude.json`. | That file is personal, so teammates never get the server. | Put team servers in the project's `.mcp.json`, committed to git. | [04](04-mcp-server-integration.md) |
| 19 | Commit secrets in `.mcp.json`. | The token leaks into git history for everyone. | Use `${VAR}` expansion, and keep the value in the environment. | [04](04-mcp-server-integration.md) |
| 20 | Build a custom MCP server for Jira, GitHub, or Slack. | It costs time to build and maintain something that already exists. | Use a community server. Build custom servers only for unique workflows. | [04](04-mcp-server-integration.md) |
| 21 | Let the agent make many calls just to learn what data exists. | It wastes calls and fills the context. | Expose a catalog (issue list, schema, document tree) as an MCP resource. | [04](04-mcp-server-integration.md) |
| 22 | Leave MCP tool descriptions short. | Claude prefers built-in tools like Grep over an MCP tool it does not understand. | Write rich MCP tool descriptions. | [04](04-mcp-server-integration.md) |
| 23 | Use Glob to find who calls a function. | Glob matches file names, not file contents. | Use Grep to search inside files. | [05](05-built-in-tools.md) |
| 24 | Use Grep to find files by name pattern. | Grep searches contents. It is the wrong tool for paths. | Use Glob, for example `**/*.test.tsx`. | [05](05-built-in-tools.md) |
| 25 | Read every file before you know what matters. | The context fills with code you do not need. | Grep for entry points, Read to follow imports, then Grep for new names. | [05](05-built-in-tools.md) |
| 26 | Use Read + Write for every change. | It rewrites whole files and risks overwriting other content. | Use Edit for targeted changes. Use Read + Write only as the fallback. | [05](05-built-in-tools.md) |

---

## Common exam traps

1. **"Add few-shot examples so Claude picks the right tool."** The cause is short or overlapping descriptions. Fix those first, because it is cheaper and fixes the root cause.
2. **"Build a routing classifier."** This is too much work for a first step. The exam prefers the lowest-effort fix that solves the cause.
3. **"An agent with 18 tools picks wrong ones, so improve the descriptions."** The agent has far more than 4 to 5 tools. So the toolset itself is the problem. Count the tools before you choose a fix.
4. **"Return `Operation failed` and let Claude figure it out."** Without a category, Claude cannot choose between retry, fix, and escalate.
5. **"Retry the empty result."** The query worked. An empty result is not a failure.
6. **"Return an empty list after a timeout so the pipeline continues."** This hides the failure. It is the worst error pattern.
7. **"`isRetryable` and `errorCategory` are part of the MCP spec."** Only `isError` is in the spec. The other two are an app convention.
8. **"Use `\"auto\"` to guarantee structured output."** `"auto"` lets Claude answer in text. On the exam, the answer is `"any"` or a forced tool.
9. **"Put the team's server in your personal config."** User scope is not shared. Team servers go in `.mcp.json`.
10. **"Build a custom server for Jira."** Community servers come before custom ones.
11. **"When Edit fails because the text is not unique, widen the match."** On the exam, the fallback is Read + Write. ⚠️ In real work, a longer unique match or `replace_all` also works (row C14 in the [README](README.md)).
12. **"Use Glob to find all callers."** Glob is for file names. Callers are found by searching contents with Grep.

> ⚠️ See [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) items R3 (`tool_choice` on the newest models), R6 (input validation errors in MCP), R7 (MCP scopes), and R11 (Edit fallback).
