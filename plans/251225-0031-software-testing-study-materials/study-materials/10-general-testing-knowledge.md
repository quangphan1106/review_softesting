# Topic 10: General Software Testing Knowledge
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Software Development Life Cycle (SDLC) & Testing

### 1.1 SDLC Models Overview

**Waterfall Model:**
```
Requirements → Design → Implementation → Testing → Deployment → Maintenance
     ↓            ↓           ↓             ↓          ↓            ↓
  (Sequential, no going back - high risk of late defect discovery)
```
- Testing happens AFTER development complete
- Late defect detection = expensive fixes
- Best for: Fixed requirements, well-understood projects

**V-Model (Verification & Validation):**
```
Requirements    ─────────────────────────────>  Acceptance Testing
      ↓                                               ↑
System Design   ─────────────────────────>  System Testing
      ↓                                          ↑
Architecture    ─────────────────────>  Integration Testing
      ↓                                     ↑
Module Design   ───────────────>  Unit Testing
      ↓                              ↑
      └──────>  Coding  ─────────────┘
```
- Each dev phase has corresponding test phase
- Test planning starts early (left side)
- Test execution happens on right side

**Agile/Scrum Model:**
```
Sprint 1          Sprint 2          Sprint 3
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Plan     │     │ Plan     │     │ Plan     │
│ Develop  │ ──> │ Develop  │ ──> │ Develop  │ ──> ...
│ Test     │     │ Test     │     │ Test     │
│ Review   │     │ Review   │     │ Review   │
└──────────┘     └──────────┘     └──────────┘
   2-4 weeks        2-4 weeks        2-4 weeks
```
- Iterative & incremental
- Testing integrated throughout each sprint
- Continuous feedback and adaptation

### 1.2 Testing in Agile

**Agile Testing Quadrants:**
```
                    Business-Facing
                         ↑
           Q2                      Q3
    ┌─────────────────┬─────────────────┐
    │ Functional Tests│  Exploratory    │
    │ Examples        │  Usability      │
    │ Story Tests     │  UAT            │
    │ Prototypes      │  Alpha/Beta     │
    │                 │                 │
    │ Support Team    │ Critique Product│
←───┼─────────────────┼─────────────────┼───→
    │ Guide Dev       │ Critique Product│ Automated
    │                 │                 │
    │ Unit Tests      │  Performance    │
    │ Component Tests │  Load Testing   │
    │ Integration     │  Security       │
           Q1                      Q4
                         ↓
                  Technology-Facing
```

**Scrum Testing Activities:**
- **Sprint Planning**: Estimate testing effort, define DoD
- **Daily Standup**: Report testing progress/blockers
- **Sprint Execution**: Continuous testing, automation
- **Sprint Review**: Demo working software
- **Retrospective**: Improve testing process

**Definition of Done (DoD) - Testing Checklist:**
```markdown
□ Unit tests written and passing (>80% coverage)
□ Integration tests passing
□ Code reviewed and approved
□ No critical/high severity bugs open
□ Regression tests executed
□ Documentation updated
□ Performance benchmarks met
```

---

## 2. Test Levels (Test Pyramid)

### 2.1 Test Pyramid

```
                    ▲
                   /│\
                  / │ \      E2E/UI Tests (Few, Slow, Expensive)
                 /  │  \     - Full system validation
                /   │   \    - Browser automation
               /────┼────\
              /     │     \  Integration Tests (Medium)
             /      │      \ - API testing
            /       │       \- Component interaction
           /────────┼────────\
          /         │         \  Unit Tests (Many, Fast, Cheap)
         /          │          \ - Individual functions
        /           │           \- Isolated, no dependencies
       ─────────────┴────────────
```

### 2.2 Test Levels Detailed

**Level 1: Unit Testing**
- Tests individual functions/methods in isolation
- Written by developers
- Fastest execution, easiest to debug
- Tools: JUnit, pytest, Jest, NUnit

```python
# Unit Test Example
def test_calculate_discount():
    # Arrange
    price = 100
    discount_percent = 20

    # Act
    result = calculate_discount(price, discount_percent)

    # Assert
    assert result == 80
```

**Level 2: Integration Testing**
- Tests interaction between components/modules
- Verifies interfaces, data flow
- Types: Big Bang, Top-Down, Bottom-Up, Sandwich

