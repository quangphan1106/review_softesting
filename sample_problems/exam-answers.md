# Software Testing Final Examination - Answers
**Course:** CS423 - Software Testing
**Academic Year:** 2022-2023, Term I

---

## Part 1: Seminar (40%)

### 1a) Test Environment Solutions for OrangeHRM

**Testing Types:**
- Functional Testing: Verify HR features (recruitment, leave, time tracking)
- Integration Testing: Test module interactions (PIM ↔ Leave ↔ Time)
- Regression Testing: Ensure new changes don't break existing features
- Security Testing: Validate access controls, data protection
- Performance Testing: Check response times under load

**Testing Tools:**
| Category | Tools |
|----------|-------|
| Web Automation | Selenium WebDriver, Cypress, Playwright |
| API Testing | Postman, REST Assured, SoapUI |
| Performance | JMeter, Gatling, k6 |
| Mobile | Appium (if mobile version) |
| Security | OWASP ZAP, Burp Suite |

**Frameworks:**
- Selenium + TestNG/JUnit (Java)
- Cypress with Mocha/Chai (JavaScript)
- Robot Framework (Keyword-driven)
- Pytest + Selenium (Python)

---

### 1b) Differences: Mobile vs Web vs Desktop Automation Testing

| Aspect | Mobile Testing | Web Testing | Desktop Testing |
|--------|---------------|-------------|-----------------|
| **Tools** | Appium, Espresso, XCUITest | Selenium, Cypress, Playwright | WinAppDriver, TestComplete, UFT |
| **Platforms** | iOS, Android | Cross-browser | Windows, macOS, Linux |
| **Locators** | Accessibility ID, XPath, UIAutomator | CSS, XPath, ID, Name | AutomationId, Name, ClassName |
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
| **Example** | "Page loads in <2s" | "100 users simultaneously" | "What happens at 500 users?" |

**Why Performance Testing is Expensive:**
1. **Infrastructure Costs:** Requires production-like environments, load generators, monitoring tools
2. **Tool Licensing:** Enterprise tools (LoadRunner, NeoLoad) are costly
3. **Specialized Skills:** Requires expertise in scripting, analysis, bottleneck identification
4. **Time-Intensive:** Script development, test execution, analysis cycles
5. **Environment Setup:** Dedicated test environments to avoid production impact
6. **Data Requirements:** Large realistic datasets needed

---

### 1d) GUI Testing vs Usability Testing

| Aspect | GUI Testing | Usability Testing |
|--------|-------------|-------------------|
| **Focus** | Visual elements, functionality | User experience, ease of use |
| **Goal** | Verify UI works correctly | Verify users can accomplish tasks |
| **Method** | Automated/Manual scripted tests | User observation, feedback sessions |
| **Metrics** | Pass/Fail, defects found | Task completion rate, time, satisfaction |
| **Who Tests** | QA Engineers | Real end users |

**Applying GUI Testing to OrangeHRM:**
1. **Navigation Testing:** Verify all menu items, breadcrumbs work correctly
2. **Form Validation:** Test input fields, error messages, required fields
3. **Layout Testing:** Check element alignment, responsive design
4. **Cross-browser Testing:** Verify UI consistency across Chrome, Firefox, Safari, Edge
5. **Accessibility Testing:** Screen reader compatibility, color contrast, keyboard navigation
6. **Visual Regression:** Compare screenshots to detect unintended UI changes

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
2. **Response Validation:** Assert status codes, response body, headers, response time
3. **Environment Management:** Variables for different environments (dev, staging, prod)
4. **Test Automation:** Write test scripts, chain requests, data-driven testing
5. **Documentation:** Auto-generate API docs from collections/specs

---

### 1g) DB Testing for OrangeHRM

**Yes, DB Testing is necessary for OrangeHRM:**

**Reasons:**
1. **Data Integrity:** Employee records, leave balances, attendance must be accurate
2. **CRUD Operations:** Verify Create/Read/Update/Delete work correctly
3. **Constraints:** Check foreign keys, unique constraints, not-null validations
4. **Stored Procedures:** If used, must be tested for correctness
5. **Data Migration:** Verify data during upgrades/migrations
6. **Performance:** Query optimization, indexing effectiveness
7. **Security:** SQL injection prevention, access controls

