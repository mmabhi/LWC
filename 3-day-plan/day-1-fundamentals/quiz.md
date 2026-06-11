# 🧪 Day 1 — Quiz: LWC Fundamentals

> **Time Limit**: 15 minutes
> **Format**: 10 multiple-choice questions
> **Passing Score**: 7/10

> [!IMPORTANT]
> Complete this quiz WITHOUT looking at the study material. This tests your retention. Write down your answers first, then check.

---

### Q1. Which web standards does LWC build upon?

- A) React Virtual DOM and JSX
- B) Web Components, ES Modules, and Shadow DOM
- C) Angular Modules and TypeScript
- D) jQuery and AJAX

---

### Q2. What is the correct file that controls where a component can be placed in the Lightning App Builder?

- A) `component.config.js`
- B) `component.js-meta.xml`
- C) `component.manifest.json`
- D) `component.settings.xml`

---

### Q3. A component folder is named `userProfile`. What is the correct HTML tag to use it in another component?

- A) `<c-userProfile>`
- B) `<c-user-profile>`
- C) `<c:userProfile>`
- D) `<lwc-user-profile>`

---

### Q4. Which statement about `@track` is TRUE?

- A) `@track` is required for ALL reactive properties
- B) `@track` is needed only for deep object/array mutation tracking
- C) `@track` makes a property public
- D) `@track` connects a property to Salesforce data

---

### Q5. What is the correct order of lifecycle hooks when a component first loads?

- A) `connectedCallback` → `constructor` → `renderedCallback`
- B) `constructor` → `renderedCallback` → `connectedCallback`
- C) `constructor` → `connectedCallback` → `renderedCallback`
- D) `renderedCallback` → `constructor` → `connectedCallback`

---

### Q6. Which lifecycle hook catches errors thrown by child components?

- A) `catchCallback()`
- B) `errorCallback()`
- C) `exceptionCallback()`
- D) `onError()`

---

### Q7. What is WRONG with this template code?

```html
<template>
    <p>Total: {price * quantity}</p>
</template>
```

- A) Missing the `$` sign before curly braces
- B) JavaScript expressions are not allowed in LWC templates
- C) Must use double curly braces `{{ }}`
- D) Nothing — this code is valid

---

### Q8. You want to conditionally show an error message only when there's an error AND the user has submitted a form. Which approach is correct?

- A) `<template lwc:if={hasError && formSubmitted}>` 
- B) Use a getter that returns `this.hasError && this.formSubmitted` and bind to it
- C) `<template if:true={hasError} if:true={formSubmitted}>`
- D) Use inline JavaScript: `<template lwc:if={() => hasError && formSubmitted}>`

---

### Q9. Which statement about `for:each` loops is TRUE?

- A) The `key` attribute is optional but recommended
- B) The `key` attribute is required and must be on the direct child element
- C) The `key` attribute should be the array index
- D) The `key` attribute goes on the `<template>` tag

---

### Q10. How can a parent component's CSS affect a child component's internal styles?

- A) Using `!important` declarations
- B) Using CSS custom properties (CSS variables)
- C) Using `::shadow` pseudo-element
- D) Using `>>>` deep combinator

---

## ✅ Answer Key

<details>
<summary>Click to reveal answers</summary>

### Answers

| Question | Answer | Key Concept |
|---|---|---|
| Q1 | **B** | LWC is built on Web Components, ES Modules, and Shadow DOM |
| Q2 | **B** | `.js-meta.xml` controls visibility, targets, and design properties |
| Q3 | **B** | camelCase folder → kebab-case tag with `c-` prefix |
| Q4 | **B** | Since Spring '20, plain fields are reactive; `@track` is only for deep mutations |
| Q5 | **C** | constructor → connectedCallback → renderedCallback |
| Q6 | **B** | `errorCallback(error, stack)` catches child component errors |
| Q7 | **B** | No JavaScript expressions in templates — use getters |
| Q8 | **B** | Use a getter for combined conditions — no expressions in templates |
| Q9 | **B** | `key` is required on the direct child element for efficient DOM diffing |
| Q10 | **B** | CSS custom properties are the only way to pierce Shadow DOM |

### Detailed Explanations

**Q1**: LWC leverages native browser capabilities — Web Components for encapsulation, ES Modules for module system, and Shadow DOM for style isolation. Unlike React or Angular, it doesn't add a heavy abstraction layer.

**Q2**: The `.js-meta.xml` file is unique to LWC/Salesforce. It defines `isExposed`, `targets` (where the component can be used), and `targetConfigs` (design-time properties). Without this file properly configured, your component won't appear in App Builder.

**Q3**: The naming convention is: folder `camelCase` (userProfile) → HTML tag `kebab-case` with namespace (c-user-profile). The `c` is the default namespace for custom components.

**Q4**: Before Spring '20, `@track` was needed for ALL reactive properties. After Spring '20, plain class fields became reactive by default for reassignment. Now `@track` is ONLY needed when you want to mutate a nested property of an object or element of an array and have the UI react to it.

**Q5**: The lifecycle always starts with `constructor()` (object creation), then `connectedCallback()` (inserted into DOM), then `renderedCallback()` (after template renders). This order never changes.

**Q6**: `errorCallback(error, stack)` acts like a try/catch boundary in the component tree. It catches errors from descendant/child components, NOT from the component itself. It receives the error object and the stack trace.

**Q7**: LWC templates intentionally do NOT support JavaScript expressions. This is a design decision for performance (faster template parsing) and security. Use getters: `get total() { return this.price * this.quantity; }`

**Q8**: Since you can't use `&&` or any expression in templates, you must create a getter:
```javascript
get showError() {
    return this.hasError && this.formSubmitted;
}
```
Then use `<template lwc:if={showError}>`.

**Q9**: The `key` attribute is mandatory on `for:each` loops. It must be placed on the direct child element (not the `<template>` tag), and it should be a unique, stable identifier (like a record ID), not an array index.

**Q10**: Shadow DOM prevents parent styles from reaching into child components. The only exception is CSS custom properties (variables), which inherit through the shadow boundary. The `::shadow` and `>>>` combinators were deprecated from the web platform.

</details>

---

## 📊 Scoring Rubric

| Score | Rating | Next Steps |
|---|---|---|
| **10/10** | 🏆 Perfect | You've mastered Day 1! Move to Day 2 with confidence. |
| **8-9/10** | 🌟 Excellent | Strong understanding. Quick review of missed topics, then Day 2. |
| **7/10** | ✅ Passing | Good foundation. Review missed concepts before starting Day 2. |
| **5-6/10** | ⚠️ Needs Work | Re-read the sections for topics you missed. Retake quiz tomorrow. |
| **Below 5** | 🔄 Review Day 1 | Go through the Day 1 material again before proceeding. |

---

**Practice More → [Day 1 Practice Questions](./practice-questions.md)** | **Next → [Day 2: Data & Events](../day-2-data-and-events/README.md)**
