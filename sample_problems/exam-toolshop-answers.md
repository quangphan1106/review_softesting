# Software Testing Final Examination - ToolShop Version
**Course:** CS423 - Software Testing
**Academic Year:** 2025-2026, Term I
**Application Under Test:** Practice Software Testing - ToolShop v5.0
**URL:** https://with-bugs.practicesoftwaretesting.com

---

## Part 1: Seminar (40%)

### 1a) Test Environment Solutions for ToolShop E-commerce

**Testing Types:**
- Functional Testing: Verify e-commerce features (registration, search, cart, checkout, payment)
- Integration Testing: Test module interactions (Cart ↔ Checkout ↔ Payment ↔ Invoice)
- Regression Testing: Ensure new changes don't break existing features
- Security Testing: SQL Injection, XSS, authentication bypass
- Performance Testing: Check response times under load

**Testing Tools:**
| Category | Tools |
|----------|-------|
| Web Automation | Selenium WebDriver, Cypress, Playwright |
| API Testing | Postman, REST Assured, Python requests |
| Performance | JMeter, Gatling, k6 |
| Security | OWASP ZAP, Burp Suite |
| Cross-browser | BrowserStack, Sauce Labs |

**Frameworks:**
- Selenium + Pytest (Python) - Data-driven with CSV
- Cypress with Mocha/Chai (JavaScript)
- Page Object Model (POM) design pattern
- Robot Framework (Keyword-driven)

---

### 1b) Differences: Mobile vs Web vs Desktop Automation Testing

| Aspect | Mobile Testing | Web Testing | Desktop Testing |
|--------|---------------|-------------|-----------------|
| **Tools** | Appium, Espresso, XCUITest | Selenium, Cypress, Playwright | WinAppDriver, TestComplete, UFT |
| **Platforms** | iOS, Android | Cross-browser (Chrome, Firefox, Edge) | Windows, macOS, Linux |
| **Locators** | Accessibility ID, XPath, UIAutomator | CSS Selector, XPath, ID, data-test | AutomationId, Name, ClassName |
| **Challenges** | Device fragmentation, gestures, orientation | Browser compatibility, responsive design | OS-specific APIs, native controls |
| **Network** | Must test offline/low connectivity | Generally stable connection | Usually local/LAN |
| **Performance** | Battery, memory constraints | Browser memory, rendering | System resources |

---

### 1c) Performance Testing vs Load Testing vs Stress Testing

| Aspect | Performance Testing | Load Testing | Stress Testing |
|--------|-------------------|--------------|----------------|
| **Goal** | Measure speed, responsiveness, stability | Test behavior under expected load | Find breaking point |
| **Load Level** | Normal conditions | Expected concurrent users | Beyond normal capacity |
| **Metrics** | Response time, throughput, resource usage | Throughput, error rate at load | Breaking point, recovery time |
| **Example ToolShop** | "Product page loads <2s" | "100 users add to cart simultaneously" | "What happens at 1000 concurrent checkouts?" |

**Why Performance Testing is Expensive:**
1. **Infrastructure Costs:** Production-like environments, load generators, monitoring tools
2. **Tool Licensing:** Enterprise tools (LoadRunner, NeoLoad) costly
3. **Specialized Skills:** Expertise in scripting, analysis, bottleneck identification
4. **Time-Intensive:** Script development, test execution, analysis cycles
5. **Environment Setup:** Dedicated test environments
6. **Data Requirements:** Large realistic datasets (products, users, orders)

---

### 1d) GUI Testing vs Usability Testing

| Aspect | GUI Testing | Usability Testing |
|--------|-------------|-------------------|
| **Focus** | Visual elements, functionality | User experience, ease of use |
| **Goal** | Verify UI works correctly | Verify users can accomplish tasks |
| **Method** | Automated/Manual scripted tests | User observation, feedback sessions |
| **Metrics** | Pass/Fail, defects found | Task completion rate, time, satisfaction |
| **Who Tests** | QA Engineers | Real end users |

