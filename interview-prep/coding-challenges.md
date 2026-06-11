# 💻 LWC Coding Challenges for Interviews

> [!NOTE]
> These 10 coding challenges mirror real interview exercises at top Salesforce consulting firms and ISVs. Practice each one in a scratch org before your interview. Time yourself — most interviewers expect a working solution within **30–45 minutes**.

## How to Use This Guide

| Step | Action |
|------|--------|
| 1 | Read the problem statement carefully |
| 2 | Review the hints (only if stuck) |
| 3 | Start with the skeleton code |
| 4 | Build your solution incrementally |
| 5 | Compare with the complete solution |
| 6 | Prepare for the follow-up questions |

---

## Challenge 1: Searchable Contact List

| Property | Value |
|----------|-------|
| **Difficulty** | 🟢 Easy |
| **Time Limit** | 20 minutes |
| **Key Concepts** | Wire service, reactive properties, filtering, getters |
| **Common In** | Junior/Mid-level interviews |

### 📋 Problem Statement

Build a Lightning Web Component that displays a list of contacts fetched from Salesforce. The user should be able to type in a search box and the list should filter **in real-time** (client-side) by contact name. Display each contact's Name, Email, and Phone in a card-style layout.

**Requirements:**
- Fetch contacts using a wire adapter connected to an Apex method
- Implement client-side search filtering (case-insensitive)
- Show a "No contacts found" message when the filter yields no results
- Display total count of visible contacts
- Handle loading and error states gracefully

### 💡 Hints

1. Use a **getter** to compute the filtered list instead of storing it in a separate property — this keeps your component reactive without manual state synchronization.
2. Remember that `@wire` results are **read-only**. You need to spread or map the data before filtering.
3. Use `toLowerCase()` on both the search term and contact names for case-insensitive matching.

### 🏗️ Skeleton Code

**searchableContactList.html**
```html
<template>
    <lightning-card title="Contact Search" icon-name="standard:contact">
        <!-- Search input goes here -->

        <!-- Contact count display -->

        <!-- Loading spinner -->

        <!-- Contact list iteration -->

        <!-- No results message -->

        <!-- Error handling -->
    </lightning-card>
</template>
```

**searchableContactList.js**
```javascript
import { LightningElement, wire } from 'lwc';
import getContacts from '@salesforce/apex/ContactController.getContacts';

export default class SearchableContactList extends LightningElement {
    searchTerm = '';
    // TODO: Wire the Apex method

    // TODO: Create a getter for filtered contacts

    // TODO: Handle search input changes
}
```

**ContactController.cls**
```java
public with sharing class ContactController {
    @AuraEnabled(cacheable=true)
    public static List<Contact> getContacts() {
        return [
            SELECT Id, Name, Email, Phone, Title
            FROM Contact
            ORDER BY Name
            LIMIT 200
        ];
    }
}
```

<details>
<summary>💡 Complete Solution</summary>

**searchableContactList.html**
```html
<template>
    <lightning-card title="Contact Search" icon-name="standard:contact">
        <div class="slds-p-around_medium">
            <!-- Search Input -->
            <lightning-input
                type="search"
                label="Search Contacts"
                placeholder="Type to search by name..."
                value={searchTerm}
                onchange={handleSearchChange}
                class="slds-m-bottom_medium"
            ></lightning-input>

            <!-- Contact Count -->
            <p class="slds-m-bottom_small slds-text-body_small slds-text-color_weak">
                Showing {filteredContactCount} contact(s)
            </p>

            <!-- Loading Spinner -->
            <template lwc:if={isLoading}>
                <lightning-spinner alternative-text="Loading contacts..." size="small"></lightning-spinner>
            </template>

            <!-- Contact List -->
            <template lwc:if={hasContacts}>
                <template for:each={filteredContacts} for:item="contact">
                    <div key={contact.Id} class="slds-box slds-m-bottom_x-small slds-theme_shade">
                        <div class="slds-grid slds-gutters">
                            <div class="slds-col slds-size_1-of-3">
                                <p class="slds-text-heading_small">{contact.Name}</p>
                                <p class="slds-text-body_small slds-text-color_weak">{contact.Title}</p>
                            </div>
                            <div class="slds-col slds-size_1-of-3">
                                <p>📧 {contact.Email}</p>
                            </div>
                            <div class="slds-col slds-size_1-of-3">
                                <p>📞 {contact.Phone}</p>
                            </div>
                        </div>
                    </div>
                </template>
            </template>

            <!-- No Results -->
            <template lwc:if={noResults}>
                <div class="slds-align_absolute-center slds-p-around_large">
                    <p class="slds-text-color_weak">No contacts match your search.</p>
                </div>
            </template>

            <!-- Error State -->
            <template lwc:if={error}>
                <p class="slds-text-color_error">{errorMessage}</p>
            </template>
        </div>
    </lightning-card>
</template>
```

**searchableContactList.js**
```javascript
import { LightningElement, wire } from 'lwc';
import getContacts from '@salesforce/apex/ContactController.getContacts';

export default class SearchableContactList extends LightningElement {
    searchTerm = '';
    contacts;
    error;
    isLoading = true;

    @wire(getContacts)
    wiredContacts({ error, data }) {
        this.isLoading = false;
        if (data) {
            this.contacts = data;
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.contacts = undefined;
        }
    }

    handleSearchChange(event) {
        this.searchTerm = event.target.value.toLowerCase();
    }

    get filteredContacts() {
        if (!this.contacts) return [];
        if (!this.searchTerm) return this.contacts;

        return this.contacts.filter(contact =>
            contact.Name.toLowerCase().includes(this.searchTerm)
        );
    }

    get filteredContactCount() {
        return this.filteredContacts.length;
    }

    get hasContacts() {
        return this.filteredContacts.length > 0;
    }

    get noResults() {
        return this.contacts && this.contacts.length > 0
            && this.filteredContacts.length === 0;
    }

    get errorMessage() {
        return this.error?.body?.message || 'An unexpected error occurred.';
    }
}
```

</details>

### 📖 Explanation

The solution uses the **wire service** with `cacheable=true` to fetch contacts once and cache them. The key architectural decision is using a **getter** (`filteredContacts`) instead of an imperative approach. Getters in LWC are recomputed every time the template re-renders, which happens whenever a tracked/reactive property like `searchTerm` changes. This means the filter is always up to date without manual synchronization. The `hasContacts` and `noResults` getters drive conditional rendering so the template remains clean and declarative.

### 🔄 Follow-up Questions

1. **"How would you add server-side search if the contact list had 50,000+ records?"** — Use an imperative Apex call with a SOSL or SOQL `LIKE` query instead of fetching all records.
2. **"How would you debounce the search input?"** — Use `setTimeout`/`clearTimeout` to delay filtering until the user stops typing for ~300ms.
3. **"What happens to the wire cache if a contact is updated elsewhere?"** — The cached data becomes stale. You'd need `refreshApex()` to force a refresh.

---

## Challenge 2: Reusable Modal Component

| Property | Value |
|----------|-------|
| **Difficulty** | 🟢 Easy |
| **Time Limit** | 20 minutes |
| **Key Concepts** | Slots, custom events, public API (@api), accessibility |
| **Common In** | Junior/Mid-level interviews |

### 📋 Problem Statement

Create a reusable modal dialog component that can be used across different parts of an application. The modal should accept its header, body content, and footer through **slots**. It should support opening/closing via a public API and fire events when the user confirms or cancels.

**Requirements:**
- Use named slots for header, body, and footer
- Expose `open()` and `close()` public methods
- Support keyboard accessibility (ESC to close)
- Include a backdrop overlay that closes the modal on click
- Fire `confirm` and `cancel` custom events
- Apply SLDS modal styling

### 💡 Hints

1. Use `@api` methods (`open`, `close`) so parent components can control the modal imperatively via `this.template.querySelector('c-reusable-modal').open()`.
2. Use `connectedCallback` and `disconnectedCallback` to add/remove the `keydown` event listener for ESC key handling.
3. Named slots use the syntax `<slot name="header"></slot>` in the child and `slot="header"` attribute in the parent.

### 🏗️ Skeleton Code

**reusableModal.html**
```html
<template>
    <!-- Modal container (conditionally rendered) -->
    <!-- Backdrop overlay -->
    <!-- Modal content with slots for header, body, footer -->
    <!-- Close button -->
</template>
```

**reusableModal.js**
```javascript
import { LightningElement, api } from 'lwc';

export default class ReusableModal extends LightningElement {
    isOpen = false;

    // TODO: Implement @api open() method
    // TODO: Implement @api close() method
    // TODO: Handle ESC key
    // TODO: Handle backdrop click
    // TODO: Fire confirm/cancel events
}
```

<details>
<summary>💡 Complete Solution</summary>

**reusableModal.html**
```html
<template>
    <template lwc:if={isOpen}>
        <!-- Backdrop -->
        <div class="slds-backdrop slds-backdrop_open" onclick={handleBackdropClick}></div>

        <!-- Modal -->
        <section
            role="dialog"
            tabindex="-1"
            aria-modal="true"
            aria-labelledby="modal-heading"
            class="slds-modal slds-fade-in-open"
        >
            <div class="slds-modal__container">
                <!-- Header -->
                <div class="slds-modal__header">
                    <lightning-button-icon
                        icon-name="utility:close"
                        alternative-text="Close"
                        variant="bare-inverse"
                        class="slds-modal__close"
                        onclick={handleCancel}
                    ></lightning-button-icon>
                    <h2 id="modal-heading" class="slds-modal__title slds-hyphenate">
                        <slot name="header">Default Modal Title</slot>
                    </h2>
                </div>

                <!-- Body -->
                <div class="slds-modal__content slds-p-around_medium">
                    <slot name="body">Default body content</slot>
                </div>

                <!-- Footer -->
                <div class="slds-modal__footer">
                    <slot name="footer">
                        <lightning-button
                            label="Cancel"
                            onclick={handleCancel}
                            class="slds-m-right_x-small"
                        ></lightning-button>
                        <lightning-button
                            label="Confirm"
                            variant="brand"
                            onclick={handleConfirm}
                        ></lightning-button>
                    </slot>
                </div>
            </div>
        </section>
    </template>
</template>
```

**reusableModal.js**
```javascript
import { LightningElement, api } from 'lwc';

export default class ReusableModal extends LightningElement {
    isOpen = false;

    // Bound reference so we can remove the listener later
    _handleKeyDown = this.handleKeyDown.bind(this);

    @api
    open() {
        this.isOpen = true;
        // Prevent background scroll
        document.body.style.overflow = 'hidden';
    }

    @api
    close() {
        this.isOpen = false;
        document.body.style.overflow = '';
    }

    connectedCallback() {
        document.addEventListener('keydown', this._handleKeyDown);
    }

    disconnectedCallback() {
        document.removeEventListener('keydown', this._handleKeyDown);
        document.body.style.overflow = '';
    }

    handleKeyDown(event) {
        if (event.key === 'Escape' && this.isOpen) {
            this.handleCancel();
        }
    }

    handleBackdropClick() {
        this.handleCancel();
    }

    handleCancel() {
        this.close();
        this.dispatchEvent(new CustomEvent('cancel'));
    }

    handleConfirm() {
        this.close();
        this.dispatchEvent(new CustomEvent('confirm'));
    }
}
```

