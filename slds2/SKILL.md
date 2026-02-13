---
name: slds2
description: Salesforce Lightning Design System (SLDS 2) platform reference — design tokens, grid, utility classes, component blueprints, icons, accessibility, and LWC/Aura usage patterns.
when_to_use: When building or modifying Salesforce Lightning UI components, applying SLDS classes, using design tokens, or referencing component blueprints and accessibility patterns.
version: "62.0"
---

# SLDS 2 — Salesforce Lightning Design System Reference

## 1. Design Tokens

Design tokens are CSS custom properties following the `var(--slds-g-*)` naming pattern. They provide a single source of truth for color, spacing, typography, elevation, and shape values across the system.

Usage: `color: var(--slds-g-color-brand-1);` or `padding: var(--slds-g-spacing-4);`

| Token | Value / Purpose | Example |
|---|---|---|
| `--slds-g-color-brand-1` | Primary brand color (Salesforce blue) | `color: var(--slds-g-color-brand-1);` |
| `--slds-g-color-error-1` | Error / destructive state | `border-color: var(--slds-g-color-error-1);` |
| `--slds-g-color-success-1` | Success / positive state | `background: var(--slds-g-color-success-1);` |
| `--slds-g-color-warning-1` | Warning / caution state | `color: var(--slds-g-color-warning-1);` |
| `--slds-g-font-size-1` | 0.625rem (10px) — smallest | `font-size: var(--slds-g-font-size-1);` |
| `--slds-g-font-size-2` | 0.75rem (12px) | `font-size: var(--slds-g-font-size-2);` |
| `--slds-g-font-size-3` | 0.8125rem (13px) | `font-size: var(--slds-g-font-size-3);` |
| `--slds-g-font-size-4` | 0.875rem (14px) — body default | `font-size: var(--slds-g-font-size-4);` |
| `--slds-g-font-size-5` | 1rem (16px) | `font-size: var(--slds-g-font-size-5);` |
| `--slds-g-font-size-6` | 1.125rem (18px) | `font-size: var(--slds-g-font-size-6);` |
| `--slds-g-font-size-7` | 1.25rem (20px) — large heading | `font-size: var(--slds-g-font-size-7);` |
| `--slds-g-spacing-1` | 0.25rem (4px) | `margin: var(--slds-g-spacing-1);` |
| `--slds-g-spacing-2` | 0.5rem (8px) | `padding: var(--slds-g-spacing-2);` |
| `--slds-g-spacing-3` | 0.75rem (12px) | `gap: var(--slds-g-spacing-3);` |
| `--slds-g-spacing-4` | 1rem (16px) — base unit | `margin-bottom: var(--slds-g-spacing-4);` |
| `--slds-g-spacing-5` | 1.25rem (20px) | `padding: var(--slds-g-spacing-5);` |
| `--slds-g-spacing-6` | 1.5rem (24px) | `gap: var(--slds-g-spacing-6);` |
| `--slds-g-spacing-7` | 2rem (32px) | `margin-top: var(--slds-g-spacing-7);` |
| `--slds-g-radius-border-1` | Standard border radius (0.25rem) | `border-radius: var(--slds-g-radius-border-1);` |
| `--slds-g-shadow-1` | Subtle elevation (cards, containers) | `box-shadow: var(--slds-g-shadow-1);` |
| `--slds-g-shadow-2` | Higher elevation (modals, popovers) | `box-shadow: var(--slds-g-shadow-2);` |

---

## 2. Grid System

The SLDS grid uses flexbox. Container: `slds-grid`. Columns: `slds-col`.

**Responsive sizing classes:** `slds-size_X-of-Y` where Y is 1–12.

| Class | Effect |
|---|---|
| `slds-grid` | Flex container (horizontal by default) |
| `slds-col` | Flex child column |
| `slds-size_1-of-2` | 50% width |
| `slds-size_1-of-3` | 33.33% width |
| `slds-size_2-of-3` | 66.67% width |
| `slds-size_1-of-4` | 25% width |
| `slds-size_3-of-4` | 75% width |
| `slds-size_1-of-6` | 16.67% width |
| `slds-size_1-of-12` | 8.33% width |
| `slds-wrap` | Allow columns to wrap to next row |
| `slds-grid_vertical` | Stack children vertically |
| `slds-grid_align-center` | Center-align children on main axis |
| `slds-grid_align-spread` | Space children evenly (justify: space-between) |
| `slds-grid_align-end` | Align children to end of main axis |
| `slds-grid_vertical-align-center` | Center-align children on cross axis |
| `slds-grid_pull-padded` | Negate padding on grid edges |