**Applying GUI Testing to ToolShop:**
1. **Navigation Testing:** Verify menu, categories, product links work correctly
2. **Form Validation:** Test checkout form, registration form, contact form
3. **Layout Testing:** Check element alignment, responsive design (mobile/tablet/desktop)
4. **Cross-browser Testing:** Verify UI consistency across Chrome, Firefox, Safari, Edge
5. **Accessibility Testing:** Screen reader compatibility, color contrast, keyboard navigation
6. **Visual Regression:** Compare screenshots to detect unintended UI changes
7. **Payment UI:** Test credit card form, payment method selection, error displays

---

### 1e) Benefits of CI System

1. **Early Bug Detection:** Automated tests run on every commit, catching issues immediately
2. **Faster Feedback:** Developers know within minutes if their code breaks something
3. **Reduced Integration Risk:** Frequent small integrations vs. big-bang merges
4. **Consistent Builds:** Automated, reproducible build process
5. **Quality Gates:** Enforce code coverage, coding standards before merge
6. **Deployment Automation:** Enables CD (Continuous Deployment) pipeline
7. **Team Collaboration:** Shared visibility into build status, test results
8. **Documentation:** Build history provides audit trail

---

### 1f) 5 Key Features of API Testing Tools

1. **Request Builder:** Create HTTP requests (GET, POST, PUT, DELETE) with headers, body, auth
2. **Response Validation:** Assert status codes, response body (JSON schema), headers, response time
3. **Environment Management:** Variables for different environments (dev, staging, prod)
4. **Test Automation:** Write test scripts, chain requests, data-driven testing with CSV/JSON
5. **Documentation:** Auto-generate API docs from OpenAPI/Swagger specs

**Applied to ToolShop API:**
- Base URL: `https://api.practicesoftwaretesting.com`
- Endpoints: `/products`, `/carts`, `/invoices`, `/users/login`, `/payment/check`
- Auth: Bearer token from `/users/login`

---

### 1g) DB Testing for ToolShop

**Yes, DB Testing is necessary for ToolShop:**

**Reasons:**
1. **Data Integrity:** Product inventory, user accounts, order history must be accurate
2. **CRUD Operations:** Verify Create/Read/Update/Delete work correctly
3. **Constraints:** Check foreign keys (order→user, cart→product), unique constraints
4. **Stock Management:** Verify inventory decreases after purchase
5. **Price Calculations:** Test total calculation, discounts, taxes
6. **Order Status:** Verify status transitions (pending→paid→shipped→delivered)
7. **Security:** SQL injection prevention, access controls

**What to Test:**
- Product data CRUD
- User registration data
- Cart items persistence
- Invoice/Order records
- Payment transaction logs

---

### 1h) Test Levels for ToolShop

| Level | Description | Example in ToolShop |
|-------|-------------|---------------------|
| **Unit Testing** | Test individual functions/methods | Test price calculation function |
| **Integration Testing** | Test module interactions | Cart + Checkout + Payment integration |
| **System Testing** | Test complete system | Full purchase workflow end-to-end |
| **Acceptance Testing** | Validate business requirements | Customer validates checkout flow |

**Additional Applicable Levels:**
- **Smoke Testing:** Quick sanity check after deployment (login, search, add to cart)
- **Regression Testing:** After each release/change
- **User Acceptance Testing (UAT):** End users validate functionality
- **Security Testing:** SQL injection, XSS, authentication bypass

---

## Part 2: Exercises (60%)

### 2a) Domain Testing - Checkout Feature (30%)

**Feature:** Complete checkout process in ToolShop
**URL:** https://with-bugs.practicesoftwaretesting.com/checkout

#### Domain Analysis (15%)

**Input Fields Analysis:**

