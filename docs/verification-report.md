# Verification Report (Step 5)

**Date:** 2026-10-15
**Scope:** the new domain files, the MCQ bank, the cheat sheet conversion, and the technical claims.
**Method:** every result below is real output from the scripts or review passes described. The scripts were kept outside the repo (session scratchpad). Their logic is summarized here so the checks can be repeated.

## 1. MCQ bank

Full table: [mcq-bank/README.md §Quality checks](../mcq-bank/README.md#quality-checks-step-5-results).

| Required check | Result |
|---|---|
| Total MCQ count is exactly 240 | **Pass:** 240 (D1 65, D2 43, D3 48, D4 48, D5 36) |
| Each question has exactly 4 options and exactly 1 correct answer | **Pass:** 240/240 |
| Each answer key letter matches the intended answer | **Pass:** the key matches the "Why X is correct" letter in 240/240 items. An independent blind re-answer (different model, keys removed) agreed in 240/240. |
| No question copied from an external site | **Pass** |

How the "no copying" result was reached:
- The cyberskill practice site was never opened.
- claudecertificationguide.com was read for one page (agentic loops) for ideas only, and nothing was copied.
- Lexical similarity against the 125 older repo questions and the guide's sample questions found 0 near-matches.
- One question that was too close in meaning (D2-Q2) was rewritten by hand.

## 2. Domain files

| Required check | Result |
|---|---|
| Every domain file has all 6 required sections | **Pass:** 5/5 files have Exam focus, Key concepts, Common traps, Worked scenario, Quick recap, Practice questions |
| No sentence longer than 25 words | **Pass:** 0 found (outside code blocks), after 4 sentences in Domain 2 were split |
| No unexplained acronym | **Pass**, with these notes on what the script flagged: |

Notes on the acronym check:
- Flags that remained were not acronyms: emphasis words like BEST, the `CLAUDE` in CLAUDE.md, and the YYYY date pattern.
- Each file explains its acronyms at first use (API, SDK, MCP, CI/CD, SLA, URI, JSON-RPC).
- The root README has a short glossary.

## 3. Technical claims

Every technical claim in the new files is either:
- **confirmed** against official docs, with a row in [verification-log.md](verification-log.md) (rows A1 to A10, B1 to B5, C1 to C19), or
- **marked for review** in [REVIEW_NEEDED.md](../REVIEW_NEEDED.md).

Counts of review markers:

| Marker | Count | Where |
|---|---|---|
| Conflicts and partial matches (exam vs docs, or guide vs cheat sheet) | 14 | REVIEW_NEEDED.md R1 to R14 |
| Claims still to check word for word (VERIFY) | 6 | REVIEW_NEEDED.md V1 to V6 |
| Decisions for the owner | 6 | REVIEW_NEEDED.md D1 to D6 |
| ⚠️ notes added to the cheat sheet | 16 | claude-architect-cheat-sheet.md |
| ⚠️ notes in the domain files (not counting the legend line in each header) | 12 | domains/*.md |

## 4. Cheat sheet conversion

- **Text extraction:** macOS PDFKit (built-in, no new dependency). The PDF had a text layer, so OCR (optical character recognition) was not needed.
- **Content check:** all 17 pages and every "If / Don't / Note / Exam traps" line are present, in the original order.
- **Fixes made:** only split words, `**` glob characters lost in the PDF, and one scrambled line in 4.5. That line is marked in place and listed as V6 for owner confirmation.

## 5. What was not verified

- **Real exam grading.** The keys follow the official guide's framing. They have not been checked against actual exam items, which are confidential.
- **Exact doc wording.** Most doc pages were read through a summarizing fetch. Exact wording should be confirmed before quoting (V1 to V5).
- **Human review.** A person should spot-check the hard questions before the bank is shared outside this repo.
