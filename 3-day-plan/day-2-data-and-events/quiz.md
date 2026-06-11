# 🧪 Day 2 — Quiz: Data & Events

> **Time Limit**: 15 minutes
> **Format**: 10 multiple-choice questions
> **Passing Score**: 7/10

> [!IMPORTANT]
> Complete this quiz WITHOUT looking at the study material. Write down your answers first, then check.

---

### Q1. What is the purpose of the `$` prefix in a wire parameter like `'$recordId'`?

- A) It indicates the parameter is a currency value
- B) It makes the parameter reactive — wire re-fires when it changes
- C) It references a global variable
- D) It marks the parameter as required

---

### Q2. Which Apex annotation allows a method to be used with `@wire`?

- A) `@AuraEnabled`
- B) `@AuraEnabled(cacheable=true)`
- C) `@RemoteAction`
- D) `@WireEnabled`

---

### Q3. In LWC, how does a child component send data to its parent?

- A) By modifying an `@api` property
- B) By dispatching a `CustomEvent` with data in the `detail` property
- C) By calling `this.parent.handleData()`
- D) By using `@wire` to push data up

---

### Q4. What does `refreshApex()` do?

- A) Refreshes the entire page
- B) Forces a re-render of the component template
- C) Invalidates the LDS cache and re-fetches wired data
- D) Restarts the LWC engine

---

### Q5. Which Lightning Message Service scope allows a component to receive messages from ANY publisher on the page?

- A) `COMPONENT_SCOPE`
- B) `PAGE_SCOPE`
- C) `APPLICATION_SCOPE`
- D) `GLOBAL_SCOPE`

---

### Q6. You want to create a quick edit form with auto-layout for a Contact record. Which component is BEST?

- A) `lightning-record-edit-form`
- B) `lightning-record-form` with `mode="edit"`
- C) `lightning-record-view-form`
- D) `lightning-input` with custom Apex

---

### Q7. When wiring to a function, what parameters does the function receive?

- A) `(data, error)` as separate parameters
- B) A single object with `{ data, error }` properties
- C) Just `data` — error is handled by the framework
- D) `(result)` where `result` is the raw JSON string

---

### Q8. What is the correct HTML attribute to listen for a custom event named `contactselected`?

- A) `on-contact-selected={handler}`
- B) `onContactSelected={handler}`
- C) `oncontactselected={handler}`
- D) `handle-contactselected={handler}`

---

### Q9. Which is TRUE about `lightning-record-edit-form` vs `lightning-record-form`?

- A) `lightning-record-form` offers more layout control
- B) `lightning-record-edit-form` uses `lightning-input-field` for custom layouts
- C) `lightning-record-form` requires you to add a submit button
- D) `lightning-record-edit-form` auto-generates the form layout

---

### Q10. What is the correct way to call an Apex method that performs a DML operation?

- A) Use `@wire` with the Apex method
- B) Call the method imperatively using `async/await` or `.then()`
- C) Use `lightning-record-form` with the Apex class name
- D) Use `refreshApex()` with the Apex method

---

## ✅ Answer Key

<details>
<summary>Click to reveal answers</summary>

### Answers

| Question | Answer | Key Concept |
|---|---|---|
| Q1 | **B** | `$` prefix makes wire parameters reactive |
| Q2 | **B** | `@AuraEnabled(cacheable=true)` required for `@wire` |
| Q3 | **B** | Child→Parent: `CustomEvent` with `dispatchEvent()` |
| Q4 | **C** | `refreshApex()` invalidates cache and re-fetches |
| Q5 | **C** | `APPLICATION_SCOPE` for page-wide LMS |
| Q6 | **B** | `lightning-record-form` with `mode="edit"` for quick auto-layout |
| Q7 | **B** | Wire function receives `{ data, error }` object |
| Q8 | **C** | Event handler: `on` + lowercase event name |
| Q9 | **B** | `-edit-form` uses `lightning-input-field` for custom layouts |
| Q10 | **B** | DML requires imperative calls (no `@wire` for DML) |

### Detailed Explanations

**Q1**: The `$` prefix creates a reactive binding. When `this.recordId` changes, the wire service detects the change and automatically re-provisions data. Without `$`, the parameter is treated as a static string literal.

**Q2**: `@AuraEnabled(cacheable=true)` enables three things: (1) the method can be used with `@wire`, (2) the response is cached client-side, (3) the method cannot perform DML. Plain `@AuraEnabled` only enables imperative calls.

**Q3**: Child→Parent communication uses standard DOM `CustomEvent`. The child creates a `new CustomEvent('eventname', { detail: data })` and dispatches it with `this.dispatchEvent()`. The parent listens with `oneventname={handler}`. You cannot modify `@api` properties to send data up.

**Q4**: `refreshApex()` invalidates the client-side cache for a specific wired result and forces a fresh server call. It's essential after DML operations (create/update/delete) to ensure the displayed data reflects the latest state. It requires the full wired result object, not just the data.

**Q5**: `APPLICATION_SCOPE` is imported from `lightning/messageService` and passed as an option to `subscribe()`. Without it, messages are only received from publishers within the same DOM subtree. With it, the subscriber receives messages from any publisher on the entire Lightning page.

**Q6**: `lightning-record-form` with `mode="edit"` provides a quick, auto-generated form layout with built-in save/cancel buttons. `lightning-record-edit-form` gives more control but requires more code. For "quick with auto-layout," `lightning-record-form` is the best choice.

**Q7**: When wiring to a function, the function receives a single object with `{ data, error }` properties. You destructure it: `wiredMethod({ data, error }) { ... }`. Both properties exist but only one will have a value at a time.

**Q8**: The HTML attribute for listening to events follows the pattern: `on` + event name (all lowercase). So for event `contactselected`, the attribute is `oncontactselected`. No hyphens, no camelCase in the attribute name.

**Q9**: `lightning-record-edit-form` uses `lightning-input-field` components that you position however you want, giving you full layout control. `lightning-record-form` has an automatic layout that you can't customize. `-edit-form` requires you to add your own submit button.

**Q10**: DML operations (insert, update, delete) require `@AuraEnabled` WITHOUT `cacheable=true`, and they must be called **imperatively** using `async/await` or `.then()/.catch()`. The `@wire` service only works with `cacheable=true` methods, which cannot perform DML.

</details>

---

## 📊 Scoring Rubric

| Score | Rating | Next Steps |
|---|---|---|
| **10/10** | 🏆 Perfect | Data & Events mastered! Move to Day 3. |
| **8-9/10** | 🌟 Excellent | Strong understanding. Quick review, then Day 3. |
| **7/10** | ✅ Passing | Good foundation. Review missed concepts. |
| **5-6/10** | ⚠️ Needs Work | Re-read Day 2 sections for topics you missed. |
| **Below 5** | 🔄 Review Day 2 | Go through the Day 2 material again. |

---

**Practice More → [Day 2 Practice Questions](./practice-questions.md)** | **Next → [Day 3: Advanced & Interview](../day-3-advanced-and-interview/README.md)**