**What to Test:**
- Employee data CRUD
- Leave balance calculations
- Time tracking records
- Report data accuracy
- Recruitment workflow data

---

### 1h) Test Levels for OrangeHRM

| Level | Description | Example in OrangeHRM |
|-------|-------------|---------------------|
| **Unit Testing** | Test individual functions/methods | Test leave calculation function |
| **Integration Testing** | Test module interactions | PIM module + Leave module integration |
| **System Testing** | Test complete system | Full recruitment workflow end-to-end |
| **Acceptance Testing** | Validate business requirements | HR manager validates leave approval flow |

**Additional Applicable Levels:**
- **Smoke Testing:** Quick sanity check after deployment
- **Regression Testing:** After each release/change
- **User Acceptance Testing (UAT):** HR department validates functionality

---

## Part 2: Exercises (60%)

### 2a) Domain Testing - Add Candidate Feature (30%)

#### Domain Analysis (15%)

**Input Fields Analysis:**

| Field | Data Type | Valid Domain | Invalid Domain | Constraints |
|-------|-----------|--------------|----------------|-------------|
| First Name | String | 3-30 chars, letters only | <3 or >30 chars, special chars, numbers | Required |
| Middle Name | String | 3-30 chars, letters only | <3 or >30 chars, special chars, numbers | Optional |
| Last Name | String | 3-30 chars, letters only | <3 or >30 chars, special chars, numbers | Required |
| Vacancy | Select | {IT Manager, QC Lead, Mobile Dev, Android Dev, BA Lead, DevOps} | Empty selection | Required |
| Email | String | Valid email format | Invalid format, empty | Required |
| Contact Number | String | Valid phone format | Invalid format | Optional |
| Resume | File | .docx, .doc, .odt, .pdf, .rtf, .txt; ≤1MB | Wrong format, >1MB | Required |
| Keywords | String | Comma-separated words | - | Optional |
| Date of Application | Date | Valid date | Invalid date format | Default: today |
| Notes | String | 0-255 chars | >255 chars | Optional |
| Consent | Checkbox | Checked/Unchecked | - | Optional |

**Equivalence Classes:**

**First Name / Middle Name / Last Name:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC1 | Valid | 3 chars (minimum) | "Amy" |
| EC2 | Valid | 15 chars (mid-range) | "Christopher" |
| EC3 | Valid | 30 chars (maximum) | "Alexandrinaelizabethmargareta" |
| EC4 | Invalid | 2 chars (below min) | "Jo" |
| EC5 | Invalid | 31 chars (above max) | "Alexandrinaelizabethmargaretar" |
| EC6 | Invalid | Contains numbers | "John123" |
| EC7 | Invalid | Contains special chars | "John@Doe" |
| EC8 | Invalid | Empty (for required) | "" |

**Email:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC9 | Valid | Standard email | "john.doe@company.com" |
| EC10 | Valid | Email with subdomain | "user@mail.company.co.uk" |
| EC11 | Invalid | Missing @ | "johncompany.com" |
| EC12 | Invalid | Missing domain | "john@" |
| EC13 | Invalid | Missing local part | "@company.com" |
| EC14 | Invalid | Multiple @ | "john@@company.com" |
| EC15 | Invalid | Empty | "" |

**Resume File:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC16 | Valid | .pdf file ≤1MB | "resume.pdf" (500KB) |
| EC17 | Valid | .docx file ≤1MB | "resume.docx" (800KB) |
| EC18 | Valid | .txt file ≤1MB | "resume.txt" (100KB) |
| EC19 | Valid | Exactly 1MB file | "resume.pdf" (1MB) |
| EC20 | Invalid | >1MB file | "resume.pdf" (2MB) |
| EC21 | Invalid | .exe file | "malware.exe" |
| EC22 | Invalid | .jpg file | "photo.jpg" |
| EC23 | Invalid | No file | - |

**Notes:**
| Class ID | Type | Description | Example |
|----------|------|-------------|---------|
| EC24 | Valid | Empty | "" |
| EC25 | Valid | 100 chars | "Some notes..." |
| EC26 | Valid | 255 chars (max) | "Maximum length note..." |
| EC27 | Invalid | 256 chars | "Exceeds maximum..." |

