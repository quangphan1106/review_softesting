# Domain Testing & Data Generation - Concept Questions

## Question 1: Equivalence Class Partitioning (20 points)

### Scenario
ToolShop's product quantity field has these rules:
- Minimum: 1 unit
- Maximum: 99 units
- Must be a whole number
- Maximum depends on available inventory

### Questions

**a) (5 points)** What is Equivalence Class Partitioning (ECP) and why is it useful?

**Expected Answer:**

**Equivalence Class Partitioning:** Technique that divides input data into partitions where all values in a partition should be treated the same way.

**Principle:** If one value in a partition works, all values in that partition should work (and vice versa for failure).

**Benefits:**

| Benefit | Explanation |
|---------|-------------|
| **Reduces test cases** | Don't need to test every possible value |
| **Systematic coverage** | Ensures all categories are tested |
| **Identifies missing cases** | Forces thinking about invalid inputs |
| **Efficient** | Maximum coverage with minimum tests |

**Example:**
```
Without ECP: Test quantity 1, 2, 3, 4, ... 99 (99 tests)
With ECP: Test one from each partition (3-4 tests)
```

**b) (5 points)** Identify equivalence classes for the quantity field (valid and invalid).

**Expected Answer:**

**Equivalence classes:**

| Class | Description | Example Values | Expected Result |
|-------|-------------|----------------|-----------------|
| **Valid: Normal range** | 1 ≤ qty ≤ 99 | 1, 50, 99 | Accepted |
| **Invalid: Below minimum** | qty < 1 | 0, -1, -100 | Error: "Minimum quantity is 1" |
| **Invalid: Above maximum** | qty > 99 | 100, 500, 1000 | Error: "Maximum quantity is 99" |
| **Invalid: Non-integer** | Decimal numbers | 1.5, 2.7, 0.5 | Error: "Must be whole number" |
| **Invalid: Non-numeric** | Text/special chars | "abc", "", "@#$" | Error: "Invalid quantity" |
| **Valid: Boundary** | Exactly at limits | 1, 99 | Accepted |

**Visual representation:**
```
Invalid    Valid        Invalid
  ←─────────┼───────────────┼──────────→
         1               99
    [-∞, 0]   [1, 99]    [100, +∞]

    Class 1    Class 2    Class 3
```

**c) (5 points)** How does inventory availability create additional equivalence classes?

**Expected Answer:**

**Inventory-dependent partitions:**

| Scenario | Inventory | Quantity Requested | Class | Expected |
|----------|-----------|-------------------|-------|----------|
| **Sufficient stock** | 50 | 30 | Valid | Success |
| **Exact stock** | 50 | 50 | Valid/Boundary | Success |
| **Insufficient stock** | 50 | 60 | Invalid | Error: "Only 50 available" |
| **Out of stock** | 0 | 1 | Invalid | Error: "Out of stock" |

**Combined partition logic:**
```
Quantity must satisfy BOTH:
  1. 1 ≤ qty ≤ 99 (system rule)
  2. qty ≤ inventory (stock rule)

Effective max = MIN(99, inventory)

If inventory = 30:
  Valid: [1, 30]
  Invalid: [31, ∞] (inventory constraint)

If inventory = 150:
  Valid: [1, 99]
  Invalid: [100, ∞] (system constraint)
```

**d) (5 points)** Design minimum test cases using ECP for the quantity field.

**Expected Answer:**

**Minimum test set (one from each class):**

| Test # | Class | Input | Inventory | Expected |
|--------|-------|-------|-----------|----------|
| 1 | Valid normal | 50 | 100 | Success |
| 2 | Below minimum | 0 | 100 | Error: min is 1 |
| 3 | Above system max | 100 | 150 | Error: max is 99 |
| 4 | Above inventory | 60 | 50 | Error: only 50 available |
| 5 | Non-integer | 5.5 | 100 | Error: must be whole number |
| 6 | Non-numeric | "abc" | 100 | Error: invalid quantity |
| 7 | Out of stock | 1 | 0 | Error: out of stock |

**Why this is minimum:**
- Each equivalence class tested exactly once
- One representative value from each partition
- Boundary values can be added for thoroughness

---

## Question 2: Boundary Value Analysis (15 points)

### Scenario
ToolShop's discount code field:
- Must be exactly 8 characters
- Only alphanumeric characters allowed
- Case-insensitive

### Questions

**a) (5 points)** What is Boundary Value Analysis (BVA) and how does it complement ECP?

