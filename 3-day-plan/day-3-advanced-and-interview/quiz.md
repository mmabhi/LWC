# 🧠 Day 3 Quiz — Advanced Topics & Interview Readiness

> **Instructions**: Answer all 10 multiple-choice questions. Each question has exactly one correct answer. Do NOT look at the answers section until you've completed all questions. Time yourself — aim for **15 minutes or less**.

---

### Question 1

What does `NavigationMixin.GenerateUrl()` return?

**A)** A URL string directly
**B)** A Promise that resolves to a URL string
**C)** A page reference object
**D)** Nothing — it navigates immediately like `Navigate()`

---

### Question 2

Which Lightning Data Service operation does **NOT** automatically update the LDS cache for other components?

**A)** `updateRecord()` from `lightning/uiRecordApi`
**B)** `createRecord()` from `lightning/uiRecordApi`
**C)** `deleteRecord()` from `lightning/uiRecordApi`
**D)** An imperative Apex method that performs DML

---

### Question 3

You need to display a record's fields in a completely custom read-only layout. Which base component should you use?

**A)** `lightning-record-form` with `mode="readonly"`
**B)** `lightning-record-view-form` with `lightning-output-field`
**C)** `lightning-record-edit-form` with all fields set to `disabled`
**D)** `lightning-record-form` with `mode="view"`

---

### Question 4

Which performance optimization removes elements from the DOM entirely instead of hiding them?

**A)** `style="display: none"`
**B)** `style="visibility: hidden"`
**C)** `lwc:if={condition}`
**D)** `class="slds-hide"`

---

### Question 5

In a Jest test for LWC, how do you wait for the DOM to re-render after changing a reactive property?

**A)** `await setTimeout(100)`
**B)** `return Promise.resolve().then(() => { /* assertions */ })`
**C)** `element.forceRender()`
**D)** `jest.runAllTimers()`

---

### Question 6

What is the primary architectural difference between Lightning Locker Service and Lightning Web Security (LWS)?

**A)** Locker uses browser-native sandboxing; LWS uses JavaScript proxies
**B)** Locker uses JavaScript proxy wrappers; LWS uses browser-native sandboxing
**C)** Locker supports LWC; LWS only supports Aura
**D)** There is no architectural difference — LWS is just a rename of Locker Service

---

### Question 7

In the Container-Presentational pattern, which statement is TRUE about Presentational (Dumb) components?

**A)** They fetch data via `@wire` and pass it to container components
**B)** They receive data only through `@api` properties and emit events via `CustomEvent`
**C)** They manage application state and business logic
**D)** They can only be used once per page

---

### Question 8

How do you mock an imperative Apex method called `getAccounts` in a Jest test?

**A)** `jest.spyOn(global, 'getAccounts')`
**B)** `jest.mock('@salesforce/apex/AccountCtrl.getAccounts', () => ({ default: jest.fn() }), { virtual: true })`
**C)** `import { mockApex } from 'lwc'; mockApex(getAccounts)`
**D)** Imperative Apex methods cannot be mocked in Jest tests

---

### Question 9

What page reference type should you use with `NavigationMixin` to navigate to a Contact list view?

**A)** `standard__listPage` with `objectApiName: 'Contact'`
**B)** `standard__objectPage` with `actionName: 'list'`
**C)** `standard__recordPage` with `actionName: 'list'`
**D)** `standard__namedPage` with `pageName: 'Contact_List'`

---

### Question 10

A component has this code. What happens when the user modifies a field and clicks Save?

```html
<lightning-record-edit-form object-api-name="Contact" record-id={recordId}
    onsubmit={handleSubmit} onsuccess={handleSuccess}>
    <lightning-input-field field-name="FirstName"></lightning-input-field>
    <lightning-input-field field-name="LastName"></lightning-input-field>
    <lightning-button type="submit" label="Save"></lightning-button>
</lightning-record-edit-form>
```

```javascript
handleSubmit(event) {
    event.preventDefault();
    const fields = event.detail.fields;
    fields.Description = 'Auto-generated';
    this.template.querySelector('lightning-record-edit-form').submit(fields);
}
```

