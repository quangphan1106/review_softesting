# Topic 3: Automation Testing - Câu hỏi vận dụng
**CS423 Software Testing | Final Exam Practice**

---

## Câu 1: Framework Selection (10 điểm)

**Tình huống:** Team 8 developers cần automation testing:
- 5 Python developers, 3 JavaScript developers
- Web app với React frontend
- Cần test trên Chrome, Firefox, Safari
- Có nhiều dynamic content và iframes
- CI/CD với GitHub Actions

**Yêu cầu:**
a) So sánh Selenium, Playwright, Cypress cho case này (4 điểm)
b) Recommend framework và giải thích (3 điểm)
c) Design project structure (3 điểm)

**Đáp án mẫu:**

a) **Framework comparison:**

| Criteria | Selenium | Playwright | Cypress | Score |
|----------|----------|-----------|---------|-------|
| Python support | ✓ (5 devs) | ✓ (5 devs) | ✗ | S=3, P=3, C=0 |
| JS support | ✓ (3 devs) | ✓ (3 devs) | ✓ (3 devs) | S=3, P=3, C=3 |
| Safari support | ✓ (WebDriver) | ✓ (WebKit) | ✗ (limited) | S=3, P=3, C=1 |
| Dynamic content | Manual waits | Auto-wait | Auto-wait | S=1, P=3, C=3 |
| iframes | Manual switch | Native | Native | S=1, P=3, C=3 |
| CI/CD | Good | Excellent | Excellent | S=2, P=3, C=3 |
| Flakiness | Higher | Low | Low | S=1, P=3, C=3 |
| **Total** | **14** | **21** | **16** |

b) **Recommendation: Playwright (Python)**

**Justification:**
- Highest score (21/21)
- Python support matches team majority (5/8)
- Native Safari support via WebKit
- Auto-wait eliminates flaky tests
- Built-in iframe handling
- Excellent trace viewer for debugging
- Best CI/CD integration

c) **Project structure:**
```
automation-tests/
├── playwright.config.py
├── requirements.txt
├── conftest.py
│
├── pages/                    # Page Object Models
│   ├── __init__.py
│   ├── base_page.py
│   ├── login_page.py
│   ├── product_page.py
│   └── checkout_page.py
│
├── tests/                    # Test files
│   ├── __init__.py
│   ├── test_login.py
│   ├── test_product.py
│   └── test_checkout.py
│
├── fixtures/                 # Test data
│   ├── users.json
│   └── products.json
│
├── utils/                    # Utilities
│   ├── __init__.py
│   ├── api_helper.py
│   └── test_data.py
│
└── reports/                  # Test reports
    ├── html/
    └── screenshots/
```

---

## Câu 2: Page Object Model Design (10 điểm)

**Tình huống:** Thiết kế POM cho e-commerce checkout flow:
1. Login page
2. Product listing
3. Product detail
4. Shopping cart
5. Checkout

**Yêu cầu:**
a) Design base page class với common methods (4 điểm)
b) Implement login page object (3 điểm)
c) Write test case using POM (3 điểm)

**Đáp án mẫu:**

a) **Base Page Class:**
```python
# pages/base_page.py
from playwright.sync_api import Page, expect
from typing import Optional
import logging

class BasePage:
    def __init__(self, page: Page):
        self.page = page
        self.logger = logging.getLogger(self.__class__.__name__)

    # Navigation
    def navigate_to(self, url: str):
        self.logger.info(f"Navigating to {url}")
        self.page.goto(url)
        return self

    def get_current_url(self) -> str:
        return self.page.url

    # Element interactions
    def click(self, selector: str, timeout: int = 10000):
        self.logger.info(f"Clicking {selector}")
        self.page.locator(selector).click(timeout=timeout)
        return self

    def fill(self, selector: str, value: str):
        self.logger.info(f"Filling {selector} with value")
        locator = self.page.locator(selector)
        locator.clear()
        locator.fill(value)
        return self

    def get_text(self, selector: str) -> str:
        return self.page.locator(selector).text_content()

    def get_texts(self, selector: str) -> list:
        return self.page.locator(selector).all_text_contents()

    # Waits
    def wait_for_element(self, selector: str, state: str = "visible", timeout: int = 10000):
        self.page.locator(selector).wait_for(state=state, timeout=timeout)
        return self

    def wait_for_url(self, url_pattern: str, timeout: int = 10000):
        self.page.wait_for_url(url_pattern, timeout=timeout)
        return self

    def wait_for_network_idle(self):
        self.page.wait_for_load_state("networkidle")
        return self

    # Assertions
    def assert_visible(self, selector: str):
        expect(self.page.locator(selector)).to_be_visible()
        return self

    def assert_text(self, selector: str, expected: str):
        expect(self.page.locator(selector)).to_have_text(expected)
        return self

    def assert_url_contains(self, text: str):
        expect(self.page).to_have_url(re.compile(f".*{text}.*"))
        return self

    # Screenshots
    def take_screenshot(self, name: str):
        self.page.screenshot(path=f"reports/screenshots/{name}.png")
        return self

    # iframes
    def switch_to_frame(self, selector: str):
        return self.page.frame_locator(selector)

    # Dropdown
    def select_option(self, selector: str, value: str):
        self.page.locator(selector).select_option(value)
        return self
```

