# 05 · Iterative Refinement Techniques

**Exam link:** Task Statement 3.5: *Apply iterative refinement techniques for progressive improvement.*
**Prerequisite:** [00](00-foundations-and-glossary.md).

---

## 1. Why iteration?

Your first request to Claude is rarely perfect. Requirements are unclear, edge cases are missed, and words mean different things to different people. **Iterative refinement** means improving the result step by step with **clear, concrete feedback**.

The exam tests four techniques:

| Technique | Solves |
|---|---|
| **Concrete input/output examples** | Prose descriptions are interpreted in different ways |
| **Test-driven iteration** | "Done" is vague; you need an objective target |
| **Interview pattern** | You don't know what you don't know (unfamiliar domains) |
| **Smart feedback batching** | Fixing issues in the wrong order causes back-and-forth |

---

## 2. Concrete input/output examples

**Problem:** You describe a transformation in words, and the results are inconsistent.

> "Normalize the customer names." What does "normalize" mean? Capitalize? Remove titles? Handle "McDonald"?

**Fix:** Show **2–3 concrete examples** of input → expected output:

```text
Normalize customer names. Examples:

Input:  "  mr. JOHN o'brien "     →  Output: "John O'Brien"
Input:  "DR. ana maría lópez"     →  Output: "Ana María López"
Input:  "mcdonald, ronald"        →  Output: "Ronald McDonald"
```

From these, Claude learns: trim spaces, drop titles, fix capitalization (including O' and Mc), keep accents, and reorder "Last, First". That's much clearer than a paragraph of rules.

**Why it works:** Examples remove ambiguity. Prose like "proper capitalization" can be read in many ways; an example has only one meaning.

**Tips:**
- Cover **different cases** (normal, tricky, edge), not three similar ones.
- 2–3 examples is usually enough; add more only for genuinely different cases.

---

## 3. Test-driven iteration

**Idea:** Write the **tests first**, then let Claude implement until they pass. Tests give an **objective** definition of "correct".

**Steps:**

1. Write a test suite covering:
   - **expected behavior** (normal cases),
   - **edge cases** (empty input, nulls, very large values),
   - **performance requirements** where relevant (e.g. "handles 100k rows in < 2 s").
2. Ask Claude to implement the code.
3. Run the tests and **share the failures** with Claude.
4. Repeat until everything passes.

```text
Run: pytest tests/test_invoice_parser.py

FAILED test_parses_negative_amounts: expected -45.00, got 45.00
FAILED test_handles_missing_due_date: TypeError: 'NoneType' object is not subscriptable

Fix these two failures. Don't change the tests.
```

**Why it works:** Failure messages are **precise feedback**. Claude sees exactly what's wrong, not "it doesn't work".

### Specific test cases for edge cases

When one edge case keeps failing, give a **specific test case** with **example input and expected output**:

```text
The migration script fails when `middle_name` is NULL.
Input row:  {"first": "Ana", "middle_name": null, "last": "Silva"}
Expected:   {"full_name": "Ana Silva"}         # no "None", no double space
```

This is far clearer than "handle nulls better".

---

## 4. The interview pattern

**Problem:** In an **unfamiliar domain**, you may not know which design questions matter. If Claude starts implementing immediately, it will make silent assumptions.

**Fix:** Ask Claude to **interview you first**:

```text
I need a caching layer for our product catalog API. Before writing any code,
ask me the questions you need answered to design this well. Ask one group at a time.
```

Claude might ask:

- How often does catalog data change? Who changes it?
- What **cache invalidation strategy** do you need: time-based (TTL), event-based (on update), or both?
- What should happen when the cache server is down: fail open (go to DB) or fail closed?
- How stale can prices be? Is it acceptable for a price to be 5 minutes old?
- Is there a multi-region deployment?

These **surface considerations you may not have thought of** (invalidation, failure modes, staleness) **before** any code is written.

**When to use it:** unfamiliar domains, vague requirements, decisions with long-term impact.

---

## 5. All at once or one at a time?

When you have **several issues** to fix, should you send them together or separately?

| Issues are… | Approach | Why |
|---|---|---|
| **Interacting**: fixing one affects another | **One detailed message** with all of them | Claude can design one consistent fix; separate fixes would conflict or undo each other |
| **Independent**: unrelated to each other | **Sequential**, one at a time | Easier to verify each fix; smaller, cleaner changes |

**Interacting example:** The date parser returns UTC, the report groups by local day, and the export shifts times again. Fixing only the parser breaks the report; fixing only the report hides the export bug. → **Describe all three in one message** so Claude fixes the timezone handling consistently.

**Independent example:** A typo in an error message, a missing index on a table, and a broken link in the README. → **Fix them one at a time.**

---

## 6. Real-world case study

**Situation:** A team asked Claude to "clean up address data". Results changed every run: sometimes postal codes were reformatted, sometimes not; "St." became "Street" in some rows only.

**Iteration:**
1. **Examples:** Added three input → output pairs showing exactly how to treat abbreviations, postal codes, and missing apartment numbers. Consistency improved immediately.
2. **Tests:** Wrote 25 test cases (including null fields and non-US addresses), then iterated by sharing failures. Two rounds later, all passed.
3. **Interview:** Before adding international support, the developer asked Claude to interview them. Claude asked about country-specific formats and whether to store the original string. Both turned out to be important requirements.
4. **Batching:** Three bugs involving the same field-splitting logic were sent **together**; an unrelated logging issue was fixed **separately**.

---

## Common Q&A

**Q: Prose instructions give inconsistent results. What's the most effective fix?**
A: Provide **2–3 concrete input/output examples** that cover different cases.

**Q: What is test-driven iteration with Claude?**
A: Write tests first (behavior, edge cases, performance), have Claude implement, then share the **test failures** as feedback until all pass.

**Q: When should I use the interview pattern?**
A: When the domain is unfamiliar or requirements are unclear, so Claude surfaces important questions (e.g. cache invalidation, failure modes) before implementing.

**Q: I have five bugs. One message or five?**
A: If they **interact**, one detailed message. If they're **independent**, fix them sequentially.

**Q: How do I fix a stubborn edge case?**
A: Give a **specific test case** with the exact input and the expected output.

---

## Key Takeaways

- **Show, don't tell:** 2–3 **input/output examples** beat long prose.
- **Tests first**, then iterate by **sharing failures**. Include edge cases and performance needs.
- Use the **interview pattern** to surface unknowns before building in unfamiliar areas.
- Give **specific test cases** (input + expected output) for edge cases.
- **Interacting issues → one message. Independent issues → one at a time.**
