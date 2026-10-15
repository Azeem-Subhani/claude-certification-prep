# Domain 4 MCQ Bank: Prompt Engineering & Structured Output

> 48 questions (12 Easy, 19 Medium, 17 Hard) covering task statements 4.1 to 4.6. [Study notes](../domain-4-prompt-engineering-structured-output/README.md) · [Anti-patterns](../domain-4-prompt-engineering-structured-output/anti-patterns.md) · [Bank index](README.md)

## Questions

### Q1 · Task 4.1 · Medium

Velora Health uses Claude to review pull requests for its billing service. The system prompt says "Report any issue that could matter." Developers say most findings are about naming and formatting. Real bugs get lost in the list.

Which prompt change is BEST?

- A. Add "Only report findings you are highly confident about" to the system prompt.
- B. List what to report, such as logic bugs and security flaws, and skip naming and style.
- C. Ask Claude to score each finding and hide every finding that scores below 0.8.
- D. Tell Claude to be conservative and keep each review to its five most important findings.

### Q2 · Task 4.3 · Easy

A logistics team moved its shipment extraction from "reply only with JSON" to tool_use with a JSON schema. A manager asks what kind of error this change removes.

Which error does the change remove?

- A. Values that are placed in the wrong field, such as the consignee in the shipper field.
- B. Line item weights that do not add up to the total weight on the document.
- C. Malformed JSON, such as missing brackets or extra text around the object.
- D. Invented values for fields that the source document does not contain.

### Q3 · Task 4.5 · Hard

Tallis Insurance runs three Claude workloads. Adjusters get a claim summary while the customer waits on the phone. A nightly re-scoring job uses the day's data, which is final at 11 p.m., and must be ready by 7 a.m. A monthly fraud scan of 60,000 closed claims is due five days after month end. In testing, most batches finished in under an hour.

Which plan is BEST for cutting cost?

- A. Batch only the monthly fraud scan; keep call summaries and nightly re-scoring synchronous.
- B. Batch the nightly re-scoring and the fraud scan, because test batches finished in under an hour.
- C. Batch all three workloads, and send call summaries as small batches to keep the wait short.
- D. Keep all three workloads synchronous, because batch results come back in a different order.

### Q4 · Task 4.2 · Medium

Brightdesk's support agent writes internal case notes with three parts: summary, steps tried, and next action. The prompt has three example notes, and most notes follow the layout. Notes for tickets with two or more separate problems often merge them into one summary.

What is the BEST change?

- A. Add a paragraph of rules that explains how to split the summary when a ticket has several problems.
- B. Add ten more example notes, picked at random from last month's tickets.
- C. Remove the examples and replace them with a detailed written layout specification.
- D. Add one or two example notes for multi-problem tickets that handle each problem separately.

### Q5 · Task 4.4 · Hard

A healthcare billing extractor pulls fields from scanned claim forms. Validation finds two failure types. In the first, the procedure total does not equal the sum of the line charges. In the second, the referring provider ID is blank because the form says "see attached referral letter," which is not in the input. The team plans to retry both types up to three times with the error message.

What is the BEST change to this plan?

- A. Retry both types, but raise the limit to five attempts for the provider ID failures.
- B. Skip retries for both types and send every failed form straight to human review.
- C. Retry only the total mismatch, and set the provider ID to null with a human follow-up.
- D. Retry only the provider ID failures, because math errors need code fixes, not model retries.

### Q6 · Task 4.6 · Easy

A team uses Claude to generate database migration scripts. They want a second Claude instance to review each script before it runs. A developer suggests giving the reviewer the full generation transcript "so it understands the choices."

What should the reviewer receive?

- A. The script and the requirements, without the conversation that produced the script.
- B. The script plus the full generation transcript, so it can follow the original reasoning.
- C. Only a short summary of the script that the generating session writes for it.
- D. The generation transcript only, so it can check the reasoning instead of the code.

### Q7 · Task 4.3 · Hard

A contract review pipeline extracts an auto_renewal field, defined as a required boolean. Some contracts say renewal is "subject to mutual agreement," and Claude returns true or false at random for these. Other contracts never mention renewal, and Claude returns false for them. Lawyers need to tell "no renewal" apart from both of these cases.

What is the BEST schema change?

- A. Keep the boolean, and add a prompt rule to choose false whenever the wording is uncertain.
- B. Keep the boolean required, and add a separate confidence score field next to it.
- C. Make the boolean optional, so Claude can leave it out when renewal is not mentioned.
- D. Use an enum of yes, no, and unclear, and allow null when renewal is not mentioned.

### Q8 · Task 4.1 · Easy

Kestrel Market uses Claude to flag product listings that may break seller policy. The prompt says "flag anything suspicious." Reviewers say most flags are listings with strong sales language, which the policy allows.

Which prompt change is BEST?