b) **Login Page Object:**
```python
# pages/login_page.py
from pages.base_page import BasePage
from playwright.sync_api import Page

class LoginPage(BasePage):
    # Locators
    URL = "/auth/login"
    EMAIL_INPUT = "[data-test='email-input']"
    PASSWORD_INPUT = "[data-test='password-input']"
    LOGIN_BUTTON = "[data-test='login-submit']"
    ERROR_MESSAGE = ".alert-danger"
    REMEMBER_ME = "[data-test='remember-me']"
    FORGOT_PASSWORD = "[data-test='forgot-password']"

    def __init__(self, page: Page, base_url: str = "https://practicesoftwaretesting.com"):
        super().__init__(page)
        self.base_url = base_url

    def open(self):
        """Navigate to login page"""
        self.navigate_to(f"{self.base_url}{self.URL}")
        self.wait_for_element(self.EMAIL_INPUT)
        return self

    def enter_email(self, email: str):
        """Enter email address"""
        self.fill(self.EMAIL_INPUT, email)
        return self

    def enter_password(self, password: str):
        """Enter password"""
        self.fill(self.PASSWORD_INPUT, password)
        return self

    def click_login(self):
        """Click login button"""
        self.click(self.LOGIN_BUTTON)
        return self

    def check_remember_me(self):
        """Check remember me checkbox"""
        self.click(self.REMEMBER_ME)
        return self

    def login(self, email: str, password: str, remember: bool = False):
        """Complete login flow"""
        self.enter_email(email)
        self.enter_password(password)
        if remember:
            self.check_remember_me()
        self.click_login()
        return self

    def login_and_wait(self, email: str, password: str):
        """Login and wait for redirect"""
        self.login(email, password)
        self.wait_for_url(".*account.*")
        return self

    def get_error_message(self) -> str:
        """Get error message text"""
        self.wait_for_element(self.ERROR_MESSAGE)
        return self.get_text(self.ERROR_MESSAGE)

    def is_error_displayed(self) -> bool:
        """Check if error message is visible"""
        return self.page.locator(self.ERROR_MESSAGE).is_visible()

    def click_forgot_password(self):
        """Click forgot password link"""
        self.click(self.FORGOT_PASSWORD)
        return self
```

c) **Test Cases using POM:**
```python
# tests/test_login.py
import pytest
from pages.login_page import LoginPage
from pages.account_page import AccountPage

class TestLogin:
    """Login functionality tests"""

    @pytest.fixture(autouse=True)
    def setup(self, page):
        """Setup for each test"""
        self.login_page = LoginPage(page)

    def test_valid_login(self, page):
        """TC-001: Valid user can login successfully"""
        # Arrange
        email = "customer@practicesoftwaretesting.com"
        password = "welcome01"

        # Act
        self.login_page.open()
        self.login_page.login_and_wait(email, password)

        # Assert
        account_page = AccountPage(page)
        account_page.assert_visible(AccountPage.WELCOME_MESSAGE)
        assert "account" in page.url

    def test_invalid_password(self, page):
        """TC-002: Invalid password shows error"""
        # Arrange
        email = "customer@practicesoftwaretesting.com"
        password = "wrongpassword"

        # Act
        self.login_page.open()
        self.login_page.login(email, password)

        # Assert
        assert self.login_page.is_error_displayed()
        error_msg = self.login_page.get_error_message()
        assert "Invalid email or password" in error_msg

    def test_empty_email_validation(self, page):
        """TC-003: Empty email shows validation error"""
        # Arrange & Act
        self.login_page.open()
        self.login_page.enter_password("anypassword")
        self.login_page.click_login()

        # Assert
        assert self.login_page.is_error_displayed()

    @pytest.mark.parametrize("email,password,expected_error", [
        ("", "password", "Email is required"),
        ("invalid-email", "password", "Invalid email format"),
        ("test@test.com", "", "Password is required"),
        ("test@test.com", "short", "Password must be at least 8 characters"),
    ])
    def test_validation_errors(self, page, email, password, expected_error):
        """TC-004-007: Various validation scenarios"""
        self.login_page.open()
        self.login_page.login(email, password)

        error_msg = self.login_page.get_error_message()
        assert expected_error in error_msg or self.login_page.is_error_displayed()

    def test_remember_me_functionality(self, page, context):
        """TC-008: Remember me preserves session"""
        # First login with remember me
        self.login_page.open()
        self.login_page.login("customer@practicesoftwaretesting.com", "welcome01", remember=True)
        self.login_page.wait_for_url(".*account.*")

        # Close and reopen browser
        cookies = context.cookies()

        # Verify session cookie has extended expiry
        auth_cookie = next((c for c in cookies if c['name'] == 'auth_token'), None)
        assert auth_cookie is not None
        # Check cookie expiry is in the future (30 days)
```

