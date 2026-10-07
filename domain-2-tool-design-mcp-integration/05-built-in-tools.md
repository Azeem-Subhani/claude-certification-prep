# 05 · Built-in Tools: Read, Write, Edit, Bash, Grep, Glob

**Exam link:** Task Statement 2.5: *Select and apply built-in tools effectively.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (where tools come from).

---

## 1. The six core built-in tools

Claude Code and the Agent SDK include built-in tools for working with files and the shell.

| Tool | What it does | Think of it as |
|---|---|---|
| **Grep** | Searches **file contents** for a text pattern (regex) | "Which files *contain* this text?" |
| **Glob** | Finds files whose **paths/names** match a pattern | "Which files are *named* like this?" |
| **Read** | Loads the **full contents** of a file | "Show me this file" |
| **Write** | Creates a file or **replaces a whole file** | "Save this as the entire file" |
| **Edit** | Replaces a **specific, unique piece of text** in a file | "Change exactly this part" |
| **Bash** | Runs a shell command | "Run tests, git, build tools" |

Choosing the right one makes the agent **faster**, **cheaper**, and **more accurate**.

---

## 2. Grep vs Glob: content vs path

This is the most tested distinction.

| Question | Tool | Example |
|---|---|---|
| Where is `processPayment` called? | **Grep** | pattern `processPayment\(` |
| Where does the error "card declined" come from? | **Grep** | pattern `card declined` |
| Which files import `stripe`? | **Grep** | pattern `from ['"]stripe['"]` |
| Find all test files for React components | **Glob** | `**/*.test.tsx` |
| Find every `Dockerfile` | **Glob** | `**/Dockerfile` |
| List all migration files | **Glob** | `db/migrations/*.sql` |

**Memory trick:** **G**rep looks **inside** files (text). **Gl**ob looks at **file names** (paths).

---

## 3. Read, Write, Edit

### Edit: targeted changes with a unique match

**Edit** finds an exact piece of text (`old_string`) and replaces it with new text. The text must be **unique** in the file, so Edit knows exactly *where* to change.

```text
Edit(file_path="src/payment.py",
     old_string="TIMEOUT_SECONDS = 10",
     new_string="TIMEOUT_SECONDS = 30")
```

**Advantages:** small, precise, and less risk of accidentally changing other parts of the file.

### When Edit fails: non-unique matches

If `old_string` appears **more than once**, Edit can't know which one to change, so it **fails**.

**Example:** A config file has `enabled: true` in five sections. `Edit(old_string="enabled: true", ...)` fails.

**Options:**

1. **Add more surrounding context** to make the match unique (include the section name or neighboring lines).
2. **Fallback: Read + Write.** Read the full file, make the change in the full content, and Write the whole file back. This is reliable when there's no unique anchor text.

> **Exam pattern:** "Edit keeps failing because the anchor text isn't unique." The answer: **Read the full file, then Write** the modified version (or include more context). Not "use Bash with `sed`" (fragile) and not "give up".

### Read and Write

- **Read** loads the whole file. Use it to understand code before changing it, or as the first half of the Read + Write fallback.
- **Write** replaces the **entire file**. Use it for **new files** or **full rewrites**. For small changes to existing files, prefer Edit. (In Claude Code, the agent must Read an existing file before it can overwrite it with Write.)

---

## 4. Bash

**Bash** runs shell commands: tests, builds, git, package managers, and project scripts.

Good uses:
- `pytest tests/payment/` to run tests
- `git log --oneline -5` to see recent changes
- `npm run lint`

**Prefer dedicated tools when one fits:**

| Task | Prefer | Instead of |
|---|---|---|
| Search file contents | Grep | `bash grep -r ...` |
| Find files by name | Glob | `bash find . -name ...` |
| Read a file | Read | `bash cat ...` |
| Change part of a file | Edit | `bash sed -i ...` |

Dedicated tools give **structured results**, respect **permissions** more precisely, and are **easier to review**. Bash is powerful, so it is also the riskiest tool to grant. Restrict it where possible.

---

## 5. Build understanding incrementally

