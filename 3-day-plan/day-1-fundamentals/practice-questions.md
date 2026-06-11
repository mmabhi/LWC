# 📝 Day 1 — Practice Questions: LWC Fundamentals

> **Instructions**: Try to answer each question before expanding the answer. Write your reasoning down — it helps solidify understanding.

---

## Multiple Choice Questions (1-8)

### Question 1: LWC vs Aura

What is the PRIMARY reason Salesforce introduced LWC to replace Aura components?

- A) Aura had too many security vulnerabilities
- B) LWC uses native web standards for better performance and developer experience
- C) Aura did not support mobile devices
- D) LWC supports two-way data binding while Aura only supported one-way

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

LWC was introduced because it leverages **native web standards** (Web Components, ES Modules, Shadow DOM). This provides:
- **Better performance**: Browser handles more natively, less framework overhead
- **Better developer experience**: Standard JavaScript skills transfer directly
- **Smaller bundle size**: Less framework abstraction code needed
- **Better tooling**: Standard tools like Jest work natively

Option A is incorrect — security was not the primary driver. Option C is incorrect — Aura supported mobile. Option D is backwards — Aura had two-way binding, LWC uses one-way by default.

</details>

---

### Question 2: Component Naming

You create a component folder named `accountSummary`. How would you reference this component in another component's HTML template?

- A) `<accountSummary></accountSummary>`
- B) `<c-accountSummary></c-accountSummary>`
- C) `<c-account-summary></c-account-summary>`
- D) `<lightning-account-summary></lightning-account-summary>`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

LWC uses a specific naming convention:
- **Folder name**: `camelCase` → `accountSummary`
- **Class name**: `PascalCase` → `AccountSummary`
- **HTML tag**: `kebab-case` with namespace prefix → `<c-account-summary>`

The `c-` prefix is the **default namespace** for custom components. The camelCase name is automatically converted to kebab-case. So `accountSummary` becomes `account-summary`, and with the prefix: `c-account-summary`.

</details>

---

### Question 3: Reactive Properties

Since Spring '20, which decorator is needed to make a simple string property reactive?

- A) `@track`
- B) `@api`
- C) `@reactive`
- D) None — plain class fields are reactive by default

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: D**

Since **Spring '20**, plain class fields (without any decorator) are **reactive by default**. When you reassign them (`this.name = 'new value'`), the template automatically re-renders.

```javascript
// This IS reactive — no decorator needed!
export default class MyComp extends LightningElement {
    name = 'World';  // Reactive for reassignment

    handleClick() {
        this.name = 'Salesforce'; // Template re-renders ✅
    }
}
```

`@track` is only needed for **deep object/array mutations** (like `this.obj.key = value`). `@api` is for public properties. `@reactive` doesn't exist in LWC.

</details>

---

### Question 4: Template Expressions

Which of the following is valid in an LWC template?

- A) `<p>{firstName + ' ' + lastName}</p>`
- B) `<p>{firstName} {lastName}</p>`
- C) `<p>{this.firstName}</p>`
- D) `<p>{{firstName}}</p>`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

LWC templates have strict rules:
- **No JavaScript expressions** allowed in templates (rules out A)
- Use **single curly braces** `{property}` for data binding (rules out D with double)
- **Never use `this`** in templates — just the property name (rules out C)
- You CAN place multiple `{property}` references next to each other (B is valid)

For computed values, use **getters** in the JavaScript class:
```javascript
get fullName() {
    return `${this.firstName} ${this.lastName}`;
}
```

</details>

---

### Question 5: Lifecycle Hooks

In which lifecycle hook should you add a window event listener?

- A) `constructor()`
- B) `connectedCallback()`
- C) `renderedCallback()`
- D) `render()`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

`connectedCallback()` is the correct place to add event listeners because:
- The component is **inserted into the DOM** at this point
- It fires **once** when the component is added (unlike `renderedCallback` which fires on every re-render)
- It has a matching cleanup hook: `disconnectedCallback()` where you should remove the listener

```javascript
connectedCallback() {
    window.addEventListener('resize', this.handleResize);
}

disconnectedCallback() {
    window.removeEventListener('resize', this.handleResize);
}
```

`constructor()` is too early (no DOM). `renderedCallback()` fires too often (every re-render). `render()` is for choosing which template to use, not for side effects.

</details>

---

### Question 6: Conditional Rendering

Which is the RECOMMENDED way to do conditional rendering in modern LWC?

- A) `if:true={condition}` and `if:false={condition}`
- B) `lwc:if={condition}`, `lwc:elseif={condition}`, `lwc:else`
- C) `ngIf={condition}` (Angular-style)
- D) Using CSS `display: none` to hide elements

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