**Expected Answer:**

**Boundary Value Analysis:** Testing technique focusing on values at the edges of equivalence classes, where bugs are most likely to occur.

**Why boundaries matter:**
- Off-by-one errors common in programming
- Comparison operators often have edge case bugs (< vs ≤)
- Boundary conditions are where behavior changes

**Relationship with ECP:**

| Technique | Focus | Example (age 18-65) |
|-----------|-------|---------------------|
| **ECP** | Representative from each partition | Test: 0, 30, 80 |
| **BVA** | Values at boundaries | Test: 17, 18, 19, 64, 65, 66 |

**Complementary approach:**
```
Age range: 18-65

ECP alone: Test 30 (middle of valid range) → May miss boundary bugs
BVA added: Test 17, 18, 65, 66 → Catches off-by-one errors

Combined: Better coverage than either alone
```

**b) (5 points)** Identify boundary values for the discount code length.

**Expected Answer:**

**Boundary values for 8-character code:**

| Value | Length | Category | Expected Result |
|-------|--------|----------|-----------------|
| "" | 0 | Invalid (too short) | Error |
| "ABCDEFG" | 7 | Invalid (boundary-1) | Error: must be 8 characters |
| "ABCDEFGH" | 8 | Valid (exact boundary) | Accept |
| "ABCDEFGHI" | 9 | Invalid (boundary+1) | Error: must be 8 characters |
| Very long string | 100+ | Invalid (way over) | Error |

**Boundary diagram:**
```
                 Valid (exactly 8)
                      │
    Invalid           │           Invalid
   (too short)        │          (too long)
        │             │              │
        ▼             ▼              ▼
  ──────┬──────┬──────┬──────┬──────┬──────
        0      7      8      9      ∞
              ↑      ↑      ↑
           BVA    BVA    BVA
          points points points
```

**c) (5 points)** Design boundary tests for alphanumeric validation.

**Expected Answer:**

**Character boundary testing:**

| Aspect | Boundary Value | Test Input | Expected |
|--------|----------------|------------|----------|
| **Letter start (A)** | "AAAAAAAA" | Valid | Accept |
| **Letter end (Z)** | "ZZZZZZZZ" | Valid | Accept |
| **Digit start (0)** | "00000000" | Valid | Accept |
| **Digit end (9)** | "99999999" | Valid | Accept |
| **Before A** | Code with @ (ASCII 64) | "ABC@DEFG" | Error |
| **After Z** | Code with [ (ASCII 91) | "ABC[DEFG" | Error |
| **Before 0** | Code with / (ASCII 47) | "ABC/DEFG" | Error |
| **After 9** | Code with : (ASCII 58) | "ABC:DEFG" | Error |
| **Space** | "ABC DEFG" | Error |
| **Mixed valid** | "A1B2C3D4" | Valid | Accept |

**ASCII boundary reference:**
```
ASCII values around alphanumerics:

47 /  ← Invalid
48 0  ← Valid digit start
57 9  ← Valid digit end
58 :  ← Invalid

64 @  ← Invalid
65 A  ← Valid letter start
90 Z  ← Valid letter end
91 [  ← Invalid

97 a  ← Valid (case-insensitive)
122 z ← Valid
```

---

## Question 3: Pairwise Testing (20 points)

### Scenario
ToolShop's checkout page has these configuration options:
- Browser: Chrome, Firefox, Safari, Edge
- OS: Windows, macOS, Linux
- Payment: Credit Card, PayPal, Apple Pay, Bank Transfer
- Shipping: Standard, Express, Pickup

Full combination: 4 × 3 × 4 × 3 = 144 test cases

### Questions

**a) (5 points)** What is pairwise testing and why is it effective?

**Expected Answer:**

**Pairwise (All-Pairs) Testing:** Testing technique ensuring every pair of parameter values is tested together at least once.

**Why effective:**
- Most bugs involve interaction between 1-2 variables
- Studies show pairwise catches 70-90% of bugs
- Dramatically reduces test cases
- Statistically proven coverage

**Comparison:**
```
Exhaustive: 144 combinations
Pairwise:   ~16-20 combinations
Reduction:  ~88% fewer tests

But catches: ~85% of interaction bugs
```

**b) (5 points)** How does pairwise reduce 144 combinations? Explain the principle.

**Expected Answer:**

**Reduction principle:**

Instead of testing ALL combinations, ensure each PAIR appears at least once.

