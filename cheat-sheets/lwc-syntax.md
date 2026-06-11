# 🧩 LWC Syntax — Complete Cheat Sheet

> Everything you need to write Lightning Web Components, in one scannable reference.

---

## 📁 Component File Structure

Every LWC lives inside a folder. The folder name **is** the component name.

```
myComponent/
├── myComponent.html          ← Template (required for UI components)
├── myComponent.js            ← Controller class (required)
├── myComponent.js-meta.xml   ← Metadata config (required for deployment)
├── myComponent.css           ← Styles (optional)
├── myComponent.svg           ← Custom icon (optional)
└── __tests__/                ← Jest tests (optional)
    └── myComponent.test.js
```

> [!IMPORTANT]
> - Folder name must be **camelCase** starting with a **lowercase** letter
> - In HTML markup, the component is referenced in **kebab-case**: `<c-my-component>`
> - Max **one** HTML file, **one** CSS file per component (additional CSS via `@import` is not supported)

| Naming Rule | Example | Used In |
|-------------|---------|---------|
| camelCase (folder/file) | `myComponent` | File system, JS imports |
| kebab-case (markup) | `c-my-component` | HTML templates |
| Namespace prefix | `c` (default) or `lightning` | HTML markup |

---

## 📝 HTML Template Syntax

### Data Binding

```html
<template>
    <!-- One-way binding (read-only in template) -->
    <p>{greeting}</p>

    <!-- Property access -->
    <p>{contact.Name}</p>

    <!-- Getter method (computed values) -->
    <p>{fullName}</p>

    <!-- NOTE: No expressions in templates! Use getters instead -->
    <!-- ❌ WRONG: {firstName + ' ' + lastName} -->
    <!-- ✅ RIGHT: {fullName} → define a getter in JS -->
</template>
```

> [!WARNING]
> LWC templates do **NOT** support inline expressions. You cannot do math, string concatenation, or ternary operations in `{}`. Always use a JavaScript getter instead.

### Conditionals

#### `lwc:if` / `lwc:elseif` / `lwc:else` (Modern — Use This ✅)

```html
<template>
    <template lwc:if={isLoading}>
        <lightning-spinner alternative-text="Loading"></lightning-spinner>
    </template>
    <template lwc:elseif={hasError}>
        <p>Something went wrong: {errorMessage}</p>
    </template>
    <template lwc:else>
        <p>Data loaded successfully!</p>
    </template>
</template>
```

#### `if:true` / `if:false` (Legacy — Avoid ❌)

```html
<!-- Deprecated — still works but don't use in new code -->
<template if:true={isVisible}>
    <p>I am visible</p>
</template>
<template if:false={isVisible}>
    <p>I am hidden</p>
</template>
```

| Directive | Status | Supports else? | Supports elseif? |
|-----------|--------|----------------|-------------------|
| `lwc:if` / `lwc:elseif` / `lwc:else` | ✅ Current | Yes | Yes |
| `if:true` / `if:false` | ❌ Legacy | No (use separate `if:false`) | No |

### Iteration / Loops

#### `for:each`

```html
<template>
    <ul>
        <template for:each={contacts} for:item="contact" for:index="idx">
            <li key={contact.Id}>
                {idx}. {contact.Name} — {contact.Email}
            </li>
        </template>
    </ul>
</template>
```

#### `iterator`

```html
<template>
    <ul>
        <template iterator:it={contacts}>
            <li key={it.value.Id}>
                <template lwc:if={it.first}>
                    <strong>★ First Contact</strong>
                </template>
                {it.value.Name}
                <template lwc:if={it.last}>
                    <em>(end of list)</em>
                </template>
            </li>
        </template>
    </ul>
</template>
```

| Feature | `for:each` | `iterator` |
|---------|-----------|------------|
| Access current item | `for:item="name"` | `iteratorName.value` |
| Access index | `for:index="idx"` | Not built-in |
| First item check | ❌ | `iteratorName.first` |
| Last item check | ❌ | `iteratorName.last` |
| `key` required? | ✅ Yes | ✅ Yes |
| Use case | Simple lists | Need first/last styling |

