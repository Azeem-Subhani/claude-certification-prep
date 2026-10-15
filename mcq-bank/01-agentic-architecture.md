# Domain 1 MCQ Bank: Agentic Architecture & Orchestration

> 65 questions: 16 Easy, 26 Medium, 23 Hard. [Study notes](../domain-1-agentic-architecture-orchestration/README.md) · [Anti-patterns](../domain-1-agentic-architecture-orchestration/anti-patterns.md) · [Bank index](README.md)

## Questions

### Q1 · Task 1.1 · Easy

A dispatch agent for Northline Freight uses the Messages API with three tools. Claude's latest response has stop_reason "tool_use". It contains one tool_use block for get_route.

What should the loop do next?

- A. Return Claude's text to the user, because a tool request means a decision was made.
- B. Run get_route, return its output as a tool_result block, and call Claude again.
- C. Call Claude again with the same message list so it can confirm its tool choice first.
- D. Run get_route and show its raw output directly to the user as the final answer.

### Q2 · Task 1.4 · Medium

Harbor Mutual's claims agent must run check_fraud_flags before issue_payout. The system prompt already states this order in clear words. An audit shows the agent skipped the fraud check on 5% of approved payouts last month.

What is the BEST fix?

- A. Rewrite the system prompt with the rule in capitals at the top and again at the end.
- B. Add six few-shot examples that show the fraud check before every single payout.
- C. Add a routing classifier that sends payout requests to a separate payout agent.
- D. Make issue_payout return an error until a passed fraud check is recorded.

### Q3 · Task 1.2 · Hard

A market research system has a coordinator and three web search subagents. Each subagent gets the same prompt: "Research the European e-bike market." The final reports are good. But 70% of fetched pages are duplicates across subagents, and search costs have tripled. A teammate suggests a dedupe step before synthesis.

What should the architect do?

- A. Give each subagent a distinct scope, such as one subtopic or one source type.
- B. Add a dedupe step that removes repeated pages before the synthesis subagent runs.
- C. Let the subagents share a list of visited URLs so each one skips pages others fetched.
- D. Replace the three subagents with one search subagent that has a larger budget.

### Q4 · Task 1.5 · Easy

A shipping agent calls two carrier tools. One returns package weight in kilograms. The other returns weight in pounds. Claude sometimes compares the numbers as if they used the same unit.

Which mechanism BEST fixes this?

- A. A PreToolUse hook that blocks a carrier call when the unit of the result is unknown.
- B. A system prompt rule that tells Claude to always convert pounds into kilograms.
- C. A PostToolUse hook that converts every weight to kilograms before Claude reads it.
- D. Few-shot examples that show Claude converting the weights before it compares them.

### Q5 · Task 1.1 · Medium

A returns agent at Cartwell stops in the middle of about a third of multi-step tasks. These stops happen right after replies like "Let me check the warehouse stock." The loop ends when the first content block of a response is text.

What is the BEST fix?

- A. Decide based on stop_reason, and continue whenever it is "tool_use".
- B. Raise the loop's iteration limit so multi-step tasks have more room to finish.
- C. Tell Claude in the system prompt not to write any text before a tool call.
- D. End the loop only when the response text contains a phrase like "task complete".

### Q6 · Task 1.3 · Medium

A contract review system uses search subagents to find risky clauses in 40 supplier contracts. They send the coordinator short prose summaries. A synthesis subagent then writes the final memo. Lawyers say they cannot trace any finding back to a contract and clause.

What is the BEST fix?

- A. Add a firm rule to the synthesis prompt to always cite the contract and the clause.
- B. Return findings as records with contract name, clause number, and quote fields.
- C. Give the synthesis subagent a document tool so it can find each clause again.
- D. Raise max_tokens for the search subagents so their prose summaries are more detailed.

### Q7 · Task 1.6 · Easy

An accounts payable team processes vendor invoices. Every invoice goes through the same steps: extract fields, check totals, match the purchase order, and write a summary. The steps never change.

Which decomposition pattern fits BEST?

- A. Dynamic adaptive decomposition, where the agent plans new steps for each invoice.
- B. One large prompt that asks Claude to do all four of the steps in a single call.
- C. A coordinator that decides at run time which of the steps each invoice needs.
- D. A fixed sequential pipeline, where each step's output feeds the next prompt.

### Q8 · Task 1.7 · Medium

Yesterday a developer used a Claude Code session named "auth-audit" to map the login code. Nobody has changed any files since then. Today the developer wants to continue the same analysis with all the earlier context.

What should the developer do?

- A. Start a new session and ask Claude to read all of the login code again.
- B. Fork the "auth-audit" session so the original history stays protected.
- C. Run claude --resume auth-audit and continue the analysis.
- D. Start a fresh session and paste in a written summary of yesterday's work.

### Q9 · Task 1.1 · Hard

A travel agent's response has stop_reason "tool_use" and three tool_use blocks: flights, hotels, and car hire. The loop runs the first tool, appends one tool_result in a user message, and calls the API. The call fails with a 400 error. The team blames rate limits, because traffic is high today.

What is the BEST fix?

- A. Retry the failed API call with exponential backoff until the request succeeds.
- B. Run all three tools and return all three tool_result blocks in one user message.
- C. Turn off parallel tool use so Claude asks for only one tool in each response.
- D. Remove the two unanswered tool_use blocks from the history before the next call.

### Q10 · Task 1.4 · Easy

A support agent for an online furniture store escalates damaged-delivery cases to human staff. The staff use a queue tool that cannot show the chat transcript.

What should the escalation message contain?

- A. The customer ID, a case summary, the root cause, the amount, and a recommended action.
- B. A short note that says "Customer needs help with a damaged item, see chat for details."
- C. The customer's last message, copied exactly, so staff see the request in their words.
- D. A link to the chat log and a priority level that the agent picks from one to five.

### Q11 · Task 1.2 · Medium

A coordinator plans research on "risks to the global coffee supply." The final report covers climate effects on farms in depth. It says nothing about shipping, prices, or labor. Logs show each subagent did its assigned task well, and synthesis used every result.

Where is the root cause MOST likely?

