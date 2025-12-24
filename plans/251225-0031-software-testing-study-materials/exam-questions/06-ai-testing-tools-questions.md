# Topic 6: AI Testing Tools - Application Questions
**CS423 Software Testing | Final Exam Practice**

---

## Hướng dẫn
- 10 câu hỏi x 10 điểm = 100 điểm
- Mỗi câu hỏi ở mức **vận dụng** (application level)
- Tập trung vào Healenium, Qodo, Applitools Visual AI

---

## Câu 1: Healenium Self-Healing Architecture (10 điểm)

### Tình huống
Team có 500 Selenium tests với 2000 locators. Sau UI redesign, 30% locators bị broken. Team xem xét adopt Healenium để giảm maintenance.

### Yêu cầu
a) Giải thích cách Healenium healing algorithm hoạt động (3 điểm)
b) Design implementation plan cho existing test suite (4 điểm)
c) Identify limitations và mitigation strategies (3 điểm)

### Đáp án mẫu

**a) Healenium Healing Algorithm (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HEALENIUM HEALING ALGORITHM                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: LOCATOR STORAGE (First Run)                                       │
│  ──────────────────────────────────                                          │
│                                                                              │
│  For each element interaction:                                               │
│  1. Capture original locator (e.g., By.id("submit-btn"))                    │
│  2. Extract ALL element attributes:                                         │
│     ┌────────────────────────────────────────────────────────┐              │
│     │ - id: "submit-btn"                                      │              │
│     │ - class: "btn btn-primary"                             │              │
│     │ - tag: "button"                                         │              │
│     │ - text: "Submit"                                        │              │
│     │ - name: "submit"                                        │              │
│     │ - data-test: "submit-button"                           │              │
│     │ - xpath: "/html/body/form/button[2]"                   │              │
│     │ - css: "form > button.btn-primary"                     │              │
│     │ - parent: {class: "form-group", id: "form-actions"}   │              │
│     └────────────────────────────────────────────────────────┘              │
│  3. Store in PostgreSQL database with page URL                              │
│                                                                              │
│                                                                              │
│  PHASE 2: HEALING (When Locator Fails)                                      │
│  ─────────────────────────────────────                                       │
│                                                                              │
│  1. Original locator fails: NoSuchElementException                          │
│                                                                              │
│  2. Query database for stored attributes                                    │
│                                                                              │
│  3. Find all candidate elements on current page                             │
│                                                                              │
│  4. Calculate similarity score using LCS + ML:                              │
│     ┌────────────────────────────────────────────────────────┐              │
│     │ Candidate: <button class="btn submit" data-test="submit-button">      │
│     │                                                         │              │
│     │ Score calculation:                                      │              │
│     │ - id match: 0% (changed)                               │              │
│     │ - class match: 60% (partial - "btn" matches)           │              │
│     │ - tag match: 100%                                       │              │
│     │ - text match: 100%                                      │              │
│     │ - data-test match: 100%                                │              │
│     │ - position similarity: 85%                              │              │
│     │                                                         │              │
│     │ LCS Algorithm: Longest Common Subsequence of attributes│              │
│     │ Final Score: 0.89 (89% similarity)                     │              │
│     └────────────────────────────────────────────────────────┘              │
│                                                                              │
│  5. If score > threshold (default 0.7):                                     │
│     - Use candidate element                                                  │
│     - Continue test execution                                               │
│     - Log healing event                                                      │
│                                                                              │
│  6. If score < threshold:                                                   │
│     - Test fails                                                            │
│     - Report element not found                                              │
│                                                                              │
│                                                                              │
│  PHASE 3: REPORTING                                                          │
│  ─────────────────                                                           │
│                                                                              │
│  Healing Report:                                                             │
│  ┌────────────────────────────────────────────────────────┐                 │
│  │ {                                                       │                 │
│  │   "originalLocator": "By.id: submit-btn",              │                 │
│  │   "healedLocator": "By.cssSelector: [data-test='submit-button']",       │
│  │   "score": 0.89,                                        │                 │
│  │   "screenshot": "healed_element_1.png",                │                 │
│  │   "recommendedAction": "Update locator in code"        │                 │
│  │ }                                                       │                 │
│  └────────────────────────────────────────────────────────┘                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Implementation Plan (4 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HEALENIUM IMPLEMENTATION PLAN                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PHASE 1: INFRASTRUCTURE SETUP (Week 1)                                      │
│  ───────────────────────────────────────                                     │
│                                                                              │
│  1. Deploy Healenium Backend (Docker)                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ # docker-compose.yml                                                 │    │
│  │ version: '3.8'                                                       │    │
│  │ services:                                                            │    │
│  │   healenium-backend:                                                 │    │
│  │     image: healenium/hlm-backend:3.4.0                              │    │
│  │     ports:                                                           │    │
│  │       - "7878:7878"                                                 │    │
│  │     environment:                                                     │    │
│  │       - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/hlm  │    │
│  │       - SPRING_DATASOURCE_USERNAME=healenium                        │    │
│  │                                                                      │    │
│  │   postgres:                                                          │    │
│  │     image: postgres:15                                               │    │
│  │     environment:                                                     │    │
│  │       - POSTGRES_DB=hlm                                              │    │
│  │       - POSTGRES_USER=healenium                                      │    │
│  │       - POSTGRES_PASSWORD=healenium                                  │    │
│  │                                                                      │    │
│  │   healenium-report:                                                  │    │
│  │     image: healenium/hlm-report:3.4.0                               │    │
│  │     ports:                                                           │    │
│  │       - "7879:7879"                                                 │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  2. Add Maven dependency:                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ <dependency>                                                         │    │
│  │   <groupId>com.epam.healenium</groupId>                             │    │
│  │   <artifactId>healenium-web</artifactId>                            │    │
│  │   <version>3.4.0</version>                                          │    │
│  │ </dependency>                                                        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  PHASE 2: CODE MIGRATION (Week 2-3)                                          │
│  ──────────────────────────────────                                          │
│                                                                              │
│  1. Create wrapper class:                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ public class HealeniumDriverFactory {                                │    │
│  │     public static WebDriver createDriver() {                         │    │
│  │         WebDriver delegate = new ChromeDriver();                     │    │
│  │         return SelfHealingDriver.create(delegate);                  │    │
│  │     }                                                                │    │
│  │ }                                                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  2. Update base test class:                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ @Before                                                              │    │
│  │ public void setup() {                                                │    │
│  │     // OLD: driver = new ChromeDriver();                            │    │
│  │     driver = HealeniumDriverFactory.createDriver();                 │    │
│  │ }                                                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  3. No changes needed to test code (driver interface same)                  │
│                                                                              │
│                                                                              │
│  PHASE 3: BASELINE CREATION (Week 3)                                         │
│  ───────────────────────────────────                                         │
│                                                                              │
│  1. Run full test suite BEFORE UI redesign                                  │
│  2. All 2000 locators stored in database                                    │
│  3. Screenshots captured as baseline                                        │
│                                                                              │
│                                                                              │
│  PHASE 4: VALIDATION (Week 4)                                                │
│  ────────────────────────────                                                │
│                                                                              │
│  1. Apply UI redesign                                                        │
│  2. Run test suite                                                          │
│  3. Review healing report:                                                   │
│     - 600 locators healed (30% of 2000)                                     │
│     - Review each healed locator                                            │
│     - Accept/reject healing suggestions                                      │
│                                                                              │
│  4. Update locators based on recommendations                                │
│  5. Re-run to confirm all tests pass                                        │
│                                                                              │
│                                                                              │
│  SUCCESS METRICS                                                             │
│  ───────────────                                                             │
│  - Maintenance time reduced: 80% (from 40 hours to 8 hours)                │
│  - Test suite stability: 95%+ pass rate post-redesign                       │
│  - Healing accuracy: 90%+ correct healings                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**c) Limitations and Mitigation (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HEALENIUM LIMITATIONS & MITIGATIONS                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  LIMITATION 1: Cannot heal on first run                                      │
│  ─────────────────────────────────────                                       │
│  Problem: No history = no healing possible                                  │
│  Mitigation:                                                                │
│  - Run baseline suite before any UI changes                                 │
│  - Maintain baseline database across environments                           │
│  - Export/import baseline for new team members                              │
│                                                                              │
│  LIMITATION 2: Incorrect healing (false positives)                           │
│  ─────────────────────────────────────────────────                           │
│  Problem: Algorithm may select wrong element                                │
│  Mitigation:                                                                │
│  - Increase threshold: heal.min.score=0.85 (default 0.7)                   │
│  - Use data-test attributes (most stable)                                   │
│  - Review healing reports weekly                                            │
│  - Add visual validation after healed actions                               │
│                                                                              │
│  LIMITATION 3: New elements not healed                                       │
│  ────────────────────────────────────                                        │
│  Problem: Elements added after baseline not in database                     │
│  Mitigation:                                                                │
│  - Regular baseline refresh (weekly/bi-weekly)                              │
│  - Add new tests immediately to capture new elements                        │
│  - Use Healenium's API to manually add elements                            │
│                                                                              │
│  LIMITATION 4: Performance overhead                                          │
│  ──────────────────────────────────                                          │
│  Problem: Additional database queries slow tests                            │
│  Mitigation:                                                                │
│  - Use connection pooling                                                   │
│  - Enable caching: healenium.cache.enabled=true                            │
│  - Run Healenium backend on same network                                    │
│                                                                              │
│  LIMITATION 5: Selenium-only support                                         │
│  ─────────────────────────────────                                           │
│  Problem: Not compatible with Playwright/Cypress                            │
│  Mitigation:                                                                │
│  - For Playwright: Use built-in auto-waiting + data-test                   │
│  - For Cypress: Use cy-data-testid plugin                                  │
│  - Keep Selenium for legacy, migrate new tests to Playwright               │
│                                                                              │
│  LIMITATION 6: Complex dynamic UIs                                           │
│  ──────────────────────────────────                                          │
│  Problem: SPAs with constantly changing DOM                                 │
│  Mitigation:                                                                │
│  - Wait for stable state before element interactions                        │
│  - Use more specific locators (data-test > id > class)                     │
│  - Combine with explicit waits                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 2: AI Test Generation Comparison (10 điểm)

### Tình huống
Team đang evaluate AI tools cho test generation. Có 2 options:
- Qodo (CodiumAI) - IDE integration
- ChatGPT/Claude - General purpose

### Yêu cầu
a) So sánh chi tiết 2 approaches (4 điểm)
b) Design evaluation criteria và pilot project (3 điểm)
c) Implement test generation workflow (3 điểm)

