# Software Testing Final Examination - ToolShop Version (Concept Only)
**Course:** CS423 - Software Testing
**Time:** 90 minutes
**Term:** I - Academic Year: 2025-2026
**Application Under Test:** Practice Software Testing - ToolShop v5.0
**URL:** https://with-bugs.practicesoftwaretesting.com

*(Notes: Neither books, phones nor laptops allowed / Không sử dụng tài liệu)*

---

## 1. Seminar (40%)

### a) Suggest the test environment solutions (testing types, testing tools, frameworks) to test ToolShop e-commerce application. (5%)

### b) What are the differences among Mobile, Web, Desktop Automation Testing Tools? (5%)

### c) What are the differences between Performance Testing, Load Testing, Stress Testing, and Spike Testing? Why is performance testing so expensive? (5%)

### d) What are the differences between GUI Testing and Usability Testing? How can you apply GUI Testing to test ToolShop checkout page? (5%)

### e) What are the benefits of CI (Continuous Integration) system? (5%)

### f) List 5 key features of API Testing tools. (5%)

### g) Do we need to apply DB Testing for ToolShop? Explain why. (5%)

### h) Which test levels can you apply for ToolShop? Give examples for each level. (5%)

---

## 2. Exercises (60%)

### a) Apply domain testing to design test cases for User Registration feature in ToolShop: (30%)

**Feature URL:** https://with-bugs.practicesoftwaretesting.com/auth/register

**Input fields:**
- **First Name:** text, 2-50 characters, letters and spaces only
- **Last Name:** text, 2-50 characters, letters and spaces only
- **Date of Birth:** date picker, user must be ≥18 years old
- **Address:** text, 5-70 characters
- **City:** text, 2-40 characters
- **State:** text, 2-40 characters
- **Country:** text, 2-40 characters
- **Postcode:** text, 4-10 characters, alphanumeric
- **Phone:** text, valid phone number format
- **Email:** text, valid email format, unique in system
- **Password:** text, 8-50 characters, must contain uppercase, lowercase, number

**Requirements:**
- Analyze the domain for inputs (15%)
- Design Test Cases (15%)

---

### b) State transition testing (30%)

**Scenario:** A company specializing in providing online tool rental and purchase service has been established. Customers can go to company website to register and become members. Members can browse products, add items to cart, and complete purchases.

**Business Rules:**

- **Guest users** can browse products, search, filter, and add items to cart (session-based cart)
- **To checkout**, users must register or login
- After registration, users get a member account with status **ACTIVE**
- Member account is valid for **1 year** from registration date
- System sends **reminder email 2 weeks** before account expiration
- If member doesn't renew within **30 days** after expiration, account is **LOCKED**
- If member doesn't renew within **1 year** after expiration, account is **REMOVED** from system

**Order System:**
- Orders go through states: **CART → CHECKOUT → PAYMENT_PROCESSING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED**
- Payment can **FAIL**, requiring retry or cancellation
- Users can **CANCEL** orders before shipping
- Users can **RETURN** orders within **14 days** after delivery

**Member Types:**
- **Standard Member** (Free): Maximum 10 orders per month, standard shipping
- **Premium Member** ($50/year): Unlimited orders, free shipping, 10% discount on all items

**Additional Rules:**
- When Standard members reach 10 orders/month, they cannot place more orders until next month
- Premium members get priority processing
- System does not handle cases of lost shipments (assume all shipments arrive)
- Other business rules, students can make reasonable assumptions

**Requirements:**
- Use the above information to:
  - Draw state chart diagram for **Order System** (10%)
  - Design state transition tables (10%)
  - Design test cases (10%)

---

# ANSWER KEY

---

## 1. Seminar Answers (40%)

### 1a) Test Environment Solutions for ToolShop (5%)

**Testing Types:**
| Type | Purpose | Example in ToolShop |
|------|---------|---------------------|
| Functional Testing | Verify features work correctly | Registration, login, checkout flow |
| Integration Testing | Test module interactions | Cart ↔ Checkout ↔ Payment |
| Regression Testing | Ensure changes don't break existing features | After bug fixes |
| Security Testing | Find vulnerabilities | SQL injection, XSS in search |
| Performance Testing | Check speed & scalability | 100 concurrent users checkout |
| API Testing | Verify backend endpoints | /products, /carts, /invoices |

