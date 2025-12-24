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

```python
def test_cases_for_age(min_age=1, max_age=120):
    """Generate comprehensive test cases combining ECP + BVA"""
    test_cases = []

    # Invalid low (ECP)
    test_cases.append((-10, "invalid", "Far below minimum"))
    test_cases.append((0, "invalid", "BVA: Just below min"))
    test_cases.append((1, "valid", "BVA: At minimum"))
    test_cases.append((2, "valid", "BVA: Just above min"))

    # Valid partitions (ECP representatives)
    test_cases.append((5, "valid", "ECP: Child representative"))
    test_cases.append((15, "valid", "ECP: Teen representative"))
    test_cases.append((50, "valid", "ECP: Adult representative"))

    # Upper boundary (BVA)
    test_cases.append((119, "valid", "BVA: Just below max"))
    test_cases.append((120, "valid", "BVA: At maximum"))
    test_cases.append((121, "invalid", "BVA: Just above max"))

    # Invalid high (ECP)
    test_cases.append((200, "invalid", "Far above maximum"))

    return test_cases
```

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

**PICT Example:**
```
# input.txt
Browser: Chrome, Firefox, Safari
OS: Windows, macOS, Linux
Screen: 800x600, 1920x1080
Language: EN, FR, ES

# Run: pict input.txt > output.csv
```

---

## 3. Data Generation Tools

### 3.1 Faker Library

**Purpose:** Generate realistic fake data programmatically.

**Installation:**
```bash
pip install faker  # Python
npm install @faker-js/faker  # JavaScript
```

**Python Example:**
```python
from faker import Faker

fake = Faker()
fake_vn = Faker('vi_VN')  # Vietnamese locale

# Basic usage
print(fake.name())        # "John Smith"
print(fake.email())       # "john.smith@example.com"
print(fake.address())     # "123 Main St, City, State"
print(fake.phone_number()) # "(555) 123-4567"

# Vietnamese data
print(fake_vn.name())     # "Nguyễn Văn A"
print(fake_vn.address())  # "123 Đường ABC, Quận 1, TP.HCM"

# Generate test data
def generate_users(count=100):
    users = []
    for _ in range(count):
        users.append({
            'id': fake.uuid4(),
            'name': fake.name(),
            'email': fake.unique.email(),  # Unique emails
            'phone': fake.phone_number(),
            'address': fake.address(),
            'dob': fake.date_of_birth(minimum_age=18, maximum_age=80),
            'created_at': fake.date_time_this_year()
        })
    return users

# Generate specific data types
fake.credit_card_number()      # "4111111111111111"
fake.credit_card_expire()      # "05/25"
fake.ipv4()                    # "192.168.1.1"
fake.url()                     # "https://example.com"
fake.company()                 # "Acme Inc."
fake.job()                     # "Software Engineer"
fake.text(max_nb_chars=200)    # Lorem ipsum text
fake.sentence(nb_words=6)      # "Random sentence here."
```

**JavaScript Example:**
```javascript
const { faker } = require('@faker-js/faker');

// Generate user data
function generateUsers(count) {
  return Array.from({ length: count }, () => ({
    id: faker.string.uuid(),
    name: faker.person.fullName(),
    email: faker.internet.email(),
    phone: faker.phone.number(),
    address: faker.location.streetAddress(),
    createdAt: faker.date.recent()
  }));
}

// Generate product data
function generateProducts(count) {
  return Array.from({ length: count }, () => ({
    id: faker.string.uuid(),
    name: faker.commerce.productName(),
    price: faker.commerce.price({ min: 10, max: 1000 }),
    category: faker.commerce.department(),
    description: faker.commerce.productDescription()
  }));
}
```

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
```
# Conditional values
if field("age") >= 18 then "adult" else "minor" end

# Random from list
random("Red", "Green", "Blue")

# Related data
this.address_city + ", " + this.address_state
```

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

### 4.3 Data Generation Script

```python
from faker import Faker
import json
import csv

fake = Faker()

# Generate test data for Toolshop
def generate_toolshop_data():
    # Categories
    categories = []
    for i in range(1, 51):
        categories.append({
            'id': i,
            'name': fake.word().title() + " " + fake.word().title(),
            'slug': fake.slug()
        })

    # Products
    products = []
    for i in range(1, 1001):
        products.append({
            'id': i,
            'name': fake.catch_phrase(),
            'description': fake.text(max_nb_chars=500),
            'price': round(fake.pyfloat(min_value=1, max_value=1000), 2),
            'category_id': fake.random_int(min=1, max=50),
            'stock': fake.random_int(min=0, max=100),
            'is_rental': fake.boolean(chance_of_getting_true=20)
        })

    # Users
    users = []
    for i in range(1, 51):
        users.append({
            'id': i,
            'first_name': fake.first_name(),
            'last_name': fake.last_name(),
            'email': fake.unique.email(),
            'password_hash': fake.sha256(),
            'dob': str(fake.date_of_birth(minimum_age=18, maximum_age=80)),
            'created_at': str(fake.date_time_this_year())
        })

    # Transactions
    transactions = []
    for i in range(1, 1001):
        transactions.append({
            'id': i,
            'user_id': fake.random_int(min=1, max=50),
            'product_id': fake.random_int(min=1, max=1000),
            'quantity': fake.random_int(min=1, max=5),
            'total': round(fake.pyfloat(min_value=10, max_value=5000), 2),
            'status': fake.random_element(['pending', 'completed', 'cancelled']),
            'created_at': str(fake.date_time_this_year())
        })

    return {
        'categories': categories,
        'products': products,
        'users': users,
        'transactions': transactions
    }

# Export to JSON
data = generate_toolshop_data()
with open('test_data.json', 'w') as f:
    json.dump(data, f, indent=2)

# Export to CSV (products example)
with open('products.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=data['products'][0].keys())
    writer.writeheader()
    writer.writerows(data['products'])
```

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
```python
test_cases = [
    # BVA for length
    ("Pass12", "invalid", "7 chars - below min"),
    ("Pass123", "invalid", "7 chars - at boundary-1"),
    ("Pass1234", "valid", "8 chars - at min boundary"),
    ("Pass12345", "valid", "9 chars - above min"),
    ("A" * 19 + "a1", "valid", "20 chars - at max boundary"),
    ("A" * 20 + "a1", "invalid", "21 chars - above max"),

    # ECP for character requirements
    ("password123", "invalid", "no uppercase"),
    ("PASSWORD123", "invalid", "no lowercase"),
    ("Passworddd", "invalid", "no digit"),
    ("Pass word1", "invalid", "contains space"),
    ("Pass1234", "valid", "all requirements met"),

    # Combined edge cases
    ("Aa1" + "x" * 5, "valid", "minimum valid: 8 chars with all reqs"),
    ("Aa1", "invalid", "has all reqs but too short"),
    ("aaa11111", "invalid", "correct length, missing uppercase"),

    # Special characters (if allowed)
    ("Pass123!", "valid", "with special char"),
    ("Pass@#$1", "valid", "multiple special chars"),
]
```

