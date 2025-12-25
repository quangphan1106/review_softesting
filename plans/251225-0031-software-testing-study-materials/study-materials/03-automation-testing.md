# Topic 3: Automation Testing (Web + Desktop + Mobile) - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 Test Automation Pyramid

```
            ┌─────────────┐
            │     E2E     │  ← Slowest, most expensive
            │   (UI/GUI)  │     10% of tests
            ├─────────────┤
            │ Integration │  ← API, service tests
            │   (API)     │     20% of tests
            ├─────────────┤
            │    Unit     │  ← Fastest, cheapest
            │   (Code)    │     70% of tests
            └─────────────┘
```

**Why This Distribution?**
- Unit tests: Fast (ms), cheap, stable, high coverage
- Integration tests: Medium speed, catch interface issues
- E2E tests: Slow, brittle, but validate full user flows

### 1.2 Web Automation Architecture

**Selenium WebDriver Architecture:**
```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Test Code   │───>│  WebDriver   │───>│   Browser    │
│  (Python/    │    │  (Protocol)  │    │  (Chrome/    │
│   Java)      │    │              │    │   Firefox)   │
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                    JSON Wire Protocol
                    or W3C WebDriver
```

**Playwright Architecture:**
```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Test Code   │───>│  Playwright  │───>│   Browser    │
│  (JS/Python/ │    │   Server     │    │  (Chromium/  │
│   .NET)      │    │ (CDP/DevTools)│   │   Firefox/   │
└──────────────┘    └──────────────┘    │   WebKit)    │
                                        └──────────────┘
                    Direct protocol connection
                    (faster than Selenium)
```

### 1.3 Key Automation Concepts

**Locator Strategies (Priority Order):**
1. **ID** - Most reliable, unique
2. **Name** - Good for forms
3. **CSS Selector** - Flexible, fast
4. **XPath** - Powerful but slower
5. **Link Text** - For anchor elements
6. **Class Name** - May not be unique

**Wait Strategies:**
| Type | Use Case | Example |
|------|----------|---------|
| **Implicit** | Global timeout | `driver.implicitly_wait(10)` |
| **Explicit** | Specific condition | `WebDriverWait(driver, 10).until(...)` |
| **Fluent** | Polling interval | `Wait(driver).until(condition, poll=0.5)` |
| **Hard Sleep** | ❌ Avoid! | `time.sleep(5)` |

**Page Object Model (POM):**
- Encapsulate page elements and actions in classes
- Separate test logic from page structure
- Improves maintainability and reusability

---

## 2. Tools & Techniques Comparison

### 2.1 Selenium vs Playwright vs Cypress (2025)

| Feature | Selenium | Playwright | Cypress |
|---------|----------|-----------|---------|
| **Speed** | Slowest | Fastest | Fast |
| **Architecture** | WebDriver protocol | CDP direct | In-browser |
| **Auto-wait** | No (manual waits) | Yes | Yes |
| **Languages** | Java, Python, C#, Ruby, JS | JS, Python, .NET, Java | JS/TS only |
| **Browsers** | All (including IE) | Chromium, Firefox, WebKit | Chrome-family, Firefox |
| **Flakiness** | Higher (timing issues) | Low (auto-retry) | Low |
| **iframes** | Manual handling | Native support | Native support |
| **Shadow DOM** | JS execution needed | Native pierce | Native support |
| **Debugging** | Standard tools | Trace viewer | Time-travel |
| **Mobile** | Appium integration | Emulation only | Emulation only |
| **Parallel** | Selenium Grid | Built-in | Built-in |
| **Learning Curve** | Medium | Low-Medium | Low |

### 2.2 When to Use Each Tool