| Field | Data Type | Valid Domain | Invalid Domain | Constraints |
|-------|-----------|--------------|----------------|-------------|
| First Name | String | 2-50 chars, letters/spaces | <2 or >50 chars, special chars, numbers | Required |
| Last Name | String | 2-50 chars, letters/spaces | <2 or >50 chars, special chars, numbers | Required |
| Address | String | 5-70 chars | <5 or >70 chars, empty | Required |
| City | String | 2-40 chars, letters | <2 or >40 chars, numbers | Required |
| State | String | 2-40 chars, letters | <2 or >40 chars | Required |
| Country | String | 2-40 chars, letters | <2 or >40 chars | Required |
| Postcode | String | 4-10 chars, alphanumeric | <4 or >10 chars, special chars | Required |
| Payment Method | Select | {Bank Transfer, Cash on Delivery, Credit Card, Buy Now Pay Later, Gift Card} | Empty selection | Required |
| Account Name | String | Valid cardholder name | Empty, special chars | Required for CC |
| Account Number | String | Valid card number (16 digits for Visa/MC) | Invalid format, wrong length | Required for CC |

**Equivalence Classes:**

**First Name / Last Name:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC1 | Valid | 2 chars (minimum) | "Jo" |
| EC2 | Valid | 25 chars (mid-range) | "Alexander" |
| EC3 | Valid | 50 chars (maximum) | "Alexandrinamargaretelisabethanne" |
| EC4 | Invalid | 1 char (below min) | "J" |
| EC5 | Invalid | 51 chars (above max) | "Alexandrinamargaretelisabethannem" |
| EC6 | Invalid | Contains numbers | "John123" |
| EC7 | Invalid | Contains special chars | "John@Doe" |
| EC8 | Invalid | Empty | "" |

**Address:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC9 | Valid | 5 chars (minimum) | "1 Oak" |
| EC10 | Valid | 35 chars (mid-range) | "123 Main Street, Apartment 4B" |
| EC11 | Valid | 70 chars (maximum) | "12345 Very Long Street Name..." |
| EC12 | Invalid | 4 chars (below min) | "1 St" |
| EC13 | Invalid | 71 chars (above max) | "Too long address..." |
| EC14 | Invalid | Empty | "" |

**Postcode:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC15 | Valid | 4 chars (minimum) | "1234" |
| EC16 | Valid | 7 chars (mid-range) | "12345AB" |
| EC17 | Valid | 10 chars (maximum) | "1234567890" |
| EC18 | Invalid | 3 chars (below min) | "123" |
| EC19 | Invalid | 11 chars (above max) | "12345678901" |
| EC20 | Invalid | Special chars | "123-456" |

**Credit Card Number:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC21 | Valid | Visa (16 digits, starts 4) | "4111111111111111" |
| EC22 | Valid | Mastercard (16 digits, starts 5) | "5555555555554444" |
| EC23 | Valid | Amex (15 digits, starts 34/37) | "371449635398431" |
| EC24 | Invalid | 15 digits (not Amex) | "411111111111111" |
| EC25 | Invalid | 17 digits | "41111111111111111" |
| EC26 | Invalid | Contains letters | "4111XXXX11111111" |
| EC27 | Invalid | Empty | "" |

**Payment Method:**
| Class ID | Type | Description |
|----------|------|-------------|
| EC28 | Valid | Bank Transfer |
| EC29 | Valid | Cash on Delivery |
| EC30 | Valid | Credit Card |
| EC31 | Valid | Buy Now Pay Later |
| EC32 | Valid | Gift Card |
| EC33 | Invalid | No selection |

---

#### Test Cases Design (15%)

