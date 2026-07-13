# ⚡ LWC Practice Questions & Solutions

> **18 beginner-friendly questions** with complete code solutions.
> Organized by topic — work through them in order for the best learning path.

---

## Table of Contents

- [Section 1: Data Binding & Rendering](#section-1-data-binding--rendering)
  - [Q1 — Hello World](#q1--hello-world)
  - [Q2 — Counter App](#q2--counter-app)
  - [Q3 — Toggle Visibility](#q3--toggle-visibility)
  - [Q4 — Display a List](#q4--display-a-list)
  - [Q5 — Greeting Card](#q5--greeting-card)
- [Section 2: Events & Communication](#section-2-events--communication)
  - [Q6 — Input Mirror](#q6--input-mirror)
  - [Q7 — Child-to-Parent Event](#q7--child-to-parent-event)
  - [Q8 — Parent-to-Child](#q8--parent-to-child)
  - [Q9 — Simple Calculator](#q9--simple-calculator)
  - [Q10 — Star Rating](#q10--star-rating)
- [Section 3: Lifecycle Hooks](#section-3-lifecycle-hooks)
  - [Q11 — Console Logger](#q11--console-logger)
  - [Q12 — Auto-Focus Input](#q12--auto-focus-input)
- [Section 4: Wire Service & Apex](#section-4-wire-service--apex)
  - [Q13 — Display Current User](#q13--display-current-user)
  - [Q14 — Account List](#q14--account-list)
  - [Q15 — Search Contacts](#q15--search-contacts)
- [Section 5: Navigation & UI](#section-5-navigation--ui)
  - [Q16 — Navigate to Record](#q16--navigate-to-record)
  - [Q17 — Toast Notification](#q17--toast-notification)
  - [Q18 — Modal / Popup](#q18--modal--popup)

---

# Section 1: Data Binding & Rendering

---

## Q1 — Hello World

> **Task:** Create a component that displays `"Hello, [Name]!"` where the name comes from a public property.

**Key Concepts:** `@api`, template expressions `{name}`

### Solution

#### `helloWorld.js`
```js
import { LightningElement, api } from 'lwc';

export default class HelloWorld extends LightningElement {
    @api name = 'World';
}
```

#### `helloWorld.html`
```html
<template>
    <lightning-card title="Hello World">
        <p class="slds-p-horizontal_small">
            Hello, {name}!
        </p>
    </lightning-card>
</template>
```

#### `helloWorld.js-meta.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__RecordPage</target>
        <target>lightning__HomePage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__AppPage,lightning__HomePage">
            <property name="name" type="String" default="World" label="Name" />
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

> [!TIP]
> `@api` makes a property **public** — a parent component or App Builder can set its value.
> Properties without `@api` are private and can only be set inside the component.

---

## Q2 — Counter App

> **Task:** Build a component with **+** and **−** buttons and a displayed count.

**Key Concepts:** Reactive state, event handlers

### Solution

#### `counterApp.js`
```js
import { LightningElement } from 'lwc';

export default class CounterApp extends LightningElement {
    count = 0;

    increment() {
        this.count++;
    }

    decrement() {
        this.count--;
    }

    reset() {
        this.count = 0;
    }
}
```

#### `counterApp.html`
```html
<template>
    <lightning-card title="Counter App" icon-name="utility:number_input">
        <div class="slds-p-around_medium slds-text-align_center">
            <p class="slds-text-heading_large slds-m-bottom_medium">{count}</p>

            <lightning-button-group>
                <lightning-button label="−" onclick={decrement}></lightning-button>
                <lightning-button label="Reset" onclick={reset}></lightning-button>
                <lightning-button label="+" onclick={increment} variant="brand"></lightning-button>
            </lightning-button-group>
        </div>
    </lightning-card>
</template>
```

#### `counterApp.js-meta.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__HomePage</target>
    </targets>
</LightningComponentBundle>
```

> [!NOTE]
> In modern LWC, you don't need `@track` for primitive values (strings, numbers, booleans).
> The framework automatically tracks top-level property changes and re-renders.
> `@track` is only needed for **deep changes** inside objects or arrays.

---

## Q3 — Toggle Visibility

> **Task:** Show/hide a paragraph on button click.

**Key Concepts:** Conditional rendering (`lwc:if`)

### Solution

#### `toggleVisibility.js`
```js
import { LightningElement } from 'lwc';

export default class ToggleVisibility extends LightningElement {
    isVisible = false;

    get buttonLabel() {
        return this.isVisible ? 'Hide Message' : 'Show Message';
    }

    toggleMessage() {
        this.isVisible = !this.isVisible;
    }
}
```

#### `toggleVisibility.html`
```html
<template>
    <lightning-card title="Toggle Visibility" icon-name="utility:preview">
        <div class="slds-p-around_medium">
            <lightning-button
                label={buttonLabel}
                onclick={toggleMessage}
                variant="brand"
            ></lightning-button>

            <template lwc:if={isVisible}>
                <div class="slds-box slds-m-top_medium slds-theme_success">
                    <p>🎉 This message is now visible! Click the button to hide it.</p>
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

> [!IMPORTANT]
> **`lwc:if` vs `if:true`** — Use `lwc:if` (introduced in API v55+). The older `if:true` / `if:false` directives are deprecated.
> Also available: `lwc:elseif={expression}` and `lwc:else`.

---

## Q4 — Display a List

> **Task:** Render a list of fruits from a JavaScript array.

**Key Concepts:** `for:each`, `key`

### Solution

#### `fruitList.js`
```js
import { LightningElement } from 'lwc';

export default class FruitList extends LightningElement {
    fruits = [
        { id: 1, name: 'Apple',      emoji: '🍎', color: '#ff6b6b' },
        { id: 2, name: 'Banana',     emoji: '🍌', color: '#ffd93d' },
        { id: 3, name: 'Cherry',     emoji: '🍒', color: '#c0392b' },
        { id: 4, name: 'Grapes',     emoji: '🍇', color: '#8e44ad' },
        { id: 5, name: 'Mango',      emoji: '🥭', color: '#f39c12' },
        { id: 6, name: 'Watermelon', emoji: '🍉', color: '#27ae60' }
    ];
}
```

#### `fruitList.html`
```html
<template>
    <lightning-card title="Fruit List" icon-name="utility:food_and_drink">
        <ul class="slds-p-around_medium">
            <template for:each={fruits} for:item="fruit">
                <li key={fruit.id} class="slds-p-vertical_x-small">
                    {fruit.emoji} {fruit.name}
                </li>
            </template>
        </ul>
    </lightning-card>
</template>
```

> [!IMPORTANT]
> The `key` attribute is **required** on the first element inside `for:each`.
> It must be a unique value (like an `Id`). It helps LWC efficiently re-render only changed items.

---

## Q5 — Greeting Card

> **Task:** Accept `firstName` and `lastName` as public properties and display a formatted greeting using a getter.

**Key Concepts:** `@api` properties, getters

### Solution

#### `greetingCard.js`
```js
import { LightningElement, api } from 'lwc';

export default class GreetingCard extends LightningElement {
    @api firstName = '';
    @api lastName = '';

    get fullName() {
        return `${this.firstName} ${this.lastName}`.trim() || 'Stranger';
    }

    get greeting() {
        const hour = new Date().getHours();
        if (hour < 12) return 'Good Morning';
        if (hour < 17) return 'Good Afternoon';
        return 'Good Evening';
    }
}
```

#### `greetingCard.html`
```html
<template>
    <lightning-card title="Greeting Card" icon-name="utility:user">
        <div class="slds-p-around_medium">
            <h2 class="slds-text-heading_medium">
                {greeting}, {fullName}! 👋
            </h2>
            <p class="slds-m-top_small slds-text-color_weak">
                Welcome to the Lightning Web Components world.
            </p>
        </div>
    </lightning-card>
</template>
```

#### Usage in a Parent Component
```html
<!-- Inside another component's template -->
<c-greeting-card first-name="Abhi" last-name="Sharma"></c-greeting-card>
```

> [!TIP]
> **Getters** (`get fullName()`) are like computed properties. They re-evaluate automatically
> whenever the reactive properties they depend on change. Use them instead of storing derived values.
> Note: In HTML, camelCase properties become **kebab-case** attributes (`firstName` → `first-name`).

---

# Section 2: Events & Communication

---

## Q6 — Input Mirror

> **Task:** Type in an input field and mirror the text below in real time.

**Key Concepts:** `onchange`, `event.target.value`

### Solution

#### `inputMirror.js`
```js
import { LightningElement } from 'lwc';

export default class InputMirror extends LightningElement {
    typedText = '';

    handleInputChange(event) {
        this.typedText = event.target.value;
    }

    get hasText() {
        return this.typedText.length > 0;
    }

    get charCount() {
        return this.typedText.length;
    }
}
```

#### `inputMirror.html`
```html
<template>
    <lightning-card title="Input Mirror" icon-name="utility:text">
        <div class="slds-p-around_medium">
            <lightning-input
                type="text"
                label="Type something..."
                onchange={handleInputChange}
            ></lightning-input>

            <template lwc:if={hasText}>
                <div class="slds-box slds-m-top_medium slds-theme_shade">
                    <p class="slds-text-heading_small">You typed:</p>
                    <p class="slds-text-heading_medium slds-text-color_success">
                        {typedText}
                    </p>
                    <p class="slds-text-body_small slds-text-color_weak">
                        Character count: {charCount}
                    </p>
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

> [!NOTE]
> `lightning-input` fires `change` events. For native `<input>`, you'd use `oninput` for real-time
> updates and `onchange` for blur-based updates. With `lightning-input`, `onchange` fires on every keystroke.

---

## Q7 — Child-to-Parent Event

> **Task:** Create a child component with a button. On click, fire a custom event that the parent catches and displays.

**Key Concepts:** `CustomEvent`, `dispatchEvent`, event handler on parent

### Solution

#### Child: `notificationSender.js`
```js
import { LightningElement } from 'lwc';

export default class NotificationSender extends LightningElement {
    sendNotification() {
        const message = 'Hello from the Child! 🚀';

        // Create and dispatch a custom event
        const notifyEvent = new CustomEvent('notify', {
            detail: { message }
        });
        this.dispatchEvent(notifyEvent);
    }
}
```

#### Child: `notificationSender.html`
```html
<template>
    <lightning-button
        label="Send Notification to Parent"
        onclick={sendNotification}
        variant="brand"
    ></lightning-button>
</template>
```

#### Parent: `notificationReceiver.js`
```js
import { LightningElement } from 'lwc';

export default class NotificationReceiver extends LightningElement {
    receivedMessage = '';

    handleNotify(event) {
        this.receivedMessage = event.detail.message;
    }

    get hasMessage() {
        return this.receivedMessage.length > 0;
    }
}
```

#### Parent: `notificationReceiver.html`
```html
<template>
    <lightning-card title="Parent-Child Events" icon-name="utility:connected_apps">
        <div class="slds-p-around_medium">
            <!-- Child component — notice the onnotify handler -->
            <c-notification-sender onnotify={handleNotify}></c-notification-sender>

            <template lwc:if={hasMessage}>
                <div class="slds-box slds-m-top_medium slds-theme_info">
                    <p><strong>Received from child:</strong> {receivedMessage}</p>
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

> [!IMPORTANT]
> **Custom Event naming rules:**
> - Event name must be **lowercase**, no underscores, no hyphens (e.g., `notify`, `valuechange`).
> - In the parent HTML, prefix with `on` → `onnotify`, `onvaluechange`.
> - Data goes in `event.detail`.

---

## Q8 — Parent-to-Child

> **Task:** Parent passes a color value to a child. The child renders a div with that background color.

**Key Concepts:** `@api` property, dynamic styling

### Solution

#### Child: `colorBox.js`
```js
import { LightningElement, api } from 'lwc';

export default class ColorBox extends LightningElement {
    @api color = '#3498db';

    get boxStyle() {
        return `
            background-color: ${this.color};
            width: 200px;
            height: 200px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-weight: bold;
            font-size: 1.2rem;
            text-shadow: 0 1px 3px rgba(0,0,0,0.4);
        `;
    }
}
```

#### Child: `colorBox.html`
```html
<template>
    <div style={boxStyle}>
        {color}
    </div>
</template>
```

#### Parent: `colorPicker.js`
```js
import { LightningElement } from 'lwc';

export default class ColorPicker extends LightningElement {
    selectedColor = '#3498db';

    colors = [
        { label: 'Blue',   value: '#3498db' },
        { label: 'Red',    value: '#e74c3c' },
        { label: 'Green',  value: '#2ecc71' },
        { label: 'Purple', value: '#9b59b6' },
        { label: 'Orange', value: '#f39c12' }
    ];

    handleColorChange(event) {
        this.selectedColor = event.detail.value;
    }
}
```

#### Parent: `colorPicker.html`
```html
<template>
    <lightning-card title="Color Picker" icon-name="utility:color_swatch">
        <div class="slds-p-around_medium">
            <lightning-combobox
                label="Choose a Color"
                value={selectedColor}
                options={colors}
                onchange={handleColorChange}
            ></lightning-combobox>

            <div class="slds-m-top_medium">
                <c-color-box color={selectedColor}></c-color-box>
            </div>
        </div>
    </lightning-card>
</template>
```

> [!TIP]
> Data flows **down** via `@api` properties and **up** via custom events.
> This is the fundamental LWC communication pattern. Master it!

---

## Q9 — Simple Calculator

> **Task:** Two input fields + four operation buttons (+, −, ×, ÷). Show the result.

**Key Concepts:** Event handling, basic logic

### Solution

#### `simpleCalculator.js`
```js
import { LightningElement } from 'lwc';

export default class SimpleCalculator extends LightningElement {
    num1 = 0;
    num2 = 0;
    result = null;
    operation = '';

    handleNum1Change(event) {
        this.num1 = parseFloat(event.target.value) || 0;
    }

    handleNum2Change(event) {
        this.num2 = parseFloat(event.target.value) || 0;
    }

    add() {
        this.operation = '+';
        this.result = this.num1 + this.num2;
    }

    subtract() {
        this.operation = '−';
        this.result = this.num1 - this.num2;
    }

    multiply() {
        this.operation = '×';
        this.result = this.num1 * this.num2;
    }

    divide() {
        this.operation = '÷';
        if (this.num2 === 0) {
            this.result = 'Error: Division by zero!';
        } else {
            this.result = this.num1 / this.num2;
        }
    }

    get hasResult() {
        return this.result !== null;
    }

    get resultDisplay() {
        return `${this.num1} ${this.operation} ${this.num2} = ${this.result}`;
    }
}
```

#### `simpleCalculator.html`
```html
<template>
    <lightning-card title="Simple Calculator" icon-name="utility:advanced_function">
        <div class="slds-p-around_medium">
            <lightning-layout multiple-rows>
                <lightning-layout-item size="6" padding="around-small">
                    <lightning-input
                        type="number"
                        label="Number 1"
                        value={num1}
                        onchange={handleNum1Change}
                    ></lightning-input>
                </lightning-layout-item>
                <lightning-layout-item size="6" padding="around-small">
                    <lightning-input
                        type="number"
                        label="Number 2"
                        value={num2}
                        onchange={handleNum2Change}
                    ></lightning-input>
                </lightning-layout-item>
            </lightning-layout>

            <div class="slds-m-top_medium slds-text-align_center">
                <lightning-button-group>
                    <lightning-button label="+" onclick={add}></lightning-button>
                    <lightning-button label="−" onclick={subtract}></lightning-button>
                    <lightning-button label="×" onclick={multiply}></lightning-button>
                    <lightning-button label="÷" onclick={divide}></lightning-button>
                </lightning-button-group>
            </div>

            <template lwc:if={hasResult}>
                <div class="slds-box slds-m-top_medium slds-text-align_center slds-theme_shade">
                    <p class="slds-text-heading_medium">{resultDisplay}</p>
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

---

## Q10 — Star Rating

> **Task:** Click on stars (1–5) to set a rating. Fire a custom event with the selected value.

**Key Concepts:** Iteration, dynamic CSS, custom events

### Solution

#### `starRating.js`
```js
import { LightningElement, api } from 'lwc';

export default class StarRating extends LightningElement {
    @api maxStars = 5;
    _rating = 0;

    @api
    get rating() {
        return this._rating;
    }
    set rating(value) {
        this._rating = parseInt(value, 10) || 0;
    }

    get stars() {
        const starList = [];
        for (let i = 1; i <= this.maxStars; i++) {
            starList.push({
                index: i,
                filled: i <= this._rating,
                cssClass: i <= this._rating ? 'star filled' : 'star empty'
            });
        }
        return starList;
    }

    handleClick(event) {
        const selectedRating = parseInt(event.target.dataset.index, 10);
        this._rating = selectedRating;

        // Fire custom event to parent
        this.dispatchEvent(new CustomEvent('ratingchange', {
            detail: { rating: this._rating }
        }));
    }
}
```

#### `starRating.html`
```html
<template>
    <div class="star-container">
        <template for:each={stars} for:item="star">
            <span
                key={star.index}
                class={star.cssClass}
                data-index={star.index}
                onclick={handleClick}
            >★</span>
        </template>
    </div>
    <p class="rating-text">{_rating} / {maxStars}</p>
</template>
```

#### `starRating.css`
```css
.star-container {
    display: flex;
    gap: 4px;
}

.star {
    font-size: 2rem;
    cursor: pointer;
    transition: transform 0.2s, color 0.2s;
}

.star:hover {
    transform: scale(1.3);
}

.star.filled {
    color: #f1c40f;
}

.star.empty {
    color: #bdc3c7;
}

.rating-text {
    margin-top: 8px;
    font-size: 0.9rem;
    color: #7f8c8d;
}
```

#### Usage in Parent
```html
<c-star-rating rating="3" onratingchange={handleRatingChange}></c-star-rating>
```

```js
handleRatingChange(event) {
    console.log('Selected rating:', event.detail.rating);
}
```

---

# Section 3: Lifecycle Hooks

---

## Q11 — Console Logger

> **Task:** Log messages in each lifecycle hook to understand the execution order.

**Key Concepts:** `constructor`, `connectedCallback`, `renderedCallback`, `disconnectedCallback`

### Solution

#### `lifecycleLogger.js`
```js
import { LightningElement } from 'lwc';

export default class LifecycleLogger extends LightningElement {
    message = 'Observing lifecycle hooks...';

    constructor() {
        super(); // MUST call super() first!
        console.log('1️⃣ CONSTRUCTOR — Component instance created');
        console.log('   → this.template is NOT available yet');
        console.log('   → Cannot access child elements');
    }

    connectedCallback() {
        console.log('2️⃣ CONNECTED CALLBACK — Component inserted into DOM');
        console.log('   → Good place to fetch data, set up subscriptions');
        console.log('   → this.template is available but not yet rendered');
        this.message = 'Component is connected to the DOM! ✅';
    }

    renderedCallback() {
        console.log('3️⃣ RENDERED CALLBACK — Component has finished rendering');
        console.log('   → DOM elements are available via this.template.querySelector');
        console.log('   → ⚠️ Be careful: this fires on EVERY re-render!');
    }

    disconnectedCallback() {
        console.log('4️⃣ DISCONNECTED CALLBACK — Component removed from DOM');
        console.log('   → Clean up: remove event listeners, clear timers');
    }
}
```

#### `lifecycleLogger.html`
```html
<template>
    <lightning-card title="Lifecycle Hooks Logger" icon-name="utility:rotate">
        <div class="slds-p-around_medium">
            <p>{message}</p>
            <p class="slds-text-color_weak slds-m-top_small">
                Open the browser console (F12) to see lifecycle logs.
            </p>
        </div>
    </lightning-card>
</template>
```

> [!IMPORTANT]
> **Lifecycle Execution Order:**
>
> | Order | Hook | When |
> |-------|------|------|
> | 1 | `constructor()` | Instance created. Call `super()` first. No DOM access. |
> | 2 | `connectedCallback()` | Added to DOM. Good for data fetching. |
> | 3 | `renderedCallback()` | After every render. Use carefully — can loop! |
> | 4 | `disconnectedCallback()` | Removed from DOM. Clean up here. |

---

## Q12 — Auto-Focus Input

> **Task:** Automatically focus an input field when the component loads.

**Key Concepts:** `renderedCallback`, `this.template.querySelector`

### Solution

#### `autoFocusInput.js`
```js
import { LightningElement } from 'lwc';

export default class AutoFocusInput extends LightningElement {
    hasRendered = false;

    renderedCallback() {
        // Only auto-focus on the FIRST render
        if (!this.hasRendered) {
            this.hasRendered = true;

            const inputEl = this.template.querySelector('lightning-input');
            if (inputEl) {
                inputEl.focus();
            }
        }
    }
}
```

#### `autoFocusInput.html`
```html
<template>
    <lightning-card title="Auto Focus Input" icon-name="utility:cursor">
        <div class="slds-p-around_medium">
            <lightning-input
                type="text"
                label="This field auto-focuses on load"
                placeholder="Start typing..."
            ></lightning-input>
        </div>
    </lightning-card>
</template>
```

> [!WARNING]
> **Always guard `renderedCallback`** with a flag (`hasRendered`).
> Without it, your code runs on **every re-render**, causing infinite loops or performance issues.

---

# Section 4: Wire Service & Apex

> [!NOTE]
> Questions 13–15 require a **Salesforce org** (scratch org or developer org).
> You'll need to deploy the Apex classes and LWC components to test them.

---

## Q13 — Display Current User

> **Task:** Show the logged-in user's name using `@wire` and the UI API.

**Key Concepts:** `@wire`, `getRecord`, `lightning/uiRecordApi`

### Solution

#### `currentUser.js`
```js
import { LightningElement, wire } from 'lwc';
import Id from '@salesforce/user/Id';
import { getRecord } from 'lightning/uiRecordApi';

import NAME_FIELD from '@salesforce/schema/User.Name';
import EMAIL_FIELD from '@salesforce/schema/User.Email';
import PROFILE_FIELD from '@salesforce/schema/User.Profile.Name';

const USER_FIELDS = [NAME_FIELD, EMAIL_FIELD, PROFILE_FIELD];

export default class CurrentUser extends LightningElement {
    userId = Id;

    @wire(getRecord, { recordId: '$userId', fields: USER_FIELDS })
    user;

    get userName() {
        return this.user?.data?.fields?.Name?.value || 'Loading...';
    }

    get userEmail() {
        return this.user?.data?.fields?.Email?.value || '';
    }

    get userProfile() {
        return this.user?.data?.fields?.Profile?.displayValue || '';
    }

    get isLoading() {
        return !this.user?.data && !this.user?.error;
    }

    get hasError() {
        return !!this.user?.error;
    }

    get errorMessage() {
        return this.user?.error?.body?.message || 'Unknown error';
    }
}
```

#### `currentUser.html`
```html
<template>
    <lightning-card title="Current User Info" icon-name="utility:user">
        <div class="slds-p-around_medium">

            <template lwc:if={isLoading}>
                <lightning-spinner alternative-text="Loading" size="small"></lightning-spinner>
            </template>

            <template lwc:if={hasError}>
                <p class="slds-text-color_error">Error: {errorMessage}</p>
            </template>

            <template lwc:if={user.data}>
                <dl class="slds-list_horizontal slds-wrap">
                    <dt class="slds-item_label slds-text-color_weak">Name:</dt>
                    <dd class="slds-item_detail">{userName}</dd>

                    <dt class="slds-item_label slds-text-color_weak">Email:</dt>
                    <dd class="slds-item_detail">{userEmail}</dd>

                    <dt class="slds-item_label slds-text-color_weak">Profile:</dt>
                    <dd class="slds-item_detail">{userProfile}</dd>
                </dl>
            </template>
        </div>
    </lightning-card>
</template>
```

> [!TIP]
> **`@wire` is reactive.** When you use `$propertyName` in the config (e.g., `'$userId'`),
> the wire re-fires automatically whenever that property changes.

---

## Q14 — Account List

> **Task:** Fetch and display a list of Accounts using a wired Apex method.

**Key Concepts:** `@wire` with Apex, `@AuraEnabled(cacheable=true)`

### Solution

#### Apex: `AccountController.cls`
```java
public with sharing class AccountController {

    @AuraEnabled(cacheable=true)
    public static List<Account> getAccounts() {
        return [
            SELECT Id, Name, Industry, Phone, Rating
            FROM Account
            ORDER BY Name
            LIMIT 10
        ];
    }
}
```

#### `accountList.js`
```js
import { LightningElement, wire } from 'lwc';
import getAccounts from '@salesforce/apex/AccountController.getAccounts';

const COLUMNS = [
    { label: 'Account Name', fieldName: 'Name', type: 'text' },
    { label: 'Industry',     fieldName: 'Industry', type: 'text' },
    { label: 'Phone',        fieldName: 'Phone', type: 'phone' },
    { label: 'Rating',       fieldName: 'Rating', type: 'text' }
];

export default class AccountList extends LightningElement {
    columns = COLUMNS;

    @wire(getAccounts)
    accounts;

    get hasAccounts() {
        return this.accounts?.data?.length > 0;
    }

    get errorMessage() {
        return this.accounts?.error?.body?.message || 'Failed to load accounts';
    }
}
```

#### `accountList.html`
```html
<template>
    <lightning-card title="Account List" icon-name="standard:account">

        <template lwc:if={accounts.error}>
            <div class="slds-p-around_medium">
                <p class="slds-text-color_error">{errorMessage}</p>
            </div>
        </template>

        <template lwc:if={hasAccounts}>
            <lightning-datatable
                key-field="Id"
                data={accounts.data}
                columns={columns}
                hide-checkbox-column
            ></lightning-datatable>
        </template>

        <template lwc:if={accounts.data}>
            <template lwc:if={!hasAccounts}>
                <div class="slds-p-around_medium slds-text-align_center">
                    <p class="slds-text-color_weak">No accounts found.</p>
                </div>
            </template>
        </template>

    </lightning-card>
</template>
```

> [!IMPORTANT]
> **`cacheable=true`** is required for `@wire` Apex methods. It caches the result on the client side.
> If your method performs DML (insert/update/delete), you **cannot** use `@wire` — use imperative Apex instead.

---

## Q15 — Search Contacts

> **Task:** Input a name and search for matching Contacts via imperative Apex.

**Key Concepts:** Imperative Apex call, error handling, debouncing

### Solution

#### Apex: `ContactController.cls`
```java
public with sharing class ContactController {

    @AuraEnabled(cacheable=true)
    public static List<Contact> searchContacts(String searchKey) {
        if (String.isBlank(searchKey)) {
            return new List<Contact>();
        }
        String key = '%' + String.escapeSingleQuotes(searchKey) + '%';
        return [
            SELECT Id, Name, Email, Phone, Account.Name
            FROM Contact
            WHERE Name LIKE :key
            ORDER BY Name
            LIMIT 20
        ];
    }
}
```

#### `contactSearch.js`
```js
import { LightningElement } from 'lwc';
import searchContacts from '@salesforce/apex/ContactController.searchContacts';

const COLUMNS = [
    { label: 'Name',    fieldName: 'Name',  type: 'text' },
    { label: 'Email',   fieldName: 'Email', type: 'email' },
    { label: 'Phone',   fieldName: 'Phone', type: 'phone' }
];

export default class ContactSearch extends LightningElement {
    searchKey = '';
    contacts = [];
    columns = COLUMNS;
    error;
    isLoading = false;

    // Debounce timer
    _delayTimeout;

    handleSearchChange(event) {
        clearTimeout(this._delayTimeout);
        const value = event.target.value;

        // Debounce: wait 300ms after user stops typing
        // eslint-disable-next-line @lwc/lwc/no-async-operation
        this._delayTimeout = setTimeout(() => {
            this.searchKey = value;
            this.performSearch();
        }, 300);
    }

    async performSearch() {
        if (this.searchKey.length < 2) {
            this.contacts = [];
            return;
        }

        this.isLoading = true;
        this.error = undefined;

        try {
            const result = await searchContacts({ searchKey: this.searchKey });
            this.contacts = result;
        } catch (error) {
            this.error = error.body?.message || 'Search failed';
            this.contacts = [];
        } finally {
            this.isLoading = false;
        }
    }

    get hasContacts() {
        return this.contacts.length > 0;
    }

    get showNoResults() {
        return !this.isLoading && this.searchKey.length >= 2 && this.contacts.length === 0 && !this.error;
    }
}
```

#### `contactSearch.html`
```html
<template>
    <lightning-card title="Contact Search" icon-name="standard:contact">
        <div class="slds-p-around_medium">

            <lightning-input
                type="search"
                label="Search Contacts"
                placeholder="Type a name (min 2 characters)..."
                onchange={handleSearchChange}
            ></lightning-input>

            <template lwc:if={isLoading}>
                <div class="slds-m-top_medium slds-text-align_center">
                    <lightning-spinner alternative-text="Searching..." size="small"></lightning-spinner>
                </div>
            </template>

            <template lwc:if={error}>
                <div class="slds-m-top_medium">
                    <p class="slds-text-color_error">{error}</p>
                </div>
            </template>

            <template lwc:if={hasContacts}>
                <lightning-datatable
                    class="slds-m-top_medium"
                    key-field="Id"
                    data={contacts}
                    columns={columns}
                    hide-checkbox-column
                ></lightning-datatable>
            </template>

            <template lwc:if={showNoResults}>
                <div class="slds-m-top_medium slds-text-align_center">
                    <p class="slds-text-color_weak">No contacts found for "{searchKey}"</p>
                </div>
            </template>

        </div>
    </lightning-card>
</template>
```

> [!TIP]
> **Imperative Apex vs Wire:**
>
> | Feature | `@wire` | Imperative |
> |---------|---------|-----------|
> | When to use | Read-only data on load | User-triggered actions, DML |
> | Caching | Automatic (LDS) | Manual |
> | Syntax | Declarative | `async/await` or `.then()` |
> | `cacheable` | Required (`true`) | Optional |
> | Reactivity | Re-fires on param change | Call manually |

---

# Section 5: Navigation & UI

---

## Q16 — Navigate to Record

> **Task:** Create a button that navigates to a specific record page.

**Key Concepts:** `NavigationMixin`

### Solution

#### `recordNavigator.js`
```js
import { LightningElement, api } from 'lwc';
import { NavigationMixin } from 'lightning/navigation';

export default class RecordNavigator extends NavigationMixin(LightningElement) {
    @api recordId;

    navigateToRecord() {
        this[NavigationMixin.Navigate]({
            type: 'standard__recordPage',
            attributes: {
                recordId: this.recordId,
                objectApiName: 'Account',
                actionName: 'view'
            }
        });
    }

    navigateToNewAccount() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Account',
                actionName: 'new'
            }
        });
    }

    navigateToList() {
        this[NavigationMixin.Navigate]({
            type: 'standard__objectPage',
            attributes: {
                objectApiName: 'Account',
                actionName: 'list'
            },
            state: {
                filterName: 'Recent'
            }
        });
    }
}
```

#### `recordNavigator.html`
```html
<template>
    <lightning-card title="Navigation Demo" icon-name="utility:routing_offline">
        <div class="slds-p-around_medium">
            <lightning-button-group>
                <lightning-button
                    label="View This Record"
                    onclick={navigateToRecord}
                    variant="brand"
                    disabled={!recordId}
                ></lightning-button>
                <lightning-button
                    label="New Account"
                    onclick={navigateToNewAccount}
                    variant="neutral"
                ></lightning-button>
                <lightning-button
                    label="Account List"
                    onclick={navigateToList}
                    variant="neutral"
                ></lightning-button>
            </lightning-button-group>
        </div>
    </lightning-card>
</template>
```

> [!NOTE]
> **NavigationMixin** is a mixin — you extend your class with `NavigationMixin(LightningElement)`.
> Common page types: `standard__recordPage`, `standard__objectPage`, `standard__navItemPage`, `standard__webPage`.

---

## Q17 — Toast Notification

> **Task:** Show a success/error toast message on button click.

**Key Concepts:** `ShowToastEvent`, `platformShowToastEvent`

### Solution

#### `toastDemo.js`
```js
import { LightningElement } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

export default class ToastDemo extends LightningElement {

    showSuccess() {
        this.dispatchEvent(new ShowToastEvent({
            title: 'Success!',
            message: 'The record was saved successfully.',
            variant: 'success'
        }));
    }

    showError() {
        this.dispatchEvent(new ShowToastEvent({
            title: 'Error',
            message: 'Something went wrong. Please try again.',
            variant: 'error',
            mode: 'sticky' // Won't auto-dismiss
        }));
    }

    showWarning() {
        this.dispatchEvent(new ShowToastEvent({
            title: 'Warning',
            message: 'You have unsaved changes.',
            variant: 'warning'
        }));
    }

    showInfo() {
        this.dispatchEvent(new ShowToastEvent({
            title: 'Info',
            message: 'This is an informational message.',
            variant: 'info'
        }));
    }
}
```

#### `toastDemo.html`
```html
<template>
    <lightning-card title="Toast Notifications" icon-name="utility:notification">
        <div class="slds-p-around_medium">
            <lightning-button-group>
                <lightning-button
                    label="Success"
                    onclick={showSuccess}
                    variant="success"
                ></lightning-button>
                <lightning-button
                    label="Error"
                    onclick={showError}
                    variant="destructive"
                ></lightning-button>
                <lightning-button
                    label="Warning"
                    onclick={showWarning}
                ></lightning-button>
                <lightning-button
                    label="Info"
                    onclick={showInfo}
                    variant="brand"
                ></lightning-button>
            </lightning-button-group>
        </div>
    </lightning-card>
</template>
```

> [!TIP]
> **Toast `mode` options:**
> - `dismissible` (default) — Auto-dismiss after ~3 seconds, user can close.
> - `pester` — Same as dismissible but stays a bit longer.
> - `sticky` — Stays until user clicks close. Use for errors.

---

## Q18 — Modal / Popup

> **Task:** Open and close a modal dialog with a message.

**Key Concepts:** Conditional rendering, SLDS modal markup, `LightningModal`

### Solution (Using `LightningModal` — Recommended)

#### `myModal.js`
```js
import { api } from 'lwc';
import LightningModal from 'lightning/modal';

export default class MyModal extends LightningModal {
    @api heading;
    @api content;

    handleConfirm() {
        this.close('confirmed');
    }

    handleCancel() {
        this.close('cancelled');
    }
}
```

#### `myModal.html`
```html
<template>
    <lightning-modal-header label={heading}></lightning-modal-header>

    <lightning-modal-body>
        <div class="slds-p-around_medium">
            <p>{content}</p>
        </div>
    </lightning-modal-body>

    <lightning-modal-footer>
        <lightning-button
            label="Cancel"
            onclick={handleCancel}
        ></lightning-button>
        <lightning-button
            label="Confirm"
            variant="brand"
            onclick={handleConfirm}
            class="slds-m-left_x-small"
        ></lightning-button>
    </lightning-modal-footer>
</template>
```

#### Parent: `modalOpener.js`
```js
import { LightningElement } from 'lwc';
import MyModal from 'c/myModal';

export default class ModalOpener extends LightningElement {
    result = '';

    async openModal() {
        const result = await MyModal.open({
            size: 'small',
            heading: 'Confirm Action',
            content: 'Are you sure you want to proceed with this action?'
        });

        this.result = result || 'dismissed';
    }

    get hasResult() {
        return this.result.length > 0;
    }
}
```

#### Parent: `modalOpener.html`
```html
<template>
    <lightning-card title="Modal Demo" icon-name="utility:popup">
        <div class="slds-p-around_medium">
            <lightning-button
                label="Open Modal"
                onclick={openModal}
                variant="brand"
            ></lightning-button>

            <template lwc:if={hasResult}>
                <p class="slds-m-top_medium">
                    Modal result: <strong>{result}</strong>
                </p>
            </template>
        </div>
    </lightning-card>
</template>
```

> [!TIP]
> **`LightningModal`** (API v55+) is the modern approach for modals. Key differences from manual SLDS:
> - Extends `LightningModal` instead of `LightningElement`
> - Uses `lightning-modal-header`, `lightning-modal-body`, `lightning-modal-footer`
> - Opened via `static async open()` — returns a Promise with the close result
> - Handles accessibility, focus trap, and backdrop automatically

---

# 📋 Quick Reference Cheat Sheet

## Property Decorators

| Decorator | Purpose | Example |
|-----------|---------|---------|
| `@api` | Public property (parent can set it) | `@api recordId;` |
| `@track` | Deep-track objects/arrays (rarely needed) | `@track items = [];` |
| `@wire` | Reactive data fetching | `@wire(getRecord, {...})` |
| *(none)* | Private reactive property | `count = 0;` |

## Event Patterns

```
Parent → Child:  @api properties
Child → Parent:  CustomEvent + dispatchEvent
Unrelated:       Lightning Message Service (LMS) or pubsub
```

## Template Directives

| Directive | Usage |
|-----------|-------|
| `lwc:if={expr}` | Conditional render |
| `lwc:elseif={expr}` | Else-if branch |
| `lwc:else` | Else branch |
| `for:each={array}` | Loop with `for:item` and `key` |
| `iterator:it={array}` | Loop with `it.value`, `it.first`, `it.last` |
| `lwc:ref="name"` | DOM reference (`this.refs.name`) |

## Common Imports

```js
// User & Record
import userId from '@salesforce/user/Id';
import { getRecord, getFieldValue } from 'lightning/uiRecordApi';

// Navigation
import { NavigationMixin } from 'lightning/navigation';

// Toast
import { ShowToastEvent } from 'lightning/platformShowToastEvent';

// Modal
import LightningModal from 'lightning/modal';

// Apex
import myMethod from '@salesforce/apex/MyController.myMethod';

// Schema
import FIELD from '@salesforce/schema/Object.Field';
```

---

> **Happy Learning! ⚡**
> Start with Q1, work your way through, and you'll have a solid LWC foundation.
> Once comfortable, explore **Lightning Message Service (LMS)**, **Lightning Data Service (LDS)**, and **Dynamic Components** for intermediate/advanced topics.
