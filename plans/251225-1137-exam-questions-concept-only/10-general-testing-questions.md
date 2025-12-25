# General Software Testing Concepts - Concept Questions

## Question 1: Software Development Lifecycle and Testing (15 points)

### Scenario
A startup is developing a new e-commerce platform and needs to decide on a development methodology.

### Questions

**a) (5 points)** Compare the V-Model and Agile approaches to software development and testing.

**Expected Answer:**

| Aspect | V-Model | Agile |
|--------|---------|-------|
| **Structure** | Sequential, phases in V shape | Iterative, short sprints |
| **Testing timing** | Each phase has corresponding test phase | Testing continuous throughout |
| **Documentation** | Heavy upfront documentation | Working software over docs |
| **Flexibility** | Changes difficult after phase complete | Changes welcomed, even late |
| **Test planning** | Done early, comprehensive | Just-in-time, evolving |
| **Delivery** | Single delivery at end | Incremental delivery |
| **Best for** | Stable requirements, regulated industries | Evolving requirements, startups |

**V-Model visualization:**
```
Requirements ─────────────────────────────────┐ Acceptance Testing
     │                                         │
Functional Spec ───────────────────────────┐ │ System Testing
        │                                   │ │
    Design ─────────────────────────────┐ │ │ Integration Testing
         │                               │ │ │
      Coding ─────────────────────────────────── Unit Testing
                                         │ │ │
                         (Development)   ▼ ▼ ▼ (Testing)
```

**Agile visualization:**
```
Sprint 1    Sprint 2    Sprint 3    Sprint 4
┌─────┐     ┌─────┐     ┌─────┐     ┌─────┐
│Plan │     │Plan │     │Plan │     │Plan │
│Build│ ──▶ │Build│ ──▶ │Build│ ──▶ │Build│
│Test │     │Test │     │Test │     │Test │
│Ship │     │Ship │     │Ship │     │Ship │
└─────┘     └─────┘     └─────┘     └─────┘
   ↓           ↓           ↓           ↓
 Working    Working    Working    Working
 Product    Product    Product    Product
            (improved) (improved) (improved)
```

**b) (5 points)** What are the different levels of testing and their purposes?

**Expected Answer:**

| Level | Scope | Who Tests | Purpose |
|-------|-------|-----------|---------|
| **Unit Testing** | Individual functions/methods | Developers | Verify code works correctly |
| **Integration Testing** | Multiple components together | Dev/QA | Verify components interact correctly |
| **System Testing** | Complete system | QA Team | Verify system meets requirements |
| **Acceptance Testing** | User perspective | Users/Stakeholders | Verify system meets business needs |

**Testing pyramid:**
```
              /\
             /  \    E2E Tests (few, slow, expensive)
            /────\
           /      \   Integration Tests (moderate)
          /────────\
         /          \  Unit Tests (many, fast, cheap)
        /────────────\
```

**c) (5 points)** Explain shift-left and shift-right testing strategies.

**Expected Answer:**

**Shift-left:** Move testing earlier in development

| Practice | Description | Benefit |
|----------|-------------|---------|
| **TDD** | Write tests before code | Bugs prevented, not found |
| **Static analysis** | Analyze code without running | Find issues at commit time |
| **Unit testing** | Developers test own code | Immediate feedback |
| **Requirements review** | Test requirements for testability | Fewer defects in design |

**Shift-right:** Testing continues after deployment

| Practice | Description | Benefit |
|----------|-------------|---------|
| **A/B testing** | Compare versions in production | Real user validation |
| **Canary releases** | Gradual rollout | Limit blast radius |
| **Production monitoring** | Track real usage | Find issues users experience |
| **Chaos engineering** | Intentionally cause failures | Improve resilience |

**Visualization:**
```
Development                              Production
─────────────────────────────────────────────────────────▶

Traditional:        │ Testing │
                ┌───┴─────────┴───┐

Shift-left: │ Testing────────────│
        ┌───┴────────────────────┴───┐

Shift-right:         │──────────Testing │
                 ┌───┴──────────────────┴───┐

Both:      │ Testing────────────────Testing │
       ┌───┴────────────────────────────────┴───┐
```

---

## Question 2: Test Design and Planning (15 points)

### Scenario
You are creating a test plan for ToolShop's new checkout feature.

### Questions

**a) (5 points)** What are the key components of a test plan?

**Expected Answer:**

