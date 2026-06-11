# 📝 Day 2 — Practice Questions: Data & Events

> **Instructions**: Answer each question before expanding the solution. For code questions, try to predict the output or identify the error.

---

## Multiple Choice Questions (1-8)

### Question 1: Wire Service Basics

Which statement about the LWC Wire Service is TRUE?

- A) Wire calls are made only once when the component loads
- B) Wire calls are reactive — they re-fire when reactive parameters change
- C) Wire calls can perform DML operations (insert, update, delete)
- D) Wire calls are made imperatively using `await`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

The Wire Service is **reactive**. It automatically re-fires when:
1. The component first loads
2. Any reactive parameter (prefixed with `$`) changes
3. Cached data is updated by another component

Wire calls are **declarative** (not imperative) and **read-only** (no DML). The `@wire` decorator automatically handles the data provisioning lifecycle.

```javascript
// '$accountId' is reactive — when accountId changes, wire re-fires
@wire(getRecord, { recordId: '$accountId', fields: FIELDS })
record;
```

</details>

---

### Question 2: @AuraEnabled Annotation

What happens if you try to use `@wire` with an Apex method that has `@AuraEnabled` (without `cacheable=true`)?

- A) It works fine — caching is optional
- B) It compiles but throws a runtime error
- C) The wire call is not reactive
- D) It compiles but always returns stale data

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

An Apex method must have `@AuraEnabled(cacheable=true)` to be used with `@wire`. Without `cacheable=true`, you'll get a **runtime error** because the Wire Service requires cacheable methods for its caching and reactive pipeline.

```java
// ✅ Works with @wire AND imperative calls
@AuraEnabled(cacheable=true)
public static List<Contact> getContacts() { ... }

// ❌ Only works with imperative calls (NOT @wire)
@AuraEnabled
public static Contact createContact(String name) { ... }
```

**Rule of thumb:**
- `cacheable=true` → for reading data (SELECT queries)
- No `cacheable` → for writing data (DML operations)

</details>

---

### Question 3: refreshApex

What must you pass to `refreshApex()` to properly refresh wired data?

- A) The component instance (`this`)
- B) The data returned by the wire (`this.contacts`)
- C) The full wired result object (the object containing both `data` and `error`)
- D) The Apex method name (`getContacts`)

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

`refreshApex()` requires the **full wired result object** — the one that contains both `data` and `error` properties. That's why you need to save it:

```javascript
wiredContactsResult; // Store full result

@wire(getContacts)
wiredContacts(result) {
    this.wiredContactsResult = result; // Save for refreshApex
    const { data, error } = result;
    if (data) this.contacts = data;
}

async handleRefresh() {
    // ✅ Correct — pass the full wired result
    await refreshApex(this.wiredContactsResult);

    // ❌ Incorrect — this.contacts is just the data
    // await refreshApex(this.contacts);
}
```

</details>

---

### Question 4: Custom Event Naming

Which custom event name is VALID in LWC?

- A) `new CustomEvent('contact-selected')`
- B) `new CustomEvent('contactSelected')`
- C) `new CustomEvent('contactselected')`
- D) `new CustomEvent('CONTACT_SELECTED')`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

Custom event names in LWC must be:
- **All lowercase**
- **No hyphens** (-)
- **No underscores** (_)
- **No uppercase letters**

This is because the HTML handler maps directly: event name `contactselected` → HTML attribute `oncontactselected`.

```javascript
// ✅ Valid
new CustomEvent('contactselected', { detail: { id: '001' } });

// ❌ Invalid — these will cause issues
new CustomEvent('contact-selected');  // No hyphens
new CustomEvent('contactSelected');   // No camelCase
new CustomEvent('CONTACT_SELECTED');  // No uppercase/underscores
```

</details>

---

### Question 5: Component Communication

A `contactList` component needs to tell its sibling `contactDetail` component which contact was selected. What is the BEST approach if they are both on the same Lightning page but not in a parent-child relationship?

- A) Use `@api` to pass data directly between siblings
- B) Use Lightning Message Service (publish/subscribe)
- C) Use `document.dispatchEvent()` with a native DOM event
- D) Store the selection in a static Apex variable

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

**Lightning Message Service (LMS)** is the recommended pattern for unrelated components. It provides:
- A managed publish/subscribe system
- Type-safe message channels
- `APPLICATION_SCOPE` for page-wide communication
- Proper lifecycle management (subscribe/unsubscribe)