```
Top-Down Integration:
    [Module A] (tested first with stubs)
       ↓
    [Module B] (stub replaced, tested)
       ↓
    [Module C] (stub replaced, tested)

Bottom-Up Integration:
    [Module C] (tested first with drivers)
       ↑
    [Module B] (driver replaced, tested)
       ↑
    [Module A] (driver replaced, tested)
```

**Level 3: System Testing**
- Tests complete integrated system
- Validates functional & non-functional requirements
- Black-box testing approach
- Environment similar to production

**Level 4: Acceptance Testing**
- **UAT (User Acceptance Testing)**: End-users validate business requirements
- **Alpha Testing**: Internal testing at developer's site
- **Beta Testing**: External testing by real users
- **Contract/Regulation Testing**: Compliance verification

---

## 3. Test Types Classification

### 3.1 Functional vs Non-Functional

```
┌────────────────────────────────────────────────────────────────┐
│                      SOFTWARE TESTING                          │
├──────────────────────────┬─────────────────────────────────────┤
│     FUNCTIONAL           │         NON-FUNCTIONAL              │
│  (What system does)      │      (How system performs)          │
├──────────────────────────┼─────────────────────────────────────┤
│ • Unit Testing           │ • Performance Testing               │
│ • Integration Testing    │ • Load Testing                      │
│ • System Testing         │ • Stress Testing                    │
│ • Regression Testing     │ • Security Testing                  │
│ • Smoke Testing          │ • Usability Testing                 │
│ • Sanity Testing         │ • Compatibility Testing             │
│ • UAT                    │ • Reliability Testing               │
└──────────────────────────┴─────────────────────────────────────┘
```

### 3.2 Smoke vs Sanity Testing

| Aspect | Smoke Testing | Sanity Testing |
|--------|---------------|----------------|
| **Purpose** | Verify build stability | Verify specific functionality |
| **Scope** | Broad, shallow | Narrow, deep |
| **When** | After new build | After bug fix/minor change |
| **Coverage** | Critical paths only | Specific module/feature |
| **Performed by** | Developers/QA | Usually QA |
| **Build rejection** | Yes, if fails | No, only module rejected |
| **Also called** | Build Verification Test | Subset of regression |

```
SMOKE TEST:                          SANITY TEST:
┌─────────────────────┐              ┌─────────────────────┐
│  Login ✓            │              │                     │
│  Dashboard loads ✓  │              │  Search Module      │
│  Navigation works ✓ │              │  ┌───────────────┐  │
│  Basic CRUD ✓       │              │  │ Search A ✓    │  │
│  Logout ✓           │              │  │ Search B ✓    │  │
│                     │              │  │ Filter X ✓    │  │
│ (Surface level all) │              │  │ Sort Y ✓      │  │
└─────────────────────┘              │  └───────────────┘  │
                                     │ (Deep dive one area)│
                                     └─────────────────────┘
```

### 3.3 Regression Testing

**Definition:** Re-testing after code changes to ensure existing functionality still works

**When to perform:**
- After bug fixes
- After new feature additions
- After code refactoring
- After configuration changes
- After environment changes

**Regression Test Selection Strategies:**
1. **Retest All**: Run entire test suite (expensive but thorough)
2. **Selective**: Choose tests based on changed areas
3. **Priority-based**: Run high-priority tests first
4. **Risk-based**: Focus on high-risk areas

```
Code Change Impact Analysis:
                    ┌──────────┐
                    │ Module A │ ← Change here
                    └────┬─────┘
                         │
            ┌────────────┼────────────┐
            ↓            ↓            ↓
      ┌──────────┐ ┌──────────┐ ┌──────────┐
      │ Module B │ │ Module C │ │ Module D │
      └──────────┘ └──────────┘ └──────────┘
            ↑            ↑            ↑
        Must test    Must test    Must test
```

### 3.4 Exploratory Testing

**Definition:** Simultaneous learning, test design, and test execution