**reusableModal.css**
```css
:host {
    position: relative;
    z-index: 9000;
}
```

**Usage in parent component:**
```html
<template>
    <lightning-button label="Open Modal" onclick={openModal}></lightning-button>

    <c-reusable-modal onconfirm={handleConfirm} oncancel={handleCancel}>
        <span slot="header">Create New Account</span>
        <div slot="body">
            <lightning-input label="Account Name" required></lightning-input>
            <lightning-input label="Industry"></lightning-input>
        </div>
        <div slot="footer">
            <lightning-button label="Save" variant="brand" onclick={handleSave}></lightning-button>
        </div>
    </c-reusable-modal>
</template>
```

```javascript
// Parent JS
openModal() {
    this.template.querySelector('c-reusable-modal').open();
}
```

</details>

### 📖 Explanation

This solution demonstrates several key LWC patterns: **slots** for content projection (allowing the parent to inject any content), **@api methods** for imperative parent-to-child control, and **custom events** for child-to-parent communication. The `bind(this)` pattern is essential for event listeners — without it, `this` inside `handleKeyDown` would refer to `document`, not the component. The `disconnectedCallback` cleanup prevents memory leaks. ARIA attributes (`role="dialog"`, `aria-modal`) ensure screen readers announce the modal correctly.

### 🔄 Follow-up Questions

1. **"How would you handle focus trapping inside the modal?"** — Implement a focus trap by tracking the first and last focusable elements and redirecting focus on Tab/Shift+Tab.
2. **"Can you use `lightning-modal` instead? What are the trade-offs?"** — `LightningModal` (extending `LightningElement`) is available in API 55+. It handles accessibility automatically but offers less customization.
3. **"How would you animate the modal opening/closing?"** — Use CSS transitions on opacity and transform, or use the SLDS `slds-fade-in-open` class toggling.

---

## Challenge 3: Paginated Data Table

| Property | Value |
|----------|-------|
| **Difficulty** | 🟡 Medium |
| **Time Limit** | 35 minutes |
| **Key Concepts** | Imperative Apex, lightning-datatable, offset pagination, state management |
| **Common In** | Mid/Senior-level interviews |

### 📋 Problem Statement

Build a paginated data table that fetches Account records from Salesforce using **server-side pagination**. The user should be able to navigate between pages, select a page size, and see which page they're on.

**Requirements:**
- Use `lightning-datatable` to display records
- Implement server-side pagination (offset-based) using imperative Apex
- Allow the user to select page size (5, 10, 25, 50)
- Show "Page X of Y" with Previous/Next buttons
- Disable Previous on first page, Next on last page
- Handle loading states during page transitions

### 💡 Hints

1. You need **two** Apex methods: one to get the total record count and another to get a page of records using `LIMIT` and `OFFSET`.
2. Calculate `totalPages` as `Math.ceil(totalRecords / pageSize)`. Recalculate when the page size changes.
3. Use imperative Apex (not `@wire`) because you need to call the method programmatically when the page or page size changes.

### 🏗️ Skeleton Code

**paginatedDataTable.js**
```javascript
import { LightningElement } from 'lwc';
import getAccounts from '@salesforce/apex/AccountPaginationController.getAccounts';
import getTotalCount from '@salesforce/apex/AccountPaginationController.getTotalCount';

export default class PaginatedDataTable extends LightningElement {
    columns = [
        { label: 'Name', fieldName: 'Name', type: 'text' },
        { label: 'Industry', fieldName: 'Industry', type: 'text' },
        { label: 'Revenue', fieldName: 'AnnualRevenue', type: 'currency' },
        { label: 'Phone', fieldName: 'Phone', type: 'phone' }
    ];

    data = [];
    currentPage = 1;
    pageSize = 10;
    totalRecords = 0;
    isLoading = false;

    // TODO: connectedCallback to load initial data
    // TODO: loadData method
    // TODO: handlePrevious / handleNext
    // TODO: handlePageSizeChange
    // TODO: Getters for totalPages, isPreviousDisabled, isNextDisabled
}
```

<details>
<summary>💡 Complete Solution</summary>

**AccountPaginationController.cls**
```java
public with sharing class AccountPaginationController {
    @AuraEnabled(cacheable=false)
    public static List<Account> getAccounts(Integer pageSize, Integer pageNumber) {
        Integer offset = (pageNumber - 1) * pageSize;
        return [
            SELECT Id, Name, Industry, AnnualRevenue, Phone
            FROM Account
            ORDER BY Name
            LIMIT :pageSize
            OFFSET :offset
        ];
    }

    @AuraEnabled(cacheable=true)
    public static Integer getTotalCount() {
        return [SELECT COUNT() FROM Account];
    }
}
```

**paginatedDataTable.html**
```html
<template>
    <lightning-card title="Accounts" icon-name="standard:account">
        <div class="slds-p-around_medium">
            <!-- Page Size Selector -->
            <div class="slds-grid slds-m-bottom_medium">
                <div class="slds-col slds-size_1-of-4">
                    <lightning-combobox
                        label="Page Size"
                        value={pageSizeStr}
                        options={pageSizeOptions}
                        onchange={handlePageSizeChange}
                    ></lightning-combobox>
                </div>
                <div class="slds-col slds-size_3-of-4 slds-align_absolute-center">
                    <p class="slds-text-body_small">
                        Showing {rangeStart}–{rangeEnd} of {totalRecords} records
                    </p>
                </div>
            </div>

            <!-- Data Table -->
            <lightning-datatable
                key-field="Id"
                data={data}
                columns={columns}
                hide-checkbox-column
                show-row-number-column
                row-number-offset={rowNumberOffset}
            ></lightning-datatable>

            <!-- Loading Overlay -->
            <template lwc:if={isLoading}>
                <lightning-spinner alternative-text="Loading" size="medium"></lightning-spinner>
            </template>

            <!-- Pagination Controls -->
            <div class="slds-grid slds-grid_align-center slds-m-top_medium">
                <lightning-button
                    label="Previous"
                    icon-name="utility:chevronleft"
                    onclick={handlePrevious}
                    disabled={isPreviousDisabled}
                    class="slds-m-right_x-small"
                ></lightning-button>
                <span class="slds-align-middle slds-p-horizontal_medium">
                    Page {currentPage} of {totalPages}
                </span>
                <lightning-button
                    label="Next"
                    icon-name="utility:chevronright"
                    icon-position="right"
                    onclick={handleNext}
                    disabled={isNextDisabled}
                ></lightning-button>
            </div>
        </div>
    </lightning-card>
</template>
```

**paginatedDataTable.js**
```javascript
import { LightningElement } from 'lwc';
import getAccounts from '@salesforce/apex/AccountPaginationController.getAccounts';
import getTotalCount from '@salesforce/apex/AccountPaginationController.getTotalCount';

export default class PaginatedDataTable extends LightningElement {
    columns = [
        { label: 'Name', fieldName: 'Name', type: 'text', sortable: true },
        { label: 'Industry', fieldName: 'Industry', type: 'text' },
        { label: 'Revenue', fieldName: 'AnnualRevenue', type: 'currency' },
        { label: 'Phone', fieldName: 'Phone', type: 'phone' }
    ];

    data = [];
    currentPage = 1;
    pageSize = 10;
    totalRecords = 0;
    isLoading = false;

    pageSizeOptions = [
        { label: '5', value: '5' },
        { label: '10', value: '10' },
        { label: '25', value: '25' },
        { label: '50', value: '50' }
    ];

    connectedCallback() {
        this.loadTotalCount();
        this.loadData();
    }

    async loadTotalCount() {
        try {
            this.totalRecords = await getTotalCount();
        } catch (error) {
            console.error('Error fetching count:', error);
        }
    }

    async loadData() {
        this.isLoading = true;
        try {
            this.data = await getAccounts({
                pageSize: this.pageSize,
                pageNumber: this.currentPage
            });
        } catch (error) {
            console.error('Error fetching accounts:', error);
            this.data = [];
        } finally {
            this.isLoading = false;
        }
    }

    handlePrevious() {
        if (this.currentPage > 1) {
            this.currentPage--;
            this.loadData();
        }
    }

    handleNext() {
        if (this.currentPage < this.totalPages) {
            this.currentPage++;
            this.loadData();
        }
    }

    handlePageSizeChange(event) {
        this.pageSize = parseInt(event.detail.value, 10);
        this.currentPage = 1; // Reset to first page
        this.loadData();
    }

    get pageSizeStr() {
        return String(this.pageSize);
    }

    get totalPages() {
        return Math.ceil(this.totalRecords / this.pageSize) || 1;
    }

    get isPreviousDisabled() {
        return this.currentPage <= 1;
    }

    get isNextDisabled() {
        return this.currentPage >= this.totalPages;
    }

    get rowNumberOffset() {
        return (this.currentPage - 1) * this.pageSize;
    }

    get rangeStart() {
        return ((this.currentPage - 1) * this.pageSize) + 1;
    }

    get rangeEnd() {
        const end = this.currentPage * this.pageSize;
        return end > this.totalRecords ? this.totalRecords : end;
    }
}
```

</details>

### 📖 Explanation

Server-side pagination is essential for large datasets because fetching all records client-side would hit governor limits and degrade performance. The offset-based approach uses `LIMIT :pageSize OFFSET :offset` in SOQL. We use **imperative Apex** (not `@wire`) because pagination requires calling the method with dynamic parameters when the user navigates. The total count is fetched separately and cached (`cacheable=true`) since it rarely changes during a session. Key UX touches include the row number offset, range display ("Showing 11–20 of 150"), and disabled button states.

### 🔄 Follow-up Questions

1. **"What are the limitations of OFFSET in SOQL?"** — OFFSET has a max of 2,000. For larger datasets, use cursor-based pagination (keyset pagination) with `WHERE Id > :lastId ORDER BY Id LIMIT :pageSize`.
2. **"How would you add sorting?"** — Handle the `onsort` event on `lightning-datatable`, pass the sort field and direction to the Apex method, and add `ORDER BY` dynamically.
3. **"How would you handle the total count becoming stale?"** — Refresh the count periodically or after DML operations. You could also use `@wire` with `refreshApex()`.

---

## Challenge 4: Parent-Child Communication Demo (Shopping Cart)

