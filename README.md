# Claude Certified Architect – Foundations: Study Prep

A self-contained study set for the **Claude Certified Architect – Foundations** exam (exam code CCAR-F).

Read only this repository and you can learn every exam domain and practice with hard, scenario-based questions.

## The exam at a glance

These facts come from the official exam guide (v1.0, July 2026). Anything not in this table is not confirmed.

| Item | Value | Guide section |
|---|---|---|
| Questions | 60. Multiple-choice and multiple-response (each item says how many answers to pick). | §3 |
| Structure | 4 scenarios, picked at random from a set of 6 | §3, §5 |
| Time | 120 minutes | §3 |
| Passing score | Scaled score of 720, on a scale of 100 to 1,000 | §3, §10 |
| Fee | $125 USD | §3 |
| Credential valid for | 12 months | §3 |
| Domain weights | D1 27% · D2 18% · D3 20% · D4 20% · D5 15% | §4 |

### The six exam scenarios (guide §5)

| # | Scenario | Main domains |
|---|---|---|
| 1 | Customer Support Resolution Agent | 1, 2, 5 |
| 2 | Code Generation with Claude Code | 3, 5 |
| 3 | Multi-Agent Research System | 1, 2, 5 |
| 4 | Developer Productivity with Claude | 2, 3, 1 |
| 5 | Claude Code for Continuous Integration | 3, 4 |
| 6 | Structured Data Extraction | 4, 5 |

### Not on the exam (guide §17)

These topics are **not** tested:
- fine-tuning
- API authentication and billing
- hosting MCP servers
- model internals and training
- embeddings
- computer use
- vision
- streaming
- rate limits and pricing math
- OAuth
- cloud-provider setup
- benchmarks
- tokenization
- prompt caching **details** (you only need to know that it exists)

## What is in this repo

| Path | What it is |
|---|---|
| [domain-1-agentic-architecture-orchestration/](domain-1-agentic-architecture-orchestration/README.md) | Domain 1: agentic loops, multi-agent systems, subagents, enforcement, hooks, decomposition, sessions |
| [domain-2-tool-design-mcp-integration/](domain-2-tool-design-mcp-integration/README.md) | Domain 2: tool descriptions, error responses, tool distribution, MCP servers, built-in tools |
| [domain-3-claude-code-configuration-workflows/](domain-3-claude-code-configuration-workflows/README.md) | Domain 3: CLAUDE.md, settings, skills, path rules, plan mode, iterative refinement, CI/CD |
| [domain-4-prompt-engineering-structured-output/](domain-4-prompt-engineering-structured-output/README.md) | Domain 4: explicit criteria, few-shot, schemas, validation and retry, batch, review design |
| [domain-5-context-management-reliability/](domain-5-context-management-reliability/README.md) | Domain 5: long context, escalation, error propagation, codebase sessions, human review, provenance |
| [mcq-bank/](mcq-bank/README.md) | 240 original practice questions, weighted by domain, with answer keys and explanations |
| [claude-architect-cheat-sheet.md](claude-architect-cheat-sheet.md) | Short decision rules for every domain, for fast review |

**Each domain folder has the same layout:**

| File | Purpose |
|---|---|
| `README.md` | Start here. Exam focus, the task statements, key themes, a worked scenario, and the sources checked |
| `00-foundations-and-glossary.md` | Terms you need before the topic files |
| `01-...` and onward | One file per task statement, read in number order |
| `anti-patterns.md` | Wrong approaches, why they fail, what to do instead, and common exam traps |

## How to study

1. **Open a domain folder's `README.md`.** Read "Exam focus" first.
2. **Read `00` and the numbered topic files** in order.
3. **Read `anti-patterns.md`.** These are the wrong answers the exam uses.
4. **Answer that domain's questions** in the [MCQ bank](mcq-bank/README.md). Answer all of them before you check the key at the end of the file.
5. **For every miss,** read the explanation and the "closest distractor" line. Then reread the matching topic file.
6. **In the last days,** review with the [cheat sheet](claude-architect-cheat-sheet.md).

⚠️ A warning sign in the notes means the **exam answer** and **today's product** differ. Learn the exam answer for the test, and know the product fact for real work.

## Study order

| Step | Domain folder | Questions | Why this order |
|---|---|---|---|
| 1 | [Domain 1: Agentic Architecture & Orchestration](domain-1-agentic-architecture-orchestration/README.md) | [65](mcq-bank/01-agentic-architecture.md) | The loop, subagents, and hooks are the base for everything else. Largest weight (27%). |
| 2 | [Domain 2: Tool Design & MCP Integration](domain-2-tool-design-mcp-integration/README.md) | [43](mcq-bank/02-tool-design-mcp.md) | Tools are what the loop calls. |
| 3 | [Domain 5: Context Management & Reliability](domain-5-context-management-reliability/README.md) | [36](mcq-bank/05-context-management-reliability.md) | It continues the multi-agent ideas from Domains 1 and 2: errors, escalation, provenance. |
| 4 | [Domain 4: Prompt Engineering & Structured Output](domain-4-prompt-engineering-structured-output/README.md) | [48](mcq-bank/04-prompt-engineering-structured-output.md) | Schemas, validation, and batch. It uses `tool_choice` from Domain 2. |
| 5 | [Domain 3: Claude Code Configuration & Workflows](domain-3-claude-code-configuration-workflows/README.md) | [48](mcq-bank/03-claude-code-configuration.md) | Claude Code applies all the earlier ideas (subagents, review, CI). |
| 6 | [Cheat sheet](claude-architect-cheat-sheet.md) + all 240 questions mixed | [240](mcq-bank/README.md) | Final review |

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