**Session-Based Test Management (SBTM):**
```
Session Charter: Explore [feature] with [resources] to discover [information]

┌─────────────────────────────────────────────────────────────┐
│ Session: Payment Processing                                 │
│ Duration: 90 minutes                                        │
│ Tester: John Doe                                           │
│ Charter: Explore checkout flow with invalid payment methods │
├─────────────────────────────────────────────────────────────┤
│ Notes:                                                      │
│ - Found issue with expired card handling                   │
│ - Edge case: switching payment method mid-checkout         │
│ - Question: What happens with 3D Secure timeout?           │
├─────────────────────────────────────────────────────────────┤
│ Bugs Found: 3 | Questions: 2 | Test Ideas: 5               │
└─────────────────────────────────────────────────────────────┘
```

### 3.5 Other Important Test Types

**Alpha Testing:**
- Internal testing at developer's site
- By internal employees (not dev team)
- Before beta release

**Beta Testing:**
- External testing at customer's site
- By real end-users
- Feedback collected for improvement

**Ad-hoc Testing:**
- Informal, unplanned testing
- No documentation
- Based on tester's intuition

**Monkey Testing:**
- Random inputs and actions
- No expected results
- Goal: Crash the system

**Gorilla Testing:**
- Repeatedly test one module
- Heavy load on specific functionality
- Stress specific component

---

## 4. Test Documentation

### 4.1 Test Plan Structure (IEEE 829)

```markdown
# Test Plan: [Project Name]

## 1. Test Plan Identifier
TP-PROJECT-001-v1.0

## 2. Introduction
Purpose and scope of testing

## 3. Test Items
- Feature A v1.2
- Feature B v2.0
- Module C v1.0

## 4. Features to be Tested
- User authentication
- Payment processing
- Report generation

## 5. Features NOT to be Tested
- Legacy admin panel (out of scope)
- Third-party integrations

## 6. Approach
- Test levels: Unit, Integration, System
- Test types: Functional, Performance, Security
- Automation strategy

## 7. Pass/Fail Criteria
- All critical tests must pass
- No open P1/P2 bugs
- Code coverage > 80%

## 8. Suspension/Resumption Criteria
Suspend if: Build failure, environment down
Resume when: Issues resolved

## 9. Test Deliverables
- Test cases
- Test scripts
- Test reports
- Defect reports

## 10. Testing Tasks & Schedule
| Task | Start | End | Owner |
|------|-------|-----|-------|
| Unit Testing | Week 1 | Week 2 | Dev Team |
| Integration | Week 3 | Week 4 | QA Team |

## 11. Environment Requirements
- Server: Ubuntu 22.04, 16GB RAM
- Database: PostgreSQL 15
- Browser: Chrome 120+

## 12. Responsibilities
- Test Manager: Overall coordination
- Test Lead: Test design, review
- Testers: Execution, reporting

## 13. Risks & Contingencies
| Risk | Mitigation |
|------|------------|
| Resource shortage | Cross-training |
| Environment delay | Use staging |

## 14. Approvals
- QA Manager: ___________
- Project Manager: ___________
```

### 4.2 Test Case Template

```markdown
┌─────────────────────────────────────────────────────────────┐
│ Test Case ID: TC-LOGIN-001                                  │
│ Test Case Name: Verify successful login with valid creds    │
│ Module: Authentication                                      │
│ Priority: High | Severity: Critical                         │
├─────────────────────────────────────────────────────────────┤
│ Preconditions:                                              │
│ - User account exists in system                             │
│ - User is on login page                                     │
├─────────────────────────────────────────────────────────────┤
│ Test Data:                                                  │
│ - Username: testuser@example.com                            │
│ - Password: ValidPass123!                                   │
├─────────────────────────────────────────────────────────────┤
│ Steps:                                                      │
│ 1. Navigate to login page                                   │
│ 2. Enter username in email field                            │
│ 3. Enter password in password field                         │
│ 4. Click "Login" button                                     │
├─────────────────────────────────────────────────────────────┤
│ Expected Result:                                            │
│ - User redirected to dashboard                              │
│ - Welcome message displayed                                 │
│ - Session created                                           │
├─────────────────────────────────────────────────────────────┤
│ Actual Result: ___________________                          │
│ Status: Pass / Fail / Blocked                               │
│ Tested By: ___________ Date: ___________                    │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Test Summary Report

```markdown
# Test Summary Report

## Project: E-Commerce Platform v2.0
## Test Period: Dec 1-20, 2024

