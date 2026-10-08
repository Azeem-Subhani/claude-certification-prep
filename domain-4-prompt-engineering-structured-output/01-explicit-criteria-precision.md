# 01 · Explicit Criteria to Improve Precision and Reduce False Positives

**Exam link:** Task Statement 4.1: *Design prompts with explicit criteria to improve precision and reduce false positives.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (precision, false positives, trust).

---

## 1. Vague instructions produce noisy output

Compare two instructions for a code review prompt:

| Vague | Explicit |
|---|---|
| "Check that comments are accurate." | "Flag a comment **only when the behavior it claims contradicts what the code actually does** (e.g. the comment says 'returns null on failure' but the code throws)." |

With the vague version, Claude flags typos in comments, outdated wording, missing comments, and "could be clearer" suggestions. Most of that is noise. With the explicit version, it flags only **real contradictions**, which are the cases that can mislead developers and cause bugs.

> **Explicit criteria = clear rules about which categories of issues count.** They work much better than general instructions.

---

## 2. Why "be conservative" doesn't work

A common reaction to noisy output is to add:

- "Be conservative."
- "Only report high-confidence findings."
- "Don't report minor issues."

These **sound** helpful but don't reliably improve precision, because:

- **"Conservative" and "minor" are undefined.** The model has to guess where the line is, and it guesses differently each time.
- **Model confidence is not well calibrated.** A model can be confidently wrong. Filtering by self-reported confidence still lets through confident false positives and drops real but "uncertain-looking" issues.
- **They don't change *what* is being looked for.** The model still checks every category; it just reports a bit less of everything.

**Specific categorical criteria** fix the root cause: they define *what kind of thing* is worth reporting.

---

## 3. Writing categorical review criteria

Define **what to report** and **what to skip**, by category:

```text
REPORT these categories:
1. Bugs: logic errors, off-by-one, null/None dereferences, wrong conditions,
   unhandled error paths that can crash or corrupt data.
2. Security: injection (SQL, shell, template), missing authentication/authorization
   checks, secrets in code, unsafe deserialization.
3. Data integrity: money stored as floats, missing transactions around multi-step writes.
4. Comment/code contradictions: a comment claims behavior the code does not have.

DO NOT REPORT:
- Formatting, whitespace, import order (handled by the linter).
- Naming preferences, unless a name is actively misleading.
- Local patterns that are consistent with the surrounding file, even if you would
  write them differently.
- Suggestions for "nicer" code with no correctness or security impact.
```

**Why this works:** The model no longer has to decide what "important" means. Every potential finding is checked against a clear list: is it in a REPORT category? If not, skip it.

---

## 4. Disable high false-positive categories (temporarily)

Sometimes one category is consistently noisy. For example, "performance suggestions" have a 70% dismissal rate, while "security" findings are 90% accurate.

**Problem:** The noisy category **damages trust in the accurate ones**. Developers see many bad performance comments and start skipping *all* bot comments, including real security issues.

**Strategy:**

1. **Temporarily disable** the noisy category (remove it from the REPORT list).
2. **Restore trust**: the remaining findings are now mostly accurate, so developers pay attention again.
3. **Improve the disabled category offline**: tighter criteria, few-shot examples ([02](02-few-shot-prompting.md)), measure on a test set.
4. **Re-enable** it only when its precision is acceptable.

```text
Category status (example)
- security          ✅ enabled   (precision 0.91)
- bugs              ✅ enabled   (precision 0.84)
- performance       ⏸ disabled  (precision 0.30, being reworked)
- comment accuracy  ✅ enabled   (precision 0.88, after explicit criteria)
```

> **Exam pattern:** "Developers ignore the bot because of noisy findings in one category, while other categories are accurate." → **Disable the noisy category temporarily** and fix its prompt. Not "add 'be conservative'" and not "raise the confidence threshold for everything".

---

## 5. Severity criteria with concrete code examples

Even with good categories, **severity** can be inconsistent: the same issue is "critical" in one PR and "minor" in another.

**Fix:** define each severity level **with concrete code examples**:

````text
SEVERITY LEVELS

critical: exploitable security issue or data loss/corruption in production.
  Example:
    query = f"SELECT * FROM users WHERE id = {user_id}"   # SQL injection
    → critical

major: bug that causes wrong results or crashes in realistic use.
  Example:
    for i in range(len(items) - 1):   # skips the last item
        process(items[i])
    → major

minor: correctness issue in rare edge cases, with low impact.
  Example:
    avg = total / count   # count can be 0 only if the input list is empty, which callers already prevent
    → minor
````

**Why examples work:** "Critical" in words is open to interpretation; an example pins it down. The model compares new findings with the examples and classifies them consistently.

---

## 6. Real-world case study

**Situation:** A company rolled out an AI code reviewer. After one month:
- Average 14 comments per PR, with a 75% dismissal rate.
- A real SQL injection finding was ignored because "the bot is always wrong".

**Attempt 1:** Added "Only report high-confidence issues. Be conservative." Comments dropped to 9 per PR, but the dismissal rate barely changed (~70%).

**Attempt 2 (worked):**
1. Rewrote the prompt with **categorical REPORT / DO NOT REPORT** lists.
2. **Disabled** the "performance" and "readability" categories (the noisiest).
3. Added **severity definitions with code examples**.

**Result:** 3 comments per PR, dismissal rate ~15%, and developers started acting on findings again. Performance checks were re-enabled two months later after improvement with few-shot examples.

---

## Common Q&A

**Q: Why doesn't "only report high-confidence findings" fix false positives?**
A: Confidence is a poor filter. The model can be confidently wrong, and the instruction doesn't define *which kinds* of issues matter. Categorical criteria do.

**Q: Isn't disabling a category risky? We'll miss issues.**
A: Temporarily, yes, but a noisy category is already being ignored by developers. Disabling it restores trust in the accurate categories while you improve it.

**Q: How specific should criteria be?**
A: Specific enough that two different reviewers would agree whether a finding belongs in a category. Name concrete patterns (e.g. "comment claims behavior the code doesn't have").

**Q: Why include code examples in severity definitions?**
A: Words like "critical" are interpreted differently each time. Examples anchor the meaning and make classification consistent.

**Q: Should "skip" criteria be included, or only "report" criteria?**
A: Both. Explicitly listing what to skip (style, local patterns) is one of the strongest ways to cut false positives.

---

## Key Takeaways

- **Explicit, categorical criteria** beat vague instructions for precision.
- "**Be conservative**" or "**high-confidence only**" don't reliably reduce false positives.
- List both **what to report** (bugs, security) and **what to skip** (style, local patterns).
- **Noisy categories destroy trust** in accurate ones. **Disable them temporarily**, fix, then re-enable.
- Define **severity levels with concrete code examples** for consistent classification.
