# Topic 6: AI Testing Tools - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 AI in Software Testing

**AI Testing Categories:**
1. **AI FOR Testing**: Using AI to improve testing (Healenium, Qodo)
2. **Testing FOR AI**: Testing AI systems (covered in Topic 7)

**How AI Helps Testing:**
- Self-healing locators (Healenium)
- Test case generation (Qodo, ChatGPT)
- Visual regression (Applitools)
- Defect prediction
- Test prioritization
- Flaky test detection

### 1.2 Self-Healing Test Automation

**The Problem:**
```
UI Change → Locator Breaks → Test Fails → Manual Fix → Repeat
        ↓
   60% of test maintenance is fixing broken locators
```

**The Solution (Healenium):**
```
UI Change → Locator Breaks → AI Finds Alternative → Test Passes
                                    ↓
                        Reports healed locators for review
```

### 1.3 AI Test Generation Approaches

| Approach | Description | Tools |
|----------|-------------|-------|
| **Code-based** | Analyzes source code to generate tests | Qodo, Diffblue |
| **Spec-based** | Generates from requirements/user stories | ChatGPT, Copilot |
| **Behavior-based** | Learns from existing tests | Testim, Mabl |
| **Exploratory** | AI explores application to find bugs | Applitools Autonomous |

---

## 2. Tools & Techniques Comparison

### 2.1 Healenium Self-Healing

**Architecture:**
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Test Code      │────>│ SelfHealingDriver│────>│    Browser      │
│  (Selenium)     │     │  (Proxy)        │     │                 │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                        ┌────────▼────────┐
                        │  Healenium      │
                        │  Backend        │
                        │  (PostgreSQL)   │
                        │  - Locator history│
                        │  - ML model     │
                        └─────────────────┘
```

**How It Works:**
1. First run: Stores all locator attributes (ID, CSS, XPath, text, class)
2. Locator fails: Queries stored history
3. Uses LCS (Longest Common Subsequence) + ML to find best match
4. Heals locator and continues test
5. Reports healed locators for manual review

**Setup:**
```java
// Maven dependency
<dependency>
    <groupId>com.epam.healenium</groupId>
    <artifactId>healenium-web</artifactId>
    <version>3.4.0</version>
</dependency>

// Usage
WebDriver delegate = new ChromeDriver();
WebDriver driver = SelfHealingDriver.create(delegate);

// Use normally - healing happens automatically
driver.findElement(By.id("old-id")).click();  // Heals if ID changed
```

**Limitations:**
- First run creates baseline (can't heal on first execution)
- Requires Selenium WebDriver
- PostgreSQL backend required
- May suggest incorrect heals (needs review)

### 2.2 AI Test Generation: Qodo vs ChatGPT

| Feature | Qodo (formerly CodiumAI) | ChatGPT |
|---------|--------------------------|---------|
| **Focus** | Specialized test generation | General-purpose |
| **IDE Integration** | VS Code, JetBrains (in-editor) | Web interface |
| **Context Understanding** | Deep code analysis | Limited context |
| **Edge Cases** | Automatically identifies | Requires prompting |
| **Languages** | Python, JS, TS, Java | All languages |
| **Accuracy** | High (specialized) | Variable |
| **Review Needed** | Yes (domain experts) | Essential |

**Qodo Example:**
```python
# Input function
def calculate_discount(price, customer_type):
    if customer_type == "premium":
        return price * 0.8
    elif customer_type == "regular":
        return price * 0.9
    else:
        return price

# Qodo generates:
def test_calculate_discount_premium_customer():
    assert calculate_discount(100, "premium") == 80.0

def test_calculate_discount_regular_customer():
    assert calculate_discount(100, "regular") == 90.0

def test_calculate_discount_unknown_customer():
    assert calculate_discount(100, "unknown") == 100.0

def test_calculate_discount_zero_price():
    assert calculate_discount(0, "premium") == 0.0

def test_calculate_discount_negative_price():
    # Edge case Qodo identifies
    assert calculate_discount(-100, "premium") == -80.0
```

**ChatGPT Prompting for Tests:**
```
Prompt: "Generate pytest test cases for this function, including:
- Happy path tests
- Edge cases (boundary values)
- Error handling tests
- Negative tests

Function:
[paste code]

