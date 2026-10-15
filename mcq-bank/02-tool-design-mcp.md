# Domain 2 MCQ Bank: Tool Design & MCP Integration

> 43 questions: 11 Easy, 17 Medium, 15 Hard. [Study notes](../domains/02-tool-design-mcp.md) · [Bank index](README.md)

## Questions

### Q1 · Task 2.1 · Medium

A logistics agent uses `export_shipments` to pull shipment records for reports. The backend returns at most 90 days of data per call and silently drops the rest. Users who ask for yearly totals get numbers that are far too low. The tool description says only "Exports shipment records."

What is the BEST fix?

- A. Add a check after each call that flags totals lower than the same period last year.
- B. Increase max_tokens so the full year of records fits in one tool response.
- C. State the 90-day limit in the description and explain how to split longer ranges.
- D. Add a system prompt rule that tells Claude to refuse reports longer than 90 days.

### Q2 · Task 2.4 · Easy

A developer added a markdown-notes MCP server for their own use to the repo's .mcp.json and pushed it. Now every teammate sees a prompt to approve a server they never asked for. The developer still wants the server for themselves.

What is the BEST change?

- A. Remove it from .mcp.json and add it in user scope, which Claude Code stores in ~/.claude.json.
- B. Keep it in .mcp.json but add "optional": true so teammates can skip the approval prompt.
- C. Move the server entry into the project CLAUDE.md so it loads only when the developer asks for it.
- D. Move it to a second shared .mcp.json in a subfolder so it only loads for files in that folder.

### Q3 · Task 2.2 · Hard

A billing support agent's loop retries any tool result with isError true up to three times, with backoff. Logs show 40% of retries follow "credit limit exceeded" results, and every one fails again. These retries add about six seconds to each affected conversation.

Which change BEST fixes the root problem?

- A. Lower the retry count from three to one for all failed tool calls.
- B. Use longer exponential backoff, starting at two seconds, so the billing service has time to recover.
- C. Let Claude read the error text and decide by itself whether to retry.
- D. Add errorCategory and isRetryable to errors, and retry only if isRetryable is true.

### Q4 · Task 2.5 · Medium

Claude Code must change a timeout constant from 30 to 45 in a 3,000-line config module. A teammate suggests reading the whole file and writing it back, to be safe.

Which approach is BEST?

- A. Read the file, then use Write to save the full file with the new value.
- B. Use Edit with an old_string that matches the constant's line exactly once.
- C. Use Grep to find the line, then use Write to replace the whole file.
- D. Use Bash to append a new constant line that overrides the old value.

### Q5 · Task 2.3 · Easy

An e-commerce help assistant answers general questions in text. It can also call `get_order_status` when a customer asks about a specific order. The model in use supports all tool_choice options.

Which tool_choice setting fits this design BEST?

- A. "auto", so Claude decides whether to answer in text or call the tool.
- B. "any", so Claude always calls a tool before it gives an answer.
- C. A forced choice of `get_order_status` on every request.
- D. "any" on the first request, then "auto" on all later requests.

### Q6 · Task 2.1 · Hard

An insurance agent has five tools, including `get_policy` and `search_policies`. Logs show two problems. Claude often calls `search_policies` when the user gives an exact policy number. When it does call `get_policy`, it often sends "POL-48213", but the tool expects digits only. Both descriptions are one sentence long.

What is the MOST effective first fix?

- A. Add a regular expression to the input_schema so `get_policy` rejects any ID with a prefix.
- B. Rename `search_policies` to `find_policies_by_holder_name` so its purpose is clear.
- C. Rewrite both descriptions with input formats, examples, and when to use the other tool.
- D. Add five few-shot examples of correct tool calls to the system prompt.

### Q7 · Task 2.2 · Easy

An MCP tool returns a result after the shipping API fails. The result has `isError: true`. Its content also includes errorCategory "transient" and isRetryable true.

Which part of this result is defined by the MCP specification?

- A. The errorCategory field and its "transient" value.
- B. The isError flag on the tool result.
- C. The isRetryable field and its true value.
- D. The pairing of errorCategory with isRetryable.

### Q8 · Task 2.4 · Medium

A team shares an inventory MCP server through the project's .mcp.json. A code review finds the server's API key written directly in the headers field. Each developer has a personal key.

What is the BEST fix?

- A. Add .mcp.json to .gitignore so the key never reaches the repository.
- B. Move the server to each developer's ~/.claude.json and delete its entry from .mcp.json.
- C. Store the key Base64-encoded in .mcp.json so it is not in plain text.
- D. Replace the key with ${INVENTORY_API_KEY} and have each developer set that variable.

