# 02 · Structured Error Responses for MCP Tools

**Exam link:** Task Statement 2.2: *Implement structured error responses for MCP tools.*
**Prerequisites:** [00](00-foundations-and-glossary.md), and Domain 1 [01 · Agentic Loops](../domain-1-agentic-architecture-orchestration/01-agentic-loops.md) (`is_error` in `tool_result`).

---

## 1. Why error design matters

Tools fail all the time: networks time out, users type bad IDs, policies block actions. What matters is **what the agent does next**. It can only make a good decision if the error tells it **what kind of failure** happened.

Compare two error messages for the same failed refund:

| Generic | Structured |
|---|---|
| `"Operation failed"` | category: business, retryable: false, message: "Refund window closed: order delivered 45 days ago; policy allows 30 days." |

With the generic error, the agent might:
- retry 5 times (wasted calls, the same result each time),
- tell the customer "something went wrong, try later" (wrong),
- or escalate a case that has a clear answer.

With the structured error, the agent knows: **don't retry**, explain the policy kindly, and offer alternatives.

> **Uniform, generic errors prevent the agent from choosing the right recovery.**

---

## 2. The MCP `isError` flag

In MCP, a tool reports an **execution failure** by returning a normal result with **`isError: true`**:

```json
{
  "content": [{"type": "text", "text": "Failed to fetch order: upstream service timed out after 10s"}],
  "isError": true
}
```

The MCP specification separates two kinds of errors:

| Kind | Examples | How reported |
|---|---|---|
| **Protocol errors** | Unknown tool name, malformed request, server crash | JSON-RPC `error` object (the call never really ran) |
| **Tool execution errors** | API failure, invalid input data, business-rule failure | Result with **`isError: true`**, visible to the model so it can react |

Because execution errors come back **as tool results**, the model sees them and can **self-correct**. It can fix an argument, try another tool, or explain to the user. In the Claude API this becomes a `tool_result` block with `is_error: true`.

---

## 3. Four error categories

| Category | Meaning | Examples | Retry? | Typical agent response |
|---|---|---|---|---|
| **Transient** | Temporary problem with the system | Timeout, 503 service unavailable, rate limit | **Yes** (with backoff) | Retry; if it keeps failing, report partial results |
| **Validation** | The input was wrong | Bad order ID format, missing field, date in the past | **Not as-is**; fix the input first | Correct the argument or ask the user |
| **Business** | Valid request, but a **policy** forbids it | Refund window expired, amount over limit, item not returnable | **No** | Explain the rule kindly; offer alternatives or escalate |
| **Permission** | Caller isn't allowed | Missing scope, account locked, access denied | **No** (not without a change in access) | Escalate or tell the user what's needed |

---

## 4. A structured error format

Return **machine-readable metadata** plus a **human-readable** explanation:

```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "{\"errorCategory\": \"business\", \"isRetryable\": false, \"code\": \"REFUND_WINDOW_EXPIRED\", \"message\": \"Order ORD-123456 was delivered 45 days ago. Refunds are allowed within 30 days.\", \"customerMessage\": \"This order is outside our 30-day refund window, but you may be eligible for store credit.\", \"suggestedAction\": \"offer_store_credit\"}"
  }]
}
```

| Field | Purpose |
|---|---|
| `errorCategory` | transient / validation / business / permission |
| `isRetryable` | Tells the agent directly whether retrying can help, **preventing wasted retries** |
| `code` | Stable identifier for logs and code |
| `message` | What went wrong, for the agent and for engineers |
| `customerMessage` | Friendly text the agent can pass to the customer (useful for **business** errors) |
| `suggestedAction` | Optional hint for recovery |

### Python sketch of an MCP tool handler

```python
import json

def error_result(category: str, retryable: bool, code: str, message: str, customer_message: str | None = None) -> dict:
    """Build an MCP tool result that marks failure and carries structured metadata."""
    payload = {"errorCategory": category, "isRetryable": retryable, "code": code, "message": message}
    if customer_message:
        payload["customerMessage"] = customer_message
    return {"isError": True, "content": [{"type": "text", "text": json.dumps(payload)}]}


def process_refund(order_id: str, amount: float) -> dict:
    if not order_id.startswith("ORD-"):
        return error_result("validation", False, "BAD_ORDER_ID",
                            f"order_id '{order_id}' must look like ORD-123456")
    try:
        order = orders_api.get(order_id, timeout=10)          # your backend client
    except TimeoutError:
        return error_result("transient", True, "ORDERS_TIMEOUT", "Orders service timed out after 10s")
    except PermissionError:
        return error_result("permission", False, "NO_REFUND_SCOPE", "Service account lacks refund permission")

    if order.days_since_delivery > 30:
        return error_result("business", False, "REFUND_WINDOW_EXPIRED",
                            f"Delivered {order.days_since_delivery} days ago; limit is 30",
                            customer_message="This order is outside our 30-day refund window.")
    ...
```

---

## 5. Business errors: non-retryable + customer-friendly

Business rule violations are **not failures of the system**. The system worked; the answer is "no". For these:

