# 03 · Path-Specific Rules for Conditional Convention Loading

**Exam link:** Task Statement 3.3: *Apply path-specific rules for conditional convention loading.*
**Prerequisite:** [01](01-claude-md-hierarchy.md) (`.claude/rules/`, subdirectory CLAUDE.md).

---

## 1. The problem: conventions that apply to only some files

Real codebases mix many kinds of files, each with its own conventions:

- React components: functional style with hooks
- API handlers: async/await with a specific error-handling pattern
- Database models: repository pattern
- Test files: a specific naming and fixture style, **spread throughout the codebase**
- Terraform: tagging and module rules

If **all** these conventions go in CLAUDE.md, every session loads all of them, even when you're only editing one CSS file. That:

- **wastes tokens** (cost and context space), and
- adds **irrelevant instructions** that can distract Claude or even get applied in the wrong place.

---

## 2. The solution: rules with `paths` frontmatter

A file in `.claude/rules/` can include YAML **frontmatter** with a **`paths`** field containing **glob patterns**. The rule loads **only when Claude works with (reads or edits) files matching those patterns.**

```markdown
---
paths:
  - "terraform/**/*"
---

# Terraform conventions

- Every resource must have `owner`, `cost_center`, and `environment` tags.
- Use modules from modules/ instead of raw resources for VPCs and databases.
- Never hard-code account IDs; use data sources or variables.
```

```markdown
---
paths:
  - "**/*.test.tsx"
  - "**/*.test.ts"
---

# Test file conventions

- Use React Testing Library; query by role, not by test ID.
- Use factories from test/factories/ instead of inline object literals.
- One `describe` per component; test names start with "should".
```

| Rule file | Loads when Claude works on… |
|---|---|
| `.claude/rules/terraform.md` (`terraform/**/*`) | Any file under `terraform/` |
| `.claude/rules/tests.md` (`**/*.test.tsx`) | Any `.test.tsx` file, **in any folder** |
| `.claude/rules/general.md` (no `paths`) | Always, every session |

---

## 3. Glob patterns in 60 seconds

| Pattern | Matches |
|---|---|
| `*` | Any characters within one folder level (not `/`) |
| `**` | Any number of folders (including none) |
| `**/*.ts` | All `.ts` files anywhere |
| `src/**/*` | Everything under `src/` |
| `**/*.test.tsx` | All `.test.tsx` files anywhere |
| `src/api/**/*.ts` | `.ts` files anywhere under `src/api/` |
| `src/**/*.{ts,tsx}` | `.ts` and `.tsx` files under `src/` |
| `migrations/*.sql` | `.sql` files directly in `migrations/` |

---

## 4. Path rules vs subdirectory CLAUDE.md

Both load conditionally. Which should you use?

| | **Subdirectory `CLAUDE.md`** | **Path-scoped rule (`paths:`)** |
|---|---|---|
| Scope | **One directory** (and below) | **Any set of files** matched by glob, across many directories |
| Good for | Conventions tied to **one area** (e.g. `packages/billing/`) | Conventions tied to a **file type** spread everywhere (tests, stories, migrations) |
| Example | Billing package's money rules | Test conventions for `**/*.test.tsx` |

**Why path rules win for spread-out files:** Test files live next to the code they test, in dozens of directories. With subdirectory CLAUDE.md files, you'd need **a copy in every folder**, with duplicated text and drift. A single rule with `paths: ["**/*.test.tsx"]` covers **all of them**, wherever they are.

> **Exam pattern:** "Test files are spread throughout the codebase and must follow the same conventions. What's the most maintainable way to apply them?" → A **`.claude/rules/` file with `paths: ["**/*.test.tsx"]`** (or similar glob), not CLAUDE.md files in every directory and not everything in the root CLAUDE.md.

---

## 5. Designing a rule set: worked example

A full-stack monorepo:

```
.claude/rules/
├── general.md            # no paths: commit style, security basics (always loaded)
├── react-components.md   # paths: ["src/components/**/*.tsx"]
├── api-handlers.md       # paths: ["src/api/**/*.ts"]
├── db-models.md          # paths: ["src/models/**/*.ts"]
├── tests.md              # paths: ["**/*.test.ts", "**/*.test.tsx"]
└── terraform.md          # paths: ["terraform/**/*"]
```

- Editing `src/api/orders.ts` → loads `general.md` + `api-handlers.md`.
- Editing `src/api/orders.test.ts` → loads `general.md` + `api-handlers.md` + `tests.md`.
- Editing `terraform/vpc/main.tf` → loads `general.md` + `terraform.md`.

Each session gets **only the relevant conventions**.

---

## 6. Real-world case study

**Situation:** A SaaS company's root CLAUDE.md had 400 lines: React, API, database, testing, and Terraform conventions all together.

**Symptoms:**
- Token usage per session was high, even for tiny changes.
- Claude occasionally applied the database "repository pattern" advice to React components.
- Someone added `CLAUDE.md` files to 14 test folders to explain testing rules. They drifted apart within weeks.

**Changes:**
1. Root CLAUDE.md cut to ~60 lines of universal content.
2. Each convention group moved to a **path-scoped rule** (as in §5).
3. The 14 test-folder CLAUDE.md files were deleted and replaced by **one** `tests.md` rule with `paths: ["**/*.test.ts", "**/*.test.tsx"]`.

**Result:** Lower context usage per session, no more cross-area convention mix-ups, and one place to maintain test rules.

---

## Common Q&A

**Q: When does a path-scoped rule load?**
A: When Claude reads or edits a file that matches one of its `paths` glob patterns. Rules without `paths` load every session.

**Q: Why not just put everything in CLAUDE.md?**
A: Everything in CLAUDE.md loads every time. Path rules load only when relevant, which saves tokens and reduces irrelevant instructions.

**Q: Test files are in 30 different folders. Subdirectory CLAUDE.md or path rule?**
A: A **path rule** with a glob like `**/*.test.tsx`. One file, applies everywhere, no duplication.

**Q: When is a subdirectory CLAUDE.md still the better choice?**
A: When the conventions belong to **one directory/area** (e.g. a specific package) rather than a file type spread around the repo.

**Q: Can a rule have several patterns?**
A: Yes. `paths` is a list, e.g. `["**/*.test.ts", "**/*.test.tsx"]`.

---

## Key Takeaways

- `.claude/rules/` files with **`paths:` glob patterns** load **only for matching files**.
- This **saves tokens** and keeps **irrelevant conventions** out of context.
- Use globs to apply rules **by file type regardless of location** (`**/*.test.tsx`).
- Prefer **path rules over subdirectory CLAUDE.md** when files are **spread across the codebase**.
- Keep root CLAUDE.md for **universal** content only.