Requirements:
- Use pytest fixtures
- Include docstrings
- Cover 100% branches"
```

### 2.3 Applitools Visual AI

**Technology:**
- 10+ years training on 4B app screens
- Visual AI replicates human perception
- Ignores rendering noise, detects meaningful changes

**Key Features:**

| Feature | Description |
|---------|-------------|
| **Visual AI** | Smart diff, ignores anti-aliasing/fonts |
| **Ultrafast Grid** | 70+ browser/device combos from single run |
| **Root Cause Analysis** | Pinpoints CSS/DOM changes |
| **Batch Verification** | Review similar changes together |
| **Layout Matching** | Ignores text, checks structure |

**Integration Example (Playwright):**
```javascript
const { Eyes, Target } = require('@applitools/eyes-playwright');

test('homepage visual', async ({ page }) => {
    const eyes = new Eyes();
    await eyes.open(page, 'App Name', 'Test Name');

    await page.goto('https://example.com');

    // Full page check
    await eyes.check('Homepage', Target.window().fully());

    // Element check
    await eyes.check('Header', Target.region('#header'));

    // Ignore dynamic content
    await eyes.check('Dashboard', Target.window()
        .ignoreRegions('.timestamp', '.ad-banner'));

    await eyes.close();
});
```

---

## 3. Practical Test Case Design

### 3.1 Self-Healing Test Implementation

**Scenario: Login Page Test with Healenium**
```java
public class LoginTest {
    private SelfHealingDriver driver;

    @Before
    public void setup() {
        WebDriver delegate = new ChromeDriver();
        driver = SelfHealingDriver.create(delegate);
    }

    @Test
    public void testLogin() {
        driver.get("https://practicesoftwaretesting.com/auth/login");

        // These locators will self-heal if UI changes
        WebElement email = driver.findElement(By.id("email"));
        WebElement password = driver.findElement(By.id("password"));
        WebElement submit = driver.findElement(By.cssSelector("[data-test='login-submit']"));

        email.sendKeys("customer@test.com");
        password.sendKeys("welcome01");
        submit.click();

        // Verify login success
        WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        wait.until(ExpectedConditions.urlContains("/account"));
    }

    @After
    public void teardown() {
        driver.quit();
    }
}
```

**Healing Report Review:**
```json
{
  "healedElements": [
    {
      "originalLocator": "By.id: email",
      "healedLocator": "By.cssSelector: [name='email']",
      "score": 0.95,
      "screenshot": "healed_element_1.png"
    }
  ]
}
```

### 3.2 AI-Generated Test Review Process

**Step 1: Generate Tests**
```python
# Use Qodo/ChatGPT to generate initial tests
# for payment module
```

**Step 2: Review for Domain Logic**
```python
# AI generated:
def test_payment_negative_amount():
    assert process_payment(-100) == {"error": "Invalid amount"}

# Manual review needed:
# - Is negative amount actually invalid?
# - What if it's a refund?
# - Add business context
def test_payment_negative_amount_is_refund():
    """Negative amounts represent refunds"""
    result = process_payment(-100)
    assert result["type"] == "refund"
    assert result["amount"] == 100
```

**Step 3: Add Missing Edge Cases**
```python
# AI might miss:
def test_payment_currency_conversion():
    """Test multi-currency handling"""
    result = process_payment(100, currency="EUR")
    assert result["converted_amount"] > 0

def test_payment_idempotency():
    """Same payment ID should not charge twice"""
    process_payment(100, payment_id="123")
    result = process_payment(100, payment_id="123")
    assert result["status"] == "duplicate"
```

### 3.3 Test Cases for AI Testing Tools

**TC-AI-001: Self-Healing Locator Change**
- **Setup**: Run test suite, change button ID
- **Expected**: Test passes, healing reported
- **Validation**: Review healed locator accuracy

**TC-AI-002: Visual AI False Positive**
- **Setup**: Change timestamp, run visual test
- **Expected**: No difference flagged (dynamic content)
- **Validation**: Verify smart ignore works

**TC-AI-003: AI Test Generation Accuracy**
- **Setup**: Generate tests for function with 5 branches
- **Expected**: All branches covered
- **Validation**: Run coverage report

**TC-AI-004: Ultrafast Grid Cross-Browser**
- **Setup**: Run single test with 10 browser configs
- **Expected**: Results for all browsers in < 30 seconds
- **Validation**: Check all browser results

---

## 4. Troubleshooting Scenarios

### 4.1 Healenium Issues

**Problem 1: Incorrect Healing**
```
Symptom: Test passes but clicks wrong element

Root Cause:
- Multiple similar elements
- Low confidence score accepted

Solution:
1. Set minimum confidence threshold:
   healenium.properties: heal.min.score=0.8

2. Use more specific original locators:
   By.cssSelector("[data-test='submit-btn']")
   instead of By.className("btn")

3. Review healing reports regularly
```

**Problem 2: Healing Backend Issues**
```
Symptom: "Cannot connect to Healenium backend"