> [!CAUTION]
> The `key` directive is **mandatory** on the first element inside any loop. It must be a unique string/number (not an index). Omitting it causes runtime errors.

### Special Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| `lwc:ref` | Query element without `querySelector` | `<div lwc:ref="myDiv">` → `this.refs.myDiv` |
| `lwc:spread` | Spread an object as attributes | `<c-child lwc:spread={props}>` |
| `lwc:is` | Dynamic component creation | `<lwc:component lwc:is={componentCtor}>` |
| `lwc:dom="manual"` | Allow manual DOM manipulation | `<div lwc:dom="manual"></div>` |
| `lwc:inner-html` | Render sanitized HTML | `<div lwc:inner-html={richText}></div>` |

---

## ⚙️ JavaScript Class Structure

```javascript
import { LightningElement, api, track, wire } from 'lwc';
import getContacts from '@salesforce/apex/ContactController.getContacts';

export default class MyComponent extends LightningElement {
    // ── Public Properties ──
    @api recordId;                    // Passed from parent or page context
    @api objectApiName;               // Auto-injected on record pages

    // ── Private Reactive Properties ──
    greeting = 'Hello World';         // Reactive by default (primitives)
    contacts = [];                    // Reactive (reassignment only)

    // ── Tracked Property (deep reactivity) ──
    @track complexObj = { a: { b: 1 } };  // Only needed for nested mutation

    // ── Wired Data ──
    @wire(getContacts, { accountId: '$recordId' })
    wiredContacts;                    // { data, error } shape

    // ── Getters (computed properties) ──
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    }

    get hasContacts() {
        return this.contacts && this.contacts.length > 0;
    }

    // ── Lifecycle Hooks ──
    constructor() { super(); }
    connectedCallback() { /* component inserted into DOM */ }
    renderedCallback() { /* after every render */ }
    disconnectedCallback() { /* component removed from DOM */ }
    errorCallback(error, stack) { /* child error boundary */ }

    // ── Event Handlers ──
    handleClick(event) {
        this.greeting = 'Clicked!';
    }

    // ── Public Methods ──
    @api
    refresh() {
        // Can be called by parent: this.template.querySelector('c-child').refresh()
    }
}
```

---

## 📦 Import Statements Catalog

### Core LWC

| Import | What It Gives You |
|--------|-------------------|
| `import { LightningElement } from 'lwc'` | Base class (always required) |
| `import { api } from 'lwc'` | `@api` decorator for public props/methods |
| `import { track } from 'lwc'` | `@track` decorator for deep reactivity |
| `import { wire } from 'lwc'` | `@wire` decorator for declarative data |

### Salesforce Modules

| Import | Purpose |
|--------|---------|
| `import { getRecord, getFieldValue } from 'lightning/uiRecordApi'` | Read record data |
| `import { createRecord, updateRecord, deleteRecord } from 'lightning/uiRecordApi'` | DML operations |
| `import { getObjectInfo } from 'lightning/uiObjectInfoApi'` | Object metadata |
| `import { getPicklistValues, getPicklistValuesByRecordType } from 'lightning/uiObjectInfoApi'` | Picklist values |
| `import { NavigationMixin } from 'lightning/navigation'` | Programmatic navigation |
| `import { CurrentPageReference } from 'lightning/navigation'` | Get current page info |
| `import { ShowToastEvent } from 'lightning/platformShowToastEvent'` | Toast notifications |
| `import { publish, subscribe, unsubscribe, MessageContext, createMessageContext, releaseMessageContext } from 'lightning/messageService'` | Lightning Message Service |
| `import { loadScript, loadStyle } from 'lightning/platformResourceLoader'` | Load static resources |
| `import { getRelatedListRecords } from 'lightning/uiRelatedListApi'` | Related list data |
| `import { refreshApex } from '@salesforce/apex'` | Refresh wired Apex cache |
| `import { notifyRecordUpdateAvailable } from 'lightning/uiRecordApi'` | Notify LDS of data change |

