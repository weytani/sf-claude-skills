---
name: Salesforce Apex Platform Reference
description: Dense quick-reference for Apex governor limits, trigger patterns, async processing, DML operations, security enforcement, common errors, and idioms. Table-heavy, optimized for fast lookup during development.
when_to_use: When writing, reviewing, or debugging Salesforce Apex code — triggers, batch jobs, queueable classes, DML operations, SOQL queries, or anything touching governor limits and platform constraints.
version: "62.0"
---

# Apex Platform Reference (API v62.0)

---

## 1. Governor Limits Quick-Reference

| Limit Name | Synchronous | Asynchronous |
|---|---|---|
| Total SOQL queries | 100 | 200 |
| Total SOQL rows retrieved | 50,000 | 50,000 |
| Total SOSL queries | 20 | 20 |
| Total DML statements | 150 | 150 |
| Total DML rows processed | 10,000 | 10,000 |
| Total callouts (HTTP/Web Service) | 100 | 100 |
| Callout timeout (single) | 120s | 120s |
| Total heap size | 6 MB | 12 MB |
| CPU time | 10,000 ms | 60,000 ms |
| Total email invocations (single send) | 10 | 10 |
| Total `@future` calls | 50 | 0 (cannot call from async) |
| Total queueable jobs enqueued | 50 | 1 (per execute) |
| Total push notifications | 10 | 10 |
| Maximum trigger depth | 16 | 16 |
| Total describe calls | 100 | 100 |
| Maximum SOQL query length | 100,000 chars | 100,000 chars |
| Total scheduled Apex jobs | 100 (org-wide) | — |

Check at runtime: `Limits.getQueries()`, `Limits.getLimitQueries()`, etc.

---

## 2. Trigger Context Variables

| Variable | Type | Description |
|---|---|---|
| `Trigger.new` | `List<sObject>` | New versions of records. Writable only in `before` triggers. |
| `Trigger.old` | `List<sObject>` | Old versions of records (previous values before update/delete). |
| `Trigger.newMap` | `Map<Id, sObject>` | Map of IDs to new record versions. `null` in `before insert` (no IDs yet). |
| `Trigger.oldMap` | `Map<Id, sObject>` | Map of IDs to old record versions. |
| `Trigger.operationType` | `System.TriggerOperation` | Enum: `BEFORE_INSERT`, `AFTER_INSERT`, `BEFORE_UPDATE`, `AFTER_UPDATE`, `BEFORE_DELETE`, `AFTER_DELETE`, `AFTER_UNDELETE`. |
| `Trigger.size` | `Integer` | Total records in both `Trigger.new` and `Trigger.old`. |
| `Trigger.isExecuting` | `Boolean` | `true` if current context is a trigger. |
| `Trigger.isBefore` | `Boolean` | `true` if before trigger. |
| `Trigger.isAfter` | `Boolean` | `true` if after trigger. |
| `Trigger.isInsert` | `Boolean` | `true` if insert operation. |
| `Trigger.isUpdate` | `Boolean` | `true` if update operation. |
| `Trigger.isDelete` | `Boolean` | `true` if delete operation. |
| `Trigger.isUndelete` | `Boolean` | `true` if undelete operation. |

---

## 3. Context Availability by Operation

| Context | `new` | `old` | `newMap` | `oldMap` |
|---|---|---|---|---|
| **before insert** | Y | X | X | X |
| **after insert** | Y (read-only) | X | Y | X |
| **before update** | Y | Y | Y | Y |
| **after update** | Y (read-only) | Y | Y | Y |
| **before delete** | X | Y | X | Y |
| **after delete** | X | Y | X | Y |
| **after undelete** | Y (read-only) | X | Y | X |

Key: **Y** = available, **X** = null/unavailable. `Trigger.new` is writable only in `before` contexts.

---

## 4. One-Trigger-Per-Object Pattern

One trigger per object, delegates all logic to a handler class. Never put business logic in the trigger body.

```apex
// Trigger
trigger AccountTrigger on Account (before insert, before update, after insert, after update, before delete, after delete, after undelete) {
    AccountTriggerHandler.dispatch(Trigger.operationType);
}
```

