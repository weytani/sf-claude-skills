---
name: Salesforce SOQL Reference
description: Dense platform reference for Salesforce Object Query Language (SOQL) — syntax, operators, relationships, aggregates, date literals, performance, and common errors.
when_to_use: When writing, debugging, or optimizing SOQL queries in Apex, Flow, or Salesforce APIs. Use this for syntax lookup, relationship query patterns, performance tuning, and error resolution.
version: "62.0"
---

# SOQL Reference (API v62.0)

## Basic Syntax

```apex
SELECT fieldList | subquery | TYPEOF
FROM objectType
[WHERE conditionExpression]
[WITH [DATA CATEGORY] filterExpression]
[GROUP BY fieldList | ROLLUP(fieldList) | CUBE(fieldList)]
[HAVING aggregateCondition]
[ORDER BY fieldList ASC|DESC NULLS FIRST|LAST]
[LIMIT numberOfRows]
[OFFSET numberOfRows]
[FOR VIEW | FOR REFERENCE | FOR UPDATE]
```

**Governor limits:** 50,000 rows per transaction (synchronous), 200 queries per transaction. Use `FOR UPDATE` to lock rows in a transaction.

---

## WHERE Operators

| Operator | Data Types | Example |
|---|---|---|
| `=` | All | `WHERE Status = 'Active'` |
| `!=` | All | `WHERE Status != 'Closed'` |
| `<` | Number, Date, DateTime | `WHERE Amount < 10000` |
| `>` | Number, Date, DateTime | `WHERE CreatedDate > 2024-01-01T00:00:00Z` |
| `<=` | Number, Date, DateTime | `WHERE Probability <= 50` |
| `>=` | Number, Date, DateTime | `WHERE AnnualRevenue >= 1000000` |
| `IN` | All (compare against list) | `WHERE Id IN :accountIds` |
| `NOT IN` | All (compare against list) | `WHERE Status NOT IN ('Closed', 'Archived')` |
| `LIKE` | String | `WHERE Name LIKE 'Acme%'` |
| `INCLUDES` | Multi-select picklist only | `WHERE Markets__c INCLUDES ('EMEA', 'APAC')` |
| `EXCLUDES` | Multi-select picklist only | `WHERE Markets__c EXCLUDES ('LATAM')` |

**LIKE wildcards:** `%` matches zero or more characters, `_` matches exactly one character. Case-insensitive for standard fields, depends on collation for custom.

**Logical operators:** `AND`, `OR`, `NOT`. Parentheses control precedence.

```apex
WHERE (Status = 'Active' OR Status = 'Pending') AND CreatedDate > LAST_MONTH
```

---

## Relationship Queries

### Child-to-Parent (Dot Notation)

Traverse up to **5 levels** of parent relationships. Uses the **parent relationship name** with dot notation.

```apex
// Standard relationship — use parent object name directly
SELECT Contact.Account.Name, Contact.Account.Owner.Email
FROM Contact
WHERE Contact.Account.Industry = 'Technology'

// Custom relationship — replace __c with __r
SELECT Custom_Child__c.Parent_Object__r.Name
FROM Custom_Child__c
```

### Parent-to-Child (Subquery)

Only **1 level deep** allowed. Uses the **child relationship name** in a subquery in the SELECT clause.

```apex
// Standard relationship — use plural child name
SELECT Name, (SELECT LastName, Email FROM Contacts)
FROM Account
WHERE Industry = 'Finance'

// Custom relationship — use __r suffix of child's lookup field
SELECT Name, (SELECT Amount__c FROM Line_Items__r)
FROM Order__c
```

### Relationship Name Rules

| Object Type | Child-to-Parent (dot) | Parent-to-Child (subquery) |
|---|---|---|
| Standard → Standard | `Contact.Account.Name` | `(SELECT LastName FROM Contacts)` |
| Custom → Standard | `Child__c.Account.Name` | `(SELECT Name FROM Children__r)` |
| Standard → Custom | `Contact.Custom__r.Name` | `(SELECT Name FROM Customs__r)` |
| Custom → Custom | `Child__c.Parent__r.Name` | `(SELECT Name FROM Children__r)` |

**Key rule:** Standard child relationships use the **plural object name** (Contacts, Cases, Opportunities). Custom child relationships use the **lookup field API name** with `__r` replacing `__c`.

To find the relationship name: Schema Builder, or `DescribeSObjectResult.getChildRelationships()` in Apex.

---

## Aggregate Functions