- A. Replace "suspicious" with "clearly suspicious" and ask Claude to be careful.
- B. Name the policy violations to flag, and state that strong sales language is allowed.
- C. Ask Claude for a confidence score and drop flags below a fixed threshold.
- D. Tell Claude to flag a listing only when it is sure a reviewer would agree.

### Q9 · Task 4.2 · Hard

Northgate Freight extracts the delivery deadline from carrier contracts. It works well when the deadline is in a terms table. When the deadline is stated inside a paragraph, the field often comes back null, even though the date is there. The prompt has three few-shot examples, all from table-based contracts. A teammate proposes making the field required.

What is the BEST fix?

- A. Add examples where the deadline appears inside a paragraph, next to the table examples.
- B. Make the field required, so Claude must always return a deadline for each contract.
- C. Add an instruction to read the whole contract carefully before it returns null.
- D. Split each contract into sections and run the extraction on each section separately.

### Q10 · Task 4.4 · Medium

To save tokens, Pellet Freight's retry step sends only the failed JSON and the validator error. The error reads "weight_kg must be a number, got string." Retries return a number. However, the number often does not match the bill of lading.

What is the BEST change?

- A. Add a prompt line that tells Claude to double-check the converted number before it answers.
- B. Change weight_kg to a string type, so the validator accepts the original text.
- C. Raise the retry limit, so Claude has more chances to reach the right value.
- D. Include the original bill of lading in the retry, with the failed output and error.

### Q11 · Task 4.5 · Easy

A team plans its first Message Batches job for 40,000 product descriptions. A developer asks how they will know which result belongs to which product.

How should the team match each result to its product?

- A. Read the results in the same order the requests were submitted.
- B. Use the batch ID, which the API gives to each request in the batch.
- C. Set a custom_id on each request, and match each result by that value.
- D. Submit one batch per product, so each batch holds a single result.

### Q12 · Task 4.6 · Hard

A CI review agent checks PRs in a payments monorepo. A 25-file PR changes a currency rounding helper and its callers in several services. The single-pass review gave detailed comments on the first files and thin comments on later ones. It also missed that one caller still uses the old rounding signature. The team proposes a model with a larger context window.

What is the BEST change?

- A. Switch to the larger context window model, so all 25 files fit with room to spare.
- B. Review each file in its own pass, then run a separate pass on cross-file data flow.
- C. Review each file in its own pass, then merge the per-file comments into one report.
- D. Keep one pass, but tell Claude to give equal attention to every file in the PR.

### Q13 · Task 4.3 · Medium

A support assistant has a lookup_order tool. It answers general policy questions from its system prompt. The team wants Claude to call the tool only when an order is involved and to reply in plain text otherwise. A teammate says "any" means Claude may use a tool or skip it. The model in use supports all tool_choice options.

Which tool_choice setting fits BEST?

- A. "auto"
- B. "any"
- C. {"type": "tool", "name": "lookup_order"}
- D. "none"

### Q14 · Task 4.1 · Hard

A legal team uses Claude to flag risky clauses in vendor contracts. The prompt says "flag clauses that create significant risk." Lawyers dismiss most flags, and most of these are standard indemnity terms the firm accepts. A data scientist proposes adding a confidence score per flag and tuning a cutoff on 500 labeled contracts.

What should the architect do FIRST?

- A. Build the confidence score and tune the cutoff, since a labeled set makes it calibrated.
- B. Add "only flag clauses you are highly confident are risky" to the system prompt.
- C. Define which clause types to flag, and list the standard terms the firm accepts.
- D. Add a second Claude pass that re-checks each flag and removes the weak ones.

### Q15 · Task 4.2 · Easy

Claude labels support tickets as "billing" or "technical." The written rules for each label are detailed. Borderline tickets, such as a wrong charge caused by a software bug, get different labels on different days.

Which technique BEST improves consistency?

- A. Add a confidence threshold and send low-scoring tickets to a human.
- B. Write longer definitions for both labels with more detail on edge cases.
- C. Tell Claude to pick the first label that applies to the ticket.
- D. Add a few borderline examples that explain why one label was chosen.

### Q16 · Task 4.4 · Easy

An invoice extraction pipeline validates each result and retries failures with the specific error.

Which failure is a retry MOST likely to fix?

- A. The invoice number is missing because the page that held it was not scanned.
- B. The tax amount was placed in the discount field of the output.
- C. The payment terms are in a separate contract that was not provided.
- D. The order date is printed on the invoice, but it is fully smudged.

### Q17 · Task 4.5 · Medium

A compliance team submits a quarterly batch of 20,000 contract reviews. Their plan is to leave the results in the API and pull them when auditors ask. Auditors usually ask six to eight weeks later.

What is the main problem with this plan?

- A. Batches with more than 10,000 requests are split, and the split results lose their custom_id values.
- B. Batch results can be read only once, so a second auditor request will fail.
- C. Quarterly jobs of this size are too large for batch and need the synchronous API.
- D. Batch results are kept for only 29 days, so the team must store them itself.