**Testing Tools:**
| Category | Tools |
|----------|-------|
| Web Automation | Selenium WebDriver, Cypress, Playwright |
| API Testing | Postman, REST Assured, Python requests |
| Performance | Apache JMeter, Gatling, k6 |
| Security | OWASP ZAP, Burp Suite |
| Cross-browser | BrowserStack, Sauce Labs |

**Frameworks:**
- Selenium + Pytest (Python) with Page Object Model
- Cypress with Mocha/Chai (JavaScript)
- Robot Framework (Keyword-driven)
- TestNG/JUnit (Java)

---

### 1b) Mobile vs Web vs Desktop Automation Testing (5%)

| Aspect | Mobile Testing | Web Testing | Desktop Testing |
|--------|---------------|-------------|-----------------|
| **Tools** | Appium, Espresso, XCUITest | Selenium, Cypress, Playwright | WinAppDriver, TestComplete |
| **Platforms** | iOS, Android | Cross-browser | Windows, macOS, Linux |
| **Locators** | Accessibility ID, UIAutomator | CSS, XPath, ID | AutomationId, ClassName |
| **Challenges** | Device fragmentation, gestures | Browser compatibility | OS-specific APIs |
| **Network** | Must test offline/poor connectivity | Generally stable | Usually local |
| **Resources** | Battery, memory constraints | Browser memory | System resources |

---

### 1c) Performance vs Load vs Stress vs Spike Testing (5%)

| Aspect | Performance | Load | Stress | Spike |
|--------|------------|------|--------|-------|
| **Goal** | Measure baseline speed | Test expected load | Find breaking point | Test sudden traffic |
| **Load Level** | Normal | Expected users | Beyond capacity | Sudden extreme |
| **Metrics** | Response time, throughput | Throughput, error rate | Breaking point, recovery | Recovery time |
| **Example** | "Page loads <2s" | "50 concurrent logins" | "What happens at 500 users?" | "Black Friday sale surge" |

**Why Performance Testing is Expensive:**
1. **Infrastructure:** Need production-like environments, load generators
2. **Tool Licensing:** JMeter free, but LoadRunner/NeoLoad costly
3. **Specialized Skills:** Expertise in scripting, analysis, bottleneck identification
4. **Time:** Script development, multiple test runs, analysis cycles
5. **Environment:** Dedicated test environments, realistic data sets
6. **Monitoring Tools:** APM tools, server monitoring

---

### 1d) GUI Testing vs Usability Testing (5%)

| Aspect | GUI Testing | Usability Testing |
|--------|-------------|-------------------|
| **Focus** | Visual elements, functionality | User experience |
| **Goal** | Verify UI works correctly | Verify users can complete tasks |
| **Method** | Scripted tests (manual/automated) | User observation, surveys |
| **Metrics** | Pass/Fail, defects | Task completion rate, satisfaction |
| **Who Tests** | QA Engineers | Real end users |
| **Output** | Bug reports | UX recommendations |

**Applying GUI Testing to ToolShop Checkout:**
1. **Form Validation:** Required fields, input formats, error messages
2. **Navigation:** Back/forward buttons, progress indicators
3. **Layout:** Element alignment, responsive design (mobile/tablet/desktop)
4. **Cross-browser:** Chrome, Firefox, Safari, Edge consistency
5. **Visual Regression:** Compare screenshots before/after changes
6. **Accessibility:** Keyboard navigation, screen reader, color contrast
7. **Error States:** Invalid card, empty fields, network errors

---

### 1e) Benefits of CI System (5%)

1. **Early Bug Detection:** Tests run on every commit, catch issues immediately
2. **Fast Feedback:** Developers know within minutes if code breaks
3. **Reduced Integration Risk:** Small frequent integrations vs big-bang merge
4. **Consistent Builds:** Automated, reproducible build process
5. **Quality Gates:** Enforce code coverage, linting before merge
6. **Deployment Ready:** Enables CD (Continuous Deployment)
7. **Team Visibility:** Shared dashboard shows build/test status
8. **Documentation:** Build history provides audit trail

---

### 1f) 5 Key Features of API Testing Tools (5%)

1. **Request Builder:** Create HTTP requests (GET, POST, PUT, DELETE) with headers, body, authentication
2. **Response Validation:** Assert status codes, JSON/XML body, headers, response time
3. **Environment Management:** Variables for dev/staging/prod, secrets management
4. **Test Automation:** Chain requests, data-driven testing with CSV/JSON, pre/post scripts
5. **Documentation:** Auto-generate API docs from collections, share with team

**Additional Features:**
- Mock servers for development
- Collection runner for batch testing
- Monitoring & scheduled runs

