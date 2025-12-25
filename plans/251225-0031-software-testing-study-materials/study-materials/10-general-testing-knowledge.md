# Topic 10: General Software Testing Knowledge
**CS423 Software Testing | Final Exam Preparation**

---

## 0. Fundamental Testing Concepts (Core Theory)

### 0.1 What is Software Testing?

**ISTQB Definition:**
> Testing is the process consisting of all lifecycle activities, both static and dynamic, concerned with planning, preparation and evaluation of software products and related work products to determine that they satisfy specified requirements, to demonstrate that they are fit for purpose and to detect defects.

**Key Objectives of Testing:**
1. **Detect Faults** - Primary goal: find as many problems as possible
2. **Establish Confidence** - Demonstrate software is fit for purpose
3. **Evaluate Quality Properties** - Reliability, Performance, Security, Usability
4. **Prevent Defects** - Through early testing and reviews

> *"Testing can show the presence of bugs, but not the absence"* - Dijkstra

**Important Mindset:**
- The purpose of finding problems is to GET THEM FIXED
- The best tester is NOT the one who finds the most bugs
- The best tester is the one who gets the most bugs FIXED

### 0.2 Error → Fault → Failure

```
┌─────────────────────────────────────────────────────────────────┐
│   A person makes an ERROR                                        │
│              ↓                                                   │
│   That creates a FAULT (Defect/Bug) in the software             │
│              ↓                                                   │
│   That can cause a FAILURE in operation                          │
└─────────────────────────────────────────────────────────────────┘
```

| Term | Definition | Example |
|------|------------|---------|
| **Error** | Human mistake/action | Misunderstanding requirements |
| **Fault (Bug)** | Defect in code/design | Missing validation check |
| **Failure** | Incorrect behavior at runtime | System crash, wrong output |

### 0.3 Testing vs Debugging

| Aspect | Testing | Debugging |
|--------|---------|-----------|
| **Goal** | Find input that causes failure | Find fault given a failure |
| **Who** | Testers | Developers |
| **Process** | Systematic execution | Investigation & correction |
| **Output** | Bug reports | Code fixes |

### 0.4 Static vs Dynamic Testing

**Static Testing** (Code NOT executed):
- Code review, inspection, walkthrough
- Static analysis tools
- Finds defects early, cheaper
- Examples: Linting, SonarQube, peer review

**Dynamic Testing** (Code IS executed):
- Execute program with test inputs
- Compare actual vs expected results
- Examples: Unit tests, integration tests, system tests

### 0.5 Verification vs Validation

```
                     User Requirements
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ↓              ↓              ↓
    Verification     Software      Validation
    "Building it              "Building the
     correctly?"               right thing?"
            │                        │
            └────────────────────────┘
```

| Verification | Validation |
|--------------|------------|
| Are we building the product RIGHT? | Are we building the RIGHT product? |
| Check against specifications | Check against user needs |
| Reviews, inspections, walkthroughs | Testing, UAT, demos |
| Static methods | Dynamic methods |

### 0.6 Test Oracle

> A **Test Oracle** is a source of information about whether the output of a program is correct or not.

```
Expected Result ──┐
                  ├──> Compare ──> Pass/Fail
Actual Result ────┘
```

**Types of Test Oracles:**
- Specification-based (expected from requirements)
- Previous version comparison
- Human judgment (for subjective qualities)
- Statistical oracles (within acceptable range)

### 0.7 Test Bed (Test Environment)

The test execution environment configured for testing:
- Hardware configuration
- Software/OS versions
- Network configuration
- Application Under Test (AUT)
- Test data and databases
- Supporting applications

---

## 0.8 Seven Principles of Software Testing (ISTQB)

