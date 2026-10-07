# 00 · Foundations and Glossary (Domain 2)

> **Read this first.** This file explains the basic parts of a tool and of MCP. Later files in this folder use these terms without defining them again.
> Basic agent terms (agent, agentic loop, `tool_use`, `tool_result`, harness, deterministic vs probabilistic) are defined in [Domain 1 · 00](../domain-1-agentic-architecture-orchestration/00-foundations-and-glossary.md).

**Exam link:** Background for all of Domain 2 (18% of the exam).

---

## 1. Why tool design matters

In Domain 1 you learned *how* an agent uses tools in a loop. Domain 2 is about the **tools themselves**:

- How the model **chooses** the right tool.
- What a tool should **return**, especially when something goes wrong.
- **Which** tools each agent should have.
- How to **connect** tools through MCP.
- How to use Claude Code's **built-in** tools well.

A great model with badly designed tools makes bad choices. Most "the agent called the wrong tool" problems are **tool design problems**, not model problems.

---

## 2. Anatomy of a tool definition

Every tool, whether you define it in the Claude API or an MCP server exposes it, has the same core parts:

| Part | What it is | Who reads it |
|---|---|---|
| **Name** | Short identifier, e.g. `lookup_order` | The model (as a hint) and your code (to route the call) |
| **Description** | Plain-language text: what the tool does, when to use it, what it returns | **The model**: this is its main guide for choosing a tool |
| **Input schema** | JSON Schema listing parameters, types, required fields | The model (to build valid arguments) and the API/your code (to validate) |
| **Output** | What the tool returns: success data or an error | The model, on the next loop iteration |

```json
{
  "name": "lookup_order",
  "description": "Get the status, items, and payment history of ONE order by its order number (format: ORD- followed by 6 digits, e.g. ORD-123456). Use this when the user mentions a specific order. Do NOT use it to find a customer's account details; use get_customer for that. Returns: status, items, charges, shipping dates (ISO 8601).",
  "input_schema": {
    "type": "object",
    "properties": {
      "order_id": {"type": "string", "description": "Order number, e.g. ORD-123456"}
    },
    "required": ["order_id"]
  }
}
```

> In the Claude API the schema field is `input_schema`. In the MCP specification it is `inputSchema`. Same idea, different spelling.

---

## 3. What is MCP?

The **Model Context Protocol (MCP)** is an open standard for connecting AI applications to tools and data. Think of it as a "USB port for AI": one standard plug, many devices.

Without MCP, every AI app needs custom code for every system (Jira, GitHub, a database…). With MCP, a system is wrapped **once** as an **MCP server**, and any MCP-compatible app can use it.

### The three roles

```
┌───────────────────────────── MCP HOST ─────────────────────────────┐
│  (the AI application: Claude Code, an Agent SDK app, Claude Desktop)│
│                                                                      │
│   ┌────────────┐      ┌────────────┐      ┌────────────┐            │
│   │ MCP client │      │ MCP client │      │ MCP client │            │
│   └─────┬──────┘      └─────┬──────┘      └─────┬──────┘            │
└─────────┼───────────────────┼───────────────────┼──────────────────┘
          │                   │                   │
   ┌──────▼──────┐     ┌──────▼──────┐     ┌──────▼──────┐
   │ MCP server  │     │ MCP server  │     │ MCP server  │
   │  (GitHub)   │     │   (Jira)    │     │ (Postgres)  │
   └─────────────┘     └─────────────┘     └─────────────┘
```

| Role | Meaning |
|---|---|
| **Host** | The AI application the user works in |
| **Client** | The connector inside the host; one per server |
| **Server** | A program that exposes tools, resources, and prompts for one system |

### Transports (how host and server talk)

| Transport | How it works | Typical use |
|---|---|---|
| **stdio** | The host starts the server as a **local process** and talks through standard input/output | Local tools, scripts, file access |
| **HTTP** (streamable HTTP) | The server runs **remotely**; the host connects over HTTPS | Shared or SaaS services |

---

## 4. Tools vs resources vs prompts

An MCP server can expose three kinds of things:

| Primitive | What it is | Who decides to use it | Example |
|---|---|---|---|
| **Tools** | **Actions** the model can call (may change things) | The **model** | `create_issue`, `run_query`, `process_refund` |
| **Resources** | **Read-only content** the app can load into context, identified by a URI | The **application / user** | A list of open issues, a database schema, a docs index |
| **Prompts** | **Reusable prompt templates** the user can pick | The **user** | "Summarize this PR", "Write release notes" |

