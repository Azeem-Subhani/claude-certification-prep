# MCQ Bank: 240 Original Practice Questions

**Purpose:** scenario-based practice at real exam difficulty. The distractors are plausible, and each one fails for a specific reason.
**Owner:** Azeem Subhani.
**Built:** 2026-10-15.
**Status:** passed the automated checks and an independent blind answer check (see below). A human should still spot-check before sharing it outside the repo.

## Files

| Domain | File | Questions | Easy | Medium | Hard | Exam weight |
|---|---|---|---|---|---|---|
| 1. Agentic Architecture & Orchestration | [01-agentic-architecture.md](01-agentic-architecture.md) | 65 | 16 | 26 | 23 | 27% |
| 2. Tool Design & MCP Integration | [02-tool-design-mcp.md](02-tool-design-mcp.md) | 43 | 11 | 17 | 15 | 18% |
| 3. Claude Code Configuration & Workflows | [03-claude-code-configuration.md](03-claude-code-configuration.md) | 48 | 12 | 19 | 17 | 20% |
| 4. Prompt Engineering & Structured Output | [04-prompt-engineering-structured-output.md](04-prompt-engineering-structured-output.md) | 48 | 12 | 19 | 17 | 20% |
| 5. Context Management & Reliability | [05-context-management-reliability.md](05-context-management-reliability.md) | 36 | 9 | 15 | 12 | 15% |
| **Total** | | **240** | **60 (25%)** | **96 (40%)** | **84 (35%)** | 100% |

**How the 240 were split:**
- The questions follow the **official domain weights** in the exam guide v1.0, §4 (240 × weight, rounded).
- Each domain uses a 25 / 40 / 35 mix of easy / medium / hard.
- Your brief's first difficulty figures added up to 130%, so the "roughly 25/40/35" target was used instead ([REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item D4).

## How to use

1. Study the domain folder first (`domain-1-...` to `domain-5-...`). Start with its `README.md`.
2. Answer every question in the domain file **before** you look at the answer key at the bottom.
3. Score yourself. For each miss, read the explanation **and** the "closest distractor" line. The distractor line shows the trap you fell into.
4. Read the domain's `anti-patterns.md`, then reread the topic file for each question you missed.
5. On a second pass, time yourself. The real exam gives 120 minutes for 60 items, about 2 minutes per item.

**Format of each question:**
- a header with the number, task statement, and difficulty
- a 2 to 4 sentence scenario
- a question that uses words like BEST, MOST likely, or FIRST
- four options, A to D

**Format of each answer:**
- the correct letter
- 2 to 3 sentences on why it is correct
- one sentence on the closest distractor

## Things the real exam has that this bank does not

- **Multiple-response items.** The exam also asks you to "choose two" (or more). The item tells you how many. This bank has single-answer items only, as requested ([REVIEW_NEEDED.md](../REVIEW_NEEDED.md) item D2).
- **Scenario sets.** On the exam, several items share one scenario. Here, each item has its own short scenario.

## Exam answer vs product behavior

The keys follow the **exam guide's framing**. Questions avoid points where the exam and today's product disagree (see [REVIEW_NEEDED.md](../REVIEW_NEEDED.md) §A). Two examples:
- A question that needs `tool_choice: "any"` or a forced tool says in the scenario that the model supports forced tool use.
- No key depends on whether skill `allowed-tools` restricts or pre-approves tools.

## Quality checks (Step 5 results)

All numbers below are real script and agent output from 2026-10-15.

| Check | Method | Result |
|---|---|---|
| Total count is exactly 240 | Parser counts `### Q<n> · Task` headers | **240** (65 + 43 + 48 + 48 + 36) |
| Exactly 4 options per question | Parser checks options A, B, C, D in order | **240/240 pass** |
| Exactly 1 key per question, letter A to D | Parser matches each question to one `### Q<n> · <letter>` key | **240/240 pass** |
| Key letter matches the explanation | "Why X is correct" letter = key letter. The closest-distractor letter ≠ key letter. | **240/240 pass** |
| Key matches the intended answer | **Blind re-answer:** a different model (Sonnet) answered every question with the keys removed | **240/240 agree** |
| Ambiguity review | The blind solvers flagged items where a second option was arguable. Each was reviewed. | 1 stem tightened (D4-Q29). The others hold. |
| Answer letter balance | Count per file | Each letter 22 to 28% per file. Never more than 2 of the same letter in a row. |
| "Longest option is correct" cue | Count per file | D1 17%, D2 19%, D3 23%, D4 25%, D5 28% (the old practice set was mostly "B") |
| No near-duplicates | Word-trigram and sequence similarity: within the bank, against the 125 older repo questions (since removed from the repo), and against the guide's sample questions | 0 flagged by script. 1 duplicate in meaning was found by hand review (D2-Q2) and rewritten. |
| No copied content | The cyberskill practice site was **not visited** by any writer. The writers were told to use no web sources. Lexical check against the older repo set and the guide samples found no matches. | Pass |
| Readability | Sentence-length script: no sentence over 25 words, outside code | **0** long sentences in the 5 bank files |
| Neutral wording | Scan for gendered pronouns in scenarios | 10 found and made neutral |

**Limits of these checks:**
- The blind solvers agreeing with the keys shows the keys are consistent and defensible. It does not prove that the real exam would grade them the same way.
- The duplicate check finds wording overlap. Two questions can test the same idea in different settings, and that is allowed.