### Executive Summary
Testing completed with 95% pass rate. 3 critical bugs found and fixed.
System ready for production with minor known issues.

### Test Metrics
| Metric | Value |
|--------|-------|
| Total Test Cases | 450 |
| Executed | 445 |
| Passed | 422 |
| Failed | 18 |
| Blocked | 5 |
| Pass Rate | 94.8% |

### Defect Summary
| Severity | Found | Fixed | Open |
|----------|-------|-------|------|
| Critical | 3 | 3 | 0 |
| High | 8 | 7 | 1 |
| Medium | 15 | 12 | 3 |
| Low | 10 | 5 | 5 |

### Test Coverage
- Functional: 95%
- Regression: 100%
- Performance: 85%
- Security: 80%

### Known Issues (Deferred)
1. Minor UI alignment on Safari mobile
2. Slow report generation (>5000 records)

### Recommendation
✅ APPROVED for production release
```

---

## 5. Test Techniques Overview

### 5.1 Black-Box Techniques

| Technique | Description | Use Case |
|-----------|-------------|----------|
| **Equivalence Partitioning** | Divide inputs into equivalent classes | Reduce test cases |
| **Boundary Value Analysis** | Test at boundaries of partitions | Find off-by-one errors |
| **Decision Table** | Test combinations of conditions | Complex business rules |
| **State Transition** | Test state changes | Workflow testing |
| **Use Case Testing** | Test user scenarios | End-to-end flows |

### 5.2 White-Box Techniques

| Technique | Description | Coverage |
|-----------|-------------|----------|
| **Statement Coverage** | Every statement executed | Basic |
| **Branch/Decision** | Every decision outcome | Medium |
| **Condition Coverage** | Every condition true/false | Higher |
| **Path Coverage** | All possible paths | Highest |
| **MC/DC** | Modified condition/decision | Aviation/Safety |

```
Code Example:
if (a > 0 AND b > 0):    # Decision with 2 conditions
    print("Positive")
else:
    print("Not positive")

Statement Coverage: 2 tests (one for each print)
Branch Coverage: 2 tests (true path, false path)
Condition Coverage: 4 combinations (a,b: T/T, T/F, F/T, F/F)
```

---

## 6. Quality Metrics

### 6.1 Test Metrics

```
┌────────────────────────────────────────────────────────────┐
│                     TEST METRICS                           │
├────────────────────────────────────────────────────────────┤
│ PROCESS METRICS          │ PRODUCT METRICS                 │
├──────────────────────────┼─────────────────────────────────┤
│ • Test case execution    │ • Defect density                │
│   rate                   │   (defects/KLOC)                │
│ • Test coverage %        │ • Defect removal efficiency     │
│ • Defect detection rate  │ • Mean time to failure          │
│ • Test efficiency        │ • Customer-found defects        │
│ • Automation ROI         │ • Code coverage %               │
└──────────────────────────┴─────────────────────────────────┘
```

**Key Formulas:**

```
Defect Density = Total Defects / Size (KLOC or FP)

Defect Removal Efficiency (DRE) =
    (Defects found during testing / Total defects) × 100%

Test Case Effectiveness =
    (Defects found by test cases / Total test cases executed) × 100%

Automation ROI =
    (Manual cost - Automation cost) / Automation cost × 100%

Test Coverage =
    (Requirements covered by tests / Total requirements) × 100%
```

### 6.2 Defect Life Cycle

```
┌─────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│   NEW   │───>│  ASSIGNED│───>│   OPEN   │───>│  FIXED   │
└─────────┘    └──────────┘    └──────────┘    └────┬─────┘
                    │                               │
                    ▼                               ▼
              ┌──────────┐                   ┌──────────┐
              │ DUPLICATE│                   │ VERIFIED │
              │ REJECTED │                   └────┬─────┘
              │ DEFERRED │                        │
              └──────────┘               ┌────────┴────────┐
                                         ▼                 ▼
                                   ┌──────────┐      ┌──────────┐
                                   │  CLOSED  │      │ REOPENED │
                                   └──────────┘      └──────────┘