| Property | Value |
|----------|-------|
| **Difficulty** | 🟡 Medium |
| **Time Limit** | 30 minutes |
| **Key Concepts** | @api, custom events, event bubbling, component composition |
| **Common In** | All experience levels |

### 📋 Problem Statement

Build a shopping cart application with two components: a **Product List** (child) and a **Shopping Cart** (parent). The product list displays available items. When the user clicks "Add to Cart" on a product, the parent cart component should update to show the item and the total price.

**Requirements:**
- Parent component (`shoppingCart`) manages the cart state
- Child component (`productList`) displays products and fires events
- Support adding items, removing items, and updating quantities
- Display a running total with correct currency formatting
- Show an empty cart message when no items are added

### 💡 Hints

1. The child fires a `CustomEvent` with the product details in `event.detail`. The parent listens with `onaddtocart` (kebab-case attribute for `addtocart` event name).
2. Use an **immutable data pattern** — create new arrays/objects instead of mutating existing ones so the framework detects changes.
3. Use `Intl.NumberFormat` or a getter to format the total price as currency.

### 🏗️ Skeleton Code

**productList.js (child)**
```javascript
import { LightningElement } from 'lwc';

export default class ProductList extends LightningElement {
    products = [
        { id: '1', name: 'Wireless Mouse', price: 29.99, image: '🖱️' },
        { id: '2', name: 'Mechanical Keyboard', price: 79.99, image: '⌨️' },
        { id: '3', name: 'USB-C Hub', price: 49.99, image: '🔌' },
        { id: '4', name: 'Monitor Stand', price: 39.99, image: '🖥️' },
        { id: '5', name: 'Webcam HD', price: 59.99, image: '📷' }
    ];

    // TODO: Handle add to cart click
    // TODO: Fire custom event with product detail
}
```

<details>
<summary>💡 Complete Solution</summary>

**productList.html (child)**
```html
<template>
    <lightning-card title="Products" icon-name="standard:product">
        <div class="slds-p-around_medium">
            <div class="slds-grid slds-wrap slds-gutters">
                <template for:each={products} for:item="product">
                    <div key={product.id} class="slds-col slds-size_1-of-3 slds-m-bottom_medium">
                        <div class="slds-box slds-text-align_center">
                            <p class="slds-text-heading_large">{product.image}</p>
                            <p class="slds-text-heading_small slds-m-top_small">{product.name}</p>
                            <p class="slds-text-color_weak slds-m-bottom_small">
                                ${product.price}
                            </p>
                            <lightning-button
                                label="Add to Cart"
                                variant="brand"
                                data-product-id={product.id}
                                onclick={handleAddToCart}
                            ></lightning-button>
                        </div>
                    </div>
                </template>
            </div>
        </div>
    </lightning-card>
</template>
```

**productList.js (child)**
```javascript
import { LightningElement } from 'lwc';

export default class ProductList extends LightningElement {
    products = [
        { id: '1', name: 'Wireless Mouse', price: 29.99, image: '🖱️' },
        { id: '2', name: 'Mechanical Keyboard', price: 79.99, image: '⌨️' },
        { id: '3', name: 'USB-C Hub', price: 49.99, image: '🔌' },
        { id: '4', name: 'Monitor Stand', price: 39.99, image: '🖥️' },
        { id: '5', name: 'Webcam HD', price: 59.99, image: '📷' }
    ];

    handleAddToCart(event) {
        const productId = event.target.dataset.productId;
        const product = this.products.find(p => p.id === productId);

        this.dispatchEvent(new CustomEvent('addtocart', {
            detail: { ...product }
        }));
    }
}
```

**shoppingCart.html (parent)**
```html
<template>
    <div class="slds-grid slds-gutters slds-p-around_medium">
        <!-- Product List (Child) -->
        <div class="slds-col slds-size_2-of-3">
            <c-product-list onaddtocart={handleAddToCart}></c-product-list>
        </div>

        <!-- Cart Summary -->
        <div class="slds-col slds-size_1-of-3">
            <lightning-card title="Shopping Cart" icon-name="standard:orders">
                <div class="slds-p-around_medium">
                    <template lwc:if={hasItems}>
                        <template for:each={cartItems} for:item="item">
                            <div key={item.id} class="slds-grid slds-m-bottom_small slds-border_bottom slds-p-bottom_small">
                                <div class="slds-col slds-size_1-of-2">
                                    <p class="slds-text-body_regular">
                                        {item.image} {item.name}
                                    </p>
                                </div>
                                <div class="slds-col slds-size_1-of-4 slds-text-align_center">
                                    <lightning-button-icon
                                        icon-name="utility:dash"
                                        alternative-text="Decrease"
                                        size="small"
                                        data-item-id={item.id}
                                        onclick={handleDecrease}
                                    ></lightning-button-icon>
                                    <span class="slds-p-horizontal_x-small">{item.quantity}</span>
                                    <lightning-button-icon
                                        icon-name="utility:add"
                                        alternative-text="Increase"
                                        size="small"
                                        data-item-id={item.id}
                                        onclick={handleIncrease}
                                    ></lightning-button-icon>
                                </div>
                                <div class="slds-col slds-size_1-of-4 slds-text-align_right">
                                    <p>${item.subtotal}</p>
                                </div>
                            </div>
                        </template>

                        <div class="slds-grid slds-m-top_medium slds-border_top slds-p-top_medium">
                            <div class="slds-col slds-size_1-of-2">
                                <p class="slds-text-heading_small"><strong>Total:</strong></p>
                            </div>
                            <div class="slds-col slds-size_1-of-2 slds-text-align_right">
                                <p class="slds-text-heading_small"><strong>${formattedTotal}</strong></p>
                            </div>
                        </div>

                        <lightning-button
                            label="Clear Cart"
                            variant="destructive"
                            onclick={handleClearCart}
                            class="slds-m-top_medium"
                        ></lightning-button>
                    </template>

                    <template lwc:if={isEmpty}>
                        <div class="slds-align_absolute-center slds-p-around_large">
                            <p class="slds-text-color_weak">Your cart is empty 🛒</p>
                        </div>
                    </template>
                </div>
            </lightning-card>
        </div>
    </div>
</template>
```

**shoppingCart.js (parent)**
```javascript
import { LightningElement } from 'lwc';

export default class ShoppingCart extends LightningElement {
    cartItems = [];

    handleAddToCart(event) {
        const product = event.detail;
        const existingIndex = this.cartItems.findIndex(item => item.id === product.id);

        if (existingIndex !== -1) {
            // Create new array with updated item (immutable pattern)
            this.cartItems = this.cartItems.map((item, index) => {
                if (index === existingIndex) {
                    const newQty = item.quantity + 1;
                    return {
                        ...item,
                        quantity: newQty,
                        subtotal: (newQty * item.price).toFixed(2)
                    };
                }
                return item;
            });
        } else {
            this.cartItems = [
                ...this.cartItems,
                {
                    ...product,
                    quantity: 1,
                    subtotal: product.price.toFixed(2)
                }
            ];
        }
    }

    handleIncrease(event) {
        const itemId = event.target.dataset.itemId;
        this.cartItems = this.cartItems.map(item => {
            if (item.id === itemId) {
                const newQty = item.quantity + 1;
                return { ...item, quantity: newQty, subtotal: (newQty * item.price).toFixed(2) };
            }
            return item;
        });
    }

    handleDecrease(event) {
        const itemId = event.target.dataset.itemId;
        this.cartItems = this.cartItems
            .map(item => {
                if (item.id === itemId) {
                    const newQty = item.quantity - 1;
                    if (newQty <= 0) return null; // Mark for removal
                    return { ...item, quantity: newQty, subtotal: (newQty * item.price).toFixed(2) };
                }
                return item;
            })
            .filter(item => item !== null);
    }

    handleClearCart() {
        this.cartItems = [];
    }

    get formattedTotal() {
        const total = this.cartItems.reduce(
            (sum, item) => sum + (item.price * item.quantity), 0
        );
        return total.toFixed(2);
    }

    get hasItems() {
        return this.cartItems.length > 0;
    }

    get isEmpty() {
        return this.cartItems.length === 0;
    }
}
```

</details>

### 📖 Explanation

This challenge tests your understanding of the **unidirectional data flow** in LWC. The parent owns the state (`cartItems`), the child fires events upward, and data flows down through properties. The **immutable data pattern** (using spread operator and `map` to create new arrays) is crucial — LWC's reactivity system tracks object references, so mutating an array element in place (e.g., `this.cartItems[0].quantity++`) won't trigger a re-render. The `handleDecrease` method demonstrates a clean pattern for conditional removal: map items, return `null` for removals, then filter out nulls.

### 🔄 Follow-up Questions

1. **"How would you persist the cart across page navigations?"** — Use `localStorage`, a Lightning Web State (LWS) store, or the Lightning Message Service.
2. **"What if the product list came from another LWC that's not a direct child?"** — Use Lightning Message Service (LMS) for cross-component communication.
3. **"How would you animate item additions/removals?"** — Use CSS transitions/animations triggered by class toggling on each cart item element.

---

## Challenge 5: Custom Lookup Component

| Property | Value |
|----------|-------|
| **Difficulty** | 🟡 Medium |
| **Time Limit** | 35 minutes |
| **Key Concepts** | Debouncing, imperative Apex, SOSL, combobox pattern |
| **Common In** | Mid/Senior-level interviews |

### 📋 Problem Statement

Build a custom lookup (typeahead) component that searches for records as the user types. The component should support **debouncing** to avoid excessive API calls, display results in a dropdown, and allow the user to select a result.

**Requirements:**
- Search triggers after 300ms of no typing (debouncing)
- Minimum 2 characters before searching
- Display results in a dropdown list with name and secondary field
- Allow selection and clearing of selection
- Show a "pill" with the selected record name
- Support a configurable object type via `@api` property

### 💡 Hints

1. **Debounce pattern**: Store the timeout ID in a class property. In the input handler, clear the previous timeout with `clearTimeout`, then set a new one with `setTimeout`.
2. Use a `renderedCallback` or a click-outside handler to close the dropdown when the user clicks elsewhere.
3. Use SOSL in Apex for cross-field searching: `FIND :searchTerm IN ALL FIELDS RETURNING Account(Id, Name, Industry)`.

### 🏗️ Skeleton Code

```javascript
import { LightningElement, api } from 'lwc';
import search from '@salesforce/apex/LookupController.search';

export default class CustomLookup extends LightningElement {
    @api objectApiName = 'Account';
    @api label = 'Search';
    @api placeholder = 'Type to search...';

    searchTerm = '';
    searchResults = [];
    selectedRecord = null;
    showDropdown = false;
    isSearching = false;
    _debounceTimer;

    // TODO: handleInputChange with debouncing
    // TODO: handleSelect
    // TODO: handleClear
    // TODO: Close dropdown on outside click
}
```

