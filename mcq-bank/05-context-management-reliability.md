# Domain 5 MCQ Bank: Context Management & Reliability

> 36 questions (9 Easy, 15 Medium, 12 Hard) covering task statements 5.1-5.6. [Study notes](../domains/05-context-management-reliability.md) · [Bank index](README.md)

## Questions

### Q1 · Task 5.2 · Medium

A support agent for Brightwell Home Insurance is checking why a claim is delayed. After two tool calls, it finds the cause: a missing photo upload that it can fix by sending a link. Before the agent replies, the customer writes: "Please just connect me with a human agent."

What should the agent do?

- A. Send the upload link first, then escalate if the customer still wants a human.
- B. Escalate to a human now, and include the claim findings in the handoff.
- C. Explain the quick fix and ask whether the customer still prefers a human.
- D. Finish the investigation so the human receives a complete case summary.

### Q2 · Task 5.1 · Medium

A team builds a returns chatbot for an online store on the Messages API. To save tokens, each request sends only the system prompt and the latest user message. Users report that the bot forgets the order number they gave two turns earlier.

What is the BEST fix?

- A. Add a system prompt line that tells Claude to remember earlier details.
- B. Turn on prompt caching so earlier turns stay available to the model.
- C. Store the order number on the server and expose it through a lookup tool.
- D. Send the full conversation history with every request to the API.

### Q3 · Task 5.3 · Easy

In a multi-agent research system, a subagent searches a patent database for a client's product idea. The query runs correctly and the database returns zero matching patents.

How should the subagent report this result?

- A. As a successful search with no matches, together with the query it ran.
- B. As an access failure, so the coordinator can schedule a retry later.
- C. As a generic "source unavailable" error to keep the report short.
- D. As a partial failure, and retry the same query with backoff before reporting.

### Q4 · Task 5.4 · Medium

A developer uses Claude Code to map how payments move through a 400-file monorepo. After ninety minutes, Claude's answers mention "the usual service layer." Earlier it named the exact classes PaymentRouter and LedgerSync. The `/context` view shows the window is far from full.

What is the MOST likely cause?

- A. The session reached its token limit and dropped the earliest messages.
- B. Claude Code reloaded CLAUDE.md, which replaced the earlier findings.
- C. Long discovery output in the session has degraded the useful context.
- D. The repository changed, so the earlier class names are no longer valid.

### Q5 · Task 5.5 · Hard

Finch Logistics extracts fields from shipping invoices. Overall field accuracy is 97% on 5,000 documents. Operations wants to stop human review next week for every field with model confidence above 0.90. A quick check shows customs declarations, 8% of volume, reach only 81% accuracy on the tariff code field. Nobody has ever compared the confidence scores with labeled outcomes.

What should the architect do FIRST?

- A. Approve the plan, but keep human review on every customs declaration for now.
- B. Raise the threshold to 0.97 so it matches the overall accuracy figure.
- C. Measure accuracy by document type and field, then calibrate the scores.
- D. Approve the plan and sample 5% of auto-approved items after launch.

### Q6 · Task 5.2 · Easy

A customer tells a mobile carrier's support agent: "This is the third time my bill is wrong, and I am furious." The problem is a duplicate fee. Standard policy lets the agent remove it.

What should the agent do?

- A. Acknowledge the frustration and offer to remove the duplicate fee now.
- B. Escalate to a human because the customer shows strong negative sentiment.
- C. Escalate because repeated billing problems always need a manager's review.
- D. Check its own confidence score and escalate if it is below 0.8.

### Q7 · Task 5.1 · Hard

An airline support agent summarizes older turns after 30 turns to stay within its budget. One customer has two open issues: a $412.80 refund for flight 5521 and baggage claim BG-7730 with status "in transit." Later in the chat, the agent offers "a partial refund" and cannot recall the baggage status. The team also notices that each booking lookup returns about 60 fields.

Which change BEST fixes the reported problem?

- A. Trim booking lookups to the needed fields with a PostToolUse hook.
- B. Summarize less often, for example after 60 turns instead of 30.
- C. Tell the summarizer to keep every number, date, and ID it finds in older turns.
- D. Keep a case facts block with one entry per issue, outside the summary.