**A)** The form submits normally with only FirstName and LastName
**B)** The form submits with FirstName, LastName, AND Description set to 'Auto-generated'
**C)** The form throws an error because `event.preventDefault()` cancels the submission permanently
**D)** The form submits twice — once normally and once with the modified fields

---

## 📊 Scoring Rubric

| Score | Rating | What It Means |
|---|---|---|
| **9-10** | 🏆 **Expert** | Outstanding! You have deep understanding of advanced LWC concepts. You're ready for technical interviews. |
| **7-8** | ✅ **Good** | Strong knowledge base. Review the 1-2 topics you missed and you'll be fully prepared. |
| **5-6** | 🟡 **Satisfactory** | Decent understanding but gaps remain. Re-read the Day 3 README sections for topics you missed. |
| **3-4** | 🟠 **Needs Improvement** | Several knowledge gaps. Go back through Day 3 material and practice the code examples hands-on. |
| **0-2** | 🔴 **Review Required** | Start from the Day 3 README beginning. Consider reviewing Day 1 and Day 2 fundamentals as well. |

---

<details>
<summary>📋 Click to reveal all answers</summary>

### Answer Key

| # | Answer | Explanation |
|---|---|---|
| **1** | **B** | `GenerateUrl()` returns a **Promise** that resolves to a URL string. Unlike `Navigate()` which performs immediate navigation, `GenerateUrl()` is async and gives you the URL to use in `<a href>` tags or other logic. |
| **2** | **D** | Imperative Apex DML bypasses the LDS cache entirely. LDS doesn't know the data changed, so other wired components don't auto-update. You must call `refreshApex()` manually. Options A, B, and C are all LDS operations that automatically sync the cache. |
| **3** | **B** | `lightning-record-view-form` with `lightning-output-field` gives you full control over the layout while keeping it read-only. Option A/D (`lightning-record-form`) auto-generates the layout — you can't customize field placement. Option C is an edit form — `disabled` fields aren't semantically read-only. |
| **4** | **C** | `lwc:if` completely removes elements from the DOM when the condition is false, freeing memory and reducing DOM size. CSS-based hiding (options A, B, D) keeps elements in the DOM — they still consume memory and participate in layout calculations. |
| **5** | **B** | LWC re-renders are scheduled as **microtasks**. `Promise.resolve().then()` waits for the microtask queue to flush, ensuring the DOM update has completed before assertions run. `setTimeout` would work but is fragile and slow. There is no `forceRender()` method. |
| **6** | **B** | Locker Service uses **JavaScript proxy wrappers** to intercept and control API access (slower, can break third-party libs). LWS uses **browser-native sandboxing** (ShadowRealms, API distortion) which is faster and more compatible. LWS is the modern successor. |
| **7** | **B** | Presentational ("Dumb") components are stateless renderers. They receive data exclusively through `@api` properties from their parent container and communicate actions back using `CustomEvent`. They never fetch data themselves — that's the container's job. |
| **8** | **B** | Imperative Apex is mocked using `jest.mock()` with the `{ virtual: true }` flag (because the module doesn't physically exist in the test environment). The mock returns `{ default: jest.fn() }`, and you can then use `.mockResolvedValue()` or `.mockRejectedValue()` in individual tests. |
| **9** | **B** | `standard__objectPage` with `actionName: 'list'` navigates to an object's list view. You can also pass `state: { filterName: 'Recent' }` to specify which list view. Options A and D use type names that don't exist. Option C (`standard__recordPage`) is for individual records, not lists. |
| **10** | **B** | The `handleSubmit` method: (1) prevents default submission with `event.preventDefault()`, (2) gets the form fields, (3) adds `Description = 'Auto-generated'`, (4) manually calls `.submit(fields)` with the modified field set. The form submits with all three fields. `preventDefault()` only stops the automatic submission — calling `.submit()` resumes it with your modifications. |

### Score Calculation

Count the number of correct answers above and refer to the Scoring Rubric.

**Your Score: ___ / 10**

</details>