```

**Defect Report Template:**

```markdown
┌─────────────────────────────────────────────────────────────┐
│ Defect ID: BUG-2024-0123                                    │
│ Title: Login fails with special characters in password      │
├─────────────────────────────────────────────────────────────┤
│ Severity: High          Priority: P1                        │
│ Status: Open            Assigned To: dev@company.com        │
│ Found In: v2.1.0        Environment: Production             │
├─────────────────────────────────────────────────────────────┤
│ Steps to Reproduce:                                         │
│ 1. Go to login page                                         │
│ 2. Enter email: test@example.com                            │
│ 3. Enter password: P@ss#word!                               │
│ 4. Click Login                                              │
├─────────────────────────────────────────────────────────────┤
│ Expected: User logs in successfully                         │
│ Actual: Error "Invalid credentials" displayed               │
├─────────────────────────────────────────────────────────────┤
│ Attachments: screenshot.png, console_log.txt                │
│ Reporter: qa@company.com    Date: 2024-12-20               │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Manual vs Automation Testing

### 7.1 When to Use Each

```
┌─────────────────────────────────────────────────────────────┐
│                    MANUAL TESTING                           │
├─────────────────────────────────────────────────────────────┤
│ ✓ Exploratory testing                                       │
│ ✓ Usability/UX evaluation                                   │
│ ✓ Ad-hoc testing                                           │
│ ✓ One-time tests                                           │
│ ✓ Visual validation                                        │
│ ✓ Complex scenarios hard to automate                        │
│ ✓ Short-term projects                                      │
│ ✓ Early development (unstable UI)                          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  AUTOMATION TESTING                         │
├─────────────────────────────────────────────────────────────┤
│ ✓ Regression testing                                        │
│ ✓ Load/Performance testing                                  │
│ ✓ Repetitive tests                                         │
│ ✓ Data-driven testing                                      │
│ ✓ Cross-browser/platform testing                           │
│ ✓ CI/CD pipeline integration                               │
│ ✓ Long-term projects                                       │
│ ✓ Stable features                                          │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Automation ROI Calculation

```
Break-even point = Automation Cost / (Manual Cost - Maintenance Cost)

Example:
- Manual test cost per run: $100
- Automation development: $2000
- Maintenance per run: $10
- Tests run per year: 50

ROI = (50 × $100 - $2000 - 50 × $10) / $2000 × 100%
ROI = ($5000 - $2000 - $500) / $2000 × 100%
ROI = 125%

Break-even: $2000 / ($100 - $10) = 22.2 runs
→ After 23 runs, automation starts saving money
```

---

## 8. Test Environment & Configuration

### 8.1 Environment Types

```
┌─────────────────────────────────────────────────────────────┐
│  DEV        →    QA/TEST    →    STAGING    →    PROD      │
│                                                             │
│  Developer      QA Team         Pre-prod         Live       │
│  testing        testing         validation       system     │
│                                                             │
│  Mock data      Test data       Prod-like        Real data  │
│  Local          Shared          Isolated         HA/DR      │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Test Data Management

**Strategies:**
1. **Production clone** (masked for privacy)
2. **Synthetic data generation**
3. **Subset of production**
4. **Manual creation**

**Data Masking Example:**
```
Original:   John Smith    john@email.com    4532-1234-5678-9012
Masked:     Xxxx Xxxxx    xxx@xxxxx.xxx     4532-XXXX-XXXX-XXXX
```

---

## 9. Quick Reference Tables

### 9.1 Test Types Summary

| Type | When | Who | Scope |
|------|------|-----|-------|
| Smoke | New build | Dev/QA | Critical paths |
| Sanity | Bug fix | QA | Changed module |
| Regression | Any change | QA/Auto | All/selected |
| Exploratory | New feature | QA | Undefined |
| UAT | Pre-release | Users | Business reqs |

### 9.2 SDLC Model Comparison

| Aspect | Waterfall | V-Model | Agile |
|--------|-----------|---------|-------|
| Testing phase | End | Parallel | Continuous |
| Flexibility | Low | Low | High |
| Documentation | Heavy | Heavy | Light |
| Customer involvement | Low | Low | High |
| Risk discovery | Late | Medium | Early |
| Best for | Fixed reqs | Critical systems | Evolving reqs |

### 9.3 Severity vs Priority

| | Critical | High | Medium | Low |
|---|---|---|---|---|
| **P1** | Fix now | Fix now | - | - |
| **P2** | Next release | Next release | Fix soon | - |
| **P3** | - | Backlog | Backlog | Backlog |
| **P4** | - | - | Nice to have | Nice to have |