### Q8 · Task 5.6 · Medium

A research system writes a market report from 12 claims. Nine claims are supported by several sources that agree. Three claims come from credible sources that disagree with each other. Stakeholders ask for a "clean" report.

What is the BEST way to present the findings?

- A. Publish only the nine agreed claims and drop the three disputed ones.
- B. Separate established findings from contested ones, citing sources for each.
- C. Keep all claims and use the value that most sources report for each.
- D. Keep all claims and add one overall confidence score at the top of the report.

### Q9 · Task 5.3 · Medium

A market research coordinator runs four subagents. The pricing subagent retried with backoff, then returned a structured error. It contains: failure type "timeout," the query it tried, prices for 14 of 20 retailers, and a suggested backup data feed. The other three subagents succeeded.

What is the BEST next step for the coordinator?

- A. Query the backup feed for the six missing retailers, then synthesize.
- B. Restart all four subagents so that all the data comes from one consistent run.
- C. Rerun the pricing subagent with the same query until it succeeds.
- D. Drop the pricing section and publish the other three sections only.

### Q10 · Task 5.4 · Hard

A team uses Claude Code to plan a migration of a large inventory system. Its exploration subagents run one after another, so the tech lead says they add no speed. The lead wants to remove them and let the main session read all files directly. The main session must keep design decisions accurate for a four-hour working session.

What is the BEST response to the lead?

- A. Remove them and move to a model with a larger context window instead.
- B. Keep them, but run them in parallel so they finally save some time.
- C. Keep them, but have them return full file contents for better accuracy.
- D. Keep them, because they keep verbose output out of the main context.

### Q11 · Task 5.2 · Hard

A hotel booking agent follows a written policy. Cancellations within 48 hours of check-in get no refund, except for weather-related hotel closures. A calm customer asks for a refund because a new government travel ban stops their trip tomorrow. The agent rates its own confidence at 0.92 that a refund is fair.

What should the agent do?

- A. Approve the refund, because its confidence is high and the case seems fair.
- B. Escalate to a human, because the policy does not address travel bans.
- C. Decline the refund, because the stay starts within the 48-hour window.
- D. Ask the customer for proof of the ban, then decide on the refund itself.

### Q12 · Task 5.1 · Easy

A research coordinator joins ten subagent reports into one long input for a synthesis step. Final outputs often miss findings from the reports placed in the middle.

Which change BEST helps?

- A. Add "read every section carefully, including the middle" to the synthesis prompt.
- B. Ask the synthesis agent to process the whole input twice.
- C. Put a key-findings summary at the top and use clear section headers.
- D. Move the most important reports into the middle of the input.

### Q13 · Task 5.5 · Medium

A healthcare billing extraction system processes 2,000 claims each day. Reviewers can check about 300 claims per day. Today, reviewers take claims in the order they arrive.

How should the review queue be ordered?

- A. Keep arrival order so every claim waits about the same amount of time.
- B. Split the 300 review slots evenly across all claim types.
- C. Pick 300 claims at random each day and review those.
- D. Put low-confidence claims and claims with conflicting sources first.

### Q14 · Task 5.3 · Hard

A nightly pipeline compares competitor prices for an online retailer. When the web fetch in the search subagent times out, the code returns an empty list with status "success" to keep the pipeline green. Last week the report said no competitor sold one product line cheaper, but sales found three that did. The dashboard showed 100% subagent success, and the coordinator prompt is also very long.

What is the BEST fix for the root cause?

- A. Return a structured timeout error with the query and any partial results.
- B. Make the coordinator treat every empty result as a failure and retry it once more.
- C. Stop the whole pipeline whenever any subagent reports a timeout.
- D. Shorten the coordinator prompt so it notices missing data more often.

### Q15 · Task 5.6 · Easy

Subagents in a legal research system send findings to a synthesis agent. The final memos often contain claims with no source.

What should each subagent finding include?

