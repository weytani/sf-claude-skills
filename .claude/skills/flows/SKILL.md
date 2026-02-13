---
name: Salesforce Flows
description: Platform reference for Salesforce Flow Builder — types, elements, timing, fault handling, and decision guidance for Flow vs Apex.
when_to_use: When building, debugging, or advising on Salesforce Flows. Use for element selection, trigger timing decisions, fault handling patterns, and determining whether logic belongs in Flow or Apex.
version: "62.0"
---

# Salesforce Flows — Platform Reference (API v62.0)

## Flow Types

| Type | Trigger | Runs As | Context | Use Case |
|------|---------|---------|---------|----------|
| Record-Triggered Flow | Record create/update/delete | System (no user interaction) | Fires on DML against a specific object | Field updates, cross-object automation, validation beyond formula limits |
| Screen Flow | User launches (button, page, quick action, Experience Cloud) | Running user | Interactive — collects input, displays output | Guided wizards, data entry forms, case deflection, self-service |
| Schedule-Triggered Flow | Time-based (daily/weekly or record-based schedule) | Default process automation user | Batch context, runs on a schedule against a set of records | Reminder emails, SLA escalations, periodic data cleanup |
| Auto-launched Flow | Invoked by another flow, Apex, REST API, or process | System or calling context | No UI — runs headlessly | Reusable logic modules, API-triggered automation, Apex-invoked orchestration |
| Platform Event-Triggered Flow | Platform event message published | System (event subscriber) | Asynchronous, separate transaction from publisher | Event-driven integrations, decoupled processing, near-real-time reactions |

## Record-Triggered Timing

### Decision Tree

```
Record DML fires
  ├─ Before Save ──→ Fast field updates on SAME record only (no DML)
  ├─ After Save (Run Immediately) ──→ DML allowed, same transaction, cross-object updates
  └─ After Save (Run Asynchronously) ──→ Separate transaction, callouts OK, eventual consistency
```

### Timing Comparison

| Timing | DML Allowed | Same Transaction | Can Update Other Records | Callouts | Use Case |
|--------|:-----------:|:----------------:|:------------------------:|:--------:|----------|
| Before Save | No | Yes | No (same record only) | No | Default field values, field derivation, lightweight validation |
| After Save (Run Immediately) | Yes | Yes (same transaction, post-commit triggers) | Yes | No | Create child records, update related records, send emails |
| After Save (Run Asynchronously) | Yes | No (separate transaction) | Yes | Yes | HTTP callouts, external system sync, heavy processing |

**Key rule:** Start with Before Save. Escalate to After Save only when you need DML on other records. Escalate to Async only when you need callouts or want isolation from the triggering transaction.

## Flow Elements

| Element | Category | Purpose | Notes |
|---------|----------|---------|-------|
| Assignment | Logic | Set variable values | Supports add, subtract, equals, add to collection |
| Decision | Logic | Branch execution path | Equivalent to if/else; evaluated top-down, first match wins |
| Loop | Logic | Iterate over a collection | Avoid DML inside loops — assign to collection, then bulk DML after |
| Get Records | Data | SOQL query (retrieve records) | Returns first record or collection; filter conditions map to WHERE clause |
| Create Records | Data | Insert one or many records | Accepts single record variable or collection for bulk insert |
| Update Records | Data | Update records matching criteria or from a variable | Can filter directly or pass a record/collection variable |
| Delete Records | Data | Delete records matching criteria or from a variable | Use with caution; no recycle bin undo in all contexts |
| Subflow | Logic | Invoke another auto-launched or screen flow | Pass variables in/out; key reusability mechanism |
| Action | Integration | Invoke Apex, send email, post to Chatter, call external service | Apex actions = `@InvocableMethod`; Custom Notifications also available |
| Screen | UI | Display form to user, collect input | Screen Flows only; supports dynamic visibility, validation, reactive components |
| Custom Error | Logic | Throw a fault with a custom message | Before-save only; blocks the save and displays error to user |
| Wait | Logic | Pause flow until a condition or time | Requires a Paused interview; not available in before-save |
| Collection Sort | Data | Sort a record collection by field(s) | Ascending/descending; sort before display or processing |
| Collection Filter | Data | Filter a collection by conditions | Reduces collection without another Get Records (saves SOQL) |
| Transform | Data | Map fields between collections or reshape data | Useful for converting between object types or restructuring data for subflows |

## Variable Types

| Type | Collection Supported | Available as Input/Output |
|------|:--------------------:|:-------------------------:|
| Text | Yes | Yes |
| Number | Yes | Yes |
| Currency | Yes | Yes |
| Boolean | Yes | Yes |
| Date | Yes | Yes |
| DateTime | Yes | Yes |
| Record (single) | No (use Record Collection) | Yes |
| Record Collection | N/A (is a collection) | Yes |
| Picklist Choice | Yes | No (Screen Flow internal use) |
| Stage | No | No (Flow orchestration internal) |
| Apex-Defined | Yes | Yes |

## Fault Handling

### Fault Connectors

Every data element (Get, Create, Update, Delete) and action element can have a **fault connector** — a separate path that executes when the element throws a runtime exception. Add one by clicking the element and selecting "Add Fault Path" (or dragging from the fault node).

### System Variables for Faults

| Variable | Contents |
|----------|----------|
| `$Flow.FaultMessage` | The runtime error message (e.g., `FIELD_CUSTOM_VALIDATION_EXCEPTION: ...`) |
| `$Flow.InterviewGuid` | Unique identifier for the flow interview instance — use for log correlation |