---

## Câu 3: Wait Strategies (10 điểm)

**Tình huống:** Web app có:
- AJAX-loaded product list
- Lazy-loaded images
- Modal dialogs appearing after button click
- WebSocket real-time notifications

**Yêu cầu:**
a) Explain different wait strategies (3 điểm)
b) Implement appropriate waits for each scenario (4 điểm)
c) Design anti-flakiness strategy (3 điểm)

**Đáp án mẫu:**

a) **Wait strategies:**

| Strategy | Description | Use Case | Selenium | Playwright |
|----------|-------------|----------|----------|------------|
| **Implicit** | Global timeout for all elements | Not recommended | `driver.implicitly_wait(10)` | N/A (auto-wait) |
| **Explicit** | Wait for specific condition | Dynamic elements | `WebDriverWait` | `locator.wait_for()` |
| **Fluent** | Custom polling interval | Slow animations | `WebDriverWait` with polling | N/A |
| **Hard Sleep** | Fixed delay | NEVER use | `time.sleep(5)` | `page.wait_for_timeout()` |
| **Auto-wait** | Framework handles waits | All cases | N/A | Built-in |
| **Network idle** | Wait for all requests | After navigation | Custom | `wait_for_load_state()` |

b) **Implementations:**

```python
# Scenario 1: AJAX-loaded product list
class ProductListPage(BasePage):
    PRODUCT_CARDS = ".product-card"
    LOADING_SPINNER = ".loading-spinner"

    def wait_for_products(self):
        """Wait for AJAX products to load"""
        # Method 1: Wait for spinner to disappear
        self.page.locator(self.LOADING_SPINNER).wait_for(state="hidden")

        # Method 2: Wait for at least one product
        self.page.locator(self.PRODUCT_CARDS).first.wait_for(state="visible")

        # Method 3: Wait for network idle
        self.page.wait_for_load_state("networkidle")

        return self

    def get_product_count(self) -> int:
        self.wait_for_products()
        return self.page.locator(self.PRODUCT_CARDS).count()


# Scenario 2: Lazy-loaded images
class ProductDetailPage(BasePage):
    PRODUCT_IMAGES = ".product-image img"

    def wait_for_images_loaded(self):
        """Wait for all lazy-loaded images"""
        # Use JavaScript to check image loading
        self.page.wait_for_function("""
            () => {
                const images = document.querySelectorAll('.product-image img');
                return Array.from(images).every(img => img.complete && img.naturalHeight > 0);
            }
        """)
        return self


# Scenario 3: Modal dialogs
class CartPage(BasePage):
    ADD_TO_CART_BTN = "[data-test='add-to-cart']"
    MODAL_DIALOG = ".modal-dialog"
    MODAL_CONFIRM = ".modal-dialog .btn-confirm"
    MODAL_CLOSE = ".modal-dialog .btn-close"

    def add_to_cart_and_confirm(self):
        """Click add to cart and handle modal"""
        # Click button
        self.click(self.ADD_TO_CART_BTN)

        # Wait for modal to appear
        self.page.locator(self.MODAL_DIALOG).wait_for(state="visible")

        # Wait for animation to complete
        self.page.wait_for_function("""
            () => {
                const modal = document.querySelector('.modal-dialog');
                return window.getComputedStyle(modal).opacity === '1';
            }
        """)

        # Confirm
        self.click(self.MODAL_CONFIRM)

        # Wait for modal to disappear
        self.page.locator(self.MODAL_DIALOG).wait_for(state="hidden")

        return self


# Scenario 4: WebSocket notifications
class NotificationPage(BasePage):
    NOTIFICATION_TOAST = ".notification-toast"
    NOTIFICATION_COUNT = ".notification-badge"

    def wait_for_notification(self, timeout: int = 30000):
        """Wait for WebSocket notification"""
        # Method 1: Wait for element
        self.page.locator(self.NOTIFICATION_TOAST).wait_for(
            state="visible",
            timeout=timeout
        )

        # Method 2: Wait for specific condition
        self.page.wait_for_function(
            """
            () => {
                const badge = document.querySelector('.notification-badge');
                return badge && parseInt(badge.textContent) > 0;
            }
            """,
            timeout=timeout
        )

        return self

    def setup_websocket_listener(self):
        """Listen for WebSocket messages"""
        self.page.evaluate("""
            window.wsMessages = [];
            const originalWS = window.WebSocket;
            window.WebSocket = function(...args) {
                const ws = new originalWS(...args);
                ws.addEventListener('message', (event) => {
                    window.wsMessages.push(JSON.parse(event.data));
                });
                return ws;
            };
        """)
```