---

### 1g) DB Testing for ToolShop (5%)

**Yes, DB Testing is necessary for ToolShop.**

**Reasons:**
| Area | Why It's Needed |
|------|-----------------|
| **Data Integrity** | Product prices, stock levels, order totals must be accurate |
| **CRUD Operations** | Verify Create/Read/Update/Delete work correctly |
| **Constraints** | Foreign keys (order→user), unique (email), not-null |
| **Business Logic** | Stock decreases after purchase, totals calculated correctly |
| **Data Migration** | Verify data integrity during version upgrades |
| **Security** | Prevent SQL injection, verify access controls |
| **Performance** | Index optimization, query performance |

**What to Test:**
- User registration creates correct records
- Order creates invoice with correct line items
- Stock updates after purchase
- Cart items persist for logged-in users
- Payment records stored securely

---

### 1h) Test Levels for ToolShop (5%)

| Level | Definition | Example in ToolShop |
|-------|------------|---------------------|
| **Unit Testing** | Test individual functions/methods | Price calculation function, discount logic |
| **Integration Testing** | Test module interactions | Cart + Checkout + Payment flow |
| **System Testing** | Test complete end-to-end system | Full purchase from browse to delivery |
| **Acceptance Testing** | Validate business requirements | Customer validates checkout meets needs |

**Additional Levels:**
| Level | Example |
|-------|---------|
| **Smoke Testing** | Quick check after deployment (login, search, add to cart) |
| **Regression Testing** | Run after bug fix to ensure no new issues |
| **UAT** | Real users test before release |
| **Security Testing** | SQL injection in search, XSS in forms |

---

## 2. Exercise Answers (60%)

### 2a) Domain Testing - User Registration (30%)

#### Domain Analysis (15%)

**Equivalence Classes for Each Field:**

**First Name / Last Name:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC1 | Valid | 2 chars (min) | "Jo" |
| EC2 | Valid | 25 chars (mid) | "Alexander" |
| EC3 | Valid | 50 chars (max) | "Alexandrinamargaret..." |
| EC4 | Invalid | 1 char (below min) | "J" |
| EC5 | Invalid | 51 chars (above max) | "Too long..." |
| EC6 | Invalid | Contains numbers | "John123" |
| EC7 | Invalid | Special chars | "John@Doe" |
| EC8 | Invalid | Empty | "" |

**Email:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC9 | Valid | Standard format | "user@domain.com" |
| EC10 | Valid | With subdomain | "user@mail.company.co.uk" |
| EC11 | Invalid | Missing @ | "userdomain.com" |
| EC12 | Invalid | Missing domain | "user@" |
| EC13 | Invalid | Duplicate email | Already registered |
| EC14 | Invalid | Empty | "" |

**Password:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC15 | Valid | 8 chars (min) | "Pass123!" |
| EC16 | Valid | 25 chars (mid) | "MySecurePassword123" |
| EC17 | Valid | 50 chars (max) | Long password... |
| EC18 | Invalid | 7 chars (below min) | "Pass12" |
| EC19 | Invalid | 51 chars (above max) | Too long... |
| EC20 | Invalid | No uppercase | "password123" |
| EC21 | Invalid | No lowercase | "PASSWORD123" |
| EC22 | Invalid | No number | "PasswordOnly" |
| EC23 | Invalid | Empty | "" |

**Date of Birth:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC24 | Valid | Exactly 18 years | DOB = today - 18 years |
| EC25 | Valid | Over 18 years | DOB = 1990-01-01 |
| EC26 | Invalid | Under 18 years | DOB = today - 17 years |
| EC27 | Invalid | Future date | DOB = tomorrow |
| EC28 | Invalid | Empty | "" |

**Postcode:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC29 | Valid | 4 chars (min) | "1234" |
| EC30 | Valid | 10 chars (max) | "1234567890" |
| EC31 | Invalid | 3 chars (below min) | "123" |
| EC32 | Invalid | 11 chars (above max) | "12345678901" |
| EC33 | Invalid | Special chars | "123-456" |

---

#### Test Cases Design (15%)