```apex
// Handler
public class AccountTriggerHandler {
    public static void dispatch(System.TriggerOperation op) {
        switch on op {
            when BEFORE_INSERT  { beforeInsert(Trigger.new); }
            when BEFORE_UPDATE  { beforeUpdate(Trigger.new, Trigger.oldMap); }
            when AFTER_INSERT   { afterInsert(Trigger.new); }
            when AFTER_UPDATE   { afterUpdate(Trigger.new, Trigger.oldMap); }
            when BEFORE_DELETE  { beforeDelete(Trigger.old); }
            when AFTER_DELETE   { afterDelete(Trigger.old); }
            when AFTER_UNDELETE { afterUndelete(Trigger.new); }
        }
    }

    private static void beforeInsert(List<Account> newRecords) {
        // field defaults, validation, stamping
    }

    private static void beforeUpdate(List<Account> newRecords, Map<Id, Account> oldMap) {
        // field changes, conditional logic
    }

    private static void afterInsert(List<Account> newRecords) {
        // related record creation, async calls
    }

    private static void afterUpdate(List<Account> newRecords, Map<Id, Account> oldMap) {
        // cross-object updates, platform events
    }

    private static void beforeDelete(List<Account> oldRecords) {
        // validation, prevent deletion
    }

    private static void afterDelete(List<Account> oldRecords) {
        // cleanup
    }

    private static void afterUndelete(List<Account> newRecords) {
        // restore related data
    }
}
```

---

## 5. Recursion Guard

Static Boolean pattern prevents re-entrant trigger execution.

```apex
public class TriggerGuard {
    private static Set<String> completedOperations = new Set<String>();

    public static Boolean hasRun(String operationKey) {
        return completedOperations.contains(operationKey);
    }

    public static void setRun(String operationKey) {
        completedOperations.add(operationKey);
    }

    public static void reset(String operationKey) {
        completedOperations.remove(operationKey);
    }
}
```

Usage in handler:

```apex
public static void afterUpdate(List<Account> newRecords, Map<Id, Account> oldMap) {
    if (TriggerGuard.hasRun('Account_afterUpdate')) return;
    TriggerGuard.setRun('Account_afterUpdate');

    // logic that might cause re-entry
    update relatedContacts;
}
```

Simple single-Boolean version (less flexible):

```apex
public class AccountTriggerGuard {
    public static Boolean hasRunAfterUpdate = false;
}
```

---

## 6. Order of Execution (Save Process)

The 16-step sequence when a record is saved:

1. Load original record from DB (or initialize for new).
2. Load new field values from request; overwrite old values.
3. Execute **before** triggers.
4. Run system validations (required fields, field formats, max length).
5. Execute custom validation rules.
6. Execute duplicate rules.
7. Save record to DB (not yet committed).
8. Execute assignment rules.
9. Execute auto-response rules.
10. Execute workflow rules; if field updates fire, re-run before/after update triggers **once more**.
11. Execute escalation rules.
12. Execute **after** triggers.
13. Execute entitlement rules.
14. Execute record-triggered flows (after-save).
15. Execute DML from trigger/flow; if child records saved, repeat from step 1 for those.
16. Commit all changes to DB. Post-commit logic: send email, enqueue async, fire platform events.

Reference `_PATTERNS.md` for expanded detail on each step and edge cases.

---

## 7. Async Apex Decision Tree

| Type | Use When | Limits | Chainable | Stateful | Code Example |
|---|---|---|---|---|---|
| **`@future`** | Simple async callout or DML; fire-and-forget. No job ID tracking needed. | 50 per txn (sync only). No `@future` from `@future`. Primitives only (no sObjects). | No | No | See below |
| **Queueable** | Need job ID, complex types, or chaining. Preferred over `@future` for new code. | 50 per txn (sync); 1 per `execute` (async). | Yes (1 deep in async, unlimited in sync test context) | Yes (instance vars persist) | See below |
| **Batch** | Process large data volumes (millions of rows). Needs `start`/`execute`/`finish`. | 5 concurrent batch jobs. 50M records in `QueryLocator`. Each `execute` gets fresh limits. | Yes (`Database.executeBatch` in `finish`) | Yes (with `Database.Stateful`) | See below |
| **Schedulable** | Cron-based recurring jobs. Often wraps Batch. | 100 scheduled jobs per org. | N/A (launches other async) | N/A | See below |

### @future

```apex
public class AccountService {
    @future(callout=true)
    public static void syncToExternal(Set<Id> accountIds) {
        List<Account> accounts = [SELECT Id, Name FROM Account WHERE Id IN :accountIds];
        // HTTP callout logic
    }
}
```

### Queueable

```apex
public class AccountQueueable implements Queueable, Database.AllowsCallouts {
    private List<Id> accountIds;

    public AccountQueueable(List<Id> accountIds) {
        this.accountIds = accountIds;
    }

    public void execute(QueueableContext ctx) {
        List<Account> accounts = [SELECT Id, Name FROM Account WHERE Id IN :accountIds];
        // processing logic
        // chain: System.enqueueJob(new NextQueueable(data));
    }
}
// Enqueue: Id jobId = System.enqueueJob(new AccountQueueable(ids));
```

### Batch

