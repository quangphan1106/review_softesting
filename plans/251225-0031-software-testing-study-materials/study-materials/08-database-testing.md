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

**Atomicity Testing:**
```sql
-- Scenario: Bank transfer between accounts
-- Alice: $100, Bob: $100

BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE name = 'Alice';
-- Simulate failure here
UPDATE accounts SET balance = balance + 50 WHERE name = 'Bob';
COMMIT;

-- Test 1: Successful transaction
-- Expected: Alice = $50, Bob = $150

-- Test 2: Failure after first UPDATE
-- Expected: Rollback, Alice = $100, Bob = $100
```

```python
def test_atomicity_rollback():
    """Verify transaction rolls back completely on failure"""
    # Setup
    db.execute("UPDATE accounts SET balance = 100 WHERE name = 'Alice'")
    db.execute("UPDATE accounts SET balance = 100 WHERE name = 'Bob'")

    # Execute with simulated failure
    try:
        db.begin_transaction()
        db.execute("UPDATE accounts SET balance = balance - 50 WHERE name = 'Alice'")
        raise Exception("Simulated failure")  # Force failure
        db.execute("UPDATE accounts SET balance = balance + 50 WHERE name = 'Bob'")
        db.commit()
    except:
        db.rollback()

    # Verify rollback
    alice_balance = db.query("SELECT balance FROM accounts WHERE name = 'Alice'")
    bob_balance = db.query("SELECT balance FROM accounts WHERE name = 'Bob'")

    assert alice_balance == 100, "Alice's balance should be unchanged"
    assert bob_balance == 100, "Bob's balance should be unchanged"
```

**Consistency Testing:**
```python
def test_consistency_constraint():
    """Verify constraints maintain data consistency"""
    # Test 1: Primary key constraint
    db.execute("INSERT INTO users (id, name) VALUES (1, 'Alice')")
    with pytest.raises(IntegrityError):
        db.execute("INSERT INTO users (id, name) VALUES (1, 'Bob')")  # Duplicate PK

    # Test 2: Foreign key constraint
    with pytest.raises(IntegrityError):
        db.execute("INSERT INTO orders (user_id, amount) VALUES (999, 100)")  # Invalid FK

    # Test 3: Check constraint
    with pytest.raises(IntegrityError):
        db.execute("INSERT INTO products (price) VALUES (-10)")  # Negative price
```

**Isolation Testing:**
```python
import threading

def test_isolation_no_dirty_read():
    """Transaction shouldn't see uncommitted changes"""
    results = []

    def transaction_a():
        db.begin_transaction()
        db.execute("UPDATE accounts SET balance = 200 WHERE name = 'Alice'")
        time.sleep(1)  # Hold transaction open
        db.rollback()  # Rollback the change

    def transaction_b():
        time.sleep(0.5)  # Start after A begins
        balance = db.query("SELECT balance FROM accounts WHERE name = 'Alice'")
        results.append(balance)

    # Run concurrently
    t1 = threading.Thread(target=transaction_a)
    t2 = threading.Thread(target=transaction_b)
    t1.start()
    t2.start()
    t1.join()
    t2.join()

    # B should NOT see A's uncommitted change
    assert results[0] == 100, "Should not see uncommitted value of 200"
```

**Durability Testing:**
```python
def test_durability_after_crash():
    """Committed data should survive crash"""
    # Insert and commit
    db.execute("INSERT INTO logs (message) VALUES ('test')")
    db.commit()

    # Simulate crash (restart connection)
    db.close()
    db.connect()

    # Verify data persists
    result = db.query("SELECT * FROM logs WHERE message = 'test'")
    assert len(result) == 1, "Data should persist after reconnection"
```

---

## 3. Structural Testing

### 3.1 Schema Validation

```python
def test_table_exists():
    """Verify required tables exist"""
    required_tables = ['users', 'products', 'orders', 'payments']
    existing_tables = db.get_table_names()

    for table in required_tables:
        assert table in existing_tables, f"Table {table} is missing"

def test_column_schema():
    """Verify column definitions"""
    expected_schema = {
        'id': {'type': 'INTEGER', 'nullable': False, 'primary_key': True},
        'email': {'type': 'VARCHAR(255)', 'nullable': False},
        'created_at': {'type': 'TIMESTAMP', 'nullable': False}
    }

    actual_schema = db.get_table_schema('users')

    for column, properties in expected_schema.items():
        assert column in actual_schema, f"Column {column} missing"
        assert actual_schema[column]['type'] == properties['type']
        assert actual_schema[column]['nullable'] == properties['nullable']
```

### 3.2 Index Testing