---

## 10. Shift-Left Testing & DevSecOps (2025 Update)

### 10.1 Shift-Left Testing

**Concept:** Move testing earlier in the SDLC to find bugs when they're cheaper to fix.

```
Traditional:
Requirements → Design → Code → Test → Deploy
                              ↑
                         Testing here (expensive)

Shift-Left:
Requirements → Design → Code → Test → Deploy
     ↑            ↑       ↑
  Testing    Testing  Testing (cheaper)
```

**Cost of Defects:**

| Phase Found | Cost Multiplier |
|-------------|-----------------|
| Requirements | 1x |
| Design | 5x |
| Coding | 10x |
| Testing | 20x |
| Production | 100x+ |

**Shift-Left Practices:**

| Practice | Description |
|----------|-------------|
| **TDD** | Write tests before code |
| **BDD** | Define behavior with stakeholders first |
| **Static Analysis** | Lint/scan code during development |
| **Pair Programming** | Real-time code review |
| **CI/CD Testing** | Automated tests on every commit |

### 10.2 DevSecOps

**Concept:** Integrate security into DevOps pipeline (Security as Code).

```
┌─────────────────────────────────────────────────────────────┐
│                    DevSecOps Pipeline                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PLAN      CODE       BUILD      TEST      DEPLOY    MONITOR│
│    │         │          │          │          │          │  │
│    ▼         ▼          ▼          ▼          ▼          ▼  │
│ Threat    SAST      Dependency  DAST     Container   SIEM   │
│ Modeling  SCA       Scanning    Pentest  Scanning    Alerts │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Security Testing Types:**

| Type | When | Tools |
|------|------|-------|
| **SAST** (Static) | Code phase | SonarQube, Semgrep, CodeQL |
| **SCA** (Composition) | Build phase | Snyk, OWASP Dependency-Check |
| **DAST** (Dynamic) | Test phase | OWASP ZAP, Burp Suite |
| **IAST** (Interactive) | Runtime | Contrast Security |

**SAST Example (GitHub Actions):**
```yaml
name: Security Scan
on: [push, pull_request]

jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        uses: semgrep/semgrep-action@v1
        with:
          config: auto

      - name: Run CodeQL
        uses: github/codeql-action/analyze@v3
```

**SCA Example:**
```yaml
  dependency-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

### 10.3 Testing in CI/CD Pipeline

**Recommended Test Stages:**

```yaml
# .github/workflows/test.yml
name: Test Pipeline

on: [push]

jobs:
  # Stage 1: Fast feedback (< 5 min)
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - run: npm test

  # Stage 2: Integration (< 15 min)
  integration-tests:
    needs: unit-tests
    runs-on: ubuntu-latest
    steps:
      - run: npm run test:integration

  # Stage 3: Security (parallel)
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - run: npm audit
      - uses: snyk/actions/node@master

  # Stage 4: E2E (< 30 min)
  e2e-tests:
    needs: [unit-tests, integration-tests]
    runs-on: ubuntu-latest
    steps:
      - run: npx playwright test
```

**Test Stage Recommendations:**

| Stage | Tests | Time | Blocker |
|-------|-------|------|---------|
| Pre-commit | Lint, unit | < 30s | Yes |
| CI | Unit, integration | < 10min | Yes |
| Staging | E2E, performance | < 30min | Yes |
| Production | Smoke, monitoring | < 5min | Alert |

---

## 11. Common Exam Topics Checklist

```markdown
□ SDLC models (Waterfall, V-Model, Agile)
□ Test levels (Unit, Integration, System, Acceptance)
□ Test types (Smoke, Sanity, Regression, Exploratory)
□ Test documentation (Plan, Case, Report)
□ Black-box techniques (ECP, BVA, Decision Table)
□ White-box techniques (Statement, Branch, Path coverage)
□ Defect lifecycle and reporting
□ Test metrics and formulas
□ Manual vs Automation decision
□ Agile testing practices
□ Shift-left testing (TDD, BDD, early testing)
□ DevSecOps (SAST, SCA, DAST integration)
```

---

*Good luck với kỳ thi! 🎓*