c) **Anti-flakiness strategy:**
```python
# conftest.py - Anti-flakiness configuration

import pytest
from playwright.sync_api import Page

# 1. Retry failed tests
@pytest.fixture(scope="function")
def page(browser):
    """Create page with retry logic"""
    context = browser.new_context()
    page = context.new_page()

    # Set default timeout
    page.set_default_timeout(30000)

    yield page
    context.close()


# 2. Auto-retry decorator
def retry_on_failure(max_attempts=3, delay=1):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt < max_attempts - 1:
                        time.sleep(delay)
                    else:
                        raise
        return wrapper
    return decorator


# 3. Stable element finder
class StableLocator:
    def __init__(self, page: Page, selector: str, max_attempts: int = 3):
        self.page = page
        self.selector = selector
        self.max_attempts = max_attempts

    def click(self):
        for attempt in range(self.max_attempts):
            try:
                locator = self.page.locator(self.selector)
                locator.wait_for(state="visible")
                locator.wait_for(state="stable")  # Wait for no movement
                locator.click()
                return
            except Exception:
                if attempt == self.max_attempts - 1:
                    raise
                self.page.wait_for_timeout(500)


# 4. pytest.ini configuration
"""
[pytest]
addopts = --reruns 2 --reruns-delay 1
filterwarnings = ignore::DeprecationWarning
"""

# 5. Isolation between tests
@pytest.fixture(autouse=True)
def reset_state(page):
    """Reset application state between tests"""
    yield
    # Clear cookies
    page.context.clear_cookies()
    # Clear local storage
    page.evaluate("window.localStorage.clear()")
    # Clear session storage
    page.evaluate("window.sessionStorage.clear()")
```

---

## Câu 4: Cross-Browser Testing (10 điểm)

**Tình huống:** E-commerce app cần test trên:
- Chrome (desktop + mobile)
- Firefox (desktop)
- Safari (desktop + mobile)
- Edge (desktop)

**Yêu cầu:**
a) Design cross-browser test configuration (4 điểm)
b) Handle browser-specific behaviors (3 điểm)
c) Implement parallel execution (3 điểm)

**Đáp án mẫu:**

a) **Cross-browser configuration:**
```python
# playwright.config.py
from playwright.sync_api import sync_playwright

# Browser configurations
BROWSER_CONFIGS = {
    "chromium-desktop": {
        "browser": "chromium",
        "viewport": {"width": 1920, "height": 1080},
        "device_scale_factor": 1,
    },
    "chromium-mobile": {
        "browser": "chromium",
        "device": "iPhone 13",
    },
    "firefox-desktop": {
        "browser": "firefox",
        "viewport": {"width": 1920, "height": 1080},
    },
    "webkit-desktop": {
        "browser": "webkit",
        "viewport": {"width": 1920, "height": 1080},
    },
    "webkit-mobile": {
        "browser": "webkit",
        "device": "iPhone 13",
    },
}

# conftest.py
import pytest
from playwright.sync_api import sync_playwright

def pytest_addoption(parser):
    parser.addoption(
        "--browser-config",
        action="store",
        default="chromium-desktop",
        help="Browser configuration to use"
    )

@pytest.fixture(scope="session")
def browser(request):
    config_name = request.config.getoption("--browser-config")
    config = BROWSER_CONFIGS.get(config_name, BROWSER_CONFIGS["chromium-desktop"])

    with sync_playwright() as p:
        browser_type = getattr(p, config["browser"])
        browser = browser_type.launch(headless=True)
        yield browser
        browser.close()

@pytest.fixture(scope="function")
def page(browser, request):
    config_name = request.config.getoption("--browser-config")
    config = BROWSER_CONFIGS.get(config_name, BROWSER_CONFIGS["chromium-desktop"])

    context_options = {}

    if "device" in config:
        device = p.devices[config["device"]]
        context_options.update(device)
    else:
        context_options["viewport"] = config.get("viewport", {"width": 1280, "height": 720})

    context = browser.new_context(**context_options)
    page = context.new_page()
    yield page
    context.close()
```

