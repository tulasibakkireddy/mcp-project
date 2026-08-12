# Playwright Locator Strategy

## Purpose

These instructions define how Playwright locators must be created, maintained, and used throughout the automation framework.

These rules apply whenever GitHub Copilot generates, modifies, or refactors Playwright automation code, including code generated with Playwright MCP.

---

# 1. Locator Location

All locators must belong inside:

* Page Objects
* Reusable Components

Never define locators directly inside test/spec files.

### Correct

```ts
export class LoginPage {
    readonly loginButton = this.page.getByRole("button", {
        name: "Log in"
    });

    constructor(private page: Page) {}
}
```

### Incorrect

```ts
test("login", async ({ page }) => {
    await page.getByRole("button", {
        name: "Log in"
    }).click();
});
```

The locator should be defined in the Page Object and the test should use the Page Object.

---

# 2. Locator Priority

Always use the highest available locator from the following priority order.

| Priority | Locator              | Recommendation       |
| -------- | -------------------- | -------------------- |
| 1        | `getByRole()`        | ✅ Preferred          |
| 2        | `getByLabel()`       | ✅ Preferred          |
| 3        | `getByPlaceholder()` | ✅ Preferred          |
| 4        | `getByTestId()`      | ✅ Preferred          |
| 5        | `getByText()`        | ⚠ Use carefully      |
| 6        | `locator()` / CSS    | ⚠ Only when required |
| 7        | XPath                | ❌ Last option        |

The mandatory priority is:

```text
getByRole()
    ↓
getByLabel()
    ↓
getByPlaceholder()
    ↓
getByTestId()
    ↓
getByText()
    ↓
CSS locator
    ↓
XPath
```

Never choose CSS or XPath when a suitable Playwright locator is available.

---

# 3. Playwright MCP Inspection

When Playwright MCP is available, inspect the page before creating a locator.

Use the available page/accessibility information to determine:

* Role
* Accessible name
* Label
* Placeholder
* Test ID
* Visible text
* Element relationships
* Stable attributes

Do not guess a locator when the page can be inspected.

### Mandatory process

```text
Inspect page
     ↓
Identify element
     ↓
Check Playwright accessibility locator
     ↓
Check CSS if necessary
     ↓
Use XPath only as final fallback
```

---

# 4. getByRole()

`getByRole()` is the first choice for interactive elements whenever an appropriate role and accessible name are available.

### Correct

```ts
readonly loginButton = this.page.getByRole("button", {
    name: "Log in"
});
```

```ts
readonly checkoutButton = this.page.getByRole("button", {
    name: "Checkout"
});
```

```ts
readonly homeLink = this.page.getByRole("link", {
    name: "Home"
});
```

### Avoid

```ts
this.page.locator("button");
```

or

```ts
this.page.locator(".btn-primary");
```

when an accessible role-based locator is available.

---

# 5. getByLabel()

Use `getByLabel()` for form controls whenever a meaningful label is available.

### Correct

```ts
readonly emailTextbox = this.page.getByLabel("Email");
```

```ts
readonly passwordTextbox = this.page.getByLabel("Password");
```

### Avoid

```ts
this.page.locator("#Email");
```

when the field has a usable label.

---

# 6. getByPlaceholder()

Use `getByPlaceholder()` when a suitable label is unavailable and the placeholder is stable and meaningful.

### Correct

```ts
readonly searchTextbox =
    this.page.getByPlaceholder("Search store");
```

### Avoid

Using placeholder when a stable accessible label exists.

```ts
this.page.getByPlaceholder("Enter email");
```

should not be preferred over:

```ts
this.page.getByLabel("Email");
```

when the label is available.

---

# 7. getByTestId()

Use `getByTestId()` when the application exposes a stable test ID and a more appropriate accessibility locator is not available.

### Correct

```ts
readonly loginButton =
    this.page.getByTestId("login-button");
```

Do not replace it with CSS unnecessarily.

### Avoid

```ts
this.page.locator('[data-testid="login-button"]');
```

when:

```ts
this.page.getByTestId("login-button");
```

is available.

---

# 8. getByText()

Use `getByText()` carefully.

Recommended for:

* Static messages
* Headings
* Navigation text
* Static content

### Correct

```ts
readonly shoppingCart =
    this.page.getByText("Shopping cart");
```

### Avoid

Using dynamic text as a primary locator.

### Dynamic content

If text changes based on test data, create a reusable dynamic locator method.

```ts
public product(name: string) {
    return this.page.getByRole("link", {
        name
    });
}
```