### Q9 · Task 2.3 · Hard

A support pipeline must classify each ticket before it takes any other action. The team set tool_choice to force `classify_ticket` on every request in the agent loop. Now the agent calls `classify_ticket` again and again and never replies. The model in use supports all tool_choice options.

What should the architect change?

- A. Force `classify_ticket` only on the first request, then use "auto" for later requests.
- B. Switch every request to "any", so Claude can choose among all of its tools.
- C. Use "auto" on every request, with a system prompt line that says to classify first.
- D. Keep the forced choice on every request and cap the loop at five iterations.

### Q10 · Task 2.5 · Easy

A developer asks Claude Code to list every Dockerfile in a monorepo. Some of them sit in nested service folders.

Which built-in tool should Claude use?

- A. Grep with the pattern "FROM " across the repository.
- B. Read on the repository's root folder.
- C. Glob with the pattern "**/Dockerfile".
- D. Grep for the word "Dockerfile" in all files.

### Q11 · Task 2.1 · Easy

A team is building a contract review agent with six tools. A new engineer asks what Claude relies on MOST when it chooses between tools.

What is the BEST answer?

- A. The order in which the tools appear in the tools list.
- B. The description written in each tool definition.
- C. The property names inside each tool's input_schema.
- D. The tool names alone, as long as each one is unique.

### Q12 · Task 2.2 · Medium

A support agent calls `issue_refund` for $900. Company policy caps agent refunds at $500. The tool now returns isError true with the text "Refund failed." The agent retries twice, then tells the customer to try again later.

What should the tool return instead?

- A. isError false with the message "Refund pending review," so the agent stops retrying.
- B. errorCategory "transient" with isRetryable true, so the agent waits and tries again.
- C. A JSON-RPC (remote procedure call) protocol error, so the host stops the call early.
- D. errorCategory "business" with isRetryable false, plus a reason the agent can share.

### Q13 · Task 2.4 · Hard

A platform team wants Claude Code to read GitHub pull requests and issues. It also wants Claude to start the company's in-house release approval flow, which runs on an internal ticket system. A senior engineer proposes one custom MCP server for both, to keep a single codebase. The budget for new server work is small.

What is the BEST plan?

- A. Build one custom MCP server that covers GitHub and the approval flow, as proposed.
- B. Use a community GitHub server, and let Claude call the internal API with ad hoc Bash commands.
- C. Use a community GitHub server, and build a small custom server only for the approval flow.
- D. Search for community servers for both needs, and wait on approvals until one exists.

### Q14 · Task 2.3 · Medium

A healthcare billing extraction step must always call `record_claim_fields`. The agent also has `lookup_payer` and `lookup_procedure_code` for other steps. With tool_choice "any", Claude sometimes calls a lookup tool at this step. The model in use supports all tool_choice options.

Which setting BEST fixes this step?

- A. tool_choice {"type": "tool", "name": "record_claim_fields"} for this request.
- B. tool_choice "auto", with a description that says this step needs record_claim_fields.
- C. tool_choice "any", with the two lookup tools listed after record_claim_fields.
- D. tool_choice "any", with a system prompt line that asks for record_claim_fields.

### Q15 · Task 2.1 · Medium

A research team drafts four descriptions for a tool named `verify_claim_against_source`.

Which description is BEST?

- A. "Verifies claims quickly and accurately against trusted sources. Use it often, for any research task, to get reliable results."
- B. "Checks one claim against one source. Input: claim text and source_id. Returns supported, contradicted, or not_found. To find sources, use search_sources."
- C. "Calls the internal FactCheck v2 service with the claim text and a source_id. Returns that service's raw response object, unchanged."
- D. "Use this tool whenever a claim, fact, statement, source, or citation appears in the task. It works for any topic or document."

### Q16 · Task 2.5 · Medium

Claude Code must change one `if (!user) return null;` line in a service file. Edit fails because that exact text appears six times in the file. Only one of the six lines should change.

Under the exam's guidance, what should Claude do next?

- A. Run Bash with sed to replace that line's text across the whole file.
- B. Use Grep to confirm the line exists, then retry Edit with the same old_string.
- C. Use Glob to find the file path again, then retry the same Edit call.
- D. Read the full file, then use Write to save the corrected content.

### Q17 · Task 2.2 · Hard

