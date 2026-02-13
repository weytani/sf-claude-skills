---
name: Salesforce Visualforce
description: Platform reference for Visualforce pages — markup language for building custom UIs on Salesforce. LEGACY technology; use LWC for new work.
when_to_use: When the user is working with Visualforce pages (.page files), VF controllers, or asks about Visualforce markup, expressions, view state, or PDF rendering in Salesforce.
version: "62.0"
---

# Salesforce Visualforce — Platform Reference

**⚠️ LEGACY TECHNOLOGY — Use Lightning Web Components (LWC) for all new development. Visualforce remains relevant for: PDF rendering (renderAs='pdf'), email templates, and some Service Console integrations.**

---

## Page Structure

Every Visualforce page begins with `<apex:page>`. Key attributes:

| Attribute | Type | Description |
|---|---|---|
| `controller` | String | Fully qualified name of a custom Apex controller class |
| `extensions` | String | Comma-separated list of controller extension classes |
| `standardController` | String | API name of the sObject for automatic CRUD (e.g., `Account`) |
| `renderAs` | String | Output format — `pdf` is the primary use case; renders page as downloadable PDF |
| `sidebar` | Boolean | Show Classic sidebar (`true`/`false`). Default `true` |
| `showHeader` | Boolean | Show Salesforce header. Default `true` |
| `lightningStylesheets` | Boolean | Apply Lightning Design System styling to standard components |
| `docType` | String | HTML doctype — `html-5.0` for HTML5 |
| `applyBodyTag` | Boolean | Whether VF wraps output in `<body>`. Set `false` to control your own `<body>` |
| `applyHtmlTag` | Boolean | Whether VF wraps output in `<html>`. Set `false` for full control (common with `renderAs="pdf"`) |
| `action` | Expression | Apex method invoked on page load (e.g., `{!init}`) — fires before rendering |
| `readOnly` | Boolean | Runs page in read-only mode — raises SOQL row limit from 50,000 to 1,000,000 |

> **LEGACY NOTE:** `sidebar` and `showHeader` only apply in Salesforce Classic. In Lightning Experience, Visualforce pages render inside an iframe and these attributes are ignored.

---

## Controller Types

| Type | Declaration | Use Case |
|---|---|---|
| **Standard Controller** | `standardController="Account"` | Automatic CRUD for a single sObject. No Apex needed. |
| **Custom Controller** | `controller="MyController"` | Full Apex class — complete control, no automatic sObject binding |
| **Controller Extension** | `standardController="Account" extensions="MyExt"` | Extends a standard or custom controller with additional logic |

**Standard Controller + Extension Pattern** (most common LEGACY pattern):

```apex
// ABOUTME: Controller extension for Account pages.
// ABOUTME: Adds custom query logic on top of the standard Account controller.
public class AccountExtension {
    private final Account acct;

    public AccountExtension(ApexPages.StandardController stdController) {
        this.acct = (Account) stdController.getRecord();
    }

    public List<Contact> getRelatedContacts() {
        return [SELECT Id, Name, Email FROM Contact WHERE AccountId = :acct.Id];
    }

    public PageReference doCustomSave() {
        update acct;
        return new PageReference('/' + acct.Id);
    }
}
```

```html
<!-- LEGACY Visualforce page using standard controller + extension -->
<apex:page standardController="Account" extensions="AccountExtension">
    <apex:form>
        <apex:pageBlock title="Account Details">
            <apex:inputField value="{!Account.Name}" />
            <apex:pageBlockTable value="{!relatedContacts}" var="c">
                <apex:column value="{!c.Name}" />
                <apex:column value="{!c.Email}" />
            </apex:pageBlockTable>
            <apex:commandButton value="Save" action="{!doCustomSave}" />
        </apex:pageBlock>
    </apex:form>
</apex:page>
```

---

## Expression Language

**Syntax:** `{!expression}`

Expressions can reference controller properties, global variables, and simple formulas. They bind the page to server-side data.

### Global Variables

