# 04 · Integrating MCP Servers into Claude Code and Agent Workflows

**Exam link:** Task Statement 2.4: *Integrate MCP servers into Claude Code and agent workflows.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (MCP host/client/server, tools vs resources).

---

## 1. Where MCP servers are configured

Claude Code reads MCP server configuration from different places, called **scopes**. The exam focuses on two:

| Scope | File | Shared? | Use for |
|---|---|---|---|
| **Project** | **`.mcp.json`** in the project root, **committed to git** | Yes, every team member who clones the repo | **Shared team tooling** (GitHub, Jira, the team's database server) |
| **User** | **`~/.claude.json`** in your home directory | No, only you | **Personal or experimental** servers you're trying out |

Current Claude Code docs also describe a **local** scope (the default for `claude mcp add`). It is stored in `~/.claude.json` but applies only to the current project, and isn't shared.

Adding servers from the command line:

```bash
# Shared with the team: written to .mcp.json
claude mcp add --scope project --transport http github https://api.githubcopilot.com/mcp/

# Personal, available in all your projects: written to ~/.claude.json
claude mcp add --scope user my-experiment -- python ~/tools/experiment_server.py
```

**Decision rule:**
- "Every developer on the team needs this" → **project `.mcp.json`**
- "I'm testing this" or "only I use this" → **user `~/.claude.json`**

> **Exam pattern:** A new teammate clones the repo and doesn't have the team's Jira tools. Likely cause: the server was configured in someone's **user** config (`~/.claude.json`), not in the project's **`.mcp.json`**.

---

## 2. Keep secrets out of the repo: environment variable expansion

`.mcp.json` is committed to version control, so it must **never contain real tokens**. Use **environment variable expansion**:

```json
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${GITHUB_TOKEN}"
      }
    },
    "internal-api": {
      "type": "http",
      "url": "${INTERNAL_API_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${INTERNAL_API_TOKEN}"
      }
    }
  }
}
```

- Server URLs and package names change over time; take them from the vendor's current docs.
- `${GITHUB_TOKEN}` is replaced at runtime with the value from **each developer's own environment**.
- `${VAR:-default}` uses a default if the variable isn't set.
- Expansion works in `command`, `args`, `env`, `url`, and `headers`.

**Result:** the config is shared, but **each person's credentials stay private**. Nobody commits a token.

**Insecure pattern to flag:**

```jsonc
"headers": { "Authorization": "Bearer ghp_xxxxxxxxxxxxxxxxxxxx" }   // ❌ secret in a committed file
```

> **Security note:** Claude Code asks you to **approve** project-scoped servers from `.mcp.json` before using them in interactive sessions, because a cloned repo could contain an untrusted server. Review what a server does before approving it.

---

## 3. All servers' tools are available together

When Claude Code (or an Agent SDK app) starts, it **discovers the tools of all configured MCP servers** and makes them available **at the same time**, alongside the built-in tools.

Consequences:

- Tool names must be distinguishable. They're namespaced as `mcp__<server>__<tool>`.
- **Descriptions compete with each other**, and with built-in tools (see §4).
- Many servers = many tools → selection gets harder (see [03](03-tool-distribution-and-tool-choice.md)). Only connect servers the work needs.

---

## 4. Write MCP tool descriptions that beat weaker built-ins

A common problem: you add a powerful MCP tool, but the agent keeps using a **built-in** tool instead.

**Example:** Your team builds `mcp__codeindex__find_references`, which uses a real code index to find every reference to a symbol, including across renamed imports. But the agent keeps using **Grep**, which only finds text matches.

**Why?** The MCP tool's description was minimal ("Finds references"). Grep is familiar and its purpose is obvious, so the model prefers it.

**Fix: explain capabilities and outputs in detail:**

```text
find_references: Find ALL references to a function, class, or variable across the codebase using
the semantic code index. Unlike text search (Grep), this resolves imports, aliases, and re-exports,
so it finds usages even when the symbol is renamed on import. Use this instead of Grep when you need
every caller of a symbol (e.g. before renaming or changing a signature).
Input: symbol name, optional file path to disambiguate.
Output: list of {file, line, kind: call|import|definition, enclosing_function}.
```

Now the model understands **when** the MCP tool is better than Grep.

---

## 5. Use community servers for standard integrations

Before building an MCP server, check whether a **maintained community or vendor server** already exists.

| Integration | Recommendation |
|---|---|
| GitHub, Jira, Slack, Postgres, Google Drive, etc. | **Use an existing server.** It's faster, already tested, and maintained by others |
| A team-specific workflow (e.g. your internal deployment approval process, a proprietary pricing engine) | **Build a custom server** |

**Before adopting any external server**, check it is **actively maintained**, from a **trusted source**, and **pinned to a version**, and review the permissions it needs. An MCP server runs code and can access data, so treat it like any dependency.

---

## 6. MCP resources: show what exists, without exploring

**Problem:** An agent needs to know *what is available* before it can act. Without help, it makes many **exploratory tool calls**:

```
list_tables → describe_table("orders") → describe_table("customers") → describe_table("payments") → ...
```

Each call costs a turn, tokens, and time.

**Solution:** expose a **content catalog** as an **MCP resource**. A resource is read-only content with a URI that the application can load into context.

| Catalog | Resource example |
|---|---|
| Database schema | `db://schema`: all tables, columns, types, relationships |
| Issue tracker summary | `jira://project/PAY/open-issues`: titles, IDs, status |
| Documentation hierarchy | `docs://index`: section tree with short summaries |

With the catalog in context, the agent **knows what exists** and goes **straight to the right tool call**:

```
(schema resource already in context) → run_query("SELECT ... FROM payments JOIN customers ...")
```

In Claude Code, users can reference MCP resources with **@ mentions** (in the format `@server:protocol://resource/path`) to pull them into the conversation.

**Rule of thumb:**
- **Resources** = *read* catalogs and reference content (what exists).
- **Tools** = *actions* and targeted queries (do something, get specific data).

---

## 7. Real-world case study

**Situation:** A fintech team adopted Claude Code across 25 developers.

**Problems:**
1. One engineer configured the Jira and Postgres servers in their own `~/.claude.json`. Nobody else had them.
2. Another engineer committed `.mcp.json` with a real database password.
3. The agent spent 6–10 tool calls exploring tables before writing any query.
4. The team was writing a custom Jira MCP server from scratch.

**Fixes:**
1. Moved Jira and Postgres to the project **`.mcp.json`**, committed for the whole team.
2. Rotated the leaked password immediately, replaced it with **`${DB_PASSWORD}`**, and added secret scanning to CI.
3. Exposed the **database schema as an MCP resource**. Exploratory calls dropped to near zero.
4. Stopped the custom Jira build and adopted a **maintained Jira server**. A **custom** server was kept only for their internal deployment-approval workflow.

---

## Common Q&A

**Q: Where should a server that the whole team needs go?**
A: The project's **`.mcp.json`**, committed to version control.

**Q: Where should I put a server I'm just experimenting with?**
A: Your **user-level `~/.claude.json`** (e.g. `claude mcp add --scope user ...`). It won't affect teammates.

**Q: How do I share a server config that needs a token?**
A: Commit `.mcp.json` with **`${TOKEN_NAME}`** placeholders; each developer sets the variable in their own environment.

**Q: Why does the agent use Grep instead of our better MCP search tool?**
A: Usually because the MCP tool's **description** doesn't explain its capabilities and when it's better. Improve the description.

**Q: Should we build our own MCP server for GitHub?**
A: Usually no. Use a maintained existing server. Build custom servers for **team-specific** workflows.

**Q: When is an MCP resource better than a tool?**
A: When the agent needs to **see what's available** (schemas, catalogs, indexes). A resource avoids many exploratory tool calls.

**Q: If I configure three MCP servers, does the agent have to pick one server first?**
A: No. Tools from **all** configured servers are discovered at connection time and are available **together**.

---

## Key Takeaways

- **Project `.mcp.json`** = shared team servers (committed). **User `~/.claude.json`** = personal/experimental servers.
- Use **`${ENV_VAR}` expansion** for credentials. Never commit secrets.
- Tools from **all configured servers are available at the same time**, so descriptions compete.
- Write **detailed MCP tool descriptions** so the agent doesn't fall back to weaker built-ins like Grep.
- Prefer **existing community servers** for standard systems; build **custom** ones for team-specific workflows.
- Expose **content catalogs as MCP resources** to cut exploratory tool calls.
