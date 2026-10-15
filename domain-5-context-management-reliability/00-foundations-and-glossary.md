# 00 · Foundations and Glossary (Domain 5)

> **Read this first.** This file explains why context management and reliability are hard, and the four risks that the rest of the domain addresses.
> Earlier domains define the stateless API and context ([Domain 1 · 00](../domain-1-agentic-architecture-orchestration/00-foundations-and-glossary.md)), structured errors ([Domain 2 · 02](../domain-2-tool-design-mcp-integration/02-structured-error-responses.md)), and precision and calibration ideas ([Domain 4 · 00](../domain-4-prompt-engineering-structured-output/00-foundations-and-glossary.md)).

**Exam link:** Background for all of Domain 5 (15% of the exam).

---

## 1. The context window is a budget

Every request to Claude includes **everything** Claude needs to see: system prompt, tool definitions, conversation history, tool results, documents. The **context window** is large, but it is still a **budget**:

- **Cost:** every token is paid for on every request that includes it.
- **Attention:** the more text there is, the easier it is for important details to be **overlooked**.
- **Limits:** long sessions eventually fill the window and must be **compressed** (summarized, compacted, or restarted).

Because the API is **stateless**, the application must **send the full history** each time to keep the conversation coherent. That history grows with each turn, especially with large tool results.

> **Domain 5 mindset:** Decide **what deserves space** in context, **where** it goes, and **what must never be lost** when context is compressed.

---

## 2. Four reliability risks

| Risk | What goes wrong | Example | File |
|---|---|---|---|
| **Information loss** | Exact facts become vague when history is summarized; details in the middle of long inputs are missed | "$1,247.83 refund" becomes "a refund was discussed" | [01](01-conversation-context-preservation.md), [04](04-large-codebase-context.md) |
| **Wrong decisions under uncertainty** | The agent guesses instead of asking, or escalates for the wrong reasons | Picks one of three "John Smith" accounts | [02](02-escalation-ambiguity-resolution.md) |
| **Hidden failures** | Errors are swallowed or over-reported, so outputs look complete but aren't | A timeout returned as "no results" | [03](03-error-propagation-multi-agent.md) |
| **Unverified confidence** | Good averages hide bad segments; claims lose their sources | "97% accurate" while handwritten forms are 60% | [05](05-human-review-confidence-calibration.md), [06](06-provenance-and-uncertainty.md) |

---

## 3. Three tools you'll use throughout

| Tool | Idea | Used in |
|---|---|---|
| **Structure** | Keep important data as fields (JSON, tables, labeled blocks), not buried in prose | Every file |
| **Isolation** | Do noisy work in a separate context (subagent) and return only the result | [01](01-conversation-context-preservation.md), [04](04-large-codebase-context.md) |
| **Persistence** | Write key facts and state to a place outside the conversation (case facts block, scratchpad file, manifest) | [01](01-conversation-context-preservation.md), [04](04-large-codebase-context.md) |

---

## 4. The six Domain 5 topics

| File | Task statement | One-line summary |
|---|---|---|
| [01](01-conversation-context-preservation.md) | 5.1 Preserve critical information | Keep exact facts safe; trim noise; mind position effects |
| [02](02-escalation-ambiguity-resolution.md) | 5.2 Escalation and ambiguity | Escalate on clear triggers; ask rather than guess |
| [03](03-error-propagation-multi-agent.md) | 5.3 Error propagation | Give the coordinator what it needs to recover; annotate gaps |
| [04](04-large-codebase-context.md) | 5.4 Large codebase exploration | Delegate, write things down, survive crashes, compact |
| [05](05-human-review-confidence-calibration.md) | 5.5 Human review and calibration | Measure by segment, sample, route by calibrated confidence |
| [06](06-provenance-and-uncertainty.md) | 5.6 Provenance and uncertainty | Every claim keeps its source and date; show conflicts |
| [MCQ bank](../mcq-bank/05-context-management-reliability.md) | All | 36 exam-level practice questions |

---

## 5. Glossary

| Term | Meaning |
|---|---|
| **Aggregate accuracy** | One overall accuracy number across all documents and fields |
| **Case facts block** | A structured block of exact transactional facts kept in every prompt, outside summarized history |
| **Claim-source mapping** | A record linking each claim to its source (URL, document, page, excerpt) |
| **`/compact`** | Claude Code command that replaces conversation history with a summary to free context |
| **Context degradation** | Answers getting vaguer or inconsistent as a long session fills with content |
| **Coverage annotation** | A note in a report showing which topics are well-supported and which have gaps |
| **Escalation trigger** | A defined condition under which the agent hands a case to a human |
| **Lost in the middle** | Models handle the beginning and end of long inputs well but may miss content in the middle |
| **Manifest** | A structured state file an agent exports so work can resume after a crash |
| **Policy gap** | A customer request that existing policy doesn't clearly cover |
| **Progressive summarization** | Repeatedly summarizing history to save space, which loses detail over time |
| **Provenance** | Where a piece of information came from, and when |
| **Scratchpad file** | A file where an agent records key findings to refer back to later |
| **Stratified sampling** | Sampling separately from each segment (document type, field, confidence band) so each is measured |
| **Temporal data** | Information tied to a time (publication date, data collection period) |

---

## Common Q&A

**Q: If the context window is huge, why manage it at all?**
A: Size isn't the only problem. Cost grows with tokens, attention gets diluted, and important details can be missed in long inputs.

**Q: Isn't summarizing history the obvious fix for long conversations?**
A: It helps with space but **loses exact details** (amounts, dates, promises). Keep those in a structured block that is never summarized.

**Q: What does "reliability" mean in this domain?**
A: The system keeps important facts, makes sensible decisions when unsure, surfaces failures honestly, and can show where its claims come from.

---

## Key Takeaways

- Context is a **budget**: cost, attention, and limits.
- Four risks: **information loss**, **wrong decisions under uncertainty**, **hidden failures**, **unverified confidence**.
- Three tools: **structure**, **isolation**, **persistence**.
- Exact facts and sources must **survive** compression.