---

# 9. CSS Locators

Use CSS only when an appropriate Playwright locator is not available.

Suitable cases include:

* No accessible locator exists
* No suitable label exists
* No suitable placeholder exists
* No stable test ID exists
* A stable CSS attribute uniquely identifies the element

### Correct

```ts
readonly productItems =
    this.page.locator(".product-item");
```

```ts
readonly emailInput =
    this.page.locator('input[name="email"]');
```

```ts
readonly productCards =
    this.page.locator('[data-component="product-card"]');
```

Prefer short, stable selectors.

### Avoid fragile CSS

```ts
this.page.locator(
    "body > div:nth-child(3) > div > ul > li"
);
```

Avoid selectors based on:

* `nth-child()`
* `nth-of-type()`
* Deep DOM traversal
* Dynamic class names
* Positional relationships
* Temporary styling classes

---

# 10. XPath

XPath is the final fallback.

Use XPath only when:

1. A suitable Playwright locator is unavailable.
2. A stable CSS locator is unavailable.
3. XPath provides a necessary relationship that cannot reasonably be expressed using Playwright or CSS.

### Acceptable

```ts
this.page.locator("//table//tr");
```

### Prefer relationship-based XPath when required

```ts
this.page.locator(
    "//button[@name='Submit']/parent::div"
);
```

```ts
this.page.locator(
    "//div[@class='row']//child::input"
);
```

```ts
this.page.locator(
    "//form/ancestor::div"
);
```

### Avoid

```ts
this.page.locator(
    "//*[@id='content']/div/div/div[2]/button"
);
```

Avoid XPath based on:

* Absolute DOM paths
* Numeric indexes
* `div[2]`
* `button[3]`
* Deep DOM traversal

---

# 11. Unique Locators

Every locator should identify the intended element uniquely.

### Correct

```ts
readonly checkoutButton =
    this.page.getByRole("button", {
        name: "Checkout"
    });
```

### Avoid

```ts
this.page.locator("button");
```

if multiple buttons exist.

Do not solve duplicate matches by immediately using `.first()` or `.nth()`.

Improve the locator instead.

---

# 12. Duplicate Locators

Never define the same locator multiple times within the same Page Object or Component.

### Incorrect

```ts
readonly loginButton = this.page.getByRole("button", {
    name: "Log in"
});

readonly btnLogin = this.page.getByRole("button", {
    name: "Log in"
});
```

### Correct

```ts
readonly loginButton = this.page.getByRole("button", {
    name: "Log in"
});
```

Reuse `loginButton`.

---

# 13. Dynamic Locators

When the locator depends on changing test data, create a reusable method.

### Correct

```ts
public product(name: string) {
    return this.page.getByRole("link", {
        name
    });
}
```

Usage:

```ts
await loginPage.product("Laptop").click();
```

### Avoid

Creating separate hard-coded locators for every product:

```ts
this.page.getByText("Laptop");
this.page.getByText("Mobile");
this.page.getByText("Tablet");
```

---

# 14. Locator Collections

Use locator collections for repeated elements.

### Correct

```ts
readonly cartItems =
    this.page.locator(".cart-item");
```

Count items with:

```ts
await this.cartItems.count();
```

Interact with individual items only when the UI genuinely requires positional selection.

---

# 15. Locator Chaining

Prefer locator chaining when it produces a more precise and readable locator.

### Correct

```ts
this.page
    .getByRole("row")
    .getByRole("button", {
        name: "Edit"
    });
```

Use chaining to narrow the search scope rather than creating brittle DOM selectors.

---

# 16. nth()

Avoid `.nth()` unless absolutely necessary.

### Avoid

```ts
locator.nth(3);
```

If `.nth()` is required, there must be a clear reason why the UI intentionally contains multiple equivalent elements.

Document the reason when appropriate.

Do not use `.nth()` simply because the locator is not unique.

---

# 17. first()

Avoid `.first()` unless the UI intentionally contains multiple matching elements and the first element is explicitly the required one.

### Avoid

```ts
this.page.getByRole("button").first();
```

Instead, improve the locator:

```ts
this.page.getByRole("button", {
    name: "Checkout"
});
```

---

# 18. last()

Avoid `.last()` for locator uniqueness.

### Avoid

```ts
this.page.getByRole("button").last();
```

Prefer a unique locator.

---

# 19. Shadow DOM

Use Playwright's built-in Shadow DOM support.

Do not use JavaScript execution to manually traverse Shadow DOM.

Avoid:

```ts
page.evaluate(() => {
    // manually traverse shadow root
});
```

Prefer Playwright locators whenever possible.

---

# 20. Frames

Use Playwright's `frameLocator()` for elements inside iframes.

### Correct

```ts
const paymentFrame =
    this.page.frameLocator("#payment-frame");

await paymentFrame
    .getByLabel("Card number")
    .fill("4111111111111111");
```

Avoid unnecessary manual frame handling.

---

# 21. Locator Naming

Locator names must describe the element and its purpose.

### Correct

```ts
loginButton

checkoutButton

emailTextbox

passwordTextbox

searchTextbox

cartItems

productCards

shoppingCartLink
```

### Avoid

```ts
btn

button1

textbox

locator

item1

element
```

Use meaningful names that communicate what the element represents.

---

# 22. Page Object Example

A Page Object should follow the locator strategy defined in this document.

```ts
import { Page } from "@playwright/test";

export class LoginPage {
    constructor(private page: Page) {}

    readonly emailTextbox =
        this.page.getByLabel("Email");

    readonly passwordTextbox =
        this.page.getByLabel("Password");

    readonly loginButton =
        this.page.getByRole("button", {
            name: "Log in"
        });

    async login(email: string, password: string) {
        await this.emailTextbox.fill(email);
        await this.passwordTextbox.fill(password);
        await this.loginButton.click();
    }
}
```

---

# 23. Test Example

Tests should consume Page Object methods and locators rather than defining raw locators.

### Correct

```ts
test("user can log in", async ({ page }) => {
    const loginPage = new LoginPage(page);

    await loginPage.login(
        "user@example.com",
        "password"
    );
});
```

### Avoid

```ts
test("user can log in", async ({ page }) => {
    await page.locator("#email").fill("user@example.com");
    await page.locator("#password").fill("password");
    await page.locator("#login").click();
});
```

---

# 24. MCP Locator Generation Rules

When Playwright MCP is used to inspect a page or generate locators:

1. Inspect the page before creating the locator.
2. Identify the element's accessible role and name.
3. Check whether `getByRole()` can identify it.
4. Check `getByLabel()` for form controls.
5. Check `getByPlaceholder()` when appropriate.
6. Check `getByTestId()` for stable test IDs.
7. Check `getByText()` for suitable static text.
8. Use CSS only when the above options are unsuitable.
9. Use XPath only when Playwright and CSS cannot provide a reliable locator.
10. Never generate XPath simply because it is possible.
11. Never generate a brittle CSS selector when a stable locator exists.
12. Always prefer readable and maintainable locators.

---

# 25. Locator Validation

Before finalizing generated Playwright code, verify that the locator:

* Identifies the intended element.
* Is as unique as reasonably possible.
* Is readable.
* Is stable.
* Uses the highest available locator priority.
* Does not depend unnecessarily on DOM structure.
* Does not use unnecessary positional selectors.
* Does not use XPath when a better locator exists.
* Belongs inside a Page Object or reusable Component.

---

# 26. Code Review Checklist

Before completing Playwright automation code, verify:

* [ ] Playwright accessibility locator was considered first.
* [ ] `getByRole()` was considered.
* [ ] `getByLabel()` was considered.
* [ ] `getByPlaceholder()` was considered.
* [ ] `getByTestId()` was considered.
* [ ] `getByText()` was considered where appropriate.
* [ ] CSS is used only when required.
* [ ] XPath is used only as a last resort.
* [ ] Locator is unique.
* [ ] Locator is readable.
* [ ] Locator is stable.
* [ ] No unnecessary `.first()`.
* [ ] No unnecessary `.last()`.
* [ ] No unnecessary `.nth()`.
* [ ] No fragile CSS selector.
* [ ] No absolute XPath.
* [ ] No duplicated locator.
* [ ] Dynamic locators are reusable.
* [ ] Locators are inside Page Objects or Components.
* [ ] No raw locators are unnecessarily placed inside spec/test files.
* [ ] Playwright MCP page inspection was used when available.

---

# 27. Mandatory Summary

Always follow this locator priority:

```text
1. getByRole()
       ↓
2. getByLabel()
       ↓
3. getByPlaceholder()
       ↓
4. getByTestId()
       ↓
5. getByText()
       ↓
6. CSS locator
       ↓
7. XPath
```

The fundamental rule is:

**Use the highest-priority Playwright locator that reliably identifies the element.**

**CSS is the fallback.**

**XPath is the last resort.**

**Never use a lower-priority locator when a reliable higher-priority locator is available.**