`lwc:if`, `lwc:elseif`, and `lwc:else` are the **modern, recommended** directives (available since Spring '23). They provide:
- True `else-if` chaining (not possible with `if:true/if:false`)
- Better performance (grouped evaluation)
- Cleaner, more readable code

The old `if:true/if:false` (Option A) still works but is **deprecated**. Option C is Angular syntax, not LWC. Option D with CSS would keep elements in the DOM (bad for performance), and wouldn't prevent unnecessary processing.

</details>

---

### Question 7: Meta XML Configuration

You want your component to appear in the Lightning App Builder for record pages. Which configuration is required in the `.js-meta.xml` file?

- A) `<isVisible>true</isVisible>`
- B) `<isExposed>true</isExposed>` with `<target>lightning__RecordPage</target>`
- C) `<isPublic>true</isPublic>`
- D) No configuration needed — all components appear in App Builder

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

You need TWO things in the meta XML:
1. `<isExposed>true</isExposed>` — Makes the component available in tools like App Builder
2. `<target>lightning__RecordPage</target>` — Specifies WHERE the component can be used

```xml
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__RecordPage</target>
    </targets>
</LightningComponentBundle>
```

Without `isExposed`, the component won't appear in any builder. Without the correct target, it won't appear on the specific page type.

</details>

---

### Question 8: CSS Scoping

How do CSS styles behave in LWC by default?

- A) Styles are global and apply to all components
- B) Styles are scoped to the component using Shadow DOM and don't leak
- C) Styles only apply if you use the `scoped` attribute
- D) Parent component styles always cascade into child components

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

LWC uses **synthetic Shadow DOM** by default, which means:
- Styles defined in a component's `.css` file are **automatically scoped** to that component
- Styles do NOT leak up to parent or down to child components
- Parent styles do NOT cascade into child components
- The **only** way to pass styles through the shadow boundary is using **CSS custom properties** (CSS variables)

```css
/* This style ONLY applies to <p> tags inside THIS component */
p { color: blue; }
```

</details>

---

## Code Output Questions (9-12)

### Question 9: Property Reactivity

What happens when the user clicks the button?

```javascript
import { LightningElement } from 'lwc';

export default class ReactivityTest extends LightningElement {
    person = { name: 'John', age: 30 };

    handleClick() {
        this.person.name = 'Jane';
    }
}
```

```html
<template>
    <p>{person.name} is {person.age} years old</p>
    <lightning-button label="Change Name" onclick={handleClick}></lightning-button>
</template>
```

- A) The display updates to show "Jane is 30 years old"
- B) Nothing visible happens — the template does NOT re-render
- C) An error is thrown because `person` is not decorated with `@track`
- D) The component crashes with a read-only error

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

The template does **NOT** re-render because:
- `person` is a plain field (no decorator)
- Plain fields are reactive only for **reassignment** (`this.person = newValue`)
- Mutating a **nested property** (`this.person.name = 'Jane'`) does NOT trigger re-render without `@track`

**Fixes:**
```javascript
// Option 1: Use @track
@track person = { name: 'John', age: 30 };

// Option 2: Reassign the entire object (spread operator)
handleClick() {
    this.person = { ...this.person, name: 'Jane' };
}
```

The underlying value of `this.person.name` IS changed to 'Jane' in memory — the UI just doesn't know about it.

</details>

---

### Question 10: Lifecycle Order

What is the console output when this component is loaded on a page?

```javascript
import { LightningElement } from 'lwc';

export default class LifecycleOrder extends LightningElement {
    constructor() {
        super();
        console.log('A');
    }

    connectedCallback() {
        console.log('B');
    }

    renderedCallback() {
        console.log('C');
    }

    disconnectedCallback() {
        console.log('D');
    }
}
```

- A) A, B, C, D
- B) B, A, C
- C) A, B, C
- D) A, C, B

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

When a component is loaded (and not removed), the lifecycle is:
1. `constructor()` → prints "A"
2. `connectedCallback()` → prints "B"
3. `renderedCallback()` → prints "C"

`disconnectedCallback()` ("D") does NOT fire on load — it only fires when the component is **removed** from the DOM.

The order is always: constructor → connectedCallback → renderedCallback. The constructor creates the instance, connectedCallback fires when inserted into the DOM, and renderedCallback fires after the template is rendered.

</details>

---

### Question 11: for:each Key

What error will this code produce?

```html
<template>
    <template for:each={items} for:item="item">
        <div>{item.name}</div>
    </template>
</template>
```

```javascript
import { LightningElement } from 'lwc';

export default class ListDemo extends LightningElement {
    items = [
        { id: 1, name: 'Item A' },
        { id: 2, name: 'Item B' }
    ];
}
```

- A) No error — this works fine
- B) Compile error: Missing `key` attribute on the `<div>` element
- C) Runtime error: `items` is not iterable
- D) Compile error: Cannot use `item` as the iteration variable name

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

The `for:each` directive **requires** a `key` attribute on the **direct child element** of the template. Without it, you get a compile error.

**Fixed code:**
```html
<template for:each={items} for:item="item">
    <div key={item.id}>{item.name}</div>
</template>
```