### Q18 · Task 4.6 · Medium

A team uses Claude to write release notes from merged PRs. They want a second check for wrong claims before publishing. A developer proposes forking the generation session, so the reviewer "already knows the context."

What is the BEST approach?

- A. Fork the session, because the reviewer can then see exactly why each claim was written.
- B. Fork the session, and tell the reviewer to be critical of every claim it sees.
- C. Use a fresh instance with the notes and the PR list, but not the generation history.
- D. Use the same session, and ask it to list three possible errors before it approves.

### Q19 · Task 4.3 · Hard

Marlo Payroll moved its timesheet extraction to tools with strict: true. A lead proposes deleting the Pydantic validators, since inputs now match the schema. The validators check that end_date is after start_date and that daily hours sum to the weekly total. A small number of responses also stop with stop_reason max_tokens.

What is the BEST decision?

- A. Delete the validators, because strict mode makes every response match the schema.
- B. Keep the validators for cross-field rules, and treat max_tokens stops as incomplete.
- C. Delete the validators, and add prompt rules about date order and weekly totals instead.
- D. Turn off strict mode and rely on the validators alone, since they catch every error type.

### Q20 · Task 4.2 · Medium

A help desk agent has two tools: search_kb and create_ticket. The team already rewrote both tool descriptions and tested them. Messages like "my export is still broken" still sometimes create duplicate tickets when a search would have answered them. The model in use supports all tool_choice options.

What is the BEST next step?

- A. Add examples of unclear messages that show which tool was chosen and why.
- B. Rewrite both tool descriptions again, with more detail on when to use each.
- C. Set tool_choice to "any," so Claude must call one of the two tools on every message.
- D. Merge the two tools into one tool that searches first and then creates a ticket.

### Q21 · Task 4.1 · Medium

Fernway's CI security reviewer flags every database query as "possible SQL injection." Most flagged queries already use bound parameters. The prompt says "Report possible injection risks."

Which change BEST improves precision?

- A. Add "be conservative and avoid false alarms" to the injection section of the prompt.
- B. Report injection findings only when Claude rates its own confidence above 0.9.
- C. Add a second Claude pass that re-checks each injection finding before it is posted.
- D. Report a query only when user input is joined into the SQL string.

### Q22 · Task 4.4 · Hard

A contract extractor pulls the effective date. In some contracts, the cover page and the signature page show different dates. Retries switch between the two dates from one attempt to the next. The legal team has not set a rule for which page wins.

What is the BEST design?

- A. Extract both dates with their page locations, and set a conflict_detected flag for review.
- B. Retry with the validator error until two attempts in a row return the same date.
- C. Add a prompt rule that always uses the date shown on the signature page.
- D. Make the date field nullable, so Claude returns null when it is not sure.

### Q23 · Task 4.5 · Hard

A weekly job writes copy for 50,000 product pages, due three days after it starts. For each product, Claude now calls a client-side get_stock tool, waits for the result, and then writes. The stock data is already in a database before the job starts. Finance wants the 50% batch discount.

What is the BEST plan?

- A. Submit the job as one batch as it is; the batch runs the get_stock loop for each request.
- B. Fetch stock data in code, add it to each request, and submit one batch.
- C. Keep the job on the synchronous API, because batch requests cannot include any tools.
- D. Submit one batch per tool round trip, and wait for each batch before sending the next.

### Q24 · Task 4.3 · Easy

An expense tool sorts receipts into an enum: travel, meals, software, or equipment. New kinds of expenses, such as conference fees, get forced into the wrong category.

What is the BEST schema change?

- A. Remove the enum and let Claude write any category name as free text.
- B. Add a prompt rule that tells Claude to choose the closest existing category.
- C. Add an "other" value, plus a detail string field for the actual expense type.
- D. Make the category field optional, so Claude can leave it blank when unsure.

### Q25 · Task 4.6 · Hard

Claude writes reconciliation scripts that post journal entries to the production ledger. A separate, independent Claude instance reviews each script before it runs. A manager wants to cut review cost by 30%. Last quarter, two of the three errors caught were in scripts under 50 lines.

What should the architect recommend?

- A. Replace the independent review with a self-check in the generating session.
- B. Keep the independent review only for scripts over 200 lines, which carry more risk.
- C. Replace the independent review with extended thinking during generation.
- D. Keep the independent review for every script, since errors reach the live ledger.

### Q26 · Task 4.2 · Hard

A claims triage prompt has six examples, each showing only the claim text and its label. Claude now labels any claim that mentions water as "flood," including burst pipes. Burst pipes belong in "plumbing." The team is also worried that the prompt keeps getting longer.

What is the BEST fix?