A travel booking agent calls `search_fares` with the date "next Friday". The tool returns isError true, errorCategory "validation", and isRetryable false, with no other text. The agent then tells users it cannot search fares. A teammate suggests setting isRetryable true for all validation errors.

What is the BEST fix?

- A. Set isRetryable true for validation errors, as the teammate suggests.
- B. Return the failure as a JSON-RPC protocol error so the client fixes the date format.
- C. Keep isRetryable false and add a message that states the expected date format.
- D. Remove the date check so the tool accepts any text and guesses the date.

### Q18 · Task 2.3 · Hard

A data-operations agent has 12 tools. Eight are format converters, such as `csv_to_json`, `json_to_xml`, and `xml_to_csv`. The other four query, validate, export, and log data. The team rewrote all twelve descriptions last month, but Claude still picks the wrong converter often.

What is the BEST next step?

- A. Merge the eight converters into one `convert_data` tool with source and target format fields.
- B. Rewrite the eight converter descriptions again, with more example queries for each.
- C. Move the eight converters to a second MCP server so the agent sees fewer choices at once.
- D. Add a routing classifier that picks the right converter for each request before Claude sees it.

### Q19 · Task 2.1 · Easy

A freight agent has `get_rates` for carrier shipping quotes and `fetch_rates` for currency exchange rates. Claude often mixes them up.

What is the BEST low-effort fix?

- A. Merge both into one `get_rates` tool with a free-text "kind" field for the rate type.
- B. Remove `fetch_rates` and list today's exchange rates in the system prompt.
- C. Add few-shot examples of each tool's correct use to the system prompt.
- D. Rename them by purpose, such as `get_carrier_quotes` and `get_exchange_rates`.

### Q20 · Task 2.4 · Medium

A shared .mcp.json points a metrics MCP server at a URL. Most developers use the staging endpoint. A few developers need to point at their own local copy instead.

Which url value BEST supports both groups?

- A. "${METRICS_URL}", with every developer required to set the variable.
- B. "${METRICS_URL:-https://metrics-staging.example.internal}"
- C. The staging URL hard-coded, with local users editing .mcp.json each session.
- D. Two server entries in .mcp.json, one for staging and one for local.

### Q21 · Task 2.2 · Easy

A `search_orders` tool queries the order database with no problems. No orders match the customer's email address.

How should the tool report this?

- A. isError true with errorCategory "validation", so the agent asks for a new email.
- B. isError true with isRetryable true, so the agent runs the search again.
- C. isError false with an empty result list and a count of zero.
- D. A JSON-RPC protocol error that says the record was not found.

### Q22 · Task 2.5 · Hard

A team will remove `legacyParseAmount`. Some modules import it directly. `money/index.ts` re-exports it as `parseAmountCompat`, which other modules use. The team needs every caller and the test files for those callers. The context budget is tight.

Which sequence is BEST?

- A. Grep legacyParseAmount, Read money/index.ts, Grep parseAmountCompat, then Glob for test files.
- B. Glob for "**/*ParseAmount*" to find caller files, then Read each match and its test file.
- C. Grep for legacyParseAmount only, then Glob for "**/*.test.ts" and Read every test file found.
- D. Read every file under src/ and tests/ so no caller is missed, then list the callers by module and file.

### Q23 · Task 2.1 · Hard

A legal team's agent has one tool, `review_contract`. It takes a free-text "instructions" field and can extract clauses, score risk, or write a summary. Downstream code parses the output, but the format changes with the instructions. Parsing fails on about 15% of contracts. Someone suggests a longer description that lists all three jobs.

What is the BEST fix?

- A. Expand the description to list the three jobs and the output format of each.
- B. Split it into `extract_clauses`, `score_clause_risk`, and `summarize_contract`, each with a defined output.
- C. Keep one tool and add an output schema with optional fields for all three jobs.
- D. Add a retry that calls `review_contract` again, with stricter instructions, whenever parsing fails.

### Q24 · Task 2.3 · Medium

A document-summary subagent was given `send_email` "just in case." It has started to email draft summaries to customers on its own. Only the notification agent should send email.

What is the BEST fix?

- A. Add a line to the subagent's prompt that says it must never send email.
- B. Rename `send_email` to `send_customer_email` so its purpose is clearer.
- C. Keep the tool, but require the coordinator to approve each email before it is sent.
- D. Remove `send_email` from the subagent and keep it on the notification agent.

### Q25 · Task 2.2 · Medium

A medical billing extraction agent calls `fetch_remittance` on a payer portal. The portal returns "403 access denied" for one payer. The service account does not have that payer's role.