**Pair coverage math:**
```
Pairs to cover:
  Browser × OS:       4 × 3 = 12 pairs
  Browser × Payment:  4 × 4 = 16 pairs
  Browser × Shipping: 4 × 3 = 12 pairs
  OS × Payment:       3 × 4 = 12 pairs
  OS × Shipping:      3 × 3 = 9 pairs
  Payment × Shipping: 4 × 3 = 12 pairs

  Total pairs: 73 pairs

A single test covers multiple pairs:
  Test: Chrome, Windows, Credit Card, Express
  Covers: Chrome+Windows, Chrome+Credit Card,
          Chrome+Express, Windows+Credit Card,
          Windows+Express, Credit Card+Express

  = 6 pairs in one test!

Result: 73 pairs ÷ ~6 pairs/test ≈ 12-20 tests needed
```

**c) (5 points)** Create a pairwise test set for the given parameters.

**Expected Answer:**

**Sample pairwise test set (generated):**

| # | Browser | OS | Payment | Shipping |
|---|---------|------|---------|----------|
| 1 | Chrome | Windows | Credit Card | Standard |
| 2 | Chrome | macOS | PayPal | Express |
| 3 | Chrome | Linux | Apple Pay | Pickup |
| 4 | Firefox | Windows | PayPal | Pickup |
| 5 | Firefox | macOS | Apple Pay | Standard |
| 6 | Firefox | Linux | Credit Card | Express |
| 7 | Safari | Windows | Apple Pay | Express |
| 8 | Safari | macOS | Credit Card | Pickup |
| 9 | Safari | Linux | PayPal | Standard |
| 10 | Edge | Windows | Bank Transfer | Express |
| 11 | Edge | macOS | Credit Card | Standard |
| 12 | Edge | Linux | PayPal | Pickup |
| 13 | Chrome | Windows | Bank Transfer | Pickup |
| 14 | Firefox | macOS | Bank Transfer | Standard |
| 15 | Safari | macOS | Bank Transfer | Express |

**Verification (sample pairs):**
- Chrome+Windows: Test 1, 13 ✓
- Safari+PayPal: Test 9 ✓
- Bank Transfer+Express: Test 10, 15 ✓

**d) (5 points)** What are the limitations of pairwise testing?

**Expected Answer:**

| Limitation | Description | Mitigation |
|------------|-------------|------------|
| **Misses 3-way interactions** | Bugs needing 3+ factors | Use 3-wise or higher for critical systems |
| **Equal weight assumption** | Treats all pairs as equally important | Prioritize high-risk combinations manually |
| **Generated test may be impractical** | Some combinations invalid | Add constraints (e.g., Apple Pay only macOS) |
| **Not exhaustive** | Some combinations untested | Supplement with risk-based testing |
| **Tool dependency** | Manual generation error-prone | Use established tools (PICT, Allpairs) |

**When to use higher-order:**
```
1-wise: Each value tested once (minimal)
2-wise: Pairwise (most common, good balance)
3-wise: Three-way interactions (safety-critical)
4-wise+: Rarely needed (diminishing returns)
```

---

## Question 4: Test Data Generation (15 points)

### Scenario
ToolShop needs test data for:
- 1000 synthetic customer records
- Realistic addresses
- Valid email formats
- Test credit card numbers

### Questions

**a) (5 points)** What approaches exist for generating test data?

**Expected Answer:**

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **Manual creation** | Hand-craft test records | Precise control | Time-consuming, limited volume |
| **Random generation** | Programmatic random values | High volume | May not be realistic |
| **Faker libraries** | Realistic fake data generation | Realistic, localized | Still synthetic |
| **Production sampling** | Anonymized production data | Most realistic | Privacy concerns, masking needed |
| **Boundary-focused** | Generate edge case values | Tests limits | May miss normal cases |
| **Model-based** | Generate from data model | Complete coverage | Requires model definition |

**b) (5 points)** How should sensitive data be handled in test environments?

**Expected Answer:**

**Data masking strategies:**

| Data Type | Masking Approach | Example |
|-----------|------------------|---------|
| **Names** | Replace with synthetic | "John Doe" → "Mike Smith" |
| **Email** | Preserve domain, fake user | "real@company.com" → "test1@example.com" |
| **Phone** | Format-preserving randomize | "555-123-4567" → "555-999-8888" |
| **Credit Card** | Use test card numbers | Real number → "4111111111111111" |
| **Address** | Synthetic but valid format | Real address → Generated valid address |
| **SSN/ID** | Generate fake with valid format | "123-45-6789" → "000-00-0000" |