- A. The claim and a confidence score from 1 to 10.
- B. The claim, source URL or document name, excerpt, and date.
- C. A short summary of all the sources it reviewed for the topic.
- D. The claim and the exact search query that found it.

### Q16 · Task 5.2 · Medium

A utility company's support agent calls `lookup_account` with the name "J. Okafor." The tool returns two accounts in the same town. One account had a payment yesterday. A developer suggests the agent should choose the most recently active account.

Which instruction is BEST for this case?

- A. Choose the most recently active account, then confirm the balance with the customer.
- B. Show the customer both account summaries and ask which one is theirs.
- C. Ask for another identifier, such as the account number or service address.
- D. Escalate to a human agent whenever a lookup returns more than one match.

### Q17 · Task 5.4 · Easy

A developer plans a full-day Claude Code exploration of an unfamiliar route-planning codebase. They want class names and data flows to stay reliable late in the day.

Which practice should they set up FROM THE START?

- A. Have Claude record key findings in a scratchpad file as it goes.
- B. Ask Claude to repeat all earlier findings in every response.
- C. Wait until answers become vague, then start writing notes in a file.
- D. Load the entire codebase into context at the very start.

### Q18 · Task 5.1 · Medium

A claims agent calls `get_policy`, which returns about 70 fields, including the full rider history. Only six fields matter for claim decisions. Long sessions run out of context. A developer proposes a system prompt rule: "Ignore policy fields you do not need."

What is the BEST fix?

- A. Add the proposed rule so Claude ignores fields it does not need.
- B. Summarize the whole conversation every ten turns to free space.
- C. Keep full results so later questions can use any of the fields.
- D. Trim results to the six needed fields before they enter history.

### Q19 · Task 5.5 · Easy

An invoice extraction team now automates fields with calibrated confidence of 0.95 or higher. They want to keep checking that automated fields stay accurate over time.

Which sampling approach is BEST?

- A. Review only the fields that score just below the 0.95 automation threshold.
- B. Review only fields linked to later customer complaints.
- C. Review a stratified random sample that includes high-confidence fields.
- D. Review every field during the first week, then stop sampling.

### Q20 · Task 5.3 · Medium

A subagent checks suppliers against a sanctions list API. For supplier "Halvern Steel," the API returns a 503 server error. For "Norland Metals," the API returns a normal response with zero matches. The subagent reports both suppliers as "no sanctions found."

What is the BEST correction?

- A. Report both as failures, so the coordinator retries both checks.
- B. Report Halvern as an access failure and Norland as a valid empty result.
- C. Report both as valid results, and add a short note about the 503 server error.
- D. Retry both checks until each one returns at least one match.

### Q21 · Task 5.2 · Hard

A streaming service customer is angry about a double charge. The support agent acknowledges this and offers an immediate refund, which it can issue. The customer replies: "No. I want to talk to a person." Management tracks escalation rate closely, and the agent rates its confidence at 0.95 that it can fix the issue.

What should the agent do now?

- A. Escalate to a human now and pass the charge details in the handoff.
- B. Issue the refund anyway, because that fixes the actual problem.
- C. Offer the refund again with a clearer explanation of the timeline.
- D. Keep working the case, because confidence is high and escalations cost money.

### Q22 · Task 5.6 · Hard

An analysis subagent finds that a clinical trial enrolled 1,240 patients according to a public registry. A company press release from the same month says 1,180. The subagent's prompt says "resolve all conflicts before returning," so it keeps searching for more sources and does not finish. The coordinator needs the report today.

What is the BEST change to the subagent's prompt?

- A. Use the registry value, because official registries are more reliable.
- B. Average the conflicting values and report the range as a footnote.
- C. Stop and return an error to the coordinator when values conflict.
- D. Finish the analysis, mark the conflict with both sources, and return.

### Q23 · Task 5.5 · Hard

A legal contract review system extracts clauses into structured fields. Leadership wants to cut human review from 100% to 20% of contracts this quarter. The team has a labeled validation set of 1,500 contracts. The model now gives one confidence score per contract, and someone proposes automating every contract above 0.9.

Which plan is BEST?