How should the tool classify this error?

- A. errorCategory "permission" with isRetryable false, so the agent escalates the access issue.
- B. errorCategory "transient" with isRetryable true, so the agent retries with backoff.
- C. errorCategory "validation" with isRetryable false, so the agent changes the payer ID and retries.
- D. isError false with an empty remittance list, so the batch run continues.

### Q26 · Task 2.4 · Easy

A vendor hosts an MCP server at a remote web URL. A team wants to add it to the project's .mcp.json.

Which configuration shape is correct?

- A. "command": "npx", with the URL passed in "args".
- B. "type": "stdio", with the URL in an "env" variable.
- C. "type": "http", with the server address in "url".
- D. "command" set to the URL, with no "args" field.

### Q27 · Task 2.1 · Medium

An e-commerce agent has `cancel_order` and `refund_order`. Both descriptions say "Handles order problems." Claude sometimes refunds orders that have not shipped yet and could simply be cancelled.

What is the BEST fix?

- A. Add a system prompt rule: "If the customer mentions money, use refund_order."
- B. Rewrite both descriptions to state purpose, inputs, and when to use the other tool.
- C. Merge both tools into one `fix_order` tool with an action parameter.
- D. Rename `refund_order` to `refund_shipped_order` and keep the current one-line descriptions.

### Q28 · Task 2.3 · Hard

A finance reporting subagent has a `run_sql` tool that accepts any SQL (database query) statement. It only needs monthly sales totals by region. It sometimes runs slow joins across unrelated tables, and once it read payroll data. The DBA (database administrator) already made the database user read-only.

What is the BEST change?

- A. Add a prompt rule that tells the subagent to query only the sales tables.
- B. Keep run_sql and add a PostToolUse hook that strips payroll columns from results.
- C. Keep run_sql but stop any query that runs longer than ten seconds.
- D. Replace run_sql with `get_monthly_sales(region, month)`, which runs one fixed query.

### Q29 · Task 2.5 · Medium

Claude Code needs to find where the app throws `TimeoutError`. It ran Glob with "**/*TimeoutError*". It found only `errors/TimeoutError.ts`, which defines the class.

What should Claude do next?

- A. Run Glob again with a wider pattern, such as "**/*Timeout*".
- B. Read every file in the errors/ folder to trace where the class is used.
- C. Run Grep for "new TimeoutError" across the codebase.
- D. Read errors/TimeoutError.ts and rely on its comments to list the callers.

### Q30 · Task 2.2 · Hard

A warehouse agent's tools report failures in different ways. One returns "ERR 52", and one returns an empty list. A third returns "Something went wrong." Claude often retries policy blocks and gives up on short outages. The team wants one fix that works for every tool.

What is the BEST approach?

- A. Keep the outputs as they are, and add a table of each tool's error codes to the system prompt.
- B. Use one error shape: isError true, errorCategory, isRetryable, and a human-readable message.
- C. Log full errors on the server, and return "Operation failed" to Claude for every failure.
- D. Have every tool retry three times in its own code before it returns any error.

### Q31 · Task 2.1 · Easy

An agent picks the wrong tool because its tool descriptions are one line each. A teammate wants to add many few-shot examples to the system prompt first.

Why is this a weak FIRST step?

- A. It costs tokens and does not fix the short descriptions that cause the errors.
- B. Few-shot examples are not allowed in a system prompt that is used with tools.
- C. Few-shot examples change only the output format of answers, never the choice of tool.
- D. It forces Claude to call a tool on every turn, like tool_choice "any".

### Q32 · Task 2.4 · Hard

A developer added a personal, experimental "scratchpad" MCP server to the project's .mcp.json. Now teammates see its tools, and Claude sometimes calls them during team tasks. Sessions also show errors because teammates lack the SCRATCH_TOKEN variable. The shared Jira and GitHub servers work fine.

What is the BEST fix?

- A. Change the token to ${SCRATCH_TOKEN:-none} so teammates stop seeing errors.
- B. Add a README note that tells teammates to ignore the scratchpad tools.
- C. Move all servers, including Jira and GitHub, into each developer's user scope in ~/.claude.json.
- D. Move only the scratchpad server to the developer's user scope in ~/.claude.json.

### Q33 · Task 2.3 · Easy

A multi-agent research system has five subagents and 20 tools in total. The lead engineer plans to give every subagent all 20 tools, for flexibility.

What is the BEST practice?