**Principles:**
```
1. Never copy production data directly to test
2. Maintain data relationships after masking
3. Preserve data format/length for testing
4. Use reversible masking for debugging if needed
5. Comply with regulations (GDPR, HIPAA)
```

**c) (5 points)** What is a test data generator and what features should it have?

**Expected Answer:**

**Test data generator features:**

| Feature | Purpose |
|---------|---------|
| **Data type support** | Generate strings, numbers, dates, etc. |
| **Format control** | Match specific patterns (phone, email, etc.) |
| **Localization** | Region-specific data (addresses, names) |
| **Uniqueness** | Guarantee unique values where needed |
| **Referential integrity** | Foreign keys reference valid records |
| **Volume control** | Generate thousands to millions of records |
| **Reproducibility** | Seed-based for consistent regeneration |
| **Custom rules** | Business-specific constraints |

**Generation pipeline:**
```
Configuration:
  - Schema definition
  - Business rules
  - Volume requirements
  - Seed for reproducibility
        │
        ▼
┌───────────────────┐
│  Generator runs   │
│  - Creates records│
│  - Applies rules  │
│  - Maintains refs │
└─────────┬─────────┘
          │
          ▼
┌───────────────────┐
│  Validation       │
│  - Format check   │
│  - Relationship   │
│  - Uniqueness     │
└─────────┬─────────┘
          │
          ▼
    Test Database
```

---

## Question 5: Decision Table Testing (10 points)

### Scenario
ToolShop's free shipping logic:
- Orders over $100 get free shipping
- Premium members always get free shipping
- Orders with promo code "FREESHIP" get free shipping
- Multiple conditions can apply

### Questions

**a) (5 points)** What is decision table testing and when is it useful?

**Expected Answer:**

**Decision table testing:** Technique that represents combinations of conditions and their resulting actions in a tabular format.

**When useful:**

| Scenario | Why Decision Table Helps |
|----------|-------------------------|
| **Multiple conditions** | Complex logic with many inputs |
| **Boolean combinations** | True/false combinations matter |
| **Business rules** | Formal specification of logic |
| **Complete coverage** | Ensures all combinations tested |
| **Documentation** | Clear representation of requirements |

**Structure:**
```
┌─────────────────┬────┬────┬────┬────┐
│ Conditions      │ R1 │ R2 │ R3 │ R4 │
├─────────────────┼────┼────┼────┼────┤
│ Condition 1     │ T  │ T  │ F  │ F  │
│ Condition 2     │ T  │ F  │ T  │ F  │
├─────────────────┼────┼────┼────┼────┤
│ Actions         │ R1 │ R2 │ R3 │ R4 │
├─────────────────┼────┼────┼────┼────┤
│ Action 1        │ X  │ X  │    │    │
│ Action 2        │    │    │ X  │ X  │
└─────────────────┴────┴────┴────┴────┘
```

**b) (5 points)** Create a decision table for the free shipping logic.

**Expected Answer:**

**Decision table for free shipping:**

| Conditions | R1 | R2 | R3 | R4 | R5 | R6 | R7 | R8 |
|------------|----|----|----|----|----|----|----|----|
| Order > $100 | Y | Y | Y | Y | N | N | N | N |
| Premium member | Y | Y | N | N | Y | Y | N | N |
| FREESHIP code | Y | N | Y | N | Y | N | Y | N |
| **Actions** |  |  |  |  |  |  |  |  |
| Free shipping | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |

**Simplification (any condition = free shipping):**
```
Free shipping IF:
  Order > $100 OR
  Premium member OR
  Has FREESHIP code

Only R8 (all No) pays for shipping
```

**Test cases derived:**

| Test | Order Value | Premium | Promo Code | Expected |
|------|-------------|---------|------------|----------|
| R1 | $150 | Yes | FREESHIP | Free shipping |
| R4 | $150 | No | None | Free shipping (order > $100) |
| R6 | $50 | Yes | None | Free shipping (premium) |
| R7 | $50 | No | FREESHIP | Free shipping (promo) |
| R8 | $50 | No | None | Paid shipping |

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | Equivalence Class Partitioning | 20 |
| Q2 | Boundary Value Analysis | 15 |
| Q3 | Pairwise Testing | 20 |
| Q4 | Test Data Generation | 15 |
| Q5 | Decision Table Testing | 10 |
| **Total** | | **80** |