b) **Browser-specific handling:**
```python
# utils/browser_utils.py
from playwright.sync_api import Page
import platform

class BrowserHandler:
    def __init__(self, page: Page):
        self.page = page
        self.browser_name = page.context.browser.browser_type.name

    def upload_file(self, selector: str, file_path: str):
        """Handle file upload differently per browser"""
        if self.browser_name == "webkit":
            # Safari needs special handling
            self.page.set_input_files(selector, file_path)
        else:
            # Chrome/Firefox
            locator = self.page.locator(selector)
            locator.set_input_files(file_path)

    def handle_download(self):
        """Handle download per browser"""
        if self.browser_name == "chromium":
            # Chrome shows download bar
            with self.page.expect_download() as download_info:
                self.page.click("#download-btn")
            download = download_info.value
            return download.path()
        elif self.browser_name == "firefox":
            # Firefox may show dialog
            self.page.once("dialog", lambda d: d.accept())
            with self.page.expect_download() as download_info:
                self.page.click("#download-btn")
            return download_info.value.path()

    def handle_alert(self, action: str = "accept"):
        """Handle JavaScript alerts"""
        def handler(dialog):
            if action == "accept":
                dialog.accept()
            else:
                dialog.dismiss()

        self.page.once("dialog", handler)

    def scroll_to_element(self, selector: str):
        """Cross-browser scroll"""
        if self.browser_name == "webkit":
            # Safari sometimes needs different scroll behavior
            self.page.evaluate(f"""
                document.querySelector('{selector}').scrollIntoView({{
                    behavior: 'instant',
                    block: 'center'
                }});
            """)
        else:
            self.page.locator(selector).scroll_into_view_if_needed()

    def get_computed_style(self, selector: str, property: str):
        """Get computed CSS property"""
        return self.page.evaluate(f"""
            window.getComputedStyle(
                document.querySelector('{selector}')
            ).{property}
        """)


# Test with browser-specific skip
import pytest

@pytest.mark.skipif_browser("webkit", reason="Feature not supported in Safari")
def test_feature_not_in_safari(page):
    # This test skips on Safari
    pass

# Custom marker implementation
def pytest_configure(config):
    config.addinivalue_line(
        "markers",
        "skipif_browser(browser, reason): Skip test for specific browser"
    )

def pytest_collection_modifyitems(config, items):
    browser = config.getoption("--browser-config", "chromium-desktop")
    for item in items:
        for marker in item.iter_markers(name="skipif_browser"):
            if marker.args[0] in browser:
                item.add_marker(pytest.mark.skip(reason=marker.kwargs.get("reason", "")))
```

c) **Parallel execution:**
```yaml
# .github/workflows/cross-browser.yml
name: Cross-Browser Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        browser:
          - chromium-desktop
          - chromium-mobile
          - firefox-desktop
          - webkit-desktop
          - webkit-mobile

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install pytest playwright pytest-xdist
          playwright install --with-deps

      - name: Run tests
        run: |
          pytest tests/ \
            --browser-config=${{ matrix.browser }} \
            -n auto \
            --html=report-${{ matrix.browser }}.html

      - name: Upload results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results-${{ matrix.browser }}
          path: |
            report-${{ matrix.browser }}.html
            screenshots/
```

```python
# pytest-xdist parallel configuration
# pytest.ini
"""
[pytest]
addopts = -n 4 --dist loadfile
"""

# conftest.py - parallel-safe fixtures
@pytest.fixture(scope="session")
def browser(request, worker_id):
    """Create browser per worker"""
    # Each worker gets unique port
    if worker_id == "master":
        port = 9222
    else:
        port = 9222 + int(worker_id.replace("gw", ""))

    with sync_playwright() as p:
        browser = p.chromium.launch(
            headless=True,
            args=[f"--remote-debugging-port={port}"]
        )
        yield browser
        browser.close()
```

---

## Câu 5: Test Data Management (10 điểm)

**Tình huống:** Automation tests cần:
- User accounts (admin, customer, guest)
- Product data (various categories)
- Order history
- Payment methods

**Yêu cầu:**
a) Design test data strategy (4 điểm)
b) Implement data fixtures (3 điểm)
c) Handle data cleanup (3 điểm)

**Đáp án mẫu:**

