# 00 · Foundations and Glossary

> **Read this first.** This file gives you the basic ideas and words used in every other file in this folder. Later files assume you know these terms and do not define them again.

**Exam link:** Background for all of Domain 1 (27% of the exam).

---

## 1. What is an "agent"?

A **large language model (LLM)** is a program that reads text and writes text. Claude is an LLM.

On its own, an LLM can only *talk*. It cannot read your database, send an email, or run code. To do real work, we give it **tools**.

- A **tool** is a function that *your* code runs on Claude's behalf (for example `get_customer`, `search_web`, `run_tests`).
- Claude does not run the tool itself. Claude *asks* for the tool by writing a structured request. Your code runs it and sends back the result.

An **agent** is an LLM that works in a cycle. It decides what to do, uses tools, and looks at the results. Then it decides again, until the task is done. The important word is **decides**. In an agent, the model chooses the next step.

### Real-world example

A customer writes: *"I was charged twice for order 4471."*

- A **chatbot** (no tools) can only say "Sorry, please contact billing."
- An **agent** can look up the customer, look up the order, see the double charge, start a refund, and then reply with the result.

---

## 2. Workflow vs agent

These two words are easy to mix up. The exam expects you to know the difference.

| | **Workflow** | **Agent** |
|---|---|---|
| Who decides the next step? | Your code (fixed path) | The model (dynamic path) |
| Predictability | High | Lower |
| Flexibility | Low | High |
| Good for | Known, repeatable tasks | Open-ended tasks where the steps are not known in advance |
| Example | "Extract fields from every invoice, then validate, then save" | "Find out why this test suite is flaky" |

**Rule of thumb:** use the simplest option that works. Only use an agent when the task truly needs the model to choose its own steps. Agents cost more (more model calls, more tokens) and are harder to test.

Many real systems are **hybrids**: an agent makes decisions, but code enforces important rules around it. You will see this pattern again and again in this domain.

---

## 3. The building blocks

### 3.1 The Messages API

The **Claude API** (also called the Messages API) is the basic way to talk to Claude. You send a list of **messages** and get one response back.