### Salesforce Schema & Context

| Import | Purpose | Example |
|--------|---------|---------|
| `import FIELD from '@salesforce/schema/Object.Field'` | Field reference | `import NAME from '@salesforce/schema/Account.Name'` |
| `import OBJECT from '@salesforce/schema/Object'` | Object reference | `import ACCOUNT from '@salesforce/schema/Account'` |
| `import Id from '@salesforce/user/Id'` | Current user ID | |
| `import isGuest from '@salesforce/user/isGuest'` | Is guest user? | |
| `import LOCALE from '@salesforce/i18n/locale'` | User locale | |
| `import CURRENCY from '@salesforce/i18n/currency'` | User currency | |
| `import DIR from '@salesforce/i18n/dir'` | Text direction (LTR/RTL) | |
| `import LABEL from '@salesforce/label/c.MyLabel'` | Custom label | |
| `import RESOURCE from '@salesforce/resourceUrl/myResource'` | Static resource URL | |
| `import hasPermission from '@salesforce/userPermission/PermName'` | User permission check | |
| `import hasCustomPerm from '@salesforce/customPermission/PermName'` | Custom permission check | |
| `import FORM_FACTOR from '@salesforce/client/formFactor'` | Device type (Large/Medium/Small) | |

### Apex Methods

```javascript
// Named import (most common)
import getAccounts from '@salesforce/apex/AccountController.getAccounts';

// You can rename during import
import fetchData from '@salesforce/apex/MyController.getData';
```

---

## 🏷️ Decorators Quick Reference

| Decorator | Purpose | Reactive? | Key Rule |
|-----------|---------|-----------|----------|
| `@api` | Public property/method | Yes (parent → child) | Read-only inside the component |
| `@track` | Deep object/array tracking | Yes (nested changes) | Only needed for mutating nested properties |
| `@wire` | Declarative data fetching | Yes (auto-provisions) | Cannot be combined with `@api` |

```javascript
// @api — public property
@api title = 'Default';         // Parent can set: <c-child title="Hello">

// @api — public method
@api focus() { /* parent calls this.template.querySelector('c-child').focus() */ }

// @track — deep tracking (only when mutating nested objects)
@track config = { theme: { color: 'blue' } };
updateColor() { this.config.theme.color = 'red'; }  // UI re-renders

// @wire — declarative data
@wire(getRecord, { recordId: '$recordId', fields: [NAME_FIELD] })
account;  // this.account.data / this.account.error
```

---

## 🎯 Event Handling

### Dispatching Custom Events

```javascript
// Simple event (no data)
this.dispatchEvent(new CustomEvent('save'));

// Event with data
this.dispatchEvent(new CustomEvent('select', {
    detail: { recordId: this.selectedId },
    bubbles: false,       // default: false
    composed: false       // default: false — stays within shadow DOM
}));

// Event that crosses shadow DOM boundaries
this.dispatchEvent(new CustomEvent('notify', {
    detail: { message: 'Hello' },
    bubbles: true,
    composed: true
}));
```

### Handling Events

```html
<!-- In parent template -->
<c-child onselect={handleSelect}></c-child>

<!-- Standard DOM events -->
<lightning-button label="Click Me" onclick={handleClick}></lightning-button>
<lightning-input onchange={handleChange}></lightning-input>
```

```javascript
// In parent JS
handleSelect(event) {
    const recordId = event.detail.recordId;
    console.log('Selected:', recordId);
}
```

| Property | Default | When `true` |
|----------|---------|-------------|
| `bubbles` | `false` | Event bubbles up through the DOM tree |
| `composed` | `false` | Event crosses shadow DOM boundaries |
| `cancelable` | `false` | Event can be canceled with `preventDefault()` |

> [!TIP]
> **Naming convention**: Custom event names must be **lowercase**, **no underscores**, **no uppercase**. Use `itemselect` not `itemSelect` or `item_select`. In HTML, prefix with `on`: `onitemselect`.