- A. Add about 20 more examples, so every kind of water damage has its own example.
- B. Add one rule to the prompt: "a burst pipe is never a flood claim, even with water damage."
- C. Rework the examples to show short reasoning, with a burst-pipe and a flood case.
- D. Add a confidence score, and send low-confidence water claims to an adjuster.

### Q27 · Task 4.1 · Easy

Claude summarizes recorded customer calls for team leads. The prompt asks it to "highlight important issues." Team leads say the highlights vary widely from call to call.

Which prompt change is BEST?

- A. Define which issue types count as important, such as cancellation threats.
- B. Ask Claude to highlight only issues it is highly confident are important.
- C. Ask Claude to highlight at most three issues per call to keep the focus.
- D. Tell Claude to think about what a manager would find most important.

### Q28 · Task 4.4 · Medium

Rally Freight uses tool_use to extract shipping labels. The JSON is always valid. However, Claude sometimes swaps the shipper and consignee addresses. The shipper must always be one of the customer's registered warehouses.

What is the BEST fix?

- A. Add regex patterns to both address fields so each must look like a valid address.
- B. Check the shipper against registered warehouses in code, and retry with that error.
- C. Switch from tool_use to a text reply, so Claude can explain which address is which.
- D. Split each address into required street, city, and postal code fields, so Claude fills every part.

### Q29 · Task 4.3 · Medium

A supplier onboarding pipeline extracts a country field as free text. Results include "USA," "United States," and "U.S.," which breaks downstream reports. The reports read each record seconds after extraction.

What is the BEST fix?

- A. Leave the field as free text, and map the values to one spelling in a nightly script.
- B. Ask Claude to copy the country name exactly as it is written in each supplier document.
- C. Make the field optional, so unclear country names are simply left blank.
- D. Add a format rule for two-letter country codes, such as US, and use an enum in the schema.

### Q30 · Task 4.5 · Medium

A team submits a nightly batch and starts a downstream reporting job two hours later. In testing, every batch finished within one hour. In production, the report sometimes runs on missing results.

What is the BEST change?

- A. Start the downstream job when the batch ends, and plan for up to 24 hours.
- B. Keep the two-hour start, since most batches finish within one hour.
- C. Split the batch into smaller batches, so each one finishes within two hours.
- D. Set a priority flag on the batch, so it finishes inside the two-hour window.

### Q31 · Task 4.2 · Easy

A code review prompt asks for findings with four fields: location, issue, severity, and suggested fix. Findings come back with fields missing or in a different order. The team wants to add few-shot examples.

Which set of examples is BEST?

- A. One example that shows the simplest possible finding.
- B. Two to four examples with all four fields, including a tricky case.
- C. Thirty examples that cover every file type in the repository.
- D. No examples, just a longer prose description of each field.

### Q32 · Task 4.6 · Medium

A CI review pipeline runs independent per-file passes and an integration pass. Each finding has a confidence score. Last month, the team calibrated the scores against 400 labeled findings.

What should happen to findings with LOW confidence?

- A. Drop them before posting, since low-confidence findings are mostly false positives.
- B. Post every finding on the PR with its score, so developers can decide for themselves.
- C. Route them to a human reviewer, and post only high-confidence findings on the PR.
- D. Re-run the same pass on low-confidence findings until their scores rise above the cutoff.

### Q33 · Task 4.1 · Hard

A medical coding assistant flags documentation gaps in clinician notes. The prompt says "flag notes that lack sufficient detail." Coders dismiss most flags as brief but complete notes. At the same time, real gaps, such as injuries with no body side recorded, are often missed.

Which change BEST fixes both problems?

- A. Add a confidence score and suppress flags below 0.85 to cut the dismissed ones.
- B. Add five examples of complete brief notes, so Claude learns what a good note looks like.
- C. Move to a larger model that can judge clinical detail more accurately.
- D. List the specific gaps to report, such as missing body side, and what to skip.

### Q34 · Task 4.4 · Easy

A review bot adds a detected_pattern field to each finding, such as "unchecked-null-return." Developers can dismiss findings on the PR.

What is the MAIN value of this field?

- A. It lets Claude skip patterns that it has already reported earlier in the same file.
- B. It makes the JSON output valid, because each finding then has a fixed, typed category.
- C. It shows which patterns developers dismiss, so prompt fixes can target them.
- D. It raises Claude's confidence score for findings that match a pattern seen before.

### Q35 · Task 4.3 · Hard

An invoice queue sometimes contains spam and unrelated letters. With tool_choice "auto," Claude sometimes replies in text, such as "This is not an invoice." The pipeline drops these replies without a record. The team needs structured output every time and must still know which documents are not invoices. The model in use supports all tool_choice options.

What is the BEST design?

- A. Force the tool, add not_an_invoice to a document_type enum, and make other fields nullable.
- B. Force the extraction tool, and keep every field required so each response is complete and usable.
- C. Keep tool_choice "auto," and parse the text replies with a separate regex classifier.
- D. Force the extraction tool, and add a prompt rule to write "N/A" in every field for non-invoices.

