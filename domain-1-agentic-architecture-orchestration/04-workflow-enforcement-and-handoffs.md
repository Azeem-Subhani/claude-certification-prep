# 04 · Multi-Step Workflows: Enforcement and Handoffs

**Exam link:** Task Statement 1.4: *Implement multi-step workflows with enforcement and handoff patterns.*
**Prerequisites:** [00](00-foundations-and-glossary.md) (deterministic vs probabilistic), [01](01-agentic-loops.md).
**Scope note:** This file explains **why and where** to enforce rules, using plain Python gates. [05](05-agent-sdk-hooks.md) shows the **Agent SDK hook** mechanism for the same goal.

---

## 1. The problem: the model can skip steps

Many business processes have a required order:

- Verify identity **before** showing account data.
- Check order status **before** issuing a refund.
- Get manager approval **before** a large credit.

You can *tell* Claude the order in the system prompt. Claude will follow it almost every time. But "almost" is the problem.

> **Prompt instructions have a non-zero failure rate.** On a high-volume system, a 1% skip rate means hundreds of wrong actions per week.

---

## 2. Two ways to control step order

| | **Prompt-based guidance** | **Programmatic enforcement** |
|---|---|---|
| How | System prompt, few-shot examples, instructions | Code that checks conditions before allowing a tool call (prerequisite gates, hooks) |
| Type | Probabilistic | Deterministic |
| Can the model bypass it? | Yes, sometimes | No |
| Best for | Style, tone, preferred order, soft guidance | Security, money, legal and compliance rules |
| Cost to change | Edit text | Edit and test code |

### When is deterministic enforcement required?

Ask: **"What happens if this step is skipped once?"**

- Wrong tone in an email → prompt guidance is fine.
- Refund sent to the wrong customer, data shown to the wrong person, regulatory breach → **programmatic enforcement is required**.

> **Exam pattern:** A question may describe a required sequence for **critical business logic** (identity, money, compliance). If the agent *sometimes* skips a step, the correct answer is a **programmatic prerequisite**. Answers like "make the prompt stronger", "add few-shot examples", or "say it is mandatory in capitals" are distractors. They reduce the failure rate but cannot remove it.

---

## 3. Pattern: programmatic prerequisite gate

A **prerequisite gate** blocks a tool call until another step has completed successfully. The gate lives in *your* code, between Claude's tool request and the real tool.

**Example rule:** `lookup_order` and `process_refund` must not run until `get_customer` has returned a **verified customer ID**.

```python
class SessionState:
    """Facts the harness has verified. Kept in code, not trusted from model text."""
    def __init__(self):
        self.verified_customer_id: str | None = None


GATED_TOOLS = {"lookup_order", "process_refund"}


def execute_tool_with_gates(name: str, tool_input: dict, state: SessionState) -> dict:
    # Gate: block downstream tools until identity is verified
    if name in GATED_TOOLS and state.verified_customer_id is None:
        return {
            "is_error": True,
            "content": "Blocked: call get_customer and verify the customer before using this tool.",
        }

    # Gate: the refund must be for the verified customer, not an ID the model typed
    if name == "process_refund" and tool_input.get("customer_id") != state.verified_customer_id:
        return {"is_error": True, "content": "Blocked: customer_id does not match the verified customer."}

    result = run_real_tool(name, tool_input)  # your implementation

    # Record verified facts from the tool's real output, not from the model's claims
    if name == "get_customer" and result.get("verified"):
        state.verified_customer_id = result["customer_id"]

    return {"is_error": False, "content": result}
```

**Design points:**

- The **state** is set from **real tool output**, never from what the model says ("I have verified the customer").
- A blocked call returns a clear **error message** so Claude can recover: it will call `get_customer` next.
- The gate is **deterministic**: it works every time, even if the prompt is ignored.

---

## 4. Pattern: decomposing multi-concern requests

Real customers often raise several issues in one message:

> *"My order #4471 arrived damaged, I was charged twice for it, and I want to change my email address."*

That is **three concerns**: damage claim, billing dispute, account change.

**Recommended approach:**

1. **Decompose**: identify each concern as a separate item.
2. **Investigate each item**, in parallel where possible, using the **shared context** (same verified customer, same order data). Don't verify the customer three times.
3. **Synthesize** one unified resolution that answers all concerns together.

