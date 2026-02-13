---
name: Cross-Cutting Platform Patterns
description: Shared concepts that span multiple Salesforce technologies — order of execution, sharing model, security, governor philosophy
when_to_use: When debugging save-order issues, understanding sharing/security behavior, or deciding between platform features
version: "62.0"
---

# Cross-Cutting Platform Patterns

## Salesforce Order of Execution (Record Save)

The full sequence when a record is saved. Every Apex and Flow developer needs this memorized.

| Step | What Happens |
|------|-------------|
| 1 | Original record loaded from DB (or initialized for insert) |
| 2 | New field values overwrite old values (from request) |
| 3 | System validation rules (required fields, field formats) |
| 4 | **Before-save Flows** (record-triggered, fast field updates) |
| 5 | **Before triggers** execute |
| 6 | System validation again + custom validation rules |
| 7 | Duplicate rules execute |
| 8 | Record saved to DB (but not committed) |
| 9 | **After triggers** execute |
| 10 | Assignment rules, auto-response rules, workflow rules |
| 11 | Workflow field updates — if any fire, before/after triggers re-fire |
| 12 | **After-save Flows** (record-triggered, can do DML) |
| 13 | Entitlement rules, roll-up summary fields |
| 14 | Cross-object formula updates, repeat validation if needed |
| 15 | Criteria-based sharing evaluation |
| 16 | **DML committed to database** |

**Key implications:**
- Before-save Flows (step 4) run *before* before triggers (step 5)
- Validation rules run *twice* — once before triggers, once after
- Workflow field updates (step 11) can re-fire triggers, causing unexpected re-entry
- After-save Flows (step 12) run after triggers but before commit — they share the transaction
- Nothing is committed until step 16. Any unhandled exception rolls back everything

## Governor Limits Philosophy

Salesforce is multi-tenant. Governor limits exist to prevent any single transaction from monopolizing shared resources.

**Key principles:**
- Limits are **per transaction**, not per trigger or class
- A trigger that calls a helper that calls a service — all share the same limits
- Async contexts (Batch, Queueable, @future) get **fresh limits** per execution
- `Limits` class methods let you check consumption at runtime

**The multi-tenant mental model:** Your org shares compute, storage, and I/O with thousands of other orgs on the same pod. Limits are the contract that keeps everyone honest.

## Sharing Model Overview

Salesforce record access is **deny by default** — users see nothing unless access is explicitly granted.

### Access Grant Hierarchy (evaluated in order)

| Layer | Scope | Admin Control |
|-------|-------|---------------|
| **Org-Wide Defaults (OWD)** | Baseline for all records per object | Setup → Sharing Settings |
| **Role Hierarchy** | Users see records owned by subordinates | Setup → Roles |
| **Sharing Rules** | Criteria-based or owner-based bulk grants | Setup → Sharing Settings |
| **Manual Sharing** | Record-by-record grants by owner | Record detail → Sharing button |
| **Apex Managed Sharing** | Programmatic grants via `Share` objects | Code: `AccountShare`, etc. |
| **Implicit Sharing** | Parent-child auto-grants (Account→Contact) | Automatic, not configurable |
| **Territory Management** | Geographic/account-based access | Setup → Territory Management |

### OWD Settings

| Setting | Meaning |
|---------|---------|
| **Private** | Only owner + users above in role hierarchy |
| **Public Read Only** | All users can read, only owner can edit |
| **Public Read/Write** | All users can read and edit |
| **Controlled by Parent** | Access inherited from parent (e.g., Contact → Account) |
| **Public Full Access** | Read, edit, transfer, delete (Campaign only) |

### Apex Sharing Keywords

| Keyword | Behavior |
|---------|----------|
| `with sharing` | Enforces current user's sharing rules |
| `without sharing` | Ignores sharing rules (runs as system) |
| `inherited sharing` | Uses sharing context of the calling class |
| *(no keyword)* | Same as `without sharing` in most contexts — **always specify explicitly** |