- A. Use the contract-level score at 0.9, and sample 20% of the automated contracts.
- B. Keep full review until the model reports confidence above 0.99 everywhere.
- C. Get field-level scores, calibrate them on the labeled set, then set thresholds.
- D. Automate the highest-volume contract types first, then review the rest.

### Q24 · Task 5.1 · Hard

A due diligence system has five research subagents. Each one returns about 8,000 tokens of notes and reasoning. The synthesis agent has a small context budget and often drops dates and sources. One engineer proposes an extra step that summarizes each report. Another proposes a model with a larger context window.

What is the BEST fix?

- A. Add a summarizing step that shortens each report before synthesis.
- B. Make subagents return structured facts with citations, dates, and relevance.
- C. Move the synthesis step to a newer model with a much larger context window.
- D. Tell the synthesis agent in its prompt to focus on dates and sources.

### Q25 · Task 5.4 · Medium

A code migration agent built on the Agent SDK works for hours across 300 files. Runs are sometimes interrupted. The team adds a state manifest file that the agent loads when it restarts.

What should the manifest contain?

- A. The full transcript of the session up to the interruption.
- B. The raw output of every tool call, in the order it ran.
- C. Only the list of files that the agent has already modified.
- D. Explored paths, key findings, the current phase, and next steps.

### Q26 · Task 5.3 · Hard

An insurance coordinator processes 2,000 claims each night through three subagents: document reader, fraud check, and payout calculator. The fraud check service is down for one region, which affects 40 claims. The current code stops the whole batch on any error, so no claims paid out yesterday. A developer proposes marking the 40 claims as "fraud check passed" so the batch can finish.

What is the BEST design?

- A. Continue the batch, hold the 40 claims with a structured error, and report coverage.
- B. Mark the 40 claims as passed, then audit them by hand the next morning.
- C. Keep stopping the batch, but restart the whole batch automatically each hour.
- D. Skip the fraud check for that region and log a general warning message.

### Q27 · Task 5.2 · Medium

An internal IT helpdesk agent helps an employee with a VPN (virtual private network) login problem. It has tried three troubleshooting steps from the knowledge base. Each step failed with the same error. The employee is calm and has not asked for a person.

What should the agent do next?

- A. Keep trying further steps from the knowledge base until one works.
- B. Wait until the employee asks for a human before escalating.
- C. Escalate with the steps tried and their results, since progress has stalled.
- D. Escalate only if its self-reported confidence drops below a set score.

### Q28 · Task 5.5 · Medium

A receipt extraction system reaches 95% overall accuracy across eight fields, which meets the target. Field results on a labeled set show merchant name at 99% and total at 98%. The tax amount field reaches only 79%. The team wants to remove human review.

What should the team do?

- A. Automate all fields, because overall accuracy meets the 95% target.
- B. Automate high-accuracy fields with sampling, and keep review on tax amount.
- C. Keep full review on all fields until overall accuracy reaches 99%.
- D. Add a prompt instruction to take extra care with the tax amount field.

### Q29 · Task 5.1 · Medium

In one chat, a customer reports a missing item in order A-1001 ($34.99) and a wrong size in order A-1047 ($62.00). They also ask about a gift card balance. The agent's case facts block has a single "current order" field. That field is overwritten each time the customer mentions an order. The agent starts mixing up the two orders.

What is the BEST fix?

- A. Summarize each issue as one paragraph inside the facts block.
- B. Handle only one issue per chat and ask the customer to return later.
- C. Keep the latest order and ask the customer to repeat older details.
- D. Keep a separate entry per issue with order ID, amount, and status.

### Q30 · Task 5.6 · Hard

Logs show that research subagents return each claim with a source URL and date. Yet about 30% of claims in the final report have no source. The synthesis agent merges and rewrites similar claims from different subagents. The team proposes a new agent that searches the web for sources after the report is written.

What is the BEST fix?

- A. Require the synthesis agent to keep each claim's source links when merging.
- B. Add a citation agent that finds sources for uncited claims after writing.
- C. Ask the subagents to include more detail in their source citations.
- D. Stop merging and list each subagent's findings in separate sections.