> **Exam-relevant idea:** If the agent keeps making many small "exploratory" tool calls just to discover *what exists* (list tables, list docs, list issues), expose that catalog as a **resource** instead. See [04](04-mcp-server-integration.md).

---

## 5. Where tools come from in Claude Code and the Agent SDK

| Source | Examples | File in this folder |
|---|---|---|
| **Built-in tools** | Read, Write, Edit, Bash, Grep, Glob, WebSearch, WebFetch | [05](05-built-in-tools.md) |
| **MCP tools** | Any tool from a connected MCP server, named `mcp__<server>__<tool>` | [04](04-mcp-server-integration.md) |
| **Custom tools** | Functions you define in your own app (Claude API or Agent SDK) | [01](01-tool-interface-design.md) |

---

## 6. The five Domain 2 topics

| File | Task statement | One-line summary |
|---|---|---|
| [01](01-tool-interface-design.md) | 2.1 Tool interface design | Write descriptions and names so the model picks the right tool |
| [02](02-structured-error-responses.md) | 2.2 Structured errors | Return errors the agent can act on |
| [03](03-tool-distribution-and-tool-choice.md) | 2.3 Tool distribution and `tool_choice` | Give each agent the right tools, and control *whether* it must call one |
| [04](04-mcp-server-integration.md) | 2.4 MCP integration | Configure servers safely and expose content as resources |
| [05](05-built-in-tools.md) | 2.5 Built-in tools | Choose between Read, Write, Edit, Bash, Grep, Glob |
| [06](06-scenario-practice-questions.md) | All | Practice questions |

---

## 7. Glossary

| Term | Meaning |
|---|---|
| **Built-in tool** | A tool that ships with Claude Code / the Agent SDK (Read, Edit, Grep…) |
| **Content catalog** | A list or index of what data exists (issues, docs, tables), often exposed as an MCP resource |
| **Cross-role tool** | A tool given to an agent outside its main specialty, for one frequent, narrow need |
| **Environment variable expansion** | Writing `${VAR}` in a config file so the real value is read from the environment at runtime |
| **`errorCategory`** | A field in an error response that classifies the failure (transient, validation, business, permission) |
| **`isError`** | MCP result flag that marks a tool result as a failure |
| **`isRetryable`** | A field in an error response that tells the agent whether trying again can help |
| **MCP client / host / server** | See §3 |
| **`.mcp.json`** | Project-level MCP config file, committed to the repo and shared with the team |
| **`~/.claude.json`** | User-level Claude Code config file holding personal MCP servers |
| **Misrouting** | The model choosing the wrong tool for a request |
| **Resource (MCP)** | Read-only content exposed by an MCP server via a URI |
| **Scoped tool access** | Giving each agent only the tools its role needs |
| **stdio / HTTP transport** | Local-process vs remote connection to an MCP server |
| **Tool description** | Natural-language text that tells the model what a tool does and when to use it |
| **`tool_choice`** | API setting that controls whether and which tool the model must call |
| **Transient error** | A temporary failure (timeout, service unavailable) that may succeed on retry |

---

## Common Q&A

**Q: Does the model see my tool's code?**
A: No. It sees only the name, description, and input schema, so those must explain everything it needs to choose and use the tool correctly.

**Q: Is MCP only for Claude?**
A: No. MCP is an open standard. Any compatible host can use any MCP server.

**Q: What's the practical difference between a tool and a resource?**
A: A tool is an **action** the model decides to call. A resource is **content** (often read-only) that the application or user loads into context. Use resources to show *what exists*; use tools to *do things*.

**Q: Do I have to write my own MCP server?**
A: Often no. Community servers exist for many common systems. Build your own for team-specific workflows (see [04](04-mcp-server-integration.md)).

**Q: How are MCP tools named inside Claude Code?**
A: `mcp__<server>__<tool>`, e.g. `mcp__github__create_issue`. Hooks and permission rules use this name.

---

## Key Takeaways

- A tool = **name + description + input schema + output**. The model chooses mostly from the **description**.
- **MCP** connects hosts (AI apps) to servers (systems) through a standard protocol, over **stdio** (local) or **HTTP** (remote).
- MCP servers expose **tools** (actions), **resources** (content), and **prompts** (templates).
- Agents get tools from three places: **built-in**, **MCP**, and **custom**.
- Many "wrong tool" problems are **design problems** you fix in descriptions, tool sets, and configuration.