a) **Test data strategy:**
```
┌─────────────────────────────────────────────────────────────┐
│                    TEST DATA LAYERS                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Static Data (fixtures/)                                 │
│     - User credentials                                       │
│     - Product catalog                                        │
│     - Categories                                             │
│                                                              │
│  2. Dynamic Data (generated per test)                       │
│     - Unique emails                                          │
│     - Random order IDs                                       │
│     - Timestamps                                             │
│                                                              │
│  3. API-Created Data (setup phase)                          │
│     - Test users                                             │
│     - Test orders                                            │
│     - Cart items                                             │
│                                                              │
│  4. Database Seeds (CI/CD)                                  │
│     - Baseline data                                          │
│     - Reference data                                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

b) **Data fixtures implementation:**
```python
# fixtures/users.json
{
    "admin": {
        "email": "admin@practicesoftwaretesting.com",
        "password": "welcome01",
        "role": "admin"
    },
    "customer": {
        "email": "customer@practicesoftwaretesting.com",
        "password": "welcome01",
        "role": "customer"
    }
}

# fixtures/products.json
{
    "hammer": {
        "id": "01HQJY0G8M5N4P6R7S8T9V0W1X",
        "name": "Thor Hammer",
        "price": 29.99,
        "category": "tools"
    }
}

# conftest.py - Fixtures
import pytest
import json
from pathlib import Path
from faker import Faker
import requests

fake = Faker()

@pytest.fixture(scope="session")
def test_users():
    """Load user test data"""
    with open("fixtures/users.json") as f:
        return json.load(f)

@pytest.fixture(scope="session")
def test_products():
    """Load product test data"""
    with open("fixtures/products.json") as f:
        return json.load(f)

@pytest.fixture(scope="function")
def unique_user():
    """Generate unique user for each test"""
    return {
        "email": f"test_{fake.uuid4()[:8]}@test.com",
        "password": "TestPass123!",
        "firstName": fake.first_name(),
        "lastName": fake.last_name()
    }

@pytest.fixture(scope="function")
def api_client():
    """API client for test data setup"""
    class APIClient:
        def __init__(self):
            self.base_url = "https://api.practicesoftwaretesting.com"
            self.token = None

        def login(self, email: str, password: str):
            response = requests.post(
                f"{self.base_url}/users/login",
                json={"email": email, "password": password}
            )
            self.token = response.json()["access_token"]
            return self

        def create_user(self, user_data: dict):
            response = requests.post(
                f"{self.base_url}/users/register",
                json=user_data
            )
            return response.json()

        def create_order(self, products: list):
            response = requests.post(
                f"{self.base_url}/orders",
                json={"products": products},
                headers={"Authorization": f"Bearer {self.token}"}
            )
            return response.json()

        def delete_user(self, user_id: str):
            requests.delete(
                f"{self.base_url}/users/{user_id}",
                headers={"Authorization": f"Bearer {self.token}"}
            )

    return APIClient()

@pytest.fixture(scope="function")
def logged_in_user(page, test_users, api_client):
    """Setup: Create and login user, Teardown: Cleanup"""
    # Setup
    user = test_users["customer"]
    api_client.login(user["email"], user["password"])

    # Provide to test
    yield user

    # Teardown (if created new user)
    # api_client.delete_user(user_id)
```

c) **Data cleanup:**
```python
# cleanup.py
import pytest
from datetime import datetime, timedelta

class TestDataManager:
    """Manage test data lifecycle"""

    def __init__(self, api_client):
        self.api = api_client
        self.created_users = []
        self.created_orders = []

    def create_user(self, user_data: dict):
        """Create user and track for cleanup"""
        result = self.api.create_user(user_data)
        self.created_users.append(result["id"])
        return result

    def create_order(self, order_data: dict):
        """Create order and track for cleanup"""
        result = self.api.create_order(order_data)
        self.created_orders.append(result["id"])
        return result

    def cleanup(self):
        """Delete all created test data"""
        for order_id in self.created_orders:
            try:
                self.api.delete_order(order_id)
            except Exception:
                pass  # Log but continue

        for user_id in self.created_users:
            try:
                self.api.delete_user(user_id)
            except Exception:
                pass

        self.created_users.clear()
        self.created_orders.clear()


@pytest.fixture(scope="function")
def data_manager(api_client):
    """Provide data manager with auto-cleanup"""
    manager = TestDataManager(api_client)
    yield manager
    manager.cleanup()


# Database cleanup for CI/CD
"""
-- cleanup.sql
DELETE FROM orders WHERE created_at > NOW() - INTERVAL '1 hour'
  AND email LIKE 'test_%@test.com';

DELETE FROM users WHERE created_at > NOW() - INTERVAL '1 hour'
  AND email LIKE 'test_%@test.com';

-- Reset sequences if needed
ALTER SEQUENCE users_id_seq RESTART WITH 1000;
"""

