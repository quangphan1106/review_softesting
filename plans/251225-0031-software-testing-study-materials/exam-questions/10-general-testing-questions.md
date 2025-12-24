# Topic 10: General Software Testing - Exam Questions
**CS423 Software Testing | Câu hỏi ôn tập mức vận dụng**

---

## Câu 1: Lựa chọn SDLC Model (10 điểm)

### Tình huống:
Công ty bạn nhận được 3 dự án mới với các đặc điểm khác nhau:

**Dự án A - Hệ thống điều khiển máy bay:**
- Yêu cầu an toàn tối đa (safety-critical)
- Requirements đã được phê duyệt bởi cơ quan hàng không
- Không cho phép thay đổi requirements sau khi ký hợp đồng

**Dự án B - Ứng dụng startup fintech:**
- Thị trường thay đổi liên tục
- Cần release features nhanh để cạnh tranh
- Customer feedback rất quan trọng

**Dự án C - Migration hệ thống legacy:**
- Chuyển đổi từ COBOL sang Java
- Requirements rõ ràng (giữ nguyên chức năng)
- Timeline cố định 18 tháng

### Yêu cầu:
a) Đề xuất SDLC model phù hợp cho từng dự án, giải thích lý do (4 điểm)
b) Vẽ diagram test activities cho dự án A với V-Model (3 điểm)
c) Thiết kế Sprint testing workflow cho dự án B (3 điểm)

### Đáp án mẫu:

**a) Đề xuất SDLC model:**

| Dự án | Model | Lý do |
|-------|-------|-------|
| **A - Máy bay** | V-Model | Safety-critical cần verification ở mỗi phase, requirements frozen, cần traceability rõ ràng từ requirement → test case |
| **B - Fintech** | Agile/Scrum | Cần flexibility, fast delivery, continuous customer feedback, market-driven changes |
| **C - Migration** | Waterfall | Requirements fixed (replicate existing), clear scope, timeline-driven, low uncertainty |

**b) V-Model diagram cho dự án A:**
```
Requirements Analysis ─────────────────────> Acceptance Testing
  - Safety requirements                        - FAA certification tests
  - Functional specs                           - User acceptance tests
         ↓                                              ↑
System Design ────────────────────────────> System Testing
  - Architecture design                        - Full system validation
  - Interface specs                            - Safety compliance tests
         ↓                                              ↑
High-Level Design ─────────────────────> Integration Testing
  - Module specifications                      - Module interaction tests
  - Data flow design                           - Interface testing
         ↓                                              ↑
Low-Level Design ──────────────────────> Unit Testing
  - Detailed algorithms                        - Function-level tests
  - Pseudo code                                - Code coverage 100%
         ↓                                              ↑
         └────────> CODING <────────────────────────────┘
```

**c) Sprint testing workflow cho dự án B:**
```
Sprint N (2 weeks)
┌──────────────────────────────────────────────────────────┐
│ Day 1-2: Sprint Planning                                 │
│   - Review user stories                                  │
│   - Define acceptance criteria                           │
│   - Estimate testing effort                              │
│   - Write test cases for stories                         │
├──────────────────────────────────────────────────────────┤
│ Day 3-8: Development + Continuous Testing                │
│   - Developers write code + unit tests                   │
│   - QA writes automation scripts                         │
│   - Daily: code review + test review                     │
│   - Pair testing sessions                                │
├──────────────────────────────────────────────────────────┤
│ Day 9-10: Integration + Regression                       │
│   - Run full regression suite                            │
│   - Exploratory testing on new features                  │
│   - Bug fixes and retesting                              │
├──────────────────────────────────────────────────────────┤
│ Day 11-12: UAT + Demo Prep                               │
│   - Product owner acceptance                             │
│   - Demo to stakeholders                                 │
│   - Retrospective (improve testing process)              │
└──────────────────────────────────────────────────────────┘
         ↓
    Release to Production
```

---

## Câu 2: Test Levels và Test Pyramid (10 điểm)

### Tình huống:
Team của bạn đang develop một e-commerce platform. Hiện tại test suite có:
- 50 E2E tests (Selenium) - chạy 45 phút
- 30 Integration tests - chạy 5 phút
- 20 Unit tests - chạy 30 giây

CI pipeline bị chậm, tests hay fail không rõ nguyên nhân, khó debug.

### Yêu cầu:
a) Phân tích vấn đề của test suite hiện tại dựa trên Test Pyramid (3 điểm)
b) Đề xuất cải thiện tỉ lệ test levels, giải thích (4 điểm)
c) Thiết kế test strategy cho module "Shopping Cart" với 3 levels (3 điểm)

### Đáp án mẫu:

**a) Phân tích vấn đề - Ice Cream Cone Anti-pattern:**
```
Current (Anti-pattern):           Ideal (Pyramid):

     ████████████                        ▲
     ████████████  50 E2E               /│\
     ████████████  (45 min)            / │ \   5-10 E2E
         ████████                     /──┼──\
         ████████  30 Integration    /   │   \
         ████████  (5 min)          /────┼────\ 30-50 Integration
            ████                   /     │     \
            ████   20 Unit        /──────┼──────\ 200+ Unit
```

**Vấn đề:**
| Issue | Nguyên nhân | Hậu quả |
|-------|-------------|---------|
| CI chậm | Quá nhiều E2E tests (45 min) | Developer feedback loop dài |
| Fail không rõ | E2E tests có nhiều dependencies | Khó xác định root cause |
| Khó debug | Tests không isolated | Không biết component nào lỗi |
| Flaky tests | E2E phụ thuộc UI, network, timing | False negatives |

**b) Đề xuất cải thiện:**

| Level | Current | Target | Lý do |
|-------|---------|--------|-------|
| Unit | 20 (20%) | 200 (70%) | Fast feedback, isolated, cheap to maintain |
| Integration | 30 (30%) | 70 (25%) | Test component interactions |
| E2E | 50 (50%) | 15 (5%) | Chỉ test critical user journeys |

**Benefits:**
- CI time: 45min → ~5min
- Faster debugging (unit tests point to exact function)
- Less flaky (unit tests deterministic)
- Cheaper maintenance

**c) Test strategy cho Shopping Cart:**

```python
# LEVEL 1: Unit Tests (15 tests)
class TestCartCalculations:
    def test_add_item_to_empty_cart(self):
        cart = ShoppingCart()
        cart.add_item(Product("iPhone", 999), quantity=1)
        assert cart.total == 999
        assert cart.item_count == 1

    def test_apply_percentage_discount(self):
        cart = ShoppingCart()
        cart.add_item(Product("Laptop", 1000), quantity=1)
        cart.apply_discount(PercentDiscount(10))  # 10% off
        assert cart.total == 900

    def test_remove_item_updates_total(self):
        cart = ShoppingCart()
        cart.add_item(Product("Phone", 500), quantity=2)
        cart.remove_item("Phone", quantity=1)
        assert cart.total == 500

    def test_free_shipping_over_threshold(self):
        cart = ShoppingCart()
        cart.add_item(Product("TV", 600), quantity=1)
        assert cart.shipping_fee == 0  # Free over $500

    def test_quantity_validation_reject_negative(self):
        cart = ShoppingCart()
        with pytest.raises(ValueError):
            cart.add_item(Product("X", 10), quantity=-1)
```

```python
# LEVEL 2: Integration Tests (5 tests)
class TestCartWithDatabase:
    def test_cart_persists_across_sessions(self, db):
        # Add items and save
        cart = ShoppingCart(user_id=123)
        cart.add_item(Product("iPhone", 999), quantity=1)
        cart.save(db)

        # Reload from DB
        loaded_cart = ShoppingCart.load(db, user_id=123)
        assert loaded_cart.item_count == 1
        assert loaded_cart.total == 999

    def test_cart_inventory_sync(self, db, inventory_service):
        cart = ShoppingCart(user_id=123)
        cart.add_item(Product("Limited", 100), quantity=5)

        # Should check inventory
        inventory_service.reserve(cart)
        assert inventory_service.get_reserved("Limited") == 5
```

