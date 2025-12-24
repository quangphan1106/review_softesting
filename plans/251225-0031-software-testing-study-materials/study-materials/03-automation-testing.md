# Topic 3: Automation Testing (Web + Desktop) - Comprehensive Study Guide
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

---

## 3. Practical Test Case Design

### 3.1 Page Object Model Implementation

**Directory Structure:**
```
tests/
├── pages/
│   ├── __init__.py
│   ├── base_page.py
│   ├── login_page.py
│   ├── product_page.py
│   └── cart_page.py
├── tests/
│   ├── test_login.py
│   ├── test_checkout.py
│   └── conftest.py
├── utils/
│   ├── driver_factory.py
│   └── config.py
└── pytest.ini
```

**Base Page (Selenium/Python):**
```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class BasePage:
    def __init__(self, driver):
        self.driver = driver
        self.wait = WebDriverWait(driver, 10)

    def find(self, locator):
        return self.wait.until(EC.presence_of_element_located(locator))

    def click(self, locator):
        self.wait.until(EC.element_to_be_clickable(locator)).click()

    def type(self, locator, text):
        element = self.find(locator)
        element.clear()
        element.send_keys(text)

    def get_text(self, locator):
        return self.find(locator).text
```

**Login Page:**
```python
from selenium.webdriver.common.by import By
from pages.base_page import BasePage

class LoginPage(BasePage):
    # Locators
    EMAIL_INPUT = (By.ID, "email")
    PASSWORD_INPUT = (By.ID, "password")
    LOGIN_BUTTON = (By.CSS_SELECTOR, "[data-test='login-submit']")
    ERROR_MESSAGE = (By.CLASS_NAME, "alert-danger")

    def __init__(self, driver):
        super().__init__(driver)
        self.url = "https://practicesoftwaretesting.com/auth/login"

    def open(self):
        self.driver.get(self.url)
        return self

    def login(self, email, password):
        self.type(self.EMAIL_INPUT, email)
        self.type(self.PASSWORD_INPUT, password)
        self.click(self.LOGIN_BUTTON)
        return self

    def get_error_message(self):
        return self.get_text(self.ERROR_MESSAGE)
```

**Test Case:**
```python
import pytest
from pages.login_page import LoginPage

class TestLogin:
    def test_valid_login(self, driver):
        login_page = LoginPage(driver)
        login_page.open()
        login_page.login("customer@practicesoftwaretesting.com", "welcome01")

        assert "My Account" in driver.page_source

    def test_invalid_password(self, driver):
        login_page = LoginPage(driver)
        login_page.open()
        login_page.login("customer@practicesoftwaretesting.com", "wrong")

        assert "Invalid email or password" in login_page.get_error_message()
```

### 3.2 Playwright Implementation

```javascript
// pages/login.page.js
class LoginPage {
  constructor(page) {
    this.page = page;
    this.emailInput = page.locator('#email');
    this.passwordInput = page.locator('#password');
    this.loginButton = page.locator('[data-test="login-submit"]');
    this.errorMessage = page.locator('.alert-danger');
  }

  async goto() {
    await this.page.goto('/auth/login');
  }

  async login(email, password) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async getErrorMessage() {
    return await this.errorMessage.textContent();
  }
}

// tests/login.spec.js
const { test, expect } = require('@playwright/test');
const { LoginPage } = require('../pages/login.page');

test('valid login', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('customer@practicesoftwaretesting.com', 'welcome01');

  await expect(page).toHaveURL(/.*account/);
});

test('invalid password', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.goto();
  await loginPage.login('customer@practicesoftwaretesting.com', 'wrong');

  await expect(loginPage.errorMessage).toContainText('Invalid');
});
```

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

**Problem 1: Element Not Found (NoSuchElementException)**
```python
# Root Causes:
# 1. Element not loaded yet
# 2. Element in iframe
# 3. Wrong locator
# 4. Element dynamically generated

# Solution 1: Explicit wait
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

element = WebDriverWait(driver, 10).until(
    EC.presence_of_element_located((By.ID, "dynamic-element"))
)

# Solution 2: iframe handling
iframe = driver.find_element(By.TAG_NAME, "iframe")
driver.switch_to.frame(iframe)
# ... interact with elements
driver.switch_to.default_content()

# Solution 3: Verify locator in DevTools
# Right-click > Inspect > Copy > Copy selector/XPath
```