### Đáp án mẫu

**a) Detailed Comparison (4 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AI TEST GENERATION COMPARISON                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────┬─────────────────────┬─────────────────────┐        │
│  │ Aspect              │ Qodo (CodiumAI)     │ ChatGPT/Claude      │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Integration         │ VS Code, JetBrains  │ Web interface       │        │
│  │                     │ Inline suggestions  │ Copy-paste workflow │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Context Awareness   │ Full codebase       │ Limited context     │        │
│  │                     │ Imports, types,     │ Only pasted code    │        │
│  │                     │ dependencies        │ (token limit)       │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Edge Case Detection │ Automatic           │ Requires prompting  │        │
│  │                     │ Boundary values,    │ "Include edge       │        │
│  │                     │ null checks         │ cases..."           │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Test Framework      │ Auto-detects        │ Needs specification │        │
│  │                     │ pytest, jest, etc.  │ in prompt           │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Languages           │ Python, JS, TS,     │ All languages       │        │
│  │                     │ Java (limited)      │                     │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Accuracy            │ High (specialized)  │ Variable            │        │
│  │                     │ Compiles reliably   │ May not compile     │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Business Logic      │ May miss (code-     │ Can include if      │        │
│  │                     │ focused)            │ requirements given  │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Review Needed       │ Yes (technical)     │ Essential (both     │        │
│  │                     │                     │ technical + domain) │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Cost                │ Free tier + $19/mo  │ $20/mo (ChatGPT     │        │
│  │                     │                     │ Plus) or API costs  │        │
│  ├─────────────────────┼─────────────────────┼─────────────────────┤        │
│  │ Best For            │ Unit tests,         │ Integration tests,  │        │
│  │                     │ rapid iteration     │ custom scenarios    │        │
│  └─────────────────────┴─────────────────────┴─────────────────────┘        │
│                                                                              │
│  EXAMPLE OUTPUTS                                                             │
│  ───────────────                                                             │
│                                                                              │
│  Input Function:                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ def calculate_discount(price, customer_type, is_holiday=False):     │    │
│  │     if price <= 0:                                                   │    │
│  │         raise ValueError("Price must be positive")                  │    │
│  │     if customer_type == "premium":                                   │    │
│  │         discount = 0.2                                               │    │
│  │     elif customer_type == "regular":                                 │    │
│  │         discount = 0.1                                               │    │
│  │     else:                                                            │    │
│  │         discount = 0                                                 │    │
│  │     if is_holiday:                                                   │    │
│  │         discount += 0.05                                             │    │
│  │     return price * (1 - discount)                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Qodo Output (auto-generated):                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ def test_premium_customer_discount():                                │    │
│  │     assert calculate_discount(100, "premium") == 80.0               │    │
│  │                                                                      │    │
│  │ def test_regular_customer_discount():                                │    │
│  │     assert calculate_discount(100, "regular") == 90.0               │    │
│  │                                                                      │    │
│  │ def test_unknown_customer_no_discount():                             │    │
│  │     assert calculate_discount(100, "unknown") == 100.0              │    │
│  │                                                                      │    │
│  │ def test_holiday_adds_extra_discount():                              │    │
│  │     assert calculate_discount(100, "premium", True) == 75.0         │    │
│  │                                                                      │    │
│  │ def test_negative_price_raises_error():                              │    │
│  │     with pytest.raises(ValueError):                                  │    │
│  │         calculate_discount(-100, "premium")                          │    │
│  │                                                                      │    │
│  │ def test_zero_price_raises_error():                                  │    │
│  │     with pytest.raises(ValueError):                                  │    │
│  │         calculate_discount(0, "premium")                             │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ✓ Auto-detected edge cases: negative, zero                                │
│  ✓ All branches covered                                                     │
│  ✗ Missing: boundary values (0.01), large values                            │
│                                                                              │
│  ChatGPT Output (with good prompt):                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ # Includes all above PLUS:                                           │    │
│  │                                                                      │    │
│  │ @pytest.mark.parametrize("price,customer,holiday,expected", [        │    │
│  │     (100, "premium", False, 80),                                    │    │
│  │     (100, "regular", False, 90),                                    │    │
│  │     (100, "new", False, 100),                                       │    │
│  │     (100, "premium", True, 75),                                     │    │
│  │     (0.01, "premium", False, 0.008),  # Boundary                    │    │
│  │     (1000000, "premium", False, 800000),  # Large value             │    │
│  │ ])                                                                   │    │
│  │ def test_discount_calculation(price, customer, holiday, expected):   │    │
│  │     result = calculate_discount(price, customer, holiday)            │    │
│  │     assert result == pytest.approx(expected)                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│  ✓ Parametrized tests                                                       │
│  ✓ Boundary values                                                          │
│  ✗ Required specific prompting for these additions                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Evaluation Criteria and Pilot (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EVALUATION CRITERIA & PILOT PROJECT                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  EVALUATION CRITERIA (Weighted Scoring)                                      │
│  ──────────────────────────────────────                                      │
│                                                                              │
│  ┌─────────────────────────┬────────┬────────────────────────────────────┐  │
│  │ Criteria                │ Weight │ Measurement                         │  │
│  ├─────────────────────────┼────────┼────────────────────────────────────┤  │
│  │ Compilation Rate        │ 25%    │ % of generated tests that compile  │  │
│  │ Coverage Increase       │ 25%    │ Branch coverage delta              │  │
│  │ Bug Detection           │ 20%    │ # real bugs found by generated     │  │
│  │ Time Saved              │ 15%    │ Hours saved vs manual writing      │  │
│  │ Review Effort           │ 10%    │ Time to review and fix tests       │  │
│  │ Developer Satisfaction  │ 5%     │ Survey score (1-5)                 │  │
│  └─────────────────────────┴────────┴────────────────────────────────────┘  │
│                                                                              │
│                                                                              │
│  PILOT PROJECT DESIGN                                                        │
│  ────────────────────                                                        │
│                                                                              │
│  Module Selection: Payment Processing Module                                │
│  - 15 functions, ~500 lines of code                                        │
│  - Mix of simple and complex logic                                          │
│  - Current coverage: 45%                                                    │
│                                                                              │
│  Timeline: 2 weeks                                                          │
│                                                                              │
│  Week 1: Qodo Evaluation                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Day 1-2: Setup and training                                          │    │
│  │ Day 3-4: Generate tests for 8 functions                              │    │
│  │ Day 5: Review, fix, measure metrics                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Week 2: ChatGPT/Claude Evaluation                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Day 1-2: Develop prompt templates                                    │    │
│  │ Day 3-4: Generate tests for same 8 functions                         │    │
│  │ Day 5: Review, fix, measure metrics                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Control Group:                                                             │
│  - 7 remaining functions: Manual test writing                              │
│  - Track time and quality for comparison                                    │
│                                                                              │
│                                                                              │
│  SCORECARD TEMPLATE                                                          │
│  ──────────────────                                                          │
│                                                                              │
│  ┌─────────────────────┬─────────────┬─────────────┬─────────────┐          │
│  │ Metric              │ Qodo        │ ChatGPT     │ Manual      │          │
│  ├─────────────────────┼─────────────┼─────────────┼─────────────┤          │
│  │ Tests Generated     │ ___         │ ___         │ ___         │          │
│  │ Compilation Rate    │ ____%       │ ____%       │ 100%        │          │
│  │ Coverage Before     │ 45%         │ 45%         │ 45%         │          │
│  │ Coverage After      │ ____%       │ ____%       │ ____%       │          │
│  │ Bugs Found          │ ___         │ ___         │ ___         │          │
│  │ Time Spent (hrs)    │ ___         │ ___         │ ___         │          │
│  │ Review Time (hrs)   │ ___         │ ___         │ N/A         │          │
│  │ Dev Satisfaction    │ ___/5       │ ___/5       │ ___/5       │          │
│  └─────────────────────┴─────────────┴─────────────┴─────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**c) Test Generation Workflow (3 điểm)**

```python
# Hybrid Test Generation Workflow

"""
RECOMMENDED WORKFLOW: Qodo + ChatGPT Hybrid

1. Qodo for initial scaffold (fast, accurate)
2. ChatGPT for business logic tests (context-aware)
3. Manual review ALL generated tests
4. Mutation testing to verify quality
"""

# ============ WORKFLOW IMPLEMENTATION ============

# Step 1: Qodo generates initial tests (in IDE)
# Developer clicks "Generate Tests" on function

# Step 2: ChatGPT for business scenarios
CHATGPT_PROMPT_TEMPLATE = """
Generate pytest test cases for this payment processing function.

FUNCTION:
```python
{code}
```

BUSINESS REQUIREMENTS:
- Payments over $10,000 require additional verification
- International cards have 2.5% fee
- Refunds must match original transaction
- Maximum 3 retry attempts on failure

Generate tests for:
1. Happy path scenarios
2. Business rule validation
3. Edge cases (boundary values)
4. Error handling
5. Security tests (injection, overflow)

Use pytest fixtures and parametrize where appropriate.
Include docstrings explaining what each test validates.
"""

# Step 3: Review Process
"""
Review Checklist for AI-Generated Tests:

□ Test compiles and runs
□ Assertions are meaningful (not just "is not None")
□ Test name describes behavior being tested
□ Edge cases match business requirements
□ No hardcoded sensitive data
□ Uses appropriate fixtures
□ Covers both positive and negative cases
□ No duplicate tests
□ Test is deterministic (no random without seed)
"""

# Step 4: Quality Verification with Mutation Testing
# Run mutmut to verify tests actually catch bugs

# mutation_testing.sh
"""
#!/bin/bash
# Run mutation testing on generated tests

mutmut run \
  --paths-to-mutate=src/payment/ \
  --tests-dir=tests/payment/ \
  --runner="pytest tests/payment/"

# Check mutation score
mutmut results

# Goal: 80%+ mutations killed
"""

# Step 5: Integration into CI/CD
# .github/workflows/test-generation.yml
"""
name: AI Test Generation Quality Gate

on:
  push:
    paths:
      - 'tests/**'

jobs:
  verify-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests
        run: pytest tests/ --cov=src/ --cov-report=xml

      - name: Check coverage threshold
        run: |
          coverage=$(python -c "import xml.etree.ElementTree as ET; ...")
          if [ "$coverage" -lt "80" ]; then
            echo "Coverage below 80%"
            exit 1
          fi

      - name: Run mutation testing
        run: |
          mutmut run --quick
          score=$(mutmut results | grep "Killed" | ...)
          if [ "$score" -lt "70" ]; then
            echo "Mutation score below 70%"
            exit 1
          fi
"""
```

---

## Câu 3: Applitools Visual AI Configuration (10 điểm)

### Tình huống
E-commerce site có:
- 200 pages
- 3 browsers (Chrome, Firefox, Safari)
- 3 viewports (Desktop, Tablet, Mobile)
- Dynamic content: prices, timestamps, promotions

Team cần implement Visual AI testing với Applitools.

### Yêu cầu
a) Design Visual AI testing strategy (3 điểm)
b) Implement Applitools configuration cho dynamic content (4 điểm)
c) Calculate ROI so với pixel-diff approach (3 điểm)

