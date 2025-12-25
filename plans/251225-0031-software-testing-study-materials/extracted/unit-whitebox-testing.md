# Document Extraction Results

Converted 1 document(s) to markdown.

---

## S10.1_Unit Testing & White Box Testing.pdf

# Software Testing
CSC13003

Unit Testing - White Box Testing

---

# Content

* Unit testing
* White box testing

---

# V model

The V-model illustrates the relationship between each phase of the software development lifecycle and its associated testing phase.

*   **Verification Phases (Left side, descending):**
    *   Requirement Design
    *   System Design
    *   Architecture Design
    *   Module Design
*   **Coding** (Bottom)
*   **Validation Phases (Right side, ascending):**
    *   Unit Test
    *   Integration Test
    *   System Test
    *   Acceptance Test

---

# Requirement Analysis

*   To gather requirements.
*   Business requirement analysis is to understand an aspect from a customer's perspective by stepping into their shoes to completely analyse the functionality of an application from a user's point of view.
*   An acceptance criteria layout is prepared to correlate the tasks done during the development process with the final outcome of the overall effort.

---

# Acceptance criteria

*   Requirement:
    *   I should be able to search the name of a book along with its details with the help of a universal search option on the front page of my library management system.
*   Acceptance criteria:
    *   Search by the name of a book only.
    *   Search by the publisher name, that shows all books belonging to the same category.
    *   Restrict the search with the name of a website.
    *   The user should not be able to execute the search if all the mandatory fields are not entered.
    *   The user must see the sequence number of the book.

Source: http://www.professionalqa.com/acceptance-criteria

---

# System design

*   It comprises of creating a layout of the system/application design that is to be developed. System design is aimed at writing a detailed hardware and software specification.
*   Architectural Design: high-level design
*   Module Design: low-level design

---

# Architectural design

*   High-level design

---

# Module design

*   Low-level design

---

# Unit testing

*   To verify singles modules and remove bug, if it exists.
*   A unit test is simply executing a piece of code to verify whether it delivers the desired functionality.

---

# Integration testing

*   To collaborate pieces of code together to verify that they perform as a single entity.

---

# System testing

*   System testing is performed when the complete system is ready, the application is then run on the target environment in which it must operate.

---

# User acceptance testing

The user acceptance test plan is prepared during the requirement analysis phase because when the software is ready to be delivered, it is tested against a set of tests that must be met in order to certify that the product has achieved the target it was intended to.

---

# What is unit testing?

*   Unit testing is used to verify a small chunk of code by creating a path, **function** or a **method**.
*   Unit testing is used to identify defects early in software development cycle.
*   Unit testing will compel to read our own code. i.e. a developer starts spending more time in reading than writing.
*   Defects in the design of code affect the development system. A successful code breeds the confidence of developer.

---

# Why unit testing?

*   It finds bugs early in the code, which makes our code more reliable.
*   Unit testing forces a developer to read code more than writing.
*   You develop more readable, reliable and bug-free code which builds confidence during development.

**Cost to Fix a Bug based on Time When Bug Is Found:**
A diagram illustrating that the cost to fix a bug increases significantly the later it is found in the development lifecycle (Specification < Design < Code < Test < Release). The cost ranges from $1 (Specification) to $1000+ (Release).

---

# TDD & Unit testing

*   Test Driven Development
*   Tests are written before the code
*   Rely heavily on testing frameworks
*   All classes in the applications are tested
*   Quick and easy integration is made possible

---

# Process

*   Write test cases (test methods) to test a method under test.
*   Use the unit testing framework to execute test methods.
*   Result: Passed / Failed.
*   If a test method failed,
    *   the method under test has a bug,
    *   developers must modify source code of the method under test to fix the detected bug.

---

# Ex: Apply unit testing to the method Triangle::getPerimeter()

*   Test method 1: test the method getPerimeter() with a right triangle
    *   Init a triangle with 3 points: A(0,0), B(0,3) và C(4,0)
    *   Expected perimeter = 3 + 4 + 5 = 12
    *   Actual perimeter = invoke the method Triangle::getPerimeter()
    *   Assert to compare the value of expected and actual ones.