**Problem 2: Stale Element Reference**
```python
# Root Cause: Element reference invalid (page changed)

# Wrong approach
element = driver.find_element(By.ID, "btn")
# ... page updates
element.click()  # StaleElementReferenceException

# Correct approach: Re-locate before use
def click_safe(driver, locator):
    element = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable(locator)
    )
    element.click()
```

**Problem 3: Test Flakiness**
```python
# Causes:
# 1. Race conditions
# 2. Network latency
# 3. Animation delays
# 4. Shared test data

# Solutions:
# 1. Wait for network idle (Playwright)
await page.wait_for_load_state('networkidle')

# 2. Retry mechanism
@pytest.mark.flaky(reruns=2)
def test_flaky_feature():
    ...

# 3. Use unique test data
import uuid
email = f"test_{uuid.uuid4()}@example.com"
```

### 4.2 Debugging Techniques

**Selenium Debugging:**
```python
# Take screenshot on failure
def test_example(driver):
    try:
        # test code
    except Exception as e:
        driver.save_screenshot("failure.png")
        raise

# Print page source
print(driver.page_source)

# Execute JavaScript for debugging
driver.execute_script("console.log(document.body.innerHTML)")
```

**Playwright Debugging:**
```javascript
// Trace viewer
await page.context().tracing.start({ screenshots: true, snapshots: true });
// ... test code
await page.context().tracing.stop({ path: 'trace.zip' });

// Pause execution
await page.pause();  // Opens inspector

// Slow motion
const browser = await chromium.launch({ slowMo: 100 });
```

---

## 5. Toolshop Homework Connection

### 5.1 From Automation Testing Homework (HW06)

**Project Structure:**
```
automation_project/
├── pages/
│   ├── login_page.py
│   ├── product_page.py
│   ├── cart_page.py
│   └── checkout_page.py
├── tests/
│   ├── test_login.py
│   ├── test_product.py
│   └── test_checkout.py
├── conftest.py
├── requirements.txt
└── pytest.ini
```

**Multi-Browser Configuration:**
```python
# conftest.py
import pytest
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager

@pytest.fixture(params=["chrome", "firefox", "edge"])
def driver(request):
    if request.param == "chrome":
        driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))
    elif request.param == "firefox":
        from selenium.webdriver.firefox.service import Service as FFService
        from webdriver_manager.firefox import GeckoDriverManager
        driver = webdriver.Firefox(service=FFService(GeckoDriverManager().install()))
    elif request.param == "edge":
        from selenium.webdriver.edge.service import Service as EdgeService
        from webdriver_manager.microsoft import EdgeChromiumDriverManager
        driver = webdriver.Edge(service=EdgeService(EdgeChromiumDriverManager().install()))

    driver.maximize_window()
    yield driver
    driver.quit()
```

**Test Results from Homework:**
```
Total Test Cases: 50
Passed: 40 (80%)
Failed: 10 (20%)

Failures by Category:
- Element not found: 4
- Assertion failures: 3
- Timeout: 2
- Stale element: 1

Browser Distribution:
- Chrome: 48/50 passed (96%)
- Firefox: 42/50 passed (84%)
- Edge: 45/50 passed (90%)
```

### 5.2 Key Test Scenarios from Homework

**Scenario 1: Product Search**
```python
def test_search_product(driver):
    home_page = HomePage(driver)
    home_page.open()
    home_page.search("hammer")

    results = home_page.get_search_results()
    assert len(results) > 0
    assert "hammer" in results[0].text.lower()
```

**Scenario 2: Add to Cart**
```python
def test_add_to_cart(driver):
    product_page = ProductPage(driver)
    product_page.open_product(123)
    product_page.add_to_cart()

    cart_page = CartPage(driver)
    cart_page.open()

    assert cart_page.get_item_count() == 1
```

---

## 6. Application-Level Exam Questions

### Question 1: Choosing Automation Framework
**Scenario:** Team of 5 developers (Python experts) needs to automate testing for:
- Web application with dynamic content
- Supports Chrome, Firefox, Safari
- Integrates with Jenkins CI/CD
- Has iframes and Shadow DOM components

**Question:** Which framework would you recommend and why?

**Answer:**
```
Recommendation: Playwright (Python)

Justification:
1. Python support - matches team expertise
2. Cross-browser - Chromium, Firefox, WebKit (Safari engine)
3. CI/CD - excellent Jenkins integration
4. iframes - native frameLocator() support
5. Shadow DOM - native pierce capability
6. Auto-wait - reduces flakiness vs Selenium
7. Trace viewer - superior debugging

Alternative: Selenium (if legacy browser support needed)
- More protocol support
- Larger ecosystem
- But: more flaky, manual waits required
```