---

## 🎨 CSS in LWC

### Scoped Styling (Default)

```css
/* myComponent.css — automatically scoped to this component */
:host {
    display: block;
    padding: 16px;
}

/* Style the host based on context */
:host(.active) {
    border: 2px solid blue;
}

/* Standard selectors — scoped automatically */
h1 {
    font-size: 1.5rem;
    color: var(--lwc-colorTextDefault, #181818);
}

/* You CANNOT style child component internals (shadow DOM encapsulation) */
```

### CSS Custom Properties (Design Tokens)

```css
/* Use SLDS design tokens via CSS custom properties */
.card {
    background: var(--lwc-colorBackground, #fff);
    border: 1px solid var(--lwc-colorBorder, #e5e5e5);
    border-radius: var(--lwc-borderRadiusMedium, 0.25rem);
    box-shadow: var(--lwc-shadowCard, 0 2px 2px 0 rgba(0, 0, 0, 0.1));
    font-family: var(--lwc-fontFamily, 'Salesforce Sans', sans-serif);
}
```

### Common SLDS Tokens

| Token | Purpose | Fallback |
|-------|---------|----------|
| `--lwc-colorTextDefault` | Default text color | `#181818` |
| `--lwc-colorBackground` | Page background | `#ffffff` |
| `--lwc-colorBorder` | Default border | `#e5e5e5` |
| `--lwc-brandPrimary` | Brand primary color | `#1B96FF` |
| `--lwc-fontSize3` | Body text size | `0.8125rem` |
| `--lwc-spacingMedium` | Medium spacing | `1rem` |

### Sharing Styles

```javascript
// In sharedStyles.css (a separate LWC component with ONLY a CSS file)
// Then import in your component:
import { LightningElement } from 'lwc';
import sharedStyles from 'c/sharedStyles';  // not actually how it works

// Correct approach: use a common CSS module
// Create: cssLibrary/cssLibrary.css
// Reference in your component's CSS:
/* @import is NOT supported. Use SLDS utility classes or a shared component */
```

> [!NOTE]
> LWC CSS does **not** support `@import`. To share styles, create a shared component with CSS and compose it, or use SLDS utility classes.

---

## 📄 XML Configuration

### Full Metadata Template

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <masterLabel>My Amazing Component</masterLabel>
    <description>A reusable component for displaying contact info</description>
    <targets>
        <target>lightning__RecordPage</target>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
        <target>lightning__FlowScreen</target>
        <target>lightning__Tab</target>
        <target>lightning__Inbox</target>
        <target>lightning__UtilityBar</target>
        <target>lightningCommunity__Page</target>
        <target>lightningCommunity__Default</target>
        <target>lightning__RecordAction</target>
        <target>lightningSnapin__ChatMessage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordPage">
            <property name="title" type="String" label="Card Title" default="Contacts"
                      description="Title displayed at the top" />
            <property name="maxRecords" type="Integer" label="Max Records" default="5"
                      min="1" max="50" />
            <property name="showHeader" type="Boolean" label="Show Header" default="true" />
            <property name="variant" type="String" label="Variant" datasource="compact,full,minimal" />
            <objects>
                <object>Account</object>
                <object>Contact</object>
            </objects>
            <supportedFormFactors>
                <supportedFormFactor type="Large" />
                <supportedFormFactor type="Small" />
            </supportedFormFactors>
        </targetConfig>
        <targetConfig targets="lightning__FlowScreen">
            <property name="inputValue" type="String" label="Input" role="inputOnly" />
            <property name="outputValue" type="String" label="Output" role="outputOnly" />
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

### Target Reference

| Target | Where Component Can Be Placed |
|--------|-------------------------------|
| `lightning__RecordPage` | Record detail pages |
| `lightning__AppPage` | App Builder custom pages |
| `lightning__HomePage` | Home page |
| `lightning__FlowScreen` | Screen flows |
| `lightning__Tab` | Custom tab |
| `lightning__UtilityBar` | Utility bar |
| `lightning__Inbox` | Outlook/Gmail integration |
| `lightning__RecordAction` | Quick actions |
| `lightningCommunity__Page` | Experience Cloud pages |
| `lightningCommunity__Default` | Experience Cloud (all pages) |
| `lightningSnapin__ChatMessage` | Embedded chat |