### Q31 · Task 5.3 · Easy

A coordinator writes a competitive analysis report from eight sources. Two sources still failed after the subagents retried them. The coordinator finishes the report with the other six.

How should the report handle the gap?

- A. Leave out the failed sources so the report reads more cleanly.
- B. Add coverage notes that state which areas are limited and why.
- C. Fill the gap with the model's general knowledge of those areas.
- D. Delay the report until all eight sources respond successfully.

### Q32 · Task 5.6 · Medium

A research system formats its whole report as bullet lists. Finance readers cannot compare quarterly revenue across five companies. News items lose the context that explains why they matter. Engineers say the product specs section works well.

What is the BEST change?

- A. Render the full report as one large table for consistency.
- B. Render the full report as prose so it reads more naturally.
- C. Use tables for financial data, prose for news, and lists for specs.
- D. Keep the bullet lists and add a summary paragraph at the top.

### Q33 · Task 5.2 · Easy

A team launches a new customer support agent. They want it to make consistent choices between resolving a case and escalating it.

What is the BEST way to achieve this?

- A. Add explicit escalation criteria with few-shot examples to the system prompt.
- B. Have the agent escalate whenever its self-rated confidence is below 0.7.
- C. Have the agent escalate whenever the customer's sentiment score is negative.
- D. Let the agent judge each case freely, without any written escalation rules.

### Q34 · Task 5.1 · Easy

A bank's support agent summarizes older chat turns to save tokens. The summaries are short and read well.

Which information is MOST at risk of being lost in these summaries?

- A. The customer's greeting and general tone.
- B. The general topic of the conversation.
- C. The agent's earlier explanations of policy.
- D. Exact amounts, dates, and reference numbers.

### Q35 · Task 5.4 · Hard

A Claude Code coordinator explores a payments codebase in two phases. In Phase 1, a subagent mapped the shared auth library and wrote a summary. In Phase 2, six subagents each analyze one service, but each one re-reads the auth library. The runs are slow, and two describe token refresh differently. The lead suggests adding "the auth library is well known" to CLAUDE.md.

What is the BEST fix?

- A. Add the lead's note about the auth library to the project CLAUDE.md.
- B. Include the Phase 1 auth summary in each Phase 2 subagent's prompt.
- C. Let the Phase 2 subagents read the coordinator's full conversation history.
- D. Use one subagent for all six services so it reads the library only once.

### Q36 · Task 5.5 · Medium

A document extraction model gives each field a confidence score. The team routes fields above 0.9 to automation. A check against a labeled validation set shows fields scored 0.9 are correct only 72% of the time.

What should the team do?

- A. Ask the model in the prompt to give more honest confidence scores.
- B. Raise the threshold to 0.99 and keep the rest of the setup the same.
- C. Set thresholds from the labeled set, where measured accuracy meets the target.
- D. Stop using confidence scores and send every field to human review.

## Answer key

### Q1 · B

**Why B is correct:** An explicit request for a human is honored at once, without first trying to resolve the issue. The agent should still pass what it found, so the human does not start from zero.

**Closest distractor (C):** Offering the fix suits a customer who has not asked for a human. Here it only delays the request.

### Q2 · D

**Why D is correct:** The Messages API keeps no state between requests. Each request must include the full conversation history, or earlier details are gone.

**Closest distractor (B):** Prompt caching exists and helps when the same long prefix is resent, but it does not store conversation state for you.

### Q3 · A

**Why A is correct:** The query ran and found nothing, so the empty result is a valid answer. Reporting the query helps the coordinator trust and use it.

**Closest distractor (D):** Retrying a valid empty result wastes time; retries are for access failures such as timeouts.

### Q4 · C

**Why C is correct:** Vague "typical pattern" language in a long session is a sign of context degradation from verbose discovery output. It happens well before the window is full.

**Closest distractor (A):** The stem shows the window is far from full, so a token limit does not explain the vague answers.

### Q5 · C

**Why C is correct:** A high overall figure can hide weak segments, and the customs case proves one exists. The team must measure by document type and field, then calibrate scores on labeled data before setting any threshold.