```apex
public class AccountBatch implements Database.Batchable<sObject>, Database.Stateful {
    public Integer recordsProcessed = 0;

    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator('SELECT Id, Name FROM Account WHERE IsActive__c = true');
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        // process scope (default 200 records)
        recordsProcessed += scope.size();
    }

    public void finish(Database.BatchableContext bc) {
        System.debug('Processed ' + recordsProcessed + ' records');
    }
}
// Execute: Id batchId = Database.executeBatch(new AccountBatch(), 200);
```

### Schedulable

```apex
public class AccountSchedulable implements Schedulable {
    public void execute(SchedulableContext ctx) {
        Database.executeBatch(new AccountBatch(), 200);
    }
}
// Schedule: System.schedule('Nightly Account Sync', '0 0 2 * * ?', new AccountSchedulable());
// Cron: Seconds Minutes Hours Day Month DayOfWeek OptionalYear
```

---

## 8. DML Operations

| Operation | DML Statement | Database Method | Partial Success | Returns |
|---|---|---|---|---|
| Insert | `insert records;` | `Database.insert(records, allOrNone)` | Yes (when `allOrNone=false`) | `Database.SaveResult[]` |
| Update | `update records;` | `Database.update(records, allOrNone)` | Yes | `Database.SaveResult[]` |
| Upsert | `upsert records Field__c;` | `Database.upsert(records, field, allOrNone)` | Yes | `Database.UpsertResult[]` |
| Delete | `delete records;` | `Database.delete(records, allOrNone)` | Yes | `Database.DeleteResult[]` |
| Undelete | `undelete records;` | `Database.undelete(records, allOrNone)` | Yes | `Database.UndeleteResult[]` |
| Merge | `merge master duplicate;` | `Database.merge(master, duplicates, allOrNone)` | Yes | `Database.MergeResult[]` |

- DML statements throw `DmlException` on failure (all-or-none).
- `Database.*` methods with `allOrNone=false` allow partial success; iterate results to check `isSuccess()` and `getErrors()`.
- `Database.insert(records, false)` = partial; `Database.insert(records, true)` = all-or-none (default).

---

## 9. Sharing Keywords

| Keyword | Behavior | Use When |
|---|---|---|
| `with sharing` | Enforces current user's sharing rules (record-level access). SOQL returns only records user can see. | Default for most classes. Required for user-facing controllers. |
| `without sharing` | Ignores sharing rules. Full data access regardless of user. | System-level operations: rollups, data migration, admin tools. Use sparingly. |
| `inherited sharing` | Inherits sharing context from caller. If no caller context, defaults to `with sharing`. | Utility/service classes that are called from multiple contexts. |
| *(no keyword)* | Legacy behavior: inherits from caller. If top-level, runs `without sharing`. **Avoid.** | Never use intentionally. Always declare explicitly. |

Sharing keywords affect record visibility (OWD, sharing rules, role hierarchy). They do **not** affect CRUD/FLS — those require separate enforcement.

---

## 10. CRUD/FLS Enforcement

| Method | Throws on Violation | Strips Silently | Works in SOQL | Notes |
|---|---|---|---|---|
| `WITH SECURITY_ENFORCED` | Yes (`System.QueryException`) | No | Yes (SOQL clause) | Simplest for queries. Fails entire query if any field is inaccessible. |
| `WITH USER_MODE` | No | Yes (strips inaccessible fields/rows) | Yes (SOQL clause) | API v60.0+. Enforces CRUD, FLS, and sharing in one keyword. Preferred. |
| `Security.stripInaccessible()` | No | Yes | No (post-query) | Call after query. Returns `SObjectAccessDecision`. Use `.getRecords()` for cleaned data. |
| `Schema.sObjectType.Account.fields.Name.getDescribe().isAccessible()` | No (manual check) | No | No | Granular per-field. Verbose but precise. Use for conditional logic. |
| `Schema.sObjectType.Account.getDescribe().isCreateable()` | No (manual check) | No | No | Object-level CRUD check. |

```apex
// WITH USER_MODE (preferred, v60.0+)
List<Account> accts = [SELECT Id, Name, Phone FROM Account WITH USER_MODE];

// WITH SECURITY_ENFORCED
List<Account> accts = [SELECT Id, Name, Phone FROM Account WITH SECURITY_ENFORCED];

// stripInaccessible
List<Account> accts = [SELECT Id, Name, Phone, SSN__c FROM Account];
SObjectAccessDecision decision = Security.stripInaccessible(AccessType.READABLE, accts);
List<Account> sanitized = decision.getRecords();
// sanitized records have inaccessible fields removed

// Manual check
if (Schema.sObjectType.Account.fields.Name.getDescribe().isUpdateable()) {
    // safe to update Name
}
```

---

## 11. Common Errors