```python
# LEVEL 3: E2E Tests (2 tests - critical paths only)
class TestCartE2E:
    def test_complete_checkout_flow(self, browser):
        # Login
        browser.goto("/login")
        browser.fill("#email", "user@test.com")
        browser.fill("#password", "password")
        browser.click("#login-btn")

        # Add to cart
        browser.goto("/product/123")
        browser.click("#add-to-cart")
        assert browser.text("#cart-count") == "1"

        # Checkout
        browser.goto("/cart")
        browser.click("#checkout")
        browser.fill("#card-number", "4242424242424242")
        browser.click("#pay")

        assert browser.url == "/order-confirmation"
        assert "Thank you" in browser.text("h1")
```

---

## Câu 3: Smoke vs Sanity Testing (10 điểm)

### Tình huống:
Bạn là QA Lead cho dự án banking app. Hôm nay có 2 builds cần test:

**Build A (v2.5.0):** New major release với features mới
- Money transfer module hoàn toàn mới
- UI redesign cho dashboard
- 50+ file changes

**Build B (v2.4.3):** Hotfix cho bug cụ thể
- Fix bug: "Transfer fails when amount has decimal"
- Chỉ 2 file changes trong transfer module

### Yêu cầu:
a) Xác định loại test phù hợp cho mỗi build, giải thích (3 điểm)
b) Thiết kế Smoke Test checklist cho Build A (4 điểm)
c) Thiết kế Sanity Test checklist cho Build B (3 điểm)

### Đáp án mẫu:

**a) Loại test phù hợp:**

| Build | Test Type | Lý do |
|-------|-----------|-------|
| **A (v2.5.0)** | Smoke Test | Major release, many changes, need to verify build stability across ALL critical features before deep testing |
| **B (v2.4.3)** | Sanity Test | Hotfix for specific bug, only need to verify the fix + related functionality, narrow scope |

**b) Smoke Test Checklist - Build A (v2.5.0):**

```markdown
# SMOKE TEST CHECKLIST - Banking App v2.5.0
# Objective: Verify build is stable enough for detailed testing
# Time limit: 30 minutes
# Pass criteria: ALL items must pass

## Authentication Module
□ TC-SM-001: Login with valid credentials → Dashboard loads
□ TC-SM-002: Logout → Returns to login page, session cleared
□ TC-SM-003: Password reset link sends email

## Dashboard (NEW UI)
□ TC-SM-004: Dashboard loads within 3 seconds
□ TC-SM-005: Account balance displays correctly
□ TC-SM-006: Recent transactions list shows
□ TC-SM-007: Navigation menu accessible

## Money Transfer (NEW MODULE)
□ TC-SM-008: Transfer form loads without error
□ TC-SM-009: Can select recipient from saved list
□ TC-SM-010: Can enter amount and submit
□ TC-SM-011: Transfer confirmation displays

## Bill Payment
□ TC-SM-012: Bill payment page loads
□ TC-SM-013: Can select biller from list
□ TC-SM-014: Payment submission works

## Account Management
□ TC-SM-015: View account details
□ TC-SM-016: Download statement (PDF)

## API Health Check
□ TC-SM-017: Core APIs return 200 OK
□ TC-SM-018: No 500 errors in first 5 minutes

## RESULT:
- Total: 18 tests
- Passed: ___
- Failed: ___
- Build Status: ACCEPT / REJECT
- Tester: ___________ Date: ___________
```

**c) Sanity Test Checklist - Build B (v2.4.3):**

```markdown
# SANITY TEST CHECKLIST - Banking App v2.4.3
# Objective: Verify bug fix for decimal transfer amount
# Time limit: 15 minutes
# Pass criteria: ALL items must pass

## Bug Fix Verification (Primary)
□ TC-SN-001: Transfer $100.50 → Success
□ TC-SN-002: Transfer $0.01 → Success (minimum)
□ TC-SN-003: Transfer $99999.99 → Success
□ TC-SN-004: Transfer $100.123 → Rounds to $100.12

## Related Functionality (Impact Area)
□ TC-SN-005: Transfer whole number $100 → Still works
□ TC-SN-006: Transfer history shows decimal correctly
□ TC-SN-007: Transaction receipt shows decimal
□ TC-SN-008: Account balance updates with decimal precision

## Boundary Cases
□ TC-SN-009: Transfer $0.00 → Rejected with error message
□ TC-SN-010: Very large decimal $999999.99 → Success

## RESULT:
- Total: 10 tests
- Passed: ___
- Failed: ___
- Sanity Status: PASS / FAIL
- Note: If FAIL, return to dev with specific failure
- Tester: ___________ Date: ___________
```

---

## Câu 4: Test Plan Creation (10 điểm)

### Tình huống:
Bạn được giao viết Test Plan cho module "User Registration" của hệ thống e-learning. Module bao gồm:
- Registration form (name, email, password, role selection)
- Email verification
- Profile setup wizard

Timeline: 2 weeks for testing
Team: 2 QA engineers, 1 automation engineer

### Yêu cầu:
a) Viết section "Features to be Tested" và "Features NOT to be Tested" (3 điểm)
b) Viết section "Approach" với test types và automation strategy (4 điểm)
c) Viết section "Pass/Fail Criteria" và "Risks" (3 điểm)

### Đáp án mẫu:

**a) Features to be Tested / NOT to be Tested:**

```markdown
## 4. FEATURES TO BE TESTED

### 4.1 Registration Form
- FTR-001: Field validations (name, email format, password strength)
- FTR-002: Duplicate email detection
- FTR-003: Role selection (Student, Instructor, Admin)
- FTR-004: Terms & Conditions acceptance
- FTR-005: CAPTCHA verification
- FTR-006: Error message display
- FTR-007: Form submission and data persistence

### 4.2 Email Verification
- FTR-008: Verification email sent within 30 seconds
- FTR-009: Verification link validity (24 hours expiry)
- FTR-010: Resend verification email
- FTR-011: Account activation after verification
- FTR-012: Expired link handling

### 4.3 Profile Setup Wizard
- FTR-013: Step-by-step wizard navigation
- FTR-014: Avatar upload (JPG, PNG, max 2MB)
- FTR-015: Bio/description input
- FTR-016: Interest tags selection
- FTR-017: Skip option functionality
- FTR-018: Progress saving between steps

## 5. FEATURES NOT TO BE TESTED

| Feature | Reason |
|---------|--------|
| Social login (Google, Facebook) | Not in current sprint scope |
| Admin user management | Covered by separate Admin module tests |
| Payment/subscription | Not part of registration flow |
| Password recovery | Separate module, tested independently |
| Accessibility (WCAG) | Dedicated accessibility audit planned |
```

**b) Approach Section:**

```markdown
## 6. APPROACH

### 6.1 Test Levels
| Level | Scope | Owner |
|-------|-------|-------|
| Unit Testing | Form validation functions, API handlers | Developers |
| Integration Testing | Form → API → Database flow | QA + Auto |
| System Testing | End-to-end registration journey | QA Team |
| UAT | Business requirements validation | Product Owner |

### 6.2 Test Types
| Type | Description | Allocation |
|------|-------------|------------|
| Functional | All features in section 4 | 60% |
| Usability | Form UX, error clarity, wizard flow | 15% |
| Security | SQL injection, XSS, password security | 15% |
| Performance | Form submission under load | 5% |
| Compatibility | Chrome, Firefox, Safari, Edge, Mobile | 5% |

### 6.3 Automation Strategy
```
┌─────────────────────────────────────────────────────┐
│            AUTOMATION BREAKDOWN                     │
├─────────────────────────────────────────────────────┤
│ AUTOMATED (70%):                                    │
│ • Form field validations (Playwright)              │
│ • API tests for registration endpoints (pytest)    │
│ • Database verification tests                       │
│ • Regression suite for CI/CD                       │
├─────────────────────────────────────────────────────┤
│ MANUAL (30%):                                       │
│ • Email verification flow (external dependency)    │
│ • Usability assessment                             │
│ • Exploratory testing                              │
│ • Visual/UI validation                             │
└─────────────────────────────────────────────────────┘

