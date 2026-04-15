# SCMD DEX-401 — Persistent Failures Cheat Sheet

**Purpose:** Exam-morning quick reference. These are concepts that have tripped you up *repeatedly* across multiple sessions and practice exams. Read slowly. Trust the rules.

---

## 1. DB Input Parameters — ALWAYS `{}`, NEVER `[]`

**Times missed:** 3+ (most persistent miss in entire study arc)

**The rule:**

```
SQL:   SELECT * FROM user WHERE firstName = :firstName AND email = :email
Input: { firstName: "William", email: "william@example.com" }
```

- Container is **`{}` Object/Map** — named `:placeholders` require key-value pairs
- **NEVER `[]` Array** — arrays are for positional parameters, which MuleSoft DB connector doesn't use
- Map **key name** must match the `:placeholder` name exactly (without the colon)
- No `attributes.` prefix unless the question explicitly says to pull from HTTP query params
- If they *did* ask for query params, it would be `attributes.queryParams.firstName` — never `attributes.firstName`

**Memory anchor:** `:named` = `{}` Map. Period. Key matches placeholder by name automatically.

---

## 2. On Error Continue — What "Continue" Actually Means

**Times missed:** 3+ (the single most dangerous conceptual gap)

**The trap:** You keep thinking "continue" means execution resumes at the next processor after the error point. **It does not.**

**What actually happens:**

| | On Error Continue | On Error Propagate |
|---|---|---|
| Flow completes normally? | **Yes** — treated as success | No — re-throws error |
| Processors AFTER the error point execute? | **NO** — they are skipped | No — error re-thrown |
| What does the caller receive? | **Error handler's payload** | Original error message |
| HTTP Listener returns: | **200** + handler's payload | **500** + original error message |

**"Continue" means:** The error handler runs, its output becomes the flow's result, and the flow exits *as if it succeeded*. The main flow processors after the failing processor **never execute**.

**The chain you keep missing (Q56 pattern):**

```
childFlow throws error
  → childFlow has On Error PROPAGATE → error bubbles UP to parent via Flow Reference
    → mainFlow has On Error CONTINUE → catches it, runs its handler
      → Client gets 200 + mainFlow's Continue handler payload
      → Set Payload "End - flow" (after Flow Reference) NEVER executes
```

---

## 3. Default Mule Error Handler — It Always Exists

**Missed:** Practice Exam 3, Q38

**The trap:** You assumed that with no error handler configured, errors would be invisible or cause a timeout.

**The rule:** When no explicit error handler exists (no flow handler, no global handler), the **Mule runtime default error handler** kicks in automatically. It behaves exactly like **On Error Propagate**:

1. Error is **logged** (by the default handler, not by any Logger processor in the flow)
2. Flow execution **stops**
3. Error **propagates** to the HTTP Listener source
4. HTTP Listener returns **500** with error message

**No handler ≠ no handling.** There is *always* an active error handler.

---

## 4. Error Handling — The Complete Decision Tree

When an error occurs, resolution follows this hierarchy:

```
1st: Try scope handler (if error is inside a Try scope)
2nd: Flow-level error handler (if flow has one)
3rd: Global error handler (ONLY if the flow has NO error handler at all)
```

**Critical gotcha:** If a flow has its own error handler but none of its scopes match the error type → the **global handler is BYPASSED**. The error propagates unhandled to the HTTP Listener (500).

**Error handler ordering:** First match wins (like firewall rules). Put specific types first, `ANY` last.

**Catching all errors in a namespace:** Type field = **empty**, When field = `#[error.errorType.namespace == "HTTP"]`. The `HTTP:*` wildcard does **NOT** work.

---

## 5. Private Flow Error Isolation

- Private flow has **On Error Continue** → calling flow **never sees the error**
- The calling flow receives the handler's payload as the Flow Reference result
- The calling flow continues normally — from its perspective, nothing went wrong

**Contrast with On Error Propagate in child flow:**
- Private flow has **On Error Propagate** → error **bubbles up** to calling flow
- Calling flow's error handler (if any) fires

---

## 6. Scope Boundaries — What Crosses What

| Boundary | Variables | Attributes | Payload |
|----------|-----------|------------|---------|
| **Flow Reference** | ✅ Travel both ways | ✅ Travel both ways | ✅ Replaced by child's output |
| **HTTP Request** | ❌ Die at boundary | ❌ Die at boundary | ❌ New response payload |
| **Batch Job** | ❌ Record scope isolated | N/A | Per-record only |

**Visual cue on exhibits:** Globe icon = HTTP Request = transport boundary = variables die. Flow Reference icon (small arrow box) = same execution context = variables survive.

**Target attribute:** When set on a connector (e.g., HTTP Request, DB Select), the result goes into the named **variable** and the **original payload is preserved**.

---

## 7. For Each Scope — Payload Restored, Variables Survive

- After For Each completes, **payload is restored** to whatever it was *before* For Each
- **Variables** set inside For Each **survive** and are accessible after the scope (last iteration value)
- The `counter` variable is **removed** after For Each exits
- For Each **requires an Array** — passing an Object/Map = **zero iterations** (no error, silent skip)

---

## 8. Batch Job — On Complete ≠ Client Response

- Batch Job processes records **asynchronously**
- **On Complete** receives an `ImmutableBatchJobResult` — a summary report (counts of successful/failed records), **never the processed records themselves**
- Client response must be sent **before** the async batch handoff (e.g., "Request accepted")
- Variables modified inside batch steps do **NOT** propagate back to the outer flow scope
- `Max Failed Records = 0` means **unlimited** failures allowed (not "stop on first failure")

---

## 9. Scatter-Gather Output — Object, Not Array

- Output is an **Object** with string keys `"0"`, `"1"`, `"2"` — each containing a full **Mule event**
- **NOT** an Array, **NOT** just payloads
- Each route receives the **entire** event and processes in parallel — not split

---

## 10. Object Store — Fail If Present / Fail On Null Value

**Times missed:** 3+ (persistent under pressure)

- **"Fail if present"** = guards against **duplicate KEYS** — "is this key already in the store?"
- **"Fail on null value"** = guards against **storing nothing** — "is the value I'm storing null?"

**Memory anchor:** Keys are never null (you always provide one). Values can be null. That's the logic.

---

## 11. DataWeave Quick Hits

- `typeOf()` on a Set Payload literal string → always returns `String` regardless of content appearance
- `++` concatenates Arrays/Objects/Strings — but **fails with type error** if you mix String + Object
- XML multi-value selector: `payload.root.*item` (gets all repeated `<item>` elements)
- XML attribute accessor: `payload.root.element.@attributeName`
- DB Select always returns **Java List of Maps** (via JDBC) — use Transform Message to convert to JSON/XML

---

## 12. Quick-Fire Fact Check

| If you see... | The answer is... |
|---|---|
| DB Input Parameters container | `{}` Map — never `[]` |
| On Error Continue — does flow resume after error? | **No** — handler output = flow result |
| No error handler configured at all | Default handler = On Error Propagate behavior = 500 |
| For Each given an Object | Zero iterations, no error |
| Scatter-Gather output type | Object (not Array) |
| On Complete in Batch Job | Gets summary report, not records |
| `Scheduler fixedFrequency` and prior run still going | **Fires anyway** — doesn't wait |
| Object Store "Fail if present" | Guards duplicate **keys** |
| Flow Reference — do variables cross? | **Yes** — both directions |
| HTTP Request — do variables cross? | **No** — transport boundary |

---

*Last updated: April 15, 2026 — Practice Exam 3 score: 93.33% (56/60)*
*Exam: April 16, 2026 at 12:15 PM*