<details>
<summary>💡 Complete Solution</summary>

**LookupController.cls**
```java
public with sharing class LookupController {
    @AuraEnabled
    public static List<SObject> search(String searchTerm, String objectApiName) {
        String searchKey = '%' + String.escapeSingleQuotes(searchTerm) + '%';
        String query = 'SELECT Id, Name FROM ' +
            String.escapeSingleQuotes(objectApiName) +
            ' WHERE Name LIKE :searchKey ORDER BY Name LIMIT 10';
        return Database.query(query);
    }
}
```

**customLookup.html**
```html
<template>
    <div class="slds-form-element">
        <label class="slds-form-element__label">{label}</label>
        <div class="slds-form-element__control">
            <!-- Selected Record Pill -->
            <template lwc:if={selectedRecord}>
                <lightning-pill
                    label={selectedRecord.Name}
                    onremove={handleClear}
                ></lightning-pill>
            </template>

            <!-- Search Input -->
            <template lwc:if={showInput}>
                <div class="slds-combobox_container">
                    <div class={comboboxClass} role="combobox" aria-expanded={showDropdown}>
                        <div class="slds-combobox__form-element slds-input-has-icon slds-input-has-icon_right">
                            <input
                                type="text"
                                class="slds-input slds-combobox__input"
                                placeholder={placeholder}
                                value={searchTerm}
                                oninput={handleInputChange}
                                onfocus={handleFocus}
                                role="textbox"
                                aria-autocomplete="list"
                            />
                            <template lwc:if={isSearching}>
                                <div class="slds-input__icon slds-input__icon_right">
                                    <lightning-spinner
                                        alternative-text="Searching"
                                        size="x-small"
                                    ></lightning-spinner>
                                </div>
                            </template>
                        </div>

                        <!-- Dropdown Results -->
                        <template lwc:if={showDropdown}>
                            <div class="slds-dropdown slds-dropdown_length-5 slds-dropdown_fluid" role="listbox">
                                <ul class="slds-listbox slds-listbox_vertical" role="presentation">
                                    <template lwc:if={hasResults}>
                                        <template for:each={searchResults} for:item="result">
                                            <li key={result.Id} role="presentation"
                                                class="slds-listbox__item">
                                                <div class="slds-media slds-listbox__option slds-listbox__option_plain slds-media_small"
                                                     data-record-id={result.Id}
                                                     onclick={handleSelect}
                                                     role="option">
                                                    <span class="slds-media__body">
                                                        <span class="slds-listbox__option-text slds-listbox__option-text_entity">
                                                            {result.Name}
                                                        </span>
                                                    </span>
                                                </div>
                                            </li>
                                        </template>
                                    </template>
                                    <template lwc:if={noResults}>
                                        <li class="slds-listbox__item">
                                            <span class="slds-media slds-listbox__option_plain slds-p-around_small">
                                                No results found for "{searchTerm}"
                                            </span>
                                        </li>
                                    </template>
                                </ul>
                            </div>
                        </template>
                    </div>
                </div>
            </template>
        </div>
    </div>
</template>
```

**customLookup.js**
```javascript
import { LightningElement, api } from 'lwc';
import search from '@salesforce/apex/LookupController.search';

const DEBOUNCE_DELAY = 300;
const MIN_SEARCH_LENGTH = 2;

export default class CustomLookup extends LightningElement {
    @api objectApiName = 'Account';
    @api label = 'Search';
    @api placeholder = 'Type to search...';

    searchTerm = '';
    searchResults = [];
    selectedRecord = null;
    showDropdown = false;
    isSearching = false;
    _debounceTimer;

    // Bound handler for outside clicks
    _outsideClickHandler = this.handleOutsideClick.bind(this);

    connectedCallback() {
        document.addEventListener('click', this._outsideClickHandler);
    }

    disconnectedCallback() {
        document.removeEventListener('click', this._outsideClickHandler);
    }

    handleInputChange(event) {
        this.searchTerm = event.target.value;

        // Clear previous timer
        clearTimeout(this._debounceTimer);

        if (this.searchTerm.length < MIN_SEARCH_LENGTH) {
            this.searchResults = [];
            this.showDropdown = false;
            return;
        }

        // Set new debounce timer
        this._debounceTimer = setTimeout(() => {
            this.performSearch();
        }, DEBOUNCE_DELAY);
    }

    async performSearch() {
        this.isSearching = true;
        try {
            this.searchResults = await search({
                searchTerm: this.searchTerm,
                objectApiName: this.objectApiName
            });
            this.showDropdown = true;
        } catch (error) {
            console.error('Search error:', error);
            this.searchResults = [];
        } finally {
            this.isSearching = false;
        }
    }

    handleSelect(event) {
        const recordId = event.currentTarget.dataset.recordId;
        this.selectedRecord = this.searchResults.find(r => r.Id === recordId);
        this.showDropdown = false;
        this.searchTerm = '';

        // Notify parent of selection
        this.dispatchEvent(new CustomEvent('select', {
            detail: { recordId: this.selectedRecord.Id, record: this.selectedRecord }
        }));
    }

    handleClear() {
        this.selectedRecord = null;
        this.searchTerm = '';
        this.searchResults = [];

        this.dispatchEvent(new CustomEvent('clear'));
    }

    handleFocus() {
        if (this.searchResults.length > 0) {
            this.showDropdown = true;
        }
    }

    handleOutsideClick(event) {
        if (!this.template.contains(event.target)) {
            this.showDropdown = false;
        }
    }

    get showInput() {
        return !this.selectedRecord;
    }

    get hasResults() {
        return this.searchResults.length > 0;
    }

    get noResults() {
        return this.searchResults.length === 0 && this.searchTerm.length >= MIN_SEARCH_LENGTH && !this.isSearching;
    }

    get comboboxClass() {
        return `slds-combobox slds-dropdown-trigger slds-dropdown-trigger_click ${
            this.showDropdown ? 'slds-is-open' : ''
        }`;
    }
}
```

</details>

### 📖 Explanation

The debounce pattern is the star of this challenge. Without it, typing "Salesforce" would fire 10 Apex calls (one per keystroke). With a 300ms debounce, only 1 call fires after the user pauses. The implementation clears any pending timeout before setting a new one — this ensures only the final keystroke triggers the search. The outside-click handler (`handleOutsideClick`) uses `this.template.contains()` to check if the click originated inside the component's shadow DOM. The pill pattern (`lightning-pill`) provides a familiar UX for showing the selected record with a remove option.

### 🔄 Follow-up Questions

1. **"How would you add keyboard navigation (arrow keys) to the dropdown?"** — Track a `focusedIndex`, increment/decrement on ArrowDown/ArrowUp, apply a highlight class, and select on Enter.
2. **"What are the security concerns with dynamic SOQL?"** — SQL injection via the `objectApiName` parameter. Use `String.escapeSingleQuotes()` and validate against an allowlist of objects.
3. **"How does the standard `lightning-record-picker` compare?"** — Available in API 59+, it handles lookup natively but has limited customization. Use it when standard behavior suffices.

---

## Challenge 6: Multi-Step Wizard Form

| Property | Value |
|----------|-------|
| **Difficulty** | 🟡 Medium |
| **Time Limit** | 40 minutes |
| **Key Concepts** | State management, validation, conditional rendering, @track |
| **Common In** | Mid/Senior-level interviews |

### 📋 Problem Statement

Build a 3-step wizard form for creating a new Account with related Contact. Each step collects different information, has validation, and the user can navigate forward/backward. On the final step, display a summary before submission.

**Steps:**
1. **Account Info** — Name (required), Industry (picklist), Phone
2. **Contact Info** — First Name, Last Name (required), Email (required, validated)
3. **Review & Submit** — Summary of all entered data with Edit buttons

**Requirements:**
- Progress indicator showing current step
- Validation on each step before allowing navigation to the next
- Back navigation preserves previously entered data
- Review step shows all data and allows going back to edit
- Submit calls an Apex method to create both records
- Success/error toast notifications

### 💡 Hints

1. Use `lightning-progress-indicator` with `lightning-progress-step` for the visual stepper.
2. Store form data in a single object (e.g., `formData = { accountName: '', industry: '', ... }`) to preserve data across steps.
3. Use `this.template.querySelector('lightning-input').reportValidity()` to trigger and check validation before navigating forward.

### 🏗️ Skeleton Code

```javascript
import { LightningElement } from 'lwc';
import createAccountWithContact from '@salesforce/apex/WizardController.createAccountWithContact';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class MultiStepWizard extends LightningElement {
    currentStep = 1;
    formData = {
        accountName: '', industry: '', phone: '',
        firstName: '', lastName: '', email: ''
    };

    // TODO: Step navigation (next, previous)
    // TODO: Validation per step
    // TODO: Conditional rendering getters (isStep1, isStep2, isStep3)
    // TODO: Submit handler
}
```

<details>
<summary>💡 Complete Solution</summary>

**WizardController.cls**
```java
public with sharing class WizardController {
    @AuraEnabled
    public static Id createAccountWithContact(
        String accountName, String industry, String phone,
        String firstName, String lastName, String email
    ) {
        Account acc = new Account(
            Name = accountName,
            Industry = industry,
            Phone = phone
        );
        insert acc;

        Contact con = new Contact(
            FirstName = firstName,
            LastName = lastName,
            Email = email,
            AccountId = acc.Id
        );
        insert con;

        return acc.Id;
    }
}
```