The `key` must be a **unique identifier** (like `id`). It helps the LWC engine efficiently track which DOM elements correspond to which data items during re-renders, avoiding unnecessary DOM mutations.

</details>

---

### Question 12: Constructor Rules

What happens when this component loads?

```javascript
import { LightningElement } from 'lwc';

export default class ConstructorTest extends LightningElement {
    message;

    constructor() {
        this.message = 'Hello World';
    }
}
```

- A) Works fine — "Hello World" displays in the template
- B) Runtime error: Must call `super()` before accessing `this`
- C) Compile error: Cannot assign properties in constructor
- D) Works but `message` is undefined in the template

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

In LWC, the `constructor()` **must** call `super()` as its first statement before accessing `this`. This is a JavaScript requirement for classes that extend another class.

**Fixed code:**
```javascript
constructor() {
    super();  // MUST be first!
    this.message = 'Hello World';
}
```

Without `super()`, JavaScript throws a `ReferenceError` because `this` is not initialized until the parent class constructor runs. This is a standard ES6 class behavior, not specific to LWC.

</details>

---

## Short Answer Questions (13-15)

### Question 13: Decorator Choice

You have a component that receives a record ID from a parent component and needs to display record details. Which decorator should you use for the `recordId` property, and why?

<details>
<summary>✅ Answer & Explanation</summary>

**Use `@api`**

```javascript
import { LightningElement, api } from 'lwc';

export default class RecordDetail extends LightningElement {
    @api recordId;
}
```

**Why `@api`?**
- The `recordId` needs to be **publicly accessible** so the parent can pass it in
- `@api` creates a **public property** that can be set from outside the component
- It follows the parent-to-child data flow pattern
- On record pages, the Lightning runtime automatically passes `recordId` to components with an `@api recordId` property

**Why NOT the others?**
- `@track` — for internal deep object tracking, not for receiving external data
- `@wire` — for connecting to Salesforce data services, not for parent data
- No decorator — would make it private, parent can't set it

</details>

---

### Question 14: renderedCallback vs connectedCallback

Explain the key differences between `connectedCallback()` and `renderedCallback()`. When would you use each?

<details>
<summary>✅ Answer & Explanation</summary>

### Key Differences

| Aspect | `connectedCallback()` | `renderedCallback()` |
|---|---|---|
| **When it fires** | Once, when component inserted into DOM | After EVERY render/re-render |
| **DOM access** | ❌ Child elements not ready | ✅ Full DOM access |
| **Frequency** | Once (on insert) | Multiple times |
| **Cleanup pair** | `disconnectedCallback()` | None |

### When to Use Each

**Use `connectedCallback()` for:**
- Setting up event listeners
- Fetching initial data
- Subscribing to message channels
- One-time initialization that doesn't need DOM

**Use `renderedCallback()` for:**
- DOM manipulation (accessing rendered elements via `querySelector`)
- Initializing third-party libraries that need DOM elements (charts, maps)
- Measuring element sizes or positions

**Important:** Always use a guard flag in `renderedCallback()` to prevent infinite loops:
```javascript
renderedCallback() {
    if (this._hasInitialized) return;
    this._hasInitialized = true;
    // One-time DOM manipulation here
}
```

</details>

---

### Question 15: Style Penetration

A parent component wants to change the background color of a child component's internal `<div>`. The child uses Shadow DOM. How can you achieve this?

<details>
<summary>✅ Answer & Explanation</summary>

Use **CSS Custom Properties** (CSS Variables) — they are the only styling mechanism that can cross the Shadow DOM boundary.

**Child component (`childComp.css`):**
```css
.inner-div {
    background-color: var(--child-bg-color, #ffffff);
    /* Uses the variable if set, otherwise defaults to white */
}
```

**Parent component (`parentComp.css`):**
```css
c-child-comp {
    --child-bg-color: #f0f8ff;
}
```

**Why this works:**
- CSS custom properties are **inherited** through the DOM tree, including through Shadow DOM boundaries
- The child "declares" which properties are customizable by using `var(--name, fallback)`
- The parent sets the variable value on the child's host element

**What DOESN'T work:**
```css
/* ❌ Parent trying to target child's internal elements directly */
c-child-comp .inner-div {
    background-color: blue;  /* Shadow DOM blocks this */
}
```

This pattern is how Salesforce's base components allow theming — they expose CSS custom properties that developers can override.

</details>

---

## 📊 Score Yourself

| Score | Level |
|---|---|
| **13-15** | 🌟 Excellent — Ready for Day 2! |
| **10-12** | ✅ Good — Review the topics you missed |
| **7-9** | ⚠️ Fair — Re-read the sections for missed questions |
| **Below 7** | 🔄 Re-study — Go through Day 1 material again |

---

**Next → [Day 1 Quiz](./quiz.md)** | **Continue → [Day 2: Data & Events](../day-2-data-and-events/README.md)**
