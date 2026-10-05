# 05 · Agent SDK Hooks: Tool Call Interception and Data Normalization

**Exam link:** Task Statement 1.5: *Apply Agent SDK hooks for tool call interception and data normalization.*
**Prerequisite:** [04 · Enforcement and Handoffs](04-workflow-enforcement-and-handoffs.md) (why deterministic enforcement matters).
**Scope note:** [04](04-workflow-enforcement-and-handoffs.md) explains *when* rules must be enforced in code. This file explains the **Agent SDK hook mechanism** and its two main uses: **blocking or redirecting tool calls** and **normalizing tool results**.

---

## 1. What is a hook?

A **hook** is a function you register with the Agent SDK that runs **automatically at a fixed point** in the agent's work. The model can't skip it, because it's your code running inside the harness.

The two hooks most important for this exam:

| Hook | When it runs | Typical use |
|---|---|---|
| **`PreToolUse`** | **Before** a tool call executes | Block, allow, or change the call (policy enforcement) |
| **`PostToolUse`** | **After** a tool returns, **before** the model sees the result | Transform, clean, or add to the result (data normalization) |

Other hook events exist (for example `UserPromptSubmit`, `Stop`, `SubagentStart`, `SubagentStop`, `PreCompact`), but the exam focuses on tool interception.

```
Claude requests tool ─▶ [PreToolUse hook] ─▶ tool runs ─▶ [PostToolUse hook] ─▶ Claude sees result
                            │                                   │
                       can DENY or                         can REPLACE or
                       MODIFY the call                     ADD TO the result
```

---

## 2. How hooks are registered (Python)

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

options = ClaudeAgentOptions(
    hooks={
        # matcher filters by tool name (regex or "A|B" list); omit it to match every tool
        "PreToolUse":  [HookMatcher(matcher="mcp__billing__process_refund", hooks=[refund_policy])],
        "PostToolUse": [HookMatcher(matcher="^mcp__", hooks=[normalize_mcp_output])],
    }
)
```

- `matcher` matches **tool names**. MCP tools are named `mcp__<server>__<tool>`.
- Each callback has the signature `async def callback(input_data, tool_use_id, context) -> dict`.
- `input_data` includes `hook_event_name`, `tool_name`, `tool_input` (and for `PostToolUse`, the tool's response).
- Returning `{}` means "no change; continue normally".
- If several hooks apply and **any** returns `deny`, the call is blocked.

---

## 3. Use 1: Intercepting outgoing tool calls (`PreToolUse`)

### Example: block refunds above $500 and redirect to a human

**Business rule:** Refunds over **$500** need human approval. The agent must never process them directly.

```python
REFUND_LIMIT = 500.00

async def refund_policy(input_data, tool_use_id, context):
    amount = float(input_data["tool_input"].get("amount", 0))

    if amount > REFUND_LIMIT:
        return {
            "hookSpecificOutput": {
                "hookEventName": "PreToolUse",
                "permissionDecision": "deny",
                # This reason is shown to the model, so make it actionable
                "permissionDecisionReason": (
                    f"Refunds over ${REFUND_LIMIT:.0f} need human approval. "
                    "Do not retry. Call escalate_to_human with customer_id, amount, "
                    "root cause, and your recommended action."
                ),
            }
        }
    return {}  # within limit: allow normal processing
```

**What happens at runtime:**

1. Claude calls `process_refund` with `amount: 750`.
2. The hook **denies** the call. The refund **never executes**.
3. Claude receives the denial reason as the tool result.
4. Claude follows the instruction and calls `escalate_to_human` with a structured handoff (see [04](04-workflow-enforcement-and-handoffs.md)).

This is **block + redirect**: the hook doesn't just say "no"; it points the agent to the **alternative workflow**.

### Other `PreToolUse` options

| Decision | Effect |
|---|---|
| `"deny"` | Block the call; the reason goes back to the model |
| `"allow"` | Approve without asking for permission |
| `"ask"` | Ask the user for permission |
| `updatedInput` | Change the tool's arguments before it runs (e.g. redirect a file path into a sandbox) |

---

## 4. Use 2: Normalizing tool results (`PostToolUse`)

### The problem: heterogeneous data

Real agents often use tools from **different systems**, and each system formats data differently:

| Tool (MCP server) | How it returns "order date" | How it returns "status" |
|---|---|---|
| `legacy_orders.lookup_order` | `1717200000` (Unix timestamp) | `3` (numeric code) |
| `shop_api.get_order` | `"2024-06-01T00:00:00Z"` (ISO 8601) | `"SHIPPED"` |
| `warehouse.get_shipment` | `"06/01/2024"` (US date string) | `"in_transit"` |

If the model sees all three formats, it must convert them itself. It may make mistakes: comparing a timestamp to a date string, or not knowing that status `3` means "shipped". Every prompt must explain every format.

### The fix: a `PostToolUse` hook that normalizes

The hook converts results into **one consistent format** before the model sees them.

```python
import json
from datetime import datetime, timezone

STATUS_CODES = {1: "pending", 2: "processing", 3: "shipped", 4: "delivered", 9: "cancelled"}

