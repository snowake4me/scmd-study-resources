# Salesforce Certified MuleSoft Developer — SCMD / DEX-401
## Complete Exam Reference
*All 12 sections · All key concepts · Practice exam verified · Common traps*

---

## Exam Weighting Summary

| Section | Weight |
|---------|--------|
| 1: Creating Application Networks | 5% |
| 2: Designing APIs | 10% |
| 3: Accessing and Modifying Mule Events | 10% |
| 4: Structuring Mule Applications ⚑ | 10% |
| 5: Building API Implementation Flows | 5% |
| 6: Using Connectors | 10% |
| 7: Processing Records ⚑ | 10% |
| 8: Transforming Data with DataWeave ⚑ | 15% |
| 9: Routing Events ⚑ | 10% |
| 10: Handling Errors ⚑ | 10% |
| 11: Debugging and Troubleshooting | 5% |
| 12: Deploying and Managing APIs | 10% |

---

## Section 1: Creating Application Networks (5%)

### API-Led Connectivity — 3 Layers

| Layer | Purpose | Examples |
|-------|---------|---------|
| System APIs | Expose backend systems without business logic | ERP, DB, mainframe connectors |
| Process APIs | Orchestrate system APIs with business logic | Order processing, validation |
| Experience APIs | Format data for specific consumers | Mobile app API, web API |

- **Key principle:** Changing a product API's **implementation** does not require changes in consuming APIs or their applications — as long as the interface (contract) stays the same.
- **Center for Enablement (C4E):** The MuleSoft-recommended organizational structure for sharing reusable assets and avoiding duplicate API development.

> ⚠ **Exam trap:** "Center of Excellence" is a common distractor. The correct MuleSoft term is **Center for Enablement (C4E)**.

### Modern API Creation — API-First Process

1. Create an API specification and get feedback from stakeholders
2. Publish to Exchange and build a prototype/mock
3. Implement the API based on the approved spec
4. Deploy and manage via API Manager

### Anypoint Platform Components

| Component | Purpose |
|-----------|---------|
| Design Center | Build RAML/OAS API specs and basic Mule flows in browser |
| Anypoint Exchange | Publish/consume APIs, connectors, templates, examples — asset catalog |
| API Manager | Apply policies, SLA tiers, analytics, manage API instances |
| Runtime Manager | Deploy and monitor Mule apps (CloudHub & on-prem) |
| Anypoint Studio | Eclipse-based IDE for local Mule flow development |
| Access Management | Users, roles, business groups, environments |

### Environments and Business Groups

- Business groups organize assets and permissions hierarchically
- Environments (Dev, Test, Prod) exist within a business group
- APIs are promoted across environments — not redeveloped from scratch

---

## Section 2: Designing APIs (10%)

### API Spec Formats

| Format | Notes |
|--------|-------|
| RAML 1.0 | MuleSoft-preferred; used throughout DEX-401 course |
| OAS 3.0 (OpenAPI) | Industry standard; increasingly used in MuleSoft shops |
| OAS 2.0 (Swagger) | Older; still encountered but not preferred |

### RESTful URI Design

- **Resource ID in path (correct):** `/customers/1234`
- **Query param for ID (wrong):** `/customers?custid=1234`
- Use nouns not verbs: `/orders` not `/getOrders`
- Hierarchical: `/stores/{storeId}` then query params for filtering

### RAML 1.0 Key Syntax

```yaml
#%RAML 1.0
title: Flights API
version: v1
baseUri: https://api.example.com/{version}
/flights:
  get:
    queryParameters:
      destination:
        type: string
        enum: [SFO, LAX, CLE]
  /{flightID}:        # URI param — nested under /flights
    get:
```

- **URI parameter syntax:** `{paramName}` in curly braces — NOT `(paramName)` or `${paramName}`
- **Indentation rules:** Method (get/post) must be indented under its resource. Sub-resources indented under parent resource.

### RAML !include Syntax

- **Full path from project root:** `examples: !include examples/BankAccountsExample.raml`
- **NOT:** `examples: !include BankAccountsExample.raml` (missing folder path)
- **NOT:** `examples: #import ...` (no `#import` in RAML)

### RAML Reuse Patterns

- **Traits:** Reusable method fragments (e.g. pageable, secured) — applied with `is:`
- **Resource Types:** Reusable resource patterns (e.g. collection, member)
- **Libraries:** Extract types/traits into .raml files, reference with `!include`
- **Data Types:** Defined under `types:` section — reuse across resources

### HTTP Methods — REST Semantics

| Method | Action | Idempotent? |
|--------|--------|------------|
| GET | Retrieve resource | Yes |
| POST | Create new resource | No |
| PUT | **Replace entire resource (full update)** | Yes |
| PATCH | **Partial update (specific fields only)** | No |
| DELETE | Delete resource | Yes |

