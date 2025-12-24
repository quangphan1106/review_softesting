# TÀI LIỆU ÔN TẬP CUỐI KỲ - SOFTWARE TESTING (CS423)

> **Lưu ý:** Đề thi có 7 câu ở mức VẬN DỤNG (không phải định nghĩa). Tập trung vào scenarios, so sánh tools, và giải quyết vấn đề thực tế.

---

## MỤC LỤC

1. [CI/CD](#1-cicd)
2. [Performance Testing](#2-performance-testing)
3. [Automation Testing](#3-automation-testing)
4. [GUI & Usability Testing](#4-gui--usability-testing)
5. [API Testing & Mocking](#5-api-testing--mocking)
6. [AI trong Testing](#6-ai-trong-testing)
7. [Database Testing](#7-database-testing)
8. [Domain Testing & Data Generation](#8-domain-testing--data-generation)

---

## 1. CI/CD

### Khái niệm cốt lõi

| Thuật ngữ | Định nghĩa | Điểm khác biệt |
|-----------|-----------|----------------|
| **CI (Continuous Integration)** | Merge code thường xuyên, auto build + test | Feedback nhanh, phát hiện lỗi sớm |
| **CD (Continuous Delivery)** | Code sẵn sàng deploy, CẦN approve thủ công | Có human failsafe |
| **Continuous Deployment** | Auto deploy lên production | Không cần approve, cần CI mature |

### 3 Cột trụ CI
1. **SCM (Git)**: GitHub Flow, branch strategy, code review qua PR
2. **Automated Build**: Compile, install dependencies, detect missing files
3. **Automated Testing**: Unit test, lint, security scan, coverage

### GitHub Actions - Components
```yaml
name: CI Pipeline
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install && npm test
```

### Câu hỏi vận dụng tiềm năng

| Scenario | Trả lời |
|----------|---------|
| "It works on my machine" problem? | Implement CI + Docker + automated tests |
| Khi nào dùng Delivery vs Deployment? | Delivery: high-risk (ngân hàng); Deployment: tech company move fast |
| Hardcode password vào GitHub? | Dùng GitHub Secrets, inject qua env variables |
| Chạy test hàng đêm - cron? | `schedule: cron: '0 2 * * *'` |

---

## 2. PERFORMANCE TESTING

### 6 Loại Performance Testing

| Loại | Mục đích | Pattern | Ví dụ |
|------|----------|---------|-------|
| **Load Testing** | Test tại expected load | Ramp-up dần | 100 users bình thường |
| **Stress Testing** | Tìm breaking point | Vượt capacity | 10,000 users đến khi fail |
| **Spike Testing** | Recovery sau sudden load | Jump nhanh | Flash sale: 50→5000 users |
| **Endurance/Soak** | Ổn định dài hạn | Steady 24h+ | Detect memory leaks |
| **Volume Testing** | Large data volume | DB 100M rows | Query performance |
| **Scalability** | Linear scaling? | Add resources | 500→5000 users + nodes |

### Key Metrics

| Metric | Ý nghĩa | Công thức |
|--------|---------|-----------|
| **Response Time** | Thời gian response | ms |
| **Throughput** | Requests/sec | TP = Requests / Time |
| **Error Rate** | % failed requests | (Failed/Total) × 100% |
| **Percentiles** | p90, p95, p99 | Quan trọng hơn averages |

### Tools So sánh

| Tool | Language | Ưu điểm | Best for |
|------|----------|---------|----------|
| **JMeter** | Java/GUI | Multi-protocol, plugins nhiều | Enterprise, complex workflows |
| **k6** | JavaScript/Go | Performance cao, CI/CD friendly | DevOps, modern web APIs |
| **Locust** | Python | Dễ học, web UI | Python teams |
| **Gatling** | Scala | High-throughput, HTML reports | Scala/Java teams |

### JMeter Components
- **Thread Group**: Virtual users, ramp-up, loops
- **Samplers**: HTTP Request, JDBC Request
- **Timers**: Think time (realistic delays)
- **Assertions**: Check response status, JSON path
- **Listeners**: View Results, Aggregate Report

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| Setup Load Test cho login API? | Define goals (RT<2s, Error<1%), 100 users, 30s ramp-up, CSV test data, assertions |
| Stress vs Load Test khác gì? | Load: expected load, <1% errors; Stress: beyond capacity, tìm breaking point |
| Response time 700ms nhưng 84% error rate? | Error rate critical, likely auth issue không phải performance |
| Detect memory leak như thế nào? | Endurance test 24h+, monitor heap grows |

---

## 3. AUTOMATION TESTING

### Desktop vs Web Automation

| Khía cạnh | Desktop | Web |
|-----------|---------|-----|
| Target | OS-installed apps | Browser-based |
| Tools | Pywinauto, WinAppDriver | Selenium, Playwright, Cypress |
| Interaction | Win32 API, GUI controls | DOM, CSS selectors |

### Selenium Core Concepts

**Locators (QUAN TRỌNG):**
| Locator | Mô tả | Ví dụ |
|---------|-------|-------|
| id | By ID attribute | `driver.find_element(By.ID, "login")` |
| xpath | XPath expression | `//button[@text='Submit']` |
| css selector | CSS selector | `.btn-primary` |
| link text | Anchor text | `driver.find_element(By.LINK_TEXT, "Click")` |

**Waits (CỰC KỲ QUAN TRỌNG):**

| Loại | Cách dùng | Vấn đề |
|------|-----------|--------|
| `Thread.sleep()` | ❌ TRÁNH | Slow, flaky, wastes time |
| **Implicit Wait** | Set once for driver | Chỉ check "present?", không check visible |
| **Explicit Wait** | Wait for specific condition | ✅ BEST: visibility, clickability |

```python
# Explicit Wait example
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

wait = WebDriverWait(driver, 10)
element = wait.until(EC.visibility_of_element_located((By.ID, "login")))
```

**Page Object Model (POM):**
- Separate class cho mỗi page
- Chứa locators + methods
- Benefits: Reusability, maintainability

### Common Challenges & Solutions

| Problem | Solution |
|---------|----------|
| Dynamic IDs (`id="ember4661"`) | Use stable text/labels as anchor |
| iFrame | `driver.switch_to.frame("frame_id")` |
| Captcha/OTP | Disable in test env, static OTP |
| Flaky tests | Explicit waits, stable locators |

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| Tại sao không dùng Thread.sleep()? | Slow + flaky. Dùng implicit/explicit waits |
| Dynamic ID problem? | Use Stable Neighbor Strategy - locate by text/labels |
| POM là gì? Benefits? | Design pattern: separate page classes, reusability |
| Khi nào nên automate? | Regression, smoke, repetitive tests. KHÔNG: unstable UI, ad-hoc |

---

## 4. GUI & USABILITY TESTING

### GUI vs UI vs UX

| Term | Scope | Focus |
|------|-------|-------|
| **UI** | Tất cả interface (voice, gesture, touch) | Phương thức tương tác |
| **GUI** | Đồ họa (buttons, menus, icons) | Visual elements |
| **Usability** | Dễ sử dụng | Efficiency, effectiveness, satisfaction |
| **UX** | Toàn bộ trải nghiệm | Emotions, attitudes |

### GUI Checklist - 4 Categories

| Category | Check items |
|----------|-------------|
| **Colors** | Contrast, consistency, distinguishing states |
| **Content** | Font consistency, alignment, spelling |
| **Images** | Display correctly, alignment, sizing |
| **Forms** | Labels, input states, radio/checkbox logic |

### Tools

| Tool | Purpose | Pros | Cons |
|------|---------|------|------|
| **Selenium** | Automation, CSS checks | Free, reads raw code | Brittle, no visual testing |
| **Playwright** | Cross-browser local | 3 engines × 3 form factors | Local only |
| **BrowserStack** | Real devices cloud | 3000+ devices | Paid, 1-min free limit |
| **Applitools** | AI Visual Testing | Smart comparison | Paid |
| **Lookback.io** | UX recording | Screen + voice + facial | 5 session limit |

### Traditional vs AI-First UI Testing

| Aspect | Traditional | AI-First |
|--------|-------------|----------|
| Element recognition | Fixed locators (XPath, CSS) | Visual, semantic, contextual |
| Stability | Brittle | Adaptive |
| Validation | Rule-based assertions | Smart/visual assertions |
| Bug detection | Only explicitly coded | Visual bugs, layout shifts |

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| 200+ products, check CO2 badge colors? | Selenium + Python automation |
| Cross-browser strategy? | Playwright (3 engines × 3 form factors) + BrowserStack (real devices) |
| Broken links on 1000+ pages? | Sitebulb (full site crawler), không Check My Links (1 page) |
| Checkout flow user feelings? | Lookback.io (screen + audio + facial) |

---

## 5. API TESTING & MOCKING

### 5 Loại API Testing

| Loại | Purpose | Ví dụ |
|------|---------|-------|
| **Functional** | API hoạt động theo spec | Login: 200 + token, 401 sai password |
| **Load** | Performance dưới tải | 1000 users, 95% < 300ms |
| **Security** | Auth, vulnerabilities | SQL injection, XSS |
| **Contract** | Schema đúng spec | Field types, required fields |
| **Integration** | Multiple systems | Order → Payment → Inventory |

### Postman Components

| Component | Purpose |
|-----------|---------|
| **API Requests** | GET, POST, PUT, DELETE |
| **Pre-request Script** | Setup: generate UUID, set headers |
| **Post-request Script** | Validate: status, extract token |
| **Collections** | Group related requests |
| **Environment Variables** | Store URLs, tokens (Dev/Prod) |

### API Mocking

**4 Loại Mocking:**
| Type | Description | Use case |
|------|-------------|----------|
| **Static** | Fixed response | Early UI dev |
| **Dynamic** | Response varies by input | Different workflows |
| **Contract-based** | From OpenAPI/Swagger | Schema alignment |
| **Behavior-driven** | Simulate conditions | Timeouts, failures |

**Benefits:** Parallel development, stable testing, early bug detection

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| Frontend cần work trước backend ready? | API Mocking (Static/Dynamic) |
| Pre-request vs Post-request Script? | Pre: setup headers/data; Post: validate + extract token |
| API accepts invalid format? | Negative Test / Boundary Test |
| Khi nào Contract Testing? | Microservices, ensure schema alignment |

---

## 6. AI TRONG TESTING

### 2 Miền Riêng Biệt

| Miền | Mô tả | Tools |
|------|-------|-------|
| **AI-Powered Testing** | Dùng AI cải thiện testing | UnitTestAI, Qodo, Healenium |
| **Testing AI Systems** | Test hệ thống AI/ML | OpenAI Evals, custom frameworks |

### AI Testing Tools

| Tool | Function |
|------|----------|
| **Self-Healing** (Healenium) | Auto-fix locators khi UI thay đổi |
| **Intelligent Test Gen** | Sinh test case từ user actions |
| **Visual AI** (Applitools) | Compare baseline vs current screenshot |
| **Postbot** | Write tests, visualize responses |

### Test Oracle Problem (QUAN TRỌNG)

**Vấn đề:** AI sinh input + execute code, NHƯNG không biết output ĐÚNG là gì
- AI có thể write test pass bugs (treat bug như "correct")
- 100% coverage ≠ 100% bug-free

**Giải pháp:** Paradigm shift: "Test Author" → "Test Auditor"
- AI generates, humans verify INTENT

### Testing AI Systems - Categories

| Category | Focus |
|----------|-------|
| **Functional** | Accuracy, Precision, Recall, F1 |
| **Performance** | Inference latency, throughput |
| **Robustness** | Adversarial examples, noise |
| **Fairness** | Bias detection, demographic parity |
| **Safety** | Prompt injection, data poisoning |

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| Test Oracle Problem là gì? | AI không biết output đúng, có thể pass bugs |
| AI testing limitation? | Misinterpret docs, limited business rules, needs human review |
| Deterministic vs Probabilistic? | Traditional: same input → same output; AI: same input → different outputs |
| Self-healing tests? | Healenium: AI learns elements, auto-fix locators |

---

## 7. DATABASE TESTING

### 3 Loại DB Testing

| Loại | Focus | Examples |
|------|-------|----------|
| **Structural** | Schema, constraints | Table validation, indexes, FK |
| **Functional** | CRUD operations | SELECT, INSERT, UPDATE, DELETE |
| **Non-Functional** | Performance, security | Load test, SQL injection |

### ACID Properties (CỰC KỲ QUAN TRỌNG)

| Property | Meaning | Test |
|----------|---------|------|
| **Atomicity** | All or nothing | Multi-INSERT: all or rollback |
| **Consistency** | Valid state → valid state | Constraints maintained |
| **Isolation** | Concurrent txns don't interfere | No dirty reads |
| **Durability** | Committed changes persist | Survive crashes |

### Index Testing

**Trade-off:**
- ✅ Tăng tốc SELECT (Read)
- ❌ Giảm tốc INSERT/UPDATE/DELETE (Write)

**Test activities:**
- Compare queries WITH vs WITHOUT indexes
- Verify indexes on frequently filtered columns
- Check composite index column order

### Tools

| Tool | Purpose |
|------|---------|
| **DBUnit** | Test data setup, assertions |
| **Maven** | Build automation, dependencies |
| **Mockaroo** | Generate realistic test data |

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| Test ACID properties? | Multi-INSERT transaction: all commit or all rollback |
| Index trade-offs? | Faster SELECT, slower INSERT/UPDATE/DELETE |
| Migration testing? | Validate data types, relationships, integrity preserved |
| SQL injection test? | Input malicious SQL, verify rejection |

---

## 8. DOMAIN TESTING & DATA GENERATION

### Domain Testing - 4 Bước

1. **Identify Variables**: Input/Output của feature
2. **Equivalence Classes**: Chia thành Valid/Invalid classes
3. **Select Test Cases**: 1 đại diện từ mỗi EC
4. **Boundary Value Analysis**: Test giá trị biên

### Boundary Value Analysis

**Password 8-50 ký tự:**
| Boundary | Values to test |
|----------|----------------|
| Lower | 7, 8, 9 |
| Upper | 49, 50, 51 |

**Stock 0-10,000:**
| Boundary | Values to test |
|----------|----------------|
| Lower | -1, 0, 1 |
| Upper | 9999, 10000, 10001 |

### Data Generation với Faker

**Key concepts:**
- `Faker.seed(42)`: Reproducibility
- Realistic data > random strings
- Hierarchical: Categories → Products → Transactions

**Example fields:**
| Field | Range/Logic |
|-------|-------------|
| Price | 2.00-500.00 (2 decimals) |
| Stock | 0-100 integer |
| DOB | 1970-2005 (age 20-55) |
| Role | 5:1 user:admin ratio |

### Câu hỏi vận dụng

| Scenario | Trả lời |
|----------|---------|
| BVA cho password 8-50 ký tự? | Test: 7, 8, 9, 49, 50, 51 |
| Faker.seed(42) để làm gì? | Reproducibility - cùng data mỗi lần chạy |
| EC vs BVA khác gì? | EC: chia classes; BVA: test edges (bổ sung) |
| 1000 transactions, 50 users? | Random user_id(1-50), mỗi user ~20 txns |

---

## CHECKLIST ÔN TẬP

### Công thức cần nhớ
```
Throughput = Requests / Time (seconds)
Error Rate = (Failed / Total) × 100%
Bug Coverage = (Bugs Detected / Total Bugs) × 100%
Code Coverage = (Lines Executed / Total Lines) × 100%
```

### Tools Summary

| Domain | Tools |
|--------|-------|
| CI/CD | GitHub Actions, Jenkins |
| Performance | JMeter, k6, Locust, Gatling |
| Web Automation | Selenium, Playwright, Cypress |
| Desktop Automation | Pywinauto, WinAppDriver |
| GUI Testing | Selenium, Applitools, BrowserStack |
| API Testing | Postman, WireMock |
| AI Testing | Healenium, Applitools, UnitTestAI, Qodo |
| DB Testing | DBUnit, Maven, Mockaroo |

### Keywords tiếng Anh quan trọng
- **Throughput** = Số requests/sec
- **Latency** = Độ trễ
- **Ramp-up** = Tăng load dần
- **Think Time** = Thời gian đợi user
- **Percentile** = p90, p95, p99
- **Locator** = XPath, CSS selector, ID
- **Assertion** = Kiểm tra kết quả
- **Fixture** = Setup/teardown
- **Flaky test** = Test không ổn định

---

## CẤU TRÚC TRẢ LỜI ĐỀ THI

### Mẫu trả lời Scenario

```
1. XÁC ĐỊNH VẤN ĐỀ
   - [Mô tả ngắn gọn scenario]

2. PHÂN TÍCH
   - [Các yếu tố cần xem xét]
   - [So sánh options nếu có]

3. GIẢI PHÁP
   - [Chọn approach/tool cụ thể]
   - [Giải thích TẠI SAO]

4. IMPLEMENTATION (nếu cần)
   - [Steps cụ thể]
   - [Code snippet nếu relevant]
```

---

**Chúc ôn tập tốt!** 🎯