- Set **`isRetryable: false`** so the agent doesn't loop.
- Include a **customer-friendly explanation** the agent can communicate directly.
- Optionally include **alternatives** (store credit, exchange, escalation path).

This stops the agent from sending messages like "There was an error, please try again later". The honest answer may be "this is outside the policy".

---

## 6. Local recovery in subagents, then escalate

In multi-agent systems (Domain 1 · [02](../domain-1-agentic-architecture-orchestration/02-multi-agent-orchestration.md)), **where** an error is handled matters.

**Recommended pattern:**

1. The **subagent recovers locally** from errors it can handle itself, typically **transient** ones (retry with backoff, try an alternative source).
2. Only errors it **cannot resolve** go up to the coordinator.
3. When it escalates, it sends **structured context**: what failed, **what it tried**, and any **partial results**.

```json
{
  "status": "partial_failure",
  "errorCategory": "transient",
  "isRetryable": true,
  "attempted": ["web_search: 'AI music adoption 2024' (timeout x3, backoff 1s/2s/4s)",
                "web_search: 'AI music market report' (timeout)"],
  "partialResults": [
    {"claim": "AI mastering tools used by 30% of indie artists", "source_url": "https://example.org/survey", "date": "2025-01-10"}
  ],
  "coverageGap": "No data found yet on AI use in film scoring"
}
```

**Why this works:**

- The coordinator isn't flooded with every small retry.
- The coordinator gets enough information to decide: re-delegate, continue with partial data, or **note the gap** in the final report.

**Anti-patterns:**

| ❌ Anti-pattern | Problem |
|---|---|
| Subagent returns `"Search failed"` only | Coordinator can't tell whether to retry, re-route, or continue |
| Subagent hides the failure and returns empty findings as success | The final report silently has gaps |
| Subagent escalates every single timeout immediately | Coordinator is overloaded; simple retries would have worked |
| Whole pipeline stops on one subagent failure | Loses all the good partial work |

---

## 7. Failed vs empty: an important difference

These two results look similar but mean very different things:

| Situation | Meaning | Correct response |
|---|---|---|
| **Access failure** | The query **did not run** (timeout, auth error) | Error result (`isError: true`) → retry or escalate |
| **Valid empty result** | The query **ran successfully** and found **no matches** | Success result with an empty list → "no matching orders found" |

```jsonc
// Valid empty result: success, nothing found
{"isError": false, "content": [{"type": "text", "text": "{\"orders\": [], \"query\": \"email=jo@example.com\"}"}]}

// Access failure: the search did not happen
{"isError": true, "content": [{"type": "text", "text": "{\"errorCategory\": \"transient\", \"isRetryable\": true, \"message\": \"Orders DB unreachable\"}"}]}
```

If a tool returns an **empty list on failure**, the agent will confidently tell a customer "you have no orders". That is a **wrong answer**, not an error. Always keep failures and empty results clearly separate.

---

## 8. Real-world case study

**Situation:** A travel company's booking agent used an MCP server whose tools returned `{"error": "Request failed"}` for every problem.

**Symptoms:**
- The agent retried "fare no longer available" errors repeatedly, so fares were never rebooked.
- When the airline API timed out, the search tool returned `[]`, and the agent told customers "no flights available" on busy routes.

**Fixes:**
1. Every error now includes `errorCategory`, `isRetryable`, and a clear message. Fare changes are `business` / not retryable, with a customer message offering the new price.
2. Timeouts return `isError: true` (`transient`, retryable) instead of an empty list.
3. The flight-search subagent retries transient errors up to 3 times with backoff. If still failing, it returns partial results and what it tried to the coordinator.

**Result:** No more retry loops on fare changes, and no more false "no flights" answers.

---

## Common Q&A

**Q: Why not just throw an exception from the tool?**
A: An unhandled exception may crash the call or become a generic protocol error. Returning a result with `isError: true` and structured details lets the **model see and react** to the failure.

**Q: Should validation errors be retryable?**
A: Not with the same input. The agent should **fix the input** (or ask the user) and then call again. Mark them `isRetryable: false` or explain what must change.

**Q: Is "no results found" an error?**
A: No. If the query ran successfully, an empty result is a **valid success**. Only mark it as an error if the query could not run.

**Q: Who should retry transient errors: the subagent or the coordinator?**
A: Ideally the **subagent**, locally, with limits. Escalate only if local recovery fails, with partial results and the attempts made.

**Q: Why include a customer-friendly message in a business error?**
A: The agent can pass it on directly and accurately, instead of inventing an explanation or showing a technical error.

**Q: What does `isRetryable` save?**
A: Wasted calls, latency, and cost from retrying errors that can never succeed (e.g. policy violations).

---

## Key Takeaways

- Use the MCP **`isError`** flag so failures come back as results the model can **see and act on**.
- Classify errors: **transient** (retry), **validation** (fix input), **business** (explain, don't retry), **permission** (escalate).
- Return **structured metadata**: `errorCategory`, `isRetryable`, clear message (+ customer message for business errors).
- **Generic "Operation failed" errors** block correct recovery.
- Subagents **recover locally** from transient errors and escalate the rest **with partial results and attempts**.
- Never confuse an **access failure** with a **valid empty result**.