```python
def test_index_exists():
    """Verify indexes are created"""
    indexes = db.get_indexes('orders')

    # Check required indexes
    assert 'idx_orders_user_id' in indexes
    assert 'idx_orders_created_at' in indexes

def test_index_performance():
    """Verify indexes improve query performance"""
    # Insert test data
    for i in range(100000):
        db.execute(f"INSERT INTO orders (user_id, amount) VALUES ({i % 100}, {i})")

    # Query without index (drop temporarily)
    db.execute("DROP INDEX idx_orders_user_id")
    start = time.time()
    db.query("SELECT * FROM orders WHERE user_id = 50")
    without_index = time.time() - start

    # Query with index
    db.execute("CREATE INDEX idx_orders_user_id ON orders(user_id)")
    start = time.time()
    db.query("SELECT * FROM orders WHERE user_id = 50")
    with_index = time.time() - start

    # Index should be significantly faster
    assert with_index < without_index * 0.5, "Index should improve performance by 50%+"
```

---

## 4. Tools & Techniques

### 4.1 DBUnit Overview

**What is DBUnit?**
- Java-based database testing extension
- Manages test data setup and cleanup
- Supports XML, CSV, Excel datasets
- Integrates with JUnit/TestNG

**Key Annotations:**
```java
// Setup data before test
@DatabaseSetup("/data/users.xml")

// Verify final state
@ExpectedDatabase(value="/expected/users.xml", assertionMode=NON_STRICT)

// Clean up after test
@DatabaseTearDown(type=DELETE_ALL, value="/data/users.xml")
```

### 4.2 DBUnit Example

**Dataset XML:**
```xml
<!-- users.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <users id="1" email="alice@test.com" name="Alice" />
    <users id="2" email="bob@test.com" name="Bob" />
</dataset>
```

**Test Class:**
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@TestExecutionListeners({
    DependencyInjectionTestExecutionListener.class,
    DbUnitTestExecutionListener.class
})
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DatabaseSetup("/data/users.xml")
    public void testFindByEmail() {
        User user = userRepository.findByEmail("alice@test.com");

        assertNotNull(user);
        assertEquals("Alice", user.getName());
    }

    @Test
    @DatabaseSetup("/data/users.xml")
    @ExpectedDatabase(value="/expected/users_after_delete.xml",
                      assertionMode=DatabaseAssertionMode.NON_STRICT)
    public void testDeleteUser() {
        userRepository.deleteById(1L);
    }

    @Test
    @DatabaseSetup("/data/empty.xml")
    @ExpectedDatabase(value="/expected/users_after_insert.xml",
                      assertionMode=DatabaseAssertionMode.NON_STRICT)
    public void testInsertUser() {
        User user = new User();
        user.setEmail("charlie@test.com");
        user.setName("Charlie");
        userRepository.save(user);
    }
}
```

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

**Faker Example:**
```python
from faker import Faker

fake = Faker()

def generate_test_users(count=100):
    users = []
    for _ in range(count):
        users.append({
            'id': fake.uuid4(),
            'email': fake.email(),
            'name': fake.name(),
            'created_at': fake.date_time_this_year()
        })
    return users

# Insert into test database
for user in generate_test_users():
    db.execute("INSERT INTO users VALUES (?, ?, ?, ?)", user.values())
```

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
```python
# TC-SEC-001: Classic SQL Injection
test_input = "' OR '1'='1"
# If vulnerable: Returns all users

# TC-SEC-002: UNION-based Injection
test_input = "' UNION SELECT username, password FROM users--"
# If vulnerable: Returns credentials

# TC-SEC-003: Boolean Blind Injection
test_input = "' AND 1=1--"
test_input_false = "' AND 1=2--"
# If vulnerable: Different responses

# TC-SEC-004: Time-based Blind Injection
test_input = "'; WAITFOR DELAY '0:0:5'--"
# If vulnerable: 5-second delay

# TC-SEC-005: Error-based Injection
test_input = "' AND EXTRACTVALUE(1, CONCAT(0x7e, (SELECT version())))--"
# If vulnerable: Version info in error
```

**SQLmap Integration:**
```bash
# Basic scan
sqlmap -u "http://target.com/api?id=1" --dbs

# POST request
sqlmap -u "http://target.com/login" --data="email=test&password=test" --dbs

# Specific database
sqlmap -u "http://target.com/api?id=1" -D targetdb --tables

# Dump table
sqlmap -u "http://target.com/api?id=1" -D targetdb -T users --dump
```

### 5.2 Prevention Testing

```python
def test_parameterized_query_prevents_injection():
    """Verify parameterized queries prevent SQL injection"""
    malicious_input = "'; DROP TABLE users;--"

    # WRONG: Vulnerable to injection
    # query = f"SELECT * FROM users WHERE email = '{malicious_input}'"

    # RIGHT: Parameterized query
    result = db.execute(
        "SELECT * FROM users WHERE email = ?",
        (malicious_input,)
    )

    # Table should still exist
    assert db.table_exists('users'), "Table should not be dropped"

