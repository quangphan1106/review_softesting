# Topic 8: Database Testing - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 What is Database Testing?

**Definition:** Testing to ensure the quality, accuracy, and security of data stored in a database and the components that interact with it.

**Scope:**
- Data integrity and consistency
- Schema validation (tables, columns, constraints)
- CRUD operations (Create, Read, Update, Delete)
- Stored procedures and triggers
- Performance and scalability
- Security (access control, injection)

### 1.2 Database Testing Layers

```
┌─────────────────────────────────────────────────────────────┐
│                   APPLICATION LAYERS                         │
├─────────────────────────────────────────────────────────────┤
│  UI Layer        → User interface tests                      │
│  Business Logic  → Business rules validation                 │
│  Data Access     → Repository/DAO tests                      │
│  Database        → Database-level tests ← Focus of Topic 8   │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Types of Database Testing

| Category | Focus | Examples |
|----------|-------|----------|
| **Structural** | Schema, constraints | Table validation, index testing |
| **Functional** | CRUD, procedures | Transaction testing, triggers |
| **Non-Functional** | Performance, security | Load testing, SQL injection |

---

## 2. ACID Properties Testing

### 2.1 ACID Overview

| Property | Definition | Test Focus |
|----------|------------|------------|
| **Atomicity** | All or nothing | Transactions rollback completely |
| **Consistency** | Data integrity maintained | Constraints enforced |
| **Isolation** | Concurrent transactions independent | No dirty reads |
| **Durability** | Committed data survives failures | Data persists after crash |

### 2.2 ACID Test Scenarios

**Atomicity Testing (Bank Transfer Example):**
| Test Case | Setup | Action | Expected |
|-----------|-------|--------|----------|
| Successful transfer | Alice=$100, Bob=$100 | Transfer $50, commit | Alice=$50, Bob=$150 |
| Failed transfer | Alice=$100, Bob=$100 | Transfer $50, simulate failure, rollback | Alice=$100, Bob=$100 |

**Test Concept:**
1. Begin transaction
2. Execute first UPDATE (deduct from Alice)
3. Simulate failure before second UPDATE
4. Verify rollback restores original balances

**Consistency Testing (Constraint Violations):**
| Constraint Type | Test Input | Expected |
|-----------------|------------|----------|
| Primary Key | Insert duplicate ID | IntegrityError |
| Foreign Key | Insert with non-existent user_id | IntegrityError |
| CHECK | Insert negative price | IntegrityError |

**Isolation Testing (No Dirty Reads):**
| Thread A | Thread B | Expected |
|----------|----------|----------|
| Begin, UPDATE balance=200 | Wait 0.5s, then SELECT | B sees original value (100) |
| Wait 1s, ROLLBACK | Read result | NOT 200 (uncommitted) |

**Test Concept:** Two concurrent threads - A updates but doesn't commit, B should NOT see A's uncommitted changes.

**Durability Testing:**
| Step | Action | Verification |
|------|--------|--------------|
| 1 | INSERT + COMMIT | Data saved |
| 2 | Close connection (simulate crash) | - |
| 3 | Reconnect + SELECT | Data persists |

---

## 3. Structural Testing

### 3.1 Schema Validation

**Test Concepts:**

| Test | What to Check | How |
|------|---------------|-----|
| Table exists | Required tables present | Query `information_schema.tables` |
| Column schema | Type, nullable, primary key | Query `information_schema.columns` |
| Constraints | PK, FK, CHECK defined | Query `information_schema.table_constraints` |

**Schema Validation Checklist:**
- [ ] All required tables exist (users, products, orders, payments)
- [ ] Column types match specification
- [ ] Nullable flags correct
- [ ] Primary keys defined
- [ ] Foreign keys reference valid tables

### 3.2 Index Testing

**Index Existence Test:**
- Query system catalog for expected indexes
- Verify: `idx_orders_user_id`, `idx_orders_created_at` exist

**Index Performance Test:**
| Step | Action | Measure |
|------|--------|---------|
| 1 | Insert 100K test rows | - |
| 2 | Query without index | Record time |
| 3 | Create index | - |
| 4 | Query with index | Record time |
| 5 | Compare | Index should be 50%+ faster |

**Key Assertion:** `with_index_time < without_index_time * 0.5`

---

## 4. Tools & Techniques

### 4.1 DBUnit Overview

**What is DBUnit?**
- Java-based database testing extension
- Manages test data setup and cleanup
- Supports XML, CSV, Excel datasets
- Integrates with JUnit/TestNG

**Key Annotations:**

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@DatabaseSetup` | Load test data before test | `@DatabaseSetup("/data/users.xml")` |
| `@ExpectedDatabase` | Verify final state after test | `@ExpectedDatabase(value="/expected/result.xml")` |
| `@DatabaseTearDown` | Clean up after test | `@DatabaseTearDown(type=DELETE_ALL)` |

