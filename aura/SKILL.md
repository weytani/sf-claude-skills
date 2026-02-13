---
name: "Salesforce Aura Components"
description: "Platform reference for Aura component development — bundle structure, markup, controllers, events, Apex integration, and migration guidance to LWC."
when_to_use: "When maintaining, debugging, or migrating existing Aura components. NOT for new development — use LWC instead."
version: "62.0"
---

# Salesforce Aura Components Reference

**LEGACY TECHNOLOGY — Use Lightning Web Components (LWC) for all new development. Aura is documented here for maintaining and migrating existing components.**

---

## 1. Bundle Structure

Every Aura component is a folder (bundle) containing up to 8 files sharing the component name.

| File | Suffix | Purpose | Required |
|------|--------|---------|----------|
| `myComponent.cmp` | `.cmp` | Component markup (XML-based template) | Yes |
| `myComponentController.js` | `Controller.js` | Client-side controller — action handlers bound from markup | No |
| `myComponentHelper.js` | `Helper.js` | Shared JS logic — called from controller or renderer | No |
| `myComponentRenderer.js` | `Renderer.js` | Override default rendering lifecycle (`render`, `rerender`, `afterRender`, `unrender`) | No |
| `myComponent.css` | `.css` | Component-scoped styles | No |
| `myComponent.design` | `.design` | Exposes attributes in Lightning App Builder / Community Builder | No |
| `myComponent.auradoc` | `.auradoc` | Documentation and example usage | No |
| `myComponent.svg` | `.svg` | Custom icon for the component in App Builder | No |

> **LEGACY NOTE**: LWC replaces this 8-file bundle with a simpler structure (HTML, JS, CSS, XML metadata).

---

## 2. Markup Reference

### aura:attribute

Declares a component attribute (property).

```xml
<aura:attribute name="accountName" type="String" default="" description="Name of the account" required="false" />
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | String | Attribute name — accessed via `{!v.name}` |
| `type` | String | Data type: `String`, `Boolean`, `Integer`, `Decimal`, `Date`, `Object`, `List`, `Map`, `Aura.Component[]`, SObject types, custom Apex classes |
| `default` | (varies) | Default value if none supplied |
| `description` | String | Documents the attribute's purpose |
| `required` | Boolean | If `true`, must be provided by parent; defaults to `false` |

### aura:if

Conditional rendering. Removes/adds DOM — does **not** toggle visibility.

```xml
<aura:if isTrue="{!v.showDetails}">
    <p>Details are visible</p>
    <aura:set attribute="else">
        <p>Details are hidden</p>
    </aura:set>
</aura:if>
```

| Parameter | Description |
|-----------|-------------|
| `isTrue` | Expression evaluating to Boolean — renders body when `true` |

### aura:iteration

Loops over a collection.

```xml
<aura:iteration items="{!v.accounts}" var="acct" indexVar="idx">
    <p>{!idx}: {!acct.Name}</p>