| Error | Cause | Fix |
|---|---|---|
| `Too many SOQL queries: 101` | SOQL inside a loop or recursive triggers. | Move queries outside loops. Bulkify: query once, process with maps. |
| `Too many DML statements: 151` | DML inside a loop. | Collect records into lists, single DML outside loop. |
| `MIXED_DML_OPERATION` | DML on setup object (User, Group) and non-setup object in same transaction. | Use `@future` or Queueable for the setup DML. Or split into separate transactions. |
| `System.LimitException: Apex CPU time limit exceeded` | Heavy computation or deeply nested loops. | Optimize algorithms. Move to async (Batch/Queueable). Reduce trigger re-entry. |
| `CANNOT_INSERT_UPDATE_ACTIVATE_ENTITY` | Unhandled exception in a trigger on a related object. | Check the full stack trace — error is in a *different* trigger. Fix that trigger. |
| `FIELD_CUSTOM_VALIDATION_EXCEPTION` | Record failed a validation rule. | Check validation rules on the object. Fix data or adjust rule. `addError()` in triggers also surfaces this way. |
| `ENTITY_IS_DELETED` | Operating on a record that was deleted in the same transaction. | Check for deletion before DML. Guard with `Trigger.oldMap` or query. |
| `System.NullPointerException` | Dereferencing a null variable. | Use safe navigation (`?.`), null checks, or `String.isBlank()`. |
| `Maximum trigger depth exceeded` | Trigger recursion > 16 levels deep. | Implement recursion guard (Section 5). Review trigger design for circular updates. |
| `Too many query rows: 50001` | Single transaction retrieved > 50,000 SOQL rows total. | Add `WHERE` filters. Use Batch Apex. Use `LIMIT`. |
| `System.CalloutException: Read timed out` | HTTP callout exceeded 120s timeout. | Increase efficiency of external API. Break into smaller calls. Move to async. |
| `System.AsyncException: Maximum stack depth has been reached` | Queueable chaining exceeded depth limit. | Implement depth counter. Use Batch for large data volumes instead. |

---

## 12. Collections and Null Safety

### Safe Navigation Operator (`?.`)

```apex
String city = account?.BillingAddress?.getCity();   // null if any part is null
String name = contact?.Account?.Name;               // chains safely
Integer len = someString?.length();                  // null if someString is null
```

### String Checks

```apex
// String.isBlank() — true for null, '', and whitespace-only
if (String.isBlank(input)) { /* handle empty */ }

// String.isEmpty() — true for null and '' only (NOT whitespace)
if (String.isEmpty(input)) { /* handle empty */ }

// String.isNotBlank() / String.isNotEmpty() — inverses
if (String.isNotBlank(input)) { /* safe to use */ }
```

### Map Safety

```apex
Map<Id, Account> accountMap = new Map<Id, Account>([SELECT Id, Name FROM Account WHERE Id IN :ids]);

// Always check containsKey before get to avoid null
if (accountMap.containsKey(someId)) {
    Account a = accountMap.get(someId);
}

// Or use safe navigation
String name = accountMap.get(someId)?.Name;

// getOrDefault pattern (manual in Apex)
Account a = accountMap.containsKey(someId) ? accountMap.get(someId) : new Account();
```

### List Initialization & Patterns

```apex
// Always initialize — uninitialized List is null, not empty
List<Account> accounts = new List<Account>();

// Null-safe iteration
for (Account a : accounts != null ? accounts : new List<Account>()) {
    // process
}

// Check before access
if (accounts != null && !accounts.isEmpty()) {
    Account first = accounts[0];
}
```

### Set Operations

```apex
Set<Id> setA = new Set<Id>{'001...', '001...'};
Set<Id> setB = new Set<Id>{'001...', '001...'};

setA.addAll(setB);      // union
setA.retainAll(setB);   // intersection
setA.removeAll(setB);   // difference
setA.contains(someId);  // membership check

// Deduplicate a list
List<Id> ids = new List<Id>(new Set<Id>(originalList));
```

### Bulkification Pattern (Map-Based Lookup)

```apex
// Build lookup map from query
Map<Id, List<Contact>> contactsByAccountId = new Map<Id, List<Contact>>();
for (Contact c : [SELECT Id, AccountId FROM Contact WHERE AccountId IN :accountIds]) {
    if (!contactsByAccountId.containsKey(c.AccountId)) {
        contactsByAccountId.put(c.AccountId, new List<Contact>());
    }
    contactsByAccountId.get(c.AccountId).add(c);
}

// Use map instead of query-per-record
for (Account a : Trigger.new) {
    List<Contact> contacts = contactsByAccountId.get(a.Id);
    if (contacts != null) {
        // process contacts for this account
    }
}
```