**Closest distractor (A):** It fixes the one weak segment already found, but the scores are still uncalibrated and other weak segments are unknown.

### Q6 · A

**Why A is correct:** The issue is simple and within the agent's ability. The agent should acknowledge the frustration and offer the fix, and escalate only if the customer then asks for a human.

**Closest distractor (B):** Sentiment is not a reliable signal of case complexity, so anger alone is not an escalation trigger.

### Q7 · D

**Why D is correct:** Summaries lose exact amounts and statuses. A case facts block kept outside the summary, with one entry per issue, keeps the refund amount and claim status exact.

**Closest distractor (C):** The facts would still pass through a lossy summary step, and a prompt instruction does not guarantee they survive.

### Q8 · B

**Why B is correct:** Readers need to see which findings are solid and which are disputed. Keeping sources on both groups lets them judge the contested claims themselves.

**Closest distractor (C):** Choosing the majority value hides a real conflict between credible sources and removes the reader's ability to decide.

### Q9 · A

**Why A is correct:** The structured error gives the coordinator what it needs to recover: partial results and a suggested alternative. Using the backup feed for only the missing retailers fills the gap at low cost.

**Closest distractor (C):** The subagent already retried with backoff, so rerunning the same query ignores the better option the error provided.

### Q10 · D

**Why D is correct:** The main value of exploration subagents is a clean main context, not speed. They absorb verbose discovery output and return short summaries, which keeps long-session decisions accurate.

**Closest distractor (B):** Parallel runs may save time, but this treats speed as the purpose and misses why the subagents matter here.

### Q11 · B

**Why B is correct:** The policy covers weather closures but is silent on government travel bans. A policy gap is a valid escalation trigger.

**Closest distractor (C):** Applying the general 48-hour rule treats a gap as a clear policy violation, when the policy never addressed this case.

### Q12 · C

**Why C is correct:** Models reliably use the start and end of long inputs but may miss the middle. A summary at the top and clear headers fix the structure.

**Closest distractor (A):** Telling the model to pay attention does not change the position effect; the input structure must change.

### Q13 · D

**Why D is correct:** With limited reviewers, the most uncertain items should go first. Low confidence and conflicting sources are the best signals of likely errors.

**Closest distractor (C):** Random sampling is for measuring accuracy of automated items, not for using scarce reviewer time on the riskiest claims.

### Q14 · A

**Why A is correct:** Returning empty "success" after a timeout hides the failure and produces false conclusions. A structured error lets the coordinator see the gap and recover.

**Closest distractor (B):** It also retries valid empty results, and it still cannot tell a real "no data" from a failed fetch.

### Q15 · B

**Why B is correct:** Each finding needs a claim-source mapping: the claim, where it came from, the supporting excerpt, and the date. This lets synthesis keep attribution.

**Closest distractor (D):** The search query shows how the claim was found, not which document supports it.

### Q16 · C

**Why C is correct:** When a lookup returns several matches, the agent should ask for another identifier. Guessing with a heuristic risks acting on the wrong account.

**Closest distractor (A):** Picking the most recent account is a heuristic guess, and confirming the balance afterward can already expose the wrong customer's data.

### Q17 · A

**Why A is correct:** A scratchpad file keeps key findings outside the growing context. Starting it at the beginning protects facts before degradation sets in.

**Closest distractor (C):** Waiting for vague answers means some findings may already be lost or wrong.

### Q18 · D

**Why D is correct:** Trimming in the tool or a PostToolUse hook removes unused fields before they take up context. The saving happens on every call.

**Closest distractor (A):** A prompt rule does not remove the tokens; the 70 fields still fill the context.

### Q19 · C

**Why C is correct:** Stratified random sampling that includes high-confidence items shows whether automated fields stay accurate. It can catch new error patterns early.

**Closest distractor (A):** Sampling only near the threshold never checks the high-confidence items that are fully automated.

### Q20 · B

**Why B is correct:** A 503 means the check never ran, so it is an access failure that may need a retry. A normal response with zero matches is a valid empty result.