| TC ID | Test Case | First Name | Last Name | DOB | Email | Password | Expected |
|-------|-----------|------------|-----------|-----|-------|----------|----------|
| TC01 | All valid minimum | Jo | Le | 18 yrs ago | jo@test.com | Pass123! | Success |
| TC02 | All valid maximum | [50 chars] | [50 chars] | 1970-01-01 | max@test.com | [50 chars] | Success |
| TC03 | First name too short | J | Smith | 1990-01-01 | test@test.com | Pass123! | Error |
| TC04 | First name too long | [51 chars] | Smith | 1990-01-01 | test@test.com | Pass123! | Error |
| TC05 | First name with numbers | John123 | Smith | 1990-01-01 | test@test.com | Pass123! | Error |
| TC06 | Empty first name | [empty] | Smith | 1990-01-01 | test@test.com | Pass123! | Error |
| TC07 | Invalid email format | John | Smith | 1990-01-01 | invalid-email | Pass123! | Error |
| TC08 | Duplicate email | John | Smith | 1990-01-01 | existing@test.com | Pass123! | Error |
| TC09 | Password too short | John | Smith | 1990-01-01 | new@test.com | Pass12 | Error |
| TC10 | Password no uppercase | John | Smith | 1990-01-01 | new@test.com | password123 | Error |
| TC11 | Password no number | John | Smith | 1990-01-01 | new@test.com | PasswordOnly | Error |
| TC12 | Under 18 years | John | Smith | [17 yrs ago] | new@test.com | Pass123! | Error |
| TC13 | Exactly 18 years | John | Smith | [18 yrs ago] | new@test.com | Pass123! | Success |
| TC14 | Over 18 years | John | Smith | 1980-01-01 | new@test.com | Pass123! | Success |
| TC15 | Postcode too short | John | Smith | 1990-01-01 | new@test.com | Pass123! | Error |
| TC16 | SQL Injection | ' OR 1=1-- | Smith | 1990-01-01 | test@test.com | Pass123! | Error/Sanitized |

**Boundary Value Test Cases:**

| TC ID | Field | Boundary | Value | Expected |
|-------|-------|----------|-------|----------|
| BV01 | First Name | Min-1 | 1 char | Error |
| BV02 | First Name | Min | 2 chars | Success |
| BV03 | First Name | Max | 50 chars | Success |
| BV04 | First Name | Max+1 | 51 chars | Error |
| BV05 | Password | Min-1 | 7 chars | Error |
| BV06 | Password | Min | 8 chars | Success |
| BV07 | Password | Max | 50 chars | Success |
| BV08 | Password | Max+1 | 51 chars | Error |
| BV09 | Age | Min-1 | 17 years | Error |
| BV10 | Age | Min | 18 years | Success |
| BV11 | Postcode | Min-1 | 3 chars | Error |
| BV12 | Postcode | Min | 4 chars | Success |
| BV13 | Postcode | Max | 10 chars | Success |
| BV14 | Postcode | Max+1 | 11 chars | Error |

---

### 2b) State Transition Testing - Order System (30%)

#### State Chart Diagram (10%)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     TOOLSHOP ORDER STATE DIAGRAM                            │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────┐
                              │    GUEST     │
                              │  (Browsing)  │
                              └──────┬───────┘
                                     │ Add to cart
                                     ▼
                              ┌──────────────┐
                              │    CART      │◄─────────────┐
                              │  (Session)   │              │
                              └──────┬───────┘              │
                                     │ Checkout             │
                                     ▼                      │
                        ┌────────────────────────┐          │
                        │    LOGIN/REGISTER      │          │
                        └────────────┬───────────┘          │
                                     │ Success              │
                                     ▼                      │
                              ┌──────────────┐              │
                              │   CHECKOUT   │              │
                              │ (Enter info) │              │
                              └──────┬───────┘              │
                                     │ Submit               │
                                     ▼                      │
                              ┌──────────────┐              │
                              │   PAYMENT    │              │
                              │  PROCESSING  │              │
                              └──────┬───────┘              │
                    ┌────────────────┴────────────────┐     │
                    │ Success              │ Failed   │     │
                    ▼                      ▼          │     │
            ┌───────────┐          ┌───────────┐     │     │
            │ CONFIRMED │          │  FAILED   │─────┴─────┘
            │           │          │ (Retry?)  │
            └─────┬─────┘          └───────────┘
                  │
                  ▼
            ┌───────────┐
            │PROCESSING │◄─────────┐
            │           │          │ Cancel (before ship)
            └─────┬─────┘          │
                  │                ▼
                  │          ┌───────────┐
                  │          │ CANCELLED │
                  │          └───────────┘
                  │ Ship
                  ▼
            ┌───────────┐
            │  SHIPPED  │
            │           │
            └─────┬─────┘
                  │ Deliver
                  ▼
            ┌───────────┐
            │ DELIVERED │
            │           │
            └─────┬─────┘
                  │ Return (≤14 days)
                  ▼
            ┌───────────┐
            │ RETURNED  │
            └───────────┘
