---
name: Salesforce Lightning Web Components
description: Platform reference for LWC development — component structure, wire service, data access, lifecycle, events, navigation, and common pitfalls.
when_to_use: Building or debugging Salesforce Lightning Web Components, choosing data access patterns, wiring adapters, handling component communication, or navigating within Lightning Experience.
version: "62.0"
---

# Lightning Web Components — Platform Reference (API v62.0)

## Component File Structure

| File | Purpose | Required |
|------|---------|----------|
| `myComponent.html` | Template markup (HTML with LWC directives: `lwc:if`, `for:each`, `lwc:ref`) | Yes |
| `myComponent.js` | Controller class extending `LightningElement` | Yes |
| `myComponent.css` | Scoped styles (Shadow DOM boundary — no style leakage) | No |
| `myComponent.js-meta.xml` | Deployment metadata: API version, targets, design attributes, capabilities | Yes |
| `__tests__/myComponent.test.js` | Jest unit tests (`@salesforce/sfdx-lwc-jest`) | No (but expected) |

> Folder name = component name. Must be camelCase, start with lowercase letter. Tag name is kebab-case with namespace: `c-my-component`.

---

## js-meta.xml Targets

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <property name="greeting" type="String" label="Greeting" />
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

| Target | Description | Key Attributes / Notes |
|--------|-------------|----------------------|
| `lightning__RecordPage` | Record detail page | `small`, `large`, `full` region sizes via `supportedFormFactors`; receives `recordId` and `objectApiName` automatically |
| `lightning__AppPage` | Custom app page | Single-region or multi-region layouts; no automatic record context |
| `lightning__HomePage` | Home page | Org default or app-specific home pages |
| `lightning__FlowScreen` | Screen Flow | Properties exposed via `@api` become Flow input/output variables; supports `FlowAttributeChangeEvent` and `FlowNavigationNextEvent` |
| `lightning__Tab` | Custom tab | Component fills the full tab content area; no automatic record context |
| `lightning__Inbox` | Outlook/Gmail integration | Einstein Activity Capture; limited API surface |
| `lightning__UtilityBar` | Utility bar panel | Persistent across navigation; use `lightning/platformWorkspaceApi` for minimize/maximize |
| `lightningCommunity__Page` | Experience Cloud page | Access community context via `@salesforce/community`; CSS tokens differ from internal org |
| `lightning__RecordAction` | Record action (quick action) | Replaces Aura quick actions; use `CloseActionScreenEvent` to close the modal |

---

## Decorators

