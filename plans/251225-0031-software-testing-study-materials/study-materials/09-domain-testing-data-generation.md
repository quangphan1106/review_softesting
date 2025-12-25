# Topic 9: Domain Testing & Data Generation - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 Domain Testing Overview

**Definition:** Systematic technique for selecting test cases based on input/output domains of a program.

**Core Techniques:**
- Equivalence Class Partitioning (ECP)
- Boundary Value Analysis (BVA)
- Decision Tables
- Combinatorial/Pairwise Testing

### 1.2 Equivalence Class Partitioning (ECP)

**Concept:** Divide input domain into classes where all values in a class are expected to behave similarly.

**Principle:** Test one value from each class (representative) instead of testing all values.

```
┌─────────────────────────────────────────────────────────────┐
│                    INPUT DOMAIN                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Invalid (Low)  │   Valid   │   Invalid (High)            │
│   ┌──────────┐   │ ┌──────┐ │  ┌──────────┐               │
│   │  x < 1   │   │ │1 ≤ x ≤│ │  │ x > 100  │               │
│   │          │   │ │  100  │ │  │          │               │
│   └──────────┘   │ └──────┘ │  └──────────┘               │
│       ▲          │    ▲     │       ▲                      │
│       │          │    │     │       │                      │
│   Test: -5       │ Test: 50 │   Test: 150                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Example - Age Field (1-120):**
| Class | Range | Representative | Expected |
|-------|-------|----------------|----------|
| Invalid (low) | < 1 | -5, 0 | Error |
| Valid (child) | 1-12 | 5 | Accept |
| Valid (teen) | 13-17 | 15 | Accept |
| Valid (adult) | 18-120 | 50 | Accept |
| Invalid (high) | > 120 | 150 | Error |

### 1.3 Boundary Value Analysis (BVA)

**Concept:** Errors tend to occur at boundaries, so test at and around boundary values.

**Standard BVA Values:**
- Minimum value (min)
- Just above minimum (min + 1)
- Nominal/typical value
- Just below maximum (max - 1)
- Maximum value (max)
- Just outside boundaries (min - 1, max + 1)

**Example - Input 1-100:**
| Value | Type | Expected |
|-------|------|----------|
| 0 | min - 1 | Invalid |
| 1 | min | Valid |
| 2 | min + 1 | Valid |
| 50 | nominal | Valid |
| 99 | max - 1 | Valid |
| 100 | max | Valid |
| 101 | max + 1 | Invalid |

### 1.4 Combining ECP and BVA

**Best Practice:** Use ECP for partitions, BVA for boundaries.

**Example - Age Field (1-120):**

| Value | Technique | Expected | Description |
|-------|-----------|----------|-------------|
| -10 | ECP | Invalid | Far below minimum |
| 0 | BVA | Invalid | Just below min |
| 1 | BVA | Valid | At minimum boundary |
| 2 | BVA | Valid | Just above min |
| 5 | ECP | Valid | Child representative |
| 15 | ECP | Valid | Teen representative |
| 50 | ECP | Valid | Adult representative |
| 119 | BVA | Valid | Just below max |
| 120 | BVA | Valid | At maximum boundary |
| 121 | BVA | Invalid | Just above max |
| 200 | ECP | Invalid | Far above maximum |

---

## 2. Combinatorial Testing

### 2.1 The Problem with Exhaustive Testing

**Example:**
- Browser: Chrome, Firefox, Safari (3 options)
- OS: Windows, macOS, Linux (3 options)
- Screen: 800×600, 1920×1080 (2 options)
- Language: EN, FR, ES (3 options)

**Exhaustive:** 3 × 3 × 2 × 3 = **54 combinations**

**Pairwise:** Only need to test all **pairs** → **~9 combinations**

### 2.2 Pairwise Testing

**Concept:** Most bugs involve interactions between ≤2 factors. Testing all pairs catches 95%+ of defects with far fewer test cases.

**Example Pairwise Test Set:**
| TC | Browser | OS | Screen | Language |
|----|---------|-------|--------|----------|
| 1 | Chrome | Windows | 800×600 | EN |
| 2 | Chrome | macOS | 1920×1080 | FR |
| 3 | Chrome | Linux | 1920×1080 | ES |
| 4 | Firefox | Windows | 1920×1080 | ES |
| 5 | Firefox | macOS | 800×600 | EN |
| 6 | Firefox | Linux | 800×600 | FR |
| 7 | Safari | Windows | 800×600 | FR |
| 8 | Safari | macOS | 1920×1080 | EN |
| 9 | Safari | Linux | 800×600 | ES |

**Verification:** Every pair (e.g., Chrome-Windows, Chrome-macOS, Chrome-800×600) appears at least once.

### 2.3 Pairwise Tools

| Tool | Type | Features |
|------|------|----------|
| **PICT** (Microsoft) | CLI | Text config, constraints |
| **ACTS** (NIST) | GUI/CLI | Advanced algorithms |
| **Pairwiser** | Web | Fast, visualization |
| **Hexawise** | Cloud | Enterprise features |

**PICT Usage:**
1. Create input.txt with parameters: `Browser: Chrome, Firefox, Safari`
2. Run: `pict input.txt > output.csv`
3. Tool generates minimum pairwise combinations

---

## 3. Data Generation Tools

### 3.1 Faker Library

**Purpose:** Generate realistic fake data programmatically.

**Installation:** `pip install faker` (Python) or `npm install @faker-js/faker` (JS)

**Common Methods:**

| Category | Methods | Example Output |
|----------|---------|----------------|
| Personal | `fake.name()`, `fake.email()` | "John Smith", "john@example.com" |
| Address | `fake.address()`, `fake.city()` | "123 Main St", "New York" |
| Phone | `fake.phone_number()` | "(555) 123-4567" |
| Date | `fake.date_of_birth(min_age=18)` | datetime object |
| Internet | `fake.ipv4()`, `fake.url()` | "192.168.1.1", "https://..." |
| Commerce | `fake.credit_card_number()`, `fake.company()` | "4111...", "Acme Inc." |
| Text | `fake.text()`, `fake.sentence()` | Lorem ipsum content |

**Key Features:**
- Unique values: `fake.unique.email()` - guarantees no duplicates
- Localization: `Faker('vi_VN')` - Vietnamese data
- Reproducible: `Faker.seed(123)` - consistent random data

### 3.2 Mockaroo

**Purpose:** Web-based data generation (no coding required).

**Features:**
- 140+ data types
- Export: CSV, JSON, SQL, Excel
- Custom formulas
- API access
- Relationships between fields

**Usage:**
1. Go to mockaroo.com
2. Define schema:
   - Field name
   - Data type (Name, Email, Number, Date, etc.)
   - Options (format, range, null %)
3. Generate (up to 1,000 rows free)
4. Download in desired format

**Mockaroo Formula Examples:**

| Formula | Result |
|---------|--------|
| `if field("age") >= 18 then "adult" else "minor" end` | Conditional values |
| `random("Red", "Green", "Blue")` | Random from list |
| `this.address_city + ", " + this.address_state` | Concatenate related fields |

### 3.3 Faker vs Mockaroo Comparison

| Aspect | Faker | Mockaroo |
|--------|-------|----------|
| **Type** | Code library | Web UI |
| **Setup** | pip/npm install | None |
| **Learning** | Code required | No-code |
| **Output** | Programmatic | CSV, JSON, SQL |
| **Relationships** | Manual | Built-in |
| **Uniqueness** | `.unique.` method | Formula |
| **Volume** | Unlimited | 1K free, 10M paid |
| **Best For** | Developers, CI/CD | Quick generation |

---

## 4. Practical Test Case Design

### 4.1 Toolshop Registration Test Cases

**Feature: User Registration**

**Input Fields:**
- First Name (2-50 characters)
- Last Name (2-50 characters)
- Email (valid email format)
- Password (8-20 characters, 1 upper, 1 lower, 1 digit)
- Date of Birth (18+ years old)

**ECP + BVA Test Cases:**

| TC | First Name | Last Name | Email | Password | DoB | Expected |
|----|------------|-----------|-------|----------|-----|----------|
| 1 | John (valid) | Doe (valid) | john@test.com | Pass1234 | 2000-01-01 | Success |
| 2 | J (1 char - BVA) | Doe | valid@test.com | Pass1234 | 2000-01-01 | Error |
| 3 | Jo (2 char - BVA) | Doe | valid@test.com | Pass1234 | 2000-01-01 | Success |
| 4 | A×51 (>50 - BVA) | Doe | valid@test.com | Pass1234 | 2000-01-01 | Error |
| 5 | John | "" (empty) | valid@test.com | Pass1234 | 2000-01-01 | Error |
| 6 | John | Doe | invalid-email | Pass1234 | 2000-01-01 | Error |
| 7 | John | Doe | valid@test.com | Pass123 (7 char) | 2000-01-01 | Error |
| 8 | John | Doe | valid@test.com | password (no upper) | 2000-01-01 | Error |
| 9 | John | Doe | valid@test.com | Pass1234 | 2010-01-01 (< 18) | Error |
| 10 | John | Doe | valid@test.com | Pass1234 | 1950-01-01 (valid age) | Success |

### 4.2 Product Search Test Cases

**Feature: Product Search & Filter**

**Parameters:**
- Query string (search text)
- Category ID (1-10)
- Price range (min: 0-10000, max: 0-10000)
- Sort (name_asc, name_desc, price_asc, price_desc)

**Pairwise Test Cases:**

| TC | Query | Category | Min Price | Max Price | Sort | Expected |
|----|-------|----------|-----------|-----------|------|----------|
| 1 | "hammer" | 1 | 0 | 100 | name_asc | Products matching |
| 2 | "hammer" | 2 | 50 | 500 | price_desc | Products matching |
| 3 | "" (empty) | 1 | 0 | 10000 | name_asc | All in category |
| 4 | "xyz" (no match) | 5 | 100 | 200 | price_asc | Empty result |
| 5 | "tool" | 10 | 10000 | 10000 | name_desc | Exact price match |
| 6 | "hammer" | null | 0 | 0 | null | All hammers |

### 4.3 Data Generation Script Concepts

**Toolshop Test Data Generation:**

| Entity | Count | Key Fields | Special Handling |
|--------|-------|------------|------------------|
| Categories | 50 | id, name, slug | Auto-generated |
| Products | 1,000 | id, name, price, category_id, stock | FK to categories, 20% rentals |
| Users | 50 | first_name, last_name, email, dob | Unique emails |
| Transactions | 1,000 | user_id, product_id, quantity, status | FK to users/products |

**Key Techniques:**
- Referential integrity: `category_id = fake.random_int(min=1, max=50)`
- Unique constraint: `fake.unique.email()`
- Distribution control: `fake.boolean(chance_of_getting_true=20)`
- Random enum: `fake.random_element(['pending', 'completed', 'cancelled'])`

**Export Formats:**
- JSON: `json.dump(data, file, indent=2)`
- CSV: `csv.DictWriter` with header

---

## 5. Toolshop Homework Connection

### 5.1 From Domain Testing Homework (HW02)

**Features Tested:**

**Feature 1: User Registration**
- ECP: Valid/invalid for each field
- BVA: Name length boundaries, age limits
- 15 test cases designed

**Feature 2: Create Product (Admin)**
- ECP: Valid product data, invalid categories
- BVA: Price (0, max), stock limits
- 12 test cases designed

**Feature 3: Search/Filter Products**
- ECP: Empty query, valid query, invalid filters
- BVA: Price range boundaries
- 10 test cases designed

### 5.2 From Data Generation Homework (HW04)

**Generated Data:**
- 50 categories
- 1,000 products
- 50 users
- 1,000 transactions

**Key Techniques:**
- Faker library for realistic data
- Referential integrity (product → category)
- Unique constraints (emails)
- Realistic distributions (20% rentals)

---

## 6. Application-Level Exam Questions

### Question 1: ECP + BVA Design
**Scenario:** Password field requirements:
- Length: 8-20 characters
- Must contain: 1 uppercase, 1 lowercase, 1 digit
- Cannot contain spaces

**Question:** Design comprehensive test cases.

**Answer:**

**BVA for Length:**

| Password | Chars | Expected | Description |
|----------|-------|----------|-------------|
| Pass12 | 6 | Invalid | Below min |
| Pass123 | 7 | Invalid | At boundary-1 |
| Pass1234 | 8 | Valid | At min boundary |
| Pass12345 | 9 | Valid | Above min |
| A×19 + a1 | 20 | Valid | At max boundary |
| A×20 + a1 | 21 | Invalid | Above max |

**ECP for Character Requirements:**

| Password | Expected | Missing Requirement |
|----------|----------|---------------------|
| password123 | Invalid | No uppercase |
| PASSWORD123 | Invalid | No lowercase |
| Passworddd | Invalid | No digit |
| Pass word1 | Invalid | Contains space |
| Pass1234 | Valid | All requirements met |

**Edge Cases:**

| Password | Expected | Reason |
|----------|----------|--------|
| Aa1xxxxx (8 chars) | Valid | Minimum valid with all reqs |
| Aa1 (3 chars) | Invalid | Has all reqs but too short |
| aaa11111 | Invalid | Correct length, no uppercase |

### Question 2: Pairwise Test Reduction
**Scenario:** Testing a form with:
- Country: US, UK, VN (3)
- Payment: Card, Bank, Cash (3)
- Shipping: Standard, Express (2)
- Gift wrap: Yes, No (2)

**Question:** Calculate exhaustive vs pairwise test count.

**Answer:**

**Calculation:**
- Exhaustive: 3 × 3 × 2 × 2 = **36 combinations**
- Pairwise: **9 test cases** (75% reduction)

**Pairwise Test Set:**

| TC | Country | Payment | Shipping | Gift |
|----|---------|---------|----------|------|
| 1 | US | Card | Standard | Yes |
| 2 | US | Bank | Express | No |
| 3 | US | Cash | Standard | No |
| 4 | UK | Card | Express | No |
| 5 | UK | Bank | Standard | Yes |
| 6 | UK | Cash | Express | Yes |
| 7 | VN | Card | Standard | Yes |
| 8 | VN | Bank | Express | Yes |
| 9 | VN | Cash | Standard | No |

**Verification:** All pairs covered (US-Card ✓, US-Bank ✓, UK-Card ✓, Card-Standard ✓, etc.)

### Question 3: Data Generation Strategy
**Scenario:** Need to test:
- 100K users for load testing
- Realistic distribution (60% US, 30% EU, 10% Asia)
- Unique emails
- Valid phone formats per region

**Question:** How would you generate this data?

**Answer:**

**Strategy:**

| Step | Technique |
|------|-----------|
| 1. Create regional Fakers | `Faker('en_US')`, `Faker(['de_DE', 'fr_FR'])`, `Faker('vi_VN')` |
| 2. Calculate counts | US: 60K, EU: 30K, Asia: 10K |
| 3. Ensure unique emails | Track in `set()`, regenerate if duplicate |
| 4. Batch insert | 1,000 records per batch for performance |

**Key Faker Concepts:**
- Locale-specific: `Faker('vi_VN')` → Vietnamese names, addresses
- Multi-locale: `Faker(['en_GB', 'de_DE'])` → Random from list
- Phone formats auto-match locale

**Performance Tips:**
- Use `db.bulk_insert()` instead of single inserts
- Batch size: 1,000 records
- Track used emails in `set()` for O(1) lookup

---

## 7. Cheatsheet / Quick Reference

### ECP Quick Reference

| Type | Rule | Example (Age 18-65) |
|------|------|---------------------|
| Valid class | Test ONE representative | 18-65 → Test: 40 |
| Invalid (low) | Test each invalid separately | < 18 → Test: 10 |
| Invalid (high) | Different errors possible | > 65 → Test: 70 |

### BVA Quick Reference

| Value Type | Purpose | Example (1-100) |
|------------|---------|-----------------|
| min - 1 | Invalid boundary | 0 |
| min | Boundary | 1 |
| min + 1 | Just inside | 2 |
| nominal | Typical | 50 |
| max - 1 | Just inside | 99 |
| max | Boundary | 100 |
| max + 1 | Invalid boundary | 101 |

### Faker Methods Cheatsheet

| Category | Methods |
|----------|---------|
| Personal | `name()`, `first_name()`, `email()`, `phone_number()`, `ssn()`, `date_of_birth()` |
| Address | `address()`, `city()`, `country()`, `zipcode()` |
| Business | `company()`, `job()`, `catch_phrase()` |
| Internet | `url()`, `domain_name()`, `ipv4()`, `mac_address()`, `user_agent()` |
| Commerce | `credit_card_number()`, `credit_card_expire()`, `price()`, `currency()` |
| Text | `text()`, `sentence()`, `paragraph()`, `word()` |

**Special Features:**
- Unique: `fake.unique.email()`
- Locale: `Faker('vi_VN')`

### Domain Testing Checklist

- [ ] Identify input parameters
- [ ] Define equivalence classes for each
- [ ] Identify boundary values
- [ ] Combine ECP + BVA for test cases
- [ ] Add invalid/error cases
- [ ] Consider pairwise for multiple parameters
- [ ] Generate test data (Faker/Mockaroo)
- [ ] Document expected results

---

## 8. Key Takeaways for Exam

1. **ECP**: One representative per class; test each invalid class separately
2. **BVA**: Test at min, max, and just outside boundaries
3. **Pairwise**: 95% defect detection with ~75% test reduction
4. **Faker**: Code-based, unlimited, good for CI/CD
5. **Mockaroo**: Web-based, no-code, 1K free tier
6. **Combine**: ECP for partitions + BVA for edges = comprehensive coverage
7. **Toolshop HW02/HW04**: Know the specific test cases designed

---

*Study time recommended: 2-3 hours*
*Practice: Design ECP+BVA test cases for a login form*