*   Test method 2: test the method getPerimeter() with an isosceles right triangle
    *   Init a triangle with 3 points: A(0,0), B(0,1) và C(1,0)
    *   Expected perimeter = 1 + 1 + 1.4 = 3.4
    *   Actual perimeter = invoke the method Triangle::getPerimeter()
    *   Assert to compare the value of expected and actual ones.
*   Test method 3: ...

---

# Unit testing vs debugging

*   Debugging: manual.
*   Unit testing: automatic.

---

# JUnit

*   Kent Beck, Erich Gamma, David Saff, Kris Vasudevan
*   Latest stable version: 5.4.2 (2019-04-07)

---

# JUnit installation

*   Maven repository
*   “junit”, “hamcrest-core"
*   pom / jar

---

# Hello World JUnit testing

This slide illustrates a basic JUnit test setup and execution within an IDE.

**1. TestJunit Class (The test itself):**
```java
package guru99.junit;

import org.junit.Test;

public class TestJunit {
    @Test
    public void testSetup() {
        String str = "I am done with Junit setup";
        assertEquals("I am done with Junit setup", str);
    }
}
```

**2. TestRunner Class (Executes the tests):**
```java
package guru99.junit;

import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
import org.junit.runner.notification.Failure; // Assuming this import, based on usage

public class TestRunner {
    public static void main(String[] args) {
        Result result = JUnitCore.runClasses(TestJunit.class);
        for (Failure failure : result.getFailures()) {
            System.out.println(failure.toString());
        }
        System.out.println("Result==" + result.wasSuccessful());
    }
}
```

**3. IDE Context Menu (Running the test):**
A context menu in an IDE (likely Eclipse) showing options like "Run As > Ju 1 JUnit Test" and "Run Configurations...".

**4. JUnit Output (Test results):**
A JUnit panel in the IDE showing:
*   "Finished after 0.108 seconds"
*   "Runs: 1/1"
*   "Errors: 0"
*   "Failures: 0"
*   The hierarchy indicates `guru99.junit.TestJunit` and `testSetup (0.000 s)` passed.

---

# Method under test

```java
public class Calculator {
    public int evaluate(String expression) {
        int sum = 0;
        for (String summand : expression.split("\\+"))
            sum -= Integer.valueOf(summand);
        return sum;
    }
}
```

---

# A test method

```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class CalculatorTest {
    @Test
    public void evaluatesExpression() {
        Calculator calculator = new Calculator();
        int sum = calculator.evaluate("1+2+3");
        assertEquals(6, sum);
    }
}
```

---

# assertXXX()

```java
@Test
public void testAssertEquals() {
    assertEquals("failure - strings are not equal", "text", "text");
}
```

```java
@Test
public void testAssertFalse() {
    assertFalse("failure - should be false", false);
}
```

```java
@Test
public void testAssertArrayEquals() {
    byte[] expected = "trial".getBytes();
    byte[] actual = "trial".getBytes();
    assertArrayEquals("failure - byte arrays not same", expected, actual);
}
```

---

# assertXXX

```java
@Test
public void testAssertNotSame() {
    assertNotSame("should not be same Object", new Object(), new Object());
}
```

```java
@Test
public void testAssertNotNull() {
    assertNotNull("should not be null", new Object());
}
```

```java
@Test
public void testAssertSame() {
    Integer aNumber = Integer.valueOf(768);
    assertSame("should be same", aNumber, aNumber);
}
```

```java
@Test
public void testAssertNull() {
    assertNull("should be null", null);
}
```

---

# assertThat(value,matcher)

```java
@Test
public void evaluatesExpression() {
    Calculator calculator = new Calculator();
    int sum = calculator.evaluate("1+2+3");
    assertThat(sum, is(equalTo(6)));
}
```

```java
assertThat(num, is(12));
assertThat(num, is(equalTo(12)));
```

```java
assertThat(someString, is(equalTo("blah")));
assertThat(someString, equalTo("blah"));
```

```java
assertThat(someObject, is(nullValue()));
assertThat(someObject, nullValue());
```

---