- A. Give every subagent all 20 tools, and write longer descriptions for each tool.
- B. Give every subagent all 20 tools, and set tool_choice "auto" for each one.
- C. Give each subagent only the 4 to 5 tools that its role needs.
- D. Give every subagent the same five tools that are used most across the system.

### Q34 · Task 2.2 · Easy

A `get_tracking` tool calls a carrier API. The call times out after 10 seconds because the carrier service is briefly down.

Along with isError true, what should the tool result include?

- A. errorCategory "business" with isRetryable false.
- B. errorCategory "transient" with isRetryable true.
- C. errorCategory "validation" with isRetryable true.
- D. errorCategory "permission" with isRetryable false.

### Q35 · Task 2.1 · Medium

An insurance agent calls `estimate_repair_cost`, which returns amounts in US cents. The description does not say this. Claude often reports "$45,000" for a $450 repair. It also makes extra `get_claim` calls to guess the currency.

What is the BEST fix?

- A. State in the description that amounts are returned as whole US cents.
- B. Add a system prompt rule that every tool returns money values in dollars.
- C. Add few-shot examples that show Claude dividing each amount by 100.
- D. Remove `get_claim` from the agent so it cannot make the extra calls.

### Q36 · Task 2.5 · Hard

A coding agent makes every change with Read and then Write, even one-line fixes in 2,000-line files. Token use is high. Twice, a formatter changed a file between the Read and the Write, and those changes were lost. The team is choosing a new rule.

Which rule is BEST?

- A. Always Read the file again just before each Write, then write the full file.
- B. Use Bash with sed for all small changes, since sed does not need a Read first.
- C. Use Edit for targeted changes, and keep Read plus Write as the fallback.
- D. Split large files into smaller ones so Read plus Write uses fewer tokens.

### Q37 · Task 2.4 · Medium

A support agent connects to a knowledge-base MCP server. Before it answers, it often calls `list_folders` and `list_articles` many times to learn which articles exist.

What is the BEST way to cut these exploratory calls?

- A. Add a `list_all_articles` tool that returns every article title and folder in one call.
- B. Expose MCP prompts that teach the agent how to browse the folders.
- C. Paste the full article index into the system prompt and update it by hand.
- D. Expose the article catalog as an MCP resource that the application provides.

### Q38 · Task 2.3 · Hard

A synthesis agent has a `verify_fact` tool, and 85% of its checks are simple. The other 15% need research across several sources and databases. The team wants low latency for common checks and no spread of tools across roles. One engineer proposes giving the synthesis agent the full search toolkit.

What is the BEST design?

- A. Give the synthesis agent the full search toolkit so it handles every check itself.
- B. Keep verify_fact for simple checks, and route the complex 15% through the coordinator.
- C. Route every check, simple or complex, through the coordinator to the search agent.
- D. Expand verify_fact so it also runs multi-source research when a check is complex.

### Q39 · Task 2.2 · Medium

A telecom support agent tries to waive a late fee. The tool returns isError true, errorCategory "business", isRetryable false, and code "POL-17". The agent tells the customer, "Error POL-17 occurred." Customers find this confusing.

What should the team add to the tool result?

- A. A customer-friendly explanation of the policy that the agent can pass on.
- B. A change to isRetryable true, so the agent tries the waiver again later.
- C. A full stack trace, so the agent can explain the technical cause.
- D. A second internal code that maps POL-17 to the policy's short name.

### Q40 · Task 2.1 · Hard

A property-management agent has five tools. After the team rewrote vague descriptions, wrong tool picks fell from 20% to 8%. Most remaining errors happen when users say "check," as in "check my lease end date." Claude then calls `get_maintenance_status` instead of `get_lease_details`. Both descriptions now look clear and distinct.

What should the architect do FIRST?

- A. Merge the two tools into one `property_info` tool with a required category parameter.
- B. Add a routing classifier that maps user intents to tools before Claude runs.
- C. Review the system prompt for instructions that tie the word "check" to a tool.
- D. Add few-shot examples of lease questions that use the word "check."

### Q41 · Task 2.4 · Hard

A data team added a warehouse MCP server with a `query_warehouse` tool. Its description says only "Query tool." Claude keeps using Read on old CSV (comma-separated values) exports in the repo instead. One teammate proposes blocking Read for the whole project. Another wants a system prompt line that says "always use MCP."

What is the BEST first fix?