Debug Steps:
1. Check PostgreSQL is running:
   docker ps | grep healenium

2. Verify connection string:
   spring.datasource.url=jdbc:postgresql://localhost:5432/healenium

3. Check network/firewall
4. Review logs: docker logs healenium-backend
```

### 4.2 AI Test Generation Issues

**Problem 1: Generated Tests Don't Compile**
```
Symptom: Syntax errors, undefined imports

Solutions:
1. Specify exact language version in prompt
2. Provide import context
3. Use IDE plugin (Qodo) instead of web ChatGPT
4. Add example test as template
```

**Problem 2: Tests Miss Business Logic**
```
Symptom: Tests pass but don't validate requirements

Root Cause: AI lacks domain context

Solutions:
1. Include requirements in prompt:
   "Given these business rules: [rules]"

2. Add existing test examples as context

3. Manual review checklist:
   - [ ] Validates business rules
   - [ ] Tests error conditions
   - [ ] Covers edge cases
   - [ ] Uses realistic data
```

### 4.3 Visual Testing Issues

**Problem 1: Too Many False Positives**
```
Symptom: Every test flags differences (fonts, anti-aliasing)

Solutions:
1. Use Layout match level:
   eyes.setMatchLevel(MatchLevel.Layout)

2. Ignore dynamic regions:
   Target.window().ignoreRegions('.dynamic')

3. Set baseline with stable environment
4. Use Visual AI (Applitools) instead of pixel-diff
```

---

## 5. Test Oracle Problem

### 5.1 What is the Test Oracle Problem?

**Definition:** The difficulty of determining the correct expected output for a test, especially in AI/ML systems.

**The Challenge:**
```
Traditional Software:
Input: 2 + 2
Expected Output: 4 (deterministic)

AI/ML Software:
Input: Image of cat
Expected Output: "cat" with 95% confidence? 90%? 85%?
                 What's the acceptable threshold?
```

### 5.2 Solutions to Test Oracle Problem

| Solution | Description | Use Case |
|----------|-------------|----------|
| **Statistical Oracle** | Accept output within range | ML models (accuracy ±3%) |
| **Differential Testing** | Compare two implementations | Multiple models |
| **Metamorphic Testing** | Test relationships between I/O | Image classifiers |
| **Dynamic Oracle** | ML learns expected behavior | Long-running systems |
| **Pseudo-Oracle** | Separate validation program | Complex calculations |

**Statistical Oracle Example:**
```python
def test_sentiment_model():
    """Accept predictions within confidence range"""
    result = sentiment_model.predict("I love this product!")

    # Oracle: positive sentiment with >80% confidence
    assert result.label == "positive"
    assert 0.80 <= result.confidence <= 1.0
```

**Metamorphic Testing Example:**
```python
def test_image_classifier_rotation():
    """Rotating image shouldn't change classification"""
    original = classify(image)
    rotated = classify(rotate(image, 90))

    # Metamorphic relation: same class regardless of rotation
    assert original.label == rotated.label
```

---

## 6. Application-Level Exam Questions

### Question 1: Healenium Architecture
**Scenario:** Test suite has 500 tests with 2000 locators. After UI redesign, 30% of locators broke.

**Question:** How would Healenium help? What are limitations?

**Answer:**
```
Benefits:
1. Auto-heal 30% broken locators (600 elements)
2. Reduce manual fix time from days to hours
3. Generate healing report for review
4. Tests continue running during redesign

Process:
- First run after redesign: Creates new baseline
- Subsequent runs: Heals changed locators
- Review reports: Accept/reject healed locators
- Update locators based on recommendations

Limitations:
1. Can't heal on first run (needs history)
2. May suggest incorrect heals (review required)
3. New elements not in history won't heal
4. Requires PostgreSQL backend
5. Some locator changes too drastic to heal

Recommendation:
- Use Healenium for maintenance reduction
- Implement data-test attributes for critical elements
- Review healing reports weekly
- Manual update for new features
```

### Question 2: AI Test Generation Strategy
**Scenario:** New payment module needs tests. Team is considering Qodo vs ChatGPT.

**Question:** Compare the approaches and recommend a strategy.

**Answer:**
```
Comparison:

Qodo:
- Pro: IDE integration (inline suggestions)
- Pro: Deep code analysis (understands context)
- Pro: Automatic edge case detection
- Con: Limited language support
- Con: May miss business logic

ChatGPT:
- Pro: Any language, flexible prompting
- Pro: Can include business requirements
- Pro: Explains generated tests
- Con: No code context (limited window)
- Con: May generate non-compiling code
- Con: Inconsistent quality