**multiStepWizard.html**
```html
<template>
    <lightning-card title="Create Account & Contact" icon-name="standard:account">
        <div class="slds-p-around_medium">
            <!-- Progress Indicator -->
            <lightning-progress-indicator current-step={currentStepStr} type="base">
                <lightning-progress-step label="Account Info" value="1"></lightning-progress-step>
                <lightning-progress-step label="Contact Info" value="2"></lightning-progress-step>
                <lightning-progress-step label="Review & Submit" value="3"></lightning-progress-step>
            </lightning-progress-indicator>

            <div class="slds-m-top_large">
                <!-- Step 1: Account Info -->
                <template lwc:if={isStep1}>
                    <div class="slds-grid slds-wrap slds-gutters">
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <lightning-input
                                label="Account Name"
                                value={formData.accountName}
                                onchange={handleFieldChange}
                                data-field="accountName"
                                required
                            ></lightning-input>
                        </div>
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <lightning-combobox
                                label="Industry"
                                value={formData.industry}
                                options={industryOptions}
                                onchange={handleFieldChange}
                                data-field="industry"
                            ></lightning-combobox>
                        </div>
                        <div class="slds-col slds-size_1-of-2">
                            <lightning-input
                                label="Phone"
                                type="tel"
                                value={formData.phone}
                                onchange={handleFieldChange}
                                data-field="phone"
                            ></lightning-input>
                        </div>
                    </div>
                </template>

                <!-- Step 2: Contact Info -->
                <template lwc:if={isStep2}>
                    <div class="slds-grid slds-wrap slds-gutters">
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <lightning-input
                                label="First Name"
                                value={formData.firstName}
                                onchange={handleFieldChange}
                                data-field="firstName"
                            ></lightning-input>
                        </div>
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <lightning-input
                                label="Last Name"
                                value={formData.lastName}
                                onchange={handleFieldChange}
                                data-field="lastName"
                                required
                            ></lightning-input>
                        </div>
                        <div class="slds-col slds-size_1-of-2">
                            <lightning-input
                                label="Email"
                                type="email"
                                value={formData.email}
                                onchange={handleFieldChange}
                                data-field="email"
                                required
                            ></lightning-input>
                        </div>
                    </div>
                </template>

                <!-- Step 3: Review & Submit -->
                <template lwc:if={isStep3}>
                    <div class="slds-grid slds-wrap slds-gutters">
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <div class="slds-box slds-theme_shade">
                                <h3 class="slds-text-heading_small slds-m-bottom_small">
                                    Account Details
                                    <lightning-button-icon
                                        icon-name="utility:edit"
                                        alternative-text="Edit Account"
                                        size="small"
                                        onclick={goToStep1}
                                        class="slds-float_right"
                                    ></lightning-button-icon>
                                </h3>
                                <p><strong>Name:</strong> {formData.accountName}</p>
                                <p><strong>Industry:</strong> {formData.industry}</p>
                                <p><strong>Phone:</strong> {formData.phone}</p>
                            </div>
                        </div>
                        <div class="slds-col slds-size_1-of-2 slds-m-bottom_medium">
                            <div class="slds-box slds-theme_shade">
                                <h3 class="slds-text-heading_small slds-m-bottom_small">
                                    Contact Details
                                    <lightning-button-icon
                                        icon-name="utility:edit"
                                        alternative-text="Edit Contact"
                                        size="small"
                                        onclick={goToStep2}
                                        class="slds-float_right"
                                    ></lightning-button-icon>
                                </h3>
                                <p><strong>Name:</strong> {formData.firstName} {formData.lastName}</p>
                                <p><strong>Email:</strong> {formData.email}</p>
                            </div>
                        </div>
                    </div>
                </template>
            </div>

            <!-- Navigation Buttons -->
            <div class="slds-grid slds-grid_align-spread slds-m-top_large slds-border_top slds-p-top_medium">
                <div>
                    <template lwc:if={showPrevious}>
                        <lightning-button
                            label="Previous"
                            icon-name="utility:back"
                            onclick={handlePrevious}
                        ></lightning-button>
                    </template>
                </div>
                <div>
                    <template lwc:if={showNext}>
                        <lightning-button
                            label="Next"
                            variant="brand"
                            icon-name="utility:forward"
                            icon-position="right"
                            onclick={handleNext}
                        ></lightning-button>
                    </template>
                    <template lwc:if={isStep3}>
                        <lightning-button
                            label="Submit"
                            variant="success"
                            icon-name="utility:check"
                            icon-position="right"
                            onclick={handleSubmit}
                            disabled={isSubmitting}
                        ></lightning-button>
                    </template>
                </div>
            </div>
        </div>
    </lightning-card>
</template>
```

**multiStepWizard.js**
```javascript
import { LightningElement } from 'lwc';
import createAccountWithContact from '@salesforce/apex/WizardController.createAccountWithContact';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class MultiStepWizard extends LightningElement {
    currentStep = 1;
    isSubmitting = false;

    formData = {
        accountName: '',
        industry: '',
        phone: '',
        firstName: '',
        lastName: '',
        email: ''
    };

    industryOptions = [
        { label: '-- None --', value: '' },
        { label: 'Technology', value: 'Technology' },
        { label: 'Finance', value: 'Finance' },
        { label: 'Healthcare', value: 'Healthcare' },
        { label: 'Manufacturing', value: 'Manufacturing' },
        { label: 'Retail', value: 'Retail' }
    ];

    handleFieldChange(event) {
        const field = event.target.dataset.field;
        // Immutable update to trigger reactivity
        this.formData = {
            ...this.formData,
            [field]: event.detail.value ?? event.target.value
        };
    }

    handleNext() {
        if (this.validateCurrentStep()) {
            this.currentStep++;
        }
    }

    handlePrevious() {
        this.currentStep--;
    }

    goToStep1() {
        this.currentStep = 1;
    }

    goToStep2() {
        this.currentStep = 2;
    }

    validateCurrentStep() {
        const inputs = this.template.querySelectorAll('lightning-input, lightning-combobox');
        let isValid = true;

        inputs.forEach(input => {
            if (!input.reportValidity()) {
                isValid = false;
            }
        });

        return isValid;
    }

    async handleSubmit() {
        this.isSubmitting = true;
        try {
            const accountId = await createAccountWithContact({
                accountName: this.formData.accountName,
                industry: this.formData.industry,
                phone: this.formData.phone,
                firstName: this.formData.firstName,
                lastName: this.formData.lastName,
                email: this.formData.email
            });

            this.dispatchEvent(new ShowToastEvent({
                title: 'Success!',
                message: `Account created successfully (Id: ${accountId})`,
                variant: 'success'
            }));

            this.resetForm();
        } catch (error) {
            this.dispatchEvent(new ShowToastEvent({
                title: 'Error',
                message: error.body?.message || 'An error occurred',
                variant: 'error'
            }));
        } finally {
            this.isSubmitting = false;
        }
    }

    resetForm() {
        this.currentStep = 1;
        this.formData = {
            accountName: '', industry: '', phone: '',
            firstName: '', lastName: '', email: ''
        };
    }

    // Getters for step visibility
    get isStep1() { return this.currentStep === 1; }
    get isStep2() { return this.currentStep === 2; }
    get isStep3() { return this.currentStep === 3; }

    get currentStepStr() { return String(this.currentStep); }

    get showPrevious() { return this.currentStep > 1; }
    get showNext() { return this.currentStep < 3; }
}
```

</details>

### 📖 Explanation

The wizard pattern uses a single `currentStep` integer to control which step is visible via conditional rendering. The form data lives in a single `formData` object, which persists across step navigations. The **immutable update** (`this.formData = { ...this.formData, [field]: value }`) is essential because LWC's reactivity doesn't deep-track object properties. Validation uses `reportValidity()`, which is a built-in method on `lightning-input` that both checks validity AND shows the error messages to the user. The `querySelectorAll` approach validates all inputs on the current step at once.

### 🔄 Follow-up Questions

1. **"How would you handle browser back/forward buttons?"** — Use the History API (`window.history.pushState`) to map each step to a URL state and listen for `popstate` events.
2. **"What if Step 2 depended on data from Step 1 (e.g., Industry-specific fields)?"** — Add conditional rendering inside Step 2 based on `formData.industry`, loading dynamic picklist values via Apex.
3. **"How would you save draft progress?"** — Use `localStorage` to persist `formData` on each change and restore it in `connectedCallback`.

---

## Challenge 7: Sortable & Filterable List

| Property | Value |
|----------|-------|
| **Difficulty** | 🟡 Medium |
| **Time Limit** | 35 minutes |
| **Key Concepts** | Array manipulation, computed properties, sorting algorithms |
| **Common In** | Mid-level interviews |

### 📋 Problem Statement

Build a component that displays a list of opportunities with the ability to sort by multiple columns (Name, Amount, Stage, Close Date) and filter by Stage. The sort should toggle between ascending and descending, and the column header should indicate the current sort direction.

**Requirements:**
- Sortable columns: Name, Amount, Stage, Close Date
- Filter by Stage (multi-select)
- Sort direction indicator (▲/▼) on active column
- Combined sort + filter (both applied simultaneously)
- Reset all filters button
- Record count display

### 💡 Hints

1. Keep `sortField` and `sortDirection` as separate reactive properties. Your getter should apply both filtering AND sorting in sequence.
2. For date sorting, compare using `new Date(a).getTime()` to handle string date comparisons correctly.
3. Use `lightning-dual-listbox` or checkboxes for multi-select stage filter.

### 🏗️ Skeleton Code

```javascript
import { LightningElement, wire } from 'lwc';
import getOpportunities from '@salesforce/apex/OpportunityController.getOpportunities';

export default class SortableFilterableList extends LightningElement {
    allRecords = [];
    sortField = 'Name';
    sortDirection = 'asc';
    selectedStages = [];

    // TODO: Getter for filtered + sorted records
    // TODO: Sort handler
    // TODO: Filter handler
    // TODO: Column header with sort indicator
}
```

<details>
<summary>💡 Complete Solution</summary>

**sortableFilterableList.html**
```html
<template>
    <lightning-card title="Opportunities" icon-name="standard:opportunity">
        <div slot="actions">
            <lightning-button
                label="Reset Filters"
                icon-name="utility:refresh"
                onclick={handleReset}
            ></lightning-button>
        </div>

        <div class="slds-p-around_medium">
            <!-- Stage Filter -->
            <div class="slds-m-bottom_medium">
                <lightning-checkbox-group
                    label="Filter by Stage"
                    options={stageOptions}
                    value={selectedStages}
                    onchange={handleStageFilter}
                ></lightning-checkbox-group>
            </div>

            <!-- Record Count -->
            <p class="slds-m-bottom_small slds-text-body_small slds-text-color_weak">
                {recordCountLabel}
            </p>

            <!-- Sortable Table -->
            <table class="slds-table slds-table_cell-buffer slds-table_bordered slds-table_striped">
                <thead>
                    <tr class="slds-line-height_reset">
                        <template for:each={columns} for:item="col">
                            <th key={col.field}
                                class="slds-is-sortable"
                                scope="col"
                                onclick={handleSort}
                                data-field={col.field}
                                style="cursor: pointer;">
                                <span>{col.label}</span>
                                <template lwc:if={col.isSorted}>
                                    <span class="slds-m-left_xx-small">
                                        {col.sortIcon}
                                    </span>
                                </template>
                            </th>
                        </template>
                    </tr>
                </thead>
                <tbody>
                    <template for:each={displayRecords} for:item="record">
                        <tr key={record.Id}>
                            <td>{record.Name}</td>
                            <td>{record.formattedAmount}</td>
                            <td>
                                <lightning-badge label={record.StageName}></lightning-badge>
                            </td>
                            <td>{record.formattedCloseDate}</td>
                        </tr>
                    </template>
                    <template lwc:if={noRecords}>
                        <tr>
                            <td colspan="4" class="slds-text-align_center slds-p-around_large">
                                No records match your criteria.
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>
    </lightning-card>
</template>
```