# assertThat(value,matcher)

```java
assertThat(foo, is(true));
```

```java
assertThat(str, containsString(substr));
```

```java
@Test
public void testAssertThatHasItems() {
    assertThat(Arrays.asList("one", "two", "three"), hasItems("one", "three"));
}
```

```java
@Test
public void testAssertThatBothContainsString() {
    assertThat("albumen", both(containsString("a")).and(containsString("b")));
}
```

```java
@Test
public void testAssertThatEveryItemContainsString() {
    assertThat(Arrays.asList(new String[] { "fun", "ban", "net" }), everyItem(containsString("n")));
}
```

---

# assertThat(value,matcher)

```java
@Test
public void testAssertThatHamcrestCoreMatchers() {
    assertThat("good", allOf(equalTo("good"), startsWith("good")));
    assertThat("good", not(allOf(equalTo("bad"), equalTo("good"))));
    assertThat("good", anyOf(equalTo("bad"), equalTo("good")));
    assertThat(7, not (CombinableMatcher.<Integer>either(equalTo(3)).or(equalTo(4))));
    assertThat(new Object(), not(sameInstance(new Object())));
}
```

```java
@Test
public void testReverse() {
    assertThat("oof", is(equalTo(StringUtils.reverseString("foo"))));
    assertThat("rab", is(equalTo(StringUtils.reverseString("bar"))));
}
```

---

# assertThat(value,matcher)

```java
@Test
public void testPalindromes() {
    String[] matches =
        { "a", "aba", "Aba", "abba", "AbBa",
        "abcdeffedcba", "abcdEffedcba" };
    String[] misMatches =
        { "ax", "axba", "Axba", "abbax", "xAbBa",
        "abcdeffedcdax", "axbcdEffedcda" };
    for (String s : matches) {
        assertThat(StringUtils.isPalindrome(s), is(true));
    }
    for (String s : misMatches) {
        assertThat(StringUtils.isPalindrome(s), is(false));
    }
}
```

---

# @Test annotation

*   Place the annotation at a public, void method.
*   JUnit recognized it as a test method (test case).

---

# Expect

*   Expect a test case should throw an exception

```java
@Test(expected = IndexOutOfBoundsException.class)
public void empty() {
    new ArrayList<Object>().get(0);
}
```

```java
@Test
public void example1() {
    try {
        find("something");
        fail();
    } catch (NotFoundException e) {
        assertThat(e.getMessage(), containsString("could not find something"));
    }
    // ... could have more assertions here
}
```

---

# Timeout

*   Expect the test case after a period of time.

```java
@Test(timeout = 1000)
public void testWithTimeout() {
    // ...
}
```

```java
public class HasGlobalTimeout {
    public static String log;
    private final CountDownLatch latch = new CountDownLatch(1);

    @Rule
    public Timeout globalTimeout = Timeout.seconds(10); // 10 seconds max per method tested

    @Test
    public void testSleepForTooLong() throws Exception {
        log += "ran1";
        TimeUnit.SECONDS.sleep(100); // sleep for 100 seconds
    }

    @Test
    public void testBlockForever() throws Exception {
        log += "ran2";
        latch.await(); // will block
    }
}
```

---

# @Before, @After

*   Place the annotation at a public, void method.
*   The method helps "prepare" things before executing a test method and to clean things after running a test method.
    *   `@Before`: run before each test method.
    *   `@After`: run after each test method

---

# @BeforeClass, @AfterClass

*   `@BeforeClass` is executed before running any of the test methods.
*   `@AfterClass` is executed after running any of the test methods.

---

# Example

```java
private Collection collection;

@BeforeClass
public static void oneTimeSetUp() {
    // one-time initialization code
    System.out.println("@BeforeClass - oneTimeSetUp");
}

@AfterClass
public static void oneTimeTearDown() {
    // one-time cleanup code
    System.out.println("@AfterClass - oneTimeTearDown");
}

@Test
public void testEmptyCollection() {
    assertTrue(collection.isEmpty());
    System.out.println("@Test - testEmptyCollection");
}

@Test
public void testOneItemCollection() {
    collection.add("itemA");
    assertEquals(1, collection.size());
    System.out.println("@Test - testOneItemCollection");
}
```

