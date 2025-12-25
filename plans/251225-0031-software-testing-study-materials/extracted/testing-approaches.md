# Document Extraction Results

Converted 1 document(s) to markdown.

---

## S02.1_Testing Approaches.pdf

# Software Testing
## CSC13003
## Software Testing Approaches

## Exercise 1
* The program's specification
    * This program is designed to add 2 numbers, which you will enter
    * Each number should be one or two digits

Figure 1.2 How the screen looks after the first test
```
? 2
? 3
5
?
The cursor (beside the question mark at the
bottom of the screen) shows you where the
next number will be displayed.
```

## Possible Test Cases
* Valid Cases: 199 x 199 = 39,601
    * -99 → -1
    * 0 → 99
* Invalid Cases: INFINITE
    * <= -100
    * >= 100
    * Not a number

## The problem
very large or infinite
number of test scenarios
+
finite amount of time
=
impossible to test everything

## The solution
Software testing strategies and methods (techniques)
exist to
reduce the number of tests to be run
whilst still providing sufficient coverage
of the system under test

## Testing Approaches
* BLACK -BOX TESTING
* CRAY-BOX TESTING
* WHITE-BOX TESTING

## Black Box Testing Approach
### BLACK BOX TESTING APPROACH
Input → Black Box → Output

## White Box Testing Approach
### WHITE BOX TESTING APPROACH
Test Case Input → Application Code (with internal logic diagram) → Test Case Output

## The two basic testing strategies

| Test Strategy | Tester's View | Knowledge Sources                                                | Methods                                     |
|---------------|---------------|------------------------------------------------------------------|---------------------------------------------|
| Black box     | Inputs        | Requirements document                                            | Equivalence class partitioning              |
|               | Outputs       | Specifications                                                   | Boundary value analysis                     |
|               |               | Domain knowledge                                                 | State transition testing                    |
|               |               | Defect analysis data                                             | Cause and effect graphing                   |
|               |               |                                                                  | Error guessing                              |
| White box     | (Diagram)     | High-level design                                                | Statement testing                           |
|               |               | Detailed design                                                  | Branch testing                              |
|               |               | Control flow graphs                                              | Path testing                                |
|               |               | Cyclomatic complexity                                            | Data flow testing                           |
|               |               |                                                                  | Mutation testing                            |
|               |               |                                                                  | Loop testing                                |

## Grey Box Testing
Grey Box Testing
Black Box + White Box = Gray Box

## Black – Grey - White Box Testing
*   **Internals Not Known:** Testing As User (Black Box)
*   **Internals Relevant to Testing Known:** Testing As User with Access to Internals (Grey Box)
*   **Internals Fully Known:** Testing As Developer (White Box)

---