---

#### Test Cases Design (15%)

| TC ID | Test Case Name | First Name | Middle Name | Last Name | Vacancy | Email | Contact | Resume | Keywords | Date | Notes | Consent | Expected Result |
|-------|----------------|------------|-------------|-----------|---------|-------|---------|--------|----------|------|-------|---------|-----------------|
| TC01 | All valid minimum | Amy | - | Lee | IT Manager | amy@test.com | 1234567890 | resume.pdf (500KB) | Java | 2022-12-19 | Short note | Yes | Success |
| TC02 | All valid maximum | Alexandrinaelizabethmargareta | Alexandrinaelizabethmargareta | Alexandrinaelizabethmargareta | QC Lead | max@company.co.uk | +1-555-123-4567 | resume.docx (1MB) | Java,Python,SQL | 2022-12-19 | 255 char note... | No | Success |
| TC03 | First name too short | Jo | - | Smith | Mobile Dev | jo@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: First name min 3 chars |
| TC04 | First name too long | Alexandrinaelizabethmargaretar | - | Smith | Android Dev | test@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: First name max 30 chars |
| TC05 | First name with numbers | John123 | - | Smith | BA Lead | test@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: No numbers allowed |
| TC06 | First name with special chars | John@Doe | - | Smith | DevOps | test@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: No special chars |
| TC07 | Empty first name | - | - | Smith | IT Manager | test@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: First name required |
| TC08 | Invalid email - no @ | John | - | Smith | QC Lead | johntest.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: Invalid email |
| TC09 | Invalid email - empty | John | - | Smith | Mobile Dev | - | - | resume.pdf | - | 2022-12-19 | - | - | Error: Email required |
| TC10 | Resume too large | John | - | Smith | IT Manager | john@test.com | - | resume.pdf (2MB) | - | 2022-12-19 | - | - | Error: File exceeds 1MB |
| TC11 | Invalid resume format | John | - | Smith | IT Manager | john@test.com | - | photo.jpg | - | 2022-12-19 | - | - | Error: Invalid file format |
| TC12 | No resume uploaded | John | - | Smith | IT Manager | john@test.com | - | - | - | 2022-12-19 | - | - | Error: Resume required |
| TC13 | Notes too long | John | - | Smith | IT Manager | john@test.com | - | resume.pdf | - | 2022-12-19 | 256 chars... | - | Error: Notes max 255 chars |
| TC14 | No vacancy selected | John | - | Smith | - | john@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Error: Vacancy required |
| TC15 | All vacancies valid | John | - | Smith | IT Manager | john@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Success |
| TC16 | Valid with all optional empty | Amy | - | Lee | IT Manager | amy@test.com | - | resume.pdf | - | 2022-12-19 | - | - | Success |

**Boundary Value Test Cases:**

| TC ID | Field | Boundary | Value | Expected |
|-------|-------|----------|-------|----------|
| BV01 | First Name | Min-1 | 2 chars | Error |
| BV02 | First Name | Min | 3 chars | Success |
| BV03 | First Name | Min+1 | 4 chars | Success |
| BV04 | First Name | Max-1 | 29 chars | Success |
| BV05 | First Name | Max | 30 chars | Success |
| BV06 | First Name | Max+1 | 31 chars | Error |
| BV07 | Notes | Max-1 | 254 chars | Success |
| BV08 | Notes | Max | 255 chars | Success |
| BV09 | Notes | Max+1 | 256 chars | Error |
| BV10 | Resume Size | Max-1 | 999KB | Success |
| BV11 | Resume Size | Max | 1MB | Success |
| BV12 | Resume Size | Max+1 | 1MB+1KB | Error |

---

### 2b) State Transition Testing - DVD/CD Rental Service (30%)