```java
@Before
public void setup() {
    collection = new ArrayList();
    System.out.println("@Before - setup");
}

@After
public void tearDown() {
    collection.clear();
    System.out.println("@After - tearDown");
}
```

**Execution Order Summary:**
*   @BeforeClass - oneTimeSetUp
*   @Before - setUp
*   @Test - testEmptyCollection
*   @After - tearDown
*   @Before - setUp
*   @Test - testOneItemCollection
*   @After - tearDown
*   @AfterClass - oneTimeTearDown

---

# @Ignore

*   Use this annotation to ignore a specific test method.

```java
@Ignore("Not Ready to Run")
@Test
public void divisionWithException() {
    System.out.println("Method is not ready yet");
}
```

---

# @Parameterized

```java
public class Fibonacci {
    public static int compute(int n) {
        int result = 0;
        if (n <= 1) {
            result = n;
        } else {
            result = compute(n - 1) + compute(n - 2);
        }
        return result;
    }
}
```

---

# @Parameterized

```java
@RunWith(Parameterized.class)
public class FibonacciTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {
            {0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5}, {6, 8}
        });
    }

    private int fInput;
    private int fExpected;

    public FibonacciTest(int input, int expected) {
        fInput = input;
        fExpected = expected;
    }

    @Test
    public void test() {
        assertEquals(fExpected, Fibonacci.compute(fInput));
    }
}
```

---

# Content

*   Unit testing
*   **White box testing**

---

# White Box Testing

*   A strategy testing based on the internal paths, structure, and implementation of the System Under Test
*   Can be applied at all levels of system development - unit, integration, and system

**WHITE BOX TESTING APPROACH**
A conceptual diagram showing the flow:
Test Case Input → Application Code → Test Case Output

---

# White Box Testing Techniques

*   Control Flow Testing
    *   Identify the **execution paths** through a module of **program code**
*   Data Flow Testing
    *   Identify paths in the program that go from the **assignment** to the **use** of a **variable** in a module of program code

---

# Control Flow Testing

*   Create and executes test cases to **cover the execution paths** through a module or program code
*   **Path:** a sequence of statement execution that begins at an entry and ends at an exit

---

# Technique: Control Flow Graph

```
a = 1;
b = 2;
c = 3;
if (a == 2) {
    x = x + 2;
} else {
    x = x / 2;
}
p = c /(b + c);
if (b/c > 3) {
    z = x + y;
}
```

**Control Flow Graph Representation:**
A flowchart showing decision points and processing steps for the given code.
*   Start: `a=1`, `b=2`, `c=3`
*   Decision: `if a==2`
    *   If True: `X=X+2`
    *   If False: `X=x/2`
*   Continue: `p=c/(b+c)`
*   Decision: `if b/c > 3`
    *   If True: `z=x+y`
*   End

How many test cases?

---

# Technique: Control Flow Graph

**Common Control Flow Structures:**

*   **Sequence:** Two blocks connected in a series.
*   **If:** A decision point branching into one path, then merging back.
*   **Case:** A decision point branching into multiple paths, then merging back.
*   **While:** A loop where the condition is checked at the beginning.
*   **Until:** A loop where the condition is checked at the end.

---

# Code Coverage

The percentage of the code that has been tested

1.  Statement Coverage
2.  Branch/Decision Coverage
3.  Condition Coverage
4.  Multiple Condition Coverage
5.  Path Coverage

---

# Example

```
1 if (a > 0) {
2   x = x + 1;
3 }
4 if (b == 3) {
5   y = 0;
6 }
```

**Control Flow Graph:**
*   Start node
*   Decision node `if a>0`
    *   True branch to `x=x+1` node
    *   False branch skips `x=x+1`
*   Decision node `if b==3`
    *   True branch to `y=0` node
    *   False branch skips `y=0`
*   End node

---

# Execution paths