Tools:
- Playwright: E2E browser automation
- pytest: API and unit tests
- Allure: Test reporting
- JIRA: Defect tracking
```

### 6.4 Test Data Strategy
- Use Faker library for synthetic user data
- Maintain test data in fixtures (reset before each run)
- Separate test email domain: @testmail.elearning.local
```

**c) Pass/Fail Criteria và Risks:**

```markdown
## 7. PASS/FAIL CRITERIA

### Entry Criteria (Start Testing)
□ Development complete for all features in scope
□ Code deployed to QA environment
□ Smoke test passed
□ Test data prepared
□ Test environment stable

### Exit Criteria (End Testing)
□ 100% test cases executed
□ Test case pass rate ≥ 95%
□ Zero Critical bugs open
□ Zero High bugs open (or accepted risk)
□ All Medium bugs documented with workarounds
□ Code coverage ≥ 80%
□ UAT sign-off received

### Suspension Criteria
- Critical blocker affecting >50% of test cases
- Test environment down >4 hours
- Build deployment failure

### Resumption Criteria
- Blocker resolved and verified
- Environment restored and smoke tested
- New stable build deployed

---

## 13. RISKS AND MITIGATION

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Email server delay | Medium | High | Implement retry mechanism; use mock email for automation |
| Resource unavailable | Low | High | Cross-train team members; document all test cases |
| Unstable QA environment | Medium | Medium | Dedicated environment; daily health checks |
| Requirements change | Low | Medium | Agile approach; flexible test cases |
| Third-party CAPTCHA issues | Medium | Medium | Mock CAPTCHA in test env; manual backup tests |
| Tight timeline | High | Medium | Prioritize critical paths; risk-based testing |

### Contingency Plan
If timeline at risk:
1. Focus on Critical/High priority tests only
2. Defer Low priority tests to regression
3. Request 2-day extension with justification
4. Reduce browser compatibility scope
```

---

## Câu 5: Regression Test Selection (10 điểm)

### Tình huống:
Hệ thống e-commerce có 500 test cases trong regression suite. Team vừa thực hiện 2 changes:

**Change 1:** Refactor Payment Gateway integration
- Modified: `PaymentService.java`, `StripeAdapter.java`, `PaymentController.java`

**Change 2:** Fix bug in Product Search
- Modified: `SearchService.java`, `SearchController.java`

Full regression chạy 8 tiếng. Sprint chỉ có 2 tiếng cho regression.

### Yêu cầu:
a) Đề xuất chiến lược Regression Test Selection, giải thích (3 điểm)
b) Phân loại 500 test cases theo priority (với ví dụ), chọn subset để chạy trong 2 tiếng (4 điểm)
c) Thiết kế Impact Analysis matrix cho 2 changes trên (3 điểm)

### Đáp án mẫu:

**a) Chiến lược Regression Test Selection:**

```
┌─────────────────────────────────────────────────────────────┐
│           SELECTIVE REGRESSION STRATEGY                     │
├─────────────────────────────────────────────────────────────┤
│ 1. IMPACT ANALYSIS                                          │
│    - Identify modified files                                │
│    - Trace dependencies (code → tests)                      │
│    - Map affected modules                                   │
│                                                             │
│ 2. RISK-BASED SELECTION                                     │
│    - Prioritize by business impact                          │
│    - Weight by defect history                               │
│    - Consider change complexity                             │
│                                                             │
│ 3. TIME-BOXED EXECUTION                                     │
│    - Run high-priority first                                │
│    - Stop when time limit reached                           │
│    - Report coverage percentage                             │
└─────────────────────────────────────────────────────────────┘
```

**Lý do chọn Selective:**
- Full regression (8h) vượt quá time available (2h)
- Changes tập trung vào 2 modules cụ thể
- Không cần test toàn bộ 500 cases
- Risk-based approach optimize coverage/time

**b) Test Priority Classification:**

```
┌─────────────────────────────────────────────────────────────┐
│ PRIORITY CLASSIFICATION (500 Test Cases)                   │
├──────────────┬─────────────┬────────────────────────────────┤
│ Priority     │ Count       │ Examples                       │
├──────────────┼─────────────┼────────────────────────────────┤
│ P0-Critical  │ 50 (10%)    │ • Payment processing           │
│ Run: Always  │ ~30 min     │ • Order completion             │
│              │             │ • User authentication          │
│              │             │ • Checkout flow                │
├──────────────┼─────────────┼────────────────────────────────┤
│ P1-High      │ 100 (20%)   │ • Product search               │
│ Run: Change  │ ~60 min     │ • Cart operations              │
│ impact       │             │ • Inventory sync               │
│              │             │ • Price calculations           │
├──────────────┼─────────────┼────────────────────────────────┤
│ P2-Medium    │ 200 (40%)   │ • Product reviews              │
│ Run: Weekly  │ ~4 hours    │ • Wishlist                     │
│              │             │ • User profile                 │
│              │             │ • Email notifications          │
├──────────────┼─────────────┼────────────────────────────────┤
│ P3-Low       │ 150 (30%)   │ • UI cosmetic                  │
│ Run: Release │ ~3 hours    │ • Help pages                   │
│              │             │ • Footer links                 │
│              │             │ • Admin reports                │
└──────────────┴─────────────┴────────────────────────────────┘
```

**2-Hour Regression Selection:**

| Category | Tests | Time | Justification |
|----------|-------|------|---------------|
| P0 Critical (all) | 50 | 30 min | Always run - core business |
| P1 Payment-related | 40 | 25 min | Change 1 impacted |
| P1 Search-related | 25 | 15 min | Change 2 impacted |
| P1 Integration points | 20 | 15 min | Cross-module dependencies |
| Buffer for failures | - | 35 min | Rerun failed, debug |
| **TOTAL** | **135** | **~2 hours** | **27% coverage, 100% risk** |

**c) Impact Analysis Matrix:**

```
┌────────────────────────────────────────────────────────────────────────────┐
│                    IMPACT ANALYSIS MATRIX                                  │
├────────────────────────────────────────────────────────────────────────────┤
│ CHANGE 1: Payment Gateway Refactor                                         │
├─────────────────────┬──────────────────────────────────────────────────────┤
│ Modified File       │ Impacted Tests                                       │
├─────────────────────┼──────────────────────────────────────────────────────┤
│ PaymentService.java │ • TC-PAY-001 to TC-PAY-025 (payment processing)     │
│                     │ • TC-ORD-010 to TC-ORD-015 (order completion)       │
│                     │ • TC-REF-001 to TC-REF-010 (refunds)                │
├─────────────────────┼──────────────────────────────────────────────────────┤
│ StripeAdapter.java  │ • TC-PAY-030 to TC-PAY-040 (Stripe-specific)        │
│                     │ • TC-PAY-050 to TC-PAY-055 (card validation)        │
├─────────────────────┼──────────────────────────────────────────────────────┤
│ PaymentController   │ • TC-API-PAY-001 to TC-API-PAY-020 (payment APIs)   │
│ .java               │ • TC-INT-001 to TC-INT-005 (checkout integration)   │
├─────────────────────┴──────────────────────────────────────────────────────┤
│ TOTAL IMPACTED: ~70 tests                                                  │
│ DOWNSTREAM: Cart, Order, Email notification tests (30 more)               │
└────────────────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────────────────────┐
│ CHANGE 2: Product Search Bug Fix                                           │
├─────────────────────┬──────────────────────────────────────────────────────┤
│ Modified File       │ Impacted Tests                                       │
├─────────────────────┼──────────────────────────────────────────────────────┤
│ SearchService.java  │ • TC-SRC-001 to TC-SRC-030 (search functionality)   │
│                     │ • TC-FLT-001 to TC-FLT-015 (filtering)              │
│                     │ • TC-SRT-001 to TC-SRT-010 (sorting)                │
├─────────────────────┼──────────────────────────────────────────────────────┤
│ SearchController    │ • TC-API-SRC-001 to TC-API-SRC-015 (search APIs)    │
│ .java               │ • TC-AUTO-001 to TC-AUTO-005 (autocomplete)         │
├─────────────────────┴──────────────────────────────────────────────────────┤
│ TOTAL IMPACTED: ~75 tests                                                  │
│ DOWNSTREAM: Product listing, Category pages (20 more)                      │
└────────────────────────────────────────────────────────────────────────────┘

COMBINED SELECTION (No Overlap):
┌────────────────────────────────────────────────────────────┐
│ Change 1 Impact:  70 + 30 downstream = 100 tests          │
│ Change 2 Impact:  75 + 20 downstream = 95 tests           │
│ Core Critical:    50 tests (always run)                   │
│ Overlap:          ~30 tests (run once)                    │
├────────────────────────────────────────────────────────────┤
│ FINAL SELECTION:  ~215 tests                              │
│ Optimized for 2h: ~135 tests (prioritized by risk)        │
└────────────────────────────────────────────────────────────┘
```