- A. The search subagents used queries that were too narrow for their assigned topics.
- B. The synthesis subagent dropped findings that did not match the report's main theme.
- C. The model's context window was too small to hold all of the search results at once.
- D. The coordinator split the topic into subtasks that covered only one area.

### Q12 · Task 1.1 · Easy

A billing agent's lookup_invoice tool raises an exception when the database times out. Right now, the exception crashes the loop and the user gets no answer.

How should the loop handle a tool that fails while running?

- A. Stop the loop and show the user a general "something went wrong" message.
- B. Leave out the tool_result for that call and ask Claude to try something else.
- C. Catch the exception and return a tool_result with is_error true and a clear message.
- D. Retry the Messages API request with backoff, because the error came from the API.

### Q13 · Task 1.3 · Easy

A coordinator calls a pricing subagent to list competitor prices. Later, it calls the same subagent again with: "Compare these competitors to our new price." The subagent replies that it has no competitor list.

What is the BEST fix?

- A. Include the competitor list from the first call in the second call's prompt.
- B. Ask the subagent to look back at its earlier conversation before it answers.
- C. Keep the subagent running between calls so it holds on to the earlier findings.
- D. Tell the subagent in its system prompt to remember the results of earlier calls.

### Q14 · Task 1.5 · Medium

Brightcart's sales agent can call apply_discount. Policy says discounts above 20% need manager approval. The team wants these calls sent to an approval queue instead of running.

Which design BEST meets this policy?

- A. A PostToolUse hook that checks each discount and reverses any above 20%.
- B. A PreToolUse hook that sends calls above 20% to the approval queue.
- C. A system prompt rule that tells Claude to ask a manager about large discounts.
- D. A line in the apply_discount description that says large discounts need approval.

### Q15 · Task 1.6 · Hard

Kestrel Health wants a security audit of an old billing platform. Nobody knows how many services it has. The design documents are five years old. A team lead wants to reuse last year's fixed six-step audit pipeline from a different system, because it is proven.

Which approach is BEST?

- A. Reuse the proven six-step pipeline and add two extra steps for the older code.
- B. Start one subagent for each service listed in the design documents, all in parallel.
- C. Load the full codebase into one long session with a large context window.
- D. Map the system first, rank risk areas, and change the plan as findings appear.

### Q16 · Task 1.7 · Hard

Last week a developer ran a long Claude Code analysis of a payments service. Since then, the team upgraded the main framework and refactored about 40 files. The developer wants to continue the analysis. The developer does not want to lose last week's useful findings.

What is the BEST way to continue?

- A. Resume the old session and ask Claude to re-read the whole codebase first.
- B. Fork the old session so the new work starts on a separate, clean branch.
- C. Start a fresh session with a structured summary of findings and changed areas.
- D. Resume the old session and tell Claude to ignore all results older than today.

### Q17 · Task 1.1 · Medium

A research agent uses a model that supports all tool_choice options. The team set tool_choice to "any" so Claude would always use its search tools. Now every run continues until the safety cap stops it. No final answer is ever written.

What is the BEST fix?

- A. Lower the safety cap so that runs stop sooner and cost less money.
- B. Use "auto" in the loop, so Claude can end its turn with a text answer.
- C. Add a system prompt rule that tells Claude exactly when to stop searching.
- D. End the loop when Claude's tool input contains the word "final".

### Q18 · Task 1.4 · Hard

A telecom customer writes: "My bill is wrong, my router keeps dropping, and I want to add a line." The agent answers only the billing issue. A teammate proposes three separate conversations, one per issue, to keep each context clean. Customers already complain about repeating their details.

What is the BEST design?

- A. Split it into three issues, check them in parallel with shared context, and reply once.
- B. Open three separate conversations, one per issue, and copy the customer ID into each.
- C. Route the message to the queue for the most urgent issue, and handle the rest later.
- D. Add a prompt rule that tells the agent to answer the issues in the order they appear.

### Q19 · Task 1.2 · Easy

A team builds a multi-agent research system. One engineer wants subagents to send results straight to each other. The architect insists that all messages go through the coordinator.

What is the MAIN benefit of the architect's design?

- A. Subagents can share one context window, which lowers the total token cost.
- B. Each subagent can remember results from earlier calls without extra prompts.
- C. One place controls routing, logging, and error handling for the whole system.
- D. The coordinator can skip delegation and answer most of the questions by itself.

### Q20 · Task 1.3 · Hard

A coordinator must analyze four regional markets. The four analyses do not depend on each other. Today it issues one Task call per turn, so a run takes about four times as long as one analysis. Users wait for the result in the app. One engineer suggests the Message Batches API, and another suggests a smaller model.

What should the architect do FIRST?

- A. Send the four analyses through the Message Batches API to cut cost and time.
- B. Merge the four regions into one subagent call that uses a longer prompt.
- C. Switch the subagents to a smaller model so each analysis returns sooner.
- D. Have the coordinator issue all four Task calls in a single response.

### Q21 · Task 1.5 · Hard

A multi-tenant analytics agent calls query_records with a filter object. Each user may see only their own tenant's rows. Claude sometimes leaves out the tenant_id filter, and the query returns rows from all tenants. A developer proposes a PostToolUse hook that removes other tenants' rows from each result.

What is the BEST fix?

- A. A PreToolUse hook that sets tenant_id from the session before every query runs.
- B. The proposed PostToolUse hook, which removes other tenants' rows from each result.
- C. A system prompt rule, stated twice, that says every query must include tenant_id.
- D. A tool schema change that marks the tenant_id field as a required input.

### Q22 · Task 1.6 · Medium

A legal team asks an agent to review 30 vendor contracts in one prompt. The first contracts get detailed notes. The last ten get one line each. A liability clause is flagged in one contract but approved in another with the same wording.

What is the BEST fix?

- A. Use a model with a larger context window so all 30 contracts fit with room to spare.
- B. Review each contract in its own pass, then run one cross-contract comparison pass.
- C. Split the contracts into three batches of ten and review each batch on its own.
- D. Add a prompt line that tells Claude to give every contract the same attention.

### Q23 · Task 1.1 · Medium

A developer builds a loop for an inventory agent. After it runs the tools, the code sends a user message. The message starts with a text block, "Here are the results:", and the tool_result blocks follow. The API rejects the request.

What should the developer change?