Recommended Strategy:
1. Use Qodo for initial test scaffold (technical tests)
2. Use ChatGPT for business logic tests (with requirements in prompt)
3. Manual review ALL generated tests
4. Add domain-specific edge cases manually
5. Run mutation testing to verify quality

Example Workflow:
- Qodo generates: Unit tests, boundary tests
- ChatGPT generates: Integration scenarios with business rules
- Manual adds: Security tests, performance tests
- Review removes: Duplicate, incorrect tests
```

### Question 3: Visual AI vs Pixel-Diff
**Scenario:** E-commerce site has frequent false positives in visual tests due to:
- Font rendering differences between CI and local
- Dynamic prices and timestamps
- Promotional banners that change

**Question:** How would Visual AI help?

**Answer:**
```
Visual AI (Applitools) Advantages:

1. Font Rendering:
   - Pixel-diff: Fails on anti-aliasing differences
   - Visual AI: Recognizes text regardless of rendering

2. Dynamic Content:
   - Pixel-diff: Must manually configure ignores
   - Visual AI: Intelligent content recognition
   - Auto-ignores timestamps, IDs, prices
   - Can set "Ignore Colors" for promotional changes

3. Promotional Banners:
   - Pixel-diff: Fails every time banner changes
   - Visual AI: Use "Layout" match level
   - Compares structure, not content

Implementation:
```javascript
// Configure for e-commerce
const eyes = new Eyes();

eyes.setConfiguration(
    config.setMatchLevel(MatchLevel.Layout)  // Structure only
          .setIgnoreDisplacements(true)       // Allow element movement
);

await eyes.check('Product Page', Target.window()
    .ignoreRegions('.price', '.promo-banner', '.timestamp')
    .layoutRegions('.product-grid'));  // Only check layout
```

Result:
- 90% reduction in false positives
- Tests focus on meaningful visual changes
- CI/CD stable across environments
```

---

## 7. Cheatsheet / Quick Reference

### Healenium Commands

```java
// Setup
WebDriver delegate = new ChromeDriver();
SelfHealingDriver driver = SelfHealingDriver.create(delegate);

// Configuration (healenium.properties)
recovery-tries=1
heal.min.score=0.7
enable.self.healing=true

// Docker setup
docker run -d --name healenium-backend \
    -e SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/healenium \
    healenium/hlm-backend:latest
```

### Applitools Commands

```javascript
// Setup
const { Eyes, Target, Configuration } = require('@applitools/eyes-playwright');
const eyes = new Eyes();

// Configuration
const config = new Configuration();
config.setApiKey(process.env.APPLITOOLS_API_KEY);
config.setBatch(new BatchInfo('Batch Name'));
config.setMatchLevel(MatchLevel.Strict);  // or Layout, Content

// Checks
await eyes.check('Full Page', Target.window().fully());
await eyes.check('Region', Target.region('#element'));
await eyes.check('Ignore', Target.window()
    .ignoreRegions('.dynamic')
    .layoutRegions('.ads'));

// Match levels
MatchLevel.Exact      // Pixel-perfect
MatchLevel.Strict     // Smart pixel diff (default)
MatchLevel.Content    // Ignores colors
MatchLevel.Layout     // Structure only
```

### AI Test Generation Prompt Template

```
Generate [framework] tests for the following code:

Requirements:
1. Test happy path
2. Test boundary values
3. Test error handling
4. Test edge cases
5. Achieve 100% branch coverage

Business Rules:
- [List business requirements]

Code:
```[language]
[paste code]
```

Output format:
- Use descriptive test names
- Include docstrings
- Use fixtures for setup
- Add assertions with clear messages
```

### AI Testing Checklist

- [ ] Self-healing configured for UI tests
- [ ] Healing reports reviewed weekly
- [ ] AI-generated tests manually reviewed
- [ ] Visual AI configured (not pixel-diff)
- [ ] Dynamic content ignored in visual tests
- [ ] Test oracle defined for AI outputs
- [ ] Confidence thresholds set
- [ ] Metamorphic relations identified

---

## 8. Key Takeaways for Exam

1. **Healenium**: LCS + ML for locator healing; needs history first
2. **Qodo vs ChatGPT**: Qodo for code analysis, ChatGPT for business logic
3. **Visual AI**: 78% less maintenance than pixel-diff
4. **Test Oracle Problem**: Statistical, differential, metamorphic solutions
5. **Review Required**: All AI-generated tests need human review
6. **2025 Stats**: 84% devs use AI tools, 41% commits AI-assisted
7. **Applitools**: 10+ years, 4B images, Ultrafast Grid

---

*Study time recommended: 3-4 hours*
*Practice: Set up Healenium with a sample Selenium project*
