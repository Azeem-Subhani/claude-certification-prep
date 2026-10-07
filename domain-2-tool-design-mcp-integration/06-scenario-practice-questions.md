# 06 · Scenario Practice Questions (Domain 2)

**Exam link:** All of Domain 2 (Task Statements 2.1–2.5).
**How to use:** Answer each question before you open the explanation. Each answer links to the file that teaches the concept. These are **original practice questions** written for study. They are not taken from the official exam or practice test.

**Where Domain 2 appears:** It is a primary domain in **Customer Support Resolution Agent**, **Multi-Agent Research System**, and **Developer Productivity with Claude**.

---

## How to approach Domain 2 questions

1. **Wrong tool chosen?** Look at the **descriptions and names** first, then the system prompt for keyword triggers.
2. **Bad recovery after a failure?** Look at the **error structure**: category, retryable flag, message.
3. **Agent doing work outside its role?** Look at **which tools it has**.
4. **Config or team-sharing problem?** Look at **scope** (`.mcp.json` vs `~/.claude.json`) and **secrets** (`${VAR}`).
5. **File/codebase task?** Match the **built-in tool** to the job: contents (Grep), paths (Glob), targeted change (Edit), fallback (Read + Write).

---

## Scenario A: Customer Support Resolution Agent

*A support agent built with the Claude Agent SDK uses MCP tools: `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`.*

### A1
Users say "Where's my order 55821?" and the agent often calls `get_customer` instead of `lookup_order`. Descriptions are "Gets customer info" and "Gets order info". What is the best first fix?

- A. Add a classifier model that routes each message to one tool.
- B. Rewrite both descriptions to state purpose, inputs (with format), outputs, and when to use each instead of the other.
- C. Merge both tools into one `get_info` tool.
- D. Add ten few-shot examples of order questions to the system prompt.

<details><summary>Answer</summary>

**B.** Descriptions are the primary selection mechanism. A adds complexity; C makes a vaguer tool; D can help but doesn't fix the root cause (overlapping, minimal descriptions). → [01](01-tool-interface-design.md)
</details>

### A2
The system prompt says: "If the user mentions 'refund', call `process_refund`." A customer writes: "Before I ask for a refund, can you tell me where my order is?" The agent calls `process_refund`. What is the root cause?

- A. `process_refund` has a poor description.
- B. A keyword-sensitive instruction in the system prompt overrides correct tool selection.
- C. The model is too small.
- D. `lookup_order` is missing from the tool list.

<details><summary>Answer</summary>

**B.** Keyword triggers create unintended tool associations. Replace them with intent-based guidance. → [01](01-tool-interface-design.md)
</details>

### A3
`process_refund` returns `"Operation failed"` for every problem. The agent retries refunds that are outside the 30-day policy five times, then tells customers to "try again later". What is the best fix?

- A. Limit retries to two.
- B. Return structured errors: `errorCategory: "business"`, `isRetryable: false`, and a customer-friendly explanation.
- C. Tell the agent in the prompt never to retry refunds.
- D. Escalate every failed refund to a human.

<details><summary>Answer</summary>

**B.** Structured metadata lets the agent stop retrying and explain the policy correctly. A and C are blunt; C also blocks retries of real transient errors. D over-escalates. → [02](02-structured-error-responses.md)
</details>

### A4
A customer searches for past orders by email. The orders database is down, and the tool returns `{"orders": []}`. The agent tells the customer they have never ordered. What is wrong?

- A. The agent should have asked for a phone number.
- B. The tool returns an access failure as a valid empty result; it should return `isError: true` with a transient, retryable error.
- C. The tool should return a fake sample order.
- D. The agent should always retry empty results.

<details><summary>Answer</summary>

**B.** Never confuse "query failed" with "query succeeded, no matches". D would waste retries on true empty results. → [02](02-structured-error-responses.md)
</details>

### A5
Your support agent has 18 tools, including email, calendar, CRM editing, and analytics. Tool selection accuracy is poor. What is the best change?