Why not the others?
- **A**: `@api` only works parent→child, not sibling→sibling
- **C**: Direct DOM events would bypass the LWC framework and could have security issues with Locker Service
- **D**: Static Apex variables don't persist across requests and won't trigger UI updates

</details>

---

### Question 6: Wire to Property vs Function

When should you wire to a FUNCTION instead of a property?

- A) When you need the data to be cached
- B) When you need to transform or process the data before using it
- C) When you want the data to be reactive
- D) When fetching data from Apex

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

Wire to a **function** when you need to:
- Transform or map the data
- Merge data from multiple sources
- Set additional state flags (loading, error)
- Filter or sort the data before display

Wire to a **property** when you just need to display data as-is.

```javascript
// Wire to property — simple display
@wire(getContacts) contacts;

// Wire to function — need to transform
@wire(getContacts)
wiredContacts({ data, error }) {
    if (data) {
        this.contacts = data.map(c => ({
            ...c,
            displayName: `${c.Name} (${c.Email})`
        }));
    }
}
```

Both wiring methods are cached and reactive. Both work with Apex. The choice is about whether you need to process the data.

</details>

---

### Question 7: Event Bubbling

What is the default value of `bubbles` and `composed` for a `CustomEvent` in LWC?

- A) `bubbles: true, composed: true`
- B) `bubbles: true, composed: false`
- C) `bubbles: false, composed: false`
- D) `bubbles: false, composed: true`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

By default, both `bubbles` and `composed` are `false`:
- `bubbles: false` — Event only reaches the **immediate parent** component
- `composed: false` — Event does **not** cross Shadow DOM boundaries

This is the safest default — it keeps events contained and prevents unintended listeners.

```javascript
// Default behavior (recommended for most cases)
new CustomEvent('myevent');
// Same as:
new CustomEvent('myevent', { bubbles: false, composed: false });
```

Only use `bubbles: true, composed: true` when you specifically need the event to travel up through the entire DOM tree and across shadow boundaries.

</details>

---

### Question 8: Lightning Record Forms

Which form component should you use when you need a fully custom layout with your own submit button?

- A) `lightning-record-form`
- B) `lightning-record-edit-form`
- C) `lightning-record-view-form`
- D) `lightning-record-create-form`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

`lightning-record-edit-form` provides:
- Full layout control using `lightning-input-field` components
- Custom submit buttons (use `type="submit"`)
- Custom error handling
- Pre-population of field values
- Custom validation logic

```html
<lightning-record-edit-form object-api-name="Contact" onsuccess={handleSuccess}>
    <!-- Your custom layout -->
    <lightning-input-field field-name="FirstName"></lightning-input-field>
    <lightning-input-field field-name="LastName"></lightning-input-field>

    <!-- Your custom submit button -->
    <lightning-button type="submit" label="Save"></lightning-button>
</lightning-record-edit-form>
```

`lightning-record-form` has a built-in layout you can't customize. `lightning-record-view-form` is read-only. `lightning-record-create-form` doesn't exist.

</details>

---

## Code Output Questions (9-12)

### Question 9: Wire Reactive Parameters

What happens when the user types "John" in the search input?

```javascript
import { LightningElement, wire } from 'lwc';
import searchContacts from '@salesforce/apex/ContactController.searchContacts';

export default class SearchComponent extends LightningElement {
    searchTerm = '';

    @wire(searchContacts, { searchTerm: '$searchTerm' })
    contacts;

    handleSearch(event) {
        this.searchTerm = event.target.value;
    }
}
```

- A) Nothing happens — wire only fires once on load
- B) The wire fires once after the user finishes typing
- C) The wire fires for each keystroke ('J', 'Jo', 'Joh', 'John') — 4 Apex calls
- D) The wire fires once because '$searchTerm' is a static string

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: C**

Because `$searchTerm` is a **reactive parameter**, the wire fires every time `this.searchTerm` changes. Since `handleSearch` updates `searchTerm` on every `change` event (which fires per keystroke for `lightning-input`), the wire will fire for each change: 'J', 'Jo', 'Joh', 'John'.

This is a **real-world problem**! To fix it, implement **debouncing**:

```javascript
searchTimeout;

handleSearch(event) {
    clearTimeout(this.searchTimeout);
    this.searchTimeout = setTimeout(() => {
        this.searchTerm = event.target.value;
    }, 300); // Wait 300ms after user stops typing
}
```

Or use an imperative Apex call instead of wire for search functionality.

</details>

---

### Question 10: @api Property Modification

What happens when this code runs?

```javascript
// childComponent.js
import { LightningElement, api } from 'lwc';

export default class ChildComponent extends LightningElement {
    @api title = 'Default';

    handleClick() {
        this.title = 'New Title'; // Modifying @api property internally
    }
}
```

- A) Works fine — the title updates to "New Title"
- B) Throws a runtime error — @api properties are read-only inside the component
- C) Compiles but silently fails — the value doesn't change
- D) Works but only temporarily — the parent can overwrite it

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

You CANNOT modify an `@api` property from inside the child component. It will throw a runtime error:
> `Error: [LWC error]: Invalid mutation of property "title" in component <c-child-component>. Property was set by the owner component and can only be set by it.`

**Why?** `@api` properties follow a one-way data contract: the **parent owns the data**, and only the parent can change it.

**Fix:** Create a local copy if you need to modify it:
```javascript
@api title = 'Default';
_localTitle;

connectedCallback() {
    this._localTitle = this.title;
}

handleClick() {
    this._localTitle = 'New Title'; // Modify the local copy
}
```

</details>

---

### Question 11: Event Detail Access

What value does `selectedId` hold after the child fires the event?

```javascript
// child.js
handleSelect() {
    this.dispatchEvent(new CustomEvent('select', {
        detail: { id: '003ABC', name: 'John Doe' }
    }));
}
```

```javascript
// parent.js
handleSelect(event) {
    const selectedId = event.detail.id;
    console.log(selectedId);
}
```

```html
<!-- parent.html -->
<c-child onselect={handleSelect}></c-child>
```

- A) `undefined` — event.detail doesn't work cross-component
- B) `'003ABC'`
- C) `{ id: '003ABC', name: 'John Doe' }`
- D) Error — 'select' is a reserved event name

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

`event.detail` contains whatever object was passed in the `detail` property of the `CustomEvent`. So:
- `event.detail` → `{ id: '003ABC', name: 'John Doe' }`
- `event.detail.id` → `'003ABC'`
- `event.detail.name` → `'John Doe'`

The event name 'select' is valid (though it's also a native event name — it's generally fine but best practice is to use more specific names like `contactselect`).

The handler `onselect={handleSelect}` in the parent correctly listens for the `select` event, and the event data is accessible via `event.detail`.

</details>

---

### Question 12: LMS Scope

What happens if Component B subscribes WITHOUT `APPLICATION_SCOPE` and Component A publishes a message? Both components are on the same Lightning page but in different DOM subtrees.

```javascript
// Component B (Subscriber) — NO APPLICATION_SCOPE
subscribe(this.messageContext, CHANNEL, (message) => {
    this.data = message.payload;
});
```

- A) Component B receives the message normally
- B) Component B does NOT receive the message because it needs `APPLICATION_SCOPE`
- C) An error is thrown at subscription time
- D) The message is queued until Component B adds `APPLICATION_SCOPE`

<details>
<summary>✅ Answer & Explanation</summary>

**Correct Answer: B**

Without `APPLICATION_SCOPE`, a subscriber only receives messages from publishers within the **same DOM subtree**. Since Components A and B are in different DOM subtrees on the Lightning page, Component B will NOT receive the message.

**Fix:** Add `APPLICATION_SCOPE`:
```javascript
import { APPLICATION_SCOPE } from 'lightning/messageService';

subscribe(
    this.messageContext,
    CHANNEL,
    (message) => { this.data = message.payload; },
    { scope: APPLICATION_SCOPE }  // Now receives from ANY publisher on the page
);
```

This is a very common gotcha in real projects and interviews!

</details>

---

## Short Answer Questions (13-15)

### Question 13: Wire vs Imperative

You need to build a component that:
1. Shows a list of contacts when it loads
2. Has a "Delete" button next to each contact that removes it from Salesforce
3. The list should refresh after deletion

Should you use Wire or Imperative Apex for each operation? Explain your approach.

<details>
<summary>✅ Answer & Explanation</summary>

**Use BOTH:**