### Question 2: Handling Dynamic Content
**Scenario:** Page has a product list that loads via AJAX. Products appear 2-5 seconds after page load. Tests are failing intermittently.

**Question:** How would you fix this?

**Answer:**
```python
# WRONG: Hard sleep
time.sleep(5)  # Unreliable, slow

# WRONG: Implicit wait alone
driver.implicitly_wait(10)  # May not wait for AJAX

# CORRECT: Explicit wait for specific condition
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

def wait_for_products(driver):
    # Wait for at least one product to appear
    products = WebDriverWait(driver, 10).until(
        EC.presence_of_all_elements_located(
            (By.CSS_SELECTOR, ".product-card")
        )
    )
    return products

# CORRECT with Playwright (auto-wait)
products = page.locator('.product-card')
await expect(products.first).to_be_visible()
```

### Question 3: Multi-Browser Test Design
**Scenario:** Test needs to run on Chrome, Firefox, Edge. Some behaviors differ between browsers.

**Question:** How would you structure tests to handle browser-specific behavior?

**Answer:**
```python
import pytest

# Parameterized fixture for browsers
@pytest.fixture(params=["chrome", "firefox", "edge"])
def browser(request):
    return create_driver(request.param)

# Generic test (runs on all)
def test_login(browser):
    # Same test, all browsers
    pass

# Browser-specific test
@pytest.mark.skipif(
    "firefox" not in pytest.config.getoption("--browser"),
    reason="Firefox-specific test"
)
def test_firefox_specific_feature(browser):
    pass

# Handle browser differences in page objects
class LoginPage:
    def submit(self):
        if self.browser_name == "safari":
            # Safari-specific handling
            self.click_with_js(self.SUBMIT_BUTTON)
        else:
            self.click(self.SUBMIT_BUTTON)
```

---

## 7. Cheatsheet / Quick Reference

### Selenium Commands

```python
# Navigation
driver.get("https://example.com")
driver.back()
driver.forward()
driver.refresh()

# Finding elements
driver.find_element(By.ID, "id")
driver.find_element(By.CSS_SELECTOR, ".class")
driver.find_element(By.XPATH, "//div[@id='x']")
driver.find_elements(By.TAG_NAME, "a")  # Returns list

# Actions
element.click()
element.send_keys("text")
element.clear()
element.text
element.get_attribute("href")

# Waits
WebDriverWait(driver, 10).until(
    EC.element_to_be_clickable((By.ID, "btn"))
)

# JavaScript execution
driver.execute_script("return document.title")
driver.execute_script("arguments[0].click()", element)

# Screenshots
driver.save_screenshot("screenshot.png")

# iframes
driver.switch_to.frame("frame_name")
driver.switch_to.default_content()

# Alerts
alert = driver.switch_to.alert
alert.accept()
alert.dismiss()
alert.text
```

### Playwright Commands

```javascript
// Navigation
await page.goto('https://example.com');
await page.goBack();
await page.goForward();
await page.reload();

// Locators
page.locator('#id');
page.locator('.class');
page.locator('text=Submit');
page.locator('[data-test="btn"]');
page.getByRole('button', { name: 'Submit' });
page.getByTestId('submit');

// Actions
await locator.click();
await locator.fill('text');
await locator.clear();
await locator.textContent();
await locator.getAttribute('href');

// Waits (mostly automatic)
await page.waitForSelector('#element');
await page.waitForLoadState('networkidle');

// Assertions
await expect(locator).toBeVisible();
await expect(locator).toHaveText('Expected');
await expect(page).toHaveURL(/.*success/);

// Screenshots
await page.screenshot({ path: 'screenshot.png' });

// iframes
const frame = page.frameLocator('#iframe');
await frame.locator('#button').click();
```

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

1. **Framework Choice**: Selenium (legacy), Playwright (modern), Cypress (JS only)
2. **POM Pattern**: Separate page structure from test logic
3. **Waits**: Always use explicit waits, never hard sleeps
4. **Flakiness**: Auto-wait (Playwright/Cypress) > manual waits (Selenium)
5. **Cross-browser**: Test on multiple browsers, handle differences
6. **Locators**: ID > CSS > XPath (by reliability)
7. **Toolshop HW06**: 80% pass rate, 50 test cases, 3 browsers

---

*Study time recommended: 3-4 hours*
*Practice: Implement a POM framework for a sample web application*