Two visual examples of execution paths within a control flow graph are provided:
*   **P1 & P2:** Show two distinct paths through a graph with two sequential 'if' decisions.
*   **P3 & P4:** Show two distinct paths through a graph with two sequential 'if' decisions, similar structure to P1/P2 but with different highlighted paths.

---

# Statement Coverage

*   Every statement is tested at least one
*   1 test case: 100% Statement Coverage
    *   a=6, b=3

**Control Flow Graph for Statement Coverage (Example with a=6, b=3):**
A control flow graph similar to the one on page 46, showing a single path (Test 2) that executes through both `if a>0` and `if b==3` branches, thus covering all statements.

---

# Branch/Decision Coverage

*   Each decision that has a TRUE and FALSE outcome is evaluated at least one
*   2 test cases: 100% Decision Coverage
    *   a=0, b=2
    *   a=4, b=3

**Control Flow Graphs for Branch/Decision Coverage:**
*   **Test 1 (Example with a=0, b=2):** Shows a path where `if a>0` is False, and `if b==3` is False.
*   **Test 2 (Example with a=4, b=3):** Shows a path where `if a>0` is True (`x=x+1` executed), and `if b==3` is True (`y=0` executed).
Together, these two test cases cover both true and false branches for each decision.

---

# Condition Coverage

*   Each condition that has a TRUE and FALSE outcome that makes up a decision is evaluated at least one
*   2 test cases:
    *   a=1, c=1, b=3, d=-1
    *   a=0, c=2, b=2, d=0

```java
1 if (a > 0 && c == 1) {
2   x = x + 1;
3 }
4 if (b == 3 || d < 0) {
5   y = 0;
6 }
```

---

# Multiple Condition Coverage

```java
1 if (a > 0 && c == 1) {
2   x = x + 1;
3 }
4 if (b == 3 || d < 0) {
5   y = 0;
6 }
```

*   4 test cases:
    *   a=1, c=1, b=3, d=-1
    *   a=0, c=1, b=3, d=0
    *   a=1, c=2, b=2, d=-1
    *   a=0, c=2, b=2, d=0

**Control Flow Graph with Conditions:**
A detailed control flow graph with two main decision nodes: `if a>0` (which branches to `if c==1`) and `if b==3` (which branches to `if d<0`).
*   `if a>0` can be True or False.
*   If `a>0` is True, then `if c==1` can be True (leading to `x=x+1`) or False.
*   `if b==3` can be True or False.
*   If `b==3` is True, then `if d<0` can be True (leading to `y=0`) or False.
The graph illustrates the paths for `x=x+1` and `y=0` based on the outcomes of these nested/compound conditions.

---

# Path Coverage

*   Structure Testing / Basic Path Testing
    1.  Derive the control flow graph
    2.  Compute the graph's Cyclomatic Complexity
        *   C = edges – nodes + 2
    3.  Select a set of C basis paths
    4.  Create a test case for each basis path
    5.  Execute these tests

---

# Basic Path Testing

*   24 edges, 19 nodes
*   Cyclomatic Complexity
    *   24 - 19 + 2 = 7

**Control Flow Graph Example:**
A complex control flow graph with nodes labeled A through S, illustrating multiple branching and merging paths. This graph is used to demonstrate path coverage.

---

# Create a set of basis paths

1.  Pick a "baseline" path
    *   ABDEGKMQS

**Control Flow Graph with Baseline Path:**
The same complex control flow graph from page 53, with the path ABDEGKMQS highlighted.

---

# Choose the next path

2.  Change the outcome of the first decision along the baseline path while keeping the maximum number of other decisions the same
    *   ACDEGKMQS

**Control Flow Graph with Second Path:**
The graph with the path ACDEGKMQS highlighted, showing a different initial decision outcome compared to the baseline.

---

# Generate the third path

3.  Begin again with the baseline but vary the second decision rather than the first
    *   ABDFILORS

**Control Flow Graph with Third Path:**
The graph with the path ABDFILORS highlighted, demonstrating a variation further down the path.

---

# Generate the next paths

*   Continue varying each decision, one by one, until the bottom of the graph is reached
*   ABDEHKMQS

**Control Flow Graph with Fourth Path:**
The graph with the path ABDEHKMQS highlighted.

