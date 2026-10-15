# Domain Map

**Purpose:** list the exam domains and task statements, and show where each one came from.
**Method:** the domain list was derived from four sources and then cross-checked. It was not assumed.

## Sources used

| Code | Source | What it gives |
|---|---|---|
| **G** | Official exam guide PDF, `Claude Certified Architect Foundations Exam Guide.pdf`, version 1.0, July 2026, §4 and §6 | Domain names, weights, and the 30 task statements. **This is the authority.** |
| **C** | Owner's cheat sheet, [claude-architect-cheat-sheet.md](../claude-architect-cheat-sheet.md) | Domain headings 1 to 5 and sections 1.1 to 5.6. No weights. |
| **R** | Existing repo folders `domain-1-...` to `domain-5-...` (before this update) | Folder names and per-topic files that match task statements 1.1 to 5.6. |
| **W** | URL path of the reference guide, `claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops` | Shows "1" = Agentic Architecture and "1-1" = Agentic loops. Only this one page was opened. |

## Result

All four sources agree on 5 domains with the same order and names. Only the guide (G) gives weights.

| # | Domain | Weight | Sources that agree | Study file | MCQ file |
|---|---|---|---|---|---|
| 1 | Agentic Architecture & Orchestration | 27% (G) | G, C, R, W | [domains/01-agentic-architecture.md](../domains/01-agentic-architecture.md) | [mcq-bank/01-agentic-architecture.md](../mcq-bank/01-agentic-architecture.md) |
| 2 | Tool Design & MCP Integration | 18% (G) | G, C, R | [domains/02-tool-design-mcp.md](../domains/02-tool-design-mcp.md) | [mcq-bank/02-tool-design-mcp.md](../mcq-bank/02-tool-design-mcp.md) |
| 3 | Claude Code Configuration & Workflows | 20% (G) | G, C, R | [domains/03-claude-code-configuration.md](../domains/03-claude-code-configuration.md) | [mcq-bank/03-claude-code-configuration.md](../mcq-bank/03-claude-code-configuration.md) |
| 4 | Prompt Engineering & Structured Output | 20% (G) | G, C, R | [domains/04-prompt-engineering-structured-output.md](../domains/04-prompt-engineering-structured-output.md) | [mcq-bank/04-prompt-engineering-structured-output.md](../mcq-bank/04-prompt-engineering-structured-output.md) |
| 5 | Context Management & Reliability | 15% (G) | G, C, R | [domains/05-context-management-reliability.md](../domains/05-context-management-reliability.md) | [mcq-bank/05-context-management-reliability.md](../mcq-bank/05-context-management-reliability.md) |

## Task statements (from G §6, short titles)

The cheat sheet (C) uses the same numbers, and the repo (R) has one file for each.

**Domain 1**
- 1.1 Agentic loops for autonomous task execution
- 1.2 Multi-agent systems with coordinator-subagent patterns
- 1.3 Subagent invocation, context passing, and spawning
- 1.4 Multi-step workflows with enforcement and handoff
- 1.5 Agent SDK hooks for tool call interception and data normalization
- 1.6 Task decomposition strategies
- 1.7 Session state, resumption, and forking

**Domain 2**
- 2.1 Tool interfaces with clear descriptions and boundaries
- 2.2 Structured error responses for MCP tools
- 2.3 Tool distribution across agents and tool_choice
- 2.4 MCP servers in Claude Code and agent workflows
- 2.5 Built-in tools (Read, Write, Edit, Bash, Grep, Glob)

**Domain 3**
- 3.1 CLAUDE.md hierarchy, scoping, and modular organization
- 3.2 Custom slash commands and skills
- 3.3 Path-specific rules
- 3.4 Plan mode vs direct execution
- 3.5 Iterative refinement
- 3.6 Claude Code in CI/CD pipelines

**Domain 4**
- 4.1 Explicit criteria to improve precision
- 4.2 Few-shot prompting
- 4.3 Structured output with tool use and JSON schemas
- 4.4 Validation, retry, and feedback loops
- 4.5 Batch processing
- 4.6 Multi-instance and multi-pass review

**Domain 5**
- 5.1 Conversation context across long interactions
- 5.2 Escalation and ambiguity resolution
- 5.3 Error propagation in multi-agent systems
- 5.4 Context in large codebase exploration
- 5.5 Human review and confidence calibration
- 5.6 Provenance and uncertainty in multi-source synthesis

## Exam scenarios (G §5)

The exam shows 4 of these 6 scenarios, picked at random. The MCQ bank uses scenario settings like these, with new details.

| # | Scenario | Primary domains (G) |
|---|---|---|
| 1 | Customer Support Resolution Agent | 1, 2, 5 |
| 2 | Code Generation with Claude Code | 3, 5 |
| 3 | Multi-Agent Research System | 1, 2, 5 |
| 4 | Developer Productivity with Claude | 2, 3, 1 |
| 5 | Claude Code for Continuous Integration | 3, 4 |
| 6 | Structured Data Extraction | 4, 5 |

## Out of scope (G §17)

These topics will not be tested:
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
- prompt caching **details** (only "knowing it exists" is in scope)