def test_input_validation():
    """Verify input validation rejects malicious patterns"""
    malicious_inputs = [
        "'; DROP TABLE--",
        "' OR '1'='1",
        "1; EXEC xp_cmdshell",
        "UNION SELECT * FROM",
    ]

    for input_val in malicious_inputs:
        with pytest.raises(ValidationError):
            validate_user_input(input_val)
```

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

```python
class TestDataMigration:
    def test_row_count_matches(self, source_db, target_db):
        """Verify all rows migrated"""
        source_count = source_db.query("SELECT COUNT(*) FROM users")[0]
        target_count = target_db.query("SELECT COUNT(*) FROM users")[0]
        assert source_count == target_count

    def test_data_integrity(self, source_db, target_db):
        """Verify data values match"""
        source_data = source_db.query("SELECT * FROM users ORDER BY id")
        target_data = target_db.query("SELECT * FROM users ORDER BY id")

        for source_row, target_row in zip(source_data, target_data):
            assert source_row['email'] == target_row['email']
            assert source_row['name'] == target_row['name']

    def test_null_handling(self, target_db):
        """Verify nulls handled correctly"""
        nulls = target_db.query(
            "SELECT COUNT(*) FROM users WHERE email IS NULL"
        )[0]
        # Depending on migration rules
        assert nulls == 0, "No null emails should exist"

    def test_foreign_key_integrity(self, target_db):
        """Verify relationships preserved"""
        orphans = target_db.query("""
            SELECT COUNT(*) FROM orders o
            LEFT JOIN users u ON o.user_id = u.id
            WHERE u.id IS NULL
        """)[0]
        assert orphans == 0, "No orphan orders should exist"

    def test_rollback(self, source_db, target_db):
        """Verify rollback works"""
        # Take snapshot
        original_count = target_db.query("SELECT COUNT(*) FROM users")[0]

        # Simulate failed migration
        try:
            target_db.begin_transaction()
            target_db.execute("INSERT INTO users (id) VALUES (99999)")
            raise Exception("Simulated failure")
        except:
            target_db.rollback()

        # Verify rollback successful
        current_count = target_db.query("SELECT COUNT(*) FROM users")[0]
        assert current_count == original_count
```

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
```python
# Atomicity: All steps or none
def test_checkout_atomicity():
    """If payment fails, inventory and order should rollback"""
    initial_inventory = db.query("SELECT quantity FROM inventory WHERE id=1")[0]

    try:
        db.begin_transaction()
        db.execute("UPDATE inventory SET quantity = quantity - 1 WHERE id=1")
        db.execute("INSERT INTO orders (product_id, status) VALUES (1, 'pending')")
        raise PaymentFailedException()  # Simulated payment failure
        db.execute("UPDATE orders SET status = 'paid' WHERE id = LAST_INSERT_ID()")
        db.commit()
    except PaymentFailedException:
        db.rollback()

    # Verify rollback
    final_inventory = db.query("SELECT quantity FROM inventory WHERE id=1")[0]
    assert initial_inventory == final_inventory
    order_count = db.query("SELECT COUNT(*) FROM orders WHERE status='pending'")[0]
    assert order_count == 0

# Consistency: Inventory can't go negative
def test_checkout_consistency():
    """Inventory constraint prevents overselling"""
    db.execute("UPDATE inventory SET quantity = 1 WHERE id=1")

    # First order succeeds
    result1 = checkout(product_id=1)
    assert result1.success

    # Second order fails (no inventory)
    result2 = checkout(product_id=1)
    assert not result2.success
    assert "out of stock" in result2.error.lower()

# Isolation: Concurrent checkouts
def test_checkout_isolation():
    """Two concurrent checkouts for last item"""
    db.execute("UPDATE inventory SET quantity = 1 WHERE id=1")
    results = []

    def checkout_thread():
        result = checkout(product_id=1)
        results.append(result)

    t1 = threading.Thread(target=checkout_thread)
    t2 = threading.Thread(target=checkout_thread)
    t1.start(); t2.start()
    t1.join(); t2.join()

    successes = sum(1 for r in results if r.success)
    assert successes == 1, "Only one checkout should succeed"

# Durability: Order survives crash
def test_checkout_durability():
    """Completed order persists after crash"""
    result = checkout(product_id=1)
    order_id = result.order_id

    db.close()
    db.connect()

    order = db.query("SELECT * FROM orders WHERE id=?", (order_id,))
    assert order is not None
    assert order['status'] == 'completed'