**sortableFilterableList.js**
```javascript
import { LightningElement, wire } from 'lwc';
import getOpportunities from '@salesforce/apex/OpportunityController.getOpportunities';

export default class SortableFilterableList extends LightningElement {
    allRecords = [];
    error;
    sortField = 'Name';
    sortDirection = 'asc';
    selectedStages = [];

    stageOptions = [
        { label: 'Prospecting', value: 'Prospecting' },
        { label: 'Qualification', value: 'Qualification' },
        { label: 'Proposal', value: 'Proposal/Price Quote' },
        { label: 'Negotiation', value: 'Negotiation/Review' },
        { label: 'Closed Won', value: 'Closed Won' },
        { label: 'Closed Lost', value: 'Closed Lost' }
    ];

    @wire(getOpportunities)
    wiredOpportunities({ error, data }) {
        if (data) {
            this.allRecords = data.map(record => ({
                ...record,
                formattedAmount: record.Amount
                    ? new Intl.NumberFormat('en-US', {
                          style: 'currency', currency: 'USD'
                      }).format(record.Amount)
                    : '—',
                formattedCloseDate: record.CloseDate || '—'
            }));
            this.error = undefined;
        } else if (error) {
            this.error = error;
            this.allRecords = [];
        }
    }

    get columns() {
        const cols = [
            { field: 'Name', label: 'Name' },
            { field: 'Amount', label: 'Amount' },
            { field: 'StageName', label: 'Stage' },
            { field: 'CloseDate', label: 'Close Date' }
        ];
        return cols.map(col => ({
            ...col,
            isSorted: col.field === this.sortField,
            sortIcon: col.field === this.sortField
                ? (this.sortDirection === 'asc' ? '▲' : '▼')
                : ''
        }));
    }

    get displayRecords() {
        let records = [...this.allRecords];

        // Apply stage filter
        if (this.selectedStages.length > 0) {
            records = records.filter(r =>
                this.selectedStages.includes(r.StageName)
            );
        }

        // Apply sort
        records.sort((a, b) => {
            let valA = a[this.sortField] || '';
            let valB = b[this.sortField] || '';

            // Handle numeric sorting for Amount
            if (this.sortField === 'Amount') {
                valA = Number(valA) || 0;
                valB = Number(valB) || 0;
            }
            // Handle date sorting
            else if (this.sortField === 'CloseDate') {
                valA = new Date(valA).getTime() || 0;
                valB = new Date(valB).getTime() || 0;
            }
            // Default string sorting (case-insensitive)
            else {
                valA = String(valA).toLowerCase();
                valB = String(valB).toLowerCase();
            }

            let comparison = 0;
            if (valA > valB) comparison = 1;
            if (valA < valB) comparison = -1;

            return this.sortDirection === 'asc' ? comparison : -comparison;
        });

        return records;
    }

    get recordCountLabel() {
        return `Showing ${this.displayRecords.length} of ${this.allRecords.length} records`;
    }

    get noRecords() {
        return this.displayRecords.length === 0 && this.allRecords.length > 0;
    }

    handleSort(event) {
        const field = event.currentTarget.dataset.field;
        if (this.sortField === field) {
            // Toggle direction
            this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            this.sortField = field;
            this.sortDirection = 'asc';
        }
    }

    handleStageFilter(event) {
        this.selectedStages = event.detail.value;
    }

    handleReset() {
        this.selectedStages = [];
        this.sortField = 'Name';
        this.sortDirection = 'asc';
    }
}
```

</details>

### 📖 Explanation

The `displayRecords` getter is the core of this solution — it applies **both filter and sort in a pipeline**. First, it filters by selected stages, then sorts the filtered array. This getter recomputes whenever any of its dependent reactive properties (`allRecords`, `selectedStages`, `sortField`, `sortDirection`) change. The sort handler cleverly toggles direction when clicking the same column and resets to ascending for new columns. Type-aware sorting (numbers vs dates vs strings) is crucial — without it, "100" would sort before "20" in string comparison.

### 🔄 Follow-up Questions

1. **"How would you implement multi-column sort (sort by Stage, then by Amount within each stage)?"** — Maintain an array of sort criteria `[{ field, direction }]` and chain comparisons in the sort function.
2. **"Would you use `lightning-datatable` sorting instead?"** — Yes, for production code. `lightning-datatable` has built-in `onsort` that provides `fieldName` and `sortDirection`, reducing boilerplate.
3. **"How would you handle 10,000+ records?"** — Move sorting and filtering server-side via dynamic SOQL, or implement virtual scrolling.

---

## Challenge 8: Toast Notification System

| Property | Value |
|----------|-------|
| **Difficulty** | 🟢 Easy |
| **Time Limit** | 15 minutes |
| **Key Concepts** | ShowToastEvent, utility modules, event dispatching |
| **Common In** | Junior interviews, often a warm-up question |

### 📋 Problem Statement

Create a reusable toast notification utility module that other components can import and use to show consistent toast messages throughout the application. The utility should support success, error, warning, and info toasts with a clean API.

**Requirements:**
- Create a utility module (not a component) that exports toast helper functions
- Support all four variants: success, error, warning, info
- Allow customizable title, message, and mode (dismissible, pester, sticky)
- Provide a convenience method for showing Apex error messages
- Demo component that uses the utility

### 💡 Hints

1. Create a **service module** (a JS file without HTML/CSS) that exports functions. Other components `import` these functions.
2. `ShowToastEvent` must be dispatched from a component's `this`, so the utility functions need to accept the component reference or return the event for the caller to dispatch.
3. Use `error.body.message` for Apex errors and `error.message` for JavaScript errors.

### 🏗️ Skeleton Code

```javascript
// toastService.js (utility module - no HTML template)
// TODO: Export helper functions for toast variants
// TODO: Export Apex error handler
```

<details>
<summary>💡 Complete Solution</summary>

**toastService.js** (create as `force-app/main/default/lwc/toastService/toastService.js`)
```javascript
/**
 * Reusable Toast Notification Service
 * Usage: import { showSuccess, showError } from 'c/toastService';
 */
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

/**
 * Shows a success toast notification.
 * @param {LightningElement} component - The component dispatching the event
 * @param {string} title - Toast title
 * @param {string} message - Toast message
 * @param {string} mode - 'dismissible' (default), 'pester', or 'sticky'
 */
export function showSuccess(component, title, message = '', mode = 'dismissible') {
    component.dispatchEvent(new ShowToastEvent({
        title,
        message,
        variant: 'success',
        mode
    }));
}

export function showError(component, title, message = '', mode = 'sticky') {
    component.dispatchEvent(new ShowToastEvent({
        title,
        message,
        variant: 'error',
        mode
    }));
}

export function showWarning(component, title, message = '', mode = 'dismissible') {
    component.dispatchEvent(new ShowToastEvent({
        title,
        message,
        variant: 'warning',
        mode
    }));
}

export function showInfo(component, title, message = '', mode = 'dismissible') {
    component.dispatchEvent(new ShowToastEvent({
        title,
        message,
        variant: 'info',
        mode
    }));
}

/**
 * Extracts and displays a user-friendly error message from various error shapes.
 * Handles Apex errors, wire errors, and generic JS errors.
 */
export function showApexError(component, error, title = 'Error') {
    let message = 'An unexpected error occurred.';

    if (typeof error === 'string') {
        message = error;
    } else if (error?.body?.message) {
        // Standard Apex error
        message = error.body.message;
    } else if (error?.body?.fieldErrors) {
        // Field-level errors
        message = Object.values(error.body.fieldErrors)
            .flat()
            .map(e => e.message)
            .join(', ');
    } else if (error?.body?.pageErrors) {
        // Page-level errors
        message = error.body.pageErrors.map(e => e.message).join(', ');
    } else if (error?.message) {
        // Generic JS error
        message = error.message;
    } else if (Array.isArray(error)) {
        message = error.map(e => e.message).join(', ');
    }

    showError(component, title, message);
}

/**
 * Generic toast function for full customization.
 */
export function showToast(component, { title, message, variant, mode, messageData }) {
    component.dispatchEvent(new ShowToastEvent({
        title,
        message,
        variant: variant || 'info',
        mode: mode || 'dismissible',
        messageData
    }));
}
```

**toastService.js-meta.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <isExposed>false</isExposed>
</LightningComponentBundle>
```

**Usage — toastDemo.js**
```javascript
import { LightningElement } from 'lwc';
import { showSuccess, showError, showWarning, showInfo, showApexError } from 'c/toastService';
import saveRecord from '@salesforce/apex/DemoController.saveRecord';

export default class ToastDemo extends LightningElement {
    async handleSave() {
        try {
            await saveRecord({ /* params */ });
            showSuccess(this, 'Record Saved', 'The record was saved successfully.');
        } catch (error) {
            showApexError(this, error, 'Save Failed');
        }
    }

    handleWarn() {
        showWarning(this, 'Heads Up', 'This action cannot be undone.');
    }

    handleInfo() {
        showInfo(this, 'Did you know?', 'You can customize your dashboard layout.');
    }
}
```

</details>

### 📖 Explanation

A **service module** in LWC is a JavaScript file that exports functions/constants but has no HTML template — it cannot be placed on a page directly. This is the recommended pattern for shared utilities. The `showApexError` function is especially valuable because Apex errors come in different shapes (body.message, fieldErrors, pageErrors), and this function normalizes them all. The `component` parameter is necessary because `ShowToastEvent` must be dispatched from a DOM-connected component — a standalone module has no DOM context.

### 🔄 Follow-up Questions

1. **"Can toasts be shown in Experience Cloud (Community) pages?"** — No, `ShowToastEvent` only works in Lightning Experience and the Salesforce mobile app. For communities, you'd build a custom toast component.
2. **"How would you unit test this service module?"** — Import the functions in a Jest test, mock `ShowToastEvent`, pass a mock component with `dispatchEvent`, and assert the event was dispatched with the correct parameters.
3. **"What's the difference between dismissible, pester, and sticky modes?"** — Dismissible shows a close button and auto-dismisses after 3s, pester auto-dismisses without a close button, sticky requires manual dismissal.

---

## Challenge 9: Lazy Loading Component (Infinite Scroll)

| Property | Value |
|----------|-------|
| **Difficulty** | 🔴 Hard |
| **Time Limit** | 45 minutes |
| **Key Concepts** | IntersectionObserver, imperative Apex, performance, virtual DOM |
| **Common In** | Senior-level interviews |

### 📋 Problem Statement

Build an infinite scroll component that loads records in batches as the user scrolls down. The component should detect when the user reaches the bottom of the list and automatically fetch the next batch. Include a loading indicator and "end of list" message.

**Requirements:**
- Load 20 records at a time
- Use `IntersectionObserver` to detect when the sentinel element is visible
- Show a loading spinner during fetch
- Display "No more records" when all data is loaded
- Prevent duplicate fetch requests while a request is in flight
- Support pull-to-refresh functionality

### 💡 Hints

1. Place a hidden **sentinel element** at the bottom of the list. When `IntersectionObserver` detects it entering the viewport, trigger the next data load.
2. Set up the `IntersectionObserver` in `renderedCallback` (not `connectedCallback`) because the sentinel element needs to exist in the DOM first.
3. Use a `_isLoadingMore` flag to prevent concurrent Apex calls when the user scrolls rapidly.

### 🏗️ Skeleton Code

```javascript
import { LightningElement } from 'lwc';
import getRecordsBatch from '@salesforce/apex/InfiniteScrollController.getRecordsBatch';