**Responsive breakpoint prefixes:** `slds-small-size_*`, `slds-medium-size_*`, `slds-large-size_*`.

### Code Example

```html
<div class="slds-grid slds-wrap slds-grid_align-spread">
  <div class="slds-col slds-size_1-of-2 slds-medium-size_1-of-3">
    <!-- Column 1 -->
  </div>
  <div class="slds-col slds-size_1-of-2 slds-medium-size_1-of-3">
    <!-- Column 2 -->
  </div>
  <div class="slds-col slds-size_1-of-1 slds-medium-size_1-of-3">
    <!-- Column 3 -->
  </div>
</div>
```

---

## 3. Utility Classes by Category

### Spacing

Pattern: `slds-m-{direction}_{size}` (margin) and `slds-p-{direction}_{size}` (padding).

| Direction | Applies To |
|---|---|
| `top` | Top only |
| `right` | Right only |
| `bottom` | Bottom only |
| `left` | Left only |
| `horizontal` | Left + Right |
| `vertical` | Top + Bottom |
| `around` | All four sides |

| Size | Value |
|---|---|
| `xxx-small` | 0.125rem (2px) |
| `xx-small` | 0.25rem (4px) |
| `x-small` | 0.5rem (8px) |
| `small` | 0.75rem (12px) |
| `medium` | 1rem (16px) |
| `large` | 1.5rem (24px) |
| `x-large` | 2rem (32px) |
| `xx-large` | 3rem (48px) |

Examples: `slds-m-top_medium`, `slds-p-around_small`, `slds-m-horizontal_large`, `slds-p-vertical_x-small`.

### Typography

| Class | Effect |
|---|---|
| `slds-text-heading_large` | Large heading (1.75rem, bold) |
| `slds-text-heading_medium` | Medium heading (1.25rem, bold) |
| `slds-text-heading_small` | Small heading (1rem, bold) |
| `slds-text-body_regular` | Body text (0.8125rem) |
| `slds-text-body_small` | Small body text (0.75rem) |
| `slds-text-title` | Section title — uppercase, letter-spaced |
| `slds-text-title_caps` | All-caps title |
| `slds-text-align_center` | Center-aligned text |
| `slds-text-align_right` | Right-aligned text |
| `slds-text-align_left` | Left-aligned text (default) |
| `slds-text-color_default` | Default text color |
| `slds-text-color_weak` | Muted / secondary text color |
| `slds-text-color_error` | Error text color |
| `slds-text-color_success` | Success text color |
| `slds-text-color_inverse` | Inverse (light on dark) text |

### Alignment / Display

| Class | Effect |
|---|---|
| `slds-show` | Display block (visible) |
| `slds-hide` | Display none (hidden) |
| `slds-hidden` | Visibility hidden (reserves space) |
| `slds-visible` | Visibility visible |
| `slds-is-relative` | Position relative |
| `slds-is-absolute` | Position absolute |
| `slds-is-fixed` | Position fixed |
| `slds-truncate` | Single-line text truncation with ellipsis |
| `slds-has-flexi-truncate` | Allow flex child truncation |
| `slds-assistive-text` | Visually hidden but screen-reader accessible |
| `slds-float_left` | Float left |
| `slds-float_right` | Float right |
| `slds-float_none` | Clear floats |

---

## 4. Component Blueprints