---

## Câu 6: Exploratory Testing Session (10 điểm)

### Tình huống:
Team vừa release tính năng "Live Chat Support" cho website. Không có requirements document chi tiết, chỉ có user stories:
- "As a customer, I can chat with support agent in real-time"
- "As a customer, I can attach images in chat"
- "As an agent, I can handle multiple chats simultaneously"

Bạn có 90 phút để exploratory testing.

### Yêu cầu:
a) Viết 3 Session Charters cho 3 sessions 30 phút (3 điểm)
b) Thực hiện Session 1, viết Exploration Notes (4 điểm)
c) Tổng hợp Session Report với bugs found, risks, và test ideas (3 điểm)

### Đáp án mẫu:

**a) Session Charters:**

```markdown
# SESSION 1: Core Chat Functionality
┌─────────────────────────────────────────────────────────────┐
│ Charter: Explore CHAT INITIATION and MESSAGE FLOW          │
│          with various message types                         │
│          to discover communication bugs and edge cases      │
├─────────────────────────────────────────────────────────────┤
│ Duration: 30 minutes                                        │
│ Focus Areas:                                                │
│ • Chat widget visibility and accessibility                  │
│ • Message sending/receiving reliability                     │
│ • Special characters, emojis, long messages                │
│ • Connection stability (network interruption)               │
└─────────────────────────────────────────────────────────────┘

# SESSION 2: File Attachment Feature
┌─────────────────────────────────────────────────────────────┐
│ Charter: Explore IMAGE ATTACHMENT functionality             │
│          with various file types and sizes                  │
│          to discover upload/display issues                  │
├─────────────────────────────────────────────────────────────┤
│ Duration: 30 minutes                                        │
│ Focus Areas:                                                │
│ • Supported image formats (JPG, PNG, GIF, WebP)            │
│ • File size limits and error handling                       │
│ • Multiple file upload                                      │
│ • Image preview and download                                │
│ • Non-image files (PDF, DOC) - should reject               │
└─────────────────────────────────────────────────────────────┘

# SESSION 3: Multi-Chat Agent Experience
┌─────────────────────────────────────────────────────────────┐
│ Charter: Explore AGENT DASHBOARD handling multiple chats    │
│          with concurrent conversations                      │
│          to discover scalability and UX issues              │
├─────────────────────────────────────────────────────────────┤
│ Duration: 30 minutes                                        │
│ Focus Areas:                                                │
│ • Switching between active chats                            │
│ • Notification for new messages                             │
│ • Chat queue management                                     │
│ • Performance with 5+ simultaneous chats                    │
│ • Chat history persistence                                  │
└─────────────────────────────────────────────────────────────┘
```

**b) Session 1 Exploration Notes:**

```markdown
# EXPLORATION NOTES - Session 1
# Tester: [Name]
# Date: 2024-12-25
# Start: 10:00 AM | End: 10:30 AM

## Test Path & Observations

10:00 - Started on homepage, looking for chat widget
       ✓ Found floating button bottom-right
       ✓ Clear "Chat with us" label
       ? Widget loads after 3 seconds - intentional delay?

10:03 - Clicked to open chat
       ✓ Smooth animation
       ✓ Welcome message appears
       ✗ BUG: No loading indicator while connecting

10:05 - Sent simple message "Hello"
       ✓ Message appears in chat
       ✓ Timestamp shown
       ✗ BUG: Timestamp shows UTC, not local time

10:08 - Tested special characters: "Xin chào! 你好 🎉"
       ✓ Vietnamese characters: OK
       ✓ Chinese characters: OK
       ✓ Emoji: OK

10:12 - Tested very long message (500 chars)
       ✓ Message sent successfully
       ✓ Word wrap works
       ✗ BUG: No character limit warning (sent 5000 chars!)

10:16 - Tested empty message
       ✓ Send button disabled - correct behavior

10:18 - Tested HTML injection: "<script>alert('xss')</script>"
       ✓ GOOD: HTML is escaped, displays as text

10:20 - Disconnected WiFi mid-chat
       ✗ BUG: No "connection lost" message
       ✗ BUG: Messages typed offline are lost
       ✓ Reconnects automatically when WiFi restored

10:25 - Rapid message sending (10 messages in 5 seconds)
       ✗ BUG: Rate limiting not implemented
       ? Question: Can this be abused for spam?

10:28 - Closed browser tab, reopened
       ✓ Chat history preserved
       ✓ Session continues

## Session Metrics
- Time on Charter: 25 min (83%)
- Time on Bug Investigation: 5 min (17%)
- Bugs Found: 5
- Questions: 2
- Test Ideas Generated: 4
```

**c) Session Report:**

```markdown
# EXPLORATORY TEST SESSION REPORT
# Feature: Live Chat Support
# Total Duration: 90 minutes (3 sessions)
# Tester: [Name]

## SUMMARY
┌─────────────────────────────────────────────────────────────┐
│ Sessions Completed: 3/3                                     │
│ Bugs Found: 12 (3 Critical, 4 High, 5 Medium)              │
│ Test Ideas Generated: 15                                    │
│ Risks Identified: 4                                         │
│ Coverage: Core functionality + edge cases                   │
└─────────────────────────────────────────────────────────────┘

## BUGS FOUND

### Critical (3)
| ID | Description | Steps | Impact |
|----|-------------|-------|--------|
| BUG-001 | Messages lost on network disconnect | Disconnect WiFi while typing, reconnect | Customer loses message, frustration |
| BUG-002 | No file type validation | Upload .exe file as "image" | Security risk - malware upload |
| BUG-003 | Agent sees wrong chat when switching fast | Click 3 chats rapidly | Agent responds to wrong customer |

### High (4)
| ID | Description |
|----|-------------|
| BUG-004 | No rate limiting on messages (spam possible) |
| BUG-005 | 50MB image upload crashes browser tab |
| BUG-006 | Timestamp shows UTC instead of local time |
| BUG-007 | No unread message indicator for agent |

### Medium (5)
| ID | Description |
|----|-------------|
| BUG-008 | No character limit warning (accepts 10,000 chars) |
| BUG-009 | No loading indicator during connection |
| BUG-010 | GIF animations don't play in preview |
| BUG-011 | Cannot download received images on mobile |
| BUG-012 | Chat history limited to 50 messages |

## RISKS IDENTIFIED

| Risk | Severity | Recommendation |
|------|----------|----------------|
| Security: No file validation | Critical | Block non-image uploads server-side |
| Performance: Large file handling | High | Implement client-side size check (5MB max) |
| UX: Offline message loss | High | Implement message queue with retry |
| Scalability: No rate limiting | Medium | Add rate limit (10 msgs/min) |

## TEST IDEAS FOR FUTURE

1. Load test: 100 concurrent chat sessions
2. Mobile browser testing (Safari iOS, Chrome Android)
3. Chat persistence across login/logout
4. Agent typing indicator functionality
5. Chat transcript email feature
6. Accessibility testing (screen reader)
7. Internationalization (RTL languages)
8. Chat bot handoff to human agent
9. File sharing from agent to customer
10. Chat satisfaction rating
11. Browser notification permissions
12. Chat widget on different pages
13. Multiple tabs open same chat
14. Agent offline/online status
15. Chat session timeout behavior

## RECOMMENDATION
⚠️ DO NOT RELEASE until Critical bugs (BUG-001, 002, 003) are fixed.
High bugs should be fixed before GA release.
```