- A. Block the Read tool for the project so Claude must use query_warehouse.
- B. Expand the description with the data, inputs, outputs, and when to prefer it.
- C. Add "always use MCP tools first" to the system prompt so every session prefers them.
- D. Move the CSV exports out of the repository so Read cannot find them.

### Q42 · Task 2.3 · Medium

A single agent has 15 tools: five for web search, five for data analysis, and five for report export. It often calls export tools in the middle of research.

What is the BEST restructure?

- A. Split the work into role-based subagents, each with only its own five tools.
- B. Move the export tools to a separate MCP server connected to the same agent.
- C. Keep one agent and set tool_choice "any" so it must pick a tool each turn.
- D. Keep one agent and add "do not export until research is done" to its system prompt.

### Q43 · Task 2.5 · Medium

A new developer asks Claude Code how a large, unfamiliar repo handles payment retries. Nobody knows which files are involved.

Which approach is BEST?

- A. Read every file under src/ first, so Claude has the full picture.
- B. Glob for "**/*retry*" and treat the matching file names as the complete set of retry code.
- C. Read the README, then answer from its architecture section alone.
- D. Grep the code for terms like "retry", Read the key matches, and follow imports.

## Answer key

### Q1 · C

**Why C is correct:** A good description states the tool's limits and how to work within them. Once Claude knows about the 90-day cap, it can split a yearly request into several calls. This fixes the cause, not just the wrong totals.

**Closest distractor (D):** Refusing long reports blocks a valid user need, when Claude could meet it by splitting the range.

### Q2 · A

**Why A is correct:** The project .mcp.json is shared through git, so everyone gets the server. A personal server belongs in user scope, which lives in ~/.claude.json and is not shared. The developer keeps the server, and teammates stop seeing it.

**Closest distractor (B):** There is no "optional" flag for this. The server would still be in the shared file and reach every teammate.

### Q3 · D

**Why D is correct:** "Credit limit exceeded" is a business error, and the same rule fails every time. Structured errorCategory and isRetryable fields let the loop retry only transient failures. This removes wasted retries without hurting real recovery.

**Closest distractor (A):** Fewer retries cut the waste but still retry business errors, and they weaken recovery from real outages.

### Q4 · B

**Why B is correct:** Edit is the tool for targeted changes, using an old_string that matches once. It sends far less content than rewriting a 3,000-line file. Read plus Write is the fallback, not the default.

**Closest distractor (A):** Read plus Write works, but it costs more tokens and risks overwriting other changes in a large file.

### Q5 · A

**Why A is correct:** "auto" lets Claude choose between a text answer and a tool call. This assistant needs both behaviors. Forcing a tool would break simple text answers.

**Closest distractor (D):** Using "any" on the first request makes Claude call a tool even for a general question.

### Q6 · C

**Why C is correct:** Both symptoms come from thin descriptions. A full description states the input format and when to use the other tool. One rewrite fixes both the wrong choice and the wrong ID format.

**Closest distractor (B):** A clearer name may help selection, but it does nothing about the ID format problem.

### Q7 · B

**Why B is correct:** The MCP specification defines isError on tool results for failures during execution. errorCategory and isRetryable are an application convention placed inside the result content. They are useful, but they are not spec fields.

**Closest distractor (C):** isRetryable is a common app convention, but the MCP specification does not define it.

### Q8 · D

**Why D is correct:** .mcp.json supports ${VAR} expansion, so the shared file holds no secret. Each developer sets their own key in the environment. The team still shares one server config. The exposed key should also be rotated.

**Closest distractor (B):** Moving the server to user scope hides the key, but the team loses its shared, version-controlled config.

### Q9 · A

**Why A is correct:** Forced tool choice guarantees the first step runs. After that, "auto" lets Claude call other tools or reply in text. Forcing on every request means Claude can never finish with a text reply.

**Closest distractor (C):** A prompt instruction does not guarantee the classify step runs first, which the pipeline requires.

### Q10 · C

**Why C is correct:** Glob finds files by name or path pattern. "**/Dockerfile" matches the file at any folder depth. It does not need to open file contents.

**Closest distractor (A):** Grep searches contents, so "FROM " can also match scripts, docs, and other files that are not Dockerfiles.

### Q11 · B

**Why B is correct:** The tool description is the main signal Claude uses to choose a tool. A clear description says what the tool does and when to use it. Names and schemas help, but less.

**Closest distractor (D):** Names give hints, but similar names still cause misrouting without clear descriptions.

### Q12 · D

**Why D is correct:** A policy cap is a business error, and retrying will fail every time. isRetryable false stops wasted retries. A clear reason lets the agent explain the limit or escalate.