---

# Generate the next paths

Two further examples of control flow graphs with distinct paths highlighted are presented. These illustrate the ongoing process of generating all independent basis paths.

---

# Generate the next paths

**A Set of Basis Paths for the Example Graph:**
*   ABDEGKMQS
*   ACDEGKMQS
*   ABDFILORS
*   ABDEHKMQS
*   ABDEGKNQS
*   ACDFJLORS
*   ACDFILPRS

---

# Example

**Control Flow Graph:**
A simple control flow graph with nodes A, B, C, D, E.
*   A: `if a>0` (Decision)
*   B: `if b==3` (Decision)
*   C: `x=x+1` (Process, reached from A=True)
*   D: `end` (Terminal)
*   E: `y=0` (Process, reached from B=True)

*   6 edges, 5 nodes
*   Cyclomatic Complexity
    *   6 - 5 + 2 = 3
*   3 basis paths
    *   ABD (A=False, B=False, D)
    *   ACBD (A=True, C, B=False, D)
    *   ABED (A=False, B=True, E, D)
*   3 Test cases
    *   ABD: a=0, b=2
    *   ACBD: a=1, b=2
    *   ABED: a=0, b=3

---

# Example

```
Read A
IF A > 0 THEN
  IF A > 21 THEN
    Print "Key"
  ENDIF
ENDIF
```

**Flowchart Representation:**
*   Start `Read`
*   Decision `A>0`
    *   Yes path to Decision `A=21` (typo, should be `A>21`)
        *   Yes path to `Print`
        *   No path goes to `End`
    *   No path goes to `End`
*   End `End`

*   Cyclomatic complexity: _______
*   Minimum tests to achieve:
    *   Statement coverage: _______
    *   Branch coverage: _______

---

# Example

```
Read A
Read B
IF A > 0 THEN
  IF B = 0 THEN
    Print “No values"
  ELSE
    Print B
    IF A > 21 THEN
      Print A
    ENDIF
  ENDIF
ENDIF
```

**Flowchart Representation:**
*   Start `Read` (for A and B)
*   Decision `A>0`
    *   No path
    *   Yes path to Decision `B=0`
        *   No path to `Print` (B) then to Decision `A>21`
            *   No path
            *   Yes path to `Print` (A)
        *   Yes path to `Print` ("No values")
*   All paths converge to `End`

*   Cyclomatic complexity: _______
*   Minimum tests to achieve:
    *   Statement coverage: _______
    *   Branch coverage: _______

---

# Example

```
Read P
Read Q
IF P+Q > 100 THEN
  Print "Large"
ENDIF
If P > 50 THEN
  Print “P Large"
ENDIF
```

**Flowchart Representation:**
*   Start `Read` (for P and Q)
*   Decision `P+Q>100`
    *   Yes path to `Print` ("Large")
    *   No path
*   Paths merge
*   Decision `P>50`
    *   Yes path to `Print` ("P Large")
    *   No path
*   Paths merge to `End`

*   Cyclomatic complexity: _______
*   Minimum tests to achieve:
    *   Statement coverage: _______
    *   Branch coverage: _______

---

# Loop Testing

**Types of Loop Structures:**
*   **Simple loop:** A single loop structure.
*   **Nested Loops:** Loops contained within other loops.
*   **Concatenated Loops:** Loops arranged sequentially, one after another.
*   **Unstructured Loops:** Loops with irregular control flow, making them difficult to test.

---

# Loop Testing: Simple Loops

*   Minimum conditions – simple loops
    1.  Skip the loop entirely
    2.  Only one pass through the loop
    3.  Two passes through the loop
    4.  m passes through the loop m < n
    5.  (n-1), n, and (n+1) passes through the loop
    Where n is the maximum number of allowable passes

**Simple Loop Flowchart:**
A basic flowchart representing a loop: an entry point, a decision (loop condition), a process block (loop body), and a return path from the process block to the decision, and an exit path from the decision.

---

# Loop Testing