| Variable | Returns | Example |
|---|---|---|
| `$User.Id` | Current user's 18-char ID | `{!$User.Id}` |
| `$User.FirstName` | Current user's first name | `{!$User.FirstName}` |
| `$User.ProfileId` | Current user's Profile ID | `{!$User.ProfileId}` |
| `$Label.labelName` | Custom label value (supports translation) | `{!$Label.Error_Message}` |
| `$Resource.resourceName` | URL to a static resource | `{!$Resource.jQuery}` or `{!URLFOR($Resource.zip, 'path/file.js')}` |
| `$Action.actionName` | URL for a standard action | `{!URLFOR($Action.Account.New)}` |
| `$Api.Session_ID` | Current session ID | `{!$Api.Session_ID}` |
| `$CurrentPage.Name` | API name of the current VF page | `{!$CurrentPage.Name}` |
| `$CurrentPage.parameters.paramName` | URL query parameter value | `{!$CurrentPage.parameters.id}` |
| `$ObjectType.sObject.fields.field` | Schema describe info for a field | `{!$ObjectType.Account.fields.Name.label}` |
| `$Setup.customSetting.field` | Hierarchy custom setting field value | `{!$Setup.MySettings__c.API_Key__c}` |
| `$Site.BaseUrl` | Base URL of the current Salesforce Site | `{!$Site.BaseUrl}` |
| `$System.OriginDateTime` | Fixed datetime: 1900-01-01 00:00:00 | `{!$System.OriginDateTime}` |
| `$Profile.Name` | Current user's profile name | `{!$Profile.Name}` |
| `$Permission.permName` | Boolean — whether user has custom permission | `{!$Permission.Can_Export}` |

---

## Common Components

> **LEGACY NOTE:** All `apex:` components are Visualforce-only. They have no equivalent in LWC — LWC uses standard HTML and Lightning base components instead.

| Component | Purpose | Key Attributes |
|---|---|---|
| `apex:form` | HTML form wrapper — required for any postback | `id`, `forceSSL` |
| `apex:inputField` | Auto-renders input based on field type & FLS | `value`, `required`, `label`, `rendered` |
| `apex:outputField` | Auto-renders read-only output respecting field type | `value`, `label`, `rendered` |
| `apex:inputText` | Plain text input — no field metadata awareness | `value`, `id`, `label`, `maxlength` |
| `apex:selectList` | Picklist / multi-select dropdown | `value`, `size`, `multiselect`, `label` |
| `apex:selectOption` | Single option inside `apex:selectList` | `itemValue`, `itemLabel`, `itemDisabled` |
| `apex:commandButton` | Button that invokes an Apex action (postback) | `action`, `value`, `reRender`, `status`, `onclick` |
| `apex:commandLink` | Link that invokes an Apex action (postback) | `action`, `value`, `reRender`, `target` |
| `apex:actionFunction` | Invisible component — invoke Apex from JavaScript | `name`, `action`, `reRender`, `oncomplete` |
| `apex:actionSupport` | Adds AJAX event handler to a parent component | `event`, `action`, `reRender`, `status` |
| `apex:actionPoller` | Periodically invokes an Apex action via AJAX | `action`, `interval` (min 5s), `reRender` |
| `apex:pageBlock` | Styled container block (Classic look) | `title`, `mode` (`edit`/`detail`), `rendered` |
| `apex:pageBlockSection` | Section within a pageBlock, auto 2-column layout | `title`, `columns`, `collapsible` |
| `apex:pageBlockTable` | Data table inside a pageBlock (styled rows) | `value`, `var`, `rows`, `first`, `rendered` |
| `apex:dataTable` | HTML table without pageBlock styling | `value`, `var`, `rows`, `cellpadding` |
| `apex:repeat` | Iterator — renders child markup for each item | `value`, `var`, `first`, `rows` |
| `apex:variable` | Declares a local variable in the page | `var`, `value` |
| `apex:outputPanel` | `<span>` or `<div>` wrapper — key reRender target | `id`, `layout` (`block`/`inline`), `rendered` |
| `apex:include` | Includes another Visualforce page inline | `pageName` |
| `apex:composition` | Template composition — define/insert pattern | `template` (parent uses `apex:define`, child uses `apex:insert`) |

---

## JavaScript Integration

### apex:actionFunction — Invoke Apex from JavaScript

Declares a JavaScript function that calls an Apex method and can rerender parts of the page.

```html
<!-- LEGACY: apex:actionFunction wiring -->
<apex:page controller="MyController">
    <apex:form>
        <apex:actionFunction name="jsRefresh"
                             action="{!refreshData}"
                             reRender="resultsPanel"
                             oncomplete="handleComplete();" />

        <apex:actionFunction name="jsUpdateName"
                             action="{!updateName}"
                             reRender="resultsPanel">
            <apex:param name="newName" assignTo="{!nameParam}" value="" />
        </apex:actionFunction>

        <button type="button" onclick="jsRefresh();">Refresh</button>
        <button type="button" onclick="jsUpdateName('Acme Corp');">Update</button>

        <apex:outputPanel id="resultsPanel">
            {!resultMessage}
        </apex:outputPanel>
    </apex:form>
</apex:page>
```

### @RemoteAction — JavaScript Remoting

Direct Apex invocation from JavaScript. No form submission, no view state, better performance.

**Apex:**