### Property Types

| Type | Description | Additional Attributes |
|------|-------------|----------------------|
| `String` | Text value | `default`, `placeholder`, `datasource` |
| `Integer` | Whole number | `default`, `min`, `max` |
| `Boolean` | True/false | `default` |
| `Color` | Color picker | `default` |
| `Date` | Date picker | `default` |
| `DateTime` | Date/time picker | `default` |

### Flow Property Roles

| Role | Direction | Usage |
|------|-----------|-------|
| `inputOnly` | Flow → Component | Component receives data from flow |
| `outputOnly` | Component → Flow | Component sends data back to flow |
| *(no role)* | Bidirectional | Both input and output |

---

## 🔧 Common Patterns

### Toast Message

```javascript
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

// Success toast
this.dispatchEvent(new ShowToastEvent({
    title: 'Success!',
    message: 'Record saved successfully',
    variant: 'success'    // success | error | warning | info
}));

// Error toast with field errors
this.dispatchEvent(new ShowToastEvent({
    title: 'Error',
    message: 'Could not save the record',
    variant: 'error',
    mode: 'sticky'        // dismissible | sticky | pester
}));
```

| Variant | Color | Icon | Use For |
|---------|-------|------|---------|
| `success` | Green | ✅ | Successful operations |
| `error` | Red | ❌ | Failures, exceptions |
| `warning` | Yellow | ⚠️ | Warnings, partial success |
| `info` | Blue | ℹ️ | Informational messages |

### Programmatic Navigation

```javascript
import { NavigationMixin } from 'lightning/navigation';

export default class MyNav extends NavigationMixin(LightningElement) {

    // Navigate to record page
    goToRecord() {
        this[NavigationMixin.Navigate]({
            type: 'standard__recordPage',
            attributes: {
                recordId: this.recordId,
                objectApiName: 'Account',
                actionName: 'view'     // view | edit | clone
            }
        });
    }

    // Navigate to list view
    goToList() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Contact',
                actionName: 'list'
            },
            state: {
                filterName: 'Recent'
            }
        });
    }

    // Navigate to custom tab / web page
    goToWeb() {
        this[NavigationMixin.Navigate]({
            type: 'standard__webPage',
            attributes: {
                url: 'https://www.salesforce.com'
            }
        });
    }

    // Generate a URL (returns Promise)
    async getRecordUrl() {
        const url = await this[NavigationMixin.GenerateUrl]({
            type: 'standard__recordPage',
            attributes: {
                recordId: this.recordId,
                actionName: 'view'
            }
        });
        return url;
    }
}
```

### Loading Spinner

```html
<template>
    <lightning-card title="My Card">
        <template lwc:if={isLoading}>
            <div class="slds-is-relative slds-m-around_large">
                <lightning-spinner
                    alternative-text="Loading"
                    size="medium"
                    variant="brand">
                </lightning-spinner>
            </div>
        </template>
        <template lwc:else>
            <!-- Content here -->
        </template>
    </lightning-card>
</template>
```

### Modal Dialog

```html
<template>
    <template lwc:if={showModal}>
        <section role="dialog" tabindex="-1" class="slds-modal slds-fade-in-open">
            <div class="slds-modal__container">
                <button class="slds-button slds-modal__close" onclick={closeModal}>
                    <lightning-icon icon-name="utility:close" size="small"></lightning-icon>
                </button>
                <div class="slds-modal__header">
                    <h1 class="slds-modal__title">Modal Title</h1>
                </div>
                <div class="slds-modal__content slds-p-around_medium">
                    <p>Modal body content</p>
                </div>
                <div class="slds-modal__footer">
                    <lightning-button label="Cancel" onclick={closeModal}></lightning-button>
                    <lightning-button label="Save" variant="brand" onclick={handleSave}></lightning-button>
                </div>
            </div>
        </section>
        <div class="slds-backdrop slds-backdrop_open"></div>
    </template>
</template>
```