| TC ID | Test Case Name | First Name | Last Name | Address | City | State | Country | Postcode | Payment | Card Number | Expected Result |
|-------|----------------|------------|-----------|---------|------|-------|---------|----------|---------|-------------|-----------------|
| TC01 | All valid - Credit Card | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Success |
| TC02 | All valid - Bank Transfer | Jane | Doe | 456 Oak Ave | Melbourne | VIC | Australia | 3000 | Bank Transfer | - | Success |
| TC03 | All valid - Cash on Delivery | Bob | Lee | 789 Pine Rd | Brisbane | QLD | Australia | 4000 | Cash on Delivery | - | Success |
| TC04 | First name too short | J | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: Min 2 chars |
| TC05 | First name too long | [51 chars] | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: Max 50 chars |
| TC06 | First name with numbers | John123 | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: No numbers |
| TC07 | Empty first name | - | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: Required |
| TC08 | Address too short | John | Smith | 1 St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: Min 5 chars |
| TC09 | Postcode too short | John | Smith | 123 Main St | Sydney | NSW | Australia | 123 | Credit Card | 4111111111111111 | Error: Min 4 chars |
| TC10 | Invalid card - wrong length | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 411111111111 | Error: Invalid card |
| TC11 | Invalid card - letters | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111XXXX11111111 | Error: Invalid card |
| TC12 | Empty card number | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | - | Error: Required |
| TC13 | No payment selected | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | - | - | Error: Required |
| TC14 | Valid Mastercard | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 5555555555554444 | Success |
| TC15 | Valid Gift Card | John | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Gift Card | 1234567890123456 | Success |
| TC16 | SQL Injection in name | ' OR 1=1-- | Smith | 123 Main St | Sydney | NSW | Australia | 2000 | Credit Card | 4111111111111111 | Error: Sanitized |

**Boundary Value Test Cases:**

| TC ID | Field | Boundary | Value | Expected |
|-------|-------|----------|-------|----------|
| BV01 | First Name | Min-1 | 1 char | Error |
| BV02 | First Name | Min | 2 chars | Success |
| BV03 | First Name | Min+1 | 3 chars | Success |
| BV04 | First Name | Max-1 | 49 chars | Success |
| BV05 | First Name | Max | 50 chars | Success |
| BV06 | First Name | Max+1 | 51 chars | Error |
| BV07 | Address | Min-1 | 4 chars | Error |
| BV08 | Address | Min | 5 chars | Success |
| BV09 | Address | Max | 70 chars | Success |
| BV10 | Address | Max+1 | 71 chars | Error |
| BV11 | Postcode | Min-1 | 3 chars | Error |
| BV12 | Postcode | Min | 4 chars | Success |
| BV13 | Postcode | Max | 10 chars | Success |
| BV14 | Postcode | Max+1 | 11 chars | Error |
| BV15 | Card Number | 15 digits (Visa) | Invalid | Error |
| BV16 | Card Number | 16 digits | Valid | Success |
| BV17 | Card Number | 17 digits | Invalid | Error |

---

### 2b) State Transition Testing - E-commerce Order System (30%)

**Scenario:** ToolShop e-commerce order management system

**Business Rules:**
- Guest users can browse products and add to cart
- Users must register/login to checkout
- Cart persists for logged-in users
- Orders go through multiple states: Cart → Checkout → Payment → Processing → Shipped → Delivered
- Users can cancel orders before shipping
- Payment can fail, requiring retry
- Stock is reserved during checkout, released if payment fails
- Registered users have order history
- Users can track order status

#### State Chart Diagram for Order System (10%)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     TOOLSHOP ORDER STATE DIAGRAM                            │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────┐
                              │    START     │
                              │   (Guest)    │
                              └──────┬───────┘
                                     │ Browse products
                                     ▼
                              ┌──────────────┐
                              │   BROWSING   │◄────────────────┐
                              │              │                 │
                              └──────┬───────┘                 │
                                     │ Add to cart             │
                                     ▼                         │
                              ┌──────────────┐                 │
                              │    CART      │─────────────────┤
                              │   (Items)    │  Remove all     │
                              └──────┬───────┘                 │
                                     │ Proceed to checkout     │
                                     ▼                         │
                        ┌────────────────────────┐             │
                        │  AUTHENTICATION CHECK  │             │
                        └────────────┬───────────┘             │
                    ┌────────────────┴────────────────┐        │
                    │ Not logged in   │ Logged in     │        │
                    ▼                 ▼               │        │
            ┌───────────┐     ┌───────────────┐      │        │
            │  LOGIN/   │────►│   CHECKOUT    │      │        │
            │ REGISTER  │     │ (Enter details)│      │        │
            └───────────┘     └───────┬───────┘      │        │
                                      │ Submit payment│        │
                                      ▼               │        │
                              ┌──────────────┐       │        │
                              │   PAYMENT    │       │        │
                              │  PROCESSING  │       │        │
                              └──────┬───────┘       │        │
                    ┌────────────────┴────────────────┐        │
                    │ Success         │ Failed        │        │
                    ▼                 ▼               │        │
            ┌───────────┐     ┌───────────────┐      │        │
            │ ORDER     │     │   PAYMENT     │──────┘        │
            │ CONFIRMED │     │   FAILED      │───────────────┘
            └─────┬─────┘     │  (Retry/Cancel)│
                  │           └───────────────┘
                  │ Processing
                  ▼
            ┌───────────┐
            │PROCESSING │
            │           │
            └─────┬─────┘
                  │
      ┌───────────┴───────────┐
      │ Ship                  │ Cancel (before ship)
      ▼                       ▼
