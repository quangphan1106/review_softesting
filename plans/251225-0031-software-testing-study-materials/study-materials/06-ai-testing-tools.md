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
- Add Maven dependency: `healenium-web`
- Wrap driver: `SelfHealingDriver.create(new ChromeDriver())`
- Use normally - healing happens automatically

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
- Input: `calculate_discount(price, customer_type)` function
- Qodo auto-generates: premium/regular/unknown tests + edge cases (zero, negative price)
- Key: Identifies edge cases automatically

**ChatGPT Prompting:**
- Include: happy path, edge cases, error handling, negative tests
- Specify: framework (pytest), coverage target, output format
- Always provide code + business context in prompt

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

**Integration Concepts:**
- `eyes.open(page, 'App', 'Test')` - Start session
- `eyes.check('Name', Target.window().fully())` - Full page
- `Target.region('#header')` - Element only
- `.ignoreRegions('.timestamp')` - Skip dynamic content
- `eyes.close()` - End session

---

## 3. Practical Test Case Design

### 3.1 Self-Healing Test Concepts

**Healenium Usage:**
- Wrap driver: `SelfHealingDriver.create(delegate)`
- Use `findElement()` normally - heals automatically
- Review healing report: original locator → healed locator + score

**Healing Report Contains:**
- Original vs healed locator
- Confidence score (e.g., 0.95)
- Screenshot for review

### 3.2 AI-Generated Test Review Process

**Review Checklist:**
1. **Generate** - Use Qodo/ChatGPT for initial scaffold
2. **Domain Review** - AI may miss business context (e.g., negative = refund, not error)
3. **Add Missing** - Currency conversion, idempotency, security tests

**Key Point:** AI lacks domain knowledge - manual review essential

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

| Problem | Symptom | Solution |
|---------|---------|----------|
| Incorrect Healing | Clicks wrong element | Set `heal.min.score=0.8`, use specific locators |
| Backend Connection | Cannot connect error | Check PostgreSQL, verify connection string |

### 4.2 AI Test Generation Issues

| Problem | Symptom | Solution |
|---------|---------|----------|
| Won't Compile | Syntax errors | Specify language version, use IDE plugin |
| Missing Business Logic | Tests pass but wrong | Include requirements in prompt, manual review |

**Review Checklist:** Validates rules, error conditions, edge cases, realistic data

### 4.3 Visual Testing Issues

| Problem | Solution |
|---------|----------|
| False Positives | Use `MatchLevel.Layout`, ignore dynamic regions |
| Font Differences | Use Visual AI instead of pixel-diff |

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

**Examples:**
- **Statistical Oracle:** Accept confidence ≥80% for sentiment model
- **Metamorphic Testing:** Rotating image shouldn't change classification result

---

## 6. Application-Level Exam Questions

### Question 1: Healenium Architecture
**Scenario:** 500 tests, 2000 locators, 30% broke after UI redesign

**Answer:**
| Benefits | Limitations |
|----------|-------------|
| Auto-heal 600 locators | Can't heal first run |
| Days → hours for fixes | May suggest wrong heals |
| Reports for review | New elements won't heal |
| Tests continue running | Requires PostgreSQL |

**Recommendation:** Use for maintenance, add data-test attributes, review reports weekly

### Question 2: AI Test Generation Strategy
**Scenario:** Payment module needs tests - Qodo vs ChatGPT?

**Answer:**
| Tool | Pros | Cons |
|------|------|------|
| Qodo | IDE integration, edge case detection | Limited languages, misses business logic |
| ChatGPT | Any language, flexible | No code context, may not compile |

**Strategy:** Qodo for technical tests → ChatGPT for business logic → Manual review ALL → Add security/performance tests

### Question 3: Visual AI vs Pixel-Diff
**Scenario:** False positives from fonts, dynamic prices, banners

**Answer:**
| Issue | Pixel-Diff | Visual AI |
|-------|------------|-----------|
| Fonts | Fails on anti-aliasing | Recognizes text |
| Dynamic | Manual ignore config | Auto-ignores timestamps |
| Banners | Fails on every change | Use Layout match |

**Result:** 90% reduction in false positives, stable CI/CD

---

## 7. Cheatsheet / Quick Reference

### Healenium Config

| Setting | Purpose |
|---------|---------|
| `SelfHealingDriver.create(delegate)` | Wrap driver |
| `heal.min.score=0.7` | Minimum confidence |
| `enable.self.healing=true` | Enable healing |

### Applitools Config

| Config/Check | Purpose |
|--------------|---------|
| `eyes.setApiKey()` | Authentication |
| `config.setMatchLevel()` | Exact/Strict/Content/Layout |
| `Target.window().fully()` | Full page |
| `.ignoreRegions('.dynamic')` | Skip areas |
| `.layoutRegions('.ads')` | Structure only |

**Match Levels:** Exact (pixel-perfect) → Strict (default) → Content (ignore colors) → Layout (structure)

### AI Test Generation Prompt

Include in prompt: framework, happy path, boundaries, errors, edge cases, coverage target, business rules, output format

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