### Q36 · Task 4.2 · Medium

In a multi-agent research system, a subagent extracts citations from sources. It works well on papers with a bibliography. It misses citations written inline in news-style reports. The prompt already says "citations may appear inline in the text."

What is the BEST change?

- A. Expand the instruction into a full paragraph about every inline citation style.
- B. Add one or two examples that show citations extracted from inline text.
- C. Make the citation list required, so the subagent must always return entries.
- D. Create a second subagent that handles only inline-citation documents.

### Q37 · Task 4.5 · Easy

A team asks why they should move their weekly audit job to the Message Batches API.

Which statement about the Batches API is correct?

- A. Each request is guaranteed to finish within one hour of submission.
- B. Requests in one batch share context, which reduces the tokens used.
- C. Batch requests can run longer client-side tool loops than normal requests.
- D. Processing costs 50% less than the same calls on the standard API.

### Q38 · Task 4.6 · Hard

A code review pipeline already uses independent per-file passes for large PRs. The reports now contain contradictions. The pass on file A says a shared helper validates input. The pass on file B flags missing validation on the same call path. A teammate notes that the per-file passes use a smaller model to save cost.

What is the BEST change?

- A. Add a cross-file integration pass that checks data flow between the changed files.
- B. Go back to one pass over all files, so Claude sees every file together.
- C. Run each per-file pass twice, and keep only findings that appear both times.
- D. Switch the per-file passes to a larger model with a bigger context window.

### Q39 · Task 4.1 · Medium

A team's codebase wraps all errors in a custom Result type, and this is documented. The review bot keeps flagging the pattern as "non-standard error handling." Developers now skim past all convention findings.

Which change is BEST?

- A. Ask Claude to report only the convention findings that it is highly confident about.
- B. Add a confidence cutoff, so convention findings below 0.7 are hidden.
- C. Add the Result wrapper to the criteria as an accepted pattern to skip.
- D. Remove every pattern and convention finding from the review scope.

### Q40 · Task 4.4 · Hard

An extraction pipeline has three problems. About 1% of text-mode replies are malformed JSON. Some outputs put the due date in the issue_date field. Some PO (purchase order) numbers are missing because the documents do not contain them. The team wants one plan that fixes each problem with the right tool.

Which plan is BEST?

- A. Retry all three with the specific error, since retries with feedback fix most failures.
- B. Use tool_use for the malformed JSON, and make every field required to stop missing values.
- C. Use tool_use for syntax, retry misplaced dates with the error, and retry absent POs with a stronger prompt.
- D. Use tool_use for syntax, retry misplaced dates with the error, and set absent POs to null.

### Q41 · Task 4.3 · Medium

A developer defines an extract_invoice tool and sends a request. The code reads the text of the first content block, but it gets a short sentence or nothing. The developer expected the JSON fields.

Where should the code read the extracted data?

- A. From the text block, because Claude repeats the tool arguments there as JSON.
- B. From the content block of type tool_use, by reading its input field.
- C. From a second request that asks Claude to return the tool arguments as text.
- D. From the tool_result block that the API adds after the tool call.

### Q42 · Task 4.2 · Hard

A parcel pipeline extracts package weight into weight_kg (nullable) and weight_is_estimate (boolean). Notes often say "a bit over 2 kg" or "about 3 lbs." Claude sometimes returns exact numbers with weight_is_estimate set to false. When no weight is mentioned, it sometimes invents one. A written rule, "never invent weights," did not help.

What is the BEST next step?

- A. Make weight_kg required, so Claude must always commit to a single number.
- B. Make the written rule stronger with capital letters and repeat it at the end.
- C. Add examples that map vague amounts to an estimate flag and missing weights to null.
- D. Retry each extraction with the message "weight may be invented" until it changes.

### Q43 · Task 4.5 · Hard

Brennan Legal reviews 340,000 discovery documents each week. The results are needed within three days, and cost matters. A developer plans one batch for all documents. Another suggests putting ten documents into each request to reduce the request count.

What is the BEST plan?

- A. Use several batches within the size limits, with a custom_id for each document.
- B. Submit one batch for all 340,000 documents, because the API splits large batches on its own.
- C. Use the synchronous API, because a weekly job of this size cannot use batches.
- D. Put ten documents into each request, so the full job fits inside one batch.

### Q44 · Task 4.6 · Easy

A developer asks Claude to review code that Claude wrote earlier in the same session. The review finds few problems, but later testing finds several.

What is the MOST likely reason the self-review was weak?

- A. The session ran out of context space before the review started.
- B. The model cannot read code that it produced earlier in the session.
- C. Review requests are ignored when they follow a generation step.
- D. The session keeps its own reasoning, so it rarely questions its choices.

### Q45 · Task 4.1 · Medium