- A. Wrap each tool_result block inside a text block so that every block has the same format.
- B. Send the tool results back in an assistant message instead of in a user message.
- C. Put all tool_result blocks first in the user message, before any text.
- D. Remove the tool_use_id fields, because Claude can match the results by their order.

### Q24 · Task 1.7 · Easy

A developer finished a shared analysis of a slow database layer in Claude Code. Now the developer wants to try two migration plans from that same point. The two attempts must stay separate.

Which option fits BEST?

- A. Resume the session and try both plans one after the other in the same history.
- B. Start two fresh sessions that have no context from the earlier analysis.
- C. Use --continue twice so that each run picks up the most recent session.
- D. Use fork_session to create two branches from the shared analysis.

### Q25 · Task 1.4 · Easy

A banking agent's prompt says: "Always verify the caller's identity before changing an address." Tests show the agent skips the check in 2% of calls. A manager asks for stronger wording to reach zero.

Why can stronger wording not reach zero?

- A. System prompts are ignored once a conversation has more than a few turns.
- B. Prompt rules make the right order likely, but they cannot guarantee it.
- C. Identity checks need few-shot examples, and this prompt does not have any.
- D. The rule must go in the tool description, because Claude ignores the system prompt.

### Q26 · Task 1.2 · Medium

A coordinator must research "the impact of automation on logistics." It will use four search subagents. The team wants broad coverage and little overlap.

Which task split is BEST?

- A. Warehouse work, freight transport, last-mile delivery, and workforce and regulation.
- B. Four subagents that each research the full topic, so the results can be cross-checked.
- C. Warehouse robots, warehouse robot vendors, warehouse robot costs, and robot safety.
- D. One subagent for each search engine, with each one searching the full topic.

### Q27 · Task 1.3 · Medium

A coordinator has two subagents defined with AgentDefinition. Both have the description "Helper agent for analysis." The coordinator often sends pricing questions to the legal reviewer and contract questions to the pricing analyst.

What is the BEST fix?

- A. Remove tools from both subagents so each one can do only its own type of work.
- B. Rename the subagents to legal_agent and pricing_agent but keep the descriptions.
- C. Give each subagent a description that says what it does and when to use it.
- D. Add keyword rules to the coordinator prompt, such as "contract" means the legal agent.

### Q28 · Task 1.5 · Easy

An IT helpdesk agent can call delete_user_account. Policy says this call must never run without an approved change ticket. The team wants code to stop the call before anything is deleted.

Which hook should they use?

- A. A PostToolUse hook that checks for the ticket after the account is deleted.
- B. A Stop hook that checks for a ticket when the agent finishes its whole task.
- C. A PostToolUse hook that removes the deletion result from Claude's view.
- D. A PreToolUse hook that blocks the call when no approved ticket exists.

### Q29 · Task 1.1 · Medium

An order-status agent calls get_order for the same order five or six times in one run. Logs show the tool runs and returns data each time. After each call, the loop appends Claude's assistant message to the history, but nothing else.

What is the MOST likely cause?

- A. The safety cap is set too high, so Claude has room to repeat the same calls.
- B. The tool output is never added to the history as a tool_result.
- C. The get_order description does not tell Claude that its results are final.
- D. The loop has no cache, so each repeated call has to reach the database again.

### Q30 · Task 1.6 · Medium

A logistics platform has an intermittent failure in its route planner. Nobody knows which service causes it. The ops team has a runbook with fixed steps for known outages. They want an agent to investigate.

Which approach fits BEST?

- A. Run the runbook steps as a fixed pipeline and report what each step finds.
- B. Send all logs from the past week to one prompt and ask Claude for the cause.
- C. Start one subagent per service, each with the same fixed checklist of steps.
- D. Use adaptive decomposition that updates the plan as evidence appears.

### Q31 · Task 1.4 · Hard

An insurance support agent escalates complex claims to human adjusters. Adjusters handle about 60 cases an hour. Their queue tool cannot display chat transcripts. Today the agent writes notes like "Customer upset about claim, please review." A manager proposes attaching the full transcript as text to each case.

What is the BEST change?

- A. Attach the full transcript as text to each case so adjusters have every detail.
- B. Tell the agent in its prompt to write longer and more detailed escalation notes.
- C. Send a structured summary: claim ID, issue, root cause, amount, and next step.
- D. Let adjusters ask the agent follow-up questions in a chat before they act.

### Q32 · Task 1.3 · Hard

A claims-review subagent gets a 14-step procedure from the coordinator: open the PDF, go to page 2, read box 7, and so on. It works for standard forms. It fails on scanned letters, emails, and forms from other states. The team plans to add steps for each new document type.

What is the BEST change?

- A. Give the subagent the goal, the facts it must find, and the quality bar instead.
- B. Add a separate step-by-step procedure for each new document type to the prompt.
- C. Switch the subagent to a larger model so it can follow longer procedures well.
- D. Reject documents that do not match the standard form and send them to staff.

### Q33 · Task 1.1 · Easy

A shipping agent's response has a text block: "I'll check the tracking status now." A tool_use block for track_parcel follows it. The stop_reason is "tool_use".

What should the loop do?

- A. Stop and show the text, because the response contains a message for the user.
- B. Run track_parcel and return its result, because the task is not finished.
- C. Ask Claude to choose between the text and the tool call before it continues.
- D. Show the text, wait for the user to reply, and then run track_parcel afterward.

### Q34 · Task 1.7 · Medium

A developer works on several Claude Code investigations in one repo. Last week the developer named a session "billing-bug" when starting it. Since then, they ran three other sessions in the same folder. Nothing changed in the billing code, and they want to return to that work.

What is the BEST approach?

- A. Run claude -c, which continues the most recent session in this folder.
- B. Run claude --fork-session on the latest session to copy its work forward.
- C. Run claude --resume billing-bug to reopen that named session.
- D. Start a new session and describe the billing bug again from memory.

### Q35 · Task 1.2 · Hard

A research system has a search subagent and a fact-check subagent. To cut latency, an engineer proposes a shared database. The search subagent would write results there, and the fact-checker would read them directly. The coordinator would see only final verdicts. Debugging wrong verdicts is already hard.

What should the architect do?