```
┌─────────────────────────────────────────────────────────────────┐
│            THE 7 PRINCIPLES OF SOFTWARE TESTING                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Testing shows the PRESENCE of defects                        │
│     → Cannot prove absence, only reduce probability              │
│                                                                  │
│  2. EXHAUSTIVE testing is impossible                             │
│     → Need optimal testing based on risk assessment              │
│                                                                  │
│  3. EARLY testing saves time and money                           │
│     → Defects cheaper to fix in early stages                     │
│                                                                  │
│  4. Defects CLUSTER together                                     │
│     → 80% of defects found in 20% of modules (Pareto)           │
│                                                                  │
│  5. PESTICIDE paradox                                            │
│     → Same tests become ineffective, must update regularly      │
│                                                                  │
│  6. Testing is CONTEXT dependent                                 │
│     → Different approaches for different applications           │
│                                                                  │
│  7. ABSENCE of errors fallacy                                    │
│     → 99% bug-free can still be unusable if wrong product       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Principle Details:**

| # | Principle | Implication |
|---|-----------|-------------|
| 1 | Presence of defects | Testing reduces undiscovered defects, cannot guarantee zero |
| 2 | Exhaustive impossible | Use risk-based testing, prioritization |
| 3 | Early testing | Shift-left, test from requirements phase |
| 4 | Defect clustering | Focus efforts on problematic modules |
| 5 | Pesticide paradox | Regularly review and update test cases |
| 6 | Context dependent | Safety-critical vs e-commerce need different approaches |
| 7 | Absence of errors fallacy | Must verify software meets business needs |

---

## 0.9 QA vs QC vs Testing (ISO 9000)

### ISO 9000 Definitions:

**Quality Control (QC):**
> The operational techniques and activities used to fulfill requirements for quality

**Quality Assurance (QA):**
> All planned and systematic activities implemented to provide adequate confidence that an entity will fulfill requirements for quality

### Detailed Comparison:

| Aspect | QA | QC | Testing |
|--------|----|----|---------|
| **Relationship** | Subset of SDLC | Subset of QA | Subset of QC |
| **Focus** | PROCESS oriented | PRODUCT oriented | PRODUCT oriented |
| **Nature** | Proactive (prevent) | Reactive (detect) | Reactive (detect) |
| **Function** | Staff function | Line function | Line function |
| **Objective** | Prevent defects | Find defects | Validate product |
| **Scope** | Entire project lifecycle | Product quality | Specific testing |

### Examples:

| QA Activities | QC Activities |
|---------------|---------------|
| Quality Audit | Walkthrough |
| Defining Process | Testing |
| Selection of tools | Inspection |
| Training | Checkpoint Review |

### Process Flow:

```
┌─────────────────────────────────────────────────────────────────┐
│                    QC Process (Problem Management)               │
│                                                                  │
│  Problem Identification → Problem Analysis → Problem Correction  │
│                                  ↓                               │
│                          Feedback to QA                          │
└──────────────────────────────────┬──────────────────────────────┘
                                   ↓
┌──────────────────────────────────┴──────────────────────────────┐
│                    QA Process (Process Improvement)              │
│                                                                  │
│  Input from QC → Data Gathering → Problem Trend Analysis        │
│        ↓                                                         │
│  Process Identification → Process Analysis → Process Improvement │
└─────────────────────────────────────────────────────────────────┘
```

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

## 1.5 Software Testing Life Cycle (STLC)

### STLC Phases with Entry/Exit Criteria:

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                    SOFTWARE TESTING LIFE CYCLE (STLC)                         │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                               │
│  1. Requirement    2. Test         3. Test Case    4. Environment            │
│     Analysis   →     Planning  →     Design    →     Setup                   │
│                                                          ↓                    │
│                                  6. Test Cycle   ←   5. Test                 │
│                                     Closure          Execution               │
│                                                                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Requirement Analysis

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Requirement Specification | Analyze requirements for scope | Requirements Traceability Matrix |
| Application Architecture | Consult stakeholders | Automation Feasibility Report |
| | Identify risks/challenges | |

### Phase 2: Test Planning

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Project Plan | Define test objectives & scope | **Test Plan** |
| Acceptance Criteria | Develop test strategy | |
| Requirement Specification | Estimate time and cost | |
| | Assign roles/responsibilities | |

### Phase 3: Test Case Design

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Test Plan | Design test cases | Test Cases |
| Requirement Specification | Generate test data | Test Data |
| | Create automation scripts | Automation Scripts |
| | Update RTM | |

### Phase 4: Environment Setup

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Test Plan | Prepare hardware/software list | Functional Test Environment |
| Test Cases | Configure and deploy environment | Smoke Test Results |
| System Architecture | Perform smoke tests | |

### Phase 5: Test Execution

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Test Cases | Run test cases | Test Results |
| Test Data | Compare actual vs expected | Defect Reports |
| Automation Script | Report defects | |

### Phase 6: Test Cycle Closure

| Entry Criteria | Activities | Deliverables |
|----------------|------------|--------------|
| Test Cases | Ensure all activities completed | Test Summary Report |
| Test Results | Document lessons learned | Test Closure Report |
| Defect Reports | Prepare closure reports | |

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

### Stubs vs Drivers:

```
┌─────────────────────────────────────────────────────────────────┐
│  DRIVER: Simulates calling unit (replaces higher-level module)  │
│  STUB: Simulates called unit (replaces lower-level module)      │
└─────────────────────────────────────────────────────────────────┘

    [Driver] ──calls──> [Unit Under Test] ──calls──> [Stub]