#### State Chart Diagram for Membership (10%)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     DVD/CD RENTAL MEMBERSHIP STATE DIAGRAM                  │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌──────────────┐
                              │   START      │
                              └──────┬───────┘
                                     │ Register & Pay
                                     │ (30$ Normal / 100$ VIP)
                                     ▼
                              ┌──────────────┐
                              │    ACTIVE    │◄─────────────────────┐
                              │   MEMBER     │                      │
                              └──────┬───────┘                      │
                                     │                              │
           ┌─────────────────────────┼─────────────────────────┐    │
           │                         │                         │    │
           ▼                         ▼                         ▼    │
   ┌───────────────┐        ┌───────────────┐         ┌────────────┴──┐
   │   QUOTA       │        │  REMINDER     │         │   RENTING     │
   │   EXHAUSTED   │        │  SENT         │         │   DISK        │
   │ (Normal only) │        │ (2 wks before)│         │               │
   └───────┬───────┘        └───────┬───────┘         └───────────────┘
           │                        │                         │
           │                        │ 30 days                 │ Return disk
           │                        │ no renewal              │
           │                        ▼                         │
           │                ┌───────────────┐                 │
           │                │   EXPIRED     │                 │
           │                │   (LOCKED)    │                 │
           │                └───────┬───────┘                 │
           │                        │                         │
           │         ┌──────────────┴──────────────┐          │
           │         │                             │          │
           │         ▼                             ▼          │
           │  ┌─────────────┐              ┌─────────────┐    │
           │  │  REACTIVATE │              │   REMOVED   │    │
           │  │  (Pay fee)  │              │ (1yr no pay)│    │
           │  └──────┬──────┘              └─────────────┘    │
           │         │                                        │
           │         │ Pay annual fee                         │
           └─────────┴────────────────────────────────────────┘

