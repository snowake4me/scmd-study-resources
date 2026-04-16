# SCMD DEX-401 — Last Minute Final Crib

> **5 items still missed on 93% practice exams (April 15, 2026)**
> Nothing else. If it's not here, you already know it.


## 1. DB Input Parameters = `{}` Map — Always

`:named` placeholders require key-value pairs = **Map `{}`**. Never Array `[]`.

```
SQL:    WHERE firstName = :firstName AND email = :email
Params: { firstName: "William", email: "william@example.com" }
```

You built this in Studio and saw the DataWeave map expression in the Input Parameters field. Trust the curly braces.

## 2. No Error Handler ≠ No Handling

Mule **always** has an active error handler — even if you configure nothing.

**Default behavior = implicit On Error Propagate:**

- Error is **logged** (by the default handler, not your Logger)
- Flow execution **stops**
- Error **propagates** to source
- HTTP Listener returns **500**

The flow will **never** silently swallow an error or time out just because no handler is configured.

---

## 3. Child Propagate → Parent Continue = Parent's Payload (200)

```
childFlow error
  → child's On Error Propagate fires → error bubbles UP
    → parent's On Error Continue fires → sets payload
      → HTTP Listener returns 200 with Continue handler's payload
```

**The trap answer:** "End - flow" (the Set Payload *after* the Flow Reference in the parent). It **never executes** — the error interrupted the parent flow before reaching it.

You built this in Lab 1 and debugger-verified it. This is a **reading speed** issue — slow down on error chain questions.

## 4. APIKit = Validation + Code Generation + Routing

| APIKit (inside Mule app) | API Manager (control plane) |
|---|---|
| **V**alidation against RAML spec | Policy enforcement |
| **C**ode generation (scaffolds flows) | SLA tiers |
| **R**outing to correct flow | Client app access |

**APIKit has nothing to do with policies.** You saw this split tonight — Autodiscovery + API Manager = governance layer; APIKit = runtime layer.

## 5. Object Store: Just Read the Setting Name

- **"Fail on null value"** — the word **value** is literally in the name → it's about the **value**
- **"Fail if present"** — the other one → by elimination, it's about the **key**

**Don't reason. Don't build a model. Just read the words.**

*Generated from Open Brain miss logs — Practice Exams 2 & 3 (both 93.33%, April 15, 2026)*