- A. Approve the shared database, because it removes one round trip for each query.
- B. Let the search subagent call the fact-check subagent directly when it finishes.
- C. Merge search and fact-checking into one subagent so there is no handoff at all.
- D. Keep results flowing through the coordinator, which passes them to the fact-checker.

### Q36 · Task 1.5 · Medium

A third-party freight API returns status as numbers from 1 to 7. Claude sometimes reads status 4 as "delivered," but 4 means "held at customs." The team cannot change the API. The system prompt already lists what each code means.

What is the BEST fix?

- A. Move the code table to the top of the system prompt and repeat it at the end.
- B. Add a PostToolUse hook that replaces each code with its plain-text meaning.
- C. Add a PreToolUse hook that asks the API to return text instead of numbers.
- D. Add few-shot examples that show the correct reading of each status code.

### Q37 · Task 1.6 · Easy

A vendor onboarding tool uses three prompts. The first extracts fields from a form. The second checks the fields against rules. The third writes a short report from the checked fields.

What is this pattern called?

- A. Prompt chaining, where each step's output becomes the next step's input.
- B. Dynamic adaptive decomposition, where the plan can change after each step.
- C. Hub-and-spoke orchestration, where a coordinator routes all of the messages.
- D. Parallel fan-out, where independent subagents all run at the same time.

### Q38 · Task 1.1 · Hard

A data migration agent uses a limit of 15 iterations as its main stop rule. Large migrations get cut off with no error. One developer proposes raising the limit to 200. Another proposes removing the limit, since stop_reason already exists.

What is the BEST design?

- A. Raise the limit to 200 and keep it as the main way that the loop stops.
- B. Remove the limit and rely only on stop_reason to end every single run.
- C. Stop on stop_reason, and keep a high limit as a safety net with a warning log.
- D. Stop when Claude writes "MIGRATION COMPLETE," and keep the limit at 15.

### Q39 · Task 1.4 · Medium

A pharmacy support agent has four new rules. The team can code-enforce only one of them this sprint. The other three will go in the prompt for now.

Which rule MOST needs code enforcement?

- A. Include the pharmacy's opening hours in every closing message to a customer.
- B. Suggest a cheaper generic drug when one exists for the customer's prescription.
- C. List medicines in alphabetical order whenever the agent writes a summary.
- D. Verify the patient's identity before sharing any prescription details.

### Q40 · Task 1.3 · Medium

A coordinator has 80,000 tokens of history. To be safe, it passes this whole history to every subagent. Costs have risen, and subagents often work on side topics from old turns.

What is the BEST change?

- A. Pass each subagent its goal, constraints, and only the findings it needs.
- B. Pass no context at all, and let each subagent search for what it needs.
- C. Keep passing the full history, and move subagents to a larger context model.
- D. Pass the same one-sentence summary of the whole history to every subagent.

### Q41 · Task 1.5 · Hard

Fenmark Bank's agent can call aml_check and transfer_funds. AML (anti-money laundering) rules say no transfer may run before a passed check. The team has a strong prompt rule and a PostToolUse hook that alerts compliance after unchecked transfers. An audit still found three unchecked transfers last quarter.

What is the BEST change?

- A. Make the PostToolUse hook also freeze the account after each unchecked transfer.
- B. Add few-shot examples of aml_check before transfer_funds to the system prompt.
- C. Add a PreToolUse hook that blocks transfer_funds until aml_check has passed.
- D. Rename transfer_funds to transfer_after_aml_check so the order is clear to Claude.

### Q42 · Task 1.6 · Hard

A team reviews a 60-file framework migration. To avoid long prompts, they split the files into six batches of ten. Each batch gets a careful review. But a function renamed in batch 2 is still called by its old name in batch 5, and nobody catches it.

What is the BEST fix?

- A. Use three batches of twenty files so that more related files land together.
- B. Review each file in its own pass, then run one cross-file integration pass.
- C. Review all 60 files in one pass with a model that has a large context window.
- D. Keep the six batches and add a prompt line that asks for cross-batch checks.

### Q43 · Task 1.7 · Medium

After a large refactor, a developer will start a fresh session instead of resuming. The developer wants to carry over what still matters from the old session.

What should the injected summary contain?

- A. The full old transcript, so that no detail from the earlier work is lost.
- B. Only the old session's final answer, since it already sums up the work.
- C. Nothing at all, so that the agent rediscovers everything from the current files.
- D. Key decisions, confirmed findings, open questions, and the changed files.

### Q44 · Task 1.1 · Medium

An agent uses a server-side web search tool. Some responses come back with stop_reason "pause_turn". The loop treats any value other than "tool_use" as finished. Users get half-written answers.

What should the loop do on "pause_turn"?

- A. Send the paused response back as it is and call the API again.
- B. Return the partial answer and mark the task as complete in the logs.
- C. Resend only the original user message so that the turn starts over.
- D. Raise max_tokens and repeat the same request without the paused response.

### Q45 · Task 1.2 · Easy

A research system's news subagent times out during a run. The other subagents finish normally.

In a hub-and-spoke design, which component should decide what happens next?

- A. The coordinator, which can retry, use other sources, or note the gap.
- B. The synthesis subagent, which should search for the news content by itself.
- C. The news subagent, which should call another subagent and ask it for help.
- D. The end user, who should rerun the full research request at a later time.

### Q46 · Task 1.3 · Easy

A developer builds a coordinator with the Claude Agent SDK. It must start subagents to handle parts of the work.

Which tool lets the coordinator start a subagent?

- A. The Bash tool, which can launch a new agent process from the shell.
- B. The Task tool, which current SDK docs call the Agent tool.
- C. The Read tool, which loads a subagent's definition file into context.
- D. The fork_session option, which creates a child agent from the coordinator.

### Q47 · Task 1.4 · Hard

Ridgeway Telecom gates process_refund until get_customer has been called. A review finds a refund paid to the wrong account. Claude had called get_customer with a misspelled name, which returned "not found." The gate still opened. The team proposes human review of every refund.

What is the BEST fix?

- A. Keep the gate as it is, and add a prompt rule to check the get_customer result.
- B. Add human review of every refund, as the team proposes, before money moves.
- C. Add a PostToolUse hook that rewrites "not found" into a clearer error message.
- D. Open the gate only when a verified customer ID matches the refund's account.