When exploring an unfamiliar codebase, **don't read every file up front**. That wastes tokens and fills the context with irrelevant code (attention dilution).

**Incremental strategy:**

```
1. Grep  → find the entry point        ("def handle_checkout" / route "/checkout")
2. Read  → read that file
3. Follow imports → Read only the modules the flow actually uses
4. Grep  → find specific details as questions come up ("where is TAX_RATE set?")
```

**Example: "How does checkout calculate tax?"**

1. `Grep "checkout"` in `routes/` → `routes/checkout.py`
2. `Read routes/checkout.py` → sees `from services.pricing import calculate_total`
3. `Read services/pricing.py` → sees `tax = tax_service.rate_for(region)`
4. `Grep "def rate_for"` → `services/tax_service.py`
5. `Read services/tax_service.py` → answer found

Five targeted steps instead of reading hundreds of files.

---

## 6. Tracing usage across wrapper modules

Large codebases often **wrap** functions and **re-export** them under other names:

```python
# payments/core.py
def charge_card(card_token, amount): ...

# payments/__init__.py
from .core import charge_card as charge          # re-exported under a new name
from .core import charge_card as legacy_charge   # and again for old callers
```

If you `Grep "charge_card"`, you **miss** every caller that uses `charge` or `legacy_charge`.

**Correct approach:**

1. **Identify all exported names** first: read the wrapper module (`payments/__init__.py`) to find every alias (`charge`, `legacy_charge`).
2. **Search for each name** across the codebase: `Grep "charge_card"`, `Grep "\bcharge\("`, `Grep "legacy_charge"`.
3. Combine the results for the complete list of usages.

> If a semantic code-search MCP tool is available (see [04](04-mcp-server-integration.md)), it may resolve aliases automatically. Built-in Grep only matches text.

---

## 7. Real-world case study

**Situation:** A developer asked an agent to "rename the `getUserProfile` function everywhere".

**What went wrong (first attempt):**
- The agent **Read** 120 files one by one looking for usages, which was slow and costly.
- It used **Edit** with `old_string="getUserProfile("`, which appeared several times in one file, so Edit failed repeatedly.
- It missed callers that imported the function as `fetchProfile` through an `index.ts` re-export.

**Improved approach:**
1. `Read src/users/index.ts` → found the alias `fetchProfile`.
2. `Grep "getUserProfile"` and `Grep "fetchProfile"` → full list of 14 files.
3. For each file: **Edit** with enough surrounding context to be unique. Where that wasn't possible, **Read + Write** the file.
4. `Bash: npm test` to confirm nothing broke.

---

## Common Q&A

**Q: I need all files ending in `.spec.ts`. Grep or Glob?**
A: **Glob** (`**/*.spec.ts`). You're matching file **names**, not contents.

**Q: I need every place that calls `sendInvoice()`. Grep or Glob?**
A: **Grep**. You're searching file **contents**.

**Q: Edit failed because the text appears three times. What now?**
A: Include more surrounding text to make it unique, or **Read** the whole file and **Write** the updated version.

**Q: Why not just use Write for every change?**
A: Write replaces the whole file, which risks accidental changes and makes reviews harder. Edit is safer for small, targeted changes.

**Q: Should the agent read the whole codebase before answering a question about it?**
A: No. **Grep** for entry points, then **Read** only what the flow needs, following imports.

**Q: Grep found no callers of a function, but the app clearly uses it. Why?**
A: It may be **re-exported or aliased** through a wrapper module. Find all exported names first, then search for each.

**Q: Can't Bash do all of this?**
A: It can, but dedicated tools are clearer, safer, and easier to permission and review. Use Bash for running commands (tests, builds, git).

---

## Key Takeaways

- **Grep = contents** (callers, error messages, imports). **Glob = paths** (file names, extensions).
- **Edit** = targeted change with a **unique** match. If not unique → add context or **Read + Write**.
- **Write** replaces whole files; **Read** loads whole files.
- **Bash** runs commands; prefer dedicated tools for search, read, and edit.
- Explore **incrementally**: Grep → Read → follow imports, not "read everything".
- For wrapped/re-exported functions: **find all exported names first**, then search for each.