```
"damaged + double charge + email change"
            │
     decompose into items
   ┌────────┼─────────────┐
   ▼        ▼             ▼
damage   billing       account
claim    dispute       update
   │        │             │
   └── shared context: verified customer, order #4471 ──┘
            │
     synthesize one reply:
"I've started a replacement, refunded the duplicate $89 charge,
 and updated your email. Here's a summary…"
```

**Why not handle only the first issue?** The customer must write again, which hurts first-contact resolution, a key support metric.
**Why not three separate conversations?** Context (identity, order) would be repeated, and the replies might conflict.

---

## 5. Pattern: structured handoff to a human

Sometimes the agent must **escalate**: the case is outside policy, too complex, or the customer asks for a person.

**Critical fact:** the human agent usually **does not have the conversation transcript**, or has no time to read it. The handoff must stand on its own.

### What a structured handoff contains

| Field | Example |
|---|---|
| Customer ID | `C-20931` (verified) |
| Issue summary | Double charge on order #4471 + damaged item |
| Root cause analysis | Payment retry after gateway timeout created a second charge |
| Actions already taken | Duplicate charge confirmed; replacement order created |
| Amount involved | $89.00 refund pending |
| Recommended action | Approve $89 refund (exceeds agent's auto-limit due to account flag) |
| Why escalated | Account has a fraud-review flag; policy requires human approval |

```python
escalation = {
    "customer_id": "C-20931",
    "issue_summary": "Double charge on order #4471; item arrived damaged",
    "root_cause": "Gateway timeout caused a duplicate payment capture",
    "actions_taken": ["Verified identity", "Confirmed duplicate charge", "Created replacement order"],
    "refund_amount": 89.00,
    "recommended_action": "Approve refund of duplicate charge",
    "escalation_reason": "Account fraud-review flag requires human approval",
}
# Passed to the escalate_to_human tool, which creates a ticket in the support system
```

**Bad handoff:** *"Customer is unhappy, please help."* The human must start again from zero and ask the customer to repeat everything.

### When to escalate

Escalate based on **clear criteria**, for example:

- The customer explicitly asks for a human.
- The action is outside policy or above a limit.
- The agent cannot make progress (repeated failures, missing information it cannot obtain).

Avoid weak signals such as "the customer sounds angry" alone, or the model's self-reported confidence score. These are unreliable.

---

## 6. Real-world case study

**Situation:** A fintech support agent processes refunds. Logs show that in about **1 in 12 conversations**, the agent called `lookup_order` using only the name the customer typed, without calling `get_customer` first. Some refunds went to the wrong account.

**Attempt 1:** The team added *"You MUST call get_customer first"* to the prompt, plus examples. Skips dropped but did not stop.

**Attempt 2 (fix):** A **prerequisite gate** that blocks `lookup_order` and `process_refund` until `get_customer` returns a verified ID. It also checks that the refund's customer ID matches it. Skips fell to **zero**, because the code makes skipping impossible.

**Extra improvement:** For cases needing escalation, the agent now fills in a **structured handoff** form. Human agents report much shorter handling times because they no longer re-interview customers.

---

## Common Q&A

**Q: If the prompt already says "always verify first", why add code?**
A: Because prompts are probabilistic. For identity, money, and compliance, even rare failures are unacceptable. Code makes the rule deterministic.

**Q: Isn't a routing classifier that limits which tools are available a good alternative?**
A: It can help with tool *selection*, but it doesn't guarantee *order*. A gate that checks "has verification happened?" directly enforces the requirement.

**Q: Where should "verified" state be stored?**
A: In the harness (your code), set from real tool output. Never trust the model's own statement that a step was done.

**Q: What should a blocked tool call return?**
A: A clear error result explaining what is missing, so the model can take the correct next step.

**Q: Why does the handoff need root cause and recommended action, not just the customer ID?**
A: The human usually can't see the transcript. Without a summary, they have to redo the investigation.

**Q: Should a multi-concern request be split into separate tickets?**
A: Investigate the concerns as separate items, but share context and give the customer **one unified answer**.

---

## Key Takeaways

- **Prompts guide; code guarantees.** Use programmatic enforcement when skipping a step is unacceptable.
- A **prerequisite gate** blocks downstream tools until verified state (from real tool output) exists.
- Blocked calls should return **helpful errors** so the agent can recover.
- **Decompose multi-concern requests**, investigate with **shared context**, and reply with **one unified resolution**.
- **Structured handoffs** (ID, root cause, actions, amount, recommendation, reason) let humans act without the transcript.