### Q48 · Task 1.5 · Medium

A sales agent can call send_email. Policy says emails to addresses outside the company domain must go to a draft folder for review. Internal emails can be sent at once.

Which design BEST fits?

- A. A system prompt rule that tells Claude to save all external emails as drafts.
- B. A PostToolUse hook that recalls external emails right after they are sent.
- C. A PreToolUse hook that redirects external emails to a review draft.
- D. Two email tools, with a description that says which one to use for external mail.

### Q49 · Task 1.6 · Medium

Every night an agent validates shipping manifests with the same five checks. The team built it as an adaptive planner that writes a new plan for each run. Some nights it skips a check, and the order of the checks changes.

What is the BEST change?

- A. Add a prompt rule that lists all five checks that the planner must include.
- B. Add a second agent that reviews each night's plan before the run starts.
- C. Replace the planner with a fixed sequential pipeline of the five checks.
- D. Give the planner more examples of good plans from successful past nights.

### Q50 · Task 1.1 · Hard

A report agent's loop runs tools when stop_reason is "tool_use" and treats every other value as success. Some users get reports that end mid-sentence. Others get empty reports. Logs show some "max_tokens" and some "refusal" values. A teammate proposes auto-retrying any non-"end_turn" response three times.

What is the BEST change?

- A. Auto-retry every response that did not end with "end_turn" up to three times.
- B. Treat only "end_turn" as success, and log and handle other values as unfinished.
- C. Keep the loop as it is, and check that the final text ends with a full stop.
- D. Raise the safety cap so the loop has more turns to finish each report.

### Q51 · Task 1.3 · Hard

A due-diligence coordinator sends three subagents to review a vendor: finance, security, and legal. Each returns results in its own shape: prose, a table, or a bullet list. The coordinator often drops or mixes up findings when it merges them. A teammate proposes a fourth subagent to reformat the outputs.

What is the BEST fix?

- A. Add a fourth subagent that reformats all three outputs into one shared shape.
- B. Ask the coordinator to read each output twice before it merges the findings.
- C. Merge the three reviews into one subagent so there is only a single output.
- D. Define one shared output structure in each subagent's delegation prompt.

### Q52 · Task 1.4 · Medium

A gate blocks process_refund until the customer's identity is verified. When it blocks, it returns only "Error." Logs show Claude then calls process_refund again and again until the safety cap stops the run.

What is the BEST fix?

- A. Make the gate say why it blocked: "Customer not verified. Call get_customer first."
- B. Lower the safety cap so that blocked runs stop sooner and cost less money.
- C. Remove the gate and put the verification rule in the system prompt instead.
- D. Add few-shot examples that show get_customer before every process_refund call.

### Q53 · Task 1.2 · Medium

A research coordinator's prompt says: "Always run search, analysis, fact-check, and report subagents in order." Users who ask simple factual questions wait 90 seconds, and costs are high.

What is the BEST change?

- A. Cache answers to common questions so that repeated questions skip the pipeline.
- B. Run all four subagents in parallel instead of one after the other in order.
- C. Move all four subagents to a smaller and faster model to cut the wait time.
- D. Let the coordinator choose which subagents each query actually needs.

### Q54 · Task 1.7 · Easy

A developer asks why a fresh session with a summary is better than resuming after a big refactor.

What is the MAIN reason?

- A. Resumed sessions cannot read any files that changed after the session started.
- B. Old tool results in the history no longer match the files and can mislead Claude.
- C. Fresh sessions always run on a newer model version than resumed sessions do.
- D. Resumed sessions lose all of their earlier history once a refactor happens.

### Q55 · Task 1.1 · Hard

A retail support loop hard-codes the tool order: lookup_order, then check_stock, then create_shipment. Customers who only ask "Where is my order?" get unwanted reshipments. A developer suggests if-else branches for each request type. The only high-risk step is create_shipment, which needs a paid order.

What is the BEST design?

- A. Add if-else branches in the loop code for each type of customer request.
- B. Keep the fixed order, but ask Claude to confirm before it runs create_shipment.
- C. Let Claude choose tools from context, and gate create_shipment on a paid order.
- D. Let Claude choose every step, and write the payment rule in the system prompt.

### Q56 · Task 1.3 · Medium

A report-writer subagent only needs to read research notes. Its AgentDefinition gives it Read, Write, Edit, and Bash. Last week it deleted a notes file while "cleaning up."

What is the BEST fix?

- A. Limit the subagent's tools in its AgentDefinition to the read tools it needs.
- B. Add a rule to the subagent's prompt that forbids deleting or moving any files.
- C. Ask the coordinator to check the notes folder after each subagent run ends.
- D. Run the subagent on a larger model that follows prompt rules more closely.

### Q57 · Task 1.4 · Hard

A customer writes about three problems: a late parcel, an item in the wrong color, and a $700 double charge. Refunds above $500 need a human. The agent can solve the first two problems itself. Today it escalates the whole case, and the human redoes all three.

What is the BEST behavior?

- A. Escalate the whole case, since one of the issues needs a human anyway.
- B. Solve two, hand off the charge with a structured summary, and reply once.
- C. Solve the first two issues, then tell the customer to call about the charge.
- D. Process the $700 refund, since the payment record clearly shows two charges.

### Q58 · Task 1.2 · Medium

A multi-agent research report contains a wrong statistic. The system uses hub-and-spoke orchestration. The team must find which agent introduced the error.

What is the BEST first step?

- A. Ask the synthesis subagent to explain where it found the wrong statistic.
- B. Rerun the full pipeline with a lower temperature and compare the reports.
- C. Check the coordinator's logs of each subagent's inputs and outputs.
- D. Add a channel where subagents can review each other's findings directly.

### Q59 · Task 1.5 · Easy

A developer adds a PostToolUse hook to process_refund. The hook checks whether the amount is above the $500 limit. The developer expects it to stop large refunds.

Why will this NOT work?

- A. PostToolUse hooks run only when the tool returns an error to the agent.
- B. PostToolUse hooks can read a tool's inputs, but they cannot read its outputs.
- C. PostToolUse hooks run only after the agent ends the whole session.
- D. PostToolUse runs after the tool, so the refund has already happened.