### Recommended Fault Pattern

```
[Element with fault risk]
  ├─ Success path → continue
  └─ Fault path →
       1. Create a custom Error_Log__c record (or publish a Log_Event__e platform event)
          - Store: $Flow.FaultMessage, $Flow.InterviewGuid, record ID context, timestamp
       2. Optionally notify admin (email alert or custom notification)
       3. Terminate gracefully (do NOT leave the flow dangling)
```

**Best practice:** Add fault paths to every DML and callout element. Unhandled faults produce cryptic "Unhandled fault" emails and block the triggering transaction. Logging to a platform event is preferred over a custom object insert because platform event publishes are independent of the current transaction's rollback.

## Flow vs Apex Decision Guide

| Scenario | Flow | Apex |
|----------|------|------|
| Simple field updates | Preferred (Before Save) | Overkill |
| Complex branching logic | Workable but gets unwieldy past ~15 decisions | Preferred — cleaner control flow |
| Bulk operations (10k+ records) | Auto-bulkifies in record-triggered context | Preferred for fine-grained governor control |
| External callouts | After Save (Async) works for simple cases | Preferred — retry logic, error handling, named credentials |
| Admin maintainability | High — visual, low-code | Low — requires developer |
| Reusability | Subflows; Apex actions bridge to code | Apex classes, `@InvocableMethod` exposes to flows |
| Testing | Limited (Flow debug, test coverage via triggers) | Full Apex test framework, assertions, mocking |
| Performance at scale | Good for moderate volume; watch governor limits | Superior — explicit limit management, efficient SOQL |
| Transaction control | Implicit (before/after save boundaries) | Explicit — `Savepoint`, `Database.rollback()`, partial success via `Database.insert(records, false)` |

**Rule of thumb:** If an admin can own it and it stays under ~20 elements, use Flow. If it needs complex iteration, error recovery, or will be called at high volume, use Apex and expose it to Flow via `@InvocableMethod`.

## Best Practices

| Practice | Detail |
|----------|--------|
| **Filter early with entry conditions** | Set entry conditions on record-triggered flows to reduce unnecessary executions. Equivalent to an early `return` in a trigger — check `$Record.Status__c = 'Active'` before running any logic. |
| **Naming conventions** | Record-Triggered: `Object_Trigger_Purpose` (e.g., `Case_AfterCreate_AssignEntitlement`). Screen: `Object_Screen_Purpose` (e.g., `Lead_Screen_QualificationWizard`). Subflow: `Sub_Purpose` (e.g., `Sub_SendNotification`). |
| **Use subflows for reusability** | Extract shared logic into auto-launched subflows. Pass record IDs or collections as input variables. Keeps parent flows clean and DRY. |
| **Bulkification awareness** | Record-triggered flows auto-bulkify: one interview handles all records in the transaction. But each Get/Create/Update/Delete inside a Loop consumes a SOQL or DML call **per iteration** — move DML outside loops. |
| **Avoid recursive triggers** | Use `$Record__Prior` comparisons in entry conditions to prevent re-entry (e.g., only run when `Status__c` is changed, not on every update). For cross-object updates that re-trigger, check "When to Run the Flow for Updated Records" and select "Only if the record that triggered the flow to run is updated." |
| **One flow per object per timing** | Avoid multiple record-triggered flows on the same object + timing. Execution order is nondeterministic. Consolidate into one flow with Decision elements. |
| **Version control** | Activate a new version rather than editing the active one. Keep previous version active until the new one is tested. Use Flow description field for change notes. |

## Common Pitfalls

| Symptom | Cause | Fix |
|---------|-------|-----|
| Flow triggered recursively / `Maximum flow trigger recursion depth exceeded` | Flow updates the same object it triggers on, causing re-entry | Add entry conditions using `$Record__Prior` field comparisons; check "Only if the record that triggered the flow is updated" under re-evaluation settings |
| `UNABLE_TO_LOCK_ROW` | Multiple flows/processes competing for the same record in parallel transactions | Reduce cross-object updates in synchronous context; use async where possible; avoid parent record updates from child triggers in high-volume scenarios |
| Unhandled fault in flow / cryptic admin email | DML or callout element failed with no fault connector | Add fault paths to every DML and action element; log `$Flow.FaultMessage` to an error record or platform event |
| Flow interview expired / `Flow interview is no longer available` | Wait element or paused interview exceeded retention (default 30 days) or user re-submitted a stale screen | Reduce wait durations; handle screen flows with `retainOnFinish`; inform users of session limits |
| `Too many SOQL queries: 101` | Get Records inside a Loop element, executed per record in a bulk transaction | Move Get Records before the loop (query once, filter in-memory with Collection Filter), or restructure to query a collection |
| `Apex CPU time limit exceeded` in flow | Large collection loop with complex assignments, or calling expensive Apex actions per iteration | Simplify loop body; move heavy logic to bulkified Apex `@InvocableMethod` that accepts `List<>` input; reduce decision elements inside loops |
| Null reference / `The flow tried to access a value on a null record` | Get Records returned no results and subsequent elements reference fields on the null variable | Add a Decision element after Get Records to check if the variable `Is Null` equals `{!$GlobalConstant.True}` before accessing fields |
| Collection variable empty after Get Records | Filter conditions too restrictive, or querying the wrong object/field API name | Debug with Flow Debug tool; verify field API names (not labels); check filter logic (AND vs OR); ensure records actually exist matching criteria |
