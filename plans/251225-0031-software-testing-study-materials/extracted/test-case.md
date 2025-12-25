# Document Extraction Results

Converted 1 document(s) to markdown.

---

## S05.2_Test Case.pdf

# Software Testing
## CSC13003
### Test case

---

## Test Case
- A test that (ideally) executes a single well defined test objective (Testing Computer Software – Kaner, Faulk, Nguyen)
- A specific set of test data and associated procedures developed for a particular objectives (IEEE 729-1983)

---

## Why write test case?
- Accountability
- Reproducibility
- Tracking
- Automation
- To find bugs
- To verify that tests are being executed correctly
- To measure test coverage

---

## Test case Essentials
- Tracking information
- Test case ID
- Test case description
- Objective/Title
- Steps
- Test data: input/output/default
- Expected results
- Observed results
- Status: Pass/Fail/Blocked/Skipped
- Test environment
- Script
- Bug ID
- Comments
- ...

---

## Test case Essentials
- Test case Objective/Title
    - The most important essential
    - Gives reader a description and idea of the test
    - A good test name makes review easier
    - Easier to pass to another person, automation team
    - In many cases, may be the only part of the test case documented

---

## Test case Essentials
- Test case Objective/Title Syntax
    **Action + Function + Operating Condition**
    - Action: Verify, Test, Validate, Execute, Run, Print...
    - Function: function, feature, validation point
    - Operating Condition: data, specific condition

| Action | Function      | Operating Condition                     |
|--------|---------------|-----------------------------------------|
| Run    | annual report | from standard data (file location)      |
| Run    | annual report | on Day 1 of fiscal year                 |
| Run    | annual report | from empty spreadsheet                  |
| Run    | annual report | on last day of fiscal year              |

---

## Test case Essentials
- Validation Point
    - This is the expected result
    - Write it as a step
    - Define clearly state: what behavior, result or point that you are attempting to validate

---

## Test Case Template

| TC ID | Description        | Steps                                    | Expected Result                                                           | Observed result                                                          | Status       |
|-------|--------------------|------------------------------------------|---------------------------------------------------------------------------|--------------------------------------------------------------------------|--------------|
| TC001 | /*Test objective*/ | /*Very clear and specific steps*/        | /*you need to pre determine what your program is supposed to do*/         | /* write down the result that you get when excecute the test case*/      | Passed/<br>Failed |
|       |                    | Pre-condition:                           |                                                                           |                                                                          |              |
|       |                    | Steps:                                   |                                                                           |                                                                          |              |
|       |                    | 1. Action 1                              |                                                                           |                                                                          |              |
|       |                    | 2. Action 2                              |                                                                           |                                                                          |              |
|       |                    | .........................                |                                                                           |                                                                          |              |

---

## What is a Good Test case?
- Accurate – tests what it is designed to test
- Economical – no unnecessary steps
- Repeatable, reusable – keep going on
- Traceable – to a requirement
- Appropriate – for test environment
- Self standing – independent of the writer
- Self cleaning – picks up after itself

---

## Q & A

---