### Q60 · Task 1.6 · Hard

An agent adds tests to an old payments codebase with an adaptive plan. Halfway through, it finds that three modules share a hidden global config. Tests for these modules fail at random. The original plan has eight modules left in a set order.

What should happen next?

- A. Update the plan to handle the shared config first, then reorder the rest.
- B. Keep the original order, and list the config issue in the final report.
- C. Stop the run and start over with a completely new plan from the beginning.
- D. Start a separate agent on the config issue and keep the old plan running.

### Q61 · Task 1.2 · Hard

A user asks for a report on four areas: market size, competitors, regulation, and pricing. The pricing subagent returns nothing, because its main sources are behind paywalls. Other public sources exist, such as vendor pricing pages. The coordinator delivers the report without a pricing section.

What should the coordinator do?

- A. Ask the synthesis subagent to fill in pricing from its own general knowledge.
- B. Rerun the whole pipeline from the start with the same four subtasks.
- C. Check coverage, send a targeted pricing query, and rerun the synthesis.
- D. Let the pricing subagent ask the competitor subagent for any price data.

### Q62 · Task 1.3 · Hard

A coordinator sends a draft report to a review subagent to check that claims match sources. It passes only the draft text. The reviewer approves claims it cannot check, and it flags some true claims as unsupported. A teammate suggests giving the reviewer a web search tool.

What is the BEST fix?

- A. Give the reviewer a web search tool so it can find each source again.
- B. Pass the draft together with the structured findings and their source metadata.
- C. Let the writer subagent send its sources directly to the review subagent.
- D. Tell the reviewer to mark every claim it cannot verify as "uncertain."

### Q63 · Task 1.4 · Medium

A loan-servicing agent must confirm a borrower's identity before it shares the account balance. The team added six few-shot examples. Failures dropped from 7% to 3%. A manager asks if this is good enough.

What should the architect recommend?

- A. Accept the result, because the examples cut failures by more than half.
- B. Add six more examples to push the failure rate even closer to zero.
- C. Move the rule to the top of the prompt, and keep the examples as well.
- D. Add a gate that blocks the balance tool until identity is confirmed.

### Q64 · Task 1.2 · Hard

Five research subagents each return 40 pages of raw search text to the coordinator. The coordinator's context fills up, and its synthesis misses key points. The team must keep citations. One engineer proposes a model with a larger context window.

What is the BEST change?

- A. Have subagents return short structured findings that keep source metadata.
- B. Move the coordinator to a model that has a much larger context window.
- C. Cut each subagent's output to its first 2,000 tokens before synthesis runs.
- D. Strip the source metadata from all results to make room for more content.

### Q65 · Task 1.7 · Hard

A developer's outage investigation session now has 150,000 tokens of history. Much of it is dead ends. No files have changed. Answers are getting vague and repeat ideas already ruled out. The developer wants to keep the confirmed findings.

What is the BEST next step?

- A. Keep resuming the session, since no files changed and the context is still valid.
- B. Start a fresh session with a summary of confirmed findings and ruled-out ideas.
- C. Fork the session so that the next attempts happen on a separate clean branch.
- D. Resume the session and tell Claude to ignore the dead ends from earlier turns.

## Answer key

### Q1 · B

**Why B is correct:** "tool_use" means Claude wants a tool and is not finished. The loop runs the tool, adds a tool_result block in a new user message, and calls Claude again.

**Closest distractor (D):** The raw tool output is input for Claude, not a final answer, so the loop must send it back to Claude.

### Q2 · D

**Why D is correct:** A payout without a fraud check is a money and compliance risk. A prerequisite gate in code blocks the payout every time. The prompt already states the order, so prompt changes have reached their limit.

**Closest distractor (B):** Few-shot examples can lower the failure rate, but they never guarantee the order.

### Q3 · A

**Why A is correct:** The subagents overlap because they all got the same scope. Giving each one a different subtopic or source type removes the duplicate work at its source.

**Closest distractor (B):** A dedupe step cleans the output, but the duplicate searches and their costs still happen.

### Q4 · C

**Why C is correct:** A PostToolUse hook changes tool results before Claude reads them. Converting every weight to one unit there makes the data consistent every time.

**Closest distractor (B):** A prompt rule asks Claude to convert, but model-side conversion is not reliable for data formats.

### Q5 · A

**Why A is correct:** A response can hold text and a tool_use block together. Checking the first block's type ends the loop too early. stop_reason is the reliable signal.

**Closest distractor (C):** It tries to change Claude's behavior to fit a broken check, instead of fixing the check itself.

### Q6 · B

**Why B is correct:** The source details were lost when subagents sent prose summaries. Structured records keep content and metadata apart, so the memo can cite each clause.

**Closest distractor (A):** The synthesis subagent cannot cite details it never received, so a prompt rule does not help.

### Q7 · D

**Why D is correct:** The steps are known and never change. A fixed sequential pipeline, also called prompt chaining, fits known steps best.

**Closest distractor (A):** Adaptive decomposition is for unknown scope, and here it only adds cost and variation.

### Q8 · C

**Why C is correct:** No files changed, so the earlier context is still valid. Resuming the named session continues the same work with its full history.

**Closest distractor (D):** A fresh session with a summary is for stale or messy history, and here it throws away valid context.

### Q9 · B

**Why B is correct:** Every tool_use block needs a matching tool_result in the next user message. The 400 error comes from two unanswered tool_use blocks, not from rate limits.

**Closest distractor (C):** Turning off parallel tool use hides the bug, but it also makes every run slower.

### Q10 · A

**Why A is correct:** The staff cannot see the chat, so the handoff must stand alone. It needs the customer ID, summary, root cause, amount, and a recommended action.

**Closest distractor (D):** The staff cannot open the transcript, so a link does not give them the facts.

### Q11 · D

**Why D is correct:** The subagents did their tasks well, and synthesis used all results. Whole areas are missing, so the scope was too narrow when the coordinator split the topic.

**Closest distractor (A):** Narrow queries would reduce depth inside a topic, but here whole topics were never assigned.

### Q12 · C

**Why C is correct:** A tool failure should go back to Claude as a tool_result with is_error true. Claude can then retry, try another path, or explain the problem.