> [!TIP]
> Prefer `<lightning-modal>` (GA since Winter '24) over hand-rolling SLDS modal markup. It handles focus trapping, accessibility, and backdrop automatically.

### Imperative Apex Call

```javascript
import getContacts from '@salesforce/apex/ContactController.getContacts';

async handleLoad() {
    this.isLoading = true;
    try {
        this.contacts = await getContacts({ accountId: this.recordId });
        this.error = undefined;
    } catch (error) {
        this.error = error.body?.message || 'Unknown error';
        this.contacts = [];
    } finally {
        this.isLoading = false;
    }
}
```

---

## 🎨 SLDS Utility Classes Quick Reference

| Category | Class | Effect |
|----------|-------|--------|
| **Margin** | `slds-m-around_small` | Margin all sides (small) |
| | `slds-m-top_medium` | Margin top (medium) |
| | `slds-m-bottom_large` | Margin bottom (large) |
| | `slds-m-left_x-small` | Margin left (x-small) |
| **Padding** | `slds-p-around_medium` | Padding all sides (medium) |
| | `slds-p-horizontal_small` | Padding left + right |
| | `slds-p-vertical_large` | Padding top + bottom |
| **Text** | `slds-text-heading_small` | Small heading |
| | `slds-text-heading_medium` | Medium heading |
| | `slds-text-heading_large` | Large heading |
| | `slds-text-align_center` | Center text |
| | `slds-text-color_error` | Error (red) text |
| | `slds-text-color_success` | Success (green) text |
| | `slds-truncate` | Truncate with ellipsis |
| **Layout** | `slds-grid` | Flex container |
| | `slds-col` | Flex child |
| | `slds-size_1-of-2` | Half width |
| | `slds-size_1-of-3` | Third width |
| | `slds-wrap` | Allow flex wrapping |
| | `slds-grid_align-center` | Center items horizontally |
| | `slds-grid_vertical-align-center` | Center items vertically |
| **Visibility** | `slds-hide` | Hide element (`display: none`) |
| | `slds-show` | Show element |
| | `slds-is-relative` | `position: relative` |
| | `slds-is-absolute` | `position: absolute` |
| **Sizing** | `slds-size_full` | 100% width |
| | `slds-size_xx-small` | Specific size |
| **Borders** | `slds-border_top` | Top border |
| | `slds-border_bottom` | Bottom border |
| | `slds-box` | Box with border + padding |

### SLDS Grid Example

```html
<div class="slds-grid slds-wrap slds-gutters">
    <div class="slds-col slds-size_1-of-2 slds-medium-size_1-of-3">
        Column 1
    </div>
    <div class="slds-col slds-size_1-of-2 slds-medium-size_1-of-3">
        Column 2
    </div>
    <div class="slds-col slds-size_1-of-1 slds-medium-size_1-of-3">
        Column 3
    </div>
</div>
```

---

## 🔑 Key Takeaways

| # | Takeaway |
|---|----------|
| 1 | Component folder name = camelCase; HTML reference = kebab-case with `c-` prefix |
| 2 | No inline expressions in templates — always use JS getters |
| 3 | Use `lwc:if`/`lwc:elseif`/`lwc:else` instead of legacy `if:true`/`if:false` |
| 4 | `key` is mandatory in all loops |
| 5 | Custom event names must be all lowercase, no underscores |
| 6 | CSS is scoped by default via shadow DOM — use `:host` for the component itself |
| 7 | `isExposed` must be `true` in XML for the component to appear in App Builder |
| 8 | Use `@salesforce/schema/` imports for compile-time field validation |
| 9 | `NavigationMixin` requires extending the class: `NavigationMixin(LightningElement)` |
| 10 | Prefer `lightning-*` base components over raw SLDS markup |

---

*Quick reference — keep this handy during development! 🚀*