### Question 2: Pairwise Test Reduction
**Scenario:** Testing a form with:
- Country: US, UK, VN (3)
- Payment: Card, Bank, Cash (3)
- Shipping: Standard, Express (2)
- Gift wrap: Yes, No (2)

**Question:** Calculate exhaustive vs pairwise test count.

**Answer:**
```
Exhaustive: 3 × 3 × 2 × 2 = 36 combinations

Pairwise (manually):
| TC | Country | Payment | Shipping | Gift |
|----|---------|---------|----------|------|
| 1  | US      | Card    | Standard | Yes  |
| 2  | US      | Bank    | Express  | No   |
| 3  | US      | Cash    | Standard | No   |
| 4  | UK      | Card    | Express  | No   |
| 5  | UK      | Bank    | Standard | Yes  |
| 6  | UK      | Cash    | Express  | Yes  |
| 7  | VN      | Card    | Standard | Yes  |
| 8  | VN      | Bank    | Express  | Yes  |
| 9  | VN      | Cash    | Standard | No   |

Total: 9 test cases (75% reduction)

Verification: All pairs covered
- US-Card ✓, US-Bank ✓, US-Cash ✓
- UK-Card ✓, UK-Bank ✓, UK-Cash ✓
- VN-Card ✓, VN-Bank ✓, VN-Cash ✓
- Card-Standard ✓, Card-Express ✓
- (etc. for all pairs)
```

### Question 3: Data Generation Strategy
**Scenario:** Need to test:
- 100K users for load testing
- Realistic distribution (60% US, 30% EU, 10% Asia)
- Unique emails
- Valid phone formats per region

**Question:** How would you generate this data?

**Answer:**
```python
from faker import Faker
from collections import defaultdict

def generate_users_with_distribution(count=100000):
    fakers = {
        'US': Faker('en_US'),
        'EU': Faker(['en_GB', 'de_DE', 'fr_FR']),
        'Asia': Faker(['ja_JP', 'ko_KR', 'vi_VN'])
    }

    distribution = {'US': 0.6, 'EU': 0.3, 'Asia': 0.1}
    used_emails = set()
    users = []

    for region, percentage in distribution.items():
        region_count = int(count * percentage)
        fake = fakers[region]

        for _ in range(region_count):
            # Generate unique email
            email = fake.email()
            while email in used_emails:
                email = fake.email()
            used_emails.add(email)

            users.append({
                'name': fake.name(),
                'email': email,
                'phone': fake.phone_number(),  # Format per locale
                'address': fake.address(),
                'region': region
            })

    return users

# Batch insert for performance
def batch_insert(users, batch_size=1000):
    for i in range(0, len(users), batch_size):
        batch = users[i:i+batch_size]
        db.bulk_insert('users', batch)

users = generate_users_with_distribution(100000)
batch_insert(users)
```

---

## 7. Cheatsheet / Quick Reference

### ECP Quick Reference

```
Valid Equivalence Classes:
- All values expected to behave the same way
- Test ONE representative from each class

Invalid Equivalence Classes:
- Test EACH invalid class separately
- Each invalid class may trigger different error

Example: Age 18-65
- Invalid (low): < 18 → Test: 10
- Valid: 18-65 → Test: 40
- Invalid (high): > 65 → Test: 70
```

### BVA Quick Reference

```
Standard 7 values:
- min - 1 (invalid)
- min (boundary)
- min + 1 (just inside)
- nominal (typical)
- max - 1 (just inside)
- max (boundary)
- max + 1 (invalid)

Example: Input 1-100
Test: 0, 1, 2, 50, 99, 100, 101
```

### Faker Cheatsheet

```python
from faker import Faker
fake = Faker()

# Personal
fake.name()              fake.first_name()
fake.email()             fake.phone_number()
fake.address()           fake.city()
fake.ssn()               fake.date_of_birth()

# Business
fake.company()           fake.job()
fake.catch_phrase()      fake.bs()

# Internet
fake.url()               fake.domain_name()
fake.ipv4()              fake.mac_address()
fake.user_agent()        fake.slug()

# Commerce
fake.credit_card_number()
fake.credit_card_expire()
fake.price()             fake.currency()

# Text
fake.text()              fake.sentence()
fake.paragraph()         fake.word()

# Unique values
fake.unique.email()

# Localization
fake_vn = Faker('vi_VN')
```

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