**Closest distractor (D):** Backoff is for errors from the Claude API itself, but this failure came from the tool.

### Q13 · A

**Why A is correct:** Each subagent call starts with an empty context. The coordinator must pass the competitor list in the second prompt.

**Closest distractor (D):** A prompt cannot give the subagent a memory it does not have.

### Q14 · B

**Why B is correct:** A PreToolUse hook runs before the tool. It can stop large discounts and send them to the approval queue every time.

**Closest distractor (A):** PostToolUse runs after the discount is applied, so it can only try to undo it.

### Q15 · D

**Why D is correct:** The scope is unknown, and the documents are stale. Adaptive decomposition maps the system first, ranks the risks, and changes the plan as it learns.

**Closest distractor (B):** Parallel subagents look efficient, but the old service list is probably incomplete.

### Q16 · C

**Why C is correct:** After a framework upgrade and 40 changed files, old tool results are stale. A fresh session with a structured summary keeps the findings without the stale history.

**Closest distractor (D):** The stale results stay in the history, and a prompt cannot reliably make Claude ignore them.

### Q17 · B

**Why B is correct:** With "any", Claude must call a tool every turn, so it can never end with "end_turn". "auto" lets Claude choose to answer when it is done.

**Closest distractor (C):** A prompt rule cannot win against a setting that forces a tool call every turn.

### Q18 · A

**Why A is correct:** The agent should split the message into separate issues and check them in parallel. It should share the customer context and send one combined reply.

**Closest distractor (B):** Separate conversations lose the shared context and make customers repeat themselves.

### Q19 · C

**Why C is correct:** Hub-and-spoke gives one place for routing, logging, and error handling. This makes the system easier to observe and control.

**Closest distractor (B):** Subagents do not remember earlier calls in any design, so this is not a benefit of the hub.

### Q20 · D

**Why D is correct:** Independent subagent tasks should start as several Task calls in one coordinator response. They then run in parallel, which fixes the delay at its cause.

**Closest distractor (C):** A smaller model shortens each call, but the four calls still run one after another.

### Q21 · A

**Why A is correct:** A PreToolUse hook can change the tool input before the query runs. Setting tenant_id from the session makes every query safe, whatever Claude sends.

**Closest distractor (B):** The query has already read other tenants' data, so filtering afterward is too late for a security rule.

### Q22 · B

**Why B is correct:** Uneven notes and inconsistent verdicts are signs of attention dilution. One pass per contract plus a cross-contract pass fixes both.

**Closest distractor (C):** Batches reduce the load, but no pass compares contracts across batches.

### Q23 · C

**Why C is correct:** In the user message, tool_result blocks must come before any text. Moving the results first makes the request valid.

**Closest distractor (B):** Tool results belong in a user message, so an assistant message is the wrong role.

### Q24 · D

**Why D is correct:** fork_session creates separate branches from the same point. Each plan can then grow from the shared analysis without affecting the other.

**Closest distractor (A):** Trying both plans in one history mixes them, so the second attempt is shaped by the first.

### Q25 · B

**Why B is correct:** Prompt guidance is probabilistic. It makes the right order likely but never guaranteed, so a security rule needs code.

**Closest distractor (C):** Few-shot examples also only make the behavior more likely.

### Q26 · A

**Why A is correct:** These four areas are distinct and together cover the topic. This split gives broad coverage with little overlap.

**Closest distractor (C):** Each subtask is distinct, but all four stay inside warehouse robots, so the split is too narrow.

### Q27 · C

**Why C is correct:** The coordinator picks a subagent based on its description. Clear descriptions that say when to use each one fix the wrong routing.

**Closest distractor (D):** Keyword rules are brittle and fail on questions that use other words.

### Q28 · D

**Why D is correct:** PreToolUse runs before the tool. It can block the deletion when no approved ticket exists.

**Closest distractor (A):** PostToolUse runs after the account is already gone.

### Q29 · B

**Why B is correct:** Claude never sees the tool output, so it asks for the same data again. Every result must go into the history as a tool_result.

**Closest distractor (D):** A cache makes repeated calls cheaper, but Claude still never sees the results.

### Q30 · D

**Why D is correct:** The cause is unknown, so the agent cannot know the steps in advance. Adaptive decomposition changes the plan as evidence appears.

**Closest distractor (A):** The runbook fits known outages, but this failure is not a known pattern.

### Q31 · C

**Why C is correct:** Adjusters cannot see transcripts and have little time. A structured summary gives them what they need to act at once.

**Closest distractor (A):** A full transcript has the facts, but adjusters must dig through it, which is slow at 60 cases an hour.

### Q32 · A

**Why A is correct:** Goals and quality criteria let the subagent adapt to new document types. Fixed procedures break when the input does not match the script.

**Closest distractor (B):** More procedures only cover known types, and the next new type fails again.

### Q33 · B

**Why B is correct:** A response can contain text and a tool call together. The stop_reason is "tool_use", so the loop runs the tool and continues.

**Closest distractor (A):** Text in a response does not mean the task is done.

### Q34 · C

**Why C is correct:** No files changed, so resuming is right. Resuming by name opens the exact session the developer wants.

**Closest distractor (A):** -c continues the latest session in the folder, which is one of the three newer sessions.

### Q35 · D

**Why D is correct:** Hub-and-spoke keeps all data flowing through the coordinator. This keeps observability and error control, which the team needs for debugging.

**Closest distractor (A):** The shared database saves a little time, but the coordinator can no longer see or control the data flow.

### Q36 · B

**Why B is correct:** A PostToolUse hook can turn each code into plain text before Claude reads it. This removes the misreading every time.

**Closest distractor (A):** The prompt already lists the codes, so repeating it will not make the reading reliable.

### Q37 · A

**Why A is correct:** Each prompt uses the output of the one before it. This is prompt chaining, a fixed sequential pipeline.

**Closest distractor (B):** Adaptive decomposition changes the plan as it goes, but these steps are fixed.

### Q38 · C

**Why C is correct:** stop_reason should decide when the loop ends. A high limit with a warning log stays as a safety net for stuck loops.

**Closest distractor (B):** stop_reason is the right main rule, but without a cap a stuck loop can run forever.

### Q39 · D