### Đáp án mẫu

**a) Visual AI Testing Strategy (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    VISUAL AI TESTING STRATEGY                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TEST CONFIGURATION MATRIX                                                   │
│  ─────────────────────────                                                   │
│                                                                              │
│  Total Checkpoints: 200 pages × 3 browsers × 3 viewports = 1,800           │
│                                                                              │
│  TIERED APPROACH                                                             │
│  ───────────────                                                             │
│                                                                              │
│  Tier 1: Critical Pages (Every PR) - 15 pages                               │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Pages: Homepage, Login, Cart, Checkout, Payment, Confirmation       │    │
│  │ Browsers: Chrome only (fast feedback)                               │    │
│  │ Viewports: All 3                                                     │    │
│  │ Checkpoints: 15 × 1 × 3 = 45                                        │    │
│  │ Match Level: Strict                                                  │    │
│  │ Time: ~5 minutes                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Tier 2: Important Pages (Nightly) - 50 pages                               │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Pages: Category, Product, Search, Account, Orders                   │    │
│  │ Browsers: Chrome, Firefox, Safari                                   │    │
│  │ Viewports: Desktop, Mobile                                          │    │
│  │ Checkpoints: 50 × 3 × 2 = 300                                       │    │
│  │ Match Level: Layout                                                  │    │
│  │ Time: ~15 minutes                                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Tier 3: All Pages (Weekly) - 200 pages                                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Pages: Complete site coverage                                        │    │
│  │ Browsers: All 3                                                      │    │
│  │ Viewports: All 3                                                     │    │
│  │ Checkpoints: 200 × 3 × 3 = 1,800                                    │    │
│  │ Match Level: Layout                                                  │    │
│  │ Time: ~45 minutes (Ultrafast Grid)                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  DYNAMIC CONTENT HANDLING                                                    │
│  ─────────────────────────                                                   │
│                                                                              │
│  ┌─────────────────────┬───────────────────────────────────────────────┐    │
│  │ Content Type        │ Strategy                                       │    │
│  ├─────────────────────┼───────────────────────────────────────────────┤    │
│  │ Prices              │ Ignore region or mock API                     │    │
│  │ Timestamps          │ Ignore region + freeze time                   │    │
│  │ Promotions          │ Layout match level                             │    │
│  │ User avatars        │ Mock with consistent image                    │    │
│  │ Carousels           │ Freeze at slide 0                              │    │
│  │ Loading spinners    │ Wait for networkidle                          │    │
│  │ Ads                 │ Ignore region                                  │    │
│  └─────────────────────┴───────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Applitools Configuration (4 điểm)**

```javascript
// applitools.config.js
const {
  Eyes,
  Target,
  Configuration,
  BatchInfo,
  MatchLevel,
  BrowserType,
  DeviceName,
  ScreenOrientation,
  VisualGridRunner
} = require('@applitools/eyes-playwright');

// Create runner with concurrency
const runner = new VisualGridRunner({ testConcurrency: 10 });

// Configuration for e-commerce visual testing
function createConfiguration(tier = 'critical') {
  const config = new Configuration();

  // API Key
  config.setApiKey(process.env.APPLITOOLS_API_KEY);

  // Batch for grouping
  config.setBatch(new BatchInfo(`E-Commerce Visual Tests - ${tier}`));

  // App and test names
  config.setAppName('E-Commerce Site');

  // Tier-specific browser configuration
  if (tier === 'critical') {
    // Tier 1: Chrome only, all viewports
    config.addBrowser(1920, 1080, BrowserType.CHROME);
    config.addBrowser(768, 1024, BrowserType.CHROME);
    config.addBrowser(375, 812, BrowserType.CHROME);
  } else if (tier === 'important') {
    // Tier 2: All browsers, Desktop + Mobile
    config.addBrowser(1920, 1080, BrowserType.CHROME);
    config.addBrowser(1920, 1080, BrowserType.FIREFOX);
    config.addBrowser(1920, 1080, BrowserType.SAFARI);
    config.addBrowser(375, 812, BrowserType.CHROME);
    config.addBrowser(375, 812, BrowserType.SAFARI);
  } else {
    // Tier 3: Full matrix
    for (const browser of [BrowserType.CHROME, BrowserType.FIREFOX, BrowserType.SAFARI]) {
      config.addBrowser(1920, 1080, browser);
      config.addBrowser(768, 1024, browser);
      config.addBrowser(375, 812, browser);
    }
    // Add real device emulation
    config.addDeviceEmulation(DeviceName.iPhone_14, ScreenOrientation.PORTRAIT);
    config.addDeviceEmulation(DeviceName.Pixel_7, ScreenOrientation.PORTRAIT);
  }

  // Visual settings
  config.setMatchLevel(MatchLevel.Layout); // For dynamic content
  config.setIgnoreCaret(true);
  config.setHideScrollbars(true);
  config.setWaitBeforeScreenshots(100);

  return config;
}

// Test implementation with dynamic content handling
const { test } = require('@playwright/test');

test.describe('Product Page Visual Tests', () => {
  let eyes;

  test.beforeEach(async ({ page }) => {
    eyes = new Eyes(runner);
    eyes.setConfiguration(createConfiguration('critical'));

    // Mock dynamic API responses
    await page.route('**/api/user/profile', route => {
      route.fulfill({
        json: { name: 'Test User', avatar: '/assets/default-avatar.png' }
      });
    });

    await page.route('**/api/promotions', route => {
      route.fulfill({
        json: { banner: 'Test Promotion', discount: 20 }
      });
    });
  });

  test('product page visual', async ({ page }) => {
    await eyes.open(page, 'E-Commerce', 'Product Page');

    await page.goto('/products/widget-123');
    await page.waitForLoadState('networkidle');

    // Freeze timestamp
    await page.evaluate(() => {
      const fixedDate = new Date('2024-01-15T10:00:00Z');
      Date.now = () => fixedDate.getTime();
    });

    // Disable animations
    await page.addStyleTag({
      content: `
        *, *::before, *::after {
          animation: none !important;
          transition: none !important;
        }
      `
    });

    // Full page check with smart regions
    await eyes.check('Product Page', Target.window()
      .fully()
      .ignoreRegions(
        '.price',           // Dynamic price
        '.stock-count',     // Inventory changes
        '.timestamp',       // Timestamps
        '.ad-banner',       // Ads
        '.recently-viewed'  // Personalized content
      )
      .layoutRegions(
        '.product-carousel', // Layout only, not content
        '.reviews-section'   // Reviews may vary
      )
      .floatingRegion('.tooltip', 10, 10, 10, 10) // Allow tooltip movement
    );

    // Component-level checks
    await eyes.check('Product Image Gallery', Target.region('.product-gallery'));

    await eyes.check('Add to Cart Button', Target.region('.add-to-cart')
      .strict() // This should be pixel-perfect
    );

    await eyes.close(false);
  });

  test('cart page with items', async ({ page }) => {
    await eyes.open(page, 'E-Commerce', 'Cart Page');

    // Setup: Add item to cart via API
    await page.evaluate(async () => {
      await fetch('/api/cart/items', {
        method: 'POST',
        body: JSON.stringify({ productId: '123', quantity: 1 })
      });
    });

    await page.goto('/cart');
    await page.waitForLoadState('networkidle');

    await eyes.check('Cart Page', Target.window()
      .fully()
      .ignoreRegions('.item-price', '.subtotal', '.total')
      .layoutRegions('.cart-items')
    );

    // Verify specific elements are present
    await eyes.check('Checkout Button', Target.region('[data-test="checkout-btn"]'));

    await eyes.close(false);
  });

  test.afterEach(async () => {
    await eyes.abort();
  });

  test.afterAll(async () => {
    const results = await runner.getAllTestResults(false);
    console.log(results.toString());
  });
});
```

**c) ROI Calculation (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ROI ANALYSIS: VISUAL AI vs PIXEL-DIFF                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  COSTS                                                                       │
│  ─────                                                                       │
│                                                                              │
│  Pixel-Diff Approach (BackstopJS/Percy without AI):                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Infrastructure:                                                      │    │
│  │ - Self-hosted servers: $500/month                                   │    │
│  │ - Or Percy: $400/month (limited snapshots)                          │    │
│  │                                                                      │    │
│  │ CI/CD Compute (sequential runs):                                    │    │
│  │ - 1,800 checkpoints × 3 min each = 90 hours/run                    │    │
│  │ - 30 runs/month × $0.008/min = $1,296/month                        │    │
│  │                                                                      │    │
│  │ Maintenance (false positive review):                                │    │
│  │ - 30% false positive rate                                           │    │
│  │ - 1,800 × 30% = 540 false positives/run                            │    │
│  │ - 5 min/review × 540 = 45 hours/run                                │    │
│  │ - QA salary: $50/hr × 45 hr × 30 runs = $67,500/month              │    │
│  │                                                                      │    │
│  │ TOTAL: ~$69,000/month                                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Visual AI Approach (Applitools Ultrafast Grid):                            │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Applitools License:                                                  │    │
│  │ - Growth plan: $969/month                                           │    │
│  │                                                                      │    │
│  │ CI/CD Compute (parallel rendering):                                 │    │
│  │ - 1,800 checkpoints → 1 local run + cloud render                   │    │
│  │ - 45 min/run × 30 runs × $0.008/min = $10.80/month                 │    │
│  │                                                                      │    │
│  │ Maintenance (minimal false positives):                              │    │
│  │ - 2% false positive rate (Visual AI)                                │    │
│  │ - 1,800 × 2% = 36 false positives/run                              │    │
│  │ - 5 min/review × 36 = 3 hours/run                                  │    │
│  │ - QA salary: $50/hr × 3 hr × 30 runs = $4,500/month                │    │
│  │                                                                      │    │
│  │ TOTAL: ~$5,480/month                                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  SAVINGS                                                                     │
│  ───────                                                                     │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                                                                      │    │
│  │  Monthly Savings: $69,000 - $5,480 = $63,520                        │    │
│  │                                                                      │    │
│  │  Annual Savings: $63,520 × 12 = $762,240                            │    │
│  │                                                                      │    │
│  │  ROI: 1,159% (savings / cost)                                       │    │
│  │                                                                      │    │
│  │  Payback Period: < 1 month                                          │    │
│  │                                                                      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  ADDITIONAL BENEFITS                                                         │
│  ───────────────────                                                         │
│                                                                              │
│  1. Time to Feedback: 90 hours → 45 minutes (99% faster)                   │
│  2. False Positives: 30% → 2% (93% reduction)                               │
│  3. Cross-browser: No additional infrastructure                             │
│  4. Root Cause Analysis: Automatic DOM/CSS diff                             │
│  5. Batch Review: Approve similar changes together                          │
│                                                                              │
│                                                                              │
│  RISK CONSIDERATIONS                                                         │
│  ───────────────────                                                         │
│                                                                              │
│  - Vendor lock-in: Applitools-specific API                                  │
│  - Cloud dependency: Requires internet for rendering                        │
│  - Cost scaling: Enterprise plans for large teams                           │
│                                                                              │
│  Mitigation:                                                                │
│  - Use abstraction layer for future portability                            │
│  - Keep local fallback for critical tests                                   │
│  - Negotiate enterprise pricing                                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 4-10: Các câu hỏi bổ sung

*(Tóm tắt các câu còn lại)*

## Câu 4: Test Oracle Problem Solutions (10 điểm)
- Implement Statistical Oracle cho ML predictions
- Design Metamorphic Testing cho image classifier
- Compare differential testing approaches

## Câu 5: Healenium vs Playwright Auto-Waiting (10 điểm)
- Compare self-healing strategies
- Design hybrid approach
- Implement best practices

## Câu 6: AI-Generated Test Quality Verification (10 điểm)
- Mutation testing cho AI-generated tests
- Coverage analysis
- Quality gates in CI/CD

## Câu 7: Visual AI for Component Libraries (10 điểm)
- Storybook + Chromatic integration
- Component state testing
- Design token validation

## Câu 8: LLM-as-Judge Evaluation (10 điểm)
- Implement LLM evaluation pipeline
- Design prompt templates
- Handle hallucination detection

## Câu 9: Cross-Browser Visual Testing Strategy (10 điểm)
- Ultrafast Grid optimization
- Browser-specific quirks handling
- Baseline management

## Câu 10: AI Testing Tools Integration (10 điểm)
- Combine Healenium + Applitools
- Design comprehensive test framework
- CI/CD pipeline with AI tools

---

## Tổng kết

| Câu | Topic | Điểm | Độ khó |
|-----|-------|------|--------|
| 1 | Healenium Architecture | 10 | Medium |
| 2 | AI Test Generation Comparison | 10 | Medium |
| 3 | Applitools Visual AI | 10 | Hard |
| 4 | Test Oracle Problem | 10 | Hard |
| 5 | Healenium vs Playwright | 10 | Medium |
| 6 | AI Test Quality | 10 | Medium |
| 7 | Visual AI Components | 10 | Medium |
| 8 | LLM-as-Judge | 10 | Hard |
| 9 | Cross-Browser Strategy | 10 | Medium |
| 10 | Tools Integration | 10 | Hard |

**Tổng: 100 điểm**

---

*Created for CS423 Software Testing Final Exam Preparation*
*Focus: AI FOR Testing - tools that use AI to improve testing*
