# Lightning Web Components (LWC) Practice Questions

This guide contains a progression of LWC interview questions ranging from basic concepts to advanced architectural scenarios, complete with code solutions.

---

## 🟢 Level 1: Basic Fundamentals

### Question 1: Two-Way Data Binding Simulator
**Scenario:** LWC enforces one-way data binding. Create a simple component with a text input field and a paragraph. As the user types into the input field, the paragraph should update instantly with what they are typing.

**Solution:**
You need to use an `onchange` or `onkeyup` event handler on the input field to update a JavaScript property.

**`liveTyper.js`**
```javascript
import { LightningElement } from 'lwc';

export default class LiveTyper extends LightningElement {
    userInput = '';

    handleInputChange(event) {
        // Update the property with the current value of the input field
        this.userInput = event.target.value;
    }
}
```

**`liveTyper.html`**
```html
<template>
    <lightning-input 
        label="Type something:" 
        value={userInput} 
        onchange={handleInputChange}>
    </lightning-input>
    <p>You are typing: {userInput}</p>
</template>
```

### Question 2: Parent-to-Child Communication
**Scenario:** You have a parent component `AccountDashboard` and a child component `AccountCard`. How do you pass an account name from the Dashboard to the Card?

**Solution:**
Use the `@api` decorator in the child component to expose a public property.

**Child: `accountCard.js`**
```javascript
import { LightningElement, api } from 'lwc';

export default class AccountCard extends LightningElement {
    // @api makes this property public so parents can set it
    @api accountName;
}
```
**Child: `accountCard.html`**
```html
<template>
    <div class="card">Account Name: {accountName}</div>
</template>
```

**Parent: `accountDashboard.html`**
```html
<template>
    <!-- Kebab-case is used in the HTML template for camelCase JavaScript properties -->
    <c-account-card account-name="Acme Corporation"></c-account-card>
</template>
```

---

## 🟡 Level 2: Intermediate Concepts

### Question 3: Child-to-Parent Communication
**Scenario:** Following up on the previous question, the `AccountCard` child component has a "Delete" button. When clicked, it needs to tell the `AccountDashboard` parent component to remove it from the list. How do you achieve this?

**Solution:**
The child component must dispatch a `CustomEvent` which the parent listens for.

**Child: `accountCard.js`**
```javascript
import { LightningElement, api } from 'lwc';

export default class AccountCard extends LightningElement {
    @api accountId;

    handleDelete() {
        // 1. Create a custom event
        // 2. Pass data in the 'detail' property
        const deleteEvent = new CustomEvent('accountdelete', { 
            detail: { id: this.accountId } 
        });
        
        // 3. Dispatch the event up to the parent
        this.dispatchEvent(deleteEvent);
    }
}
```

**Parent: `accountDashboard.html`**
```html
<template>
    <!-- Listen for 'onaccountdelete' (the word 'on' + your event name) -->
    <c-account-card 
        account-id="001xyz" 
        onaccountdelete={handleChildDelete}>
    </c-account-card>
</template>
```

**Parent: `accountDashboard.js`**
```javascript
import { LightningElement } from 'lwc';

export default class AccountDashboard extends LightningElement {
    handleChildDelete(event) {
        // Retrieve the data passed by the child
        const deletedId = event.detail.id;
        console.log('Parent reacting to delete for ID: ', deletedId);
        // Logic to remove from list goes here...
    }
}
```

### Question 4: The Wire Service (`@wire`)
**Scenario:** You need to retrieve a list of Contacts using an Apex controller method named `getContactList`. You want the data to be retrieved automatically when the component loads. How do you implement this?

**Solution:**
Import the Apex method and use the `@wire` decorator to automatically provision the data.

**`contactList.js`**
```javascript
import { LightningElement, wire } from 'lwc';
// Import the apex method. Assuming it is annotated with @AuraEnabled(cacheable=true)
import getContactList from '@salesforce/apex/ContactController.getContactList';

export default class ContactList extends LightningElement {
    
    // The @wire service automatically calls the Apex method and provisions the results
    // to the 'contacts' property, which will have { data, error }
    @wire(getContactList)
    contacts;
}
```

**`contactList.html`**
```html
<template>
    <!-- Check if data exists -->
    <template if:true={contacts.data}>
        <ul>
            <!-- Loop through the wired data -->
            <template for:each={contacts.data} for:item="contact">
                <li key={contact.Id}>{contact.Name}</li>
            </template>
        </ul>
    </template>
    
    <!-- Handle Errors -->
    <template if:true={contacts.error}>
        <p>Error retrieving contacts!</p>
    </template>
</template>
```

---

## 🔴 Level 3: Advanced Architecture

### Question 5: Component Lifecycle Hooks
**Scenario:** You have a component that relies on a third-party charting library (e.g., Chart.js). You need to initialize the chart by querying a `<canvas>` element in your HTML. In which lifecycle hook should you perform this initialization and why?

**Solution:**
You should use `renderedCallback()`.

**Explanation:**
* `connectedCallback()` fires when the component is inserted into the DOM, but **before** the HTML template has been rendered. If you try to use `this.template.querySelector('canvas')` here, it will return null.
* `renderedCallback()` fires **after** the component's HTML has finished rendering. This is the correct place to interact with DOM elements or initialize third-party libraries.

> [!WARNING]
> `renderedCallback()` fires multiple times (every time the component's reactive properties change and cause a re-render). To prevent initializing a third-party library multiple times, you must use a boolean flag (e.g., `isChartInitialized`) to ensure your initialization logic only runs once.

### Question 6: Lightning Message Service (LMS)
**Scenario:** You have two components on a Lightning Page: A `FilterComponent` in the left sidebar, and a `ResultsGrid` in the main body. They do not share a parent-child relationship. When a user selects a filter on the left, the grid on the right needs to update. How do you communicate between these unrelated components?

**Solution:**
You should use the **Lightning Message Service (LMS)**.

**Steps:**
1. Create a Lightning Message Channel metadata file (e.g., `FilterUpdate.messageChannel-meta.xml`).
2. The `FilterComponent` imports the channel and uses the `publish` function from the `lightning/messageService` module to broadcast the filter data.
3. The `ResultsGrid` component imports the same channel and uses the `subscribe` function (usually in the `connectedCallback`) to listen for broadcasts and update its data accordingly.

> [!NOTE]
> Previously, developers used a custom "PubSub" pattern (Event Handler) for this, but LMS is now the official best practice as it also allows communication across LWC, Aura, and Visualforce pages.