```

**Member Account States:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MEMBER ACCOUNT STATE DIAGRAM                            │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────┐
                              │    START     │
                              └──────┬───────┘
                                     │ Register
                                     ▼
                              ┌──────────────┐
                              │    ACTIVE    │◄─────────────┐
                              │   MEMBER     │              │
                              └──────┬───────┘              │
                                     │                      │
                    ┌────────────────┴────────────────┐     │
                    │ 2 weeks before    │ Order       │     │
                    │ expiry            │             │     │
                    ▼                   ▼             │     │
            ┌───────────┐       ┌───────────────┐     │     │
            │ REMINDER  │       │ ORDER LIMIT   │     │     │
            │   SENT    │       │ (Standard:10) │     │     │
            └─────┬─────┘       └───────────────┘     │     │
                  │                                   │     │
      ┌───────────┴───────────┐                       │     │
      │ Renew      │ 30 days  │                       │     │
      │            │ no action│                       │     │
      ▼            ▼          │                       │     │
┌─────────┐  ┌───────────┐    │                       │     │
│ RENEWED │  │  EXPIRED  │    │                       │     │
│         │  │ (LOCKED)  │    │                       │     │
└────┬────┘  └─────┬─────┘    │                       │     │
     │             │          │                       │     │
     │    ┌────────┴────────┐ │                       │     │
     │    │ Renew  │ 1 year │ │                       │     │
     │    ▼        ▼        │ │                       │     │
     │  ┌─────┐ ┌─────────┐ │ │                       │     │
     │  │ACTIVE│ │ REMOVED │ │ │                       │     │
     │  └─────┘ └─────────┘ │ │                       │     │
     └──────────────────────┴─┴───────────────────────┘     │
                                                           │
                              Renew/Upgrade ───────────────┘
```

---

#### State Transition Tables (10%)

**Order State Transition Table:**

| Current State | Event/Condition | Action | Next State |
|---------------|-----------------|--------|------------|
| GUEST | Add product | Create session cart | CART |
| CART | Add more items | Update cart | CART |
| CART | Remove all items | Empty cart | GUEST |
| CART | Proceed checkout | Check auth | LOGIN |
| LOGIN | Login success | Show checkout | CHECKOUT |
| LOGIN | Register success | Create account → checkout | CHECKOUT |
| CHECKOUT | Submit valid info | Process payment | PAYMENT_PROCESSING |
| CHECKOUT | Invalid info | Show errors | CHECKOUT |
| PAYMENT_PROCESSING | Payment success | Create order | CONFIRMED |
| PAYMENT_PROCESSING | Payment failed | Show retry option | FAILED |
| FAILED | Retry | Re-process | PAYMENT_PROCESSING |
| FAILED | Cancel | Return to cart | CART |
| CONFIRMED | Process order | Prepare shipment | PROCESSING |
| PROCESSING | Ship order | Update tracking | SHIPPED |
| PROCESSING | Cancel request | Refund | CANCELLED |
| SHIPPED | Confirm delivery | Mark delivered | DELIVERED |
| SHIPPED | Cancel request | Reject (too late) | SHIPPED |
| DELIVERED | Return (≤14 days) | Process return | RETURNED |
| DELIVERED | Return (>14 days) | Reject | DELIVERED |

**Member Account Transition Table:**

| Current State | Event | Condition | Next State |
|---------------|-------|-----------|------------|
| START | Register | Valid info | ACTIVE |
| ACTIVE | 2 weeks before expiry | - | REMINDER_SENT |
| ACTIVE | Place order | Standard <10/month | ACTIVE |
| ACTIVE | Place order | Standard =10/month | ORDER_LIMIT |
| ACTIVE | Place order | Premium | ACTIVE |
| REMINDER_SENT | Renew | Pay fee | ACTIVE |
| REMINDER_SENT | 30 days no action | - | EXPIRED |
| EXPIRED | Renew | Pay fee | ACTIVE |
| EXPIRED | 1 year no action | - | REMOVED |
| ORDER_LIMIT | New month | - | ACTIVE |
| ORDER_LIMIT | Upgrade to Premium | Pay $50 | ACTIVE |

**Member Type Comparison:**