**Closest distractor (A):** Marking the call as a success hides the failure, and the agent tells the customer something false.

### Q13 · C

**Why C is correct:** A standard integration like GitHub should use an existing community server first. Custom work is for team-specific workflows, like the internal approval flow. This also keeps new server work small.

**Closest distractor (B):** Ad hoc Bash calls give Claude no described, scoped tool for the approval flow, which is less reliable and less safe.

### Q14 · A

**Why A is correct:** A forced tool choice names the exact tool, so Claude must call record_claim_fields. "any" only guarantees some tool call. Here the wrong tool is the problem.

**Closest distractor (D):** A prompt line makes the right tool more likely but does not guarantee it under "any".

### Q15 · B

**Why B is correct:** It states the purpose, inputs, outputs, and when to use a different tool. These parts help Claude choose and call the tool correctly. The other drafts miss most of them.

**Closest distractor (D):** A keyword list is too broad and pulls Claude toward this tool for unrelated tasks.

### Q16 · D

**Why D is correct:** The exam's fallback when Edit cannot find a unique match is Read then Write. Claude reads the whole file and writes it back with the one line changed. This is reliable when the anchor text repeats.

**Closest distractor (B):** Retrying Edit with the same old_string fails again, because the text still appears six times.

### Q17 · C

**Why C is correct:** Here isRetryable false means the agent should not repeat the same call. It does not mean the agent should give up. A message with the expected format lets the agent fix the input and make a new call. That restores the feature without changing the category.

**Closest distractor (A):** Marking validation errors retryable invites the same bad call again, which fails the same way.

### Q18 · A

**Why A is correct:** Eight tools that are versions of one job should become one tool with a type or format parameter. This cuts the toolset to about five tools. Descriptions were already rewritten, so the tool count is the real problem.

**Closest distractor (B):** Another description rewrite repeats a fix that already failed and leaves 12 similar choices.

### Q19 · D

**Why D is correct:** The names are confusing because they are nearly the same. Renaming each tool to match its purpose is cheap and direct. It fixes the cause of the mix-ups.

**Closest distractor (C):** Few-shot examples cost tokens on every call and leave the confusing names in place.

### Q20 · B

**Why B is correct:** ${VAR:-default} uses the variable when it is set and the default when it is not. Most developers get staging with no setup. Others set METRICS_URL to their local copy.

**Closest distractor (A):** Plain ${METRICS_URL} has no fallback, so it breaks for anyone who has not set the variable.

### Q21 · C

**Why C is correct:** The query worked, so this is a valid empty result, not an error. isError false with zero results tells the agent no orders match. The agent should not retry.

**Closest distractor (A):** The email may be valid, so calling it a validation error pushes the agent to doubt correct input.

### Q22 · A

**Why A is correct:** Grep finds direct callers by content. Reading the re-export file reveals the alias, and a second Grep finds its callers. Glob then finds the matching test files without reading everything.

**Closest distractor (C):** It misses callers that use the alias and wastes context by reading every test file.

### Q23 · B

**Why B is correct:** One generic tool with free-text instructions has no stable output contract. Splitting it into purpose-specific tools gives each a defined input and output. Downstream parsing then gets a predictable format.

**Closest distractor (A):** A longer description helps selection, but one generic tool still returns changing formats.

### Q24 · D

**Why D is correct:** An agent with a tool outside its role tends to misuse it. Scoped access means each agent gets only the tools its role needs. Removing the tool makes the misuse impossible.

**Closest distractor (A):** A prompt rule lowers the risk but does not guarantee it, while the tool stays available.

### Q25 · A

**Why A is correct:** A 403 for a missing role is a permission error. Retrying will not change access, so isRetryable is false. The agent should escalate so the right access can be approved.

**Closest distractor (B):** Retrying with backoff wastes calls, because the account's access will not change on its own.

### Q26 · C

**Why C is correct:** A remote server uses "type": "http" with a "url" field. "command" and "args" are for local servers that Claude Code starts as a process.

**Closest distractor (A):** command and args start a local process, which does not fit a server hosted by the vendor.

### Q27 · B

**Why B is correct:** The two descriptions are identical, so Claude cannot tell the tools apart. Clear descriptions with boundaries are the low-effort first fix. They tell Claude when cancel fits better than refund.

**Closest distractor (D):** A better name helps a little, but the identical descriptions still give Claude no rule for choosing.

### Q28 · D

