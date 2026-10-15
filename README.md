# Claude Certified Architect – Foundations: Study Prep

A self-contained study set for the **Claude Certified Architect – Foundations** exam (exam code CCAR-F).

The goal: after reading only this repository, you understand every exam domain and can answer hard, scenario-based questions. You should not need any other website.

## The exam at a glance

These facts come from the official exam guide (v1.0, July 2026). Anything not in this table is unconfirmed.

| Item | Value |
|---|---|
| Questions | 60. Multiple-choice and multiple-response (each item says how many answers to pick). |
| Structure | 4 scenarios, picked at random from a set of 6 |
| Time | 120 minutes |
| Passing score | Scaled score of 720, on a scale of 100 to 1,000 |
| Domain weights | D1 27% · D2 18% · D3 20% · D4 20% · D5 15% |

Full source details: [docs/verification-log.md §D](docs/verification-log.md).

## How to use this repo

1. **Read a domain file.** Each one has the same six sections:
   - **Exam focus:** what the exam tests in this domain
   - **Key concepts:** short explanations, one idea at a time
   - **Common traps:** tempting wrong answers, and why they are wrong
   - **Worked scenario:** one realistic example, step by step
   - **Quick recap:** the main points
   - **Practice questions:** a link to the question bank
2. **Do that domain's questions** in the [MCQ bank](mcq-bank/README.md). Answer all of them first, then check the answer key at the end of the file.
3. **For every wrong answer,** read the explanation. Then reread the matching "Key concepts" part.
4. **Use the [cheat sheet](claude-architect-cheat-sheet.md)** for fast review in the last days before the exam.
5. **Go deeper when needed.** The `domain-*` folders have longer notes per task statement, code examples, and older warm-up questions.

⚠️ A warning sign in the notes means the **exam answer** and **today's product** differ. Learn the exam answer for the test, and know the product fact for real work. All such items are in [REVIEW_NEEDED.md](REVIEW_NEEDED.md).

## Study order

The order follows dependencies: each domain builds on the one before.

| Step | Read | Then practice | Why this order |
|---|---|---|---|
| 1 | [Domain 1: Agentic Architecture & Orchestration](domains/01-agentic-architecture.md) | [65 questions](mcq-bank/01-agentic-architecture.md) | The loop, subagents, and hooks are the base for everything else. Largest weight (27%). |
| 2 | [Domain 2: Tool Design & MCP Integration](domains/02-tool-design-mcp.md) | [43 questions](mcq-bank/02-tool-design-mcp.md) | Tools are what the loop calls. |
| 3 | [Domain 5: Context Management & Reliability](domains/05-context-management-reliability.md) | [36 questions](mcq-bank/05-context-management-reliability.md) | It continues the multi-agent ideas from Domains 1 and 2: errors, escalation, provenance. |
| 4 | [Domain 4: Prompt Engineering & Structured Output](domains/04-prompt-engineering-structured-output.md) | [48 questions](mcq-bank/04-prompt-engineering-structured-output.md) | Schemas, validation, and batch. It uses `tool_choice` from Domain 2. |
| 5 | [Domain 3: Claude Code Configuration & Workflows](domains/03-claude-code-configuration.md) | [48 questions](mcq-bank/03-claude-code-configuration.md) | Claude Code applies all the earlier ideas (subagents, review, CI). |
| 6 | [Cheat sheet](claude-architect-cheat-sheet.md) + full mixed review | All 240 questions | Final review |

## Repository map

| Path | What it is |
|---|---|
| [domains/](domains/) | **Main study files.** One per domain, six sections each. |
| [mcq-bank/](mcq-bank/README.md) | **240 original practice questions.** Weighted by domain, with answer keys and explanations. |
| [claude-architect-cheat-sheet.md](claude-architect-cheat-sheet.md) | One-page-per-domain decision rules, converted from the owner's PDF, with ⚠️ verify notes |
| [docs/domain-map.md](docs/domain-map.md) | How the 5 domains and 30 task statements were derived, with sources |
| [docs/verification-log.md](docs/verification-log.md) | Every technical claim and the official page it was checked against |
| [docs/audit.md](docs/audit.md) | Audit of the repo before this update: file summaries and gaps |
| [REVIEW_NEEDED.md](REVIEW_NEEDED.md) | Conflicts, unverified claims, and open decisions |
| `domain-1-...` to `domain-5-...` | Deep-dive notes per task statement, plus older warm-up questions (easier than the bank) |
| `labs/`, `notes/`, `practice-questions/` | Empty placeholders |

## Short glossary

These terms appear in every domain:

| Term | Meaning |
|---|---|
| **API** (application programming interface) | The way your code talks to Claude |
| **SDK** (software development kit) | A code library. The Claude Agent SDK helps you build agents. |
| **MCP** (Model Context Protocol) | An open standard for connecting Claude to outside tools and data |
| **CI/CD** (continuous integration / continuous delivery) | Automated checks and releases on every code change |
| **PR** (pull request) | A proposed code change for review |
| **MCQ** (multiple-choice question) | A question with options A to D |
| **SLA** (service-level agreement) | A promised time limit for results |

## Ground rules

- No secrets, API keys, or `.env` files in this repo (see `.gitignore`).
- No Cinnova client data or client-specific architecture. Every scenario uses invented companies and IDs.
- Practice questions and notes are original. Nothing was copied from third-party practice sites.

## Owner

Azeem Subhani