| Decorator | Purpose | Reactive | Example |
|-----------|---------|----------|---------|
| `@api` | Public property/method — parent can set values or call methods | Yes (on assignment) | `@api recordId;` |
| `@wire` | Bind a wire adapter to a property or function; auto-invoked when reactive params change | Yes | `@wire(getRecord, { recordId: '$recordId', fields }) record;` |
| `@track` | Deep-tracks object/array mutations (rarely needed since Spring '20 — all fields are reactive for primitives) | Yes (deep) | `@track complexObj = { nested: { val: 1 } };` |

> **Since Spring '20**: All class fields are reactive. `@track` is only needed when you mutate an object/array in-place and need the template to re-render (e.g., `this.items.push(x)`). Prefer assigning a new reference instead: `this.items = [...this.items, x]`.

---

## Wire Service Patterns

### Wire to Property

```js
import { LightningElement, wire, api } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';
import NAME_FIELD from '@salesforce/schema/Account.Name';

export default class MyComponent extends LightningElement {
    @api recordId;

    @wire(getRecord, { recordId: '$recordId', fields: [NAME_FIELD] })
    record;

    get accountName() {
        return this.record?.data?.fields?.Name?.value;
    }

    get hasError() {
        return !!this.record?.error;
    }
}
```

### Wire to Function

```js
import { LightningElement, wire, api } from 'lwc';
import { getRecord } from 'lightning/uiRecordApi';
import NAME_FIELD from '@salesforce/schema/Account.Name';

export default class MyComponent extends LightningElement {
    @api recordId;
    accountName;
    error;

    @wire(getRecord, { recordId: '$recordId', fields: [NAME_FIELD] })
    wiredRecord({ data, error }) {
        if (data) {
            this.accountName = data.fields.Name.value;
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.accountName = undefined;
        }
    }
}
```

### Reactive Parameters

- Prefix with `$` to make a wire parameter reactive: `{ recordId: '$recordId' }`
- The `$` references a class property. When the property changes, the wire re-invokes.
- Without `$`, the value is passed once at wire initialization and never updated.

### Error Handling Pattern

```js
get errorMessage() {
    if (!this.error) return '';
    // Wire errors can be { body: { message } } or { message }
    return this.error?.body?.message
        ?? this.error?.message
        ?? 'Unknown error';
}
```

### Refreshing Wired Data

```js
import { refreshApex } from '@salesforce/apex';

// Store the full wired result (wire to function pattern)
wiredAccountResult;

@wire(getAccounts)
wiredAccounts(result) {
    this.wiredAccountResult = result; // Store provisioned value
    const { data, error } = result;
    if (data) { this.accounts = data; }
    if (error) { this.error = error; }
}

async handleRefresh() {
    await refreshApex(this.wiredAccountResult);
}
```

> `refreshApex` must receive the **exact provisioned value** from the wire — not a destructured subset. This is why you store the full `result` object.

---

## Data Access Priority Ladder

Use the lightest-weight option that meets your needs. Move down only when the level above cannot satisfy the requirement.

| Priority | Method | When to Use | Caching | Governor Limits |
|----------|--------|-------------|---------|-----------------|
| 1 | **LDS** (`lightning/ui*Api`) | Single-record CRUD, standard fields, no complex logic | Client-side record cache (shared across components) | No Apex limits; counts toward LDS limits |
| 2 | **GraphQL Wire Adapter** (`lightning/uiGraphQLApi`) | Multi-object queries, related records, filtered lists (up to 2000 records) | LDS-managed cache | No Apex limits; subject to GraphQL-specific limits |
| 3 | **uiObjectInfoApi** | Object/picklist metadata, record type info, field describe | Cached | No Apex limits |
| 4 | **Apex** (`@wire` or imperative) | Complex business logic, SOQL beyond LDS capability, DML with triggers, callouts | `@wire`: cached (use `refreshApex`); imperative: `cacheable=true` opt-in | Full governor limits apply |

### LDS Adapters Quick Reference

| Adapter | Module | Purpose |
|---------|--------|---------|
| `getRecord` | `lightning/uiRecordApi` | Read one record |
| `getRecords` | `lightning/uiRecordApi` | Read multiple records (up to 200) |
| `createRecord` | `lightning/uiRecordApi` | Create one record |
| `updateRecord` | `lightning/uiRecordApi` | Update one record |
| `deleteRecord` | `lightning/uiRecordApi` | Delete one record |
| `getFieldValue` | `lightning/uiRecordApi` | Extract field value from record |
| `getFieldDisplayValue` | `lightning/uiRecordApi` | Extract display value (formatted) |
| `getObjectInfo` | `lightning/uiObjectInfoApi` | Object metadata |
| `getPicklistValues` | `lightning/uiObjectInfoApi` | Picklist values for a field |
| `getPicklistValuesByRecordType` | `lightning/uiObjectInfoApi` | Picklist values filtered by record type |

---

## Lifecycle Hooks

| Hook | When It Fires | Can Access DOM | Common Use Cases |
|------|--------------|----------------|------------------|
| `constructor()` | Component instance created; before DOM exists | No | Initialize state, set default values. Must call `super()` first. Never touch `this.template`. |
| `connectedCallback()` | Component inserted into DOM | No (`this.template` exists but children may not be rendered yet) | Fetch data, subscribe to events/LMS channels, set up intervals. Runs parent-first. |
| `renderedCallback()` | After every render/re-render cycle | Yes | DOM measurements, third-party library init, canvas drawing. Runs child-first. **Guard against infinite loops.** |
| `disconnectedCallback()` | Component removed from DOM | No (being torn down) | Cleanup: unsubscribe LMS, clear intervals/timeouts, remove global listeners. |
| `errorCallback(error, stack)` | Unhandled error in descendant component | N/A | Error boundary — log, display fallback UI. Only catches descendant errors, not self. |

> `renderedCallback` fires on **every** re-render. Always gate side-effects: `if (this._initialized) return; this._initialized = true;`

---

## Event Communication

### Parent-to-Child: `@api` Properties and Methods

**Parent template:**
```html
<c-child-component
    record-id={recordId}
    onrefresh={handleChildRefresh}>
</c-child-component>
```

**Child (property):**
```js
export default class ChildComponent extends LightningElement {
    @api recordId;
}
```

**Child (public method — parent calls imperatively):**
```js
// Child
@api
refresh() {
    // re-fetch or recalculate
}

// Parent JS
this.template.querySelector('c-child-component').refresh();
```

### Child-to-Parent: CustomEvent

**Child dispatches:**
```js
handleClick() {
    this.dispatchEvent(new CustomEvent('select', {
        detail: { recordId: this.recordId },
        bubbles: false,    // default: false — stops at parent
        composed: false    // default: false — does not cross Shadow DOM
    }));
}
```

**Parent handles:**
```html
<!-- Handler name = "on" + event name. Event name must be lowercase, no hyphens. -->
<c-child-component onselect={handleSelect}></c-child-component>
```

```js
handleSelect(event) {
    const selectedId = event.detail.recordId;
}
```

### Cross-Tree (Unrelated Components): Lightning Message Service (LMS)

**1. Create message channel** (`force-app/main/default/messageChannels/MyChannel.messageChannel-meta.xml`):
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
    <masterLabel>MyChannel</masterLabel>
    <isExposed>true</isExposed>
    <lightningMessageFields>
        <fieldName>recordId</fieldName>
    </lightningMessageFields>
</LightningMessageChannel>
```

**2. Publisher:**
```js
import { LightningElement, wire } from 'lwc';
import { publish, MessageContext } from 'lightning/messageService';
import MY_CHANNEL from '@salesforce/messageChannel/MyChannel__c';

export default class Publisher extends LightningElement {
    @wire(MessageContext) messageContext;

    handlePublish() {
        publish(this.messageContext, MY_CHANNEL, {
            recordId: '001xx000003ABCDEF'
        });
    }
}
```

**3. Subscriber:**
```js
import { LightningElement, wire } from 'lwc';
import { subscribe, unsubscribe, MessageContext } from 'lightning/messageService';
import MY_CHANNEL from '@salesforce/messageChannel/MyChannel__c';

export default class Subscriber extends LightningElement {
    @wire(MessageContext) messageContext;
    subscription = null;

    connectedCallback() {
        this.subscription = subscribe(
            this.messageContext,
            MY_CHANNEL,
            (message) => this.handleMessage(message)
        );
    }

    disconnectedCallback() {
        unsubscribe(this.subscription);
        this.subscription = null;
    }

    handleMessage(message) {
        console.log('Received recordId:', message.recordId);
    }
}
```

---

## Navigation Patterns

All navigation requires the `NavigationMixin`:

```js
import { NavigationMixin } from 'lightning/navigation';

export default class MyComponent extends NavigationMixin(LightningElement) { }
```

### Navigate to Record Page

```js
this[NavigationMixin.Navigate]({
    type: 'standard__recordPage',
    attributes: {
        recordId: '001xx000003ABCDEF',
        objectApiName: 'Account',
        actionName: 'view'    // 'view' | 'edit' | 'clone'
    }
});
```

### Navigate to List View

```js
this[NavigationMixin.Navigate]({
    type: 'standard__objectPage',
    attributes: {
        objectApiName: 'Contact',
        actionName: 'list'
    },
    state: {
        filterName: 'Recent'  // list view API name
    }
});
```

### Navigate to Custom Tab

```js
this[NavigationMixin.Navigate]({
    type: 'standard__navItemPage',
    attributes: {
        apiName: 'CustomTabName'
    }
});
```

### Navigate to URL

```js
this[NavigationMixin.Navigate]({
    type: 'standard__webPage',
    attributes: {
        url: 'https://example.com'
    }
});
```

### Navigate to Related List

```js
this[NavigationMixin.Navigate]({
    type: 'standard__recordRelationshipPage',
    attributes: {
        recordId: '001xx000003ABCDEF',
        objectApiName: 'Account',
        relationshipApiName: 'Contacts',
        actionName: 'view'
    }
});
```

### Generate URL (without navigating)

```js
connectedCallback() {
    this[NavigationMixin.GenerateUrl]({
        type: 'standard__recordPage',
        attributes: {
            recordId: this.recordId,
            actionName: 'view'
        }
    }).then((url) => {
        this.recordUrl = url;
    });
}
```

---

## Common Pitfalls

| Symptom | Cause | Fix |
|---------|-------|-----|
| `Cannot read property of undefined` on wired data | Wire adapter has not provisioned data yet; template renders before wire resolves | Guard with optional chaining (`record?.data?.fields`) or `lwc:if={record.data}` in template |
| Component not rendering on Lightning page | Missing or incorrect `<target>` in `js-meta.xml`; `isExposed` set to `false` | Add correct target (e.g., `lightning__RecordPage`), set `<isExposed>true</isExposed>`, redeploy |
| Event not received by parent | Event name mismatch (`onSelect` vs `onselect`), or handler attribute missing from parent template | Event names must be all lowercase with no hyphens. Parent uses `on` + event name: `onselect={handleSelect}`. Verify `bubbles`/`composed` if crossing boundaries. |
| `@api` property not updating child | Parent mutates an existing object reference instead of assigning a new one | Assign a new object/array: `this.config = { ...this.config, key: newVal }` — LWC compares references, not deep values |
| CSS not applying to child components | Shadow DOM encapsulation prevents styles from crossing component boundaries | Use CSS custom properties (`--my-var`) on host, `var(--my-var)` in child. Or use `lightning-card` / SLDS design tokens. Avoid `::slotted` hacks. |
| `renderedCallback` infinite loop / max call stack | Modifying a reactive property inside `renderedCallback` triggers re-render, which calls `renderedCallback` again | Gate with a flag: `if (this._hasRendered) return; this._hasRendered = true;` — or move logic to `connectedCallback` |
| Wire adapter returning stale data | Parameter passed without `$` prefix, so wire does not react to property changes | Use reactive syntax: `{ recordId: '$recordId' }` not `{ recordId: this.recordId }` |
| Toast not showing | Missing import or dispatching on wrong element | Import `ShowToastEvent` from `lightning/platformShowToastEvent`. Dispatch on `this`: `this.dispatchEvent(new ShowToastEvent({ title, message, variant }))`. Toast only works in Lightning Experience, not in LWR-based Experience Cloud sites. |