```

| Component | Purpose | Returns |
|-----------|---------|---------|
| **Stub** | Replace lower-level module | Static value, input value |
| **Driver** | Replace higher-level module | Calls the unit under test |

### Integration Approaches:

**Big-Bang Integration:**
```
Integrate ALL units at once → Test as single unit
```
- Advantages: Suitable for small systems, fast
- Disadvantages: Hard to locate defects, high-risk modules not prioritized

**Top-Down Integration:**
```
    [Module A] ←──── Test first (with stubs for B, C)
       ↓
    [Module B] ←──── Replace stub, test (with stub for D)
       ↓
    [Module C]
```
- Advantages: Early prototype possible, critical design flaws found early
- Disadvantages: Lower modules tested late, many stubs needed

**Bottom-Up Integration:**
```
    [Module D] ←──── Test first (with driver)
       ↑
    [Module B] ←──── Replace driver, test
       ↑
    [Module A]
```
- Advantages: Easier fault isolation, no stubs needed
- Disadvantages: No early prototype, critical modules tested late

**Sandwich Integration:**
- Combine Top-Down and Bottom-Up
- Test middle layer with both drivers and stubs

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

---

## 5.5 Code Coverage (Detailed)

### What is Code Coverage?
- Software testing metric
- Determines number of lines validated under test procedure
- **Developers**: Dead code detection and elimination
- **QA Engineers**: Check missed or uncovered test cases

### Why Code Coverage?
- Easy maintenance of code base
- Exposure of bad code
- Faster time to market

### Coverage Types Hierarchy:

```
┌─────────────────────────────────────────────────────────────────┐
│           CODE COVERAGE HIERARCHY (Strength)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│    Multiple Condition Coverage    ←── Strongest (most tests)    │
│              ↑                                                   │
│    MC/DC (Modified Condition/Decision)                          │
│              ↑                                                   │
│    Condition/Decision Coverage                                   │
│              ↑                                                   │
│    Path Coverage                                                 │
│              ↑                                                   │
│    Decision/Branch Coverage                                      │
│              ↑                                                   │
│    Statement Coverage             ←── Weakest (fewest tests)    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Coverage Types Explained:

**1. Function Coverage**
- Has each function/subroutine been called?

**2. Statement Coverage**
- Has each node/statement been executed?
- Weakest coverage criterion

```c
void foo(int a, int b, int x) {
    if ((a > 1) && (b == 0))
        x = x / a;        // Statement 1
    if (a == 2 || x > 1)
        x = x + 1;        // Statement 2
}

// Test: a=2, b=0, x=4 → Both statements executed → 100% statement coverage
```

**3. Decision/Branch Coverage**
- Has every edge (true/false branch) been executed?
- Decision coverage ⊇ Statement coverage

```c
// Same code
// Test 1: a=2, b=0, x=4  (both conditions true)
// Test 2: a=0, b=0, x=0  (both conditions false)
// → 100% Decision coverage
```

**4. Condition Coverage**
- Has each Boolean sub-expression evaluated to both true AND false?
- Does NOT necessarily imply decision coverage

```c
// For: if ((a > 1) && (b == 0))
// Test 1: a=2, b=0  → (a>1)=T, (b==0)=T
// Test 2: a=1, b=1  → (a>1)=F, (b==0)=F
// → 100% Condition coverage but NOT 100% Decision coverage!
```

**5. Condition/Decision Coverage**
- Both decision AND condition coverage satisfied