1. **Loading contacts → Wire** (`@wire` with `cacheable=true` Apex method)
   - Data loads automatically when component mounts
   - Leverages LDS cache
   - Reactive — other components can trigger updates

2. **Deleting a contact → Imperative** (regular `@AuraEnabled` Apex method)
   - DML operations (DELETE) cannot use `cacheable=true`
   - Must be called imperatively on button click
   - Need control over when it fires

3. **Refreshing after delete → `refreshApex()`**
   - After the imperative delete succeeds, call `refreshApex(wiredResult)` to re-fetch the list

```javascript
@wire(getContacts)
wiredContacts(result) {
    this.wiredResult = result; // Save for refreshApex
    if (result.data) this.contacts = result.data;
}

async handleDelete(event) {
    const contactId = event.target.dataset.id;
    try {
        await deleteContact({ contactId });
        await refreshApex(this.wiredResult); // Refresh the list
    } catch (error) {
        this.showError(error);
    }
}
```

</details>

---

### Question 14: Communication Pattern

You have this component hierarchy:

```
AppPage
├── Sidebar
│   └── FilterPanel
└── MainContent
    └── DataTable
```

When the user changes a filter in `FilterPanel`, `DataTable` should update. What communication pattern would you use and why?

<details>
<summary>✅ Answer & Explanation</summary>

**Use Lightning Message Service (LMS)** because `FilterPanel` and `DataTable` are NOT in a parent-child relationship. They are in completely different DOM subtrees.

**Why not other patterns?**
- **Custom Events** only travel UP (child→parent), and these components aren't in the same subtree
- **@api properties** only work parent→child, and there's no direct parent-child relationship
- **Sibling pattern** (events up + @api down) would work IF they shared a common parent, but `Sidebar` and `MainContent` are siblings under `AppPage`, making the data flow complex and tightly coupled

**Implementation:**
1. Create a `FilterChanged` message channel
2. `FilterPanel` **publishes** filter changes when the user updates filters
3. `DataTable` **subscribes** with `APPLICATION_SCOPE` and re-fetches data based on the received filters

This keeps the components **decoupled** — they don't need to know about each other, just about the message channel contract.

</details>

---

### Question 15: Error Handling Strategy

Your component calls an Apex method that might throw different types of errors:
- Apex custom exceptions
- DML exceptions (duplicate record)
- SOQL limit exceptions

How would you handle all of these gracefully in LWC?

<details>
<summary>✅ Answer & Explanation</summary>

**Use a `reduceErrors` utility** because error shapes vary between different error types:

```javascript
// Utility function to normalize all error shapes
reduceErrors(errors) {
    if (!Array.isArray(errors)) errors = [errors];

    return errors
        .filter(error => !!error)
        .map(error => {
            // UI API errors (array of {message})
            if (Array.isArray(error.body)) {
                return error.body.map(e => e.message);
            }
            // Apex errors with body.message
            if (error.body && typeof error.body.message === 'string') {
                return error.body.message;
            }
            // Page-level errors (field errors)
            if (error.body && error.body.fieldErrors) {
                return Object.values(error.body.fieldErrors)
                    .flat()
                    .map(fe => fe.message);
            }
            // Plain JS error
            if (typeof error.message === 'string') {
                return error.message;
            }
            return error.statusText || 'Unknown error';
        })
        .flat()
        .join('; ');
}
```

**Best practice:**
1. Create a shared `errorUtils.js` service module
2. Import it across components: `import { reduceErrors } from 'c/errorUtils'`
3. Always show user-friendly messages via `ShowToastEvent`
4. Log detailed errors to console for debugging
5. Implement a fallback UI state (error message display) in the template

In Apex, wrap your logic in try-catch and throw `AuraHandledException` with meaningful messages:
```java
try {
    insert contact;
} catch (DmlException e) {
    throw new AuraHandledException('Could not create contact: ' + e.getMessage());
}
```

</details>

---

## 📊 Score Yourself

| Score | Level |
|---|---|
| **13-15** | 🌟 Excellent — Ready for Day 3! |
| **10-12** | ✅ Good — Review the topics you missed |
| **7-9** | ⚠️ Fair — Re-read the sections for missed questions |
| **Below 7** | 🔄 Re-study — Go through Day 2 material again |

---

**Next → [Day 2 Quiz](./quiz.md)** | **Continue → [Day 3: Advanced & Interview](../day-3-advanced-and-interview/README.md)**