> ⚠ **PUT = full replacement. PATCH = partial update. Common exam trap.**

---

## Section 3: Accessing and Modifying Mule Events (10%)

### Mule Event — 4 Elements

| Element | Description | Mutable? |
|---------|-------------|---------|
| payload | The message data (body) | Yes — Set Payload or Transform Message |
| attributes | Source metadata (HTTP headers, params, etc.) | No — read only |
| vars | Flow variables — persist across the flow | Yes — Set Variable |
| error | Error details — only populated in error handlers | No — read only |

### HTTP Listener Attributes — Access Patterns

| What you want | Correct expression | Mule 3 (wrong — eliminate) |
|--------------|-------------------|--------------------------|
| URI path param {ID} | `attributes.uriParams.ID` | `attributes.'http.uri.params'.ID` |
| Query param ?dest=SFO | `attributes.queryParams.dest` | `inboundProperties.'http.query.params'` |
| HTTP header | `attributes.headers.'content-type'` | `inboundProperties.'content-type'` |
| HTTP method | `attributes.method` | `inboundProperties.'http.method'` |
| Request path | `attributes.requestPath` | `inboundProperties.'http.request.path'` |

> ⚠ **ELIMINATE on sight:** `inboundProperties`, `outboundProperties`, `flowVars`, `sessionVars`, `message.payload` — all Mule 3 only.

### Variables — Access Syntax