| Component | Description | Example for ToolShop Checkout |
|-----------|-------------|------------------------------|
| **Objectives** | What testing will achieve | Verify checkout flow, payment processing |
| **Scope** | What's in/out of testing | In: Web checkout; Out: Mobile app |
| **Test items** | Features to be tested | Cart, shipping, payment, confirmation |
| **Approach** | Testing strategy and types | Functional, performance, security |
| **Pass/fail criteria** | When testing is complete | 0 critical bugs, >90% cases pass |
| **Resources** | People, tools, environments | 2 QA, Playwright, staging environment |
| **Schedule** | Timeline and milestones | 2 weeks testing, 1 week regression |
| **Risks** | Potential issues and mitigation | Payment gateway downtime - use mocks |
| **Deliverables** | Test artifacts produced | Test cases, results, bug reports |

**b) (5 points)** What is risk-based testing and how do you prioritize tests?

**Expected Answer:**

**Risk-based testing:** Prioritize testing based on probability and impact of failures.

**Risk assessment matrix:**

| | Low Impact | High Impact |
|---|------------|-------------|
| **High Probability** | Medium priority | **Highest priority** |
| **Low Probability** | Low priority | Medium priority |

**Prioritization factors:**

| Factor | High Priority | Low Priority |
|--------|--------------|--------------|
| **Business criticality** | Checkout, payment | About page |
| **User frequency** | Common flows | Edge cases |
| **Complexity** | Complex logic | Simple CRUD |
| **Change frequency** | Recently modified | Stable code |
| **Historical bugs** | Previously buggy areas | Reliable areas |

**c) (5 points)** Explain the difference between smoke testing and sanity testing.

**Expected Answer:**

| Aspect | Smoke Testing | Sanity Testing |
|--------|---------------|----------------|
| **Purpose** | Verify build is stable enough for testing | Verify specific functionality after change |
| **Scope** | Broad, shallow | Narrow, focused |
| **When** | After new build/deployment | After bug fix or minor change |
| **Depth** | Surface-level checks | Deeper but limited area |
| **Also called** | Build verification test | Confidence testing |

**Example workflow:**
```
New build deployed
        │
        ▼
┌───────────────┐
│ Smoke Tests   │ ─── Fail ───▶ Reject build
│ - App starts  │               (don't waste time testing)
│ - Login works │
│ - Main pages  │
└───────┬───────┘
        │ Pass
        ▼
┌───────────────┐
│ Full Testing  │
└───────┬───────┘
        │ Bug found and fixed
        ▼
┌───────────────┐
│ Sanity Tests  │ ─── Verify fix works
│ - Fixed area  │     and didn't break
│ - Related     │     related functionality
│   features    │
└───────────────┘
```

---

## Question 3: Test Documentation and Traceability (15 points)

### Scenario
ToolShop needs to maintain traceability between requirements, test cases, and defects.

### Questions

**a) (5 points)** What is test traceability and why is it important?

**Expected Answer:**

**Test traceability:** Ability to link requirements to test cases to defects to code.

**Traceability matrix example:**

| Requirement | Test Cases | Status | Defects |
|-------------|------------|--------|---------|
| REQ-001: User login | TC-001, TC-002, TC-003 | Pass | - |
| REQ-002: Add to cart | TC-010, TC-011, TC-012 | Fail | BUG-045 |
| REQ-003: Checkout | TC-020, TC-021 | Pass | - |

**Why important:**

| Benefit | Explanation |
|---------|-------------|
| **Coverage visibility** | Know all requirements are tested |
| **Impact analysis** | When requirement changes, know which tests affected |
| **Defect tracking** | Link bugs back to requirements |
| **Compliance** | Prove testing coverage for audits |
| **Reporting** | Show progress per requirement |

**b) (5 points)** What makes a good test case?

**Expected Answer:**

**Good test case characteristics:**

| Characteristic | Description | Example |
|----------------|-------------|---------|
| **Clear ID** | Unique identifier | TC-CHECKOUT-001 |
| **Descriptive title** | What it tests | "Verify credit card payment succeeds" |
| **Preconditions** | State before test | "User logged in, cart has items" |
| **Test steps** | Clear actions | "1. Click checkout 2. Enter card..." |
| **Expected result** | What should happen | "Order confirmation displayed" |
| **Actual result** | What happened (for results) | "Order confirmed, email received" |
| **Independent** | Can run standalone | Doesn't depend on other tests |
| **Repeatable** | Same result each time | No random factors |