**Closest distractor (C):** Calling the 503 a valid result still tells the coordinator that Halvern was checked, which is false.

### Q21 · A

**Why A is correct:** The customer has now clearly asked for a person after hearing the offer. That request must be honored at once, with the case details in the handoff.

**Closest distractor (C):** The first offer was correct, but repeating it now ignores an explicit request for a person.

### Q22 · D

**Why D is correct:** The analysis agent should finish its work and mark the conflict with both values and sources. The coordinator then decides how to handle it.

**Closest distractor (A):** Choosing a source by its type hides a real conflict that the reader should see.

### Q23 · C

**Why C is correct:** Routing should use field-level scores calibrated on labeled data. Only then can the team set thresholds and reduce review safely.

**Closest distractor (A):** Sampling is good, but an uncalibrated contract-level score can hide weak fields inside contracts that score high.

### Q24 · B

**Why B is correct:** When downstream context is limited, upstream agents should return structured data, not long reasoning. Facts with citations, dates, and relevance keep what synthesis needs.

**Closest distractor (A):** A summary step adds cost and loses the same dates and sources it is meant to protect.

### Q25 · D

**Why D is correct:** A manifest stores the compact state needed to continue. It records what was explored, what was found, where the work stands, and what comes next.

**Closest distractor (C):** Modified files alone do not say what was learned or which step comes next.

### Q26 · A

**Why A is correct:** One failing component should not stop all claims, and failures must stay visible. Holding the 40 claims with a structured error lets the rest pay out safely.

**Closest distractor (C):** Restarting the whole batch still blocks every claim on a single regional failure.

### Q27 · C

**Why C is correct:** Inability to make meaningful progress is a valid escalation trigger. The handoff should list what was tried, so the human does not repeat it.

**Closest distractor (A):** More attempts after repeated identical failures waste the employee's time with little chance of success.

### Q28 · B

**Why B is correct:** Field-level results show which fields are safe to automate. Strong fields can be automated with ongoing sampling, while the weak tax field stays with humans.

**Closest distractor (C):** It is safe, but it blocks automation of fields that already meet the target and wastes reviewer time.

### Q29 · D

**Why D is correct:** Multi-issue chats need one structured entry per issue. Each order then keeps its own ID, amount, and status.

**Closest distractor (A):** Paragraph summaries can still blur amounts and statuses between the two orders.

### Q30 · A

**Why A is correct:** Logs show the sources arrive intact, so attribution is lost during synthesis. The synthesis prompt must keep claim-source links when it merges claims.

**Closest distractor (B):** A later search may attach a different source than the one that supported the claim.

### Q31 · B

**Why B is correct:** Coverage notes tell readers which areas rely on less data and why. The report stays useful and honest about its gaps.

**Closest distractor (D):** Waiting for every source can block the report for a long time when partial results are already useful.

### Q32 · C

**Why C is correct:** Each content type needs a format that fits it. Tables support number comparison, prose keeps news context, and lists suit specs.

**Closest distractor (A):** A single table helps the financial data but strips the context from the news items.

### Q33 · A

**Why A is correct:** Written criteria with examples of when to escalate and when to resolve give the agent clear, consistent rules.

**Closest distractor (B):** Self-reported confidence does not track real case complexity, so it is an unreliable trigger.

### Q34 · D

**Why D is correct:** Summaries tend to turn exact numbers and dates into vague phrases. That is why transactional facts belong in a separate facts block.

**Closest distractor (C):** Policy explanations can be restated later from the policy itself, but exact customer values cannot.

### Q35 · B

**Why B is correct:** Subagents do not inherit the coordinator's context. Passing the Phase 1 summary in each prompt removes repeated reading and gives every subagent the same facts.

**Closest distractor (D):** One subagent for six services would fill its own context and lose the benefit of isolated, focused work.

### Q36 · C

**Why C is correct:** Raw scores are not reliable until they are compared with labeled outcomes. Thresholds should sit where measured accuracy meets the target.

**Closest distractor (B):** A higher threshold chosen without measurement is still a guess on uncalibrated scores.