### 4.2 DBUnit Example

**Dataset Format (XML):**
- Each element = table row
- Attributes = column values
- Example: `<users id="1" email="alice@test.com" name="Alice" />`

**Test Pattern:**

| Test Type | Annotations | Verification |
|-----------|-------------|--------------|
| Read test | `@DatabaseSetup` | Assert returned data matches |
| Delete test | `@DatabaseSetup` + `@ExpectedDatabase` | Expected file has row removed |
| Insert test | `@DatabaseSetup("/empty.xml")` + `@ExpectedDatabase` | Expected file has new row |

**Assertion Modes:**
- `NON_STRICT`: Ignore extra columns/rows in actual
- `STRICT`: Exact match required

### 4.3 Data Generation Tools

| Tool | Type | Best For |
|------|------|----------|
| **Mockaroo** | Web UI | Quick data, non-developers |
| **Faker** | Code library | Programmatic generation |
| **DBUnit** | XML datasets | Test fixtures |

**Mockaroo Usage:**
1. Define schema fields (name, type, format)
2. Generate up to 1,000 rows (free tier)
3. Export as CSV, JSON, SQL, XML

**Faker Library:**

| Method | Returns |
|--------|---------|
| `fake.uuid4()` | Random UUID |
| `fake.email()` | Random email |
| `fake.name()` | Random full name |
| `fake.date_time_this_year()` | Random datetime |
| `fake.address()` | Random address |
| `fake.phone_number()` | Random phone |

**Usage Pattern:**
- Create `Faker()` instance
- Generate N test records with random data
- Insert into test database via parameterized queries

---

## 5. Security Testing

### 5.1 SQL Injection Testing

**Types of SQL Injection:**

| Type | Technique | Detection |
|------|-----------|-----------|
| **In-band** | UNION-based | Error messages, data in response |
| **Blind** | Boolean-based | Different responses for true/false |
| **Time-based** | SLEEP queries | Response delay differences |
| **Out-of-band** | DNS/HTTP exfiltration | External monitoring |
| **Error-based** | Force DB errors | Error messages reveal data |

**Test Cases:**

| TC ID | Type | Payload | Vulnerable Behavior |
|-------|------|---------|---------------------|
| SEC-001 | Classic | `' OR '1'='1` | Returns all users |
| SEC-002 | UNION | `' UNION SELECT username, password FROM users--` | Returns credentials |
| SEC-003 | Boolean Blind | `' AND 1=1--` vs `' AND 1=2--` | Different responses |
| SEC-004 | Time-based | `'; WAITFOR DELAY '0:0:5'--` | 5-second delay |
| SEC-005 | Error-based | `' AND EXTRACTVALUE(...)--` | Version info in error |

**SQLmap Commands:**

| Command | Purpose |
|---------|---------|
| `sqlmap -u "URL?id=1" --dbs` | Basic scan, list databases |
| `sqlmap -u "URL" --data="email=x" --dbs` | POST request scan |
| `sqlmap -u "URL?id=1" -D db --tables` | List tables in database |
| `sqlmap -u "URL?id=1" -D db -T users --dump` | Dump table data |

### 5.2 Prevention Testing