- Each message has a **role**: `user` (input to Claude) or `assistant` (Claude's output).
- The API is **stateless**: Claude does not remember earlier calls. *You* send the full conversation every time.

This last point is very important. Everything Claude "knows" in a call is what you put in that call.

### 3.2 Context and the context window

- **Context** = all the text Claude can see in one request: the system prompt, the tool definitions, and the message history.
- The **context window** = the maximum amount of context, measured in **tokens** (pieces of words; roughly 3/4 of a word each).
- When context gets long, the model has more to pay attention to. Important details can get lost. This is called **attention dilution**.

### 3.3 System prompt

The **system prompt** is a special instruction block that sets Claude's role, rules, and goals for the whole conversation. Example: *"You are a support agent for Acme. Always verify the customer before discussing orders."*

### 3.4 Tool definitions and tool calls

You describe each tool with a **name**, a **description**, and an **input schema** (a JSON Schema that lists the parameters).

```python
get_customer_tool = {
    "name": "get_customer",
    "description": "Look up a customer by email. Returns customer_id, name, and verification status.",
    "input_schema": {
        "type": "object",
        "properties": {"email": {"type": "string", "description": "Customer email address"}},
        "required": ["email"],
    },
}
```

When Claude wants to use a tool, its response contains a **`tool_use` block** (the tool call). Your code runs the tool and sends back a **`tool_result` block**. File [01](01-agentic-loops.md) explains this cycle in detail.

### 3.5 The harness

The **harness** is the code *around* the model: it calls the API, runs the tools, keeps the history, and applies rules. Your design choices about the harness are what this domain tests.

### 3.6 The Claude Agent SDK

The **Claude Agent SDK** is a library from Anthropic (Python package `claude-agent-sdk`, TypeScript package `@anthropic-ai/claude-agent-sdk`). It is the same harness that powers Claude Code, packaged so you can build your own agents. It gives you, ready-made:

- the agent loop
- built-in tools (Read, Write, Edit, Bash, Grep, Glob, WebSearch, WebFetch)
- **subagents** (helper agents)
- **hooks** (code that runs at fixed points, for example before a tool call)
- **sessions** (saved conversations you can resume or fork)
- permissions and MCP support

Many exam questions are about Agent SDK features. Files [03](03-subagent-configuration.md), [05](05-agent-sdk-hooks.md), and [07](07-session-state-resumption-forking.md) cover them.

### 3.7 MCP (Model Context Protocol)

**MCP** is an open standard for connecting tools and data sources to AI applications. An **MCP server** exposes tools (for example, a company's order system). An MCP client (such as an Agent SDK agent) can use them. MCP design is mostly Domain 2, but MCP tools appear in Domain 1 scenarios, for example `get_customer`, `lookup_order`, `process_refund`, `escalate_to_human`.

---

## 4. Two ideas that run through the whole domain

### 4.1 Deterministic vs probabilistic

- **Deterministic** = always gives the same result. Code is deterministic: `if amount > 500: block()` will block every time.
- **Probabilistic** = usually gives the expected result, but not always. A model following a prompt is probabilistic. Even a very good prompt has a small failure rate.

> **Exam pattern:** Some rules *must always* hold (money, security, legal compliance). For these, the correct answer is almost always a **code-based control** (a hook, a gate, a check). It is not "improve the prompt" or "add examples".

### 4.2 Context must be explicit

Claude only knows what is in its context. A helper agent does not see what the main agent saw unless you put it in the helper's prompt. Many wrong answers on the exam assume that information "flows automatically". It does not.

---

## 5. The seven topics of Domain 1

| File | Task statement | One-line summary |
|---|---|---|
| [01](01-agentic-loops.md) | 1.1 Agentic loops | How the decide → act → observe cycle works and when it stops |
| [02](02-multi-agent-orchestration.md) | 1.2 Multi-agent orchestration | A coordinator agent that manages several specialist agents |
| [03](03-subagent-configuration.md) | 1.3 Subagent configuration | How to define, start, and feed context to subagents |
| [04](04-workflow-enforcement-and-handoffs.md) | 1.4 Enforcement and handoffs | Forcing step order and handing work to humans |
| [05](05-agent-sdk-hooks.md) | 1.5 Agent SDK hooks | Code that intercepts tool calls and tool results |
| [06](06-task-decomposition.md) | 1.6 Task decomposition | Breaking big tasks into smaller steps |
| [07](07-session-state-resumption-forking.md) | 1.7 Sessions | Continuing, branching, and restarting agent work |
| [MCQ bank](../mcq-bank/01-agentic-architecture.md) | All | 65 exam-level practice questions |

---

## 6. Glossary

| Term | Meaning |
|---|---|
| **Agent** | An LLM that repeatedly chooses actions (often tool calls) until a goal is reached |
| **Agentic loop** | The repeat cycle: call model → run requested tools → return results → call model again |
| **AgentDefinition** | Agent SDK object that defines a subagent type (description, prompt, tools, model) |
| **Attention dilution** | Lower quality when the model must pay attention to too much context at once |
| **Context** | Everything the model can see in one request |
| **Coordinator** (orchestrator) | The main agent that splits work, sends it to subagents, and combines results |
| **Decomposition** | Splitting a big task into smaller subtasks |
| **Deterministic** | Always the same outcome (code rules) |
| **Escalation** | Passing a case to a human |
| **Fork** | A new session that starts as a copy of an existing one, then goes its own way |
| **Handoff** | Passing work (with a summary) from one agent or person to another |
| **Harness** | The code around the model that runs the loop, tools, and rules |
| **Hook** | Agent SDK callback that runs at a fixed point (e.g. before or after a tool call) |
| **Hub-and-spoke** | Architecture where all communication goes through one central coordinator |
| **MCP** | Model Context Protocol, an open standard for connecting tools to AI apps |
| **Probabilistic** | Usually the expected outcome, but with some failure rate (model behavior) |
| **Prompt chaining** | A fixed sequence of model calls where each step's output feeds the next |
| **Session** | A saved agent conversation with an ID, which can be resumed or forked |
| **`stop_reason`** | Field in each API response that says *why* the model stopped writing |
| **Subagent** | A helper agent started by another agent to do one focused job |
| **Tool** | A function your code runs when the model asks for it |
| **`tool_result`** | Content block that returns a tool's output to the model |
| **`tool_use`** | Content block in which the model asks for a tool call |

---

## Common Q&A

**Q: Does Claude run my tools?**
A: No. Claude *requests* a tool call. Your code (or the Agent SDK) runs it and returns the result. This is why you can put rules around tools: your code is always in the middle.

**Q: If the API is stateless, how does an agent "remember" earlier steps?**
A: The harness sends the full message history, including earlier tool calls and results, in each new request.

**Q: Is every system that uses tools an agent?**
A: No. If your code decides the order of calls, it is a workflow that uses tools. It becomes agentic when the model chooses which tool to use next.

**Q: When should I *not* build an agent?**
A: Use a workflow when the steps are known and fixed. Use it when errors are very costly and hard to catch. Use it when a single model call does the job. A workflow is cheaper, faster, and easier to test.

**Q: What is the difference between the Claude API and the Agent SDK?**
A: The Claude API is the basic request/response endpoint; you write the loop and tools yourself. The Agent SDK gives you a full harness (loop, built-in tools, subagents, hooks, sessions) on top of it.

---

## Key Takeaways

- An agent is a model that **chooses its own next step** in a loop, using tools your code runs.
- Use a **workflow** when the path is known; use an **agent** when it is not.
- The API is **stateless**: context is only what you send.
- **Deterministic code beats probabilistic prompts** whenever a rule must always hold.
- **Nothing is shared automatically** between agents; context must be passed explicitly.
