# 06 · Integrating Claude Code into CI/CD Pipelines

**Exam link:** Task Statement 3.6: *Integrate Claude Code into CI/CD pipelines.*
**Prerequisites:** [01](01-claude-md-hierarchy.md) (CLAUDE.md), Domain 1 [07 · Sessions](../domain-1-agentic-architecture-orchestration/07-session-state-resumption-forking.md).

---

## 1. What changes in CI?

**CI/CD** (Continuous Integration / Continuous Deployment) pipelines run automatically on events like a pull request. Typical Claude Code jobs:

- Automated **code review** of pull requests
- **Test generation** for changed code
- **Summaries** or release notes

CI is different from interactive use:

| Interactive | CI |
|---|---|
| A human answers questions and approves actions | **Nobody is there** to answer |
| Output is read by a person | Output is read by **a script** (e.g. to post PR comments) |
| You can fix mistakes in conversation | Each run must be **reliable and repeatable** |

---

## 2. `-p` (`--print`): non-interactive mode

By default, `claude` starts an **interactive** session and waits for input. In a pipeline that means the job **hangs forever**.

**Fix:** use `-p` (or `--print`) to run **non-interactively**: Claude takes the prompt, does the work, prints the result, and exits.

```bash
# ❌ Hangs in CI: waits for interactive input
claude "Analyze this pull request for security issues"

# ✅ Runs once and exits
claude -p "Analyze this pull request for security issues"
```

> **Exam pattern:** "The pipeline job hangs; logs show Claude Code waiting for interactive input." → Add **`-p`**.

Also restrict what the run can do. For example, pre-approve only the tools it needs with `--allowedTools`, so it doesn't stop for approvals it can't get.

---

## 3. Structured output: `--output-format json` and `--json-schema`

A script can't reliably parse free text like "I found a few issues, the first one is…". For automation, request **structured output**:

- **`--output-format json`**: returns JSON with the result plus metadata (session ID, cost, etc.).
- **`--json-schema '<schema>'`**: makes the output **conform to a JSON Schema** you define. The structured data appears in a `structured_output` field.

```bash
claude -p "Review the diff in pr.diff. Report only real problems." \
  --output-format json \
  --json-schema '{
    "type": "object",
    "properties": {
      "findings": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "file": {"type": "string"},
            "line": {"type": "integer"},
            "severity": {"type": "string", "enum": ["blocker", "major", "minor"]},
            "message": {"type": "string"}
          },
          "required": ["file", "line", "severity", "message"]
        }
      }
    },
    "required": ["findings"]
  }' > review.json

# Each finding can now be posted as an inline PR comment at file:line
jq -c '.structured_output.findings[]' review.json
```

**Why:** Every finding has `file`, `line`, `severity`, `message`, so the pipeline can post **inline PR comments** automatically with no fragile text parsing.

---

## 4. CLAUDE.md gives CI runs project context

A CI run starts with **no conversation**. How does it know your review criteria or testing standards? Through **CLAUDE.md** (and rules files), which Claude Code loads in CI too.

Put CI-relevant knowledge there:

```markdown
## Review criteria
- Flag: missing input validation, SQL built by string concatenation, secrets in code.
- Don't flag: formatting (handled by the linter), naming preferences.

## Testing standards
- Unit tests use pytest; integration tests live in tests/integration/.
- Valuable tests cover: error paths, boundary values, permission checks.
- Low-value tests to avoid: testing getters/setters, re-testing framework behavior.
- Available fixtures: `db_session`, `api_client`, `make_user()`, `make_order()` in tests/conftest.py.
```

Documenting **testing standards, what makes a test valuable, and available fixtures** improves generated tests and **reduces low-value test output**.

> **Current product note:** The Claude Code docs recommend `--bare` for scripted runs because it starts faster and behaves the same on every machine. But `--bare` **skips CLAUDE.md**, hooks, skills, and MCP auto-discovery. If your CI relies on CLAUDE.md for context (as the exam assumes), either don't use `--bare`, or pass the context explicitly (e.g. `--append-system-prompt-file`).

---

## 5. Use an independent reviewer, not the author session

If the **same Claude session** that wrote the code also reviews it, the review is weaker. The session already "believes" its own reasoning and assumptions are correct, so it tends to confirm them rather than question them.