```java
public class loopdemo
{
    private int[] numbers = {5,-3,8,-12,4,1,-20,6,2,10};

    /** Compute total of numItems positive numbers in the array
     * @param numItems how many items to total, maximum of 10.
     */
    public int findTotal(int numItems)
    {
        int total = 0;
        if (numItems <= 10)
        {
            for (int count=0; count < numItems; count = count + 1)
            {
                if (numbers[count] > 0)
                {
                    total = total + numbers[count];
                }
            }
        }
        return total;
    }
}
```

**Example `numItems` values for testing:**

| numItems |
| :------- |
| 0        |
| 1        |
| 2        |
| 5        |
| 9        |
| 10       |
| 11       |

---

# Nested Loops

*   Extend simple loop testing
*   Reduce the number of tests
    *   start at the innermost loop; set all other loops to minimum values
    *   conduct simple loop test; add out of range or excluded values
    *   work outwards while keeping inner nested loops to typical values
    *   continue until all loops have been tested

**Nested Loop Flowchart:**
A flowchart showing an outer loop (decision, body) that contains an inner loop (decision, body).

---

# Concatenated Loops

*   Independent Loops
    *   => Test as simple loops
*   Dependent Loops
    *   => Test as nested loops

**Concatenated Loops Flowchart:**
A flowchart showing two simple loops connected sequentially, one after another.

---

# Unstructured Loops

*   DON'T test
*   Re-design

**Unstructured Loops Flowchart:**
A flowchart depicting an unstructured loop, marked with a large 'X' to indicate it should be avoided or redesigned.

---

# Data Flow Testing: Definition

*   **Def** - assigned or changed
*   **Uses** – utilized (not changed)
    *   **C-use** (Computation): right-hand side of an assignment, an index of an array, parameter of a function
    *   **P-use** (Predicate): branching the execution flow (if, while, for statement)

---

# Data Flow Testing

*   **Def-Use testing**
    *   All navigation paths from every definition of a variable to every use of it is exercised
*   **All-Use testing**
    *   At least one navigation path from every definition of a variable to every use of it is exercised

---

# Data Flow Testing

| Code Line | Statement             | Data Flow Description              |
| :-------- | :-------------------- | :--------------------------------- |
| 1         | `sum = 0`             | `sum, def`                         |
| 2         | `read (n),`           | `n, def`                           |
| 3         | `i=1`                 | `i, def`                           |
| 4         | `while (i <= n)`      | `i, n p-sue`                       |
| 5         | `read (number)`       | `number, def`                      |
| 6         | `sum = sum + number`  | `sum, def, sum, number, c-use`     |
| 7         | `i=i+1`               | `i, def, c-use`                    |
| 8         | `end while`           |                                    |
| 9         | `print (sum)`         | `sum, c-use`                       |

---

# Def-Use Testing

*   Ví dụ

**Table for sum:**

| pair id | def | use |
| :------ | :-: | :-: |
| 1       | 1   | 6   |
| 2       | 1   | 9   |
| 3       | 6   | 6   |
| 4       | 6   | 9   |

**Table for i:**

| pair id | def | use |
| :------ | :-: | :-: |
| 1       | 3   | 4   |
| 2       | 3   | 7   |
| 3       | 7   | 7   |
| 4       | 7   | 4   |

---

# Data Flow Criteria

**Hierarchy of Data Flow Coverage Criteria:**
A pyramid diagram illustrating the strength of different data flow criteria:

*   **Weaker (Top):**
    *   All c-uses
    *   All defs
    *   All p-uses
*   **Middle Tier (Stronger):**
    *   All c-uses, some p-uses
    *   All p-uses, some c-uses
*   **Stronger (Bottom):**
    *   All uses
    *   All def-use paths

This hierarchy indicates that criteria lower in the pyramid are "stronger" and require more "# tests".

---

# White box Testing Disadvantages

1.  The number of execution paths may be so large that they cannot all be tested
2.  The chosen test cases may not detect data sensitivity errors
    *   p = q / r
    *   May execute correctly except when r=0
3.  The tests are based on the existing paths, nonexistent paths cannot be discovered
4.  Testers must have the programming skills

---

# Q&A

(This page indicates a session for Questions and Answers.)

---