**Parameterized Query Test:**
- Send malicious input: `'; DROP TABLE users;--`
- Use parameterized query: `db.execute("SELECT * FROM users WHERE email = ?", (input,))`
- Verify: Table still exists after query

**Input Validation Test:**
| Malicious Input | Expected |
|-----------------|----------|
| `'; DROP TABLE--` | ValidationError |
| `' OR '1'='1` | ValidationError |
| `1; EXEC xp_cmdshell` | ValidationError |
| `UNION SELECT * FROM` | ValidationError |

---

## 6. Data Migration Testing

### 6.1 Migration Testing Strategy

```
┌─────────────────────────────────────────────────────────────┐
│                  DATA MIGRATION PHASES                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  PRE-MIGRATION        MIGRATION           POST-MIGRATION    │
│  ┌──────────┐        ┌──────────┐        ┌──────────┐      │
│  │ Validate │   →    │ Execute  │   →    │ Verify   │      │
│  │ Source   │        │ Transfer │        │ Target   │      │
│  └──────────┘        └──────────┘        └──────────┘      │
│       │                   │                   │             │
│   Row counts          Transform           Reconcile         │
│   Data types          Map fields          Test rollback     │
│   Constraints         Handle nulls        Verify integrity  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Migration Test Cases

| Test | What to Verify | How |
|------|----------------|-----|
| **Row count** | All rows migrated | `COUNT(*)` source = target |
| **Data integrity** | Values match | Compare each row field-by-field |
| **Null handling** | Nulls processed correctly | Check no unexpected NULLs |
| **FK integrity** | Relationships preserved | Find orphan records (LEFT JOIN WHERE NULL) |
| **Rollback** | Failed migration reverts | Simulate failure, check original state |

**Key Queries:**

| Verification | Query Pattern |
|--------------|---------------|
| Count match | `SELECT COUNT(*) FROM table` on both DBs |
| Find orphans | `SELECT COUNT(*) FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL` |
| Null check | `SELECT COUNT(*) FROM users WHERE email IS NULL` |

**Rollback Test Pattern:**
1. Record original count
2. Begin transaction
3. Insert test data
4. Simulate failure (raise exception)
5. Rollback
6. Verify count unchanged

---

## 7. Application-Level Exam Questions

### Question 1: ACID Testing Scenario
**Scenario:** E-commerce checkout process:
1. Deduct item from inventory
2. Create order record
3. Process payment
4. Send confirmation

**Question:** How would you test ACID properties?

**Answer:**

| Property | Test Scenario | Expected Result |
|----------|---------------|-----------------|
| **Atomicity** | Payment fails after inventory deduction | Rollback: inventory restored, no order created |
| **Consistency** | Two orders for last item in stock | First succeeds, second fails ("out of stock") |
| **Isolation** | Two concurrent checkouts for last item | Only ONE succeeds |
| **Durability** | Complete order, then crash (close DB) | Order persists after reconnect |

**Atomicity Test Steps:**
1. Record initial inventory
2. Begin transaction: UPDATE inventory, INSERT order
3. Simulate payment failure (raise exception)
4. Rollback
5. Verify: inventory unchanged, no pending orders

**Consistency Test Steps:**
1. Set inventory = 1
2. First checkout → success
3. Second checkout → fail with "out of stock"

**Isolation Test Steps:**
1. Set inventory = 1
2. Run 2 concurrent checkout threads
3. Assert: exactly 1 success, 1 failure

**Durability Test Steps:**
1. Complete checkout, get order_id
2. Close DB connection
3. Reconnect and query order
4. Verify: order exists, status = 'completed'

### Question 2: Index Optimization
**Scenario:** Query is slow:
- `SELECT * FROM orders WHERE user_id = 123 AND created_at > '2024-01-01' ORDER BY created_at DESC`

**Question:** Design test to verify index improves performance.

**Answer:**

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Generate 1M test rows | Realistic data volume |
| 2 | Time query WITHOUT index | Baseline performance |
| 3 | Create composite index | `CREATE INDEX idx_orders_user_date ON orders(user_id, created_at DESC)` |
| 4 | Time query WITH index | Compare performance |
| 5 | Run EXPLAIN | Verify index is used |

**Key Assertions:**
- `with_index_time < without_index_time * 0.1` (90%+ improvement)
- EXPLAIN plan shows `idx_orders_user_date` being used

**Important:** Column order in composite index matters! Put equality columns first (`user_id`), then range/sort columns (`created_at`).

### Question 3: SQL Injection Prevention
**Scenario:** Login form tested by security team. They found vulnerability.

**Question:** Design comprehensive SQL injection test suite.

**Answer:**

**Test Payloads:**

| Payload | Type | Purpose |
|---------|------|---------|
| `' OR '1'='1` | Classic | Authentication bypass |
| `' OR '1'='1'--` | Comment | Ignore password check |
| `'; DROP TABLE users;--` | Destructive | Delete data |
| `' UNION SELECT * FROM users--` | UNION | Extract data |
| `admin'--` | Auth bypass | Login as admin |
| `1; WAITFOR DELAY '0:0:5'--` | Time-based | Detect vulnerability |
| `' AND 1=1--` / `' AND 1=2--` | Boolean blind | Different responses |