| Component | CSS Class | Key Variants | Notes |
|---|---|---|---|
| Card | `slds-card` | `slds-card_boundary` | Header, body, footer sections. Wraps in `slds-card__header`, `slds-card__body`, `slds-card__footer`. |
| Modal | `slds-modal` | `slds-modal_large`, `slds-modal_medium` | Requires `slds-backdrop` overlay. Sections: `slds-modal__header`, `slds-modal__content`, `slds-modal__footer`. |
| Popover | `slds-popover` | `slds-popover_error`, `slds-popover_warning`, `slds-popover_walkthrough` | Nubbin positioning: `slds-nubbin_top`, `slds-nubbin_bottom`, `slds-nubbin_left`, `slds-nubbin_right`. |
| Data Table | `slds-table` | `slds-table_bordered`, `slds-table_striped`, `slds-table_col-bordered`, `slds-table_fixed-layout` | Use `slds-is-sorted` on column headers. Wrap in `slds-table_header-fixed` for fixed headers. |
| Button | `slds-button` | `slds-button_neutral`, `slds-button_brand`, `slds-button_destructive`, `slds-button_outline-brand`, `slds-button_success`, `slds-button_icon` | Use `slds-button_stretch` for full-width. Disabled via `disabled` attribute. |
| Button Group | `slds-button-group` | `slds-button-group-row`, `slds-button-group-list` | Group related actions. Last button uses `slds-button_last`. |
| Tabs (Default) | `slds-tabs_default` | `slds-tabs_scoped` | Active tab: `slds-is-active`. Panels: `slds-tabs_default__content`, toggled via `slds-show`/`slds-hide`. |
| Pill | `slds-pill` | `slds-pill_link`, `slds-pill_bare` | Container: `slds-pill_container` or `slds-listbox_selection-group`. Remove button inside. |
| Badge | `slds-badge` | `slds-badge_inverse`, `slds-badge_lightest` | Inline status indicator. Often inside `slds-badge__icon` for icon+text. |
| Form | `slds-form` | `slds-form_stacked`, `slds-form_horizontal`, `slds-form_compound`, `slds-form_inline` | Wrap fields in `slds-form-element`. Label: `slds-form-element__label`. Control: `slds-form-element__control`. |
| Input | `slds-input` | — | Wrap in `slds-form-element__control`. Pair with `slds-form-element__label`. Error: add `slds-has-error` to `slds-form-element`. |
| Breadcrumb | `slds-breadcrumb` | — | Ordered list (`<ol>`). Items: `slds-breadcrumb__item`. Use `<a>` tags inside. |
| Page Header | `slds-page-header` | `slds-page-header_record-home`, `slds-page-header_related-list` | Sections: `slds-page-header__title`, `slds-page-header__meta-text`, `slds-page-header__control`. |
| Spinner | `slds-spinner` | `slds-spinner_brand`, `slds-spinner_inverse` | Sizes: `slds-spinner_xx-small`, `slds-spinner_x-small`, `slds-spinner_small`, `slds-spinner_medium`, `slds-spinner_large`. Wrap in `slds-spinner_container`. |
| Toast | `slds-notify` | `slds-notify_toast` | Themes: `slds-theme_success`, `slds-theme_error`, `slds-theme_warning`, `slds-theme_info`. Container: `slds-notify_container`. |

---

## 5. Icons

SLDS icons are organized into five categories. In LWC, use the `lightning-icon` base component.

| Category | Prefix | Example Names | Description |
|---|---|---|---|
| Standard | `standard:` | `account`, `contact`, `opportunity`, `lead`, `case`, `task`, `event`, `campaign` | Object icons — colored circular backgrounds |
| Utility | `utility:` | `search`, `close`, `add`, `edit`, `delete`, `check`, `warning`, `info`, `chevrondown`, `chevronright`, `download`, `refresh` | UI action icons — no background |
| Custom | `custom:` | `custom1` through `custom113` | Custom object icons — numbered |
| Doctype | `doctype:` | `pdf`, `excel`, `word`, `ppt`, `csv`, `image`, `zip`, `xml`, `html` | File type icons |
| Action | `action:` | `new_task`, `log_a_call`, `new_note`, `email`, `share`, `approval` | Quick action icons — colored circular backgrounds |

### LWC Icon Syntax

```html
<!-- Standard icon -->
<lightning-icon icon-name="standard:account" size="small"></lightning-icon>

<!-- Utility icon -->
<lightning-icon icon-name="utility:search"></lightning-icon>

<!-- With alternative text for accessibility -->
<lightning-icon icon-name="utility:warning" alternative-text="Warning" variant="warning"></lightning-icon>
```

**Icon sizes:** `xx-small`, `x-small`, `small`, `medium` (default), `large`.

**Icon variants (utility):** `default`, `inverse`, `warning`, `error`, `success`.

---

## 6. Accessibility Patterns

### Required ARIA Attributes