**Test case template:**
```
┌─────────────────────────────────────────────────────┐
│ Test Case ID: TC-CHECKOUT-001                       │
│ Title: Verify successful credit card payment        │
├─────────────────────────────────────────────────────┤
│ Preconditions:                                      │
│   - User is logged in                               │
│   - Cart contains at least one item                 │
│   - User has valid shipping address saved           │
├─────────────────────────────────────────────────────┤
│ Test Steps:                                         │
│   1. Navigate to checkout                           │
│   2. Select saved shipping address                  │
│   3. Enter valid credit card (4111...)              │
│   4. Click "Place Order"                            │
├─────────────────────────────────────────────────────┤
│ Expected Result:                                    │
│   - Order confirmation page displayed               │
│   - Order number generated                          │
│   - Confirmation email received                     │
│   - Cart emptied                                    │
├─────────────────────────────────────────────────────┤
│ Priority: High  │ Status: Ready  │ Automated: Yes  │
└─────────────────────────────────────────────────────┘
```

**c) (5 points)** How should test results and defects be documented?

**Expected Answer:**

**Test execution documentation:**

| Element | Description |
|---------|-------------|
| **Test run ID** | Unique identifier for test execution |
| **Environment** | Browser, OS, build version |
| **Executor** | Who ran the test |
| **Date/time** | When executed |
| **Status** | Pass/Fail/Blocked/Skipped |
| **Evidence** | Screenshots, logs |
| **Notes** | Observations, workarounds |

**Defect report components:**

| Field | Purpose | Example |
|-------|---------|---------|
| **ID** | Unique tracking | BUG-123 |
| **Title** | Clear problem statement | "Checkout fails for international addresses" |
| **Severity** | Impact level | Critical/High/Medium/Low |
| **Priority** | Fix urgency | P1 (immediate) to P4 (backlog) |
| **Environment** | Where reproduced | Chrome 120, Windows 11, Staging |
| **Steps to reproduce** | Exact sequence | 1. Login 2. Add item... |
| **Expected vs Actual** | Gap | Should succeed / Shows error |
| **Attachments** | Evidence | Screenshot, video, logs |
| **Related test case** | Traceability | TC-CHECKOUT-005 |

---

## Question 4: Testing Metrics and Reporting (15 points)

### Scenario
Management wants to see the status of testing for ToolShop's release.

### Questions

**a) (5 points)** What metrics should be tracked during testing?

**Expected Answer:**

| Metric | Formula | Purpose |
|--------|---------|---------|
| **Test case execution** | Executed / Total | Progress tracking |
| **Pass rate** | Passed / Executed | Quality indicator |
| **Defect density** | Defects / KLOC | Code quality |
| **Defect detection rate** | Found / (Found + Escaped) | Testing effectiveness |
| **Test coverage** | Lines tested / Total lines | Coverage completeness |
| **Defect leakage** | Production bugs / Total bugs | Quality gate effectiveness |
| **MTTR** | Time to fix defects | Development responsiveness |

**b) (5 points)** How do you communicate testing status effectively?

**Expected Answer:**

**Dashboard approach:**
```
┌─────────────────────────────────────────────────────┐
│           Test Execution Summary                    │
├─────────────────────────────────────────────────────┤
│ Overall Progress:  ████████████░░░░░░  78%         │
│                                                     │
│ Test Cases:   Passed: 156  Failed: 12  Blocked: 8  │
│                                                     │
│ Defects by Severity:                                │
│   Critical: 2 ⚠️   High: 5   Medium: 12   Low: 8   │
│                                                     │
│ Trend (last 5 days):                                │
│   Pass rate: 85% → 87% → 88% → 89% → 92% ↑        │
│                                                     │
│ Release Readiness: ⚠️ AT RISK                      │
│   Reason: 2 critical defects open                   │
└─────────────────────────────────────────────────────┘
```

**Report types:**

| Audience | Report Focus | Frequency |
|----------|--------------|-----------|
| **Team** | Detailed progress, blockers | Daily |
| **PM** | Overall status, risks, milestones | Weekly |
| **Executives** | Go/no-go summary, major issues | Release points |

**c) (5 points)** What is test exit criteria and how is it defined?

**Expected Answer:**

**Exit criteria:** Conditions that must be met to end testing phase.

**Example criteria:**

| Criterion | Target | Actual | Status |
|-----------|--------|--------|--------|
| Test case execution | 100% | 98% | ⚠️ |
| Pass rate | ≥95% | 92% | ❌ |
| Critical defects | 0 open | 0 | ✅ |
| High defects | ≤2 open | 3 | ❌ |
| Code coverage | ≥80% | 82% | ✅ |
| Performance | P95 < 500ms | 450ms | ✅ |

**Exit decision process:**
```
All criteria met?
        │
    ┌───┴───┐
   Yes      No
    │        │
    ▼        ▼
 Release   Exception process
            │
      ┌─────┴─────┐
      ▼           ▼
   Defer      Accept risk
   release    (documented)
```

---

## Question 5: Testing Types and Techniques (20 points)