LEGEND:
─────────────────────────────────────────────────────────────────────────
Normal Member: 30$/year, max 300 disks/year, unlimited at a time
VIP Member:    100$/year, unlimited disks/year, max 10 at a time
─────────────────────────────────────────────────────────────────────────
```

**States Description:**

| State | Description |
|-------|-------------|
| START | Initial state, potential customer |
| ACTIVE MEMBER | Can rent disks, membership valid |
| RENTING DISK | Currently has rented disks |
| REMINDER SENT | 2 weeks before expiration, reminder sent |
| QUOTA EXHAUSTED | Normal member reached 300 disks/year limit |
| EXPIRED (LOCKED) | 30 days after reminder, account locked |
| REACTIVATE | Pay fee to unlock account |
| REMOVED | 1 year after expiration, account deleted |

---

#### State Transition Tables (10%)

**Main Membership State Transition Table:**

| Current State | Event/Condition | Action | Next State |
|---------------|-----------------|--------|------------|
| START | Register + Pay $30 | Create Normal account | ACTIVE (Normal) |
| START | Register + Pay $100 | Create VIP account | ACTIVE (VIP) |
| ACTIVE | Rent disk (quota OK) | Ship disk, update count | RENTING |
| ACTIVE | 2 weeks before expiry | Send reminder email | REMINDER_SENT |
| ACTIVE | Quota reached (Normal) | Lock rental feature | QUOTA_EXHAUSTED |
| RENTING | Return disk | Update inventory | ACTIVE |
| RENTING | Return disk (via post office) | Update within 2 days | ACTIVE |
| REMINDER_SENT | Pay renewal fee | Extend 1 year | ACTIVE |
| REMINDER_SENT | 30 days elapsed, no payment | Lock account | EXPIRED |
| QUOTA_EXHAUSTED | Year resets / Renew | Reset quota | ACTIVE |
| EXPIRED | Pay renewal fee | Unlock account | ACTIVE |
| EXPIRED | 1 year elapsed, no payment | Delete account | REMOVED |
| REMOVED | - | Terminal state | - |

**Normal Member Rental Rules:**

| Current Disk Count | Yearly Count | Event | Action | Result |
|--------------------|--------------|-------|--------|--------|
| 0-∞ | <300 | Rent disk | Allow, increment yearly count | Success |
| 0-∞ | 300 | Rent disk | Deny rental | Quota Exhausted |
| >0 | Any | Return disk | Accept, update inventory | Disks returned |

**VIP Member Rental Rules:**

| Current On Loan | Yearly Count | Event | Action | Result |
|-----------------|--------------|-------|--------|--------|
| <10 | Unlimited | Rent disk | Allow rental | Success |
| 10 | Unlimited | Rent disk | Deny rental | Max 10 at a time |
| >0 | Any | Return disk | Accept, update inventory | Disks returned |

---

#### Test Cases Design (10%)

**Membership Lifecycle Test Cases:**

| TC ID | Test Scenario | Precondition | Action | Expected Result |
|-------|---------------|--------------|--------|-----------------|
| ST01 | Normal member registration | New customer | Register + Pay $30 | Normal account created, status: ACTIVE |
| ST02 | VIP member registration | New customer | Register + Pay $100 | VIP account created, status: ACTIVE |
| ST03 | Normal member rents disk | Active Normal member, <300 rented this year | Rent 1 disk | Disk shipped, yearly count +1 |
| ST04 | VIP member rents disk | Active VIP, <10 on loan | Rent 1 disk | Disk shipped |
| ST05 | Return disk via post office | Member has disk on loan | Send via post | Disk received within 2 days, inventory updated |
| ST06 | Return disk at agent | Member has disk on loan | Return at authorized agent | Disk received, inventory updated immediately |
| ST07 | Reminder email sent | Active member, 2 weeks before expiry | System check | Reminder email sent to member |
| ST08 | Renewal after reminder | Reminder sent, <30 days | Pay $30/$100 | Account renewed, status: ACTIVE |
| ST09 | Account locked after 30 days | Reminder sent, 30 days no action | Time elapsed | Account status: EXPIRED (locked) |
| ST10 | Reactivate expired account | Expired account | Pay annual fee | Account status: ACTIVE |
| ST11 | Account removed after 1 year | Expired 1 year, no renewal | Time elapsed | Account deleted from system |

**Normal Member Quota Test Cases:**

| TC ID | Test Scenario | Yearly Count | Action | Expected Result |
|-------|---------------|--------------|--------|-----------------|
| ST12 | Rent at count 299 | 299 | Rent disk | Success, count = 300 |
| ST13 | Rent at count 300 | 300 | Rent disk | Denied: quota exhausted |
| ST14 | Year resets quota | 300, new year | System reset | Count = 0, can rent again |
| ST15 | Rent unlimited at once | 50 | Rent 50 more disks | Success (100 on loan) |

**VIP Member Limit Test Cases:**

| TC ID | Test Scenario | On Loan Count | Action | Expected Result |
|-------|---------------|---------------|--------|-----------------|
| ST16 | Rent when 9 on loan | 9 | Rent 1 disk | Success, 10 on loan |
| ST17 | Rent when 10 on loan | 10 | Rent 1 disk | Denied: max 10 at a time |
| ST18 | Return then rent | 10, return 2 | Rent 1 disk | Success, 9 on loan |
| ST19 | Rent 500 in year | 200 already rented | Rent 1 disk | Success (unlimited yearly) |

**Edge Cases & Error Handling:**

| TC ID | Test Scenario | Condition | Expected Result |
|-------|---------------|-----------|-----------------|
| ST20 | Pay with credit card | Valid card | Payment accepted |
| ST21 | Pay with bank transfer | Valid transfer | Payment accepted |
| ST22 | Invalid payment | Invalid card/transfer | Payment rejected, account not created |
| ST23 | Rent while locked | Expired account | Rental denied |
| ST24 | Rent while quota locked | Normal, 300 rented | Rental denied |

**State Transition Coverage Matrix:**

| From State | To State | Covered by TC |
|------------|----------|---------------|
| START → ACTIVE (Normal) | ST01 |
| START → ACTIVE (VIP) | ST02 |
| ACTIVE → RENTING | ST03, ST04 |
| RENTING → ACTIVE | ST05, ST06 |
| ACTIVE → REMINDER_SENT | ST07 |
| REMINDER_SENT → ACTIVE | ST08 |
| REMINDER_SENT → EXPIRED | ST09 |
| EXPIRED → ACTIVE | ST10 |
| EXPIRED → REMOVED | ST11 |
| ACTIVE → QUOTA_EXHAUSTED | ST13 |
| QUOTA_EXHAUSTED → ACTIVE | ST14 |

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
| 1g | DB testing for OrangeHRM | ~5% |
| 1h | Test levels for OrangeHRM | ~5% |
| 2a | Domain testing - Add Candidate | 30% |
| 2b | State transition testing - DVD Rental | 30% |
| **Total** | | **100%** |