---

## Câu 7: Defect Life Cycle (10 điểm)

### Tình huống:
Bạn là QA Engineer, phát hiện bug sau khi test:
- Module: Shopping Cart
- Bug: Khi add 2 sản phẩm cùng lúc, quantity hiện sai (hiện 1 thay vì 2)
- Bạn cần report và track bug này qua defect life cycle

### Yêu cầu:
a) Viết Defect Report chi tiết theo template chuẩn (4 điểm)
b) Vẽ defect life cycle diagram cho bug này với các transitions possible (3 điểm)
c) Mô tả actions cần thực hiện ở mỗi trạng thái (3 điểm)

### Đáp án mẫu:

**a) Defect Report:**

```markdown
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DEFECT REPORT                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ Defect ID:      BUG-CART-2024-0156                                         │
│ Title:          Cart quantity incorrect when adding duplicate items         │
│                 simultaneously                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ Project:        E-Commerce Platform v3.0                                   │
│ Module:         Shopping Cart                                               │
│ Component:      CartService.addItem()                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ Severity:       HIGH                                                        │
│ Priority:       P1 - Critical Business Impact                               │
│ Type:           Functional Bug                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ Status:         NEW                                                         │
│ Assigned To:    [Unassigned]                                               │
│ Reported By:    qa.engineer@company.com                                    │
│ Report Date:    2024-12-25                                                  │
│ Found In:       Build 3.0.15-beta                                          │
│ Environment:    QA - Ubuntu 22.04, Chrome 120, PostgreSQL 15              │
├─────────────────────────────────────────────────────────────────────────────┤
│ DESCRIPTION:                                                                │
│ When a user rapidly clicks "Add to Cart" button twice on the same product, │
│ the cart shows quantity as 1 instead of expected 2. This appears to be a   │
│ race condition where the second request overwrites the first.              │
├─────────────────────────────────────────────────────────────────────────────┤
│ STEPS TO REPRODUCE:                                                         │
│ 1. Navigate to product page: /product/SKU-12345                            │
│ 2. Open browser DevTools → Network tab (to observe requests)               │
│ 3. Quickly double-click "Add to Cart" button (within 500ms)                │
│ 4. Observe cart icon in header                                             │
│                                                                             │
│ EXPECTED RESULT:                                                            │
│ • Cart badge shows "2"                                                     │
│ • Cart contains: Product SKU-12345 × 2                                     │
│ • Subtotal = unit_price × 2                                                │
│                                                                             │
│ ACTUAL RESULT:                                                              │
│ • Cart badge shows "1"                                                     │
│ • Cart contains: Product SKU-12345 × 1                                     │
│ • Subtotal = unit_price × 1                                                │
│ • Network shows both requests return 200 OK                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ ADDITIONAL INFO:                                                            │
│ • Reproducibility: 100% (10/10 attempts)                                   │
│ • Workaround: Wait 1 second between clicks                                 │
│ • Suspect: Race condition in cart update logic                             │
│ • Related: DB transaction isolation level may be factor                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ ATTACHMENTS:                                                                │
│ 1. screenshot-cart-wrong-qty.png                                           │
│ 2. network-trace.har                                                       │
│ 3. console-log.txt                                                         │
│ 4. video-reproduction.mp4                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ BUSINESS IMPACT:                                                            │
│ • Revenue loss: Customers charged for 1 item instead of 2                  │
│ • Customer trust: Wrong quantities shipped                                 │
│ • Support load: Complaints about incorrect orders                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Defect Life Cycle Diagram:**

```
                                    ┌─────────────────┐
                                    │    DUPLICATE    │
                                    │   (Closed as    │
                                    │   duplicate of  │
                               ┌───>│   BUG-XXX)      │
                               │    └─────────────────┘
                               │
     ┌───────┐            ┌────┴─────┐           ┌──────────┐
     │  NEW  │───────────>│ ASSIGNED │──────────>│   OPEN   │
     │       │  Triaged   │          │  Dev      │  (In     │
     │       │  by Lead   │  To Dev  │  starts   │  Progress)│
     └───────┘            └────┬─────┘  work     └────┬─────┘
                               │                      │
                               │                      │ Dev
                               ▼                      │ commits fix
                         ┌──────────┐                 ▼
                         │ REJECTED │           ┌──────────┐
                         │(Not a bug│           │  FIXED   │
                         │ or Won't │           │(Ready for│
                         │ fix)     │           │ QA)      │
                         └──────────┘           └────┬─────┘
                               ▲                      │
                               │                      │ QA
                               │                      │ verifies
              ┌────────────────┴──────────────┐       │
              │                               │       ▼
        ┌─────┴──────┐                  ┌─────┴────────────┐
        │  REOPENED  │<─────────────────│    VERIFIED      │
        │ (Fix didn't│    Verify failed │  (Fix confirmed) │
        │  work)     │                  └────────┬─────────┘
        └────────────┘                           │
              │                                  │ QA closes
              │ Re-assigned to dev               ▼
              └──────────────────────>    ┌──────────┐
                                          │  CLOSED  │
                                          │(Complete)│
                                          └──────────┘

DEFERRED PATH:
     ASSIGNED ──────> DEFERRED ──────> (Reopened in future release)
               "Will fix
                later"
```

**c) Actions ở mỗi trạng thái:**

| Status | Người thực hiện | Actions |
|--------|----------------|---------|
| **NEW** | QA Engineer | - Submit defect report đầy đủ<br>- Attach screenshots/logs<br>- Set severity/priority<br>- Notify team lead |
| **ASSIGNED** | Test Lead/PM | - Review defect validity<br>- Check for duplicates<br>- Assign to appropriate developer<br>- Set target fix version |
| **DUPLICATE** | Test Lead | - Link to original defect<br>- Close with reference<br>- Notify reporter |
| **REJECTED** | Developer/PM | - Provide rejection reason<br>- Mark as "Not a bug" hoặc "Won't fix"<br>- Notify QA for agreement |
| **OPEN** | Developer | - Analyze root cause<br>- Write unit tests for bug<br>- Implement fix<br>- Code review<br>- Update status when done |
| **FIXED** | Developer | - Commit fix với defect ID<br>- Update fix notes<br>- Request QA verification<br>- Specify build version |
| **VERIFIED** | QA Engineer | - Retest với same steps<br>- Verify fix works<br>- Check no regression<br>- Update test case if needed |
| **REOPENED** | QA Engineer | - Document why fix failed<br>- Provide new evidence<br>- Re-assign to developer |
| **CLOSED** | QA Engineer | - Confirm fix in release<br>- Update regression suite<br>- Document lessons learned |
| **DEFERRED** | PM/Lead | - Document defer reason<br>- Set target future release<br>- Add to backlog<br>- Communicate to stakeholders |

---

## Câu 8: Test Metrics Analysis (10 điểm)

### Tình huống:
Sprint 5 vừa kết thúc. Bạn có data sau:

| Metric | Value |
|--------|-------|
| Test Cases Planned | 200 |
| Test Cases Executed | 180 |
| Test Cases Passed | 150 |
| Test Cases Failed | 25 |
| Test Cases Blocked | 5 |
| Defects Found | 35 |
| Defects Fixed | 28 |
| Defects Open | 7 |
| Lines of Code | 15,000 |
| Code Coverage | 72% |

### Yêu cầu:
a) Tính các metrics: Test Execution Rate, Pass Rate, Defect Density, DRE (4 điểm)
b) Phân tích kết quả, xác định vấn đề và đề xuất cải thiện (3 điểm)
c) Tạo Test Summary Dashboard cho Sprint Report (3 điểm)

### Đáp án mẫu:

**a) Tính các metrics:**

```markdown
## TEST METRICS CALCULATIONS