| Attribute | When to Use | Example |
|---|---|---|
| `aria-label` | Interactive element with no visible text label | `<button aria-label="Close dialog">X</button>` |
| `aria-labelledby` | Element labeled by another element's text | `<div aria-labelledby="heading-id">` |
| `aria-describedby` | Additional descriptive text (help text, errors) | `<input aria-describedby="error-msg-id">` |
| `aria-hidden="true"` | Decorative elements to hide from screen readers | `<span aria-hidden="true" class="slds-icon">` |
| `aria-live` | Dynamic content that updates without page reload | `<div aria-live="polite">` for non-urgent; `"assertive"` for urgent |
| `aria-expanded` | Collapsible sections, dropdowns, menus | `<button aria-expanded="false" aria-controls="panel-id">` |
| `aria-selected` | Selected tab, option, or row | `<li role="tab" aria-selected="true">` |
| `aria-current` | Current item in a set (breadcrumbs, nav) | `<a aria-current="page">` |
| `role` | Define widget semantics when HTML element is insufficient | `role="dialog"`, `role="tablist"`, `role="alert"`, `role="status"` |

### Keyboard Navigation

| Pattern | Implementation |
|---|---|
| Focus management | Move focus into modals on open, return focus on close. Use `tabindex="-1"` to make non-interactive elements programmatically focusable. |
| Tab order | Use `tabindex="0"` for custom interactive elements. Avoid `tabindex` > 0. |
| Escape key | Close modals, popovers, dropdowns on `Escape` keypress. |
| Arrow keys | Navigate within composite widgets (tabs, menus, listboxes). |
| Focus trap | Modals must trap focus — Tab/Shift+Tab cycle within the modal. |
| Skip links | Provide skip navigation link at top of page for keyboard users. |

### Color Contrast Requirements

| Text Type | Minimum Contrast Ratio |
|---|---|
| Normal text (< 18px or < 14px bold) | 4.5:1 |
| Large text (>= 18px or >= 14px bold) | 3:1 |
| UI components and graphical objects | 3:1 |
| Non-text contrast (icons, borders) | 3:1 |

Never rely on color alone to convey meaning — always pair with text, icons, or patterns.

---

## 7. SLDS in LWC vs Aura

| Aspect | LWC | Aura |
|---|---|---|
| SLDS loading | Auto-loaded in Lightning Experience; base components include styles | Auto-loaded in Lightning Experience |
| Base components | `<lightning-button>`, `<lightning-input>`, `<lightning-card>`, etc. | `<lightning:button>`, `<lightning:input>`, `<lightning:card>`, etc. |
| Custom CSS | Shadow DOM scoped — styles don't leak in/out. Write in component `.css` file. | Styles can bleed; use component bundle `.css` file. |
| Design tokens in CSS | `@import` not needed in LWC — use `var(--slds-g-*)` directly in `.css` | Use `var(--lwc-*)` or `t()` function in design tokens |
| Applying SLDS classes | Use `class="slds-*"` in template HTML | Use `class="slds-*"` in component markup |
| Dynamic classes | Compute in JS getter: `get myClass() { return condition ? 'slds-show' : 'slds-hide'; }` | Use `<aura:if>` or expression syntax `{!v.condition ? 'slds-show' : 'slds-hide'}` |
| Icon usage | `<lightning-icon icon-name="utility:close">` | `<lightning:icon iconName="utility:close">` |
| Scoping | Shadow DOM encapsulation (synthetic or native) | No shadow DOM — global CSS scope within namespace |
| Static resources | Not typically needed for SLDS | Can load SLDS as static resource for Visualforce embedding |
| Component naming | `kebab-case` in HTML: `<c-my-component>` | `camelCase` in markup: `<c:myComponent>` |

### LWC Token Usage Example

```css
/* myComponent.css */
.container {
  padding: var(--slds-g-spacing-4);
  color: var(--slds-g-color-brand-1);
  font-size: var(--slds-g-font-size-5);
  border-radius: var(--slds-g-radius-border-1);
  box-shadow: var(--slds-g-shadow-1);
}
```

### Aura Component Example

```html
<!-- myComponent.cmp -->
<aura:component>
  <lightning:card title="Account Details">
    <div class="slds-p-around_medium">
      <lightning:button label="Save" variant="brand" onclick="{!c.handleSave}" />
    </div>
  </lightning:card>
</aura:component>
```