export default class InfiniteScroll extends LightningElement {
    records = [];
    offset = 0;
    batchSize = 20;
    hasMore = true;
    isLoading = false;
    _observer;

    // TODO: Set up IntersectionObserver
    // TODO: loadMore method
    // TODO: Clean up observer on disconnect
}
```

<details>
<summary>💡 Complete Solution</summary>

**InfiniteScrollController.cls**
```java
public with sharing class InfiniteScrollController {
    @AuraEnabled
    public static List<Contact> getRecordsBatch(Integer batchSize, Integer offset) {
        return [
            SELECT Id, Name, Email, Phone, Title, Account.Name
            FROM Contact
            ORDER BY Name
            LIMIT :batchSize
            OFFSET :offset
        ];
    }
}
```

**infiniteScroll.html**
```html
<template>
    <lightning-card title="Contacts (Infinite Scroll)" icon-name="standard:contact">
        <div slot="actions">
            <lightning-button-icon
                icon-name="utility:refresh"
                alternative-text="Refresh"
                onclick={handleRefresh}
            ></lightning-button-icon>
        </div>

        <div class="scroll-container slds-p-around_medium" style="max-height: 600px; overflow-y: auto;">
            <!-- Record List -->
            <template for:each={records} for:item="record">
                <div key={record.Id} class="slds-box slds-m-bottom_xx-small slds-theme_default record-item">
                    <div class="slds-grid slds-gutters">
                        <div class="slds-col slds-size_1-of-3">
                            <p class="slds-text-heading_small">{record.Name}</p>
                            <p class="slds-text-body_small slds-text-color_weak">{record.Title}</p>
                        </div>
                        <div class="slds-col slds-size_1-of-3">
                            <p class="slds-text-body_small">📧 {record.Email}</p>
                        </div>
                        <div class="slds-col slds-size_1-of-3">
                            <p class="slds-text-body_small">🏢 {record.accountName}</p>
                        </div>
                    </div>
                </div>
            </template>

            <!-- Loading Spinner -->
            <template lwc:if={isLoading}>
                <div class="slds-align_absolute-center slds-p-around_medium">
                    <lightning-spinner alternative-text="Loading more..." size="small"></lightning-spinner>
                    <span class="slds-m-left_small slds-text-color_weak">Loading more records...</span>
                </div>
            </template>

            <!-- End of List -->
            <template lwc:if={showEndMessage}>
                <div class="slds-align_absolute-center slds-p-around_medium">
                    <p class="slds-text-color_weak">✅ All {records.length} records loaded</p>
                </div>
            </template>

            <!-- Sentinel Element (invisible trigger for IntersectionObserver) -->
            <template lwc:if={hasMore}>
                <div class="scroll-sentinel" data-id="sentinel"></div>
            </template>
        </div>

        <!-- Record Count Footer -->
        <div slot="footer">
            <p class="slds-text-body_small">{records.length} records loaded</p>
        </div>
    </lightning-card>
</template>
```

**infiniteScroll.js**
```javascript
import { LightningElement } from 'lwc';
import getRecordsBatch from '@salesforce/apex/InfiniteScrollController.getRecordsBatch';

export default class InfiniteScroll extends LightningElement {
    records = [];
    offset = 0;
    batchSize = 20;
    hasMore = true;
    isLoading = false;
    _observer;
    _observerInitialized = false;

    connectedCallback() {
        this.loadMore();
    }

    renderedCallback() {
        // Set up IntersectionObserver once the sentinel is in the DOM
        if (!this._observerInitialized && this.hasMore) {
            this.initializeObserver();
        }
    }

    disconnectedCallback() {
        this.destroyObserver();
    }

    initializeObserver() {
        const sentinel = this.template.querySelector('.scroll-sentinel');
        if (!sentinel) return;

        const scrollContainer = this.template.querySelector('.scroll-container');

        this._observer = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting && !this.isLoading && this.hasMore) {
                        this.loadMore();
                    }
                });
            },
            {
                root: scrollContainer, // Observe within the scroll container
                rootMargin: '0px 0px 200px 0px', // Trigger 200px before reaching bottom
                threshold: 0.1
            }
        );

        this._observer.observe(sentinel);
        this._observerInitialized = true;
    }

    destroyObserver() {
        if (this._observer) {
            this._observer.disconnect();
            this._observer = null;
            this._observerInitialized = false;
        }
    }

    async loadMore() {
        if (this.isLoading || !this.hasMore) return;

        this.isLoading = true;
        try {
            const newRecords = await getRecordsBatch({
                batchSize: this.batchSize,
                offset: this.offset
            });

            if (newRecords.length < this.batchSize) {
                this.hasMore = false;
                this.destroyObserver();
            }

            // Map records to add computed fields
            const mapped = newRecords.map(record => ({
                ...record,
                accountName: record.Account ? record.Account.Name : '—'
            }));

            // Append to existing records (immutable)
            this.records = [...this.records, ...mapped];
            this.offset += newRecords.length;

        } catch (error) {
            console.error('Error loading records:', error);
        } finally {
            this.isLoading = false;
        }
    }

    async handleRefresh() {
        this.destroyObserver();
        this.records = [];
        this.offset = 0;
        this.hasMore = true;
        this._observerInitialized = false;

        await this.loadMore();
        // Re-initialize observer after the DOM re-renders
        // renderedCallback will handle this
    }

    get showEndMessage() {
        return !this.hasMore && this.records.length > 0;
    }
}
```

**infiniteScroll.css**
```css
.scroll-sentinel {
    height: 1px;
    width: 100%;
}

.record-item {
    transition: opacity 0.3s ease-in;
    animation: fadeIn 0.3s ease-in;
}

@keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
}
```

</details>

### 📖 Explanation

`IntersectionObserver` is a browser API that asynchronously watches for an element entering or leaving the viewport (or a parent container). It's far more performant than scroll event listeners because it doesn't fire on every pixel scrolled. The **sentinel pattern** places an invisible element at the bottom of the list — when the observer detects it's visible, it means the user has scrolled near the bottom. The `rootMargin: '0px 0px 200px 0px'` extends the detection zone 200px below the visible area, so data starts loading *before* the user reaches the actual bottom (pre-fetching for smoother UX). The `isLoading` guard prevents concurrent requests that could cause duplicate data.

### 🔄 Follow-up Questions

1. **"What are the SOQL OFFSET limits and how would you handle them?"** — OFFSET max is 2,000. For deeper pagination, switch to keyset/cursor-based: `WHERE Id > :lastId ORDER BY Id LIMIT :batchSize`.
2. **"How would you implement virtual scrolling for 100K+ records?"** — Only render items visible in the viewport plus a buffer. Calculate which items are visible based on scroll position and item height, and render only those DOM elements.
3. **"How does `IntersectionObserver` differ from a scroll event listener?"** — `IntersectionObserver` runs asynchronously off the main thread, doesn't need debouncing, and is much more performant. Scroll listeners fire on every scroll pixel and can cause jank.

---

## Challenge 10: Real-Time Dashboard with Platform Events

| Property | Value |
|----------|-------|
| **Difficulty** | 🔴 Hard |
| **Time Limit** | 45 minutes |
| **Key Concepts** | Platform Events, Emp API, reactive data, real-time updates |
| **Common In** | Senior/Architect-level interviews |

### 📋 Problem Statement

Build a real-time dashboard that displays live metrics (new Cases created, Opportunity stage changes, etc.) using Salesforce **Platform Events**. The dashboard should subscribe to a custom platform event and update the UI in real-time without page refreshes.

**Requirements:**
- Create a custom Platform Event (`Dashboard_Update__e`)
- Subscribe to the event using the `lightning/empApi` module
- Display a live feed of recent events (last 20)
- Show aggregate metrics (counts by type) that update in real-time
- Include subscribe/unsubscribe toggle
- Handle subscription errors and reconnection
- Visual indicator for new incoming events (highlight animation)

### 💡 Hints

1. Use `import { subscribe, unsubscribe, onError, setDebugFlag } from 'lightning/empApi'` for platform event subscription.
2. The channel name format for platform events is `/event/Dashboard_Update__e`.
3. Call `subscribe` in `connectedCallback` and `unsubscribe` in `disconnectedCallback` to manage the subscription lifecycle.

### 🏗️ Skeleton Code

```javascript
import { LightningElement } from 'lwc';
import { subscribe, unsubscribe, onError, setDebugFlag } from 'lightning/empApi';

export default class RealTimeDashboard extends LightningElement {
    channelName = '/event/Dashboard_Update__e';
    subscription = null;
    isSubscribed = false;
    events = [];
    metrics = {};

    // TODO: Subscribe to platform event
    // TODO: Handle incoming messages
    // TODO: Update metrics
    // TODO: Unsubscribe on disconnect
}
```

<details>
<summary>💡 Complete Solution</summary>

**Platform Event Definition** (deploy via metadata):
```xml
<!-- Dashboard_Update__e.object-meta.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<CustomObject xmlns="http://soap.sforce.com/2006/04/metadata">
    <deploymentStatus>Deployed</deploymentStatus>
    <eventType>HighVolume</eventType>
    <label>Dashboard Update</label>
    <pluralLabel>Dashboard Updates</pluralLabel>
    <fields>
        <fullName>Event_Type__c</fullName>
        <label>Event Type</label>
        <length>100</length>
        <type>Text</type>
    </fields>
    <fields>
        <fullName>Record_Name__c</fullName>
        <label>Record Name</label>
        <length>255</length>
        <type>Text</type>
    </fields>
    <fields>
        <fullName>User_Name__c</fullName>
        <label>User Name</label>
        <length>255</length>
        <type>Text</type>
    </fields>
    <fields>
        <fullName>Details__c</fullName>
        <label>Details</label>
        <length>5000</length>
        <type>LongTextArea</type>
        <visibleLines>3</visibleLines>
    </fields>