- A. Use a larger model.
- B. Reduce the agent to the 4–5 tools its role needs, and move other capabilities to separate agents or workflows.
- C. Sort the tools alphabetically.
- D. Add "choose tools carefully" to the system prompt.

<details><summary>Answer</summary>

**B.** Too many tools increase decision complexity; scoped access improves reliability and safety. → [03](03-tool-distribution-and-tool-choice.md)
</details>

### A6
Your extraction step must always produce output via the `record_case` tool, but sometimes the model replies with a plain sentence instead. Per the exam guide, which setting guarantees a tool call?

- A. `tool_choice: {"type": "auto"}`
- B. `tool_choice: {"type": "any"}`
- C. `tool_choice: {"type": "none"}`
- D. A higher `max_tokens`.

<details><summary>Answer</summary>

**B.** `"any"` requires a tool call. (Forcing `{"type": "tool", "name": "record_case"}` would also work.) Note: on the newest models, use `auto` + `strict: true` or structured outputs instead, because forced choice returns a 400. → [03](03-tool-distribution-and-tool-choice.md)
</details>

---

## Scenario B: Multi-Agent Research System

*A coordinator delegates to web search, document analysis, and synthesis subagents.*

### B1
The tools `analyze_content` and `analyze_document` have nearly identical descriptions. Uploaded PDFs are often sent to `analyze_content`. What is the best fix?

- A. Delete `analyze_document`.
- B. Rename `analyze_content` to `extract_web_results` and rewrite both descriptions with clear boundaries (web results vs uploaded files).
- C. Add a retry when the wrong tool is chosen.
- D. Tell users not to upload PDFs.

<details><summary>Answer</summary>

**B.** Rename + rewrite to remove functional overlap. → [01](01-tool-interface-design.md)
</details>

### B2
A generic `analyze_document(document, instructions)` tool produces inconsistent outputs, and downstream agents struggle to parse them. What design is best?

- A. Add "always return JSON" to the description.
- B. Split it into purpose-specific tools (`extract_data_points`, `summarize_content`, `verify_claim_against_source`) with defined input/output contracts.
- C. Let the coordinator rewrite the outputs.
- D. Increase `max_tokens`.

<details><summary>Answer</summary>

**B.** Purpose-specific tools give clear selection and predictable outputs. → [01](01-tool-interface-design.md)
</details>

### B3
The web search subagent hits a timeout on one query. What is the best way for this failure to reach the coordinator?

- A. Return `"search failed"`.
- B. Return an empty findings list marked as success.
- C. Retry locally first; if it still fails, return structured error context: failure type, retryable flag, what was attempted, and any partial results.
- D. Stop the whole pipeline.

<details><summary>Answer</summary>

**C.** Recover locally from transient errors, then escalate with enough context for the coordinator to decide (re-delegate, continue with partial results, or note a coverage gap). → [02](02-structured-error-responses.md)
</details>

### B4
The synthesis agent has `web_search` and sometimes runs its own unplanned searches, adding unverified sources to the report. What should you do?

- A. Keep `web_search` but tell it to use reputable sources.
- B. Remove `web_search` from the synthesis agent's tools.
- C. Give the synthesis agent every tool so it can be self-sufficient.
- D. Turn off the web search subagent.

<details><summary>Answer</summary>

**B.** Agents misuse tools outside their specialization. Scope tools to the role. → [03](03-tool-distribution-and-tool-choice.md)
</details>

### B5
The synthesis agent often needs quick checks of single claims. Each check currently goes synthesizer → coordinator → search agent → coordinator → synthesizer, which is slow. What is the best design?

- A. Give the synthesis agent full `web_search`.
- B. Give the synthesis agent a narrow `verify_fact(claim, source_url)` tool; keep complex research routed through the coordinator.
- C. Merge the synthesis and search agents.
- D. Skip fact verification.

<details><summary>Answer</summary>

