# 03 · Structured Output with Tool Use and JSON Schemas

**Exam link:** Task Statement 4.3: *Enforce structured output using tool use and JSON schemas.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (syntax vs semantic errors), Domain 2 [03 · `tool_choice`](../domain-2-tool-design-mcp-integration/03-tool-distribution-and-tool-choice.md).
**Scope note:** Domain 2 explained the `tool_choice` options in general. This file applies them to **extraction pipelines** and focuses on **schema design**.

---

## 1. Why not just ask for JSON in the prompt?

"Return JSON with fields name, date, total" usually works, but **not always**:

- Missing or extra commas, unclosed brackets → **parse errors**.
- Text before the JSON ("Here is the data: …").
- Field names that drift (`total` vs `amount_total`).

At scale, even a 1% failure rate means thousands of broken records.

**Tool use with a JSON schema** is the most reliable approach. You define a "tool" whose **input schema is the structure you want**. Claude "calls" it, and the tool call's **input is your structured data**. The API handles the JSON, so **syntax errors are eliminated**.

> Current Anthropic APIs also offer **structured outputs**: `strict: true` on tool definitions, and JSON outputs via `output_config.format`. Both use constrained decoding to guarantee schema-compliant output (with documented exceptions such as refusals or hitting `max_tokens`).

---

## 2. An extraction tool

```python
import anthropic

client = anthropic.Anthropic()

extract_invoice = {
    "name": "extract_invoice",
    "description": "Record the fields extracted from ONE invoice document.",
    "strict": True,  # schema-compliant arguments (current API feature)
    "input_schema": {
        "type": "object",
        "properties": {
            "invoice_number": {"type": "string"},
            "invoice_date": {"type": "string", "format": "date",
                             "description": "YYYY-MM-DD"},
            "po_number": {"type": ["string", "null"],
                          "description": "Purchase order number, or null if the document has none"},
            "currency": {"type": "string", "description": "ISO 4217 code, e.g. USD"},
            "line_items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "amount": {"type": "number"},
                    },
                    "required": ["description", "amount"],
                    "additionalProperties": False,
                },
            },
            "stated_total": {"type": "number"},
            "payment_terms": {
                "type": "string",
                "enum": ["net_15", "net_30", "net_60", "due_on_receipt", "unclear", "other"],
            },
            "payment_terms_detail": {"type": ["string", "null"],
                                     "description": "Original wording when payment_terms is 'other' or 'unclear'"},
        },
        "required": ["invoice_number", "invoice_date", "po_number", "currency",
                     "line_items", "stated_total", "payment_terms", "payment_terms_detail"],
        "additionalProperties": False,
    },
}

response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=4096,
    tools=[extract_invoice],
    messages=[{"role": "user", "content": f"Extract the invoice fields.\n\n<document>\n{document_text}\n</document>"}],
)

# The structured data is the tool call's input
data = next(b.input for b in response.content if b.type == "tool_use")
```

---

## 3. `tool_choice` in extraction pipelines

(Options defined in Domain 2 · [03](../domain-2-tool-design-mcp-integration/03-tool-distribution-and-tool-choice.md).) Applied to extraction:

| Situation | Setting (exam answer) | Why |
|---|---|---|
| Normal turn; text is fine sometimes | `"auto"` | Model **may reply with text** instead of calling a tool |
| **Several extraction schemas** (invoice, receipt, contract) and the **document type is unknown** | `"any"` | Guarantees **some** extraction tool is called (structured output), and the model picks the right schema |
| A specific step must run **first** (e.g. `extract_metadata` before enrichment) | `{"type": "tool", "name": "extract_metadata"}` | Forces that tool; later steps follow in further turns |

> **Current-model note:** The exam tests `"any"` and forced selection. On the newest models (Claude Opus 5.5, Sonnet 5.5, Fable 5.1) those two settings return a 400 error. Use `"auto"` with an explicit instruction plus `strict: true`, or JSON outputs (`output_config.format`). Older models still accept forced choice.

---

## 4. Schemas fix syntax, not semantics

A strict schema guarantees:
- valid JSON,
- required fields present,
- correct types,
- enum values from the list.

It does **not** guarantee:

| Semantic error | Example |
|---|---|
| Values don't add up | Line items sum to 1,240 but `stated_total` is 1,420 |
| Value in the wrong field | Due date written into `invoice_date` |
| Invented values | A PO number "found" when the document has none |
| Wrong interpretation | "Net 30 EOM" mapped to `net_30` |