</CustomObject>
```

**Publishing Platform Event (Apex Trigger Example):**
```java
// Trigger on Case to publish platform events
trigger CaseEventTrigger on Case (after insert) {
    List<Dashboard_Update__e> events = new List<Dashboard_Update__e>();

    for (Case c : Trigger.new) {
        events.add(new Dashboard_Update__e(
            Event_Type__c = 'New Case',
            Record_Name__c = c.Subject,
            User_Name__c = UserInfo.getName(),
            Details__c = 'Priority: ' + c.Priority + ', Status: ' + c.Status
        ));
    }

    if (!events.isEmpty()) {
        EventBus.publish(events);
    }
}
```

**realTimeDashboard.html**
```html
<template>
    <lightning-card title="Real-Time Dashboard" icon-name="standard:dashboard">
        <div slot="actions">
            <lightning-button
                label={subscriptionButtonLabel}
                variant={subscriptionButtonVariant}
                icon-name={subscriptionIcon}
                onclick={toggleSubscription}
            ></lightning-button>
        </div>

        <div class="slds-p-around_medium">
            <!-- Connection Status -->
            <div class="slds-m-bottom_medium">
                <lightning-badge
                    label={connectionStatus}
                    class={connectionBadgeClass}
                ></lightning-badge>
            </div>

            <!-- Metric Cards -->
            <div class="slds-grid slds-gutters slds-m-bottom_large">
                <template for:each={metricCards} for:item="metric">
                    <div key={metric.type} class="slds-col slds-size_1-of-4">
                        <div class="slds-box slds-text-align_center slds-theme_shade">
                            <p class="slds-text-heading_large">{metric.count}</p>
                            <p class="slds-text-body_small slds-text-color_weak">{metric.type}</p>
                        </div>
                    </div>
                </template>
            </div>

            <!-- Live Event Feed -->
            <h3 class="slds-text-heading_small slds-m-bottom_small">
                Live Feed (Last 20 Events)
            </h3>

            <div class="feed-container" style="max-height: 400px; overflow-y: auto;">
                <template lwc:if={hasEvents}>
                    <template for:each={events} for:item="event">
                        <div key={event.id} class={event.cssClass}>
                            <div class="slds-grid slds-gutters">
                                <div class="slds-col slds-size_1-of-6">
                                    <lightning-badge label={event.type} class="slds-m-right_small"></lightning-badge>
                                </div>
                                <div class="slds-col slds-size_2-of-6">
                                    <p class="slds-text-body_regular"><strong>{event.recordName}</strong></p>
                                </div>
                                <div class="slds-col slds-size_2-of-6">
                                    <p class="slds-text-body_small slds-text-color_weak">{event.details}</p>
                                </div>
                                <div class="slds-col slds-size_1-of-6 slds-text-align_right">
                                    <p class="slds-text-body_small slds-text-color_weak">{event.timestamp}</p>
                                    <p class="slds-text-body_small">{event.userName}</p>
                                </div>
                            </div>
                        </div>
                    </template>
                </template>
                <template lwc:if={noEvents}>
                    <div class="slds-align_absolute-center slds-p-around_large">
                        <p class="slds-text-color_weak">
                            {emptyMessage}
                        </p>
                    </div>
                </template>
            </div>
        </div>
    </lightning-card>
</template>
```

**realTimeDashboard.js**
```javascript
import { LightningElement } from 'lwc';
import { subscribe, unsubscribe, onError, setDebugFlag } from 'lightning/empApi';

const MAX_EVENTS = 20;

export default class RealTimeDashboard extends LightningElement {
    channelName = '/event/Dashboard_Update__e';
    subscription = null;
    isSubscribed = false;
    events = [];
    metrics = {};
    _eventCounter = 0;

    connectedCallback() {
        // Enable debug logging for Emp API
        setDebugFlag(true);

        // Register global error handler
        this.registerErrorListener();

        // Auto-subscribe on load
        this.handleSubscribe();
    }

    disconnectedCallback() {
        this.handleUnsubscribe();
    }

    registerErrorListener() {
        onError(error => {
            console.error('Emp API error:', JSON.stringify(error));
            this.isSubscribed = false;

            // Attempt reconnection after 5 seconds
            // eslint-disable-next-line @lwc/lwc/no-async-operation
            setTimeout(() => {
                if (!this.isSubscribed) {
                    console.log('Attempting reconnection...');
                    this.handleSubscribe();
                }
            }, 5000);
        });
    }

    handleSubscribe() {
        if (this.isSubscribed) return;

        const messageCallback = (response) => {
            this.handleMessage(response);
        };

        subscribe(this.channelName, -1, messageCallback).then(response => {
            this.subscription = response;
            this.isSubscribed = true;
            console.log('Subscribed to:', JSON.stringify(response.channel));
        }).catch(error => {
            console.error('Subscribe error:', error);
        });
    }

    handleUnsubscribe() {
        if (!this.isSubscribed || !this.subscription) return;

        unsubscribe(this.subscription, response => {
            this.isSubscribed = false;
            this.subscription = null;
            console.log('Unsubscribed:', JSON.stringify(response));
        });
    }

    toggleSubscription() {
        if (this.isSubscribed) {
            this.handleUnsubscribe();
        } else {
            this.handleSubscribe();
        }
    }

    handleMessage(response) {
        const payload = response.data.payload;

        const newEvent = {
            id: `evt-${++this._eventCounter}`,
            type: payload.Event_Type__c,
            recordName: payload.Record_Name__c,
            userName: payload.User_Name__c,
            details: payload.Details__c,
            timestamp: new Date().toLocaleTimeString(),
            cssClass: 'slds-box slds-m-bottom_xx-small event-new', // animation class
            replayId: response.data.event.replayId
        };

        // Prepend new event and limit to MAX_EVENTS
        this.events = [newEvent, ...this.events].slice(0, MAX_EVENTS);

        // Update metrics
        this.updateMetrics(newEvent.type);

        // Remove animation class after 2 seconds
        // eslint-disable-next-line @lwc/lwc/no-async-operation
        setTimeout(() => {
            this.events = this.events.map(evt =>
                evt.id === newEvent.id
                    ? { ...evt, cssClass: 'slds-box slds-m-bottom_xx-small' }
                    : evt
            );
        }, 2000);
    }

    updateMetrics(eventType) {
        const updated = { ...this.metrics };
        updated[eventType] = (updated[eventType] || 0) + 1;
        this.metrics = updated;
    }

    get metricCards() {
        return Object.entries(this.metrics).map(([type, count]) => ({
            type,
            count
        }));
    }

    get hasEvents() {
        return this.events.length > 0;
    }

    get noEvents() {
        return this.events.length === 0;
    }

    get emptyMessage() {
        return this.isSubscribed
            ? 'Waiting for events... 📡'
            : 'Subscribe to start receiving events.';
    }

    get subscriptionButtonLabel() {
        return this.isSubscribed ? 'Unsubscribe' : 'Subscribe';
    }

    get subscriptionButtonVariant() {
        return this.isSubscribed ? 'destructive' : 'success';
    }

    get subscriptionIcon() {
        return this.isSubscribed ? 'utility:stop' : 'utility:play';
    }

    get connectionStatus() {
        return this.isSubscribed ? '🟢 Connected' : '🔴 Disconnected';
    }

    get connectionBadgeClass() {
        return this.isSubscribed ? 'slds-theme_success' : 'slds-theme_error';
    }
}
```

**realTimeDashboard.css**
```css
.event-new {
    animation: highlight 2s ease-out;
}

@keyframes highlight {
    0% {
        background-color: #fffbcc;
        transform: translateX(-5px);
    }
    100% {
        background-color: transparent;
        transform: translateX(0);
    }
}

.feed-container {
    border: 1px solid #d8dde6;
    border-radius: 4px;
}
```

</details>

### 📖 Explanation

The `lightning/empApi` module provides a CometD-based streaming client that connects to Salesforce's event bus. The `subscribe` function takes a channel name (`/event/EventName__e`), a replay ID (`-1` for new events only, `-2` for all retained events), and a callback. The `-1` replay ID is typical for dashboards because you only want live events, not historical ones. The `onError` handler is critical for production — it catches network disconnections and streaming failures. The auto-reconnection logic (`setTimeout` with retry) handles transient network issues. The animation pattern (add class, then remove after timeout) provides visual feedback for new events arriving. Metrics are computed dynamically from event types, making the dashboard automatically adapt to any event type without hardcoding.

### 🔄 Follow-up Questions

1. **"What's the difference between Platform Events and Change Data Capture events?"** — Platform Events are custom-defined with arbitrary payloads; CDC events are automatically generated for record CUD operations on standard/custom objects. CDC uses channel `/data/AccountChangeEvent`.
2. **"How do you handle the streaming API's daily limit?"** — The limit is based on concurrent subscribers and daily delivered event notifications. Use High-Volume Platform Events (which have much higher limits) and consider consolidating subscribers.
3. **"How would you persist dashboard state across page navigations?"** — Store the event feed and metrics in `localStorage` or use a Lightning Web State store. On reconnection, use replay ID to fetch missed events.
4. **"What happens if the user's session times out while subscribed?"** — The CometD connection drops. The `onError` handler detects this. In production, implement a heartbeat check and prompt the user to re-authenticate.

---

## 📊 Challenge Difficulty Summary

| # | Challenge | Difficulty | Key Skills Tested |
|---|-----------|-----------|-------------------|
| 1 | Searchable Contact List | 🟢 Easy | Wire service, getters, filtering |
| 2 | Reusable Modal Component | 🟢 Easy | Slots, @api methods, events, a11y |
| 3 | Paginated Data Table | 🟡 Medium | Imperative Apex, state management |
| 4 | Shopping Cart | 🟡 Medium | Parent-child events, immutability |
| 5 | Custom Lookup | 🟡 Medium | Debouncing, dropdown UX |
| 6 | Multi-Step Wizard | 🟡 Medium | Validation, form state, conditional rendering |
| 7 | Sortable & Filterable List | 🟡 Medium | Array manipulation, computed properties |
| 8 | Toast Notification System | 🟢 Easy | Service modules, error handling |
| 9 | Infinite Scroll | 🔴 Hard | IntersectionObserver, performance |
| 10 | Real-Time Dashboard | 🔴 Hard | Platform Events, Emp API, streaming |

> [!TIP]
> **Interview Strategy**: Start with the Easy challenges to build confidence, then tackle Medium ones. Save Hard challenges for when you're comfortable with the fundamentals. In a live interview, always **talk through your thought process** — interviewers value your reasoning as much as the code.

> [!IMPORTANT]
> **Before Your Interview Checklist:**
> - ✅ Set up a Salesforce scratch org with sample data
> - ✅ Practice each challenge at least twice
> - ✅ Time yourself — aim for completion within the stated time limits
> - ✅ Be ready to explain **why** you made specific design choices
> - ✅ Know the follow-up question answers — they reveal deeper understanding