### 1. Test Execution Rate
Formula: (Executed / Planned) × 100%
= (180 / 200) × 100%
= 90%

Interpretation: 10% tests not executed (blocked or skipped)

### 2. Test Case Pass Rate
Formula: (Passed / Executed) × 100%
= (150 / 180) × 100%
= 83.33%

Interpretation: ~17% failure rate - concerning

### 3. Defect Density
Formula: Defects / KLOC (thousands of lines of code)
= 35 / 15
= 2.33 defects per KLOC

Industry benchmark: 1-25 defects/KLOC (varies by domain)
Our result: Within acceptable range

### 4. Defect Removal Efficiency (DRE)
Formula: (Defects found in testing / Total defects) × 100%

Assumption: If 7 open bugs escape to production later
DRE = (35 / (35 + estimated escapes)) × 100%

If we assume 0 escapes (best case):
DRE = 35/35 × 100% = 100%

Realistic estimate (industry avg 10% escape):
If 4 bugs escape: DRE = 35/(35+4) × 100% = 89.7%

### 5. Additional Metrics

# Defect Fix Rate
= (Fixed / Found) × 100%
= (28 / 35) × 100%
= 80%

# Blocked Test Rate
= (Blocked / Planned) × 100%
= (5 / 200) × 100%
= 2.5%

# Test Effectiveness
= Defects found / Test cases executed
= 35 / 180
= 0.194 (1 defect per ~5 tests)
```

**b) Phân tích và đề xuất:**

```markdown
## ANALYSIS & RECOMMENDATIONS

### Problems Identified

| Problem | Evidence | Severity |
|---------|----------|----------|
| Low Pass Rate | 83.33% (target: >95%) | HIGH |
| Incomplete Execution | 10% not executed | MEDIUM |
| Open Defects | 7 bugs still open at sprint end | HIGH |
| Low Code Coverage | 72% (target: 80%) | MEDIUM |

### Root Cause Analysis

1. **Low Pass Rate (83.33%)**
   - 25 failed tests indicate significant quality issues
   - Possible causes:
     - Requirements unclear
     - Code quality issues
     - Insufficient dev unit testing

2. **Blocked Tests (5)**
   - Environment issues?
   - Dependency on other modules?
   - Data setup problems?

3. **Open Defects (7)**
   - Dev capacity issue
   - Late bug discovery
   - Complex fixes needed

### Recommendations

┌─────────────────────────────────────────────────────────────────┐
│ IMMEDIATE ACTIONS (Sprint 6)                                    │
├─────────────────────────────────────────────────────────────────┤
│ 1. Address 7 open defects first before new features            │
│ 2. Investigate blocked tests - remove blockers                 │
│ 3. Increase code coverage to 80% minimum                       │
│ 4. Add unit test requirement to Definition of Done             │
├─────────────────────────────────────────────────────────────────┤
│ PROCESS IMPROVEMENTS                                            │
├─────────────────────────────────────────────────────────────────┤
│ 1. Earlier testing involvement (shift-left)                    │
│ 2. Daily bug triage to address issues faster                   │
│ 3. Pair programming for complex features                       │
│ 4. Pre-sprint environment validation                           │
├─────────────────────────────────────────────────────────────────┤
│ TARGETS FOR SPRINT 6                                            │
├─────────────────────────────────────────────────────────────────┤
│ • Pass Rate: >92%                                               │
│ • Execution Rate: 100%                                          │
│ • Open Defects at Sprint End: 0 Critical/High                  │
│ • Code Coverage: >80%                                           │
└─────────────────────────────────────────────────────────────────┘
```

**c) Test Summary Dashboard:**

```markdown
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SPRINT 5 - TEST SUMMARY DASHBOARD                        │
│                    Period: Dec 1-14, 2024                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   TEST EXECUTION                        DEFECT STATUS                       │
│   ┌────────────────────┐               ┌────────────────────┐              │
│   │ ████████████░░ 90% │               │ Fixed    ████████ 28             │
│   │ Executed: 180/200  │               │ Open     ██ 7                    │
│   └────────────────────┘               │ Total    35                      │
│                                         └────────────────────┘              │
│                                                                             │
│   TEST RESULTS                          SEVERITY BREAKDOWN                  │
│   ┌─────────────────────────────┐      ┌────────────────────┐              │
│   │ Passed  ████████████░░ 150  │      │ Critical  ██ 3     │              │
│   │ Failed  ████░ 25            │      │ High      ████ 8   │              │
│   │ Blocked █ 5                 │      │ Medium    ██████ 15│              │
│   │                             │      │ Low       ████ 9   │              │
│   │ Pass Rate: 83.33%           │      └────────────────────┘              │
│   └─────────────────────────────┘                                          │
│                                                                             │
│   KEY METRICS                                                               │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │ Metric              │ Value  │ Target │ Status              │          │
│   ├─────────────────────┼────────┼────────┼─────────────────────│          │
│   │ Execution Rate      │ 90%    │ 100%   │ ⚠️ Below Target     │          │
│   │ Pass Rate           │ 83.33% │ 95%    │ ❌ Critical          │          │
│   │ Defect Density      │ 2.33   │ <3.0   │ ✅ On Target         │          │
│   │ Fix Rate            │ 80%    │ 100%   │ ⚠️ Below Target     │          │
│   │ Code Coverage       │ 72%    │ 80%    │ ⚠️ Below Target     │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                             │
│   TREND (Last 5 Sprints)                                                   │
│   Pass Rate:  S1[92%] → S2[89%] → S3[85%] → S4[84%] → S5[83%] ↓           │
│   Coverage:   S1[65%] → S2[68%] → S3[70%] → S4[71%] → S5[72%] ↗           │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────┐          │
│   │ SPRINT HEALTH: ⚠️ AT RISK                                   │          │
│   │                                                             │          │
│   │ Critical Issues:                                            │          │
│   │ • 7 open defects (3 Critical, 4 High)                      │          │
│   │ • Pass rate declining for 4 consecutive sprints            │          │
│   │                                                             │          │
│   │ Recommendation: Address quality debt before Sprint 6       │          │
│   └─────────────────────────────────────────────────────────────┘          │
│                                                                             │
│   Prepared by: QA Lead | Date: 2024-12-14 | Next Review: Sprint 6 Retro   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 9: Manual vs Automation Decision (10 điểm)

### Tình huống:
Project Manager hỏi: "Chúng ta có nên automate tất cả test cases không? Team có 500 test cases, hiện 100% manual."

Budget: $20,000 for automation tools và training
Timeline: 3 tháng
Team: 3 QA (chưa biết automation)

### Yêu cầu:
a) Phân tích pros/cons của việc automate 100% và đề xuất tỉ lệ hợp lý (3 điểm)
b) Phân loại 500 test cases: nên/không nên automate với tiêu chí (4 điểm)
c) Tính ROI cho automation proposal của bạn (3 điểm)

### Đáp án mẫu:

**a) Analysis - Should We Automate 100%?**