| Function | GROUP BY Required | Notes |
|---|---|---|
| `COUNT()` | No | Returns integer. Cannot use with other SELECT fields unless grouped. `SELECT COUNT() FROM Account` |
| `COUNT(field)` | Yes (if other fields selected) | Counts non-null values. `SELECT Industry, COUNT(Id) FROM Account GROUP BY Industry` |
| `COUNT_DISTINCT(field)` | Yes (if other fields selected) | Counts unique non-null values. |
| `SUM(field)` | Yes | Numeric fields only. `SELECT AccountId, SUM(Amount) FROM Opportunity GROUP BY AccountId` |
| `AVG(field)` | Yes | Numeric fields only. |
| `MIN(field)` | Yes | Works on Number, Date, DateTime, String. |
| `MAX(field)` | Yes | Works on Number, Date, DateTime, String. |

**Using aliases and HAVING:**

```apex
SELECT StageName, COUNT(Id) cnt, SUM(Amount) total
FROM Opportunity
GROUP BY StageName
HAVING COUNT(Id) > 5 AND SUM(Amount) > 100000
```

**Rules:**
- Aggregate results return `List<AggregateResult>`. Access fields via `.get('alias')` or `.get('expr0')`, `.get('expr1')` if no alias.
- `GROUP BY` cannot include long text area, multi-select picklist, or rich text fields.
- `GROUP BY ROLLUP(field)` adds subtotals. `GROUP BY CUBE(f1, f2)` adds cross-tabulation subtotals.
- Max 2,000 rows from aggregate queries. Use `HAVING` to filter, not `WHERE` (which filters before aggregation).

---

## Date Literals

| Literal | Meaning |
|---|---|
| `TODAY` | Current day (midnight to midnight, user timezone) |
| `YESTERDAY` | Previous day |
| `TOMORROW` | Next day |
| `THIS_WEEK` | Week containing today (starts on locale-defined day) |
| `LAST_WEEK` | Previous full week |
| `NEXT_WEEK` | Next full week |
| `THIS_MONTH` | Current calendar month |
| `LAST_MONTH` | Previous full calendar month |
| `NEXT_MONTH` | Next full calendar month |
| `THIS_QUARTER` | Current calendar quarter |
| `LAST_QUARTER` | Previous full calendar quarter |
| `NEXT_QUARTER` | Next full calendar quarter |
| `THIS_YEAR` | Current calendar year |
| `LAST_YEAR` | Previous full calendar year |
| `NEXT_YEAR` | Next full calendar year |
| `LAST_N_DAYS:n` | From n days ago through today (inclusive) |
| `NEXT_N_DAYS:n` | From today through n days from now (inclusive) |
| `N_DAYS_AGO:n` | Exactly n days ago (single day) |
| `LAST_N_MONTHS:n` | From n months ago to end of current month |
| `NEXT_N_MONTHS:n` | From start of current month to n months from now |
| `LAST_N_QUARTERS:n` | From n quarters ago through end of current quarter |
| `NEXT_N_QUARTERS:n` | From start of current quarter through n quarters from now |
| `THIS_FISCAL_QUARTER` | Current fiscal quarter (org fiscal year settings) |
| `THIS_FISCAL_YEAR` | Current fiscal year |
| `LAST_FISCAL_QUARTER` | Previous full fiscal quarter |
| `LAST_FISCAL_YEAR` | Previous full fiscal year |
| `NEXT_FISCAL_QUARTER` | Next full fiscal quarter |
| `NEXT_FISCAL_YEAR` | Next full fiscal year |

```apex
SELECT Id, Name FROM Opportunity
WHERE CloseDate = LAST_N_DAYS:90 AND StageName != 'Closed Won'
```

**Date format in WHERE (non-literal):** `2024-03-15` (Date), `2024-03-15T14:30:00Z` (DateTime in UTC), `2024-03-15T14:30:00-07:00` (DateTime with offset).

---

## TYPEOF (Polymorphic Relationships)

Polymorphic fields can reference multiple object types (e.g., `Task.WhatId` → Account, Opportunity, etc.; `Event.WhoId` → Contact, Lead). `TYPEOF` lets you select different fields depending on the referenced object type.

**Syntax:**

```apex
SELECT
  TYPEOF relationshipName
    WHEN ObjectType THEN fieldList
    WHEN ObjectType THEN fieldList
    ELSE fieldList
  END
FROM objectType
```

**Example — querying Task.What:**

```apex
SELECT Id, Subject,
  TYPEOF What
    WHEN Account THEN Name, Industry, AnnualRevenue
    WHEN Opportunity THEN Name, StageName, Amount
    ELSE Name
  END
FROM Task
WHERE CreatedDate > LAST_MONTH
```