**Why D is correct:** Sharing prescription details before verifying identity is a security and privacy failure. Rules like this need deterministic code.

**Closest distractor (B):** A generic-drug suggestion involves money, but a miss only loses a saving, so a prompt is enough.

### Q40 · A

**Why A is correct:** Subagents should get only what their task needs. Focused context cuts cost and keeps them on task.

**Closest distractor (C):** A larger context model fits the history, but subagents still drift to side topics.

### Q41 · C

**Why C is correct:** Only a PreToolUse hook can stop the transfer before it runs. It blocks transfer_funds until aml_check has passed.

**Closest distractor (A):** Freezing the account reacts after the unchecked transfer has already gone out.

### Q42 · B

**Why B is correct:** Per-file passes keep attention focused. The cross-file pass then finds issues that span files, such as the renamed function.

**Closest distractor (A):** Bigger batches still have edges, so issues across batches stay hidden.

### Q43 · D

**Why D is correct:** A good summary keeps what is still true and useful. It lists decisions, findings, open questions, and the changed files, without stale tool output.

**Closest distractor (A):** The full transcript brings the stale results back into the new session.

### Q44 · A

**Why A is correct:** "pause_turn" means a server-side tool loop paused and is not finished. The loop sends the response back as it is and calls the API again.

**Closest distractor (D):** "pause_turn" does not mean the output was too long, and dropping the paused response loses the work.

### Q45 · A

**Why A is correct:** In hub-and-spoke, the coordinator handles errors. It can retry, try other sources, or report the gap.

**Closest distractor (C):** Subagents do not call each other directly in a hub-and-spoke design.

### Q46 · B

**Why B is correct:** The coordinator starts subagents with the Task tool. Current SDK docs call it the Agent tool, and "Task" still works.

**Closest distractor (D):** fork_session branches a session, but it does not start a subagent.

### Q47 · D

**Why D is correct:** The gate checked that a call happened, not that it succeeded. It must require a verified customer ID that matches the refund's account.

**Closest distractor (B):** Human review of every refund would catch the error, but it is slow and costly when a better gate fixes the cause.

### Q48 · C

**Why C is correct:** PreToolUse runs before the email is sent. It can redirect external emails to a review draft every time.

**Closest distractor (A):** A prompt rule usually works, but it cannot guarantee the policy.

### Q49 · C

**Why C is correct:** The five checks are fixed and known. A fixed pipeline runs them all in the same order every night.

**Closest distractor (A):** The planner would still make its own plan, so a prompt list only makes skips less likely.

### Q50 · B

**Why B is correct:** Only "end_turn" means the task is finished. "max_tokens" and "refusal" mean the task is not done, so the loop should log them and handle the cause.

**Closest distractor (A):** Blind retries repeat the same truncation, and retrying a refusal does not fix it.

### Q51 · D

**Why D is correct:** A shared output structure in each delegation prompt makes results easy to merge. It fixes the problem at the source.

**Closest distractor (A):** A reformat agent adds cost and latency, and it can lose findings while it converts them.

### Q52 · A

**Why A is correct:** Claude can only recover if the error says what is missing. A clear message tells Claude to call get_customer first.

**Closest distractor (B):** A lower cap ends the waste sooner, but the refund still never happens.

### Q53 · D

**Why D is correct:** The coordinator should pick only the subagents a query needs. Simple questions can then skip most of the pipeline.

**Closest distractor (B):** These steps depend on each other, so running them in parallel does not work.

### Q54 · B

**Why B is correct:** After a big refactor, old tool results describe code that no longer exists. Claude may trust them and give wrong answers.

**Closest distractor (A):** A resumed session can read changed files, but the stale results remain in its history.

### Q55 · C

**Why C is correct:** Claude should choose tools from context, which adapts to each request. Code should enforce only the high-risk rule, which is the paid-order check.

**Closest distractor (D):** Model choice is right, but a prompt rule cannot guarantee the payment check.

### Q56 · A

**Why A is correct:** Tools in the AgentDefinition should follow least privilege. A subagent without Write, Edit, or Bash cannot delete files.

**Closest distractor (B):** A prompt rule lowers the risk, but it does not remove the ability to delete.

### Q57 · B

**Why B is correct:** The agent should handle each issue on its own. It solves what it can, escalates only the charge with a full handoff, and replies once.

**Closest distractor (A):** Escalating everything wastes human time on issues the agent can solve.

### Q58 · C

**Why C is correct:** In hub-and-spoke, the coordinator sees every input and output. Its logs show where the wrong statistic first appeared.

**Closest distractor (A):** The synthesis subagent may not know its source, and its answer could be wrong too.

### Q59 · D

**Why D is correct:** PostToolUse runs after the tool. By then the refund is already paid, so a PreToolUse hook must block it.

**Closest distractor (B):** PostToolUse can read and change tool results, so this reason is false.

### Q60 · A

**Why A is correct:** Adaptive decomposition changes the plan when new facts appear. The shared config blocks reliable tests, so it should come first.

**Closest distractor (D):** A separate agent splits the work, but the old plan keeps writing tests that fail at random.

### Q61 · C

**Why C is correct:** The coordinator should check the result against the requested areas. It then sends a targeted query to other sources and reruns synthesis.

**Closest distractor (B):** A full rerun with the same subtasks costs more and hits the same paywalls.

### Q62 · B

**Why B is correct:** The reviewer cannot check claims without the sources. The coordinator should pass the draft with the structured findings and their metadata.

**Closest distractor (D):** Marking claims "uncertain" is honest, but the reviewer still cannot do its real job.

### Q63 · D

**Why D is correct:** Sharing a balance with the wrong person is a security failure. A gate makes the identity check deterministic.

**Closest distractor (C):** Better prompts lower failures, but they never reach zero.

### Q64 · A

**Why A is correct:** Short structured findings save context and keep citations. The coordinator can then synthesize all key points.

**Closest distractor (B):** A larger context window fits more text, but key points still get lost in raw pages.

### Q65 · B

**Why B is correct:** The history is long and full of dead ends, which weakens answers. A fresh session with a structured summary keeps what matters.

**Closest distractor (D):** The dead ends stay in the history, and a prompt cannot reliably make Claude ignore them.
