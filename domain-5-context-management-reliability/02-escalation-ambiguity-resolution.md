# 02 · Escalation and Ambiguity Resolution Patterns

**Exam link:** Task Statement 5.2: *Design effective escalation and ambiguity resolution patterns.*
**Prerequisites:** Domain 1 [04 · Enforcement and Handoffs](../domain-1-agentic-architecture-orchestration/04-workflow-enforcement-and-handoffs.md) (structured handoffs), Domain 4 [02 · Few-Shot](../domain-4-prompt-engineering-structured-output/02-few-shot-prompting.md).
**Scope note:** Domain 1 covered **how** to hand off (the structured summary). This file covers **when** to escalate, when **not** to, and how to handle **ambiguous** situations.

---

## 1. Two ways to get escalation wrong

| Mistake | Example | Cost |
|---|---|---|
| **Escalating too much** | Sends simple, standard cases to humans (a damaged item with photo evidence) | Low first-contact resolution; human agents overloaded |
| **Escalating too little** | Tries to handle a policy exception itself, or ignores "I want a person" | Wrong decisions; angry customers |

Both often happen **at the same time** when escalation rules are vague: the agent escalates easy cases and keeps hard ones.

---

## 2. Good escalation triggers

Escalate when one of these is true:

| Trigger | Example |
|---|---|
| **Customer explicitly asks for a human** | "I want to speak to a person." |
| **Policy exception or policy gap**: the request isn't clearly covered by policy, or needs an exception | Customer asks you to match a **competitor's** price, but policy only covers price adjustments for **your own site** |
| **No meaningful progress possible** | Required system is down; information can't be obtained; repeated failed attempts |

**Not** good triggers on their own:

| Weak trigger | Why it's unreliable |
|---|---|
| **Customer sentiment** ("sounds angry") | Angry customers often have **simple** problems the agent can fix fast; calm customers can have very complex ones |
| **Model's self-reported confidence** | Not calibrated; the model can be confidently wrong or unsure about easy cases |
| **"The case seems complex"** with no definition | Vague, so it is applied inconsistently |

> **Exam point:** Sentiment and self-reported confidence are **unreliable proxies for case complexity**. Use **explicit criteria**.

---

## 3. Make criteria explicit, with few-shot examples

Write escalation criteria into the **system prompt**, and add **few-shot examples** showing escalate vs resolve:

```text
ESCALATE to a human (call escalate_to_human) when:
1. The customer explicitly asks for a human, manager, or person. Do this immediately.
2. The request needs a policy exception, or policy does not clearly cover it.
3. You cannot make progress (required tool unavailable after retries, or missing info you cannot obtain).

RESOLVE yourself when the request is clearly covered by policy and your tools,
even if the customer is upset.

Example A: resolve
Customer: "This is ridiculous, my blender arrived cracked!! Photo attached."
Policy: damaged items with photo evidence → free replacement.
Action: Apologize, acknowledge frustration, create the replacement. Do not escalate.
Why: Standard case fully covered by policy; frustration alone is not a reason to escalate.

Example B: escalate
Customer: "BestDeals sells this for $40 less. Match it."
Policy: price adjustments apply only to price drops on our own site.
Action: Escalate with a structured handoff.
Why: Policy is silent on competitor price matching, so this is a policy gap.

Example C: escalate immediately
Customer: "I don't want to talk to a bot. Get me a human."
Action: Escalate now. Do not investigate first or try to persuade.
Why: Explicit request for a human.
```

---

## 4. Honor explicit requests for a human immediately

When a customer **explicitly** asks for a human:

- ✅ **Escalate right away.**
- ❌ Don't "just quickly check the order first".
- ❌ Don't try to talk them out of it.

Investigating first ignores the customer's stated wish and adds frustration. (The agent can still pass along any information it already has in the structured handoff.)

### But frustration alone isn't a request

There's an important difference:

| Customer says | Agent should |
|---|---|
| "This is so frustrating, my order is late again!" | **Acknowledge the frustration** and **offer to resolve** it (if within capability). |
| "This is frustrating. **I want a human.**" | **Escalate immediately.** |
| (After the agent offers help) "No, I **still** want a person." | **Escalate**: the customer **reiterated** their preference. |

