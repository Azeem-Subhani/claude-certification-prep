# 01 · Tool Interface Design: Descriptions, Names, and Boundaries

**Exam link:** Task Statement 2.1: *Design effective tool interfaces with clear descriptions and boundaries.*
**Prerequisite:** [00 · Foundations](00-foundations-and-glossary.md) (anatomy of a tool definition).

---

## 1. The description is the main selection signal

When Claude decides which tool to call, it reads the **tool descriptions** (plus names and schemas) and compares them with the request. It has no other information about your tools.

So:

> **Minimal descriptions → unreliable tool selection**, especially when two tools sound similar.

### Example: two vague tools

```jsonc
{"name": "get_customer", "description": "Retrieves customer information"}
{"name": "lookup_order", "description": "Retrieves order details"}
```

A user writes: *"Check on my order #12345."* Both descriptions say "retrieves… information/details". Customers *have* orders, so `get_customer` looks reasonable too. In production, the agent often calls `get_customer` first, wasting a turn or getting the wrong data.

### The same tools with good descriptions

```jsonc
{
  "name": "get_customer",
  "description": "Look up a customer ACCOUNT by email or phone. Returns customer_id, name, verification status, and account flags. Use this to identify or verify WHO the customer is. Do not use it to check an order's status; use lookup_order with the order number instead."
}
{
  "name": "lookup_order",
  "description": "Get details of ONE ORDER by order number (e.g. '12345' or 'ORD-12345'). Returns status, items, charges, and shipping dates. Use this whenever the user mentions an order number or asks where an order is. Requires a verified customer_id from get_customer."
}
```

Now the request "check on my order #12345" clearly matches `lookup_order`.

> **Exam pattern:** When logs show the agent choosing the wrong one of two similar tools, and both have short descriptions, the **first and most effective fix** is to **improve the descriptions**. Answers like "add a routing classifier", "fine-tune the model", or "merge both tools" are usually more complex than needed or reduce capability.

---

## 2. What a good description contains

| Element | Why it helps | Example text |
|---|---|---|
| **Purpose** (one clear job) | Tells the model *what* the tool does | "Get details of one order by order number." |
| **When to use it** | Links tool to user intents | "Use when the user mentions an order number or delivery." |
| **When NOT to use it / alternatives** | Separates it from similar tools | "For account details, use get_customer instead." |
| **Input format** | Prevents bad arguments | "order_id: 'ORD-' + 6 digits, e.g. ORD-123456." |
| **Example queries** | Shows real phrasing that should trigger it | "e.g. 'Where is my package?', 'Status of ORD-123456'" |
| **Edge cases** | Explains unusual inputs | "If the user gives only a tracking number, use track_shipment instead." |
| **Output** | Tells the model what it will get | "Returns status, items, charges, ISO 8601 dates." |
| **Preconditions / boundaries** | Prevents misuse | "Requires verified customer_id. Read-only; cannot change orders." |

Good descriptions are often **3–6 sentences**. That's normal and recommended.

---

## 3. Overlap causes misrouting

**Misrouting** = the model sends a request to the wrong tool. The most common cause is **overlapping or ambiguous** tools.

### Example: research system

```
analyze_content: "Analyzes content and returns insights."
analyze_document: "Analyzes a document and returns insights."
```

Is a web page "content" or a "document"? Is a PDF? The model can't tell, so it guesses.

### Fix 1: rename + rewrite to remove overlap

```
extract_web_results: "Extract key claims, dates, and source URLs from WEB SEARCH RESULTS
                        (HTML pages returned by web_search). Not for PDFs or uploaded files."
analyze_document:    "Analyze an UPLOADED DOCUMENT (PDF, DOCX) the user provided.
                        Returns claims with document name and page number."
```

The **name** now signals the domain (web vs uploaded document), and the **descriptions** state clear boundaries.

### Fix 2: split a generic tool into purpose-specific tools

A single `analyze_document(document, instructions)` tool can do anything, so the model must guess how to use it and outputs vary.

Split it by **purpose**, each with a defined **input/output contract**:

| New tool | Input | Output |
|---|---|---|
| `extract_data_points` | document, list of fields to extract | `{field: value, page}` per field |
| `summarize_content` | document, target length | summary text + key points |
| `verify_claim_against_source` | claim, source document | `{supported: true/false/partial, evidence_quote, page}` |

**Why this helps:**

- Each tool has **one job**, so selection is easier.
- Outputs are **predictable**, so downstream steps and agents can rely on them.
- Each tool can be **tested** on its own.

> **Balance:** Don't split too far. Ten nearly identical tiny tools create new overlap. Split when **purposes and outputs are truly different**. (Too many tools per agent is covered in [03](03-tool-distribution-and-tool-choice.md).)

---

## 4. Naming guidelines

- Use **verb + specific object**: `lookup_order`, `refund_payment`, `search_web`, `extract_web_results`.
- Avoid **generic verbs** that could mean anything: `process`, `handle`, `analyze`, `get_data`, `do_action`.
- Keep names **consistent** across tools (`get_customer` / `get_order`, not `get_customer` / `fetchOrderInfo`).
- Put the **distinguishing word** in the name (`web` vs `document`, `customer` vs `order`).

---

## 5. System prompt wording can override good descriptions

The system prompt also influences tool choice. **Keyword-sensitive instructions** can create unintended links between words and tools.

### Example

System prompt: *"Whenever the user mentions their **account**, call `get_customer` first."*

User: *"My order hasn't arrived. It's on my **account** under order 4471."*

The word "account" triggers `get_customer`, even though `lookup_order` is clearly the right first step and its description says so. The prompt's **keyword rule** overrode the well-written description.

### How to fix it

- Review system prompts for **keyword triggers** ("whenever the user says X, call Y").
- Replace them with **intent-based** guidance: *"Verify the customer's identity before any account changes or refunds."*
- Keep tool-selection details **in the tool descriptions**, not scattered in the system prompt.
- Test with requests that contain trigger words in **unrelated** contexts.

---

## 6. Real-world case study

**Situation:** A research platform's synthesis agent had three tools: `analyze_content`, `analyze_document`, `analyze_data`. Logs showed:

- PDFs uploaded by users were sent to `analyze_content` 30% of the time.
- Web pages were sometimes sent to `analyze_document`, which expected page numbers and failed.

**Changes:**

1. Renamed `analyze_content` → `extract_web_results`, with a description that says "web search results only".
2. Split `analyze_document` into `extract_data_points`, `summarize_content`, and `verify_claim_against_source`, each with a clear output schema.
3. Removed a system-prompt line: *"For any content analysis, use analyze_content."* It had been forcing the wrong tool whenever users said "content".

**Result:** Misrouting dropped sharply in the test set, and downstream agents received consistent output formats.

---

## Common Q&A

**Q: Aren't long descriptions a waste of tokens?**
A: Tool descriptions are small compared with the cost of wrong tool calls (extra turns, wrong data, wrong actions). Clear, multi-sentence descriptions are recommended.

**Q: Two tools are confused. Should I merge them into one?**
A: Usually not. Merging creates a generic tool with unclear behavior. First, make their descriptions and names **clearly different**. Merge only if they truly do the same job.

**Q: Should I add few-shot examples to the system prompt to fix tool selection?**
A: Fix the **descriptions** first; they're the primary mechanism. Examples can help, but they don't fix overlapping tool definitions.

**Q: What's the risk of rules like "if the user says X, use tool Y" in the system prompt?**
A: They create **keyword-based** associations that fire in the wrong contexts and can override good descriptions.

**Q: When should I split a tool?**
A: When it serves several different purposes with different outputs, and callers (or the model) struggle to use it consistently.

**Q: Should the description mention other tools?**
A: Yes, when they are easy to confuse. "Use X instead for Y" is one of the most effective ways to set boundaries.

---

## Key Takeaways

- **Descriptions drive tool selection.** Minimal descriptions = unreliable choices between similar tools.
- A good description states **purpose, when to use, when not to (alternatives), input format, examples, edge cases, output**.
- **Overlap causes misrouting.** Fix it by **renaming** and **rewriting** with clear boundaries.
- **Split generic tools** into purpose-specific tools with defined input/output contracts.
- **Audit system prompts** for keyword triggers that override good descriptions.