A test-gap reviewer has the instruction "flag missing tests." It flags every untested getter, setter, and generated file. Developers stop reading its output.

Which instruction is BEST?

- A. "Flag only the missing tests that matter most to the team and the product."
- B. "Report untested business logic; skip getters, setters, and generated files."
- C. "Flag missing tests only when your confidence is above 0.8, and skip the rest."
- D. "Report no more than three missing tests in each PR, ranked by file size."

### Q46 · Task 4.4 · Medium

A hospital discharge extractor retries failed validations with the original document, the failed output, and the specific error. Most failures pass on the first retry. A few documents still fail the same check after two retries.

What should the pipeline do with these documents?

- A. Stop at the retry limit, and route the document to human review with the errors.
- B. Keep retrying until the output passes, since each retry includes the error.
- C. Accept the last output, and turn off the failing check for that document.
- D. Restart with the original request and no error, to avoid bias from earlier tries.

### Q47 · Task 4.3 · Medium

A procurement team tags supplier emails as positive, negative, or neutral. Mixed or sarcastic emails get forced into one label. Analysts want to see which emails are truly hard to read.

What is the BEST schema change?

- A. Add a prompt rule that says to choose neutral when the tone is hard to read.
- B. Make the field optional, so Claude can skip the emails that are hard to label.
- C. Add an "unclear" value to the enum for emails with mixed or ambiguous tone.
- D. Add an "other" value with a free-text field that describes the tone.

### Q48 · Task 4.2 · Medium

A coverage agent is asked to "find test coverage gaps." It reports only files with low overall coverage. It misses untested else branches inside well-covered files. The prompt already explains what a branch-level gap is.

What is the BEST change?

- A. Expand the explanation of branch-level gaps into a longer, more detailed paragraph.
- B. Give Claude the coverage report, and ask it to list every file under 80%.
- C. Ask Claude to read every test file in full before it reports any coverage gaps.
- D. Add two examples that show an untested else branch reported as a gap.

## Answer key

### Q1 · B

**Why B is correct:** Explicit report and skip categories tell Claude what counts as a finding. This removes the naming and style noise at its source. Vague wording does not change what Claude treats as an issue.

**Closest distractor (A):** "Only report high-confidence findings" is a general instruction that does not improve precision like specific categories do.

### Q2 · C

**Why C is correct:** A tool with a JSON schema gives structured input that is always parseable. This removes syntax errors such as missing brackets or extra text. It does not check what the values mean.

**Closest distractor (D):** Invented values are a semantic problem, fixed by nullable fields, not by the move to tool_use.

### Q3 · A

**Why A is correct:** The call summary blocks a live customer, so it must stay synchronous. The nightly job has an eight-hour window, but a batch can take up to 24 hours. Only the five-day fraud scan fits the batch window.

**Closest distractor (B):** Fast test batches are not a guarantee, and the nightly window is shorter than the 24-hour processing limit.

### Q4 · D

**Why D is correct:** The existing examples work for simple tickets. The failures come from a case the examples do not show. One or two targeted examples for multi-problem tickets cover the gap.

**Closest distractor (A):** Adding more written rules is the anti-pattern when examples are the tool that fixes format consistency.

### Q5 · C

**Why C is correct:** A math mismatch is a fixable error, so a retry with the specific error can help. The provider ID is in a document that was not provided. No retry can create it, so null and human follow-up is right.

**Closest distractor (B):** Sending total mismatches to humans wastes effort, because a retry with error feedback often fixes them.

### Q6 · A

**Why A is correct:** An independent reviewer works best without the generator's reasoning. It then judges the script against the requirements with fresh eyes. The transcript would pull it toward the same decisions.

**Closest distractor (B):** Sharing the full transcript recreates the self-review problem the second instance is meant to avoid.

### Q7 · D

**Why D is correct:** An "unclear" enum value captures truly ambiguous wording. A nullable value captures absence without inventing an answer. This keeps "no renewal" separate from both other cases.

**Closest distractor (C):** Making the field optional fixes the absent case but still forces ambiguous contracts into true or false.

### Q8 · B

**Why B is correct:** Naming the violations and the allowed case gives Claude explicit criteria. This directly removes flags on allowed sales language. Vague words like "suspicious" leave the decision open.

**Closest distractor (C):** Confidence thresholds are an anti-pattern for false positives when the criteria themselves are unclear.

### Q9 · A

**Why A is correct:** The examples only show table layouts, so Claude misses paragraph-style deadlines. Adding examples from varied layouts teaches it where else to look. The data exists, so this targets the real gap.

**Closest distractor (B):** A required field pushes Claude to invent a deadline when one is truly absent, which adds a new error.

### Q10 · D

**Why D is correct:** A retry needs the original document, the failed output, and the specific error. Without the source, Claude can only guess a number. With the bill of lading, it can find the real weight and convert it.

**Closest distractor (C):** More retries without the source document only give more chances to guess.

### Q11 · C

