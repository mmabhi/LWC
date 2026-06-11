# LWC Interview Question: Rendering Images

## The Interview Question

**Scenario:**
You are building an LWC for a custom "Employee Profile" card. You need to display two different images on this card:

1. **The Company Logo:** This is a static image that is bundled with your Salesforce package.
2. **The Employee Profile Picture:** The URL for this image is retrieved dynamically from the `User` object via an Apex controller (e.g., `https://example.com/profiles/jdoe.png`).

**Your Tasks:**
1. How do you import and render the **Company Logo** in your component? Assume the image is named `acme_logo.png` and you want to use the best practice for storing packaged assets in Salesforce.
2. How do you render the **Employee Profile Picture** assuming the URL is stored in a JavaScript property named `profilePictureUrl`?
3. Provide the basic `HTML` and `JavaScript` code snippets to achieve this.

***

**Bonus / Follow-up Questions:**
* What if the company logo was inside a zip file called `acme_assets.zip` under a folder named `images`? How would your import change?
* How would you handle the scenario where the dynamic profile picture URL is broken or fails to load, so that you can display a default fallback image instead of a broken image icon?

---

## The Solution

The best practice for storing packaged, static assets like a company logo is to use **Salesforce Static Resources**. For dynamic URLs coming from a database, we bind a JavaScript variable to the `src` attribute.

### 1. Core Solution

**`employeeProfile.js` (JavaScript)**
```javascript
import { LightningElement, api } from 'lwc';
// 1. Import the static resource for the company logo
import COMPANY_LOGO from '@salesforce/resourceUrl/acme_logo';

export default class EmployeeProfile extends LightningElement {
    // Expose the imported static resource to the HTML template
    companyLogoUrl = COMPANY_LOGO;

    // 2. Dynamic property holding the employee picture URL (likely populated by Apex or an @api prop)
    @api profilePictureUrl = 'https://example.com/profiles/jdoe.png'; 
}
```

**`employeeProfile.html` (HTML)**
```html
<template>
    <div class="profile-card">
        <!-- 1. Rendering the Static Resource Logo -->
        <img src={companyLogoUrl} alt="Acme Company Logo" class="logo" />

        <!-- 2. Rendering the Dynamic Profile Picture -->
        <img src={profilePictureUrl} alt="Employee Profile Picture" class="profile-pic" />
    </div>
</template>
```

> [!NOTE]
> **Why this is the correct answer:**
> * Interviewers want to see that you know how to use `@salesforce/resourceUrl`. Hardcoding absolute paths like `/resource/12345/acme_logo` is an anti-pattern because it breaks if the resource is updated or moved.
> * You correctly demonstrated one-way data binding (`{property}`) to pass JavaScript variables into HTML attributes.

### 2. Bonus Solutions

#### Bonus 1: Handling ZIP files
If the static resource is an archive (e.g., `acme_assets.zip`), you import the root of the archive and append the path to the specific image in your JavaScript or HTML.

**Solution:**
```javascript
import ACME_ASSETS from '@salesforce/resourceUrl/acme_assets';

export default class EmployeeProfile extends LightningElement {
    // Append the relative path to the image inside the zip folder
    companyLogoUrl = ACME_ASSETS + '/images/acme_logo.png';
}
```

#### Bonus 2: Handling Broken Images
If `profilePictureUrl` points to a 404 error, the browser will show an ugly broken image icon. You can handle this gracefully using the standard HTML `onerror` event.

**HTML:**
```html
<img 
    src={profilePictureUrl} 
    alt="Employee Profile Picture" 
    onerror={handleImageError} 
/>
```

**JavaScript:**
```javascript
import DEFAULT_AVATAR from '@salesforce/resourceUrl/default_avatar';

export default class EmployeeProfile extends LightningElement {
    @api profilePictureUrl;
    
    // Fallback if the image fails to load
    handleImageError(event) {
        // Stop infinite loops in case the default avatar is also broken
        event.target.onerror = null; 
        // Swap the broken src with our static fallback image
        event.target.src = DEFAULT_AVATAR; 
    }
}
```

> [!TIP]
> **Key Takeaways for the Interview:**
> 1. **Static Resources are king** for static assets. Know how to import them.
> 2. **Never hardcode URLs** to Salesforce assets.
> 3. Mentioning **Accessibility (`alt` text)** on your `<img>` tags shows attention to detail.
> 4. Knowing how to handle **`onerror`** shows you've built resilient UI components in the real world.