These need **validation and feedback loops**: see [04](04-validation-retry-feedback-loops.md).

---

## 5. Schema design: don't force the model to invent

### 5.1 Required vs optional (nullable)

If a field is **required** and **non-nullable**, but the document **doesn't contain** that information, the model is pushed to **fabricate** something to satisfy the schema.

**Fix:** make fields **nullable** (or optional) when source documents may not contain them:

```json
"po_number": {"type": ["string", "null"], "description": "null if the document has no PO number"}
```

Then pair it with a few-shot example showing `null` for a missing value ([02](02-few-shot-prompting.md)).

> **Exam pattern:** "Extraction produces plausible-looking but fake values for fields that aren't in some documents." → **Make those fields nullable/optional** (and say "use null if not present"). Not "add 'don't hallucinate' to the prompt".

### 5.2 Enums with `"unclear"` and `"other"` + detail

Enums keep categories consistent, but the real world has cases you didn't list.

| Value | Use |
|---|---|
| Normal values | `net_15`, `net_30`, `net_60`, `due_on_receipt` |
| `"unclear"` | The document is **ambiguous**: don't force a guess |
| `"other"` + `*_detail` string | A **valid** value not in your list: keep the original text so you can extend the enum later |

This makes categorization **extensible**: you can review the `other` details and add new enum values over time.

### 5.3 Normalization rules in the prompt

A schema says a field is a string; it doesn't say **which format**. Sources vary: `03/04/2025`, `4 March 2025`, `2025-03-04`. Add **normalization rules** to the prompt, alongside the schema:

```text
Normalization rules:
- Dates: YYYY-MM-DD. If day/month order is ambiguous (e.g. 03/04/2025), use the vendor's
  country to decide; if still unknown, set the field to null and explain in notes.
- Currency: ISO 4217 code (USD, EUR), never symbols.
- Amounts: numbers without thousands separators; negative for credits.
```

---

## 6. Real-world case study

**Situation:** An accounts-payable team extracted data from supplier invoices by asking for JSON in the prompt.

**Problems:**
1. ~2% of responses failed to parse.
2. Many invoices have no PO number, yet every record had one, often the invoice number copied over.
3. Payment terms came back as free text in 40 different spellings.

**Fixes:**
1. Switched to an **extraction tool with a strict JSON schema**: parse failures went to zero.
2. Made `po_number` **nullable**, with an instruction and an example for missing values: fake PO numbers stopped.
3. Changed payment terms to an **enum** with `"unclear"` and `"other"` + `payment_terms_detail`; added **normalization rules** for dates and currency.
4. Since documents could be invoices, credit notes, or receipts, defined **three extraction tools** and required a tool call so the model picked the right schema. (On the newest models: `auto` + strict tools + instruction.)

---

## Common Q&A

**Q: Why use tool use for structured output instead of asking for JSON?**
A: Tool use with a JSON schema reliably produces schema-compliant output and removes JSON syntax errors.

**Q: Does a strict schema guarantee correct data?**
A: No. It guarantees **structure**. Semantic errors (wrong sums, wrong fields, invented values) still need validation.

**Q: I have several document types and don't know which one I'll get. Which `tool_choice`?**
A: Per the exam: `"any"`, so a tool must be called and the model chooses the matching schema.

**Q: How do I stop the model inventing values for missing fields?**
A: Make those fields **nullable/optional**, tell it to use `null` when absent, and show an example.

**Q: Why add `"other"` with a detail field to an enum?**
A: To capture valid values you didn't anticipate without forcing a wrong category, and to learn which values to add later.

**Q: Why put normalization rules in the prompt if I have a schema?**
A: The schema controls type and structure, not formatting conventions inside strings (dates, currency codes).

---

## Key Takeaways

- **Tool use + JSON schema** = most reliable structured output; **eliminates syntax errors**.
- `tool_choice` for extraction: **`any`** when the document type is unknown; **forced** for a required first step. (Newest models: `auto` + `strict` instead.)
- Schemas **don't prevent semantic errors**. Validate meaning separately.
- Make fields **nullable** when data may be missing, so the model **doesn't fabricate**.
- Use enums with **`"unclear"`** and **`"other"` + detail** for ambiguous and new categories.
- Add **normalization rules** to the prompt alongside the schema.