**Rules:**
- Only works on polymorphic relationship fields (`WhatId`, `WhoId`, `OwnerId` on some objects).
- Cannot use `TYPEOF` in `WHERE`, `GROUP BY`, or `ORDER BY`.
- Use `ELSE` as a fallback — it fires when the referenced type doesn't match any `WHEN`.
- Alternative without TYPEOF: `WHERE What.Type = 'Account'` filters by type, then use `What.Name`.

---

## Semi-Joins and Anti-Joins

### Semi-Join (IN subquery) — Records that HAVE related records

```apex
// Accounts that have at least one Opportunity
SELECT Id, Name FROM Account
WHERE Id IN (SELECT AccountId FROM Opportunity)

// Contacts on Accounts in the Technology industry
SELECT Id, Name FROM Contact
WHERE AccountId IN (SELECT Id FROM Account WHERE Industry = 'Technology')
```

### Anti-Join (NOT IN subquery) — Records that LACK related records

```apex
// Accounts with no Contacts
SELECT Id, Name FROM Account
WHERE Id NOT IN (SELECT AccountId FROM Contact)

// Leads not linked to any Campaign
SELECT Id, Name FROM Lead
WHERE Id NOT IN (SELECT LeadId FROM CampaignMember)
```

**Rules:**
- Subquery must return a single field (typically an Id or lookup field).
- Max **1 level** of subquery nesting (no subqueries inside subqueries inside subqueries).
- You can use up to **2** semi-join or anti-join subqueries in a single WHERE clause.
- The inner query is subject to the same governor limits.

---

## Performance

### Selective Query Thresholds

| Index Type | Selectivity Threshold |
|---|---|
| Standard index | Filter returns <10% of total object records |
| Custom index (non-standard) | Filter returns <5% of total object records |
| For objects with >1M records | Filter returns <333,333 records (hard cap) |

A query is **selective** if the leading filter (or filter combination via AND) uses an indexed field and meets the threshold. Non-selective queries on large objects (>200K records) fail at runtime.

### Default Indexed Fields

| Field | Always Indexed |
|---|---|
| `Id` | Yes |
| `Name` | Yes |
| `OwnerId` | Yes |
| `CreatedDate` | Yes |
| `SystemModstamp` | Yes (LastModifiedDate is NOT indexed by default) |
| `RecordTypeId` | Yes |
| Lookup fields (foreign keys) | Yes |
| Master-Detail fields | Yes |
| External ID fields (`__c` marked External ID) | Yes |
| Unique fields | Yes |
| `Email` on Contact/Lead | Yes |

**NOT indexed by default:** `LastModifiedDate`, `CreatedById`, formula fields, text fields (unless External ID), picklist fields. Request custom indexes via Salesforce Support.

### Query Plan Tool

Use the **Query Plan** tool in Developer Console (Enable: Help → Settings → Enable Query Plan) or the REST API:

```
/services/data/v62.0/query/?explain=SELECT Id FROM Account WHERE Name = 'Acme'
```

Returns: leading operation type, cost estimate, index used, selectivity info.

### Performance Tips

- Filter on indexed fields first. Combine selective filters with `AND`.
- Avoid `!=`, `NOT IN`, `EXCLUDES` on large datasets — they prevent index usage.
- Avoid leading wildcards in `LIKE` (`'%acme'`) — not indexable. Trailing wildcards are fine (`'Acme%'`).
- `LIMIT` does NOT make a query selective — selectivity is determined before LIMIT is applied.
- Avoid `OR` across different fields — each branch must be independently selective.
- Use skinny tables (request via Support) for extremely high-volume objects.

---

## Dynamic SOQL

Build queries at runtime using `Database.query()`.

```apex
// Basic dynamic query with bind variable
String industry = 'Technology';
List<Account> accts = Database.query(
    'SELECT Id, Name FROM Account WHERE Industry = :industry'
);

// Dynamic field list
String fields = 'Id, Name, Industry';
String query = 'SELECT ' + fields + ' FROM Account LIMIT 100';
List<Account> results = Database.query(query);
```

### Bind Variables

| Syntax | Supported | Notes |
|---|---|---|
| `:variableName` | Yes | Primitives, lists, sets, maps (keys). **Safe from injection.** |
| `:listVariable` | Yes (in `IN` clause) | `WHERE Id IN :idSet` |
| Bind in dynamic string concat | **No** | `:var` only works with `Database.query()`, not inside concatenated strings passed to it from other methods. Bind vars must be in scope. |

### Injection Prevention

**Always** use `String.escapeSingleQuotes()` when concatenating user input:

```apex
// DANGEROUS — SQL injection vulnerable
String userInput = 'Acme\' OR Name != \'';
String q = 'SELECT Id FROM Account WHERE Name = \'' + userInput + '\'';

// SAFE — escape user input
String q = 'SELECT Id FROM Account WHERE Name = \''
    + String.escapeSingleQuotes(userInput) + '\'';

// SAFEST — use bind variables (preferred)
String safeName = userInput;
List<Account> accts = Database.query(
    'SELECT Id FROM Account WHERE Name = :safeName'
);
```

**Rule of thumb:** Use bind variables whenever possible. Fall back to `escapeSingleQuotes()` only when bind variables aren't feasible (e.g., dynamic field names, object names).

---

## SOQL vs SOSL

| Dimension | SOQL | SOSL |
|---|---|---|
| **Purpose** | Query specific object with precise filters | Full-text search across multiple objects |
| **Syntax** | `SELECT ... FROM ... WHERE ...` | `FIND {searchTerm} IN ... RETURNING ...` |
| **Searches** | Single object (with relationships) | Multiple objects simultaneously |
| **Search scope** | Field-level equality/comparison | Text index (tokenized, stemmed) |
| **Wildcard** | `LIKE 'Acme%'` | `FIND {Acme*}` (also `?` for single char) |
| **Returns** | `List<SObject>` | `List<List<SObject>>` (one list per object) |
| **Governor limit** | 50,000 rows / 200 queries per txn | 2,000 rows / 20 queries per txn |
| **Exact match** | Yes — precise filters | No — text search, may return partial matches |
| **Use when** | You know which object and fields to filter | Searching across objects, or text search without knowing exact field values |
| **Relationship queries** | Yes (parent/child subqueries) | No (flat results per object) |
| **Aggregate functions** | Yes (COUNT, SUM, AVG, etc.) | No |
| **ORDER BY** | Yes | Yes (within each RETURNING object) |
| **Apex syntax** | `[SELECT ...]` or `Database.query()` | `[FIND ...]` or `Search.query()` |

```apex
// SOQL — precise lookup
List<Account> accts = [SELECT Id, Name FROM Account WHERE Industry = 'Technology'];

// SOSL — text search across objects
List<List<SObject>> results = [FIND 'Acme*' IN NAME FIELDS
    RETURNING Account(Id, Name), Contact(Id, LastName)];
List<Account> accounts = (List<Account>) results[0];
List<Contact> contacts = (List<Contact>) results[1];
```

---

## Common Errors

| Error Message | Cause | Fix |
|---|---|---|
| `Didn't understand relationship 'X' in field path` | Wrong relationship name in dot notation or subquery. | Verify API name. Standard children: plural (`Contacts`). Custom: `__r` suffix. Use Schema Builder or `getChildRelationships()`. |
| `MALFORMED_QUERY: unexpected token` | Syntax error — missing quote, comma, keyword typo. | Check string literals are single-quoted, commas between fields, no trailing comma before FROM. |
| `Non-selective query against large object type` | Query on object with >200K rows without indexed filter meeting selectivity threshold. | Add indexed field filter. Check with Query Plan tool. Request custom index if needed. |
| `SOQL statements cannot be empty` | Empty string passed to `Database.query()`. | Validate query string is non-null and non-empty before execution. |
| `Aggregate query has too many rows for direct assignment` | Aggregate query returns >2,000 grouped rows. Use `List<AggregateResult>`. | Add `HAVING` or more selective `WHERE` to reduce groups. Or use `Database.query()` to handle as list. |
| `System.QueryException: List has no rows for assignment` | `[SELECT ... LIMIT 1]` assigned to single SObject but returned 0 rows. | Assign to `List<SObject>` and check `.isEmpty()`, or wrap in try-catch. |
| `System.LimitException: Too many SOQL queries: 201` | >200 SOQL queries in synchronous transaction (100 in async). | Bulkify — move queries out of loops. Use collections and `WHERE Id IN :idSet`. |
| `System.LimitException: Too many query rows: 50001` | Query returned >50,000 total rows across all queries in transaction. | Add selective filters, use LIMIT, or move to Batch Apex for large data volumes. |
| `Only root queries support aggregate expressions` | Aggregate function used in a subquery. | Move aggregate to outer query. Subqueries only support simple field selection. |
| `Field 'X' cannot be grouped` | `GROUP BY` on long text area, rich text, or multi-select picklist. | Remove ungroupable field. Group by Id or a supported field type instead. |
| `No such column 'X' on entity 'Y'` | Field API name misspelled or field doesn't exist on object. | Verify field API name in Object Manager. Custom fields end in `__c`. |
| `Implementation restriction: sub-selects can only be two levels deep` | Nested subquery too deep (subquery in a subquery in a subquery). | Flatten the query. Execute inner query first, pass results via bind variable. |