**6. Path Coverage**
- Has every possible path been executed?
- Path coverage ⊇ Decision coverage

```c
// 4 possible paths through the code:
// Test 1: a=2, b=0, x=4  → Path: S1 executed, S2 executed
// Test 2: a=0, b=0, x=0  → Path: S1 skipped, S2 skipped
// Test 3: a=2, b=1, x=4  → Path: S1 skipped, S2 executed
// Test 4: a=3, b=0, x=0  → Path: S1 executed, S2 skipped
```

**7. Modified Condition/Decision Coverage (MC/DC)**
- For safety-critical applications (aviation, medical)
- Each condition must independently affect the decision outcome

```
For: if (a OR b) AND c

Standard C/D Coverage (2 tests):
  a=T, b=T, c=T → Result=T
  a=F, b=F, c=F → Result=F

MC/DC requires each variable to independently affect outcome:
  a=F, b=F, c=T → Result=F
  a=T, b=F, c=T → Result=T  ← 'a' independently affects result
  a=F, b=T, c=T → Result=T  ← 'b' independently affects result
  a=T, b=T, c=F → Result=F  ← 'c' independently affects result
```

**8. Multiple Condition Coverage**
- ALL combinations of conditions tested
- Most thorough, most expensive

```
For: if (a OR b) AND c  (3 variables)
→ 2³ = 8 test cases required:
  a=F, b=F, c=F
  a=F, b=F, c=T
  a=F, b=T, c=F
  a=F, b=T, c=T
  a=T, b=F, c=F
  a=T, b=F, c=T
  a=T, b=T, c=F
  a=T, b=T, c=T
```

### Control Flow Graph & Cyclomatic Complexity

**Basic Path Testing Process:**
1. Derive Control Flow Graph
2. Compute Cyclomatic Complexity: **C = Edges - Nodes + 2**
3. Select C basis paths
4. Create test case for each basis path

```c
// Example:
if (a > 0) {
    x = x + 1;      // Node B
}
if (b == 3) {
    y = 0;          // Node C
}

// CFG: 6 edges, 5 nodes
// Cyclomatic Complexity = 6 - 5 + 2 = 3
// 3 basis paths needed:
// Path 1: A→D (a≤0, b≠3)
// Path 2: A→B→D (a>0, b≠3)
// Path 3: A→C→D (a≤0, b==3)
```

### Loop Testing

**Simple Loop Testing (n = max iterations):**
1. Skip loop entirely (0 iterations)
2. Only one pass (1 iteration)
3. Two passes (2 iterations)
4. m passes (where m < n)
5. n-1, n, n+1 passes

**Nested Loops:**
- Start at innermost loop, set outer loops to minimum
- Test innermost with simple loop tests
- Work outward, keeping inner loops at typical values

**Concatenated Loops:**
- If independent: Test as simple loops
- If dependent: Test as nested loops

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

### 6.3 Bug Report Essentials (10 Key Elements)

1. **Bug ID** - Unique identifier
2. **Function Name** - Module where bug found (Login, Checkout, etc.)
3. **Problem Summary** - Test Objective + Actual Result (vs Expected)
4. **How to Reproduce** - Detailed steps with screenshots
5. **Reported By** - Tester name/ID
6. **Date** - When defect reported
7. **Assign To** - Developer who will fix
8. **Status** - New/Closed/Reopened/Rejected/Deferred/In-progress/Fixed
9. **Priority** - Fixing urgency (schedule-related)
10. **Severity** - Impact on application (technical impact)

### 6.4 Severity vs Priority (CRITICAL DISTINCTION)

**Severity** = Impact on APPLICATION (Technical)
**Priority** = Urgency of FIXING (Business/Schedule)

#### Severity Levels:

| Severity | Description | Example |
|----------|-------------|---------|
| **Fatal** | Great damage, system crash, data loss | System crashes on login |
| **Serious** | Impacts main features | Can delete without authentication |
| **Medium** | Minimal deviation from requirements | UI misalignment on mobile |
| **Cosmetic** | Very minor, not affecting operation | Wrong tab order, missing shortcut |

#### Priority Levels:

| Priority | Description | Timeframe |
|----------|-------------|-----------|
| **Immediate/Critical** | Fix NOW, great damage risk | 1 day |
| **High** | Impacts main features | 2-4 days |
| **Medium** | Minimal deviation | 5-8 days |
| **Low** | Very minor affect | Fix later |

#### Severity vs Priority Matrix:

```
                        PRIORITY
                 High              Low
           ┌─────────────────┬─────────────────┐
     High  │ Critical bug    │ Rare edge case  │
           │ Fix immediately │ but serious     │
SEVERITY   ├─────────────────┼─────────────────┤
           │ Visible issue   │ Cosmetic issue  │
     Low   │ on main page    │ on rare screen  │
           │ (CEO sees it!)  │ Fix later       │
           └─────────────────┴─────────────────┘
```

**Examples:**
- **High Severity, Low Priority**: Crash on admin-only feature (rarely used)
- **Low Severity, High Priority**: Company logo wrong on homepage (CEO noticed!)

### 6.5 Good Bug Report Characteristics

- Written, Numbered
- Simple, Understandable
- Reproducible
- Legible, Non-judgmental

### 6.6 Bad Bug Report Examples:

| Bad | Better |
|-----|--------|
| "It does not work!" | "Error 404: Access denied when clicking Submit" |
| "I just clicked and it crashes" | "Error 404: Page not found when clicking Export" |
| "Windows" | "Windows 11, Chrome 120.0.1132.47" |
| "System is really slow" | "System response time 5 minutes instead of 3 seconds" |
| "Error message is stupid" | "Error message is unclear: 'Error Code 500'" |

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

## 7.5 Test Management (COMPREHENSIVE)

### 7.5.1 Independent Testing

**Definition:** Degree of separation between tester and developer

**Five Degrees of Independence:**

| Level | Description | Independence |
|-------|-------------|--------------|
| 1 | Developer tests own code | None |
| 2 | Developer/tester within dev team | Low |
| 3 | Independent test team in organization | Medium |
| 4 | Testers from business/user community | High |
| 5 | External testers (outsourcing) | Highest |

**Benefits of Independence:**
- Recognize different kinds of failures
- Verify, challenge, or reject stakeholder assumptions
- Different cognitive biases from developers

**Drawbacks:**
- Isolation from development team
- Developers may lose sense of responsibility for quality
- May be seen as bottleneck or blamed for delays

### 7.5.2 Test Roles and Responsibilities

#### Test Manager Tasks:
- **Test Policy**: Organization-wide testing rules
- **Test Strategy**: High-level testing approach
- **Test Plan**: Project-specific testing document
- **Test Monitoring & Control**: Progress reports, summary reports
- **Metrics**: Define and track quality metrics
- **Tools Selection**: Choose appropriate test tools
- **Environment Decisions**: Test environment setup
- **Develop Team**: Skill up testers, career development

#### Tester Tasks:
- Review and contribute to test plans
- Assess requirements for testability
- Create test conditions, cases, procedures, data
- Test environment setup
- Test execution
- Test automation
- Non-functional testing
- Review tests developed by others

### 7.5.3 Test Planning (IEEE 829 Steps)

```
┌─────────────────────────────────────────────────────────────────┐
│                    8 STEPS OF TEST PLANNING                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Analyze the Product                                     │
│     • Who will use it?                                          │
│     • What is it used for?                                      │
│     • How will it work?                                         │
│     • What hardware/software required?                          │
│                                                                  │
│  Step 2: Develop Test Strategy                                   │
│     • 2.1 Define Scope of Testing (in-scope vs out-scope)       │
│     • 2.2 Identify Testing Types                                │
│     • 2.3 Document Risk & Issues                                │
│     • 2.4 Create Test Logistics (who, when)                     │
│                                                                  │
│  Step 3: Define Test Objectives                                  │
│     • List features to test                                     │
│     • Define target/goal for each                               │
│                                                                  │
│  Step 4: Define Test Criteria                                    │
│     • Suspension Criteria (when to pause)                       │
│     • Exit Criteria (when testing complete)                     │
│                                                                  │
│  Step 5: Resource Planning                                       │
│     • Human Resources (roles, skills)                           │
│     • System Resources (servers, tools)                         │
│                                                                  │
│  Step 6: Plan Test Environment                                   │
│     • Hardware/software requirements                            │
│     • Network configuration                                     │
│                                                                  │
│  Step 7: Schedule & Estimation                                   │
│     • Task estimation (effort)                                  │
│     • Project schedule (timeline)                               │
│                                                                  │
│  Step 8: Test Deliverables                                       │
│     • Before: Test Plan, Test Cases, Test Design Specs          │
│     • During: Test Scripts, Test Data, RTM, Logs               │
│     • After: Test Results, Defect Reports, Release Notes       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 7.5.4 Test Criteria

**Suspension Criteria:**
- When to PAUSE testing
- Example: If 40% test cases fail, suspend until dev fixes

**Exit Criteria:**
- When testing is COMPLETE
- Example: 95% critical tests pass, no P1/P2 open bugs

**Key Metrics:**
- **Run Rate** = Executed / Total test cases (must be 100%)
- **Pass Rate** = Passed / Executed test cases (target depends on project)

### 7.5.5 Risk Analysis (3-Step Process)

```
Step 1: IDENTIFY Risks
     ↓