```markdown
## ANALYSIS: 100% AUTOMATION

### PROS of Full Automation
✓ Faster regression testing
✓ Consistent execution
✓ 24/7 testing capability
✓ Reusable across releases

### CONS of Full Automation
✗ High initial cost (tools + training + development time)
✗ Not all tests are automatable (exploratory, usability)
✗ Maintenance overhead for UI tests
✗ Learning curve for team (3 months minimum)
✗ Some tests run once (not worth automating)

### RECOMMENDATION
┌─────────────────────────────────────────────────────────────┐
│ DO NOT AUTOMATE 100%                                        │
│                                                             │
│ Recommended ratio:                                          │
│ • Automated: 60-70% (300-350 tests)                        │
│ • Manual: 30-40% (150-200 tests)                           │
│                                                             │
│ Why?                                                        │
│ • Some tests require human judgment                        │
│ • ROI negative for one-time tests                          │
│ • Team needs time to build automation skills               │
│ • UI tests are brittle and expensive to maintain           │
└─────────────────────────────────────────────────────────────┘
```

**b) Test Case Classification:**

```markdown
## TEST CASE CLASSIFICATION (500 Tests)

### SHOULD AUTOMATE (350 tests - 70%)

| Category | Count | Criteria | Examples |
|----------|-------|----------|----------|
| Regression Core | 100 | Run every sprint, stable features | Login, Checkout, Payment |
| API Tests | 80 | No UI dependency, fast execution | REST endpoints, data validation |
| Data-Driven | 50 | Same steps, different data | User registration variations |
| Cross-Browser | 40 | Repetitive across browsers | Layout checks on Chrome/Firefox/Safari |
| Smoke Tests | 30 | Run on every build | Critical path verification |
| Database | 30 | Repetitive queries | CRUD operations, data integrity |
| Performance | 20 | Require load simulation | Response time, throughput |

### SHOULD NOT AUTOMATE (150 tests - 30%)

| Category | Count | Criteria | Examples |
|----------|-------|----------|----------|
| Exploratory | 40 | Needs human creativity | New feature exploration |
| Usability/UX | 35 | Subjective assessment | Is UI intuitive? |
| One-time Setup | 25 | Run once | Initial config, migration |
| Visual Design | 20 | Aesthetic judgment | Colors, fonts, spacing |
| Complex Workflows | 15 | Frequent changes | Admin processes |
| Edge Cases | 10 | Rarely executed | Extreme scenarios |
| Ad-hoc | 5 | Unplanned testing | Customer-reported issues |

### DECISION MATRIX

┌────────────────────────────────────────────────────────────────┐
│                    AUTOMATE?                                   │
│                                                                │
│              YES                         NO                    │
│    ┌──────────────────────┬──────────────────────┐            │
│    │ • Runs frequently    │ • Runs once/rarely   │            │
│ H  │ • Stable feature     │ • Frequently changes │ FREQUENCY  │
│ I  │ • High regression    │ • Low risk           │            │
│ G  │   risk               │ • Needs human        │            │
│ H  │ • Data-driven        │   judgment           │            │
│    │ • API/Backend        │ • Visual/UX          │            │
│    │ • Critical path      │ • Exploratory        │            │
│    └──────────────────────┴──────────────────────┘            │
│                        ROI VALUE                               │
└────────────────────────────────────────────────────────────────┘
```

**c) ROI Calculation:**

```markdown
## AUTOMATION ROI CALCULATION

### COSTS (One-time + Ongoing)

| Item | Cost |
|------|------|
| Automation Tool License (Playwright/Selenium) | $0 (open-source) |
| Training for 3 QAs (online courses) | $3,000 |
| Infrastructure (CI/CD runners) | $2,000/year |
| Development Time (3 months × 3 QAs × 50% time) | $15,000 |
| **TOTAL YEAR 1** | **$20,000** |

| Ongoing (Year 2+) | Cost/Year |
|-------------------|-----------|
| Maintenance (20% of dev effort) | $3,000 |
| Infrastructure | $2,000 |
| **TOTAL ONGOING** | **$5,000/year** |

### SAVINGS

| Current Manual Testing | Cost |
|------------------------|------|
| Full regression (500 tests × 10 min/test) | 83 hours |
| Frequency: 2x per sprint (26x/year) | 2,158 hours/year |
| QA hourly cost | $30/hour |
| **ANNUAL MANUAL COST** | **$64,740** |

| After Automation (350 auto, 150 manual) | Cost |
|-----------------------------------------|------|
| Automated tests (350 × 1 min) | 5.8 hours/run |
| Manual tests (150 × 10 min) | 25 hours/run |
| Total per run | 30.8 hours |
| Annual (26 runs) | 801 hours |
| **ANNUAL COST** | **$24,030** |

### ROI CALCULATION

Year 1:
Savings = $64,740 - $24,030 = $40,710
Investment = $20,000
Net Benefit Year 1 = $40,710 - $20,000 = $20,710
ROI Year 1 = ($20,710 / $20,000) × 100% = 103.5%

Year 2+:
Savings = $64,740 - $24,030 = $40,710
Investment = $5,000 (maintenance)
Net Benefit = $35,710
ROI Year 2 = ($35,710 / $5,000) × 100% = 714%

Break-even: ~6 months

### SUMMARY
┌─────────────────────────────────────────────────────────────┐
│ AUTOMATION INVESTMENT ANALYSIS                              │
├─────────────────────────────────────────────────────────────┤
│ Initial Investment:     $20,000                             │
│ Annual Savings:         $40,710                             │
│ Break-even Point:       ~6 months                           │
│ 3-Year Net Benefit:     $20,710 + $35,710×2 = $92,130      │
│ 3-Year ROI:             307%                                │
├─────────────────────────────────────────────────────────────┤
│ RECOMMENDATION: ✅ APPROVE AUTOMATION PROJECT               │
└─────────────────────────────────────────────────────────────┘
```

---

## Câu 10: Test Environment Management (10 điểm)

### Tình huống:
Team của bạn có các môi trường:
- DEV: Developers test locally
- QA: QA team testing
- STAGING: Pre-production
- PROD: Live system

Gần đây có nhiều issues:
- Bug không reproduce được ở QA nhưng xảy ra ở PROD
- Test data bị conflict giữa các testers
- Staging không giống PROD config

### Yêu cầu:
a) Phân tích nguyên nhân các issues và đề xuất Environment Strategy (4 điểm)
b) Thiết kế Test Data Management strategy (3 điểm)
c) Tạo Environment Checklist để đảm bảo parity giữa STAGING và PROD (3 điểm)

### Đáp án mẫu:

**a) Root Cause Analysis & Strategy:**

```markdown
## ROOT CAUSE ANALYSIS

### Issue 1: Bug không reproduce ở QA nhưng xảy ra ở PROD
| Possible Causes | Evidence to Check |
|-----------------|-------------------|
| Different software versions | Compare app version, dependencies |
| Different config settings | Compare env variables, feature flags |
| Different data volume | PROD has millions of records, QA has thousands |
| Different infrastructure | Load balancer, CDN, caching differences |
| Network/latency differences | PROD users geographically distributed |

### Issue 2: Test data conflict giữa testers
| Possible Causes | Impact |
|-----------------|--------|
| Shared database | Tester A modifies data Tester B needs |
| No data isolation | Tests interfere with each other |
| No reset mechanism | Corrupted data accumulates |
| Hardcoded test data | Same user accounts used by multiple testers |

### Issue 3: STAGING không giống PROD
| Differences | Risk |
|-------------|------|
| Smaller infrastructure | Performance issues not caught |
| Different config | Feature behavior differs |
| Sanitized data | Edge cases not covered |
| Missing integrations | Third-party failures not caught |

## ENVIRONMENT STRATEGY

┌─────────────────────────────────────────────────────────────────────────────┐
│                        RECOMMENDED ENVIRONMENT ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐        │
│   │    DEV    │───>│    QA     │───>│  STAGING  │───>│   PROD    │        │
│   └───────────┘    └───────────┘    └───────────┘    └───────────┘        │
│        │                │                │                │                │
│        ▼                ▼                ▼                ▼                │
│   ┌─────────────────────────────────────────────────────────────────┐      │
│   │ Config    │ Local      │ QA-specific│ PROD-clone  │ Production  │      │
│   │ Data      │ Mock/Seed  │ Test data  │ Sanitized   │ Real        │      │
│   │           │            │ (isolated) │ PROD copy   │             │      │
│   │ Infra     │ Docker     │ Shared     │ PROD-like   │ Full        │      │
│   │           │ local      │ cluster    │ scaled down │ scale       │      │
│   └─────────────────────────────────────────────────────────────────┘      │
│                                                                             │
│   KEY PRINCIPLES:                                                           │
│   1. STAGING must mirror PROD (same config, same architecture)             │
│   2. Each tester gets isolated test data namespace                         │
│   3. Automated environment provisioning (Infrastructure as Code)           │
│   4. Data refresh mechanism from sanitized PROD copy                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Test Data Management Strategy:**

```markdown
## TEST DATA MANAGEMENT STRATEGY