```apex
// ABOUTME: Controller with RemoteAction for high-performance JS remoting.
// ABOUTME: Returns serialized data directly to client-side JavaScript.
public class RemotingController {
    @RemoteAction
    public static String doSomething(String param) {
        return 'Processed: ' + param;
    }

    @RemoteAction
    public static List<Account> searchAccounts(String term) {
        String wildcard = '%' + String.escapeSingleQuotes(term) + '%';
        return [SELECT Id, Name FROM Account WHERE Name LIKE :wildcard LIMIT 20];
    }
}
```

**Visualforce JavaScript:**

```html
<apex:page controller="RemotingController">
    <script>
        function callRemote() {
            Visualforce.remoting.Manager.invokeAction(
                '{!$RemoteAction.RemotingController.doSomething}',
                'hello',
                function(result, event) {
                    if (event.status) {
                        console.log('Result: ' + result);
                    } else {
                        console.error(event.message);
                    }
                },
                { escape: true }
            );
        }
    </script>
    <button onclick="callRemote();">Call Remote</button>
</apex:page>
```

### When to Use Each

| Approach | Best For | View State? | Performance |
|---|---|---|---|
| `apex:actionFunction` | Simple rerender scenarios, form-bound data | Yes — full postback cycle | Slower — serializes view state |
| `@RemoteAction` | Large data, complex JS UIs, performance-critical | No — stateless | Faster — bypasses view state entirely |

---

## View State

Visualforce maintains server-side state between postbacks in a hidden form field called the **view state**.

**Limit: 170 KB** (serialized, encrypted). Exceeding this throws `Maximum view state size limit (170KB) exceeded`.

The `transient` keyword excludes a field from serialization into the view state:

```apex
public class MyController {
    // Included in view state
    public String importantField { get; set; }

    // Excluded from view state — must be re-fetched on each postback
    transient public List<Account> largeResultSet { get; set; }
}
```

### View State Reduction Techniques

| Technique | How It Helps |
|---|---|
| `transient` keyword on controller fields | Excludes large/reconstructable data from serialization |
| Reduce component tree | Fewer `apex:` components = less state to track |
| Pagination | Limit records per page instead of loading thousands |
| Lazy loading | Load data only when a section is expanded/requested |
| JavaScript Remoting (`@RemoteAction`) | Bypasses view state entirely — data lives in JS |
| Use `apex:repeat` instead of `apex:dataTable` | Lighter component with less state overhead |
| Set `readOnly="true"` on `apex:page` | No view state at all (page cannot post back) |
| Move logic to JavaScript + REST API | Eliminate server-side state for interactive features |

---

## Common Pitfalls

> **LEGACY NOTE:** Many of these errors are unique to Visualforce's postback/view-state model and do not occur in LWC.

| Symptom | Cause | Fix |
|---|---|---|
| `Maximum view state size limit (170KB) exceeded` | Controller stores too much data in non-transient fields | Mark large collections as `transient`; paginate results; switch to `@RemoteAction` for data-heavy operations |
| `Attempt to de-reference a null object` | A controller getter returns null and the page dereferences a property on it | Add null checks in Apex (`if (obj != null)`); use `rendered="{!obj != null}"` on components |
| `Content cannot be displayed: page not found` | VF page API name is wrong, page is not deployed, or user lacks permission | Verify page name in Setup > Visualforce Pages; check profile/permission set access |
| `Too many SOQL queries` (101 limit) | Queries inside loops, or `action` attribute triggers queries stacked with getter queries | Move queries out of loops; use collections and maps; audit all getters invoked during page render |
| `SObject row was retrieved via SOQL without querying the requested field` | Standard controller auto-query doesn't include a field referenced on the page; or custom SOQL omits it | Add the field to your SOQL `SELECT`; for standard controllers, call `stdController.addFields(new List<String>{'FieldName'})` in the extension constructor |
| AJAX rerender does nothing / component not updating | `reRender` target ID doesn't match, or target component has `rendered="false"` (not in DOM) | Wrap target in an `apex:outputPanel` that is always rendered; rerender the wrapper's ID instead |

---

## Quick Reference — LEGACY Migration Guidance

| Visualforce Pattern | Modern Equivalent (LWC) |
|---|---|
| `<apex:page>` | LWC component + Lightning App Page |
| `<apex:form>` + `<apex:commandButton>` | `lightning-button` + wire/imperative Apex |
| `<apex:inputField>` | `lightning-input-field` inside `lightning-record-edit-form` |
| `<apex:pageBlockTable>` | `lightning-datatable` |
| `@RemoteAction` | `@AuraEnabled(cacheable=true)` + `@wire` |
| `$CurrentPage.parameters` | `CurrentPageReference` wire adapter |
| View State | No equivalent — LWC is stateless on the server |

**Keep Visualforce for:** `renderAs="pdf"`, Visualforce email templates, and edge-case Classic-only integrations. **Use LWC for everything else.**