**Why C is correct:** Batch results can come back in any order. A custom_id on each request is the field used to match results to inputs.

**Closest distractor (A):** Result order is not guaranteed to match submission order.

### Q12 · B

**Why B is correct:** Per-file passes fix the uneven attention across 25 files. The missed signature problem is a cross-file issue, so it needs its own integration pass. Both parts are needed.

**Closest distractor (C):** Per-file passes alone fix the thin comments but still miss issues that cross file boundaries.

### Q13 · A

**Why A is correct:** "auto" lets Claude decide whether to call a tool or reply in text. That matches a mix of order questions and general questions.

**Closest distractor (B):** "any" forces Claude to call some tool, so it could not answer policy questions in plain text.

### Q14 · C

**Why C is correct:** The root cause is a vague criterion, "significant risk." Clear clause types and an accepted-terms list fix that first. Confidence routing is useful only after clear criteria exist.

**Closest distractor (A):** A calibrated cutoff on top of vague criteria filters noise without fixing what Claude is told to flag.

### Q15 · D

**Why D is correct:** Few-shot examples that show reasoning for unclear cases make judgments consistent. The written rules already exist and did not solve the problem.

**Closest distractor (B):** Adding more detailed instructions is the anti-pattern when instructions have already failed.

### Q16 · B

**Why B is correct:** A value in the wrong field is a structure error, and the right value exists in the document. A retry with the specific error can fix it.

**Closest distractor (D):** The date is printed, but it cannot be read, so a retry cannot recover it.

### Q17 · D

**Why D is correct:** Batch results stay available for 29 days. Auditors ask after six to eight weeks, so the team must download and store results itself.

**Closest distractor (C):** A 20,000-request job is well within the batch limit of 100,000 requests.

### Q18 · C

**Why C is correct:** A fork copies the generation history, so the reviewer shares the same reasoning. A fresh instance with the notes and the source list checks claims independently.

**Closest distractor (B):** Telling a forked reviewer to be critical does not remove the shared reasoning context.

### Q19 · B

**Why B is correct:** Strict schemas stop syntax errors but not cross-field errors like date order or wrong sums. A max_tokens stop means the output may be cut off. Code validation is still needed for both.

**Closest distractor (A):** Strict mode matches the schema, but it cannot check meaning, and cut-off outputs are an exception.

### Q20 · A

**Why A is correct:** The descriptions are already improved, so the next step is examples. Examples with reasoning show how to handle unclear requests. Claude can then apply that judgment to new messages.

**Closest distractor (B):** Better descriptions come first, but the team already did that, so more rewriting adds little.

### Q21 · D

**Why D is correct:** The criterion names the exact condition that makes a query risky. Queries with bound parameters do not match it, so they are skipped. This removes the false positives without hiding real injection risks.

**Closest distractor (C):** A second pass adds cost but still uses the same vague rule, so the false alarms remain.

### Q22 · A

**Why A is correct:** The source contradicts itself, so no retry can find one right answer. Extracting both values with a conflict flag keeps the facts. A human can then decide.

**Closest distractor (C):** Picking a page is a business decision the legal team has not made, so the extractor should not invent one.

### Q23 · B

**Why B is correct:** A batch request cannot run a client-side tool loop, because each round trip needs a new request. The stock data is known in advance, so code can include it in each request. The job then fits one batch.

**Closest distractor (D):** Chaining batches works in theory, but each round can take up to 24 hours and risks the three-day deadline.

### Q24 · C

**Why C is correct:** An "other" value with a detail string keeps the category list stable. It still records new expense types clearly.

**Closest distractor (A):** Free text loses the consistency that an enum gives to downstream systems.

### Q25 · D

**Why D is correct:** The scripts change production financial data, so review quality matters more than cost. Last quarter's errors also show short scripts are not safe to skip.

**Closest distractor (B):** A size cut-off looks like a fair trade-off, but the data shows short scripts carried most of the caught errors.

### Q26 · C

**Why C is correct:** Input-output examples without reasoning lead Claude to copy surface features. A few examples with reasons, including a contrasting pair, teach the real rule. This also keeps the prompt short.

**Closest distractor (B):** A single rule fixes burst pipes but not the habit of matching surface words in other cases.

### Q27 · A

**Why A is correct:** "Important" is vague, so Claude decides it differently each time. Naming the issue types gives explicit criteria and consistent highlights.

**Closest distractor (C):** A cap on the number of highlights does not tell Claude which issues to pick.

### Q28 · B

**Why B is correct:** Swapped addresses are a semantic error, which a schema cannot catch. A code check against a known rule detects it. A retry with that specific failure helps Claude correct it.

**Closest distractor (A):** Schema patterns check format, and both addresses already have a valid format.

### Q29 · D

**Why D is correct:** A format rule in the prompt, next to a schema constraint, gives the same value every time. This is the standard way to handle inconsistent source formats.

