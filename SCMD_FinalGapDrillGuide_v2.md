**SCMD / DEX-401**

**Final Gap Drill Guide  v2**

*Weak-spot focus only · Exam day: April 9, 2026*

*Updated with empirical Studio testing · April 8 evening*

**1 · The Mule Event Model  (Root Cause of Many Misses)**

The single most important mental model. Every trap on the exam flows from this.

| **Who Creates vs Who Mutates** |
| --- |
| **SOURCES — create a new Mule event** | HTTP Listener, JMS Listener, Scheduler, File Listener, DB Listener |
| **CONNECTORS — mutate the SAME event** | HTTP Request, DB Select, JMS Publish, File Read, Transform Message |
| Scatter-Gather | Fork-and-merge scope. Takes ONE event in, returns ONE event out. Internal parallel copies are an implementation detail. |
| Key rule | Inside a flow = ONE Mule event evolving. Scatter-Gather is a black box that collapses back to one event. |

| **What Survives a Network Boundary (HTTP Request)** |
| --- |
| payload | REPLACED by HTTP response body |
| attributes | REPLACED by HTTP response attributes |
| vars | PRESERVED — always, no exceptions |
| target attribute trick | Result → vars.targetName; original payload PRESERVED |

| **What Travels Across a Flow Reference** |
| --- |
| payload, attributes, vars | ALL travel — both directions. Same Mule event. |

**2 · Error Handling  (Most Heavily Tested Section)**

| **On Error Continue — Behavior Depends on Context** |
| --- |
| Top-level flow (HTTP Listener) | Handler fires, then returns handler's payload to client. Main flow does NOT continue past error point. |
| Private flow (called via Flow Ref) | Handler fires, then returns control to CALLING FLOW as if Flow Ref completed successfully. Calling flow CONTINUES from next processor. |
| Try scope | Handler fires, then SAME FLOW continues from processor AFTER the Try scope. |
| The trap | Q: Private flow catches error, calling flow has Set Payload 'Success' after Flow Ref. Answer = 'Success' — calling flow resumes. |

| **On Error Propagate** |
| --- |
| All contexts | Handler fires, original error re-thrown. HTTP Listener returns original error message. |

| **Error Matching Rules** |
| --- |
| Order | Top-down, first match wins (like firewall rules) |
| ANY | Catches everything — if placed first, blocks all specific handlers below it |
| No chaining | Only the first matching handler runs. Period. |

| **Global Error Handler Rule** |
| --- |
| Applies only when... | The flow has NO error handler at all |
| Bypassed when... | Flow has its own handler — even if no scope within it matches the error |

| **Namespace-Based Error Catching — EMPIRICALLY VERIFIED IN STUDIO** |
| --- |
| Correct pattern | Type field: EMPTY  │  When: #[error.errorType.namespace == "HTTP"] |
| Also works | Type field: ANY  │  When: #[error.errorType.namespace == "HTTP"] |
| WRONG | Type field: HTTP:*  (wildcard not supported in Type field) |
| Verified | Empty Type + When tested in Studio: caught HTTP:NOT_FOUND, did NOT catch APP:MYERROR. When clause is a genuine filter. |
| Exam answer | Empty Type field is the documented textbook pattern |

| **Error Mapping** |
| --- |
| What it does | Rewrites error TYPE before any handler evaluates it |
| Available on | Connector operations only (HTTP Request, DB, JMS, File, WSC etc.) — NOT on Transform Message, Set Variable, scopes, routers |
| Example | HTTP:NOT_FOUND → APP:API_RESOURCE_NOT_FOUND |
| Critical rule | Handlers see ONLY the mapped type, never the original |

**3 · Connector Payload Behavior  (Memorize This Table)**

| **What Each Connector Does to payload** |
| --- |
| **HTTP Request** | REPLACES payload AND attributes |
| **DB Select** | REPLACES payload → Java List of Maps (NOT JSON). Empty result = [] not null. |
| DB Select params | Map keys must match :named parameters in SQL — NOT column names, NOT arbitrary keys |
| **JMS Publish (one-way)** | payload UNCHANGED |
| **JMS Publish-Consume (request-reply)** | REPLACES payload with reply message |
| **File List** | REPLACES payload → Array of Mule message objects (NOT strings) |
| **Any connector with target=** | Payload PRESERVED; result → vars.targetName |
| **For Each (scope)** | Returns ORIGINAL payload after all iterations complete |
| **Scatter-Gather** | Returns Object with keys 0,1,2 — each a full Mule event. Access: payload[0].payload |

| **Scatter-Gather access syntax (empirically confirmed): **payload[0].payload accesses first route's data. payload[1].payload for second, etc. Output is Object not Array — but DataWeave numeric index access works. |
| --- |

**4 · Validation Component**

| **Validation Mental Model** |
| --- |
| How it works | ASSERTION — condition must be TRUE. If false → throws VALIDATION error. |
| message parameter | Describes the REQUIREMENT, not the failure. |
| is-null | Throws if payload IS NOT null (condition = is it null? false if payload exists) |
| is-not-null | Throws if payload IS null |

**5 · Batch Processing**