**Better:** run the review in a **separate, independent instance**: a fresh `claude -p` run with no memory of how the code was written. Give it only the diff, the context files, and the review criteria.

```
❌ Session A writes the feature → Session A reviews it
✅ Session A writes the feature → new Session B (fresh context) reviews the diff
```

This is like human code review: a second pair of eyes catches what the author missed.

---

## 6. Re-reviews after new commits: no duplicate comments

A PR gets reviewed, the developer pushes fixes, and the pipeline reviews again. If the second run doesn't know about the first, it **re-posts the same comments**, which is noisy and annoying.

**Fix:** include **prior review findings** in the new run's context and instruct Claude to report **only new or still-unaddressed issues**:

```bash
claude -p "Review the latest diff.
Previous findings are in prior_findings.json.
- Do NOT repeat findings that have been fixed.
- Report findings that are still unaddressed, marked status: \"still_open\".
- Report new problems, marked status: \"new\"." \
  --output-format json --json-schema "$(cat review_schema.json)"
```

---

## 7. Test generation: provide existing tests

When asking Claude to generate tests for changed code, give it the **existing test files** for that code. Otherwise it may suggest tests for scenarios **already covered**, creating duplicates.

```bash
claude -p "Generate tests for the changes in src/billing/invoice.py.
Existing tests are in tests/billing/test_invoice.py. Only add scenarios not already covered."
```

Combined with testing standards in CLAUDE.md (§4), this produces **fewer, more valuable** tests.

---

## 8. Putting it together: example GitHub Actions step

```yaml
- name: Claude review
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}   # from CI secrets, never committed
  run: |
    git diff origin/main...HEAD > pr.diff
    claude -p "Review pr.diff using the review criteria in CLAUDE.md.
               Prior findings: prior_findings.json. Report only new or unaddressed issues." \
      --allowedTools "Read" \
      --output-format json \
      --json-schema "$(cat .github/review_schema.json)" > review.json
    python .github/scripts/post_inline_comments.py review.json
```

Checklist covered: `-p` (no hang), restricted tools, structured output, CLAUDE.md context, independent run, de-duplication, and the secret kept in CI secrets.

---

## 9. Real-world case study

**Situation:** A fintech team added Claude review to CI. Week one:
1. Jobs **hung** until the 6-hour CI timeout.
2. A script tried to parse free-text reviews with regex and broke constantly.
3. After every push, the same 12 comments were reposted.
4. Generated tests re-tested things already covered and ignored their fixtures.

**Fixes:** `-p` mode. `--output-format json` + `--json-schema` with file/line/severity. Prior findings passed in with "only new or unaddressed". Existing tests provided. Fixtures and testing standards documented in CLAUDE.md. Review run in a fresh instance separate from the code-generation job.

---

## Common Q&A

**Q: The CI job hangs. Why?**
A: Claude Code is waiting for interactive input. Run it with **`-p`**.

**Q: How do I get output my pipeline can parse reliably?**
A: `--output-format json` plus `--json-schema` with the exact fields you need (e.g. file, line, severity, message).

**Q: How does a CI run know our standards?**
A: From **CLAUDE.md** (and rules) in the repo: review criteria, testing standards, fixtures.

**Q: Why not let the same session that generated the code review it?**
A: It's biased toward its own reasoning. An **independent instance** with fresh context reviews more effectively.

**Q: How do I stop duplicate comments on re-review?**
A: Pass prior findings into context and tell Claude to report **only new or still-unaddressed** issues.

**Q: Generated tests duplicate existing ones. Fix?**
A: Provide the **existing test files** in context, and document valuable-test criteria and fixtures in CLAUDE.md.

---

## Key Takeaways

- **`-p`** runs Claude Code non-interactively. Without it, CI jobs **hang**.
- **`--output-format json` + `--json-schema`** give machine-parseable findings for **inline PR comments**.
- **CLAUDE.md** carries project context into CI: review criteria, testing standards, fixtures. (Beware: `--bare` skips it.)
- Review with an **independent instance**, not the session that wrote the code.
- On re-review, include **prior findings** and report **only new/unaddressed** issues.
- For test generation, provide **existing tests** to avoid duplicates.