**Rule of thumb:** Use `with sharing` by default. Use `without sharing` only when the business logic requires seeing all records (e.g., a rollup service). Use `inherited sharing` for utility classes that shouldn't make their own sharing decision.

## Security Model

### Profile vs Permission Set

| Aspect | Profile | Permission Set |
|--------|---------|---------------|
| Assignment | One per user (required) | Many per user (additive) |
| Direction | Moving toward minimal profiles | Preferred for all grants |
| Best practice | Use "Minimum Access" profile | Grant everything via perm sets |

### CRUD and FLS

| Level | What It Controls | Where Set |
|-------|-----------------|-----------|
| **Object CRUD** | Create, Read, Update, Delete per object | Profile or Permission Set |
| **Field-Level Security (FLS)** | Read/Edit per field per object | Profile or Permission Set |

**Enforcement in Apex:**

```apex
// Option 1: SOQL-level enforcement (Spring '20+)
List<Account> accts = [SELECT Name FROM Account WITH SECURITY_ENFORCED];

// Option 2: Strip inaccessible fields from results
SObjectAccessDecision decision = Security.stripInaccessible(
    AccessType.READABLE,
    [SELECT Name, AnnualRevenue FROM Account]
);
List<Account> safe = decision.getRecords();

// Option 3: Manual check (oldest approach)
if (Schema.sObjectType.Account.fields.Name.isAccessible()) {
    // safe to read
}
```

| Method | Throws on Violation | Strips Silently | Works in SOQL |
|--------|-------------------|-----------------|---------------|
| `WITH SECURITY_ENFORCED` | Yes (insufficient access) | No | Yes |
| `stripInaccessible()` | No | Yes | No (post-query) |
| `Schema.describe` | No (manual check) | No | No |

## Platform Events

Publish/subscribe messaging on the Salesforce platform.

### When to Use Platform Events

| Use Case | Platform Events | Other Options |
|----------|----------------|---------------|
| Cross-system integration | Yes | External services, MuleSoft |
| Decoupled Apex-to-Apex | Yes | Custom notifications, Queueable |
| Trigger-to-LWC real-time | Yes (via empApi) | Streaming API |
| Logging/audit trail | Yes | Custom objects, Big Objects |
| Error notification | Yes | Custom notifications, email |

### Key Characteristics

- **Fire-and-forget** — publisher doesn't wait for subscribers
- **Survive transaction rollback** — published events persist even if the publishing transaction fails (when using `EventBus.publish` after `setSavepoint/rollback`)
- **Replay** — subscribers can replay events from the last 72 hours (or 3 days)
- **Governor limits** — publishing counts against DML limits; subscribing via trigger counts against trigger limits
- **`AFTER_COMMIT` vs `AFTER_EACH`** — controls when the trigger fires relative to the batch

```apex
// Publishing
MyEvent__e evt = new MyEvent__e(Payload__c = 'data');
Database.SaveResult sr = EventBus.publish(evt);

// Subscribing (Apex trigger)
trigger MyEventTrigger on MyEvent__e (after insert) {
    for (MyEvent__e evt : Trigger.new) {
        // process event
    }
}
```

## Custom Metadata Types vs Custom Settings

| Aspect | Custom Metadata Types | Custom Settings (List) | Custom Settings (Hierarchy) |
|--------|----------------------|----------------------|---------------------------|
| Deployable | Yes (metadata API) | No (data) | No (data) |
| Packageable | Yes | Partially | Partially |
| SOQL to query | Yes (no limit count) | Yes (counts against limit) | No — use `getInstance()` |
| Cached | Yes | Yes (list) | Yes (hierarchy) |
| Per-user values | No | No | Yes (profile/user level) |
| Editable in Apex | No (read-only) | Yes | Yes |
| **Use when** | Config that deploys with code | App-level settings, mutable | User/profile-specific settings |

**Modern guidance:** Prefer Custom Metadata Types for anything that should travel with your metadata deployments. Use Custom Settings only when you need mutability in Apex or per-user/profile configuration.