| **Critical Batch Rules** |
| --- |
| Max Failed Records = 0 (default) | ZERO TOLERANCE — batch stops on first failure |
| On Complete payload | BatchJobResult SUMMARY REPORT — NOT the processed records |
| Variable scope | Vars modified INSIDE batch steps do NOT propagate to outer flow |
| For Each input | Must be Array — Map/Object → ZERO iterations |
| Batch vs For Each | Batch = parallel. For Each = sequential. |

**6 · DataWeave Syntax**

| **DataWeave Rules to Drill** |
| --- |
| String concat | ++ (double plus). Single + is WRONG. |
| Type coercion | Capitalized, no colon: String, Number, DateTime — NOT :string |
| Custom type def | type Name = BaseType { constraint: value } |
| Function definition | fun myFun(arg) = expression |
| Module import | import modules::Utility |
| Module function call | Utility::myFunction(arg)  — NOT Utility.myFunction() |
| Call a flow from DataWeave | lookup("flowName", payload)  — NOT flowRef(), NOT flow::call() |
| Inline expression | 'The value is #[payload.x]'  — string with embedded DW pocket |
| Full DW expression | #["The value is " ++ payload.x]  — pure DataWeave, no surrounding quotes |
| XML attribute syntax | @(key: value, key2: value2) — commas NOT semicolons |
| vars access | vars.myVar — NOT variables.myVar, NOT flowVars.myVar |
| filter → orderBy → groupBy | Order matters: filter FIRST, then orderBy, then groupBy |

| **Inline vs Full DW: **Inline ('text #[expr]') used in string fields like Logger. Full DW (#[expr]) used in processor config fields expecting a computed value. Both are valid in appropriate contexts. |
| --- |

**7 · Structuring Mule Applications**

| **Global Elements vs Flow Config** |
| --- |
| Global elements contain | host, port, connector configs, Configuration Properties, TLS context |
| pom.xml | Maven dependencies ONLY |
| mule-artifact.json | App metadata ONLY |
| Config file reference | Global element: configuration-properties file="config.yaml" |
| Property syntax | ${training.host} with dot — NOT #[training.host] |

| **Flow Types** |
| --- |
| Flow | Has source, can have own error handler |
| Private flow | No source, CAN have own error handler |
| Sub-flow | No source, NO own error handler — ALWAYS inherits calling flow's handler |
| Both private flow and sub-flow | Invoked via Flow Reference — looks the same from outside |
| APIKit generates | ONE flow per HTTP method + resource combination |

**8 · Debugging ****&**** API Manager**

| **Debugger Behavior** |
| --- |
| Breakpoint shows state | BEFORE the processor executes (pre-execution) |
| Inside For Each | Shows current iteration element as payload |
| After Scatter-Gather | Shows LinkedHashMap with keys 0,1,2 |

| **API Manager — SLA Policies** |
| --- |
| After enabling SLA policy | Add client_id and client_secret headers to RAML spec + redeploy proxy |
| Exchange connector | Must PUBLISH API spec to Exchange first |

**9 · Rapid-Fire Exam Traps  (Eliminate on Sight)**

| **If You See...  →  The Answer Is...** |
| --- |
| inboundProperties / outboundProperties / flowVars | WRONG — Mule 3. Eliminate immediately. |
| attributes.'http.uri.params'.state | WRONG — Mule 3. Use attributes.uriParams.state |
| #[training.host] as property placeholder | WRONG — DataWeave syntax. Use ${training.host} |
| Scatter-Gather output is an Array | WRONG — Object with keys 0,1,2. Access: payload[0].payload |
| DB Select returns JSON | WRONG — Java List of Maps. Transform Message needed. |
| File List returns string filenames | WRONG — Array of Mule message objects |
| On Complete gets processed records | WRONG — BatchJobResult summary report |
| Max Failed Records = 0 means unlimited | WRONG — zero tolerance, stops on first failure |
| For Each with Map/Object input | Zero iterations — must be Array |
| Sub-flow with own error handler | Impossible — sub-flows always inherit calling flow's handler |
| Config file location → pom.xml | WRONG — global element (Configuration Properties) |
| + for DataWeave string concat | WRONG — use ++ |
| :string for type coercion | WRONG — use String (capitalized, no colon) |
| PUT = partial update | WRONG — PUT replaces entire resource. PATCH = partial. |
| Flow Reference creates new Mule event | WRONG — same event, all travels (payload, attrs, vars) |
| flowRef() / flow::call() in DataWeave | WRONG — use lookup("flowName", payload) |
| HTTP:* in Type field | WRONG — use empty Type + When: #[error.errorType.namespace == "HTTP"] |
| On Error Continue = flow resumes after error point | WRONG — short-circuits to handler, then exits/returns normally |

**10 · Tonight****'****s Instructions**

You're at ~85-90% readiness. The gap was execution semantics, not knowledge. That gap is closed.

**DO THIS tonight:**

- Read sections 1-9 of this guide once, slowly.

- On anything that surprises you: pause, say the rule out loud.

- Stop by 9 PM. Sleep matters more than one more pass.

**DO NOT:**

- Re-read the full reference doc.

- Learn any new concepts.

- Cram tomorrow morning — light confidence review only.

**You know this. Go pass it.  🎯**