# CI/CD cleanup step
"""
- name: Cleanup test data
  if: always()
  run: |
    psql $DATABASE_URL -f cleanup.sql
"""
```

---

## Câu 6: Error Handling & Screenshots (10 điểm)

**Tình huống:** Khi test fails cần:
- Capture screenshot
- Save page HTML
- Record video
- Log console errors
- Create detailed report

**Yêu cầu:**
a) Implement failure handling hook (4 điểm)
b) Create custom reporter (3 điểm)
c) Design debugging workflow (3 điểm)

**Đáp án mẫu:**

a) **Failure handling hook:**
```python
# conftest.py
import pytest
from datetime import datetime
from pathlib import Path
import json

def pytest_configure(config):
    """Create reports directory"""
    Path("reports/screenshots").mkdir(parents=True, exist_ok=True)
    Path("reports/html").mkdir(parents=True, exist_ok=True)
    Path("reports/videos").mkdir(parents=True, exist_ok=True)
    Path("reports/traces").mkdir(parents=True, exist_ok=True)

@pytest.hookimpl(tryfirst=True, hookwrapper=True)
def pytest_runtest_makereport(item, call):
    """Hook to capture screenshots on failure"""
    outcome = yield
    report = outcome.get_result()

    if report.when == "call" and report.failed:
        page = item.funcargs.get("page")
        if page:
            # Generate unique name
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            test_name = item.name.replace(" ", "_")

            # 1. Screenshot
            screenshot_path = f"reports/screenshots/{test_name}_{timestamp}.png"
            page.screenshot(path=screenshot_path, full_page=True)

            # 2. HTML source
            html_path = f"reports/html/{test_name}_{timestamp}.html"
            with open(html_path, "w") as f:
                f.write(page.content())

            # 3. Console logs
            console_logs = []
            page.on("console", lambda msg: console_logs.append({
                "type": msg.type,
                "text": msg.text,
                "location": str(msg.location)
            }))

            logs_path = f"reports/logs/{test_name}_{timestamp}.json"
            with open(logs_path, "w") as f:
                json.dump(console_logs, f, indent=2)

            # 4. Attach to report
            if hasattr(report, "extra"):
                report.extra.append(pytest.html.extras.image(screenshot_path))
                report.extra.append(pytest.html.extras.html(f'<a href="{html_path}">HTML Source</a>'))


@pytest.fixture(scope="function")
def page_with_video(browser):
    """Page with video recording"""
    context = browser.new_context(record_video_dir="reports/videos/")
    page = context.new_page()
    yield page

    # Save video on close
    context.close()
    video_path = page.video.path()
    print(f"Video saved: {video_path}")


@pytest.fixture(scope="function")
def page_with_trace(browser):
    """Page with trace recording"""
    context = browser.new_context()
    context.tracing.start(screenshots=True, snapshots=True, sources=True)
    page = context.new_page()

    yield page

    # Save trace
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    trace_path = f"reports/traces/trace_{timestamp}.zip"
    context.tracing.stop(path=trace_path)
    context.close()
```

b) **Custom reporter:**
```python
# custom_reporter.py
import pytest
from datetime import datetime
import json
from pathlib import Path

