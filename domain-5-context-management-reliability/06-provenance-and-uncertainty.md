# 06 · Preserving Provenance and Handling Uncertainty in Multi-Source Synthesis

**Exam link:** Task Statement 5.6: *Preserve information provenance and handle uncertainty in multi-source synthesis.*
**Prerequisites:** Domain 1 [03 · Subagent Configuration](../domain-1-agentic-architecture-orchestration/03-subagent-configuration.md) (structured formats preserve attribution), [01](01-conversation-context-preservation.md) (metadata in upstream outputs).
**Scope note:** Domain 1 showed that structured data keeps source URLs between agents. This file covers what the **synthesis** must do with sources: keep **claim-source mappings** through summarization, handle **conflicting** sources, read **dates** correctly, and **present** results clearly.

---

## 1. Provenance is lost during summarization

**Provenance** = where a piece of information came from (and when).

In a research pipeline, findings are **compressed** at several steps: subagent summary → coordinator summary → synthesis. Each compression risks **dropping the link** between a claim and its source:

```
"Smith et al. (2025, n=1,200) found 68% of indie musicians use AI mastering tools."
     ▼ summary
"Most indie musicians use AI mastering tools."
     ▼ synthesis
"AI tools are now standard in music production."      ← no source, stronger claim, wrong
```

The final claim is **unsupported**, **overstated**, and **impossible to verify**.

---

## 2. Structured claim-source mappings

Require subagents to output **claim-source mappings**, and require the synthesis agent to **preserve and merge** them:

```json
{
  "claim_id": "C7",
  "claim": "68% of surveyed indie musicians used AI mastering tools in 2024.",
  "sources": [
    {
      "source_url": "https://example.org/indie-music-survey-2025",
      "document_name": "Indie Music Tech Survey 2025",
      "excerpt": "68% of respondents reported using an AI-based mastering service at least once in 2024.",
      "publication_date": "2025-01-15",
      "data_collection": "2024-09 to 2024-11",
      "methodology": "online survey, n=1,200, self-selected sample"
    }
  ]
}
```

**Synthesis rules:**
- Every statement in the report must reference **one or more `claim_id`s**.
- When two subagents report the **same** claim, **merge** their sources (don't drop one).
- Keep the **original characterization** ("self-selected survey", "pilot study"), so a weak study isn't presented as strong evidence.

---

## 3. Conflicting sources: annotate, don't pick

Credible sources sometimes **disagree**:

| Source | Claim |
|---|---|
| Industry report A (2025) | AI reduced publishing editorial jobs by **12%** |
| University study B (2025) | No significant change in editorial employment (**±2%**) |

**Wrong:** arbitrarily pick one ("12%") and present it as fact. The reader never learns that the evidence is contested.

**Right:** **annotate the conflict with attribution**:

> **Contested:** Estimates of AI's effect on editorial jobs differ. An industry report (A, 2025) estimates a 12% reduction, based on employer surveys; a university study (B, 2025) finds no significant change, based on national labor statistics. The difference may reflect methodology (employer self-reports vs official data).

### Let the coordinator reconcile

When a document-analysis subagent finds **conflicting values**, it should **complete its analysis**, include **both** values, and **flag** the conflict explicitly. It should **not** quietly resolve it. The **coordinator** (which sees all sources) decides how to handle it **before** passing results to synthesis:

```json
{
  "field": "editorial_job_change_2024",
  "values": [
    {"value": "-12%", "source": "A", "method": "employer survey"},
    {"value": "0% ± 2%", "source": "B", "method": "labor statistics"}
  ],
  "conflict_detected": true
}
```

---

## 4. Temporal data: dates prevent false contradictions

Two numbers can look contradictory when they're simply from **different times**:

| Source | Claim | Date |
|---|---|---|
| C | 20% of studios use AI tools | Data from **2022** |
| D | 55% of studios use AI tools | Data from **2025** |

Without dates, this looks like a conflict. With dates, it's a **trend**: adoption grew from 20% to 55%.

**Require** subagents to output **publication date** and **data collection period** for every claim. Then the synthesis can:
- show **change over time** correctly,
- prefer **recent** data where appropriate,
- avoid labeling a time difference as a contradiction.

---

## 5. Structure the report by certainty

Separate findings by how well they're supported:

```markdown
## Well-established findings
- AI-assisted mastering is widely used by independent musicians (C7: 3 independent surveys, 2024–2025).

## Contested findings
- Effect on editorial employment: estimates range from −12% to no change (A vs B; see methodology notes).

## Limited evidence / gaps
- Film scoring: one trade-press source only; academic database unavailable during research.
```

Readers immediately see **what is solid**, **what is debated**, and **what is unknown**. (This connects to the coverage annotations in [03](03-error-propagation-multi-agent.md).)

---

## 6. Render each content type appropriately

Don't force everything into one format. Choose the form that fits the content:

| Content type | Best rendering | Why |
|---|---|---|
| **Financial / numeric data** (revenues, rates by year) | **Tables** | Easy comparison across rows and columns |
| **News and narrative developments** | **Prose** | Context, cause, and sequence matter |
| **Technical findings** (specs, steps, requirements) | **Structured lists** | Scannable, precise |

Converting **everything** into a single uniform format (e.g. all bullets, or all tables) makes some content harder to read and can strip context (a news story in a table cell loses its meaning).

---

## 7. Real-world case study

**Situation:** A consulting firm's research system produced a client report stating "AI has cut publishing jobs by 12%." The client's own analysts found a major study showing no change and questioned the whole report.

**Root causes:**
- The synthesis **picked one** of two conflicting statistics.
- Summarization had **dropped the source and methodology** for the 12% figure.
- A 2022 adoption figure and a 2025 figure were presented as "conflicting estimates" because **dates were missing**.

**Changes:**
1. Subagents output **claim-source mappings** with excerpts, dates, and methodology.
2. Document-analysis subagents **flag conflicts** with both values; the **coordinator reconciles** before synthesis.
3. Reports have **well-established / contested / gaps** sections.
4. Financial figures rendered as **tables**, news as **prose**, technical findings as **lists**.

---

## Common Q&A

**Q: How does attribution get lost?**
A: When findings are summarized or compressed without keeping claim-source links.

**Q: Two credible sources give different numbers. Which should the report use?**
A: **Both**, with attribution and an annotation explaining the conflict (and methodology differences if known). Don't pick one arbitrarily.

**Q: Who should resolve conflicts between sources?**
A: The analysis subagent **flags** them with both values; the **coordinator** decides how to reconcile before synthesis.

**Q: Why require publication or data collection dates?**
A: So differences over time aren't misread as contradictions, and the report can show trends.

**Q: Should all findings be presented as tables for consistency?**
A: No. Financial data as **tables**, news as **prose**, technical findings as **structured lists**.

---

## Key Takeaways

- **Summarization drops provenance** unless claims keep **structured source mappings** (URL, document, excerpt, date, method).
- Synthesis must **preserve and merge** claim-source mappings.
- **Conflicting credible sources** → **annotate with attribution**; let the **coordinator reconcile**.
- **Dates** (publication and collection) prevent **false contradictions** and reveal trends.
- Structure reports into **well-established / contested / gaps**.
- **Render by content type**: tables, prose, structured lists.