**Test Cases:**

| Test | Action | Expected |
|------|--------|----------|
| **Injection blocked** | Send each payload to login | Status != 200 |
| **No error details** | Check response text | No "sql", "syntax", "error" |
| **Timing resistant** | Compare normal vs injection response time | Difference < 0.5s |
| **DB integrity** | Run all payloads, check tables | Tables still exist |

**Key Assertions:**
- All injection payloads return non-200 status
- Response doesn't reveal SQL syntax errors
- Response time consistent (no time-based attacks)
- Tables (users, orders, products) intact after attacks

---

## 8. Cheatsheet / Quick Reference

### SQL Test Queries

| Purpose | Query |
|---------|-------|
| Table exists | `SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'users'` |
| Column schema | `SELECT column_name, data_type, is_nullable FROM information_schema.columns WHERE table_name = 'users'` |
| Index exists (PostgreSQL) | `SELECT * FROM pg_indexes WHERE tablename = 'orders'` |
| Find orphan records | `SELECT * FROM orders o LEFT JOIN users u ON o.user_id = u.id WHERE u.id IS NULL` |
| Transaction pattern | `BEGIN; UPDATE ...; ROLLBACK; -- or COMMIT;` |

### DBUnit Annotations

| Annotation | Purpose | Example |
|------------|---------|---------|
| `@DatabaseSetup` | Load test data | `@DatabaseSetup("/data/test.xml")` |
| `@ExpectedDatabase` | Verify final state | `@ExpectedDatabase(value="/expected/result.xml", assertionMode=NON_STRICT)` |
| `@DatabaseTearDown` | Clean up | `@DatabaseTearDown(type=DELETE_ALL)` |
| Multiple files | Load multiple datasets | `@DatabaseSetup(value={"/data/users.xml", "/data/orders.xml"})` |

### Database Testing Checklist

- [ ] Schema validation (tables, columns, types)
- [ ] Constraint testing (PK, FK, CHECK)
- [ ] ACID properties (atomicity, consistency, isolation, durability)
- [ ] Index performance testing
- [ ] Stored procedure/trigger testing
- [ ] SQL injection prevention
- [ ] Data migration validation
- [ ] Backup/restore testing
- [ ] Performance under load

---

## 9. Key Takeaways for Exam

1. **ACID**: Test each property with specific scenarios
2. **DBUnit**: @DatabaseSetup/@ExpectedDatabase for test data
3. **Index Testing**: Verify performance improvement with EXPLAIN
4. **SQL Injection**: Test multiple payload types, check timing
5. **Migration**: Pre/during/post validation, test rollback
6. **Isolation Levels**: Read committed prevents dirty reads
7. **Constraints**: PK, FK, CHECK, NOT NULL prevent bad data

---

*Study time recommended: 3-4 hours*
*Practice: Set up DBUnit with a sample Spring Boot project*