| Feature | Standard (Free) | Premium ($50/year) |
|---------|-----------------|-------------------|
| Orders/month | Max 10 | Unlimited |
| Shipping | Standard | Free |
| Discount | None | 10% all items |
| Processing | Normal | Priority |

---

#### Test Cases Design (10%)

**Order Flow Test Cases:**

| TC ID | Scenario | Precondition | Action | Expected |
|-------|----------|--------------|--------|----------|
| ST01 | Guest adds to cart | New session | Add product | Cart created |
| ST02 | Guest checkout | Cart has items | Click checkout | Redirect to login |
| ST03 | Login then checkout | At login page | Enter valid credentials | Go to checkout |
| ST04 | Valid payment | At checkout | Submit valid card | Order confirmed |
| ST05 | Payment failed | At checkout | Submit invalid card | Show retry option |
| ST06 | Retry payment | Payment failed | Fix card, retry | Order confirmed |
| ST07 | Cancel from failed | Payment failed | Click cancel | Return to cart |
| ST08 | Cancel before ship | Order confirmed | Request cancel | Cancelled, refunded |
| ST09 | Cancel after ship | Order shipped | Request cancel | Rejected |
| ST10 | Return within 14 days | Delivered <14 days | Request return | Return approved |
| ST11 | Return after 14 days | Delivered >14 days | Request return | Rejected |

**Member Account Test Cases:**

| TC ID | Scenario | Precondition | Action | Expected |
|-------|----------|--------------|--------|----------|
| ST12 | New registration | Guest | Register valid info | Account ACTIVE |
| ST13 | Standard order limit | Standard, 10 orders | Place 11th order | Rejected |
| ST14 | Premium unlimited | Premium | Place 20 orders | All successful |
| ST15 | Reminder email | 2 weeks before expiry | System check | Email sent |
| ST16 | Renew after reminder | Reminder sent | Pay renewal | Account extended |
| ST17 | Account expired | 30 days after expiry | Try to order | Account LOCKED |
| ST18 | Reactivate expired | Expired account | Pay renewal | Account ACTIVE |
| ST19 | Account removed | 1 year expired | Try login | Account not found |
| ST20 | Upgrade to Premium | Standard member | Pay $50 | Upgrade successful |

**State Coverage Matrix:**

| From State | To State | Covered by TC |
|------------|----------|---------------|
| GUEST → CART | ST01 |
| CART → LOGIN | ST02 |
| LOGIN → CHECKOUT | ST03 |
| CHECKOUT → PAYMENT | ST04 |
| PAYMENT → CONFIRMED | ST04, ST06 |
| PAYMENT → FAILED | ST05 |
| FAILED → PAYMENT | ST06 |
| FAILED → CART | ST07 |
| CONFIRMED → PROCESSING | ST08 |
| PROCESSING → CANCELLED | ST08 |
| PROCESSING → SHIPPED | ST09 |
| SHIPPED → DELIVERED | ST10 |
| DELIVERED → RETURNED | ST10 |
| ACTIVE → REMINDER | ST15 |
| REMINDER → ACTIVE | ST16 |
| REMINDER → EXPIRED | ST17 |
| EXPIRED → ACTIVE | ST18 |
| EXPIRED → REMOVED | ST19 |

---

## Summary

| Part | Topic | Points |
|------|-------|--------|
| 1a | Test environment solutions | 5% |
| 1b | Mobile/Web/Desktop testing | 5% |
| 1c | Performance/Load/Stress/Spike | 5% |
| 1d | GUI vs Usability testing | 5% |
| 1e | CI benefits | 5% |
| 1f | API testing features | 5% |
| 1g | DB testing for ToolShop | 5% |
| 1h | Test levels | 5% |
| 2a | Domain testing - Registration | 30% |
| 2b | State transition - Order System | 30% |
| **Total** | | **100%** |

---

## Quick Reference - Key Concepts

### Domain Testing Steps:
1. Identify Input/Output Variables
2. Identify Equivalence Classes (Valid/Invalid)
3. Select Test Cases (1 per class)
4. Boundary Value Analysis (min-1, min, min+1, max-1, max, max+1)

### State Transition Testing Steps:
1. Identify all states
2. Identify all events/transitions
3. Draw state diagram
4. Create transition table
5. Design test cases to cover all transitions

### Test Levels (V-Model):
- Unit → Integration → System → Acceptance

### Performance Testing Types:
- **Load:** Expected users
- **Stress:** Beyond capacity → find breaking point
- **Spike:** Sudden traffic surge
- **Endurance/Soak:** Long duration stability