def to_iso8601(value) -> str:
    """Convert Unix timestamps or ISO strings to one ISO 8601 UTC format."""
    if isinstance(value, (int, float)):
        return datetime.fromtimestamp(value, tz=timezone.utc).isoformat()
    return datetime.fromisoformat(str(value).replace("Z", "+00:00")).astimezone(timezone.utc).isoformat()

async def normalize_mcp_output(input_data, tool_use_id, context):
    raw = input_data.get("tool_response")
    try:
        data = json.loads(raw) if isinstance(raw, str) else dict(raw)
    except (TypeError, ValueError):
        return {}  # not JSON we understand: leave the result unchanged

    if "order_date" in data:
        data["order_date"] = to_iso8601(data["order_date"])
    if isinstance(data.get("status"), int):
        data["status"] = STATUS_CODES.get(data["status"], f"unknown_code_{data['status']}")

    return {
        "hookSpecificOutput": {
            "hookEventName": "PostToolUse",
            # Replaces what the model sees as the tool result
            "updatedToolOutput": json.dumps(data),
        }
    }
```

**Benefits:**

- The model always sees **ISO 8601 dates** and **readable status words**.
- Prompts get simpler: no "status 3 means shipped" instructions.
- Fewer reasoning errors, because the conversion is done **deterministically** by code.

> **SDK detail:** In current docs, `updatedToolOutput` replaces the tool output for any tool. The older `updatedMCPToolOutput` (MCP tools only) is deprecated. You can also return `additionalContext` to **append** information rather than replace the result. Check the docs for your SDK version.

Other good `PostToolUse` uses: **redacting** sensitive fields (e.g. removing full card numbers before the model sees them), trimming very large results, and adding units.

---

## 5. Hooks vs prompts: choosing correctly

| Need | Use a **hook** | Use a **prompt** |
|---|---|---|
| "Never refund more than $500 without approval" | ✅ Guaranteed | ❌ Probabilistic |
| "Never show full card numbers to the model" | ✅ `PostToolUse` redaction | ❌ The model has already seen the data |
| "Convert all dates to one format" | ✅ Deterministic conversion | ⚠️ Possible, but error-prone and adds prompt length |
| "Be polite and concise" | ❌ Not a tool event | ✅ |
| "Prefer checking order history before suggesting a fix" | ⚠️ Overkill | ✅ Soft guidance |

> **Exam rule:** When **business rules require guaranteed compliance**, choose **hooks** over prompt-based enforcement. Prompts give **probabilistic** compliance; hooks give **deterministic** guarantees.

**Hook vs gate from [04](04-workflow-enforcement-and-handoffs.md):** They are the same idea. A gate is enforcement code in your tool-execution layer (any harness). A hook is the Agent SDK's built-in way to plug that code in.

---

## 6. Real-world case study

**Situation:** An insurance company's claims agent uses three MCP servers: a 1990s policy system, a modern claims API, and a payments service.

**Problems found in testing:**

1. The agent sometimes said a policy had "expired" because it compared a Unix timestamp to an ISO date incorrectly.
2. In 3 of 400 test conversations, the agent approved payouts above the $2,000 auto-approval limit, despite clear prompt instructions.

**Fixes:**

1. A **`PostToolUse` hook** on all `mcp__` tools converts every date to ISO 8601 UTC and maps numeric policy status codes to words. The date mistakes stopped.
2. A **`PreToolUse` hook** on `mcp__payments__issue_payout` denies amounts over $2,000 and tells the agent to call `escalate_to_adjuster`. Over-limit payouts dropped to **zero**, with no prompt change needed.

---

## Common Q&A

**Q: What's the difference between `PreToolUse` and `PostToolUse`?**
A: `PreToolUse` runs **before** the tool executes and can block or modify the call. `PostToolUse` runs **after** the tool returns and can change what the model sees.

**Q: If a `PreToolUse` hook denies a call, does the agent crash?**
A: No. The model receives the denial reason and continues. That's why the reason should explain the alternative (e.g. "escalate to a human").

**Q: Why normalize in a hook instead of fixing every MCP server?**
A: You may not control those servers (third-party or legacy). A hook gives one central place to normalize, without changing each system.

**Q: Can I redact data with a `PreToolUse` hook?**
A: Redaction of **results** needs `PostToolUse`, because the data only exists after the tool runs. `PreToolUse` can change the **arguments** going into a tool.

**Q: My prompt says "never refund over $500" and it works 99.5% of the time. Do I still need a hook?**
A: Yes, for a financial rule. 0.5% of a high volume is still many violations. Only the hook guarantees compliance.

**Q: Do hooks replace prompts?**
A: No. Use prompts for guidance and style, hooks for hard rules and data transformation. Most production agents use both.

---

## Key Takeaways

- **Hooks** are SDK callbacks that run at fixed points; the model can't skip them.
- **`PreToolUse`** = intercept outgoing calls → **deny, allow, ask, or modify** (e.g. block refunds > $500 and redirect to escalation).
- **`PostToolUse`** = intercept results → **normalize, redact, or add context** before the model sees them.
- Normalize **heterogeneous formats** (Unix timestamps, ISO 8601, numeric codes) in one hook instead of in prompts.
- **Guaranteed compliance → hooks. Soft guidance → prompts.**