Step 2: ANALYZE Impact (Probability × Impact)
     ↓
Step 3: COUNTERMEASURES (Response, Register, Monitor)
```

**Risk Types:**

| Risk Type | Description | Example |
|-----------|-------------|---------|
| **Project Risk** | Impacts project progress | Schedule delay, resource shortage |
| **Product Risk** | System fails user expectations | Missing features, unreliable software |
| **Organization Risk** | Related to human resources | Skill gaps, insufficient staff |
| **Technical Risk** | Technical process issues | Wrong environment setup, wrong test procedure |
| **Business Risk** | External factors | Budget cuts, customer changes |

**Risk Priority Matrix:**

```
Priority = Probability × Impact

| Probability | Impact | Priority | Action |
|-------------|--------|----------|--------|
| High (3)    | High (3) | 9 | Immediate mitigation, daily monitoring |
| High (3)    | Medium (2) | 6 | Weekly monitoring at progress meeting |
| Medium (2)  | Low (1) | 2 | Accept and monitor at milestones |
```

**Risk Response Strategies:**
- **Avoidance**: Eliminate the risk entirely
- **Reduction**: Mitigate impact with corrective measures
- **Sharing/Transfer**: Transfer to another party (insurance, outsource)
- **Accept**: Acknowledge and prepare budget/contingency

### 7.5.6 Test Estimation Methods

**1. Work Breakdown Structure (WBS)**
- Break project into smallest tasks
- Allocate tasks to team members
- Estimate effort for each task

**2. Function Point Method**
```
| Complexity | Weightage |
|------------|-----------|
| Complex    | 5         |
| Medium     | 3         |
| Simple     | 1         |

Total Effort = Total Function Points × Effort per Point
Cost = Total Effort (hours) × Hourly Rate
```

**3. Three-Point Estimation**
```
E = (a + 4m + b) / 6

Where:
  a = Best-case estimate
  m = Most likely estimate
  b = Worst-case estimate
```

### 7.5.7 Test Case Design Essentials

**Test Case Objective/Title Syntax:**
```
Action + Function + Operating Condition