### 1. Data Isolation Approach

┌─────────────────────────────────────────────────────────────┐
│                  DATA ISOLATION METHODS                     │
├─────────────────────────────────────────────────────────────┤
│ METHOD 1: Namespace Prefixing                               │
│ • Each tester has unique prefix                             │
│ • Example: qa1_user@test.com, qa2_user@test.com            │
│ • Tests clean up own data after execution                   │
├─────────────────────────────────────────────────────────────┤
│ METHOD 2: Dedicated Database per Tester                     │
│ • qa_tester1_db, qa_tester2_db                             │
│ • Complete isolation                                        │
│ • Higher infrastructure cost                                │
├─────────────────────────────────────────────────────────────┤
│ METHOD 3: Database Transactions (Recommended for Unit/API)  │
│ • Each test runs in transaction                             │
│ • Rollback after test                                       │
│ • Zero cleanup needed                                       │
└─────────────────────────────────────────────────────────────┘

### 2. Test Data Categories

| Category | Source | Refresh Frequency |
|----------|--------|-------------------|
| Static Reference Data | Fixtures/Seeds | Per release |
| User Accounts | Generated (Faker) | Per test run |
| Transaction Data | Created by tests | Per test case |
| Edge Case Data | Manually crafted | As needed |
| Performance Data | PROD sanitized | Weekly |

### 3. Data Lifecycle

# Test Data Factory Pattern
class TestDataFactory:
    def __init__(self, namespace: str):
        self.namespace = namespace  # e.g., "qa_john"

    def create_user(self, **overrides):
        user = {
            "email": f"{self.namespace}_{uuid4()}@testmail.local",
            "name": fake.name(),
            "created_by_test": True,
            **overrides
        }
        return db.users.insert(user)

    def cleanup(self):
        """Clean all data created by this namespace"""
        db.users.delete_many({"email": {"$regex": f"^{self.namespace}_"}})
        db.orders.delete_many({"test_namespace": self.namespace})

# Usage in test
def test_user_checkout():
    factory = TestDataFactory("qa_john")
    try:
        user = factory.create_user()
        # ... run test ...
    finally:
        factory.cleanup()  # Always clean up

### 4. Data Masking for PROD Clone

| Field | Original | Masked |
|-------|----------|--------|
| Email | john@real.com | user_12345@masked.local |
| Name | John Smith | Xxxx Xxxxx |
| Phone | +1-555-1234 | +1-555-0000 |
| SSN | 123-45-6789 | XXX-XX-XXXX |
| Credit Card | 4532-1234-5678-9012 | 4532-XXXX-XXXX-0000 |
```

**c) Environment Parity Checklist:**

```markdown
## STAGING-PROD PARITY CHECKLIST

### Pre-Deployment Verification

┌─────────────────────────────────────────────────────────────────────────────┐
│ ENVIRONMENT PARITY CHECKLIST                                                │
│ Staging Environment: staging.company.com                                    │
│ Production Environment: www.company.com                                     │
│ Reviewer: _____________ Date: _____________                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. APPLICATION VERSION                                                      │
│ □ App version matches (staging == prod-candidate)                          │
│ □ All microservices on same version                                        │
│ □ Database schema version matches                                          │
│                                                                             │
│ 2. INFRASTRUCTURE                                                           │
│ □ Same cloud provider/region type (AWS/GCP/Azure)                          │
│ □ Same container runtime version (Docker, K8s)                             │
│ □ Same load balancer configuration                                         │
│ □ Same CDN settings                                                        │
│ □ Same SSL/TLS configuration                                               │
│                                                                             │
│ 3. CONFIGURATION                                                            │
│ □ Environment variables audited (diff tool)                                │
│   Allowed differences: API keys, URLs, logging levels                      │
│ □ Feature flags match (or documented differences)                          │
│ □ Rate limiting settings match                                             │
│ □ Timeout settings match                                                   │
│ □ Cache TTL settings match                                                 │
│                                                                             │
│ 4. DEPENDENCIES                                                             │
│ □ Same Node.js/Python/Java version                                         │
│ □ Same npm/pip package versions (lock file)                                │
│ □ Same database version (PostgreSQL 15.x)                                  │
│ □ Same Redis/cache version                                                 │
│ □ Same message queue version (RabbitMQ, Kafka)                             │
│                                                                             │
│ 5. DATA CHARACTERISTICS                                                     │
│ □ Representative data volume (min 10% of PROD)                             │
│ □ Edge case data present                                                   │
│ □ Data types coverage (all entity types exist)                             │
│ □ Referential integrity maintained                                         │
│                                                                             │
│ 6. INTEGRATIONS                                                             │
│ □ Third-party APIs use sandbox/test mode                                   │
│ □ Payment gateway in test mode                                             │
│ □ Email uses test SMTP (no real emails)                                    │
│ □ SMS uses test provider                                                   │
│ □ OAuth providers configured                                               │
│                                                                             │
│ 7. MONITORING & LOGGING                                                     │
│ □ Same logging format                                                      │
│ □ Monitoring agents installed                                              │
│ □ Error tracking enabled (Sentry, etc.)                                    │
│ □ APM configured (New Relic, Datadog)                                      │
│                                                                             │
│ 8. SECURITY                                                                 │
│ □ Same authentication mechanism                                            │
│ □ Same authorization rules                                                 │
│ □ Same CORS policy                                                         │
│ □ Same CSP headers                                                         │
│ □ Same firewall rules (or equivalent)                                      │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ DOCUMENTED DIFFERENCES (Accepted)                                           │
│ ┌─────────────────────────────────────────────────────────────────────────┐ │
│ │ 1. Infrastructure scale (STAGING: 2 nodes, PROD: 10 nodes)             │ │
│ │ 2. Log retention (STAGING: 7 days, PROD: 90 days)                      │ │
│ │ 3. Backup frequency (STAGING: daily, PROD: hourly)                     │ │
│ │ 4. _______________________________________________                      │ │
│ └─────────────────────────────────────────────────────────────────────────┘ │
│                                                                             │
│ VERIFICATION RESULT:                                                        │
│ □ PASSED - Environment parity confirmed                                    │
│ □ FAILED - Issues found (list below)                                       │
│                                                                             │
│ Issues: ________________________________________________________________   │
│ ________________________________________________________________________   │
│                                                                             │
│ Approved by: _________________ Signature: _____________ Date: ___________  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tổng kết

| Câu | Chủ đề | Điểm |
|-----|--------|------|
| 1 | SDLC Model Selection | 10 |
| 2 | Test Levels & Pyramid | 10 |
| 3 | Smoke vs Sanity Testing | 10 |
| 4 | Test Plan Creation | 10 |
| 5 | Regression Test Selection | 10 |
| 6 | Exploratory Testing | 10 |
| 7 | Defect Life Cycle | 10 |
| 8 | Test Metrics Analysis | 10 |
| 9 | Manual vs Automation | 10 |
| 10 | Environment Management | 10 |
| **Tổng** | | **100** |

---

*Chúc bạn ôn thi tốt! 🎓*