</aura:iteration>
```

| Parameter | Description |
|-----------|-------------|
| `items` | Expression resolving to an Array or List |
| `var` | Variable name for current item |
| `indexVar` | Variable name for current index (0-based) |

### Expression Syntax

| Syntax | Meaning | Example |
|--------|---------|---------|
| `{!v.attributeName}` | Read attribute value | `{!v.accountName}` |
| `{!c.controllerMethod}` | Bind controller action | `onclick="{!c.handleClick}"` |
| `{!e.eventParam}` | Access event parameter | `{!e.message}` |
| `{! expression }` | Evaluate expression | `{!v.count + 1}` |
| `{#v.attributeName}` | Unbound expression (one-time, no two-way binding) | `{#v.readOnlyVal}` |

### Value Providers

| Provider | Name | Description |
|----------|------|-------------|
| `v` | View / Attribute | Component attributes — `{!v.myAttr}` |
| `c` | Controller | Client-side controller actions — `{!c.handleSave}` |
| `e` | Event | Event parameters inside handler — `{!e.value}` |

> **LEGACY NOTE**: LWC eliminates value providers entirely. Properties are accessed directly in templates: `{property}`.

---

## 3. Controller / Helper Pattern

**LEGACY NOTE**: The controller-calls-helper pattern is fundamental to Aura. LWC eliminates this split — all logic lives in one JS class.

### Component Markup (`myComponent.cmp`)

```xml
<aura:component implements="force:appHostable" access="global">
    <aura:attribute name="accounts" type="Account[]" />
    <aura:handler name="init" value="{!this}" action="{!c.doInit}" />

    <lightning:button label="Refresh" onclick="{!c.handleRefresh}" />

    <aura:iteration items="{!v.accounts}" var="acct">
        <p>{!acct.Name}</p>
    </aura:iteration>
</aura:component>
```

### Controller (`myComponentController.js`)

```javascript
({
    doInit : function(component, event, helper) {
        helper.fetchAccounts(component);
    },

    handleRefresh : function(component, event, helper) {
        helper.fetchAccounts(component);
    }
})
```

### Helper (`myComponentHelper.js`)

```javascript
({
    fetchAccounts : function(component) {
        var action = component.get("c.getAccounts");
        action.setCallback(this, function(response) {
            var state = response.getState();
            if (state === "SUCCESS") {
                component.set("v.accounts", response.getReturnValue());
            } else {
                console.error("Failed: ", response.getError());
            }
        });
        $A.enqueueAction(action);
    }
})
```

**Why the helper exists**: Controllers are per-component-instance and cannot call each other. Helper methods are shared across the component lifecycle, enabling reusability (called from controller, renderer, or other helper methods) and testability (logic in one place).

---

## 4. Events

### Component Events vs Application Events

| Aspect | Component Event | Application Event |
|--------|----------------|-------------------|
| **Scope** | Parent-child chain only | Any component that registers a handler |
| **Propagation** | Bubbles up DOM hierarchy | Broadcast — all listeners fire |
| **Registration (markup)** | `<aura:registerEvent name="myEvt" type="c:MyEvent" />` | `<aura:registerEvent name="myEvt" type="c:MyAppEvent" />` |
| **Handling (parent)** | `<c:child onmyEvt="{!c.handleEvt}" />` | `<aura:handler event="c:MyAppEvent" action="{!c.handleEvt}" />` |
| **Use case** | Direct parent-child communication | Cross-component, decoupled communication |
| **Performance** | Lightweight — scoped propagation | Heavier — framework-wide broadcast |
| **Event definition** | `<aura:event type="COMPONENT">` | `<aura:event type="APPLICATION">` |

> **LEGACY NOTE**: LWC replaces Component Events with standard `CustomEvent` and Application Events with Lightning Message Service (LMS).

### Component Event Example

**Event definition (`myEvent.evt`)**:

```xml
<aura:event type="COMPONENT" description="Fires when item is selected">
    <aura:attribute name="selectedId" type="String" />
</aura:event>
```

**Child component — register and fire**:

```xml
<!-- childComponent.cmp -->
<aura:component>
    <aura:registerEvent name="itemSelected" type="c:myEvent" />
    <lightning:button label="Select" onclick="{!c.fireSelection}" />
</aura:component>
```

```javascript
// childComponentController.js
({
    fireSelection : function(component, event, helper) {
        var evt = component.getEvent("itemSelected");
        evt.setParams({ "selectedId" : "001XXXXXXXXXXXX" });
        evt.fire();
    }
})
```

**Parent component — handle**:

```xml
<!-- parentComponent.cmp -->
<aura:component>
    <c:childComponent onitemSelected="{!c.handleSelection}" />
</aura:component>
```

```javascript
// parentComponentController.js
({
    handleSelection : function(component, event, helper) {
        var selectedId = event.getParam("selectedId");
        console.log("Selected: " + selectedId);
    }
})
```

---

## 5. Apex Integration

**LEGACY NOTE**: Aura uses `$A.enqueueAction` for all server calls. LWC replaces this with `@wire` decorators and imperative Apex imports.

### Server-Side Controller (Apex)

```java
public with sharing class AccountController {
    @AuraEnabled(cacheable=true)
    public static List<Account> getAccounts(String searchTerm) {
        String wildcard = '%' + searchTerm + '%';
        return [SELECT Id, Name FROM Account WHERE Name LIKE :wildcard LIMIT 50];
    }
}
```

Component declaration:

```xml
<aura:component controller="AccountController">
```

### $A.enqueueAction — Full Pattern

```javascript
// In helper.js
({
    callServer : function(component) {
        // 1. Get the action
        var action = component.get("c.getAccounts");

        // 2. Set parameters
        action.setParams({
            searchTerm : component.get("v.searchText")
        });

        // 3. Set callback
        action.setCallback(this, function(response) {
            var state = response.getState();

            if (state === "SUCCESS") {
                component.set("v.accounts", response.getReturnValue());

            } else if (state === "INCOMPLETE") {
                console.log("No response from server — offline?");

            } else if (state === "ERROR") {
                var errors = response.getError();
                if (errors && errors[0] && errors[0].message) {
                    console.error("Error: " + errors[0].message);
                } else {
                    console.error("Unknown error");
                }
            }
        });

        // 4. Enqueue
        $A.enqueueAction(action);
    }
})
```

### Storable Actions (Client-Side Caching)

```javascript
var action = component.get("c.getAccounts");
action.setStorable();  // Caches response; serves from cache on repeat calls
$A.enqueueAction(action);
```

| Storable Behavior | Detail |
|-------------------|--------|
| **Cache scope** | Per user, per action + params combination |
| **Cache duration** | Framework-managed; no manual TTL control |
| **Refresh** | Framework may re-fetch in background and re-invoke callback with fresh data |
| **Requirement** | Apex method must be `@AuraEnabled(cacheable=true)` |
| **Callback fires** | Potentially **twice** — once from cache, once from server |

### Error Handling Pattern

```javascript
action.setCallback(this, function(response) {
    if (response.getState() === "SUCCESS") {
        component.set("v.data", response.getReturnValue());
    } else {
        var errors = response.getError();
        var message = "Unknown error";
        if (errors && Array.isArray(errors) && errors.length > 0) {
            message = errors[0].message;
            // Check for field-level errors
            if (errors[0].fieldErrors) {
                var fieldErrors = errors[0].fieldErrors;
                for (var field in fieldErrors) {
                    message += "\n" + field + ": " + fieldErrors[field][0].message;
                }
            }
            // Check for page-level errors
            if (errors[0].pageErrors) {
                errors[0].pageErrors.forEach(function(e) {
                    message += "\n" + e.message;
                });
            }
        }
        // Show toast
        var toastEvent = $A.get("e.force:showToast");
        toastEvent.setParams({
            title   : "Error",
            message : message,
            type    : "error"
        });
        toastEvent.fire();
    }
});
```

---

## 6. Migration to LWC

**LEGACY NOTE**: Every row below is a reason to migrate. LWC is standards-based, faster, and actively developed.

| Aura Concept | LWC Equivalent |
|-------------|----------------|
| `aura:attribute name="x" type="String"` | `@api x;` — public reactive property |
| `{!v.attr}` in markup | `{property}` in template (no `!`, no `v.`) |
| `{!c.method}` in markup | `{method}` in template (no `!`, no `c.`) |
| `component.get("v.x")` / `component.set("v.x", val)` | Direct property access: `this.x` / `this.x = val` |
| `$A.enqueueAction(action)` | `@wire(apexMethod)` or imperative `import apexMethod from '@salesforce/apex/...'` |
| `<aura:if isTrue="{!v.show}">` | `<template if:true={show}>` or `<template lwc:if={show}>` / `lwc:elseif` / `lwc:else` |
| `<aura:iteration items="{!v.list}" var="item">` | `<template for:each={list} for:item="item">` or `<template lwc:for:each={list}>` |
| Component event (`aura:event type="COMPONENT"`) | Standard DOM `CustomEvent` — `this.dispatchEvent(new CustomEvent('select', { detail: {...} }))` |
| Application event (`aura:event type="APPLICATION"`) | Lightning Message Service (`@salesforce/messageChannel`) |
| `<aura:method name="doThing">` | `@api doThing()` — public method on LWC class |
| `helper.js` (separate file) | Imported ES modules or private class methods in the component JS |
| `force:navigateToSObjectRecord` | `NavigationMixin` — `this[NavigationMixin.Navigate]({ type: 'standard__recordPage', ... })` |

---

*This reference covers Aura Components API version 62.0. For all new Salesforce UI development, use Lightning Web Components (LWC).*