```text
Customer: "Ugh, my package is late AGAIN. This is so annoying."
Agent:    "I'm sorry, that's really frustrating, especially twice. I can check where it is
           right now and arrange a refund of the shipping fee. Would you like me to do that,
           or would you prefer I connect you with a member of our team?"
```

Offering the choice respects the customer and still resolves simple issues quickly.

---

## 5. Policy gaps → escalate, don't improvise

When policy is **ambiguous** or **silent** on a specific request, the agent should **not invent policy**. Inventing policy risks unfair or costly decisions and inconsistent treatment of customers.

**Example:** Policy says "We adjust prices if **our** price drops within 14 days." A customer asks for a match with a **competitor's** price.

- ❌ Agent decides "close enough" and gives a $40 credit.
- ❌ Agent flatly refuses ("not allowed") when the policy doesn't actually say that.
- ✅ Agent explains that competitor matching isn't covered by the standard policy and **escalates** to someone who can decide.

---

## 6. Ambiguity: multiple matches → ask, don't guess

A lookup like `get_customer(name="John Smith")` may return **several matches**.

**Wrong:** pick one using a heuristic:
- "the most recent account",
- "the one with the most orders",
- "the first result".

Any of these can lead to **showing or changing the wrong person's data**: a privacy and correctness failure.

**Right:** **ask for additional identifiers**:

```text
Agent: "I found more than one account under that name. To make sure I have the right one,
        could you share the email address or the last 4 digits of the phone number on the account?"
```

The same rule applies to other ambiguity. Say there are two open orders and the customer says "cancel my order". Ask which one (see the few-shot pattern in Domain 4 · [02](../domain-4-prompt-engineering-structured-output/02-few-shot-prompting.md)).

---

## 7. Real-world case study

**Situation:** An electronics retailer's agent had **55% first-contact resolution** (target 80%). Logs showed:
- It escalated many **standard damage replacements** (photo provided, clearly covered by policy), often because customers sounded upset.
- It **tried to handle** price-match requests against competitors itself, inventing credits.
- When two accounts matched a name, it picked the newest one, once exposing another customer's order history.

**Changes:**
1. **Explicit escalation criteria** + **few-shot examples** (resolve vs escalate vs escalate-immediately).
2. Removed "escalate if the customer seems upset" and the confidence-based rule.
3. Competitor price matches → **escalate as a policy gap**.
4. Multiple matches → **ask for email or phone digits**.

**Result:** First-contact resolution rose toward the target, policy-exception decisions moved to humans, and wrong-account incidents stopped.

---

## Common Q&A

**Q: What are good escalation triggers?**
A: Explicit request for a human, policy exceptions or gaps, and inability to make meaningful progress.

**Q: Should an angry customer be escalated?**
A: Not automatically. Acknowledge the frustration and resolve it if it's within capability. Escalate if they **ask** for a human (or repeat that request).

**Q: Why not escalate when the model reports low confidence?**
A: Self-reported confidence is poorly calibrated and doesn't reliably reflect case complexity.

**Q: The customer asks for a human. Should the agent first check their order to save time?**
A: No. Honor the request **immediately**.

**Q: Policy doesn't mention competitor price matching. What should the agent do?**
A: Treat it as a **policy gap** and escalate. Don't invent a policy.

**Q: The lookup returns three matching customers. What now?**
A: Ask for **additional identifiers** (email, phone digits, order number). Never pick by heuristic.

---

## Key Takeaways

- Escalate on **explicit triggers**: **human requested**, **policy exception/gap**, **no progress possible**.
- **Sentiment** and **self-reported confidence** are **unreliable** escalation signals.
- Put criteria **in the system prompt with few-shot examples**.
- **Explicit request for a human → escalate immediately**; frustration alone → acknowledge and offer to resolve.
- **Policy silent or ambiguous → escalate**, don't improvise.
- **Multiple matches → ask for more identifiers**, never guess.