- **Correct:** `vars.bookISBN`
- **Wrong:** `variables.bookISBN` (wrong prefix)
- **Wrong:** `bookISBN` (no prefix — won't resolve)
- **Wrong:** `attributes.bookISBN` (attributes = HTTP metadata, not vars)

### DataWeave Expressions in Logger / Set Payload

| Goal | Correct syntax |
|------|---------------|
| Inline expression in string | `'The year is #[payload.year]'` |
| Full DataWeave expression | `#["The year is " ++ payload.year]` |
| Variable access | `#[vars.myVar]` |
| String concatenation | `#[payload.first ++ " " ++ payload.last]` |

> ⚠ Use `++` for string concatenation, **NOT** single `+`.

### Target Attribute

- **Purpose:** Redirect a processor's output to a variable instead of replacing the payload
- **Effect:** Original payload is **PRESERVED**. Result stored in `vars.targetName`
- **Access:** `vars.targetName` (not payload)

> ⚠ **Q13 trap:** When HTTP Request has a target attribute set, the payload at the next processor is STILL the original payload — not the HTTP response. The response is in the variable.

### Scope Boundaries — What Travels Where ⚑

| Boundary | What crosses | What stays behind |
|----------|-------------|------------------|
| Flow Reference (same app) | payload, attributes, vars — ALL travel both ways | Nothing — same Mule event |
| HTTP Request (network call) | payload (as request body) only | attributes, vars stay in calling flow |
| After HTTP Request returns | payload = HTTP response body; NEW attributes | Original query params/headers gone |

> ✓ **Q16:** After HTTP Request in main flow — vars from before the HTTP Request ARE still accessible in the main flow after it returns. But the original attributes (color query param) are replaced by HTTP response attributes.

> ✓ **Q19:** Flow Reference → child flow gets payload + attributes + vars. All three.

---

## Section 4: Structuring Mule Applications ⚑ (10%)

### Flow Types

| Type | Has source? | Own error handler? | Shares caller's error handler? |
|------|------------|-------------------|-------------------------------|
| Flow | Yes | Yes | No |
| Sub-flow | No | **No** | **Yes — always inherits** |
| Private flow | No | Yes — optional | Only if no own handler |

> ⚠ **Sub-flow vs Private flow:** Sub-flows ALWAYS use the calling flow's error handler. Private flows CAN have their own. This distinction is heavily tested.

### HTTP Listener Path Parameter Syntax

- **Capture URI segment as parameter:** `/accounts/{ID}`
- **Access it:** `#[attributes.uriParams.ID]`
- **Wrong:** `/accounts/ID` (literal, not a param)
- **Wrong:** `/accounts/#[ID]` (DataWeave not used in path config)

### Global Elements and Configuration Files

- **Global elements:** Reusable connector configs — HTTP Request config, DB pool, TLS context
- **Config file location:** Specified in a **global element** (Configuration Properties)
- **Property access syntax:** `${propertyFile.key}` (dot notation, dollar sign, curly braces)
- **pom.xml:** Maven dependencies ONLY — never flow config, never property file location
- **mule-artifact.json:** Mule app metadata — NOT property file location

> ⚠ **Q21 trap:** Config file location goes in a **GLOBAL ELEMENT**, not pom.xml and not mule-artifact.json.

> ⚠ **Q20 trap:** Property placeholder syntax is `${training.host}` with dot notation, NOT `#[training.host]` (that's DataWeave) and NOT `${training:host}` (wrong separator).

### Minimum Global Elements for Multiple HTTP Listeners

- HTTP Listeners sharing the same host AND port → share **ONE** global element
- Different ports → different global elements (one per unique host:port)

> ✓ **Q17:** If you have listeners on port 8081 and port 8082, minimum = 2 global elements.

### APIKit Router

- Auto-generated when importing a RAML spec into Studio
- Creates **ONE flow per HTTP method + resource combination**
- GET /flights and POST /flights → 2 separate generated flows
- HTTP Listener base path: `/api/` then APIKit Router maps to resource paths
- Full URL: listener base (`/api/`) + resource path (`/flights`) = `/api/flights`
- APIKit validates requests against RAML — invalid enum values → `APIKIT:BAD_REQUEST` before reaching flow

### Flow Reference vs HTTP Request — Side by Side ⚑

| Characteristic | Flow Reference | HTTP Request |
|---------------|---------------|-------------|
| Communication | Internal — same Mule app | External — HTTP network call |
| Variables travel? | Yes — both directions | No — variables stay in calling flow |
| Attributes travel in? | Yes | No — replaced by HTTP response attrs on return |
| Error propagation | Propagates unless caught in called flow | HTTP errors become HTTP:* error types |
| Called flow needs source? | No — private/sub-flow | Yes — HTTP Listener on receiving end |

---

## Section 5: Building API Implementation Flows (5%)

### HTTP Listener Base Path

- **For multiple endpoints sharing a base:** `/apis/` (e.g. `/apis/orders` and `/apis/customers`)
- **NOT:** `/apis/*` (wildcard goes on the per-listener path, not the shared config base)

### APIKit-Generated Flows

- One private flow per HTTP method + resource combination in the RAML
- Flow name pattern: `get:\flights:api-config`
- RAML with 2 resources × 2 methods = 4 generated flows

> ✓ **Q23:** Count method+resource combinations in the RAML, not just resources.

### REST Connect / Exchange Connector Generation

1. Create API spec (RAML or OAS) in Design Center
2. **Publish the API spec to Anypoint Exchange**
3. Exchange auto-generates a REST Connect connector
4. Connector appears in Studio palette — drag and drop

> ⚠ **Q25:** You MUST publish to Exchange first. Simply adding to `src/main/resources` is not enough.

### HTTP Methods — Implementation Mapping

| HTTP Method | REST Semantics | APIKit generates flow for each? |
|-------------|---------------|-------------------------------|
| GET | Retrieve — no body in request | Yes |
| POST | Create — body contains new resource | Yes |
| PUT | Full replace — body is complete replacement | Yes |
| PATCH | Partial update — body has changed fields only | Yes |
| DELETE | Remove resource | Yes |

---

## Section 6: Using Connectors (10%)

### File Connector

- **File List:** Returns Array of Mule message objects — NOT strings or filenames
- **File Read:** Returns single message; file renamed with `.bak` extension after reading (by default)
- **File Write:** Writes payload to specified path

> ✓ **Q26:** After File Read, the source file is renamed to `.bak` — it's not deleted, and its content is unchanged.

### Database Connector

- **Select (no rows match):** Returns **empty array `[]`** — NOT null, NOT false, NOT exception
- **Select (rows match):** Returns **Java List of Maps** (JDBC ResultSet) — NOT JSON
- **Insert/Update/Delete:** Returns affected row count
- **Input Parameters (named):** Map of parameter names matching SQL `:placeholders`

> ⚠ DB Select returns Java, not JSON. Always need Transform Message to convert to JSON/XML output.

### DB Insert — Input Parameters Pattern

```
SQL: INSERT INTO orders VALUES (:oid, :custId, :status)
Input Parameters: #[{ oid: payload.oid, custId: payload.custId, status: payload.status }]
```

- Key names in the map must match the `:named` placeholders in SQL
- NOT a positional array — use a named map

### Saving Payload Before Overwrite

- **Pattern:** Set Variable → store DB result → HTTP Request → combine `vars.dbResult` with new payload
- **Use Target Attribute:** On DB Select, set `target=dbResult` — payload preserved, result in `vars.dbResult`

> ✓ **Q31:** Save DB payload to a variable BEFORE the HTTP Request. The idiomatic Mule 4 approach.

### JMS Connector — Publish vs Publish-Consume

| Operation | Behavior | Payload after operation |
|-----------|----------|------------------------|
| JMS Publish | Fire and forget — sends message, doesn't wait | **Unchanged** (original payload) |
| JMS Publish-Consume | Sends and waits for reply (synchronous) | Reply message payload |
| JMS On New Message | Trigger/listener — fires flow when message arrives | Incoming JMS message |

### Web Service Consumer (SOAP)

- **WSDL fetch timing:** DESIGN TIME — bad WSDL URL causes immediate Studio error, not runtime
- **Before Consume:** Transform Message to build the SOAP XML body
- **After Consume:** Transform Message to parse the SOAP response

> ⚠ `Set Property`, `Set Attachment`, `Build SOAP`, `JSON to XML` = Mule 3. In Mule 4: always **Transform Message**.

> ✓ **Q56:** WSC:BAD_REQUEST → add Transform Message (SOAP payload) BEFORE the Consume operation.

### HTTP Request Connector

- Config (global element): protocol, host, port, base path
- Per-request: path, method, query params, headers, body
- Response: payload = response body; attributes = response status, headers

### DataWeave — Writing to Files (CSV example)

```dataweave
%dw 2.0
output application/csv
---
payload map { transaction_id: $.id, name: $.name, write_date: now() }
```

- `output application/csv` produces CSV with headers from key names
- `now()` returns current timestamp as DateTime

---

## Section 7: Processing Records ⚑ (10%)

### For Each Scope

- **Input:** Must be an **Array** — a Map/Object produces ZERO iterations
- **Each iteration:** Current element becomes payload; original payload replaced
- **After For Each:** Payload restored to original; variables set inside ARE accessible outside
- **Sequential:** Processes elements one at a time in array order

### For Each vs Batch — Key Difference

| Feature | For Each | Batch Job |
|---------|----------|-----------|
| Processing order | Sequential | **Parallel** |
| Input type | Array | Any collection (auto-split) |
| Variable scope | Vars accessible outside | Inner vars don't reach On Complete |
| Error handling | Stops entire scope on error | Record marked failed; job may continue |
| Use case | Small in-memory collections | Large datasets, files, DB results |

### Batch Job Structure

| Phase | Description |
|-------|-------------|
| Input (implicit) | Collection split into individual records |
| Batch Step(s) | Each step processes every record; steps run sequentially per record |
| On Complete | Runs ONCE after all records — receives a **SUMMARY REPORT** object, not the records |

### Critical Batch Rules ⚑

- **Record isolation:** Each record processed independently — variables do not accumulate across records
- **Variable scoping:** Variables modified INSIDE batch steps do NOT propagate back to the outer flow scope
- **On Complete payload:** `BatchJobResult` summary object — log it for stats, not for record data
- **Failed record default:** Record is marked failed and skipped in subsequent steps; other records continue
- **Max Failed Records = 0:** Default — zero failures allowed; if any record fails, the entire batch stops

> ⚠ **Q34 trap:** Logger in On Complete logs the SUMMARY REPORT, not the processed records.

> ⚠ **Q33:** Default behavior when a record fails = batch job stops processing all records (Max Failed Records defaults to 0 = zero tolerance).

### Batch — Sequential vs Parallel Execution

- **For Each:** Sequential — `[100ms, 200ms, 1000ms, 2000ms]` → logs in array order
- **Batch Step:** Parallel — same items complete in order of processing time (fastest first)

### Watermarking

- **Manual watermarking:** Retrieve last processed ID from Object Store → use in DB query `WHERE id > lastID` → store new max ID back to Object Store
- **Automatic watermarking:** Scheduler with watermark enabled + Object Store connector (auto-managed)
- **Object Store:** Persists values across flow executions (unlike variables which reset)
- **Variable:** Lives only for the current flow execution — NOT suitable for watermarking

> ⚠ **Q35:** Manual watermarking uses **Object Store**, NOT a variable.

> ⚠ **Q37:** To persist data across different flow executions → **Object Store** key-value pairs.

### Persisting Data Across Executions

| Storage type | Persists across executions? | Use case |
|-------------|---------------------------|---------|
| Variables (vars) | No — reset each execution | In-flow temporary data |
| Object Store | Yes | Watermarks, counters, session state |
| Database | Yes | Structured business data |
| File | Yes | Large data, audit logs |

---

## Section 8: Transforming Data with DataWeave ⚑ (15%)

### DataWeave Script Structure

```dataweave
%dw 2.0
output application/json
---
{ name: payload.firstName ++ " " ++ payload.lastName }
```

### Function Definition — fun keyword ⚑

```dataweave
fun newProdCode(itemID: Number, productCategory: String) =
  "PC-" ++ productCategory ++ (itemID as String)
```

- **Keyword:** `fun` (NOT `function`, NOT `var`)
- **Assignment:** `=` (NOT `->`)
- **Return:** expression after `=` (no `return` keyword)
- **Parameter types:** Capitalized — `Number`, `String`, `Date`, `Boolean`, `Array`, `Object`

> ⚠ **Q38:** `fun` keyword, `=` assignment. `function` and `->` are wrong.

### Importing DataWeave Modules ⚑

```dataweave
import modules::Utility
---
Utility::pascalize("max mule")
```

- **Path separator:** `::` (double colon) — NOT dot notation
- **Call syntax:** `ModuleName::functionName(arg)`
- **Import all:** `import * from modules::Utility` — then call `pascalize()` directly

> ⚠ **Q39:** `import modules::Utility` then `Utility::pascalize()`. NOT `import modules.Utility` (wrong separator).

### Essential Operators

| Operator / Function | Use / Notes |
|--------------------|------------|
| `++` | String/array concatenation — use instead of `+` |
| `as Type` | Type coercion — `as String`, `as Number`, `as Date` |
| `map` | Transform each element of Array → returns **Array** |
| `filter` | Keep elements matching condition → returns **Array** |
| `flatMap` | map then flatten one level |
| `reduce` | Accumulate Array to single value |
| `groupBy` | Group Array into **Object** by key |
| `orderBy` | Sort Array or Object by key |
| `pluck` | Transform Object to Array |
| `?` (safe nav) | Returns null if key missing (no error) |
| `contains` | Check if Array/String contains value |

### DataWeave Operation Chaining Order ⚑

```dataweave
payload filter $.price < 500 orderBy $.price groupBy $.toAirport
```

- **Order matters:** filter → orderBy → groupBy (not groupBy first)

> ⚠ **Q41:** filter FIRST, then orderBy, then groupBy. Wrong order = wrong result.

### map / filter / groupBy Return Types

- `map` returns: **Array** — always
- `filter` returns: **Array** — always
- `groupBy` returns: **Object** (keys = group values, values = arrays)
- `reduce` returns: Single accumulated value (any type)

### Type System ⚑

- **Coercion:** `as String`, `as Number`, `as Date`, `as Boolean`
- **Never:** `as :string` or lowercase — **capitalized types only**

### Custom Type Definition ⚑

```dataweave
type DateYearFirst = Date { format: "yyyy/MM/dd" }
```

- **Keyword:** `type` (no colon after it)
- **Assignment:** `=` (not `as`, not `:`)
- **Base type:** Capitalized (`Date` not `date`)
- **Constraints:** Curly braces `{ }` (not parentheses)

### Number Formatting

```dataweave
20.3844 as String { format: ".##" }   // "20.38"
```

- **Syntax:** `value as String { format: "pattern" }`
- **Curly braces** around the format constraint (not parentheses)

> ⚠ **Q43:** `20.3844 as String {format: ".0#"}` — curly braces not parentheses.

### XML Attribute Syntax ⚑

```dataweave
employee @(firstName: payload.first, lastName: payload.last): null
```

- **Attribute wrapper:** `@()` — NOT `()`
- **Attribute separator:** comma — NOT semicolon
- **Element value:** `null` (not empty string)

> ⚠ **Q40:** `@()` with commas between attributes, value is `null`. Semicolons and `()` are wrong.

### Choice Router When Expression

```dataweave
#[ 'MuleSoft' == payload.'company' ]
```

- **Comparison:** `==` (double equals)
- **String literals:** Single quotes `'...'` or double quotes `"..."`
- **Key access:** `payload.company` or `payload.'company'` (quoted if key has special chars)

> ⚠ **Q44:** NOT `#[company = "MuleSoft"]` (single `=` is assignment). NOT `#[if(...)]`.

### Output Formats

| MIME type | Result |
|-----------|--------|
| `application/json` | JSON string |
| `application/xml` | XML string |
| `application/java` | Java objects (List, Map) — default between processors |
| `application/csv` | CSV string |
| `text/plain` | Plain string |
| `application/dw` | DataWeave internal format — intermediate only |

### Calling a Flow from DataWeave

```dataweave
lookup("flowName", payload)
```

- Calls a private flow synchronously from DataWeave expression
- Returns the called flow's output payload
- Called flow must have no source (private flow)

---

## Section 9: Routing Events ⚑ (10%)

### Choice Router

- Evaluates conditions **TOP TO BOTTOM** — first match wins (like firewall rules)
- Each condition is a DataWeave boolean: `#[vars.airline == "american"]`
- **Default branch:** executes when NO condition matches
- Only ONE branch executes per message

> ✓ APIKit validates enum params before Choice router — invalid enum → `APIKIT:BAD_REQUEST`, never reaches Choice.

### Scatter-Gather ⚑

- **Routing:** Sends the **ENTIRE event** to ALL routes simultaneously (in **parallel** — not split)
- **Output type:** Object (LinkedHashMap) — keys `"0"`, `"1"`, `"2"`... one per route
- **Output values:** Each value is a full Mule message object (payload + attributes)
- **After completion:** `attributes = null`; `payload` = the assembled Object
- **All routes run:** Scatter-Gather waits for ALL routes before continuing

> ⚠ **CRITICAL:** Scatter-Gather output is an **OBJECT**, not an Array. Exam answer = "object containing three Mule event objects".

> ⚠ **Q48:** The ENTIRE event is sent to each route and processed in parallel — not split.

### Scatter-Gather with Partial Failure (Try Scope Pattern)

```
Scatter-Gather
  Route 1: [Try] → Flow Ref getAmericanFlights
             [Error handling: On Error Continue → Transform []]
  Route 2: [Try] → Flow Ref getUnitedFlights  (same pattern)
  Route 3: [Try] → Flow Ref getDeltaFlights   (same pattern)
```

- Each route independently protected — one failure doesn't stop others
- Failed route returns `[]` — Scatter-Gather assembles partial results (graceful degradation)

### First Successful

- Tries routes in sequence — stops on first successful route
- If a route throws, moves to next route

### Round Robin

- Each new message routed to next route in rotation
- Useful for load distribution across identical processors

### Validate Component ⚑

- Evaluates a DataWeave boolean condition
- If TRUE → passes through unchanged
- If FALSE → **throws an error** with configurable error type (e.g. `APP:INVALID_DESTINATION`)
- Does NOT return a value — it's a gate, not a transformation

> ✓ Validate is a gateway. Success = flow continues. Failure = error thrown (caught by error handler).

---

## Section 10: Handling Errors ⚑ (10%)

### Error Resolution Hierarchy

| Level | Scope | Applied when? |
|-------|-------|--------------|
| 1st: Try scope handler | Inside Try scope | Error occurs within that Try scope |
| 2nd: Flow error handler | Flow level | Error not caught by any Try scope |
| 3rd: Global error handler | Application wide | Flow has **NO error handler defined at all** |

> ⚠ If a flow has its own error handler and none of its scopes match the error — the **global handler is BYPASSED**. Error propagates unhandled to the HTTP Listener.

- **Global handler location:** Must be specified as a **global element** — NOT in pom.xml, NOT in properties file.

### On Error Continue vs On Error Propagate ⚑

| Behavior | On Error Continue | On Error Propagate |
|----------|------------------|-------------------|
| Flow completes normally? | Yes | No — re-throws error after handler |
| Processors after error point run? | No — main flow skipped | No — error re-thrown |
| Calling flow sees error? | No — receives handler's payload | Yes — error propagates up |
| HTTP Listener returns: | **Handler's payload** | **Original error message** |

> ⚠ On Error Continue does NOT resume after the failing processor. It handles in the error handler scope and exits the flow normally.

> ⚠ **Q51:** On Error Propagate → HTTP Listener returns the **ORIGINAL error message** ("Validate - Payload is an Integer"), not any payload set in the handler.

### Private Flow Error Isolation ⚑

- Private flow has its own On Error Continue that catches the error:
  - → Calling flow **NEVER sees the error**
  - → Calling flow receives the handler's payload as the Flow Reference result
  - → Calling flow continues normally

> ✓ **Q50:** Private flow catches own error → main flow returns "Success - main flow". The error is invisible to the caller.

### Try Scope Error Handling

- Try scope wraps processors to create a local error handling zone
- On Error Continue in Try scope → flow continues **AFTER** the Try scope
- On Error Propagate in Try scope → error re-thrown to flow-level handler

> ✓ **Q52:** Try scope catches error with On Error Continue → flow continues after Try scope with handler's payload.

### Error Type Structure

```
error.errorType.namespace    // "HTTP", "DB", "APP", "APIKIT"
error.errorType.identifier  // "NOT_FOUND", "CONNECTIVITY", "BAD_REQUEST"
error.description           // Human-readable message string
error.cause                 // Underlying Java exception
```

### Catching Errors by Namespace ⚑

```
Type: HTTP:NOT_FOUND        // catches specific type
Type: ANY                   // catches everything
Type: (empty)  When: #[error.errorType.namespace == "HTTP"]  // catches all HTTP:*
```

> ⚠ `HTTP:*` wildcard does **NOT** work in the Type field. Use the **When field** with DataWeave for namespace matching.

### Handler Ordering — First Match Wins

- Multiple handlers in one error handling block: evaluated top to bottom
- First matching handler fires — remaining handlers ignored
- Order: **specific error types FIRST, ANY last**

### APIKit Error Handlers (Auto-generated)

| Error Type | HTTP Status |
|-----------|------------|
| APIKIT:NOT_FOUND | 404 |
| APIKIT:BAD_REQUEST | 400 |
| APIKIT:METHOD_NOT_ALLOWED | 405 |
| APIKIT:NOT_ACCEPTABLE | 406 |
| APIKIT:UNSUPPORTED_MEDIA_TYPE | 415 |
| APIKIT:NOT_IMPLEMENTED | 501 |

---

## Section 11: Debugging and Troubleshooting (5%)

### Anypoint Studio Debugger

- **Breakpoint:** Pauses **BEFORE** the breakpoint processor runs — not after
- **F6 (Next processor):** Execute current processor, pause at next
- **F8 (Resume):** Run to next breakpoint or completion
- **Hot breakpoints:** Add/remove without restarting the app
- **Variables panel:** Shows payload, attributes, vars, error at pause point

### Breakpoint Payload — What You See

| Breakpoint location | Payload shown |
|--------------------|--------------|
| On a processor | State BEFORE that processor runs |
| Inside For Each | Current iteration element (e.g. first CSV row value: 100) |
| After Scatter-Gather | LinkedHashMap with keys "0","1","2" |
| Inside error handler | error object populated; payload = error payload |

> ✓ **Q55:** Breakpoint inside For Each on DB operation → payload = current CSV record value (e.g. 100 for first row).

### Maven Dependency Issues

- **Dependency not in MuleSoft repo:** Install to the computer's **local Maven repository** (`~/.m2`)
- **NOT:** Deploy to MuleSoft Maven repository
- **NOT:** Edit pom.xml (dependency is correct — it just needs to be installed)
- **NOT:** Add to `MULE_HOME/bin`

> ✓ **Q54:** Missing Maven dependency → install to local Maven repo.

### MUnit Test Structure

```
MUnit Test Suite
  └── MUnit Test
        ├── Behavior (Given) — Mock When, Set Event
        ├── Execution (When) — Flow Reference to flow under test
        └── Validation (Then) — Assert That, Verify Call
```

### MUnit Processors

- **Mock When:** Intercepts a processor and returns configured response — bypasses real call
- **Assert That:** Validates a condition on the result — fails test if false
- **Verify Call:** Confirms a mocked processor was called (optionally N times)
- **Set Event:** Sets up test input payload, attributes, variables

### WSC Bad Request Fix

- **Error:** `WSC:BAD_REQUEST` from Web Service Consumer
- **Cause:** Missing or malformed SOAP body before Consume operation
- **Fix:** Add **Transform Message BEFORE** the Consume — build the SOAP XML payload

---

## Section 12: Deploying and Managing APIs and Integrations (10%)

### Deployment Options

| Target | How to deploy |
|--------|--------------|
| CloudHub 2.0 | Runtime Manager UI, Maven plugin, or CI/CD pipeline |
| On-premises server | Runtime Manager agent installed on server, deploy via UI |
| Anypoint Studio | Run/Debug locally for development only |

### CloudHub 2.0

- **Workers:** vCore instances running your app — more workers = more parallel capacity
- **Worker sizes:** 0.1, 0.2, 1, 2, 4 vCores
- **Shared Space:** Multi-tenant CloudHub infrastructure
- **Private Space:** Dedicated isolated infrastructure for compliance/security
- **Region:** Geographic deployment location (US East, EU, APAC, etc.)

### ${http.port} on CloudHub ⚑

- **Purpose:** CloudHub automatically maps the internal HTTP port to allow external clients to connect
- **Use it:** Set HTTP Listener port to `${http.port}` for CloudHub compatibility
- **NOT:** VPN access, MuleSoft support access, or API Manager registration

> ✓ **Q59:** `${http.port}` enables CloudHub to automatically change the HTTP port for external client connectivity.

### Application Properties

- Define in Runtime Manager as application properties to override `config.yaml` values
- Properties are environment-specific — same key, different value per environment
- Sensitive properties can be flagged as secure (masked in UI)
- Reference in flows: `${propertyName}`

> ⚠ **Never hardcode credentials in flow XML.** Use `${propertyName}` references in global elements.

### API Manager — Autodiscovery ⚑

- **Purpose:** Register a deployed Mule app with API Manager to apply policies
- **No extra vCores needed:** Autodiscovery lets the app itself connect to API Manager
- **vs API Proxy:** API proxy = separate app (uses extra vCores). Autodiscovery = same app registers itself.

> ✓ **Q57:** IT won't allocate extra vCores → use **Autodiscovery** to register the existing app with API Manager.

### Exporting for CloudHub Deployment

- Export from Anypoint Studio → Anypoint Studio Archive (`.jar`)
- Include project dependencies (connector JARs) in the export
- Do NOT attach project sources (not needed for deployment, increases size unnecessarily)

> ✓ **Q58:** Smallest deployable archive = with dependencies, without attaching project sources.

### SLA-Based Policy Enforcement

- After enabling SLA policy in API Manager: **Add required client_id and client_secret headers to RAML spec → redeploy the API proxy**
- NOT: Restart the proxy (policy caches don't need clearing), add property placeholders, or add environment variables

> ✓ **Q60:** SLA policy enforcement requires RAML changes + proxy redeployment.

### API Promotion Across Environments

- APIs move Dev → Test → Prod via API Manager
- Each environment has its own API instance with own policies and SLA tiers
- Promote: update the API instance — don't rebuild from scratch

---

## QUICK REFERENCE: Common Exam Traps & Answer Keys

| If you see / are asked... | The answer is... |
|--------------------------|-----------------|
| `inboundProperties` / `outboundProperties` / `flowVars` | **WRONG** — Mule 3 only. Eliminate immediately. |
| `Set Property` / `Set Attachment` / `Build SOAP` | **WRONG** — Mule 3 only. Use Transform Message. |
| `attributes.'http.uri.params'.state` | **WRONG** — Mule 3. Use `attributes.uriParams.state` |
| Where do config property files go? | Global element (not pom.xml, not mule-artifact.json) |
| Property placeholder syntax | `${training.host}` with dot — NOT `#[training.host]` |
| Scatter-Gather output type | Object with keys "0","1","2" — **NOT Array** |
| What does Scatter-Gather send to each route? | The **ENTIRE event** — in parallel |
| DB Select returns... | Java List of Maps — NOT JSON. Transform Message needed. |
| DB Select with no matching rows | Empty array `[]` — NOT null, NOT false, NOT exception |
| File List returns... | Array of Mule message objects — NOT strings or filenames |
| After File Read, what happens to source file? | Renamed to `.bak` — content unchanged |
| On Complete payload in Batch Job | BatchJobResult **summary report** — NOT the processed records |
| Batch Max Failed Records = 0 (default) | Zero failures allowed — batch stops on first failure |
| For Each with a Map/Object input | **Zero iterations** — input MUST be an Array |
| For Each processing order | Sequential. Batch Steps = parallel. |
| Persist data across flow executions | **Object Store** — NOT variables (variables reset each run) |
| DataWeave function keyword | `fun` (NOT `function`, NOT `var`) |
| DataWeave function assignment | `=` (NOT `->`) |
| DataWeave module import path separator | `::` double colon (NOT dot): `import modules::Utility` |
| DataWeave module function call | `Utility::pascalize()` (NOT `Utility.pascalize()`) |
| DataWeave string concatenation | `++` (double plus) — NOT single `+` |
| DataWeave type coercion | `as String` (capitalized) — NOT `as :string` |
| DataWeave XML attribute wrapper | `@()` with **commas** — NOT `()` and NOT semicolons |
| DataWeave filter→orderBy→groupBy order | filter FIRST, then orderBy, then groupBy |
| DataWeave `map` returns | **Array** — always |
| Number formatting syntax | `20.3844 as String {format: ".0#"}` — curly braces |
| On Error Continue at flow level | Handler's **payload** returned to HTTP Listener |
| On Error Propagate at flow level | **Original error message** returned to HTTP Listener |
| Private flow with own error handler, calling flow gets: | Handler's payload — calling flow continues normally |
| Global error handler applies when? | ONLY when flow has **NO error handler** of its own |
| Catch all HTTP namespace errors | `Type: (empty)  When: #[error.errorType.namespace == "HTTP"]` |
| `HTTP:*` in Type field | **WRONG** — namespace wildcard not supported in Type field |
| Multiple error handlers — which fires? | **First match wins** (firewall rule order) |
| Variables across HTTP Request to external server? | **NO** — variables never cross network boundaries |
| Variables accessible in calling flow AFTER HTTP Request? | **YES** — vars survive in the calling flow |
| Variables across Flow Reference? | **YES** — both directions, same Mule event |
| Target attribute effect on payload | Original payload **PRESERVED** — result goes to variable |
| Sub-flow error handling | Sub-flows share calling flow's error handler (no own handler) |
| APIKit generates how many flows? | One per **HTTP method + resource combination** |
| REST Connect connector — first step | **Publish API spec to Anypoint Exchange** |
| WSDL fetched at what time? | **Design time** — bad URL = immediate Studio error |
| Fix WSC:BAD_REQUEST | Add Transform Message (SOAP payload) **BEFORE** Consume |
| Center for Enablement vs Center of Excellence | MuleSoft = **C4E (Center for Enablement)** |
| `!include` path syntax | Full path from project root: `!include examples/File.raml` |
| REST URI with resource ID | Path param: `/customers/1234` — NOT query param |
| PUT vs PATCH | **PUT = full replacement. PATCH = partial update.** |
| JMS Publish payload after operation | **Unchanged** (fire-and-forget) |
| JMS Publish-Consume payload after operation | **Reply message payload** |
| `${http.port}` on CloudHub | CloudHub auto-maps port for external client access |
| API Manager without extra vCores | Use **Autodiscovery** in the existing Mule app |
| SLA policy enforcement | Add headers to RAML spec + redeploy API proxy |
| Smallest deployable archive | With dependencies, without attached project sources |
| Missing Maven dependency fix | Install to **local Maven repository** (`~/.m2`) |