**Closest distractor (A):** A nightly script runs too late, because the reports read each record right after extraction.

### Q30 · A

**Why A is correct:** The Batches API has no latency guarantee and allows up to 24 hours. The downstream job should wait for the batch to end, not a fixed time.

**Closest distractor (C):** Smaller batches may finish faster, but they still have no time guarantee.

### Q31 · B

**Why B is correct:** Two to four targeted examples that show the exact fields give a consistent format. A tricky case shows how to handle edge cases.

**Closest distractor (A):** One simple example does not show how to handle harder findings.

### Q32 · C

**Why C is correct:** The scores are calibrated, so they can drive routing. High-confidence findings go to developers. Low-confidence findings go to a human reviewer, not into the trash.

**Closest distractor (B):** Posting everything puts low-confidence noise in front of developers and hurts trust.

### Q33 · D

**Why D is correct:** Both symptoms come from one vague criterion, "sufficient detail." Naming the gaps to report and the cases to skip fixes both. It cuts the false flags and catches the misses.

**Closest distractor (B):** Examples of good notes help a little, but they do not define which gaps must be reported.

### Q34 · C

**Why C is correct:** A detected_pattern field lets the team group dismissals by pattern. That shows which code constructs cause false positives. The team can then improve the prompt for those patterns.

**Closest distractor (A):** The field supports analysis by the team, not skipping repeat findings inside one review.

### Q35 · A

**Why A is correct:** Forcing the tool guarantees structured output. A document_type value for non-invoices records the case in data, not text. Nullable fields stop Claude from inventing invoice values.

**Closest distractor (D):** "N/A" strings in every field are placeholder values that downstream code can mistake for real data.

### Q36 · B

**Why B is correct:** The written instruction already exists and did not work. Examples that show extraction from inline text teach Claude to handle that structure.

**Closest distractor (A):** A longer instruction repeats the approach that already failed.

### Q37 · D

**Why D is correct:** The Batches API charges 50% of standard prices. Speed is not guaranteed, and the main benefit is lower cost for non-urgent work.

**Closest distractor (A):** Most batches finish within an hour, but processing can take up to 24 hours.

### Q38 · A

**Why A is correct:** Per-file passes cannot see how files connect, so they contradict each other on shared paths. A cross-file integration pass resolves this. The model size is a red herring.

**Closest distractor (C):** Running each pass twice improves repeatability, but neither pass can see the other file.

### Q39 · C

**Why C is correct:** Explicit criteria should list local patterns to skip. Naming the accepted Result pattern removes the false positives and keeps other convention checks.

**Closest distractor (D):** Removing all convention findings is too broad and drops useful checks.

### Q40 · D

**Why D is correct:** tool_use fixes syntax errors. A retry with the error fixes misplaced values. Data that is not in the document needs null and human review, not more attempts.

**Closest distractor (C):** No prompt or retry can create a PO number that the document does not contain.

### Q41 · B

**Why B is correct:** The structured data lives in the input field of the tool_use content block. Any text block is optional and is not the extraction result.

**Closest distractor (D):** Your own code sends tool_result blocks; the API does not add them to Claude's response.

### Q42 · C

**Why C is correct:** Few-shot examples reduce hallucination for informal measurements and absent values. They show exactly how to map vague amounts and missing weights. The written rule already failed.

**Closest distractor (B):** Stronger wording is still just more instruction, which already did not work.

### Q43 · A

**Why A is correct:** One batch can hold up to 100,000 requests or 256 MB. Splitting into several batches keeps the batch discount. A custom_id per document keeps results easy to match.

**Closest distractor (D):** Bundling documents reduces the count, but it makes results harder to match and to resend when one fails.

### Q44 · D

**Why D is correct:** A model that keeps its generation reasoning is less likely to question its own decisions. That is why an independent instance catches more issues.

**Closest distractor (A):** Context limits can matter in long sessions, but they are not the main reason self-review fails.

### Q45 · B

**Why B is correct:** The instruction names what to report and what to skip. This removes trivial findings and keeps the useful ones.

**Closest distractor (C):** A confidence cutoff does not define which missing tests are worth reporting.

### Q46 · A

**Why A is correct:** Retries help with fixable errors, but they have limits. If the same check keeps failing, the document needs human review. The errors give the reviewer a clear starting point.

**Closest distractor (B):** Unlimited retries waste cost when the cause may be something a retry cannot fix.

### Q47 · C

**Why C is correct:** An "unclear" enum value lets Claude mark truly ambiguous cases. Analysts can then find them instead of seeing forced labels.

**Closest distractor (D):** "other" plus detail is for new categories, not for tone that fits several existing labels.

### Q48 · D

**Why D is correct:** The written explanation already exists and did not change the output. Examples showing a branch-level gap teach Claude the pattern to report.

**Closest distractor (A):** A longer explanation repeats the approach that has already failed.