### Scenario
ToolShop needs comprehensive testing strategy covering various testing types.

### Questions

**a) (5 points)** Explain functional vs non-functional testing.

**Expected Answer:**

| Aspect | Functional Testing | Non-Functional Testing |
|--------|-------------------|----------------------|
| **Focus** | What system does | How system performs |
| **Question** | Does feature work? | Is it fast/secure/usable? |
| **Requirements** | Business requirements | Quality attributes |
| **Examples** | Login, checkout, search | Performance, security, usability |
| **Pass/fail** | Feature works or not | Meets threshold or not |

**Non-functional categories:**

| Type | Tests For | Metric |
|------|-----------|--------|
| **Performance** | Speed, responsiveness | Response time, throughput |
| **Security** | Vulnerability, access control | Vulnerabilities found, compliance |
| **Usability** | User experience | Task completion, satisfaction |
| **Reliability** | Consistency, uptime | MTBF, availability |
| **Compatibility** | Works across platforms | Browsers, devices supported |

**b) (5 points)** What is regression testing and how should it be managed?

**Expected Answer:**

**Regression testing:** Verifying that changes haven't broken existing functionality.

**When to run:**

| Trigger | Regression Scope |
|---------|------------------|
| Bug fix | Related area + smoke |
| New feature | Affected areas + critical paths |
| Release candidate | Full regression suite |
| Infrastructure change | Full regression suite |

**Management strategies:**

| Strategy | Description |
|----------|-------------|
| **Test selection** | Run only tests for changed areas |
| **Test prioritization** | Critical tests first |
| **Automation** | Automate regression suite |
| **Continuous testing** | Run subset on every commit |
| **Scheduled full runs** | Complete suite nightly/weekly |

**Regression test pyramid:**
```
                    /\
                   /  \  Full Suite (weekly)
                  /    \
                 /──────\
                /        \  Affected Area (PR merge)
               /          \
              /────────────\
             /              \  Smoke Suite (every commit)
            /────────────────\
```

**c) (5 points)** Explain black-box vs white-box vs gray-box testing.

**Expected Answer:**

| Aspect | Black-Box | White-Box | Gray-Box |
|--------|-----------|-----------|----------|
| **Access** | External only | Full code access | Partial access |
| **Tester knows** | Requirements, interface | Code, architecture | Some internals |
| **Focus** | Behavior | Code coverage | Integration points |
| **Who performs** | QA testers | Developers | Dev or QA |
| **Techniques** | ECP, BVA, decision tables | Statement, branch, path coverage | API testing, integration |

**Visualization:**
```
Black-box:           Gray-box:           White-box:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  ?????????  │     │ ╔═════════╗ │     │ if(x > 0) { │
│  ?????????  │     │ ║ Known   ║ │     │   return x; │
│  ?????????  │     │ ║ interface║ │     │ } else {    │
│             │     │ ╚═════════╝ │     │   return 0; │
└─────────────┘     └─────────────┘     └─────────────┘
   Input/Output        Some knowledge      Full visibility
      only              of internals
```

**d) (5 points)** What is exploratory testing and when is it valuable?

**Expected Answer:**

**Exploratory testing:** Simultaneous learning, test design, and test execution.

**Characteristics:**

| Aspect | Scripted Testing | Exploratory Testing |
|--------|------------------|---------------------|
| **Test design** | Upfront | During execution |
| **Documentation** | Detailed steps | Session notes |
| **Freedom** | Follow script | Tester judgment |
| **Coverage** | Predetermined | Adaptive |
| **Repeatability** | Exact | Variable |

**When valuable:**

| Scenario | Why Exploratory Helps |
|----------|----------------------|
| **New feature** | Discover unexpected behaviors |
| **Limited time** | Quick assessment of quality |
| **Complex system** | Find integration issues |
| **After automation** | Catch what scripts miss |
| **User perspective** | Test like real user would |

**Session-based approach:**
```
Charter: "Explore checkout with unusual product combinations"
Time-box: 60 minutes

Session notes:
- 00:05 - Added 99 items of same product - slow but works
- 00:15 - Mixed digital and physical - shipping confusion
- 00:25 - BUG: Can add out-of-stock item via quick-add
- 00:40 - Discount codes tested, one expired code accepted
- 00:55 - Session summary, bugs logged

Outcome: 2 bugs found, 3 areas need deeper testing
```

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | SDLC and Testing | 15 |
| Q2 | Test Design and Planning | 15 |
| Q3 | Documentation and Traceability | 15 |
| Q4 | Metrics and Reporting | 15 |
| Q5 | Testing Types and Techniques | 20 |
| **Total** | | **80** |