┌───────────┐          ┌───────────┐
│  SHIPPED  │          │ CANCELLED │
│           │          │           │
└─────┬─────┘          └───────────┘
      │ Deliver
      ▼
┌───────────┐
│ DELIVERED │
│           │
└─────┬─────┘
      │ Return request
      ▼
┌───────────┐
│ RETURNED  │ (if within return window)
│           │
└───────────┘


LEGEND:
─────────────────────────────────────────────────────────────────────────
Guest: Can browse, add to cart (session-based)
Registered User: Cart persists, order history, can track orders
Payment Methods: Credit Card, Bank Transfer, Cash on Delivery, Gift Card
─────────────────────────────────────────────────────────────────────────
```

**States Description:**

| State | Description |
|-------|-------------|
| START | Initial state, guest user |
| BROWSING | User viewing products |
| CART | User has items in cart |
| AUTH_CHECK | System checks if user logged in |
| LOGIN/REGISTER | User authentication required |
| CHECKOUT | User entering shipping/payment details |
| PAYMENT_PROCESSING | Payment being validated |
| PAYMENT_FAILED | Payment declined/error |
| ORDER_CONFIRMED | Payment successful, order created |
| PROCESSING | Order being prepared |
| SHIPPED | Order sent to customer |
| DELIVERED | Order received by customer |
| CANCELLED | Order cancelled by user/system |
| RETURNED | Order returned by customer |

---

#### State Transition Tables (10%)

**Main Order State Transition Table:**

| Current State | Event/Condition | Action | Next State |
|---------------|-----------------|--------|------------|
| START | View products | Display catalog | BROWSING |
| BROWSING | Add product to cart | Create/update cart | CART |
| BROWSING | Search/Filter | Display filtered results | BROWSING |
| CART | Add more items | Update cart quantity | CART |
| CART | Remove item | Update cart | CART (or BROWSING if empty) |
| CART | Proceed to checkout | Check authentication | AUTH_CHECK |
| AUTH_CHECK | Not logged in | Redirect to login | LOGIN/REGISTER |
| AUTH_CHECK | Logged in | Show checkout form | CHECKOUT |
| LOGIN/REGISTER | Login success | Merge session cart | CHECKOUT |
| LOGIN/REGISTER | Register success | Create account, continue | CHECKOUT |
| CHECKOUT | Submit valid details | Process payment | PAYMENT_PROCESSING |
| CHECKOUT | Invalid details | Show errors | CHECKOUT |
| PAYMENT_PROCESSING | Payment success | Create invoice | ORDER_CONFIRMED |
| PAYMENT_PROCESSING | Payment failed | Show error | PAYMENT_FAILED |
| PAYMENT_FAILED | Retry payment | Re-process | PAYMENT_PROCESSING |
| PAYMENT_FAILED | Cancel | Release stock | BROWSING |
| ORDER_CONFIRMED | Process order | Prepare shipment | PROCESSING |
| PROCESSING | Ship order | Update status | SHIPPED |
| PROCESSING | Cancel request | Release stock, refund | CANCELLED |
| SHIPPED | Confirm delivery | Update status | DELIVERED |
| DELIVERED | Return request (≤14 days) | Initiate return | RETURNED |
| DELIVERED | Return request (>14 days) | Deny return | DELIVERED |

**Cart Operations Table:**

| Current Cart | Action | Condition | Result |
|--------------|--------|-----------|--------|
| Empty | Add product | Product in stock | Cart with 1 item |
| Empty | Checkout | - | Error: Cart empty |
| Has items | Add same product | In stock | Quantity +1 |
| Has items | Add same product | Out of stock | Error: Insufficient stock |
| Has items | Remove item | Last item | Empty cart |
| Has items | Remove item | Multiple items | Cart updated |
| Has items | Update quantity | Valid quantity | Cart updated |
| Has items | Update quantity | > stock | Error: Max stock |

**Payment Processing Table:**

| Payment Method | Condition | Validation | Result |
|----------------|-----------|------------|--------|
| Credit Card | Valid card number | Luhn check pass | PAYMENT_SUCCESS |
| Credit Card | Invalid card number | Luhn check fail | PAYMENT_FAILED |
| Credit Card | Expired card | Date check fail | PAYMENT_FAILED |
| Bank Transfer | Valid account | Account verified | PAYMENT_SUCCESS |
| Cash on Delivery | Always | No validation | ORDER_CONFIRMED (COD) |
| Gift Card | Valid code, sufficient balance | Code + balance check | PAYMENT_SUCCESS |
| Gift Card | Invalid code | Code check fail | PAYMENT_FAILED |
| Gift Card | Insufficient balance | Balance check fail | PAYMENT_FAILED |

---

#### Test Cases Design (10%)

**Order Lifecycle Test Cases:**

| TC ID | Test Scenario | Precondition | Action | Expected Result |
|-------|---------------|--------------|--------|-----------------|
| ST01 | Guest browses products | New session | View product list | Products displayed |
| ST02 | Guest adds to cart | Browsing products | Add product | Cart created with item |
| ST03 | Guest checkout redirect | Cart has items | Click checkout | Redirect to login |
| ST04 | User login before checkout | At login page | Enter valid credentials | Redirect to checkout |
| ST05 | Register new user | At register page | Submit valid details | Account created, to checkout |
| ST06 | Submit valid checkout | At checkout, logged in | Enter valid details + CC | Order confirmed |
| ST07 | Payment with Bank Transfer | At checkout | Select Bank Transfer | Order confirmed |
| ST08 | Payment with COD | At checkout | Select Cash on Delivery | Order confirmed |
| ST09 | Payment failed - invalid card | At checkout | Enter invalid CC | Error: Payment failed |
| ST10 | Retry failed payment | Payment failed | Click retry, fix card | Order confirmed |
| ST11 | Cancel from payment failed | Payment failed | Click cancel | Return to cart |
| ST12 | Order processing | Order confirmed | System processes | Status: Processing |
| ST13 | Order shipped | Processing complete | Admin ships | Status: Shipped |
| ST14 | Order delivered | Shipped | Confirm delivery | Status: Delivered |
| ST15 | Cancel before shipping | Order confirmed | Request cancel | Status: Cancelled, refund |
| ST16 | Cancel after shipping | Shipped | Request cancel | Error: Cannot cancel |
| ST17 | Return within 14 days | Delivered <14 days | Request return | Return initiated |
| ST18 | Return after 14 days | Delivered >14 days | Request return | Error: Window expired |

**Cart State Test Cases:**

| TC ID | Test Scenario | Cart State | Action | Expected Result |
|-------|---------------|------------|--------|-----------------|
| ST19 | Add first item | Empty | Add product (stock: 10) | Cart: 1 item, qty: 1 |
| ST20 | Add same item | 1 item (qty: 1) | Add same product | Cart: 1 item, qty: 2 |
| ST21 | Add different item | 1 item | Add different product | Cart: 2 items |
| ST22 | Remove single item | 1 item | Remove item | Cart empty |
| ST23 | Update quantity | 1 item (qty: 1) | Set qty: 5 | Cart: qty: 5 |
| ST24 | Update quantity > stock | 1 item, stock: 3 | Set qty: 10 | Error: Max 3 available |
| ST25 | Checkout empty cart | Empty | Click checkout | Error: Add items first |
| ST26 | Cart persists on login | Guest cart: 2 items | Login | Cart preserved: 2 items |

**Payment Method Test Cases:**

| TC ID | Test Scenario | Payment Method | Input | Expected Result |
|-------|---------------|----------------|-------|-----------------|
| ST27 | Valid Visa | Credit Card | 4111111111111111 | Payment success |
| ST28 | Valid Mastercard | Credit Card | 5555555555554444 | Payment success |
| ST29 | Invalid card number | Credit Card | 1234567890123456 | Payment failed |
| ST30 | Valid bank transfer | Bank Transfer | Valid account | Payment success |
| ST31 | Valid COD | Cash on Delivery | - | Order confirmed |
| ST32 | Valid gift card | Gift Card | GIFT-1234-5678 | Payment success |
| ST33 | Invalid gift card | Gift Card | INVALID-CODE | Payment failed |
| ST34 | Gift card insufficient | Gift Card | GIFT-LOW-BAL | Payment failed |

**Security Test Cases:**

| TC ID | Test Scenario | Input | Expected Result |
|-------|---------------|-------|-----------------|
| ST35 | SQL Injection in search | `' OR 1=1--` | Query sanitized, no data leak |
| ST36 | XSS in checkout name | `<script>alert(1)</script>` | Script escaped/rejected |
| ST37 | Access order without login | Direct URL to order | Redirect to login |
| ST38 | Access other user's order | Logged in, different user's order ID | Access denied |