**Use Selenium when:**
- Legacy browser support needed (IE 11)
- Existing Selenium infrastructure
- Team expertise in non-JS languages (Java, C#)
- Mobile testing with Appium required

**Use Playwright when:**
- Modern web applications
- Cross-browser testing (Chromium, Firefox, WebKit)
- API + UI testing combined
- Need trace viewer for debugging
- Parallel execution is critical

**Use Cypress when:**
- JavaScript/TypeScript team
- Real-time test development with hot reload
- Component testing needed
- Single browser family sufficient

### 2.3 Desktop Automation Tools

| Tool | Platform | Language | Best For |
|------|----------|----------|----------|
| **Pywinauto** | Windows | Python | Win32/UIA apps |
| **AutoIt** | Windows | AutoIt | Legacy Windows |
| **WinAppDriver** | Windows | Any (Appium) | UWP/Win32 |
| **Sikuli** | Cross-platform | Python/Java | Image-based |
| **AutoGUI** | Cross-platform | Python | Simple scripting |

### 2.4 Mobile Automation Tools

| Tool | Platform | Language | Best For |
|------|----------|----------|----------|
| **Appium** | iOS + Android | Java, Python, JS, C# | Cross-platform mobile |
| **Espresso** | Android only | Java/Kotlin | Native Android (fast) |
| **XCUITest** | iOS only | Swift/Obj-C | Native iOS (fast) |
| **Detox** | iOS + Android | JavaScript | React Native apps |
| **Flutter Driver** | iOS + Android | Dart | Flutter apps |
| **Maestro** | iOS + Android | YAML | Simple E2E flows |

**Mobile Automation Architecture (Appium):**
```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Test Code   │───>│   Appium     │───>│  Device/     │
│  (Python/    │    │   Server     │    │  Emulator    │
│   Java)      │    │  (Node.js)   │    │  (iOS/Android)│
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                    WebDriver Protocol
                    + Mobile Extensions
```

**When to Use Each Mobile Tool:**

**Use Appium when:**
- Cross-platform testing (iOS + Android with same tests)
- Team expertise in Selenium/WebDriver
- Testing native, hybrid, or mobile web apps
- Need to test on real devices + emulators

**Use Espresso/XCUITest when:**
- Single platform (Android or iOS only)
- Need fastest execution speed
- Dev team writes tests (same language as app)
- CI integration with native build tools

**Use Detox when:**
- React Native applications
- JavaScript/TypeScript team
- Need gray-box testing capabilities

**Mobile Locator Strategies:**
| Strategy | Android | iOS |
|----------|---------|-----|
| **ID** | `resource-id` | `accessibilityIdentifier` |
| **Accessibility** | `content-desc` | `accessibilityLabel` |
| **Class** | `android.widget.Button` | `XCUIElementTypeButton` |
| **XPath** | Supported (slow) | Supported (slow) |
| **UIAutomator** | Android only | N/A |
| **Predicate** | N/A | iOS only (fast) |

**Mobile Test Concepts (Appium):**
- Setup: Create driver with `UiAutomator2Options` (platform, device, app path)
- Connect: `webdriver.Remote("http://localhost:4723", options)`
- Find elements: `driver.find_element("id", "com.app:id/email")`
- Actions: `send_keys()`, `click()`
- Assert: Check element text contains expected value

**Key Mobile Testing Challenges:**
1. **Device fragmentation** - Many screen sizes, OS versions
2. **Gestures** - Swipe, pinch, long press (not in web)
3. **App states** - Background, foreground, killed
4. **Permissions** - Camera, location, notifications
5. **Network conditions** - Offline, slow, switching
6. **Battery/performance** - Resource constraints

---

## 3. Practical Test Case Design

### 3.1 Page Object Model Implementation

**Directory Structure:**
- `pages/` - Base page + page objects (login_page, product_page, cart_page)
- `tests/` - Test files + conftest.py (fixtures)
- `utils/` - Driver factory, config

**POM Pattern Concepts:**

| Layer | Purpose | Example |
|-------|---------|---------|
| **BasePage** | Common methods (find, click, type, wait) | Explicit wait wrapper |
| **PageObject** | Page-specific locators + actions | `LoginPage.login(email, pwd)` |
| **TestCase** | Business logic assertions | `assert "My Account" in page` |

**Key Points:**
- Define locators as class constants: `EMAIL = (By.ID, "email")`
- Use explicit waits: `WebDriverWait(driver, 10).until(EC.element_to_be_clickable)`
- Methods return `self` for chaining: `page.open().login(email, pwd)`
- Separate page structure from test logic

### 3.2 Playwright POM Concepts

**PageObject:**
- Constructor: Store `page` + define locators using `page.locator()`
- Methods: `goto()`, `login(email, pwd)`, `getErrorMessage()`
- Actions: `fill()`, `click()`, `textContent()`

**Test:**
- Create page object: `const loginPage = new LoginPage(page)`
- Call methods: `loginPage.goto()`, `loginPage.login()`
- Assert with `expect()`: `await expect(page).toHaveURL(/.*account/)`

**Key Difference vs Selenium:**
- Auto-wait built-in (no explicit waits needed)
- Locators evaluated at action time (no stale element)

### 3.3 Test Cases for Automation

**TC-AUTO-001: Login Flow**
- **Precondition**: User registered
- **Steps**: Navigate to login, enter credentials, submit
- **Expected**: Redirect to account page
- **Locators**: ID for inputs, data-test for button

**TC-AUTO-002: Dynamic Element Wait**
- **Precondition**: Page with AJAX content
- **Steps**: Trigger action, wait for element
- **Expected**: Element appears within timeout
- **Technique**: Explicit wait with condition

**TC-AUTO-003: Cross-Browser Validation**
- **Precondition**: Same test on 3 browsers
- **Steps**: Run test on Chrome, Firefox, Safari
- **Expected**: Consistent behavior
- **Technique**: Parameterized fixtures

**TC-AUTO-004: iframe Interaction**
- **Precondition**: Page with embedded iframe
- **Steps**: Switch to iframe, interact, switch back
- **Expected**: Actions successful in both contexts
- **Locators**: Frame locator, then element locator

---

## 4. Troubleshooting Scenarios

### 4.1 Common Automation Failures

| Problem | Root Causes | Solutions |
|---------|-------------|-----------|
| **Element Not Found** | Not loaded, in iframe, wrong locator | Explicit wait, switch to frame, verify in DevTools |
| **Stale Element** | Page changed after finding | Re-locate before use with explicit wait |
| **Test Flakiness** | Race conditions, network, animations | Wait for networkidle, retry mechanism, unique test data |

**Key Fixes:**
- Explicit wait: `WebDriverWait(driver, 10).until(EC.element_to_be_clickable(locator))`
- iframe: `driver.switch_to.frame(iframe)` then `switch_to.default_content()`
- Unique data: `f"test_{uuid.uuid4()}@example.com"`

### 4.2 Debugging Techniques

| Tool | Technique | Purpose |
|------|-----------|---------|
| **Selenium** | `driver.save_screenshot("fail.png")` | Capture state on failure |
| **Selenium** | `print(driver.page_source)` | Inspect DOM |
| **Playwright** | `page.pause()` | Opens inspector mid-test |
| **Playwright** | Trace viewer (`tracing.start/stop`) | Record full session |
| **Playwright** | `slowMo: 100` | Slow down for visual debugging |

---

## 5. Toolshop Homework Connection

### 5.1 From Automation Testing Homework (HW06)

**Project Structure:** pages/ (POM classes) + tests/ + conftest.py (fixtures)

**Multi-Browser Concept:**
- Use `@pytest.fixture(params=["chrome", "firefox", "edge"])`
- webdriver-manager auto-downloads drivers
- Same test runs 3x (once per browser)

**Test Results:**

| Metric | Value |
|--------|-------|
| Total TCs | 50 |
| Pass Rate | 80% (40/50) |
| Chrome | 96% |
| Firefox | 84% |
| Edge | 90% |

**Failure Categories:** Element not found (4), Assertion (3), Timeout (2), Stale (1)

### 5.2 Key Test Scenarios

| Scenario | Steps | Assertion |
|----------|-------|-----------|
| Product Search | Open home → search "hammer" | Results > 0, contains "hammer" |
| Add to Cart | Open product → add to cart → open cart | Item count = 1 |

---

## 6. Application-Level Exam Questions

### Question 1: Choosing Automation Framework
**Scenario:** Python team, dynamic web app, Chrome/Firefox/Safari, iframes + Shadow DOM

**Answer:** Playwright (Python)
- Python support ✓
- Cross-browser (Chromium, Firefox, WebKit) ✓
- Native iframe + Shadow DOM support ✓
- Auto-wait reduces flakiness
- Alternative: Selenium (if legacy browser needed)

### Question 2: Handling Dynamic Content
**Scenario:** AJAX loads products 2-5s after page load, tests fail intermittently

**Answer:**
- ❌ Hard sleep: `time.sleep(5)` - unreliable, slow
- ❌ Implicit wait alone - may not wait for AJAX
- ✓ Explicit wait: `WebDriverWait(driver, 10).until(EC.presence_of_all_elements_located(locator))`
- ✓ Playwright: Auto-wait with `expect(locator).to_be_visible()`

### Question 3: Multi-Browser Test Design
**Scenario:** Tests need to run on Chrome, Firefox, Edge with some browser-specific behavior

**Answer:**
- Parameterized fixture: `@pytest.fixture(params=["chrome", "firefox", "edge"])`
- Generic tests: Run on all browsers automatically
- Browser-specific: Use `@pytest.mark.skipif` or conditional in page object
- Handle differences in page objects, not test cases

---

## 7. Cheatsheet / Quick Reference

### Selenium Commands

| Category | Command |
|----------|---------|
| **Navigate** | `driver.get(url)`, `back()`, `forward()`, `refresh()` |
| **Find** | `find_element(By.ID, "id")`, `By.CSS_SELECTOR`, `By.XPATH` |
| **Action** | `click()`, `send_keys()`, `clear()`, `.text`, `.get_attribute()` |
| **Wait** | `WebDriverWait(driver, 10).until(EC.element_to_be_clickable(loc))` |
| **JS** | `execute_script("return document.title")` |
| **Screenshot** | `save_screenshot("file.png")` |
| **iframe** | `switch_to.frame(frame)`, `switch_to.default_content()` |
| **Alert** | `switch_to.alert`, `accept()`, `dismiss()`, `.text` |

### Playwright Commands

| Category | Command |
|----------|---------|
| **Navigate** | `page.goto(url)`, `goBack()`, `goForward()`, `reload()` |
| **Locator** | `locator('#id')`, `getByRole()`, `getByTestId()` |
| **Action** | `click()`, `fill()`, `clear()`, `textContent()` |
| **Wait** | Auto-wait, or `waitForSelector()`, `waitForLoadState()` |
| **Assert** | `expect(loc).toBeVisible()`, `.toHaveText()`, `.toHaveURL()` |
| **Screenshot** | `page.screenshot({ path: 'file.png' })` |
| **iframe** | `frameLocator('#iframe').locator('#btn').click()` |

### Automation Best Practices

- [ ] Use Page Object Model
- [ ] Avoid hard sleeps - use explicit waits
- [ ] Keep locators in one place
- [ ] Use meaningful test names
- [ ] Run tests in parallel
- [ ] Take screenshots on failure
- [ ] Use unique test data
- [ ] Clean up after tests
- [ ] Don't share state between tests
- [ ] Integrate with CI/CD

---

## 8. Key Takeaways for Exam

1. **Framework Choice**:
   - Web: Selenium (legacy), Playwright (modern), Cypress (JS only)
   - Desktop: Pywinauto (Windows), WinAppDriver, Sikuli (image-based)
   - Mobile: Appium (cross-platform), Espresso (Android), XCUITest (iOS)
2. **POM Pattern**: Separate page structure from test logic
3. **Waits**: Always use explicit waits, never hard sleeps
4. **Flakiness**: Auto-wait (Playwright/Cypress) > manual waits (Selenium)
5. **Cross-browser**: Test on multiple browsers, handle differences
6. **Locators**: ID > CSS > XPath (by reliability)
7. **Mobile challenges**: Device fragmentation, gestures, app states, permissions
8. **Toolshop HW06**: 80% pass rate, 50 test cases, 3 browsers

---

*Study time recommended: 3-4 hours*
*Practice: Implement a POM framework for a sample web application*