**B.** A scoped cross-role tool handles the frequent, simple need; complex cases keep the coordinator's control. → [03](03-tool-distribution-and-tool-choice.md)
</details>

### B6
Agents can call `fetch_url` with any URL. In testing, one agent fetched an internal admin page. What is the best change?

- A. Add "don't fetch internal URLs" to the system prompt.
- B. Replace `fetch_url` with a constrained `load_document` tool that validates URLs (https only, allowed document types, no internal addresses).
- C. Log all fetches for later review.
- D. Remove web access from all agents.

<details><summary>Answer</summary>

**B.** Constrained tools enforce the rule deterministically and give predictable outputs. A is probabilistic; C detects but doesn't prevent; D removes needed capability. → [03](03-tool-distribution-and-tool-choice.md)
</details>

### B7
Every incoming document must have `extract_metadata` run first; then the agent should freely choose among enrichment tools. Per the exam guide, how should you configure `tool_choice`?

- A. `"auto"` on every turn.
- B. Forced `{"type": "tool", "name": "extract_metadata"}` on the first turn, then `"auto"` in follow-up turns.
- C. `"any"` on every turn.
- D. `"none"` on the first turn.

<details><summary>Answer</summary>

**B.** Force the required first step, then let the model choose in later turns. → [03](03-tool-distribution-and-tool-choice.md)
</details>

---

## Scenario C: Developer Productivity with Claude

*Engineers use Claude Code and Agent SDK tools to explore codebases and automate work.*

### C1
A team wants everyone to have the same Jira and GitHub MCP servers when they clone the repo, without sharing anyone's tokens. What is the best setup?

- A. Each developer adds the servers to their own `~/.claude.json`.
- B. Commit a project `.mcp.json` that uses `${JIRA_TOKEN}` and `${GITHUB_TOKEN}`; each developer sets the variables locally.
- C. Commit `.mcp.json` with a shared service-account token.
- D. Email the config file to new hires.

<details><summary>Answer</summary>

**B.** Project scope for shared tooling, environment variable expansion for secrets. C commits a secret. A and D don't scale or stay in sync. → [04](04-mcp-server-integration.md)
</details>

### C2
A developer wants to try an experimental MCP server without affecting teammates. Where should it go?

- A. The project `.mcp.json`.
- B. The user-level `~/.claude.json`.
- C. `CLAUDE.md`.
- D. A new branch's `.mcp.json`.

<details><summary>Answer</summary>

**B.** Personal or experimental servers belong in user scope. → [04](04-mcp-server-integration.md)
</details>

### C3
Your team built an MCP `find_references` tool backed by a semantic code index, but the agent keeps using Grep and misses aliased imports. What is the best fix?

- A. Remove Grep from the agent.
- B. Improve the MCP tool's description to explain its capabilities, outputs, and when it beats text search.
- C. Rename it `grep2`.
- D. Tell developers to stop using aliases.

<details><summary>Answer</summary>

**B.** Detailed descriptions help the model prefer the more capable MCP tool. A removes a useful tool for other tasks. → [04](04-mcp-server-integration.md)
</details>

### C4
Before writing any query, the agent spends 8 tool calls listing and describing database tables. What is the best improvement?

- A. Increase the max turns.
- B. Expose the database schema as an MCP resource so the agent sees what exists without exploring.
- C. Add a `list_everything` tool.
- D. Give the agent direct database admin access.

<details><summary>Answer</summary>

**B.** Resources expose content catalogs and cut exploratory calls. → [04](04-mcp-server-integration.md)
</details>

### C5
Your team needs Claude to work with Jira. A developer proposes building a custom MCP server. What is the best recommendation?

- A. Build a custom server for full control.
- B. Use a maintained existing Jira MCP server (after checking source, maintenance, and permissions); build custom servers only for team-specific workflows.
- C. Avoid MCP and paste Jira tickets into chat.
- D. Write a Bash script that scrapes the Jira website.

