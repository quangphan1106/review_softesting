# Document Extraction Results

Converted 1 document(s) to markdown.

---

## S06.1_Test Report.pdf

# Software Testing
CSC13003
Test Report

---

# Content

*   Bug/Defect Life Cycle
*   Bug/Defect Report
*   Test Summary Report

---

# Bug Life Cycle

(Image depicting the Bug Life Cycle flow chart is present on the original slide.)

Source: https://www.guru99.com/defect-life-cycle.html

---

# Bug Report

*   A detailed document about how the bug was found
*   “The point of writing a problem report (bug report) is to get bugs fixed"
    *   By Cem Kaner.

---

# Bug Report Essentials

1.  Bug ID
2.  Function name
3.  Problem summary
4.  How to reproduce it
5.  Reported by
6.  Date
7.  Assign to
8.  Status
9.  Priority
10. Severity

---

# 1. Bug ID

*   Unique identification number for the bug
*   Bug ID is different from Test case ID

---

# 2. Function name

*   The function in which the bug was found
*   Example:
    *   Login
    *   Logout
    *   Account list
    *   Add account
    *   Delete account

---

# 3. Problem summary

*   Summary about the problem
*   Test Objective + Actual result (vs. Expected result)
*   Example:
    *   There is no error message when account exists
    *   The room price is wrong when check-in date is equal to check-out date

---

# 4. How to reproduce it

*   Detailed steps along with screenshots with which the developer can reproduce the defects
*   Test steps + Expected result + Actual result
*   Example:

```
Description
Environment: iPhone 6s / Chrome Web Browser version 52.0
Steps to Reproduce:
1. Open Chrome
2. Navigate to "https://www.google.com"
3. In the top right, click Gmail
4. In the top right, click Sign In
Expected Behavior: User is directed to the Sign In screen
Actual Behavior: User receives a 521 error (screenshot attached)
```

---

# 5. Reported by

*   Name/ID of the tester who report the defect

# 6. Date

*   Date when the defect is reported

# 7. Assign to

*   Name/ID of the developer who will fix the defect

# 8. Status

*   New/ Closed / Reopened
*   Rejected/ Deferred/ Duplicate / In-progress/ Fixed

---

# 9. Priority

*   Priority is related to defect fixing urgency

| Defect Priority    | Description                                                                                             |
| :----------------- | :------------------------------------------------------------------------------------------------------ |
| Immediate (Critical) | The defect need to be fixed **immediately** or in **01 day** because it may cause great damage to the product |
| High               | The defect should be fixed within **02-04 days** because it impacts the product's **main features**   |
| Medium             | The defect should be fixed within **05-08 days** because it causes **minimal** deviation from the product requirement |
| Low                | The defect will be fixed later because it has very **minor** affect the product operation               |

---

# 10. Severity

*   Describe the impact of the defect on the application

| Defect Severity | Mô tả                                                                                                                                                                             |
| :-------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Fatal           | The defect cause **great damage** to the product. Ex: system crash, lost data                                                                                                     |
| Serious         | The defect impacts the product's **main features** Ex: User can delete comment without login                                                                                        |
| Medium          | The defect causes **minimal** deviation from the product requirement Ex: The GUI of the website does not display correctly on mobile devices                                        |
| Cosmetic        | The defect has very **minor** affect the product operation Ex: Incorrect tab order, no default focus, missing short key...                                                          |

---

# Bug Report Characteristics

*   Written
*   Numbered
*   Simple
*   Understandable
*   Reproducible
*   Legible
*   Non-judgmental

---

# How to reproduce the defect

*   Record all test steps
    *   Keyboards and mouse activities
    *   Screen capture

---

# Bad Bug Report

*   Isn't filed at all
*   Is filed via email
*   Contains no specific information
    *   "It does not work!"
    *   ➡️ "Error 404: Access denied"
*   Just reports the symptom
    *   "I just clicked and it crashes"
    *   ➡️ "Error 404: Page not found when clicking the Export button"

---

# Bad Bug Report (cont.)

*   Unknown or unclear environment/platform
    *   "Windows"
    *   ➡️ "Windows 7, Google Chrome 20.0.1132.47m"
*   Uses adjectives instead of numbers
    *   "System is really slow"
    *   ➡️ "System does not response after 3s but 5m"
*   Uses judgment
    *   "Error message is stupid"
    *   ➡️ "Error message is unclear"

---

# Test summary report

*   Summarize test activities and results
    *   Summary
    *   Test Case result report
    *   Defect Report
    *   Open point

---

# Test summary report

*   Statistics bug/defect by functions

**TEST REPORT**

| Project name | <Project name> | Reviewer  | <Reviewer> |
| :----------- | :------------- | :-------- | :--------- |
| Creator      | <Creator>      | Approver  | <Approver> |

Note: Date: <yyyy/mm/dd>

Test Coverage: 46%
Successful Test Coverage: 33%

| No | Items       | Tested | Passed | Failed | Blocked | Skipped | Not Yet Tested | Total | Tested Coverage |
| :-- | :---------- | :----- | :----- | :----- | :------ | :------ | :------------- | :---- | :-------------- |
| 1  | Function 1  | 23     | 15     | 5      | 3       | 7       | 18             | 48    | 48%             |
| 2  | Function 2  | 26     | 20     | 4      | 2       | 10      | 22             | 58    | 45%             |
| 3  |             |        |        |        |         |         |                |       |                 |
| 4  |             |        |        |        |         |         |                |       |                 |
| 5  |             |        |        |        |         |         |                |       |                 |
| **Total** |             | **49** | **35** | **9**  | **5**   | **17**  | **40**         | **106** |                 |

---

# Test summary report

*   Statistics by Defect Type

| Defect Type              | Fatal | Serious | Medium | Cosmetic | Total (W.def) | %    |
| :----------------------- | :---- | :------ | :----- | :------- | :------------ | :--- |
| Business logic           | 1     | 9       | 332    | 31       | 1082          | 58.7 |
| Coding logic             | 1     | 2       | 112    |          |               |      |
| Coding standard          |       |         | 4      |          |               |      |
| Data - Database integrity |       |         | 3      |          |               |      |
| Design issue             |       |         | 1      |          |               |      |
| Feature missing          |       |         | 18     |          |               |      |
| Functionality (Other)    |       |         | 5      | 2        | 17            | 0.9  |
| Other                    |       | 2       | 3      | 1        | 20            | 1.1  |
| Performance              |       |         | 2      |          | 6             | 0.3  |
| Req misunderstanding     |       |         | 1      |          | 3             | 0.2  |
| Security - Access Control |       |         | 30     | 152      | 242           | 13.1 |
| User Interface           |       |         |        |          |               |      |
| **Total**                | **2** | **13**  | **517** | **206**  | **1842**      | **100** |

*Note: The overlaid text on the original slide reads: "*
*   - Requirement workshop
*   - Review code
*   - Prototype designer
*   - Coding convention

---

# Test summary report

*   Statistics by Defect Type

(A bar chart titled "Defect Type Report" showing Defect Weight and Percentage across various Defect Types is present on the original slide. The data is largely represented in the previous table.)

---

# Test summary report

*   Statistics by Defect Severity

(A pie chart titled "Ratio of Defect Severity" showing the distribution of Fatal (1.7%), Serious (6.7%), Medium (8.3%), and Cosmetic (83.3%) defects is present on the original slide.)

---

(This page contains two large letters, Q and A, representing a Q&A section.)

---

