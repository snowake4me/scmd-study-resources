# SCMD DEX-401 — Retake Morning Review

**Exam: Thursday April 16, 2026 at 12:15 PM**

Read this once. Trust the rules. Slow-read the question stems.

## The #1 Stickiest Concept — "On Error Continue" Does NOT Resume

Missed this pattern **twice today**. Drill it into muscle memory.

**When childFlow has Propagate, mainFlow has Continue:**

1. Child error → child's **Propagate** bubbles error UP to main
2. Main's **Continue** catches it → runs its handler
3. Client gets **200 OK** + main's Continue handler payload
4. Any `Set Payload` in mainFlow *after* the Flow Reference **NEVER executes**

**"Continue" means:** the error handler's output becomes the flow's result. The flow exits *as if it succeeded*. Execution does **NOT** resume after the failing processor.

| | Continue | Propagate |
|---|---|---|
| Client gets | **200** + handler payload | **500** + original error |
| Flow after error runs? | **No** | No |

## Error Handler Resolution

1. **Try scope** handler (if inside Try)
2. **Flow** handler (if flow has one)
3. **Global** handler — *ONLY if flow has NO handler at all*

⚠ Flow has handler but none match? **Global is bypassed** — error propagates unhandled.

⚠ No handler configured anywhere? **Mule default** kicks in = acts like Propagate = logged + 500.

⚠ Namespace catch: Type = **empty**, When = `#[error.errorType.namespace == "HTTP"]`. `HTTP:*` wildcard does NOT work.

## DB Input Parameters — ALWAYS `{}`

```
SQL:   WHERE firstName = :firstName AND email = :email
Input: { firstName: "William", email: "william@example.com" }
```

- Container is **always `{}` Map**, NEVER `[]`
- Map key matches `:placeholder` name (no colon, no `attributes.` prefix)

## Scope Boundaries

| Boundary | Variables cross? |
|---|---|
| Flow Reference (small arrow icon) | **Yes** — both directions |
| HTTP Request (globe icon) | **No** — transport boundary |
| Batch record scope | **No** — isolated per record |

**Target attribute:** result → variable, **original payload preserved**.

## Iteration & Routing

- **For Each** — payload **restored** after scope; variables **survive**; counter removed
- **For Each** given Object/Map = **zero iterations** (silent, no error)
- **Batch On Complete** — gets **summary report**, NEVER the records
- **Batch Max Failed Records = 0** means stop on ANY error
- **Scatter-Gather output** — **Object** of Mule events (keys `"0"`, `"1"`, `"2"`), NOT Array
- **DB Select** returns **Java List of Maps**, NOT JSON

## Object Store Store Operation

- **"Fail if present"** → guards duplicate **KEYS** ("is this key already in the store?")
- **"Fail on null value"** → guards **storing null** ("is the value I'm storing null?")

Keys are never null. Values can be. That's the logic.

## APIKit & REST Connect

- **APIKit = 3 functions:** Automatic Validation, Code Generation, Intelligent Routing
  - NOT Policy Adherence (that's API Manager)
- **REST Connect auth:** Basic, OAuth 2.0, Pass Through
  - NOT TLS Custom

## Autodiscovery

API ID in the Mule app pairs the deployed runtime with API Manager. You **DO** need the API ID — that's *how* Autodiscovery works.

## DataWeave Quick Hits

- `map (value, index) -> { field: value.car_type }` — use named params **directly**, NO `$` prefix
- `$` and `$$` shortcuts only work when you **don't** name params
- `typeOf()` on a Set Payload literal string → `String`, regardless of content
- `++` mixing String + Object → **type error**

## ⚠ The Meta-Rule: Read Carefully

Your miss pattern is reading too fast once questions feel familiar.

- **Set Variable ≠ Set Payload** — Set Variable does NOT change the payload
- **Debugger breakpoint** shows state **BEFORE** that processor executes
- Look at the **icon** on exhibits — globe = HTTP Request (transport), arrow-box = Flow Reference (same context)
- Before clicking: re-read the question stem, especially negations ("which is NOT", "which does NOT")

---

**You've got this, Billy. 93% twice in a row. Go get it.**