Examples:
- Verify login with valid credentials
- Run annual report on Day 1 of fiscal year
- Validate search with empty query
```

**Good Test Case Characteristics:**
- **Accurate** - Tests what it's designed to test
- **Economical** - No unnecessary steps
- **Repeatable** - Can run again and again
- **Traceable** - Links to requirement
- **Appropriate** - Fits test environment
- **Self-standing** - Independent of writer
- **Self-cleaning** - Cleans up after itself

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

## 9.5 Documentation Testing

### What is Documentation Testing?

Documentation is now a major part of software systems - it might exceed the amount of source code and is often integrated into the software (e.g., help systems). **Testers must cover both code AND documentation.**

### Classes of Software Documentation

| Category | Examples |
|----------|----------|
| **Packaging & Marketing** | Box text, graphics, ads, inserts |
| **Legal** | Warranty, registration, EULA |
| **Installation** | Setup instructions, labels, stickers |
| **User Guidance** | User's manual, online help, tutorials |
| **Learning** | Wizards, computer-based training |
| **Reference** | Samples, examples, templates |
| **Feedback** | Error messages |

### Why Test Documentation?

| Benefit | Description |
|---------|-------------|
| **Improves Usability** | Not all software is intuitive |
| **Improves Reliability** | Bug in user manual = bug in software |
| **Lowers Support Cost** | Writing documentation helps debug the system |

### Documentation Testing Checklist

```
┌─────────────────────────────────────────────────────────────────┐
│               DOCUMENTATION TESTING CHECKLIST                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. AUDIENCE                                                      │
│     □ Documentation matches target audience level                 │
│     □ Not too novice or too advanced                             │
│                                                                   │
│  2. TERMINOLOGY                                                   │
│     □ Suitable for the audience                                   │
│     □ Terms used consistently throughout                          │
│     □ Abbreviations and acronyms defined                          │
│                                                                   │
│  3. CONTENT & SUBJECT MATTER                                      │
│     □ Appropriate subjects covered                                │
│     □ No subjects missing                                         │
│     □ Proper depth of coverage                                    │
│     □ No missing features described accidentally                  │
│                                                                   │
│  4. FACTUAL ACCURACY                                              │
│     □ All information technically correct                         │
│     □ Table of contents accurate                                  │
│     □ Index entries point to correct pages                        │
│     □ Chapter references correct                                  │
│     □ Website URLs working                                        │
│     □ Phone numbers correct                                       │
│                                                                   │
│  5. STEP-BY-STEP PROCEDURES                                       │
│     □ No missing steps                                            │
│     □ Tester results match documentation                          │
│     □ Steps in correct order                                      │
│                                                                   │
│  6. FIGURES & SCREEN CAPTURES                                     │
│     □ Accurate and precise                                        │
│     □ From latest version of software                             │
│     □ Figure captions correct                                     │
│                                                                   │
│  7. SAMPLES & EXAMPLES                                            │
│     □ All examples work as advertised                             │
│     □ Code samples compile/run correctly                          │
│                                                                   │
│  8. SPELLING & GRAMMAR                                            │
│     □ Spell-checked                                               │
│     □ Grammar reviewed                                            │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Auto-Generated Code Documentation

Tools for generating documentation from source code comments:

| Tool | Language | Description |
|------|----------|-------------|
| **Javadoc** | Java | Standard Java documentation generator |
| **Doxygen** | C/C++/Java | Multi-language documentation generator |
| **Sphinx** | Python | Python documentation generator (RST format) |
| **JSDoc** | JavaScript | JavaScript API documentation |
| **Swagger/OpenAPI** | REST APIs | API documentation from annotations |

**Example - Javadoc Format:**
```java
/**
 * The time class represents a moment of time.
 *
 * @author John Doe
 */
class Time {
    /**
     * Constructor that sets the time to a given value.
     * @param timemillis milliseconds since Jan 1, 1970
     */
    public Time(long timemillis) { ... }
}
```

**Benefits of Auto-Generated Docs:**
- Code and documentation stay synchronized
- Reference guide style for quick lookups
- Consistent formatting
- Reduced maintenance effort

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
□ Fundamental concepts (Error/Fault/Failure, Testing vs Debugging, V&V)
□ 7 Principles of Software Testing (ISTQB)
□ QA vs QC vs Testing (ISO 9000 definitions)
□ SDLC models (Waterfall, V-Model, Agile)
□ STLC phases (6 phases with entry/exit criteria)
□ Test levels (Unit, Integration, System, Acceptance)
□ Test types (Smoke, Sanity, Regression, Exploratory)
□ Integration approaches (Big-Bang, Top-Down, Bottom-Up, Stubs/Drivers)
□ Test documentation (Plan, Case, Report)
□ Black-box techniques (ECP, BVA, Decision Table)
□ White-box techniques (Statement, Branch, Path, MC/DC coverage)
□ Code Coverage types and calculations
□ Defect lifecycle and reporting
□ Severity vs Priority distinction
□ Test metrics and formulas (DRE, Defect Density, Pass Rate)
□ Test Management (Independent Testing, Roles, Planning)
□ Risk Analysis and Estimation methods
□ Manual vs Automation decision
□ Documentation Testing
□ Agile testing practices
□ Shift-left testing (TDD, BDD, early testing)
□ DevSecOps (SAST, SCA, DAST integration)
```

---

*Good luck với kỳ thi! 🎓*