**Why D is correct:** A constrained tool limits the subagent to the one query it needs. It removes slow ad hoc joins and access to unrelated tables. Read-only access does not stop reads of payroll data.

**Closest distractor (B):** The hook runs after the query, so the payroll data was already read and the slow joins still run.

### Q29 · C

**Why C is correct:** Finding where code throws an error is a content search, so Grep is the right tool. Glob only matches file names and paths. The throw sites live in other files' contents.

**Closest distractor (A):** A wider Glob still matches only file names, not the code that throws the error.

### Q30 · B

**Why B is correct:** One structured error shape lets Claude tell transient errors from business and other errors. isRetryable tells it whether to retry. The message helps it explain or fix the input.

**Closest distractor (D):** Blind retries inside each tool still retry policy blocks and keep the confusing outputs.

### Q31 · A

**Why A is correct:** The cause is thin descriptions, so expanding them is the first fix. Few-shot examples add tokens to every request. They also leave the real cause in place.

**Closest distractor (C):** Few-shot examples can influence tool choice; the issue is cost and missing the cause, not that they never work.

### Q32 · D

**Why D is correct:** The server is personal and experimental, so it belongs in the developer's user scope. That removes its tools and errors from teammates' sessions. The shared servers stay in .mcp.json.

**Closest distractor (A):** A default value hides the error message, but teammates still load the server and Claude still calls its tools.

### Q33 · C

**Why C is correct:** Too many tools make tool choice harder and less reliable. Scoped access gives each subagent only the tools its role needs. Four to five tools per agent is the target.

**Closest distractor (A):** Longer descriptions do not solve the decision load of 20 tools per agent.

### Q34 · B

**Why B is correct:** A timeout during a short outage is a transient error. A retry after a short wait is likely to succeed. isRetryable true tells the agent this.

**Closest distractor (C):** A timeout is not a problem with the input, and validation errors should not be marked retryable.

### Q35 · A

**Why A is correct:** A good description states the output format, including units. Once Claude knows amounts are in cents, it can convert them correctly. It also stops the extra calls made to guess the currency.

**Closest distractor (C):** Few-shot examples cost tokens on every call, while one line in the description fixes the cause.

### Q36 · C

**Why C is correct:** Edit changes only the matched text, so it uses fewer tokens. It does not overwrite the rest of the file with an old copy. Read plus Write stays as the fallback when Edit cannot find a unique match.

**Closest distractor (A):** Reading again shrinks the race window but keeps the high token cost and the full-file overwrite.

### Q37 · D

**Why D is correct:** MCP resources expose content catalogs by URI, and the application provides them. The agent can see what exists without exploratory tool calls. Tools are for actions, resources are for content like this index.

**Closest distractor (A):** A single list tool reduces calls but is still an exploratory call that Claude must decide to make each time.

### Q38 · B

**Why B is correct:** A small scoped tool serves the frequent simple checks with low latency. Complex research belongs with the search agent, through the coordinator. This keeps each agent's toolset focused.

**Closest distractor (D):** Expanding verify_fact turns a narrow tool into a generic one that duplicates the search agent's role.

### Q39 · A

**Why A is correct:** For business rule errors, the result should include a customer-friendly explanation. The agent can then tell the customer what the policy means. Internal codes alone do not help the customer.

**Closest distractor (D):** A policy name is still internal language, and the agent still lacks words it can share with the customer.

### Q40 · C

**Why C is correct:** The descriptions are already fixed, yet errors cluster on one keyword. That pattern points to wording outside the tool definitions. System prompt instructions can override good descriptions, so check them first.

**Closest distractor (D):** Few-shot examples add tokens and may hide the cause instead of removing it.

### Q41 · B

**Why B is correct:** Claude prefers built-in tools when an MCP tool's description is thin. A full description of data, inputs, and outputs shows why the MCP tool is better. This is the lowest-effort fix for the cause.

**Closest distractor (C):** A broad "always use MCP" rule can pull Claude to MCP tools for unrelated tasks and leaves the thin description.

### Q42 · A

**Why A is correct:** The tools serve three different roles. Role-based subagents each get only their own tools. Export tools are then unavailable during research.

**Closest distractor (D):** A prompt rule may reduce early exports, but the agent still chooses among 15 tools.

### Q43 · D

**Why D is correct:** Grep finds entry points by content, and Read follows only the files that matter. Following imports traces the flow step by step. This saves context compared with reading everything.

**Closest distractor (B):** File names may not contain "retry", so a Glob match list can miss the real logic.