```

### Question 2: Index Optimization
**Scenario:** Query is slow:
```sql
SELECT * FROM orders
WHERE user_id = 123
AND created_at > '2024-01-01'
ORDER BY created_at DESC
```

**Question:** Design test to verify index improves performance.

**Answer:**
```python
def test_composite_index_optimization():
    """Verify composite index improves query performance"""
    # Setup: 1M rows of test data
    generate_test_orders(1_000_000)

    # Query without proper index
    start = time.time()
    db.query("""
        SELECT * FROM orders
        WHERE user_id = 123
        AND created_at > '2024-01-01'
        ORDER BY created_at DESC
    """)
    without_index = time.time() - start

    # Create composite index (column order matters!)
    db.execute("""
        CREATE INDEX idx_orders_user_date
        ON orders(user_id, created_at DESC)
    """)

    # Query with index
    start = time.time()
    db.query("""
        SELECT * FROM orders
        WHERE user_id = 123
        AND created_at > '2024-01-01'
        ORDER BY created_at DESC
    """)
    with_index = time.time() - start

    # Assertions
    assert with_index < without_index * 0.1, \
        f"Index should improve by 90%+ (was {with_index}s vs {without_index}s)"

    # Verify index is used (explain plan)
    plan = db.explain("""
        SELECT * FROM orders
        WHERE user_id = 123
        AND created_at > '2024-01-01'
        ORDER BY created_at DESC
    """)
    assert "idx_orders_user_date" in plan, "Index should be used in query plan"
```

### Question 3: SQL Injection Prevention
**Scenario:** Login form tested by security team. They found vulnerability.

**Question:** Design comprehensive SQL injection test suite.

**Answer:**
```python
class TestSQLInjectionPrevention:
    """Comprehensive SQL injection test suite"""

    @pytest.fixture
    def login_endpoint(self):
        return "/api/login"

    # Test payloads
    injection_payloads = [
        ("' OR '1'='1", "Classic OR injection"),
        ("' OR '1'='1'--", "Comment injection"),
        ("'; DROP TABLE users;--", "Destructive injection"),
        ("' UNION SELECT * FROM users--", "UNION injection"),
        ("admin'--", "Authentication bypass"),
        ("1; WAITFOR DELAY '0:0:5'--", "Time-based blind"),
        ("' AND 1=1--", "Boolean blind (true)"),
        ("' AND 1=2--", "Boolean blind (false)"),
    ]

    @pytest.mark.parametrize("payload,description", injection_payloads)
    def test_injection_blocked(self, login_endpoint, payload, description):
        """Each injection payload should be blocked"""
        response = requests.post(login_endpoint, json={
            "email": payload,
            "password": "test"
        })

        # Should NOT succeed
        assert response.status_code != 200, f"Injection not blocked: {description}"

        # Should NOT reveal error details
        assert "sql" not in response.text.lower()
        assert "syntax" not in response.text.lower()
        assert "error" not in response.text.lower()

    def test_timing_attack_resistant(self, login_endpoint):
        """Response time should be consistent regardless of input"""
        normal_time = measure_response_time(login_endpoint, {"email": "test", "password": "test"})
        injection_time = measure_response_time(login_endpoint, {"email": "'; WAITFOR DELAY '0:0:5'--", "password": "test"})

        # Time difference should be minimal
        assert abs(injection_time - normal_time) < 0.5, \
            "Time-based injection may be possible"

    def test_database_integrity_after_attacks(self):
        """Database tables should be intact after attack attempts"""
        # Run attack payloads
        for payload, _ in self.injection_payloads:
            requests.post(login_endpoint, json={"email": payload, "password": "test"})

        # Verify tables still exist
        assert db.table_exists("users")
        assert db.table_exists("orders")
        assert db.table_exists("products")
```

---

## 8. Cheatsheet / Quick Reference

### SQL Test Queries

```sql
-- Table exists
SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'users';

-- Column schema
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'users';

-- Index exists
SELECT * FROM pg_indexes WHERE tablename = 'orders';

-- Foreign key check
SELECT * FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;  -- Find orphans

-- Transaction test
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ROLLBACK or COMMIT;
```

### DBUnit Annotations

```java
// Load test data
@DatabaseSetup("/data/test.xml")

// Verify final state
@ExpectedDatabase(value="/expected/result.xml",
                  assertionMode=DatabaseAssertionMode.NON_STRICT)

// Clean up
@DatabaseTearDown(type=DatabaseOperation.DELETE_ALL)

// Multiple setups
@DatabaseSetup(value={"/data/users.xml", "/data/orders.xml"})
```

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