<details><summary>Answer</summary>

**B.** Prefer community/vendor servers for standard integrations. → [04](04-mcp-server-integration.md)
</details>

### C6
You need to find every place the error message "Payment gateway unavailable" is produced. Which tool?

- A. Glob
- B. Grep
- C. Read every file
- D. Write

<details><summary>Answer</summary>

**B.** Searching file **contents**. → [05](05-built-in-tools.md)
</details>

### C7
You need all Storybook files, named like `Button.stories.tsx`. Which tool and pattern?

- A. Grep `stories`
- B. Glob `**/*.stories.tsx`
- C. Bash `ls`
- D. Read `src/`

<details><summary>Answer</summary>

**B.** Matching file **paths/names**. → [05](05-built-in-tools.md)
</details>

### C8
Edit fails because `retries: 3` appears in four places in a YAML file, and you need to change only the one under `payments:`. What is the most reliable approach?

- A. Use Write with only the changed line.
- B. Include more surrounding context (e.g. the `payments:` section lines) to make the match unique, or Read the full file and Write the modified version.
- C. Use `sed` in Bash to replace all occurrences.
- D. Delete the file and recreate it from memory.

<details><summary>Answer</summary>

**B.** Make the anchor unique or use the Read + Write fallback. A would erase the rest of the file; C changes all four. → [05](05-built-in-tools.md)
</details>

### C9
You must find all callers of `charge_card`, but `payments/__init__.py` re-exports it as `charge` and `legacy_charge`. Grep for `charge_card` finds only 2 callers. What should you do?

- A. Conclude there are only 2 callers.
- B. Read the wrapper module to list all exported names, then Grep for each name.
- C. Glob for `*charge*`.
- D. Read every Python file in the repo.

<details><summary>Answer</summary>

**B.** Identify all exported names first, then search each. → [05](05-built-in-tools.md)
</details>

### C10
An agent asked "How does the app send password reset emails?" starts by reading all 300 files in the repo. What is the better strategy?

- A. Read files in alphabetical order.
- B. Grep for entry points (e.g. "reset_password"), Read those files, then follow imports to only the relevant modules.
- C. Glob for all `.py` files and Read them.
- D. Ask the user to find the file.

<details><summary>Answer</summary>

**B.** Build understanding incrementally instead of reading everything upfront. → [05](05-built-in-tools.md)
</details>

---

## Mixed concept checks

### M1
In MCP, how should a tool report that an upstream API call failed?
- A. A JSON-RPC protocol error.
- B. A normal result with `isError: true` and a useful message.
- C. An empty result.
- D. Crash the server.

<details><summary>Answer</summary>**B.** Tool execution errors are returned as results with `isError: true`, so the model can see and react. → [02](02-structured-error-responses.md)</details>

### M2
Which error should normally be marked `isRetryable: true`?
- A. "Account lacks permission to issue refunds."
- B. "Order ID format invalid."
- C. "Service temporarily unavailable (503)."
- D. "Refund exceeds policy limit."

<details><summary>Answer</summary>**C.** Transient. → [02](02-structured-error-responses.md)</details>

---

## Score yourself

| Score | Next step |
|---|---|
| 22–25 | Ready for Domain 2. Revisit any you guessed. |
| 17–21 | Re-read the files linked from your wrong answers. |
| < 17 | Re-read files 00–05 in order, then try again in a few days. |

---

## Key Takeaways

- Wrong-tool problems → **descriptions, names, overlap, keyword triggers**.
- Bad-recovery problems → **structured errors** (category, retryable, message) and **failed vs empty**.
- Misuse and unreliability → **scoped, constrained tools**, about 4–5 per agent, with narrow cross-role tools.
- Config problems → **`.mcp.json` for team, `~/.claude.json` for personal, `${VAR}` for secrets, resources for catalogs**.
- File work → **Grep (contents), Glob (paths), Edit (unique change), Read + Write (fallback), incremental exploration**.