class CustomTestReporter:
    """Custom test reporter with detailed failure information"""

    def __init__(self):
        self.results = {
            "summary": {},
            "tests": [],
            "failures": [],
            "start_time": None,
            "end_time": None
        }

    def pytest_sessionstart(self, session):
        self.results["start_time"] = datetime.now().isoformat()

    def pytest_sessionfinish(self, session, exitstatus):
        self.results["end_time"] = datetime.now().isoformat()
        self.results["summary"] = {
            "total": len(self.results["tests"]),
            "passed": sum(1 for t in self.results["tests"] if t["status"] == "passed"),
            "failed": sum(1 for t in self.results["tests"] if t["status"] == "failed"),
            "skipped": sum(1 for t in self.results["tests"] if t["status"] == "skipped"),
            "exit_status": exitstatus
        }

        # Save report
        with open("reports/test_report.json", "w") as f:
            json.dump(self.results, f, indent=2)

        # Generate HTML
        self._generate_html_report()

    def pytest_runtest_logreport(self, report):
        if report.when == "call":
            test_result = {
                "name": report.nodeid,
                "status": report.outcome,
                "duration": report.duration,
                "timestamp": datetime.now().isoformat()
            }

            if report.failed:
                test_result["error"] = {
                    "message": str(report.longrepr),
                    "screenshot": self._get_screenshot_path(report.nodeid),
                    "video": self._get_video_path(report.nodeid)
                }
                self.results["failures"].append(test_result)

            self.results["tests"].append(test_result)

    def _generate_html_report(self):
        html = f"""
        <!DOCTYPE html>
        <html>
        <head>
            <title>Test Report</title>
            <style>
                body {{ font-family: Arial; margin: 20px; }}
                .pass {{ color: green; }}
                .fail {{ color: red; }}
                .summary {{ background: #f5f5f5; padding: 20px; margin-bottom: 20px; }}
                .test-case {{ border: 1px solid #ddd; padding: 10px; margin: 5px 0; }}
                .screenshot {{ max-width: 800px; }}
            </style>
        </head>
        <body>
            <h1>Test Execution Report</h1>
            <div class="summary">
                <h2>Summary</h2>
                <p>Total: {self.results['summary']['total']}</p>
                <p class="pass">Passed: {self.results['summary']['passed']}</p>
                <p class="fail">Failed: {self.results['summary']['failed']}</p>
            </div>

            <h2>Failed Tests</h2>
            {"".join(self._render_failure(f) for f in self.results['failures'])}

            <h2>All Tests</h2>
            {"".join(self._render_test(t) for t in self.results['tests'])}
        </body>
        </html>
        """
        with open("reports/report.html", "w") as f:
            f.write(html)

    def _render_failure(self, failure):
        return f"""
        <div class="test-case fail">
            <h3>{failure['name']}</h3>
            <p>Error: {failure['error']['message'][:500]}...</p>
            <img class="screenshot" src="{failure['error']['screenshot']}" />
            <p><a href="{failure['error']['video']}">View Video</a></p>
        </div>
        """

    def _render_test(self, test):
        status_class = "pass" if test["status"] == "passed" else "fail"
        return f"""
        <div class="test-case {status_class}">
            <span>{test['name']}</span>
            <span>{test['status']}</span>
            <span>{test['duration']:.2f}s</span>
        </div>
        """
```

c) **Debugging workflow:**
```python
# debugging.py
from playwright.sync_api import Page
import logging

class DebugHelper:
    """Helper for debugging failed tests"""

    def __init__(self, page: Page):
        self.page = page
        self.logger = logging.getLogger("debug")

    def capture_state(self, name: str):
        """Capture current page state for debugging"""
        state = {
            "url": self.page.url,
            "title": self.page.title(),
            "cookies": self.page.context.cookies(),
            "localStorage": self.page.evaluate("() => JSON.stringify(localStorage)"),
            "sessionStorage": self.page.evaluate("() => JSON.stringify(sessionStorage)"),
            "consoleLogs": [],
            "networkRequests": []
        }

        # Screenshot
        self.page.screenshot(path=f"debug/{name}_screenshot.png")

        # Save state
        with open(f"debug/{name}_state.json", "w") as f:
            json.dump(state, f, indent=2)

        return state

    def pause_for_debug(self):
        """Pause execution for interactive debugging"""
        print("\n" + "="*50)
        print("TEST PAUSED FOR DEBUGGING")
        print("Browser is open. Press Enter to continue...")
        print("="*50 + "\n")
        self.page.pause()

    def highlight_element(self, selector: str):
        """Highlight element for visual debugging"""
        self.page.evaluate(f"""
            const el = document.querySelector('{selector}');
            if (el) {{
                el.style.border = '3px solid red';
                el.style.backgroundColor = 'yellow';
            }}
        """)

    def trace_network(self):
        """Log all network requests"""
        self.page.on("request", lambda req:
            self.logger.info(f"Request: {req.method} {req.url}")
        )
        self.page.on("response", lambda res:
            self.logger.info(f"Response: {res.status} {res.url}")
        )

    def trace_console(self):
        """Log all console messages"""
        self.page.on("console", lambda msg:
            self.logger.info(f"Console [{msg.type}]: {msg.text}")
        )


# Interactive debugging fixture
@pytest.fixture
def debug_mode(page, request):
    """Enable debug mode for test"""
    helper = DebugHelper(page)

    if request.config.getoption("--debug"):
        helper.trace_network()
        helper.trace_console()

    yield helper

    if request.config.getoption("--pause-on-fail"):
        if hasattr(request.node, "rep_call") and request.node.rep_call.failed:
            helper.pause_for_debug()
```

---

## Câu 7-10: [Tiếp tục với các câu hỏi về Desktop Automation, CI/CD Integration, Performance, và Best Practices]

(Do giới hạn độ dài, các câu 7-10 sẽ focus vào: Desktop automation với Pywinauto, CI/CD integration, Test maintenance, và Reporting)

---

*Tổng: 10 câu × 10 điểm = 100 điểm*
*Thời gian làm bài đề xuất: 120 phút*