**State Transition Coverage Matrix:**

| From State | To State | Covered by TC |
|------------|----------|---------------|
| START → BROWSING | ST01 |
| BROWSING → CART | ST02 |
| CART → AUTH_CHECK | ST03 |
| AUTH_CHECK → LOGIN | ST03 |
| LOGIN → CHECKOUT | ST04 |
| REGISTER → CHECKOUT | ST05 |
| CHECKOUT → PAYMENT_PROCESSING | ST06 |
| PAYMENT_PROCESSING → ORDER_CONFIRMED | ST06, ST07, ST08 |
| PAYMENT_PROCESSING → PAYMENT_FAILED | ST09 |
| PAYMENT_FAILED → PAYMENT_PROCESSING | ST10 |
| PAYMENT_FAILED → CART | ST11 |
| ORDER_CONFIRMED → PROCESSING | ST12 |
| PROCESSING → SHIPPED | ST13 |
| PROCESSING → CANCELLED | ST15 |
| SHIPPED → DELIVERED | ST14 |
| DELIVERED → RETURNED | ST17 |

---

## Summary

| Part | Topic | Max Marks |
|------|-------|-----------|
| 1a | Test environment solutions | ~5% |
| 1b | Mobile/Web/Desktop differences | ~5% |
| 1c | Performance/Load/Stress testing | ~5% |
| 1d | GUI vs Usability testing | ~5% |
| 1e | CI system benefits | ~5% |
| 1f | API testing tools features | ~5% |
| 1g | DB testing for ToolShop | ~5% |
| 1h | Test levels for ToolShop | ~5% |
| 2a | Domain testing - Checkout | 30% |
| 2b | State transition testing - Order System | 30% |
| **Total** | | **100%** |

---

## References

- ToolShop Main: https://practicesoftwaretesting.com/
- ToolShop (with bugs): https://with-bugs.practicesoftwaretesting.com/
- API Documentation: https://api.practicesoftwaretesting.com/api/documentation
- Test Accounts:
  - Admin: admin@practicesoftwaretesting.com / welcome01
  - Customer: customer@practicesoftwaretesting.com / welcome01
