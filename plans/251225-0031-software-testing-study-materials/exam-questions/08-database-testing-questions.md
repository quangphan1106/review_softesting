# Topic 8: Database Testing - Câu Hỏi Vận Dụng
**CS423 Software Testing | Final Exam Preparation**

---

## Câu 1: ACID Properties Testing (10 điểm)

### Tình huống:
Bạn đang phát triển hệ thống đặt vé máy bay. Quy trình đặt vé bao gồm:
1. Kiểm tra số ghế còn trống
2. Trừ số ghế khả dụng
3. Tạo booking record
4. Xử lý thanh toán
5. Gửi confirmation email

Trong 1 giây, có thể có 100+ người cùng đặt ghế cuối cùng của chuyến bay.

### Yêu cầu:
a) Thiết kế test case kiểm tra Atomicity khi payment fails (3 điểm)
b) Thiết kế test case kiểm tra Isolation cho concurrent bookings (4 điểm)
c) Giải thích cách test Durability sau system crash (3 điểm)

### Đáp án mẫu:

**a) Atomicity Testing (3 điểm):**
```python
def test_booking_atomicity_payment_failure():
    """Verify complete rollback when payment fails"""
    # Setup
    flight_id = 1
    db.execute("UPDATE flights SET available_seats = 1 WHERE id = ?", (flight_id,))
    initial_seats = db.query("SELECT available_seats FROM flights WHERE id = ?", (flight_id,))[0]
    initial_bookings = db.query("SELECT COUNT(*) FROM bookings WHERE flight_id = ?", (flight_id,))[0]

    # Execute with simulated payment failure
    try:
        db.begin_transaction()

        # Step 1: Check seats
        seats = db.query("SELECT available_seats FROM flights WHERE id = ? FOR UPDATE", (flight_id,))[0]
        assert seats > 0, "No seats available"

        # Step 2: Deduct seat
        db.execute("UPDATE flights SET available_seats = available_seats - 1 WHERE id = ?", (flight_id,))

        # Step 3: Create booking
        db.execute("INSERT INTO bookings (flight_id, status) VALUES (?, 'pending')", (flight_id,))

        # Step 4: Payment fails
        raise PaymentFailedException("Card declined")

        # Step 5: Would send email
        db.commit()
    except PaymentFailedException:
        db.rollback()

    # Verify complete rollback
    final_seats = db.query("SELECT available_seats FROM flights WHERE id = ?", (flight_id,))[0]
    final_bookings = db.query("SELECT COUNT(*) FROM bookings WHERE flight_id = ?", (flight_id,))[0]

    assert final_seats == initial_seats, "Seats should be restored"
    assert final_bookings == initial_bookings, "No booking should be created"
```

**b) Isolation Testing - Concurrent Bookings (4 điểm):**
```python
import threading
import time

def test_booking_isolation_last_seat():
    """Only one concurrent booking should succeed for last seat"""
    flight_id = 1
    db.execute("UPDATE flights SET available_seats = 1 WHERE id = ?", (flight_id,))

    results = []
    errors = []

    def book_seat(user_id):
        try:
            conn = get_new_connection()  # Each thread needs own connection
            conn.begin_transaction()

            # SELECT FOR UPDATE creates row-level lock
            seats = conn.query(
                "SELECT available_seats FROM flights WHERE id = ? FOR UPDATE",
                (flight_id,)
            )[0]

            if seats <= 0:
                conn.rollback()
                errors.append(f"User {user_id}: No seats")
                return

            # Simulate processing time
            time.sleep(0.1)

            conn.execute(
                "UPDATE flights SET available_seats = available_seats - 1 WHERE id = ?",
                (flight_id,)
            )
            conn.execute(
                "INSERT INTO bookings (flight_id, user_id, status) VALUES (?, ?, 'confirmed')",
                (flight_id, user_id)
            )
            conn.commit()
            results.append(f"User {user_id}: Success")
        except Exception as e:
            conn.rollback()
            errors.append(f"User {user_id}: {str(e)}")

    # Launch 10 concurrent booking attempts
    threads = [threading.Thread(target=book_seat, args=(i,)) for i in range(10)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()

    # Assertions
    successful_bookings = len([r for r in results if "Success" in r])
    assert successful_bookings == 1, f"Only 1 should succeed, got {successful_bookings}"

    final_seats = db.query("SELECT available_seats FROM flights WHERE id = ?", (flight_id,))[0]
    assert final_seats == 0, "No seats should remain"

    booking_count = db.query("SELECT COUNT(*) FROM bookings WHERE flight_id = ?", (flight_id,))[0]
    assert booking_count == 1, "Exactly 1 booking should exist"
```

**c) Durability Testing (3 điểm):**
```python
def test_booking_durability_after_crash():
    """Committed booking should survive system crash"""
    # Step 1: Create and commit booking
    db.begin_transaction()
    db.execute("INSERT INTO bookings (flight_id, user_id, status) VALUES (1, 100, 'confirmed')")
    booking_id = db.query("SELECT LAST_INSERT_ID()")[0]
    db.commit()  # Data is now durable

    # Step 2: Simulate crash by force-closing connection
    db.force_close()  # Không graceful shutdown

    # Step 3: Reconnect (simulates server restart)
    db = DatabaseConnection()
    db.connect()

    # Step 4: Verify data persists
    booking = db.query("SELECT * FROM bookings WHERE id = ?", (booking_id,))
    assert booking is not None, "Booking should exist after crash"
    assert booking['status'] == 'confirmed', "Status should be preserved"
    assert booking['user_id'] == 100, "User ID should be preserved"

# Additional durability test: WAL (Write-Ahead Logging)
def test_durability_with_wal():
    """Verify WAL ensures durability"""
    # Check WAL is enabled
    wal_status = db.query("SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit'")[0]
    assert wal_status['Value'] == '1', "WAL should flush on every commit"

    # Perform write and verify fsync
    db.execute("INSERT INTO audit_log (action) VALUES ('test')")
    db.commit()

    # Data should be on disk, not just in buffer
    # Verify by checking file modification time or using sync tools
```

---

## Câu 2: DBUnit Test Data Management (10 điểm)

### Tình huống:
Bạn đang test UserRepository trong Spring Boot application. Repository có các methods:
- `findByEmail(String email)`
- `findActiveUsers()`
- `deactivateUser(Long id)`
- `updateLastLogin(Long id, Timestamp time)`

### Yêu cầu:
a) Viết DBUnit dataset XML cho test data (3 điểm)
b) Viết test class với @DatabaseSetup và @ExpectedDatabase (4 điểm)
c) Giải thích assertionMode NON_STRICT vs NON_STRICT_UNORDERED (3 điểm)

### Đáp án mẫu:

**a) DBUnit Dataset XML (3 điểm):**
```xml
<!-- src/test/resources/data/users-setup.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <users id="1"
           email="alice@test.com"
           name="Alice"
           status="ACTIVE"
           last_login="2024-01-15 10:00:00"
           created_at="2024-01-01 00:00:00"/>
    <users id="2"
           email="bob@test.com"
           name="Bob"
           status="ACTIVE"
           last_login="2024-01-10 08:00:00"
           created_at="2024-01-02 00:00:00"/>
    <users id="3"
           email="charlie@test.com"
           name="Charlie"
           status="INACTIVE"
           last_login="2023-12-01 12:00:00"
           created_at="2024-01-03 00:00:00"/>
</dataset>

<!-- src/test/resources/expected/users-after-deactivate.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <users id="1"
           email="alice@test.com"
           name="Alice"
           status="INACTIVE"/>  <!-- Changed from ACTIVE -->
    <users id="2"
           email="bob@test.com"
           name="Bob"
           status="ACTIVE"/>
    <users id="3"
           email="charlie@test.com"
           name="Charlie"
           status="INACTIVE"/>
</dataset>

<!-- src/test/resources/data/empty-users.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<dataset>
    <users/>  <!-- Empty table definition -->
</dataset>
```

**b) Test Class với DBUnit Annotations (4 điểm):**
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@TestExecutionListeners({
    DependencyInjectionTestExecutionListener.class,
    TransactionalTestExecutionListener.class,
    DbUnitTestExecutionListener.class
})
@DbUnitConfiguration(databaseConnection = "dbUnitConnection")
public class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    @DatabaseSetup("/data/users-setup.xml")
    public void testFindByEmail_ExistingUser_ReturnsUser() {
        // Act
        User user = userRepository.findByEmail("alice@test.com");

        // Assert
        assertNotNull(user);
        assertEquals("Alice", user.getName());
        assertEquals(UserStatus.ACTIVE, user.getStatus());
    }

    @Test
    @DatabaseSetup("/data/users-setup.xml")
    public void testFindByEmail_NonExisting_ReturnsNull() {
        // Act
        User user = userRepository.findByEmail("unknown@test.com");

        // Assert
        assertNull(user);
    }

    @Test
    @DatabaseSetup("/data/users-setup.xml")
    public void testFindActiveUsers_ReturnsOnlyActive() {
        // Act
        List<User> activeUsers = userRepository.findActiveUsers();

        // Assert
        assertEquals(2, activeUsers.size());
        assertTrue(activeUsers.stream().allMatch(u -> u.getStatus() == UserStatus.ACTIVE));
        assertTrue(activeUsers.stream().anyMatch(u -> u.getEmail().equals("alice@test.com")));
        assertTrue(activeUsers.stream().anyMatch(u -> u.getEmail().equals("bob@test.com")));
    }

    @Test
    @DatabaseSetup("/data/users-setup.xml")
    @ExpectedDatabase(
        value = "/expected/users-after-deactivate.xml",
        assertionMode = DatabaseAssertionMode.NON_STRICT
    )
    public void testDeactivateUser_UpdatesStatus() {
        // Act
        userRepository.deactivateUser(1L);

        // Note: @ExpectedDatabase automatically verifies final state
    }

    @Test
    @DatabaseSetup("/data/users-setup.xml")
    public void testUpdateLastLogin_UpdatesTimestamp() {
        // Arrange
        Timestamp newLoginTime = Timestamp.valueOf("2024-01-20 15:30:00");

        // Act
        userRepository.updateLastLogin(1L, newLoginTime);

        // Assert
        User user = userRepository.findById(1L).orElseThrow();
        assertEquals(newLoginTime, user.getLastLogin());
    }

    @Test
    @DatabaseSetup("/data/empty-users.xml")
    @ExpectedDatabase(
        value = "/expected/users-after-insert.xml",
        assertionMode = DatabaseAssertionMode.NON_STRICT_UNORDERED
    )
    public void testSaveUser_InsertsNewRecord() {
        // Arrange
        User newUser = new User();
        newUser.setEmail("david@test.com");
        newUser.setName("David");
        newUser.setStatus(UserStatus.ACTIVE);

        // Act
        userRepository.save(newUser);
    }
}
```

**c) Assertion Modes Explanation (3 điểm):**

| Mode | Behavior | Use Case |
|------|----------|----------|
| **STRICT** | Exact match required | All columns, all rows, exact order |
| **NON_STRICT** | Ignore missing columns in expected | Only verify specified columns |
| **NON_STRICT_UNORDERED** | Ignore order + missing columns | When row order doesn't matter |

```java
// NON_STRICT example:
// Expected XML chỉ có: id, email, status
// Actual table có: id, email, name, status, last_login, created_at
// → CHỈ so sánh 3 columns trong expected, IGNORE các columns khác

@ExpectedDatabase(
    value = "/expected/partial-columns.xml",
    assertionMode = DatabaseAssertionMode.NON_STRICT  // OK: ignores name, last_login, created_at
)

// NON_STRICT_UNORDERED example:
// Expected rows: [Bob, Alice, Charlie]
// Actual rows: [Alice, Bob, Charlie]
// → Vẫn PASS vì không quan tâm thứ tự

@ExpectedDatabase(
    value = "/expected/users-any-order.xml",
    assertionMode = DatabaseAssertionMode.NON_STRICT_UNORDERED  // OK: order doesn't matter
)

// STRICT sẽ FAIL nếu:
// - Có column trong actual không có trong expected
// - Thứ tự rows khác nhau
// - Bất kỳ giá trị nào khác
```

---

## Câu 3: SQL Injection Testing (10 điểm)

### Tình huống:
Security team phát hiện login endpoint có thể bị SQL injection:
```python
# Vulnerable code
def login(email, password):
    query = f"SELECT * FROM users WHERE email = '{email}' AND password = '{password}'"
    user = db.execute(query).fetchone()
    return user
```

### Yêu cầu:
a) Liệt kê 5 payloads SQL injection cần test (2 điểm)
b) Viết test suite kiểm tra SQL injection prevention (5 điểm)
c) Viết code fix vulnerability và test verify fix (3 điểm)

### Đáp án mẫu:

**a) SQL Injection Payloads (2 điểm):**

| # | Payload | Type | Attack Goal |
|---|---------|------|-------------|
| 1 | `' OR '1'='1` | Authentication Bypass | Login without password |
| 2 | `admin'--` | Comment Injection | Bypass password check |
| 3 | `'; DROP TABLE users;--` | Destructive | Delete data |
| 4 | `' UNION SELECT * FROM admin_users--` | Data Extraction | Access sensitive data |
| 5 | `'; WAITFOR DELAY '0:0:5'--` | Time-based Blind | Detect vulnerability |

**b) SQL Injection Prevention Test Suite (5 điểm):**
```python
import pytest
import requests
import time

class TestSQLInjectionPrevention:
    """Comprehensive SQL injection test suite"""

    BASE_URL = "http://localhost:8000/api"

    # Test payloads với expected behavior
    INJECTION_PAYLOADS = [
        # (payload, description, should_be_blocked)
        ("' OR '1'='1", "Classic OR injection", True),
        ("' OR '1'='1'--", "Comment injection", True),
        ("' OR '1'='1'/*", "Block comment injection", True),
        ("admin'--", "Authentication bypass", True),
        ("admin' #", "MySQL comment injection", True),
        ("'; DROP TABLE users;--", "Destructive DROP", True),
        ("'; DELETE FROM users;--", "Destructive DELETE", True),
        ("'; UPDATE users SET role='admin';--", "Privilege escalation", True),
        ("' UNION SELECT username,password FROM users--", "UNION extraction", True),
        ("' UNION SELECT null,null,null--", "UNION column detection", True),
        ("1; WAITFOR DELAY '0:0:5'--", "Time-based blind (MSSQL)", True),
        ("1'; SELECT SLEEP(5);--", "Time-based blind (MySQL)", True),
        ("' AND 1=1--", "Boolean blind (true)", True),
        ("' AND 1=2--", "Boolean blind (false)", True),
        ("' AND SUBSTRING(username,1,1)='a'--", "Blind data extraction", True),
        ("\\x27 OR 1=1", "Hex encoded", True),
        ("%27%20OR%20%271%27%3D%271", "URL encoded", True),
    ]

    @pytest.fixture(autouse=True)
    def setup(self):
        """Ensure test user exists and tables are intact"""
        self.valid_credentials = {
            "email": "test@example.com",
            "password": "ValidPassword123"
        }
        # Verify baseline state
        assert self.table_exists("users"), "Users table should exist before test"

    @pytest.mark.parametrize("payload,description,should_block", INJECTION_PAYLOADS)
    def test_injection_blocked_in_email(self, payload, description, should_block):
        """Each injection payload in email field should be blocked"""
        response = requests.post(f"{self.BASE_URL}/login", json={
            "email": payload,
            "password": "anything"
        })

        # Should NOT return 200 (successful login)
        assert response.status_code != 200, f"Injection not blocked: {description}"

        # Should NOT leak SQL error details
        response_text = response.text.lower()
        assert "sql" not in response_text, "SQL error leaked"
        assert "syntax" not in response_text, "Syntax error leaked"
        assert "query" not in response_text, "Query info leaked"
        assert "mysql" not in response_text, "Database type leaked"
        assert "postgresql" not in response_text, "Database type leaked"

    @pytest.mark.parametrize("payload,description,should_block", INJECTION_PAYLOADS)
    def test_injection_blocked_in_password(self, payload, description, should_block):
        """Each injection payload in password field should be blocked"""
        response = requests.post(f"{self.BASE_URL}/login", json={
            "email": "valid@test.com",
            "password": payload
        })

        assert response.status_code != 200, f"Injection not blocked: {description}"

    def test_time_based_injection_resistant(self):
        """Response time should be consistent regardless of payload"""
        # Baseline with normal request
        start = time.time()
        requests.post(f"{self.BASE_URL}/login", json=self.valid_credentials)
        normal_time = time.time() - start

        # Test with time-based injection
        time_payloads = [
            "'; WAITFOR DELAY '0:0:5'--",
            "'; SELECT SLEEP(5);--",
            "' OR SLEEP(5)--",
        ]

        for payload in time_payloads:
            start = time.time()
            requests.post(f"{self.BASE_URL}/login", json={
                "email": payload,
                "password": "test"
            })
            injection_time = time.time() - start

            # Difference should be < 1 second (not 5 seconds)
            assert abs(injection_time - normal_time) < 1.0, \
                f"Time-based injection may be possible: {payload}"

    def test_database_intact_after_attacks(self):
        """Database tables should be intact after attack attempts"""
        # Run all attack payloads
        for payload, _, _ in self.INJECTION_PAYLOADS:
            requests.post(f"{self.BASE_URL}/login", json={
                "email": payload,
                "password": payload
            })

        # Verify all critical tables still exist
        critical_tables = ["users", "orders", "products", "payments"]
        for table in critical_tables:
            assert self.table_exists(table), f"Table {table} was deleted!"

    def test_valid_login_still_works(self):
        """Valid login should work after security measures"""
        response = requests.post(f"{self.BASE_URL}/login", json=self.valid_credentials)

        assert response.status_code == 200
        assert "token" in response.json()

    def table_exists(self, table_name):
        """Helper to check if table exists"""
        result = db.query(
            "SELECT COUNT(*) FROM information_schema.tables WHERE table_name = ?",
            (table_name,)
        )
        return result[0] > 0
```

**c) Fix Vulnerability và Verification (3 điểm):**
```python
# FIXED CODE - Using Parameterized Queries
def login_secure(email, password):
    """Secure login using parameterized queries"""
    # Option 1: Parameterized query (RECOMMENDED)
    query = "SELECT * FROM users WHERE email = ? AND password = ?"
    user = db.execute(query, (email, password)).fetchone()

    # Option 2: Using ORM (SQLAlchemy)
    # user = session.query(User).filter(
    #     User.email == email,
    #     User.password == password
    # ).first()

    return user

# Test verifying fix works
def test_parameterized_query_prevents_injection():
    """Verify parameterized queries prevent all injection types"""
    malicious_inputs = [
        "' OR '1'='1",
        "'; DROP TABLE users;--",
        "admin'--",
    ]

    for payload in malicious_inputs:
        # Should NOT authenticate
        result = login_secure(payload, "anything")
        assert result is None, f"Injection succeeded with: {payload}"

        # Should NOT damage database
        assert db.table_exists("users"), "Table was dropped!"

    # Valid login should still work
    result = login_secure("real@user.com", "correctpassword")
    assert result is not None, "Valid login should work"

def test_special_chars_handled_safely():
    """Verify special characters are escaped properly"""
    # These are VALID inputs that look like SQL but aren't attacks
    edge_cases = [
        ("o'reilly@test.com", "pass123"),  # Irish name with apostrophe
        ("user+tag@test.com", "test'123"),  # Email plus addressing
    ]

    for email, password in edge_cases:
        # Should NOT throw SQL error
        try:
            result = login_secure(email, password)
            # Result may be None (user doesn't exist) but no exception
        except Exception as e:
            pytest.fail(f"Valid input caused error: {email} - {e}")
```

---

## Câu 4: Data Migration Testing (10 điểm)

### Tình huống:
Bạn đang migrate database từ MySQL sang PostgreSQL cho hệ thống e-commerce:
- 500,000 users
- 2,000,000 orders
- 10,000,000 order_items

Schema có sự thay đổi:
- `users.phone` VARCHAR(20) → `users.phone_number` VARCHAR(15) với format validation
- `orders.status` ENUM → `orders.status` với foreign key tới `order_statuses` table
- `order_items.price` DECIMAL(10,2) → NUMERIC(12,4)

### Yêu cầu:
a) Thiết kế pre-migration validation tests (3 điểm)
b) Thiết kế post-migration verification tests (4 điểm)
c) Viết rollback test scenario (3 điểm)

### Đáp án mẫu:

**a) Pre-Migration Validation Tests (3 điểm):**
```python
class TestPreMigrationValidation:
    """Validate source data before migration"""

    def test_source_row_counts(self, source_db):
        """Record baseline counts for verification"""
        counts = {
            'users': source_db.query("SELECT COUNT(*) FROM users")[0],
            'orders': source_db.query("SELECT COUNT(*) FROM orders")[0],
            'order_items': source_db.query("SELECT COUNT(*) FROM order_items")[0],
        }

        # Save for post-migration comparison
        save_baseline_counts(counts)

        assert counts['users'] == 500000, "Unexpected user count"
        assert counts['orders'] == 2000000, "Unexpected order count"
        assert counts['order_items'] == 10000000, "Unexpected order_items count"

    def test_phone_format_compatibility(self, source_db):
        """Identify phones that won't fit in new VARCHAR(15)"""
        long_phones = source_db.query("""
            SELECT id, phone, LENGTH(phone) as len
            FROM users
            WHERE LENGTH(phone) > 15
        """)

        if long_phones:
            # Log for manual review/transformation
            log_data_issues("phones_too_long", long_phones)

        # Report but don't fail - transformation will handle
        print(f"Found {len(long_phones)} phones needing transformation")

    def test_order_status_mapping(self, source_db):
        """Verify all status values can be mapped"""
        source_statuses = source_db.query("""
            SELECT DISTINCT status FROM orders
        """)

        valid_mappings = {
            'pending': 1,
            'processing': 2,
            'shipped': 3,
            'delivered': 4,
            'cancelled': 5,
        }

        for row in source_statuses:
            status = row['status']
            assert status in valid_mappings, f"Unknown status: {status}"

    def test_price_precision(self, source_db):
        """Verify prices won't lose precision in new format"""
        precision_issues = source_db.query("""
            SELECT id, price
            FROM order_items
            WHERE price != ROUND(price, 4)
        """)

        assert len(precision_issues) == 0, \
            f"Found {len(precision_issues)} prices with >4 decimal places"

    def test_referential_integrity(self, source_db):
        """Verify no orphan records before migration"""
        orphan_orders = source_db.query("""
            SELECT o.id FROM orders o
            LEFT JOIN users u ON o.user_id = u.id
            WHERE u.id IS NULL
        """)
        assert len(orphan_orders) == 0, f"Found {len(orphan_orders)} orphan orders"

        orphan_items = source_db.query("""
            SELECT oi.id FROM order_items oi
            LEFT JOIN orders o ON oi.order_id = o.id
            WHERE o.id IS NULL
        """)
        assert len(orphan_items) == 0, f"Found {len(orphan_items)} orphan items"
```

**b) Post-Migration Verification Tests (4 điểm):**
```python
class TestPostMigrationVerification:
    """Verify data integrity after migration"""

    def test_row_count_matches(self, source_db, target_db):
        """Verify all rows migrated"""
        baseline = load_baseline_counts()

        tables = ['users', 'orders', 'order_items']
        for table in tables:
            source_count = baseline[table]
            target_count = target_db.query(f"SELECT COUNT(*) FROM {table}")[0]

            assert target_count == source_count, \
                f"{table}: Expected {source_count}, got {target_count}"

    def test_data_integrity_sampling(self, source_db, target_db):
        """Verify sample records match exactly"""
        # Sample 1000 random users
        sample_ids = source_db.query("""
            SELECT id FROM users ORDER BY RANDOM() LIMIT 1000
        """)

        for row in sample_ids:
            user_id = row['id']

            source_user = source_db.query(
                "SELECT * FROM users WHERE id = ?", (user_id,)
            )[0]
            target_user = target_db.query(
                "SELECT * FROM users WHERE id = %s", (user_id,)
            )[0]

            # Compare fields (accounting for transformations)
            assert source_user['email'] == target_user['email']
            assert source_user['name'] == target_user['name']
            # Phone was transformed
            assert normalize_phone(source_user['phone']) == target_user['phone_number']

    def test_aggregates_match(self, source_db, target_db):
        """Verify aggregate values match"""
        # Total order value
        source_total = source_db.query(
            "SELECT SUM(price * quantity) FROM order_items"
        )[0]
        target_total = target_db.query(
            "SELECT SUM(price * quantity) FROM order_items"
        )[0]

        # Allow small floating point difference
        assert abs(source_total - target_total) < 0.01, \
            f"Total mismatch: {source_total} vs {target_total}"

        # Order counts by status
        source_by_status = source_db.query("""
            SELECT status, COUNT(*) as cnt FROM orders GROUP BY status
        """)
        target_by_status = target_db.query("""
            SELECT os.name as status, COUNT(*) as cnt
            FROM orders o
            JOIN order_statuses os ON o.status_id = os.id
            GROUP BY os.name
        """)

        source_dict = {r['status']: r['cnt'] for r in source_by_status}
        target_dict = {r['status']: r['cnt'] for r in target_by_status}

        assert source_dict == target_dict, "Status counts mismatch"

    def test_new_constraints_enforced(self, target_db):
        """Verify new schema constraints work"""
        # Phone format validation
        with pytest.raises(Exception):
            target_db.execute("""
                INSERT INTO users (email, phone_number)
                VALUES ('test@test.com', 'invalid-phone-format')
            """)

        # Foreign key constraint
        with pytest.raises(Exception):
            target_db.execute("""
                INSERT INTO orders (user_id, status_id)
                VALUES (1, 999)  -- Invalid status_id
            """)

    def test_indexes_recreated(self, target_db):
        """Verify performance indexes exist"""
        indexes = target_db.query("""
            SELECT indexname FROM pg_indexes
            WHERE tablename = 'orders'
        """)
        index_names = [i['indexname'] for i in indexes]

        required_indexes = [
            'idx_orders_user_id',
            'idx_orders_status_id',
            'idx_orders_created_at',
        ]

        for idx in required_indexes:
            assert idx in index_names, f"Missing index: {idx}"
```

**c) Rollback Test Scenario (3 điểm):**
```python
class TestMigrationRollback:
    """Test rollback capability"""

    def test_rollback_restores_source(self, source_db, backup_db):
        """Verify rollback restores original state"""
        # Step 1: Take pre-migration snapshot
        pre_migration_counts = {
            'users': source_db.query("SELECT COUNT(*) FROM users")[0],
            'orders': source_db.query("SELECT COUNT(*) FROM orders")[0],
        }

        # Step 2: Simulate partial migration failure
        try:
            start_migration()

            # Migrate users successfully
            migrate_table('users')

            # Orders migration fails midway
            raise MigrationError("Simulated failure at 50% orders")

        except MigrationError:
            # Step 3: Execute rollback
            execute_rollback()

        # Step 4: Verify source is unchanged
        post_rollback_counts = {
            'users': source_db.query("SELECT COUNT(*) FROM users")[0],
            'orders': source_db.query("SELECT COUNT(*) FROM orders")[0],
        }

        assert pre_migration_counts == post_rollback_counts, \
            "Source data should be unchanged after rollback"

    def test_rollback_cleans_target(self, target_db):
        """Verify rollback removes partial data from target"""
        # After rollback, target should be empty or in pre-migration state
        target_users = target_db.query("SELECT COUNT(*) FROM users")[0]
        target_orders = target_db.query("SELECT COUNT(*) FROM orders")[0]

        assert target_users == 0, "Target users should be empty after rollback"
        assert target_orders == 0, "Target orders should be empty after rollback"

    def test_rollback_time_acceptable(self):
        """Verify rollback completes within SLA"""
        start = time.time()
        execute_rollback()
        rollback_time = time.time() - start

        # Rollback should complete within 30 minutes
        assert rollback_time < 1800, \
            f"Rollback took {rollback_time}s, exceeds 30 minute SLA"
```

---

## Câu 5: Index Performance Testing (10 điểm)

### Tình huống:
Query sau đang chạy chậm (5+ giây) trên bảng `orders` với 10 triệu records:
```sql
SELECT o.*, u.name, u.email
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending'
AND o.created_at > '2024-01-01'
AND o.total_amount > 100
ORDER BY o.created_at DESC
LIMIT 50
```

### Yêu cầu:
a) Phân tích EXPLAIN plan và đề xuất indexes (3 điểm)
b) Viết test verify index cải thiện performance (4 điểm)
c) Thiết kế test cho covering index vs regular index (3 điểm)

### Đáp án mẫu:

**a) EXPLAIN Analysis và Index Proposal (3 điểm):**
```sql
-- Original EXPLAIN output (problematic)
EXPLAIN ANALYZE
SELECT o.*, u.name, u.email
FROM orders o
JOIN users u ON o.user_id = u.id
WHERE o.status = 'pending'
AND o.created_at > '2024-01-01'
AND o.total_amount > 100
ORDER BY o.created_at DESC
LIMIT 50;

/*
QUERY PLAN:
Sort (cost=250000.00..250100.00 rows=50000 width=200) (actual time=5234.12..5234.15 rows=50 loops=1)
  -> Nested Loop (cost=0.00..249000.00 rows=50000)
    -> Seq Scan on orders o (cost=0.00..200000.00 rows=50000)   -- PROBLEM: Full table scan
         Filter: (status = 'pending' AND created_at > '2024-01-01' AND total_amount > 100)
         Rows Removed by Filter: 9950000
    -> Index Scan on users_pkey (cost=0.42..1.00 rows=1)
Planning Time: 0.5 ms
Execution Time: 5234.50 ms   -- TOO SLOW!
*/

-- PROPOSED INDEXES:

-- Index 1: Composite index for WHERE + ORDER BY
CREATE INDEX idx_orders_status_created ON orders(status, created_at DESC);

-- Index 2: Covering index (includes all needed columns)
CREATE INDEX idx_orders_covering ON orders(status, created_at DESC)
INCLUDE (user_id, total_amount);

-- Index 3: Partial index (only pending orders)
CREATE INDEX idx_orders_pending ON orders(created_at DESC, total_amount)
WHERE status = 'pending';
```

**b) Performance Improvement Test (4 điểm):**
```python
import time
import statistics

class TestIndexPerformance:
    """Verify index improves query performance"""

    SLOW_QUERY = """
        SELECT o.*, u.name, u.email
        FROM orders o
        JOIN users u ON o.user_id = u.id
        WHERE o.status = 'pending'
        AND o.created_at > '2024-01-01'
        AND o.total_amount > 100
        ORDER BY o.created_at DESC
        LIMIT 50
    """

    def setup_method(self):
        """Ensure 10M test records exist"""
        count = db.query("SELECT COUNT(*) FROM orders")[0]
        assert count >= 10_000_000, "Need 10M records for realistic test"

    def measure_query_time(self, iterations=5):
        """Run query multiple times and return stats"""
        times = []
        for _ in range(iterations):
            # Clear query cache
            db.execute("DISCARD ALL")  # PostgreSQL
            # db.execute("RESET QUERY CACHE")  # MySQL

            start = time.time()
            db.query(self.SLOW_QUERY)
            times.append(time.time() - start)

        return {
            'min': min(times),
            'max': max(times),
            'avg': statistics.mean(times),
            'median': statistics.median(times),
        }

    def test_baseline_without_index(self):
        """Establish baseline (slow) performance"""
        # Drop any existing indexes
        db.execute("DROP INDEX IF EXISTS idx_orders_status_created")
        db.execute("DROP INDEX IF EXISTS idx_orders_covering")

        stats = self.measure_query_time()

        # Baseline should be slow
        assert stats['avg'] > 3.0, \
            f"Baseline should be >3s, got {stats['avg']:.2f}s"

        # Save baseline for comparison
        self.baseline_avg = stats['avg']
        print(f"Baseline: {stats['avg']:.2f}s (min={stats['min']:.2f}, max={stats['max']:.2f})")

    def test_composite_index_improves_performance(self):
        """Verify composite index provides 90%+ improvement"""
        # Create composite index
        db.execute("""
            CREATE INDEX idx_orders_status_created
            ON orders(status, created_at DESC)
        """)

        stats = self.measure_query_time()

        # Should be at least 90% faster
        improvement = (self.baseline_avg - stats['avg']) / self.baseline_avg * 100

        assert improvement >= 90, \
            f"Expected 90%+ improvement, got {improvement:.1f}%"
        assert stats['avg'] < 0.5, \
            f"Query should be <0.5s with index, got {stats['avg']:.2f}s"

        print(f"With index: {stats['avg']:.2f}s ({improvement:.1f}% improvement)")

    def test_index_used_in_plan(self):
        """Verify query plan uses the index"""
        plan = db.query(f"EXPLAIN {self.SLOW_QUERY}")
        plan_text = '\n'.join([str(row) for row in plan])

        # Should use index scan, not seq scan
        assert "Index Scan" in plan_text or "Bitmap Index Scan" in plan_text, \
            "Query should use index scan"
        assert "Seq Scan on orders" not in plan_text, \
            "Query should NOT use sequential scan on orders"
        assert "idx_orders_status_created" in plan_text, \
            "Query should use our index"
```

**c) Covering Index vs Regular Index Test (3 điểm):**
```python
def test_covering_vs_regular_index(self):
    """Compare covering index vs regular index performance"""

    # Query that benefits from covering index
    SELECT_SPECIFIC = """
        SELECT status, created_at, total_amount, user_id
        FROM orders
        WHERE status = 'pending'
        AND created_at > '2024-01-01'
        ORDER BY created_at DESC
        LIMIT 100
    """

    # Test 1: Regular index (requires table lookup)
    db.execute("DROP INDEX IF EXISTS idx_orders_covering")
    db.execute("""
        CREATE INDEX idx_orders_regular
        ON orders(status, created_at DESC)
    """)

    regular_stats = self.measure_query_time_for(SELECT_SPECIFIC)

    # Test 2: Covering index (index-only scan)
    db.execute("DROP INDEX idx_orders_regular")
    db.execute("""
        CREATE INDEX idx_orders_covering
        ON orders(status, created_at DESC)
        INCLUDE (total_amount, user_id)
    """)

    covering_stats = self.measure_query_time_for(SELECT_SPECIFIC)

    # Covering index should be faster (no table lookup)
    assert covering_stats['avg'] < regular_stats['avg'], \
        "Covering index should be faster than regular index"

    # Verify index-only scan in plan
    plan = db.query(f"EXPLAIN {SELECT_SPECIFIC}")
    plan_text = '\n'.join([str(row) for row in plan])
    assert "Index Only Scan" in plan_text, \
        "Covering index should enable index-only scan"

    print(f"Regular: {regular_stats['avg']:.3f}s")
    print(f"Covering: {covering_stats['avg']:.3f}s")
    print(f"Improvement: {(1 - covering_stats['avg']/regular_stats['avg'])*100:.1f}%")
```

---

## Câu 6: Stored Procedure Testing (10 điểm)

### Tình huống:
Stored procedure `transfer_funds` chuyển tiền giữa 2 tài khoản:
```sql
CREATE PROCEDURE transfer_funds(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE from_balance DECIMAL(10,2);

    START TRANSACTION;

    SELECT balance INTO from_balance
    FROM accounts WHERE id = from_account FOR UPDATE;

    IF from_balance < amount THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient funds';
    END IF;

    UPDATE accounts SET balance = balance - amount WHERE id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE id = to_account;

    INSERT INTO transactions (from_acc, to_acc, amount, created_at)
    VALUES (from_account, to_account, amount, NOW());

    COMMIT;
END;
```

### Yêu cầu:
a) Thiết kế test cases cho happy path và edge cases (4 điểm)
b) Test concurrent transfers (race condition) (3 điểm)
c) Test error handling và rollback (3 điểm)

### Đáp án mẫu:

**a) Happy Path và Edge Cases (4 điểm):**
```python
class TestTransferFunds:
    """Test suite for transfer_funds stored procedure"""

    def setup_method(self):
        """Reset account balances before each test"""
        db.execute("UPDATE accounts SET balance = 1000 WHERE id = 1")
        db.execute("UPDATE accounts SET balance = 500 WHERE id = 2")
        db.execute("DELETE FROM transactions")

    # HAPPY PATH TESTS
    def test_successful_transfer(self):
        """Normal transfer between two accounts"""
        # Act
        db.call_procedure("transfer_funds", (1, 2, 100))

        # Assert
        acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
        acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

        assert acc1 == 900, "Sender should have 1000-100=900"
        assert acc2 == 600, "Receiver should have 500+100=600"

        # Verify transaction log
        txn = db.query("SELECT * FROM transactions ORDER BY id DESC LIMIT 1")[0]
        assert txn['from_acc'] == 1
        assert txn['to_acc'] == 2
        assert txn['amount'] == 100

    def test_transfer_exact_balance(self):
        """Transfer entire balance"""
        db.call_procedure("transfer_funds", (1, 2, 1000))

        acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
        acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

        assert acc1 == 0, "Sender should have 0 after full transfer"
        assert acc2 == 1500, "Receiver should have 500+1000=1500"

    def test_transfer_minimum_amount(self):
        """Transfer smallest possible amount (0.01)"""
        db.call_procedure("transfer_funds", (1, 2, 0.01))

        acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
        acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

        assert acc1 == 999.99
        assert acc2 == 500.01

    # EDGE CASES
    def test_transfer_to_same_account(self):
        """Transfer to same account should fail or be handled"""
        with pytest.raises(Exception):
            db.call_procedure("transfer_funds", (1, 1, 100))

    def test_transfer_zero_amount(self):
        """Zero amount transfer should be rejected"""
        with pytest.raises(Exception):
            db.call_procedure("transfer_funds", (1, 2, 0))

    def test_transfer_negative_amount(self):
        """Negative amount should be rejected"""
        with pytest.raises(Exception):
            db.call_procedure("transfer_funds", (1, 2, -100))

    def test_transfer_nonexistent_source(self):
        """Transfer from non-existent account"""
        with pytest.raises(Exception):
            db.call_procedure("transfer_funds", (999, 2, 100))

    def test_transfer_nonexistent_destination(self):
        """Transfer to non-existent account"""
        with pytest.raises(Exception):
            db.call_procedure("transfer_funds", (1, 999, 100))
```

**b) Concurrent Transfer Testing (3 điểm):**
```python
def test_concurrent_transfers_no_race_condition(self):
    """Two concurrent transfers from same account should not overdraw"""
    # Setup: Account 1 has exactly 1000
    db.execute("UPDATE accounts SET balance = 1000 WHERE id = 1")
    db.execute("UPDATE accounts SET balance = 0 WHERE id = 2")
    db.execute("UPDATE accounts SET balance = 0 WHERE id = 3")

    results = {'success': 0, 'failed': 0}

    def transfer_800_to_acc2():
        try:
            conn = get_new_connection()
            conn.call_procedure("transfer_funds", (1, 2, 800))
            results['success'] += 1
        except Exception as e:
            results['failed'] += 1

    def transfer_800_to_acc3():
        try:
            conn = get_new_connection()
            conn.call_procedure("transfer_funds", (1, 3, 800))
            results['success'] += 1
        except Exception as e:
            results['failed'] += 1

    # Launch concurrent transfers
    t1 = threading.Thread(target=transfer_800_to_acc2)
    t2 = threading.Thread(target=transfer_800_to_acc3)

    t1.start()
    t2.start()
    t1.join()
    t2.join()

    # Assertions
    assert results['success'] == 1, \
        f"Only 1 transfer should succeed, got {results['success']}"
    assert results['failed'] == 1, \
        f"1 transfer should fail (insufficient funds)"

    # Verify no overdraw
    acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
    assert acc1 >= 0, f"Account overdrew! Balance: {acc1}"

    # Total money should be conserved
    total = db.query("SELECT SUM(balance) FROM accounts")[0]
    assert total == 1000, f"Money was created/destroyed! Total: {total}"

def test_many_concurrent_transfers_stress(self):
    """Stress test with many concurrent transfers"""
    # Setup
    db.execute("UPDATE accounts SET balance = 10000 WHERE id = 1")

    success_count = 0
    fail_count = 0
    lock = threading.Lock()

    def small_transfer():
        nonlocal success_count, fail_count
        try:
            conn = get_new_connection()
            # Random small transfers
            conn.call_procedure("transfer_funds", (1, 2, 10))
            with lock:
                success_count += 1
        except:
            with lock:
                fail_count += 1

    # Launch 100 concurrent transfers of $10 each
    threads = [threading.Thread(target=small_transfer) for _ in range(100)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()

    # Verify consistency
    acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
    acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

    expected_acc1 = 10000 - (success_count * 10)
    assert acc1 == expected_acc1, \
        f"Account 1 inconsistent: expected {expected_acc1}, got {acc1}"
    assert acc2 == success_count * 10, \
        f"Account 2 inconsistent: expected {success_count * 10}, got {acc2}"

    print(f"Success: {success_count}, Failed: {fail_count}")
```

**c) Error Handling và Rollback (3 điểm):**
```python
def test_insufficient_funds_rollback(self):
    """Verify complete rollback on insufficient funds"""
    # Setup
    db.execute("UPDATE accounts SET balance = 100 WHERE id = 1")
    db.execute("UPDATE accounts SET balance = 500 WHERE id = 2")
    initial_txn_count = db.query("SELECT COUNT(*) FROM transactions")[0]

    # Attempt transfer more than available
    with pytest.raises(Exception) as exc_info:
        db.call_procedure("transfer_funds", (1, 2, 150))

    assert "Insufficient funds" in str(exc_info.value)

    # Verify NO changes were made
    acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
    acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]
    final_txn_count = db.query("SELECT COUNT(*) FROM transactions")[0]

    assert acc1 == 100, "Sender balance should be unchanged"
    assert acc2 == 500, "Receiver balance should be unchanged"
    assert final_txn_count == initial_txn_count, "No transaction should be logged"

def test_partial_failure_full_rollback(self):
    """If any step fails, entire transaction rolls back"""
    # Setup with trigger that fails on certain condition
    db.execute("""
        CREATE TRIGGER fail_on_large_transfer
        BEFORE INSERT ON transactions
        FOR EACH ROW
        BEGIN
            IF NEW.amount > 5000 THEN
                SIGNAL SQLSTATE '45000'
                SET MESSAGE_TEXT = 'Transfer too large';
            END IF;
        END
    """)

    initial_acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
    initial_acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

    # This will fail at INSERT INTO transactions (trigger)
    with pytest.raises(Exception) as exc_info:
        db.call_procedure("transfer_funds", (1, 2, 6000))

    # Verify COMPLETE rollback (UPDATE accounts should also rollback)
    final_acc1 = db.query("SELECT balance FROM accounts WHERE id = 1")[0]
    final_acc2 = db.query("SELECT balance FROM accounts WHERE id = 2")[0]

    assert final_acc1 == initial_acc1, \
        "Sender should be unchanged after trigger failure"
    assert final_acc2 == initial_acc2, \
        "Receiver should be unchanged after trigger failure"

    # Cleanup
    db.execute("DROP TRIGGER fail_on_large_transfer")
```

---

## Câu 7: Schema Validation Testing (10 điểm)

### Tình huống:
Bạn cần verify database schema sau khi deploy mới match với specification:
- Table `products`: id, name, description, price, stock, category_id, created_at
- Constraints: price >= 0, stock >= 0, category_id FK to categories
- Indexes: idx_products_category, idx_products_price

### Yêu cầu:
a) Viết test verify table và column schema (4 điểm)
b) Viết test verify constraints hoạt động đúng (3 điểm)
c) Viết test verify indexes tồn tại và được sử dụng (3 điểm)

### Đáp án mẫu:

**a) Table và Column Schema Verification (4 điểm):**
```python
class TestProductsSchema:
    """Verify products table schema matches specification"""

    EXPECTED_SCHEMA = {
        'id': {
            'type': 'integer',
            'nullable': False,
            'primary_key': True,
            'auto_increment': True
        },
        'name': {
            'type': 'varchar',
            'max_length': 255,
            'nullable': False
        },
        'description': {
            'type': 'text',
            'nullable': True
        },
        'price': {
            'type': 'decimal',
            'precision': 10,
            'scale': 2,
            'nullable': False,
            'default': '0.00'
        },
        'stock': {
            'type': 'integer',
            'nullable': False,
            'default': 0
        },
        'category_id': {
            'type': 'integer',
            'nullable': False,
            'foreign_key': ('categories', 'id')
        },
        'created_at': {
            'type': 'timestamp',
            'nullable': False,
            'default': 'CURRENT_TIMESTAMP'
        }
    }

    def test_table_exists(self):
        """Verify products table exists"""
        result = db.query("""
            SELECT COUNT(*) FROM information_schema.tables
            WHERE table_schema = DATABASE() AND table_name = 'products'
        """)[0]
        assert result == 1, "products table should exist"

    def test_all_columns_exist(self):
        """Verify all required columns exist"""
        columns = db.query("""
            SELECT column_name FROM information_schema.columns
            WHERE table_name = 'products'
        """)
        actual_columns = {c['column_name'] for c in columns}
        expected_columns = set(self.EXPECTED_SCHEMA.keys())

        missing = expected_columns - actual_columns
        extra = actual_columns - expected_columns

        assert len(missing) == 0, f"Missing columns: {missing}"
        # Extra columns may be OK (e.g., updated_at) but log warning
        if extra:
            print(f"Warning: Extra columns found: {extra}")

    def test_column_types(self):
        """Verify column data types match specification"""
        columns = db.query("""
            SELECT column_name, data_type, character_maximum_length,
                   numeric_precision, numeric_scale, is_nullable,
                   column_default, column_key
            FROM information_schema.columns
            WHERE table_name = 'products'
        """)

        for col in columns:
            col_name = col['column_name']
            if col_name not in self.EXPECTED_SCHEMA:
                continue

            expected = self.EXPECTED_SCHEMA[col_name]

            # Check type
            assert expected['type'] in col['data_type'].lower(), \
                f"{col_name}: expected type {expected['type']}, got {col['data_type']}"

            # Check nullable
            actual_nullable = col['is_nullable'] == 'YES'
            assert actual_nullable == expected.get('nullable', True), \
                f"{col_name}: nullable mismatch"

            # Check max_length for varchar
            if 'max_length' in expected:
                assert col['character_maximum_length'] == expected['max_length'], \
                    f"{col_name}: max_length mismatch"

            # Check precision/scale for decimal
            if expected['type'] == 'decimal':
                assert col['numeric_precision'] == expected['precision'], \
                    f"{col_name}: precision mismatch"
                assert col['numeric_scale'] == expected['scale'], \
                    f"{col_name}: scale mismatch"

    def test_primary_key(self):
        """Verify primary key is correctly defined"""
        pk = db.query("""
            SELECT column_name FROM information_schema.key_column_usage
            WHERE table_name = 'products' AND constraint_name = 'PRIMARY'
        """)

        assert len(pk) == 1, "Should have exactly 1 primary key column"
        assert pk[0]['column_name'] == 'id', "Primary key should be 'id'"
```

**b) Constraint Verification (3 điểm):**
```python
def test_price_check_constraint(self):
    """Price must be >= 0"""
    # Valid: price = 0
    db.execute("""
        INSERT INTO products (name, price, stock, category_id)
        VALUES ('Test', 0, 10, 1)
    """)

    # Valid: price > 0
    db.execute("""
        INSERT INTO products (name, price, stock, category_id)
        VALUES ('Test2', 99.99, 10, 1)
    """)

    # Invalid: price < 0
    with pytest.raises(Exception) as exc_info:
        db.execute("""
            INSERT INTO products (name, price, stock, category_id)
            VALUES ('Test3', -1, 10, 1)
        """)

    # Should fail due to CHECK constraint
    assert "check" in str(exc_info.value).lower() or \
           "constraint" in str(exc_info.value).lower()

def test_stock_check_constraint(self):
    """Stock must be >= 0"""
    # Invalid: negative stock
    with pytest.raises(Exception):
        db.execute("""
            INSERT INTO products (name, price, stock, category_id)
            VALUES ('Test', 10, -5, 1)
        """)

def test_foreign_key_constraint(self):
    """category_id must reference existing category"""
    # Setup: ensure category 1 exists
    db.execute("INSERT IGNORE INTO categories (id, name) VALUES (1, 'Electronics')")

    # Valid: existing category
    db.execute("""
        INSERT INTO products (name, price, stock, category_id)
        VALUES ('Valid Product', 100, 10, 1)
    """)

    # Invalid: non-existent category
    with pytest.raises(Exception) as exc_info:
        db.execute("""
            INSERT INTO products (name, price, stock, category_id)
            VALUES ('Invalid Product', 100, 10, 9999)
        """)

    assert "foreign key" in str(exc_info.value).lower() or \
           "constraint" in str(exc_info.value).lower()

def test_not_null_constraints(self):
    """Required fields cannot be NULL"""
    required_fields = ['name', 'price', 'stock', 'category_id']

    for field in required_fields:
        with pytest.raises(Exception):
            # Attempt insert with NULL value
            if field == 'name':
                db.execute("""
                    INSERT INTO products (name, price, stock, category_id)
                    VALUES (NULL, 10, 5, 1)
                """)
            # ... similar for other fields
```

**c) Index Verification (3 điểm):**
```python
def test_required_indexes_exist(self):
    """Verify all required indexes are created"""
    indexes = db.query("""
        SELECT index_name, column_name
        FROM information_schema.statistics
        WHERE table_name = 'products'
        ORDER BY index_name, seq_in_index
    """)

    index_names = {idx['index_name'] for idx in indexes}

    required = ['idx_products_category', 'idx_products_price', 'PRIMARY']
    for req in required:
        assert req in index_names, f"Missing index: {req}"

def test_index_used_for_category_query(self):
    """Verify category index is used when filtering by category"""
    plan = db.query("""
        EXPLAIN SELECT * FROM products WHERE category_id = 1
    """)
    plan_text = str(plan)

    assert 'idx_products_category' in plan_text, \
        "Query should use category index"
    assert 'ALL' not in plan_text, \
        "Query should not do full table scan"

def test_index_used_for_price_range_query(self):
    """Verify price index is used for price range queries"""
    plan = db.query("""
        EXPLAIN SELECT * FROM products
        WHERE price BETWEEN 100 AND 500
        ORDER BY price
    """)
    plan_text = str(plan)

    assert 'idx_products_price' in plan_text, \
        "Query should use price index"

def test_composite_index_column_order(self):
    """Verify composite index has correct column order"""
    index_columns = db.query("""
        SELECT column_name, seq_in_index
        FROM information_schema.statistics
        WHERE table_name = 'products' AND index_name = 'idx_products_category'
        ORDER BY seq_in_index
    """)

    # For (category_id, name) composite index
    assert index_columns[0]['column_name'] == 'category_id', \
        "First column should be category_id"
```

---

## Câu 8: Trigger Testing (10 điểm)

### Tình huống:
Bạn có trigger tự động update `products.stock` khi có order:
```sql
CREATE TRIGGER update_stock_after_order
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products
    SET stock = stock - NEW.quantity
    WHERE id = NEW.product_id;

    IF (SELECT stock FROM products WHERE id = NEW.product_id) < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Insufficient stock';
    END IF;
END;
```

### Yêu cầu:
a) Thiết kế test cases cho trigger behavior (4 điểm)
b) Test trigger với concurrent inserts (3 điểm)
c) Test trigger rollback behavior (3 điểm)

### Đáp án mẫu:

**a) Trigger Behavior Tests (4 điểm):**
```python
class TestStockTrigger:
    """Test suite for stock update trigger"""

    def setup_method(self):
        """Reset test data"""
        db.execute("DELETE FROM order_items WHERE order_id >= 1000")
        db.execute("DELETE FROM orders WHERE id >= 1000")
        db.execute("UPDATE products SET stock = 100 WHERE id = 1")

    def test_single_order_decreases_stock(self):
        """Inserting order_item should decrease product stock"""
        initial_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]

        # Create order
        db.execute("INSERT INTO orders (id, user_id) VALUES (1000, 1)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1000, 1, 5)")

        final_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]

        assert final_stock == initial_stock - 5, \
            f"Stock should decrease by 5: {initial_stock} -> {final_stock}"

    def test_multiple_items_same_product(self):
        """Multiple order_items for same product"""
        db.execute("UPDATE products SET stock = 100 WHERE id = 1")

        db.execute("INSERT INTO orders (id, user_id) VALUES (1001, 1)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1001, 1, 10)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1001, 1, 15)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1001, 1, 25)")

        final_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]

        assert final_stock == 100 - 10 - 15 - 25, \
            f"Stock should be 50, got {final_stock}"

    def test_exact_stock_order(self):
        """Order exactly equal to available stock"""
        db.execute("UPDATE products SET stock = 10 WHERE id = 1")

        db.execute("INSERT INTO orders (id, user_id) VALUES (1002, 1)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1002, 1, 10)")

        final_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]
        assert final_stock == 0, "Stock should be exactly 0"

    def test_insufficient_stock_fails(self):
        """Order exceeding stock should fail"""
        db.execute("UPDATE products SET stock = 5 WHERE id = 1")

        db.execute("INSERT INTO orders (id, user_id) VALUES (1003, 1)")

        with pytest.raises(Exception) as exc_info:
            db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (1003, 1, 10)")

        assert "Insufficient stock" in str(exc_info.value)

        # Stock should remain unchanged
        stock = db.query("SELECT stock FROM products WHERE id = 1")[0]
        assert stock == 5, "Stock should be unchanged after failed order"
```

**b) Concurrent Insert Testing (3 điểm):**
```python
def test_concurrent_orders_stock_consistency(self):
    """Multiple concurrent orders should not cause overselling"""
    db.execute("UPDATE products SET stock = 10 WHERE id = 1")

    results = {'success': 0, 'failed': 0}

    def place_order(order_id, quantity):
        try:
            conn = get_new_connection()
            conn.execute(f"INSERT INTO orders (id, user_id) VALUES ({order_id}, 1)")
            conn.execute(f"""
                INSERT INTO order_items (order_id, product_id, quantity)
                VALUES ({order_id}, 1, {quantity})
            """)
            conn.commit()
            results['success'] += 1
        except Exception:
            conn.rollback()
            results['failed'] += 1

    # 5 concurrent orders, each for 3 items (total 15, but only 10 in stock)
    threads = []
    for i in range(5):
        t = threading.Thread(target=place_order, args=(2000 + i, 3))
        threads.append(t)

    for t in threads:
        t.start()
    for t in threads:
        t.join()

    # At most 3 orders should succeed (3*3=9 <= 10)
    assert results['success'] <= 3, \
        f"Too many orders succeeded: {results['success']}"

    # Stock should not be negative
    final_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]
    assert final_stock >= 0, f"Stock went negative: {final_stock}"

    # Verify: stock + sold = original
    total_sold = db.query("""
        SELECT COALESCE(SUM(quantity), 0) FROM order_items
        WHERE product_id = 1 AND order_id >= 2000
    """)[0]

    assert final_stock + total_sold == 10, \
        f"Stock inconsistency: {final_stock} + {total_sold} != 10"
```

**c) Trigger Rollback Behavior (3 điểm):**
```python
def test_trigger_rollback_on_transaction_failure(self):
    """Trigger changes should rollback if transaction fails"""
    db.execute("UPDATE products SET stock = 100 WHERE id = 1")

    try:
        db.begin_transaction()

        # This should trigger stock update
        db.execute("INSERT INTO orders (id, user_id) VALUES (3000, 1)")
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (3000, 1, 10)")

        # Verify trigger executed
        mid_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]
        assert mid_stock == 90, "Trigger should have updated stock"

        # Force transaction failure
        raise Exception("Simulated failure")

        db.commit()
    except:
        db.rollback()

    # Trigger changes should also rollback
    final_stock = db.query("SELECT stock FROM products WHERE id = 1")[0]
    assert final_stock == 100, \
        f"Stock should rollback to 100, got {final_stock}"

def test_trigger_partial_batch_rollback(self):
    """If one item in batch fails, all should rollback"""
    db.execute("UPDATE products SET stock = 5 WHERE id = 1")
    db.execute("UPDATE products SET stock = 100 WHERE id = 2")

    try:
        db.begin_transaction()

        db.execute("INSERT INTO orders (id, user_id) VALUES (3001, 1)")

        # First item: OK (stock 5 - 3 = 2)
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (3001, 1, 3)")

        # Second item: FAIL (stock 2 - 10 = -8)
        db.execute("INSERT INTO order_items (order_id, product_id, quantity) VALUES (3001, 1, 10)")

        db.commit()
    except:
        db.rollback()

    # Both stock updates should rollback
    stock1 = db.query("SELECT stock FROM products WHERE id = 1")[0]
    assert stock1 == 5, f"Product 1 stock should rollback to 5, got {stock1}"
```

---

## Câu 9: Database Backup/Restore Testing (10 điểm)

### Tình huống:
Bạn cần test backup/restore procedure cho production database:
- Full backup hàng ngày
- Incremental backup mỗi giờ
- Point-in-time recovery requirement

### Yêu cầu:
a) Thiết kế test verify backup integrity (3 điểm)
b) Test restore procedure hoạt động đúng (4 điểm)
c) Test point-in-time recovery (3 điểm)

### Đáp án mẫu:

**a) Backup Integrity Verification (3 điểm):**
```python
class TestBackupIntegrity:
    """Verify backup files are valid and complete"""

    def test_full_backup_created(self):
        """Verify full backup file is created"""
        # Trigger backup
        run_backup("full")

        backup_file = get_latest_backup("full")

        assert backup_file.exists(), "Backup file should exist"
        assert backup_file.size() > 0, "Backup file should not be empty"

        # Verify checksum
        expected_checksum = calculate_checksum(backup_file)
        stored_checksum = get_backup_metadata(backup_file)['checksum']

        assert expected_checksum == stored_checksum, \
            "Backup checksum mismatch - file may be corrupted"

    def test_backup_contains_all_tables(self):
        """Verify backup contains all required tables"""
        backup_file = get_latest_backup("full")

        # Extract table list from backup
        tables_in_backup = extract_table_list(backup_file)

        # Get actual tables from database
        actual_tables = db.query("""
            SELECT table_name FROM information_schema.tables
            WHERE table_schema = DATABASE()
        """)
        actual_table_names = {t['table_name'] for t in actual_tables}

        # All tables should be in backup
        missing = actual_table_names - tables_in_backup
        assert len(missing) == 0, f"Tables missing from backup: {missing}"

    def test_backup_row_counts_match(self):
        """Verify backup contains correct number of rows"""
        # Record counts before backup
        counts_before = {}
        for table in ['users', 'orders', 'products']:
            counts_before[table] = db.query(f"SELECT COUNT(*) FROM {table}")[0]

        # Create backup
        run_backup("full")
        backup_file = get_latest_backup("full")

        # Verify counts in backup metadata
        backup_meta = get_backup_metadata(backup_file)

        for table, count in counts_before.items():
            assert backup_meta['row_counts'][table] == count, \
                f"{table}: expected {count}, got {backup_meta['row_counts'][table]}"

    def test_incremental_backup_captures_changes(self):
        """Verify incremental backup captures recent changes"""
        # Get baseline
        last_full = get_latest_backup("full")

        # Make changes
        db.execute("INSERT INTO users (email) VALUES ('new@test.com')")
        new_user_id = db.query("SELECT LAST_INSERT_ID()")[0]

        # Create incremental backup
        run_backup("incremental")
        incr_backup = get_latest_backup("incremental")

        # Verify new data is in incremental
        changes = extract_changes(incr_backup)

        assert any(c['table'] == 'users' and c['id'] == new_user_id
                   for c in changes), \
            "Incremental backup should contain new user"
```

**b) Restore Procedure Testing (4 điểm):**
```python
class TestRestoreProcedure:
    """Test database restore from backup"""

    def setup_method(self):
        """Create test database for restore"""
        self.test_db = create_test_database("restore_test")

    def teardown_method(self):
        """Cleanup test database"""
        drop_database(self.test_db)

    def test_full_restore_success(self):
        """Restore from full backup should recreate database"""
        # Create backup of production
        backup_file = create_backup(production_db, "full")

        # Restore to test database
        restore_result = restore_backup(backup_file, self.test_db)

        assert restore_result.success, \
            f"Restore failed: {restore_result.error}"

        # Verify table structure matches
        prod_tables = get_table_list(production_db)
        restored_tables = get_table_list(self.test_db)

        assert prod_tables == restored_tables, \
            "Restored tables don't match production"

    def test_data_integrity_after_restore(self):
        """Verify data integrity after restore"""
        # Sample data from production
        prod_users = production_db.query("""
            SELECT id, email, name FROM users
            ORDER BY id LIMIT 100
        """)

        # Restore and verify
        backup_file = get_latest_backup("full")
        restore_backup(backup_file, self.test_db)

        restored_users = self.test_db.query("""
            SELECT id, email, name FROM users
            ORDER BY id LIMIT 100
        """)

        assert prod_users == restored_users, \
            "Restored user data doesn't match production"

    def test_restore_with_incremental(self):
        """Restore full + incremental backups"""
        # Get latest full backup
        full_backup = get_latest_backup("full")
        full_backup_time = get_backup_time(full_backup)

        # Get all incrementals since full backup
        incrementals = get_incremental_backups_since(full_backup_time)

        # Restore in order
        restore_backup(full_backup, self.test_db)
        for incr in incrementals:
            apply_incremental(incr, self.test_db)

        # Verify matches current production
        prod_count = production_db.query("SELECT COUNT(*) FROM orders")[0]
        restored_count = self.test_db.query("SELECT COUNT(*) FROM orders")[0]

        assert restored_count == prod_count, \
            f"Order count mismatch: {restored_count} vs {prod_count}"

    def test_restore_time_acceptable(self):
        """Restore should complete within SLA"""
        backup_file = get_latest_backup("full")

        start = time.time()
        restore_backup(backup_file, self.test_db)
        restore_time = time.time() - start

        # 10GB database should restore in < 30 minutes
        assert restore_time < 1800, \
            f"Restore took {restore_time}s, exceeds 30 minute SLA"
```

**c) Point-in-Time Recovery Testing (3 điểm):**
```python
class TestPointInTimeRecovery:
    """Test ability to recover to specific point in time"""

    def test_pitr_before_incident(self):
        """Recover to state just before data loss incident"""
        # Timeline:
        # T1: Good state
        # T2: Accidental DELETE
        # T3: Detected incident
        # Goal: Recover to T1

        # Record timestamp T1
        t1 = datetime.now()

        # Record data at T1
        users_at_t1 = db.query("SELECT COUNT(*) FROM users")[0]

        # Simulate incident at T2
        time.sleep(1)
        db.execute("DELETE FROM users WHERE id > 1000")  # Accidental delete

        users_after_delete = db.query("SELECT COUNT(*) FROM users")[0]
        assert users_after_delete < users_at_t1, "Verify delete happened"

        # Recovery to T1
        recovered_db = point_in_time_recovery(
            target_time=t1,
            target_db="pitr_test"
        )

        # Verify recovery
        users_recovered = recovered_db.query("SELECT COUNT(*) FROM users")[0]
        assert users_recovered == users_at_t1, \
            f"Expected {users_at_t1} users, got {users_recovered}"

    def test_pitr_specific_transaction(self):
        """Recover to just before specific transaction"""
        # Insert known record with timestamp
        db.execute("""
            INSERT INTO audit_log (action, timestamp)
            VALUES ('MARKER', NOW())
        """)

        marker_time = db.query("""
            SELECT timestamp FROM audit_log
            WHERE action = 'MARKER'
            ORDER BY id DESC LIMIT 1
        """)[0]['timestamp']

        # Make changes after marker
        db.execute("UPDATE products SET price = price * 2")

        # Recover to just before marker
        recovered_db = point_in_time_recovery(
            target_time=marker_time - timedelta(seconds=1),
            target_db="pitr_test"
        )

        # Verify prices are original (not doubled)
        original_price = recovered_db.query(
            "SELECT price FROM products WHERE id = 1"
        )[0]
        current_price = db.query(
            "SELECT price FROM products WHERE id = 1"
        )[0]

        assert current_price == original_price * 2, \
            "Current price should be doubled"
        # Recovery should have original price (not doubled)
```

---

## Câu 10: Performance Testing cho Database (10 điểm)

### Tình huống:
Hệ thống e-commerce cần handle:
- 1000 concurrent users
- 100 orders/second peak
- Query response < 100ms (P95)

### Yêu cầu:
a) Thiết kế connection pool stress test (3 điểm)
b) Test query performance under load (4 điểm)
c) Test deadlock detection và recovery (3 điểm)

### Đáp án mẫu:

**a) Connection Pool Stress Test (3 điểm):**
```python
import threading
import time
from concurrent.futures import ThreadPoolExecutor

class TestConnectionPool:
    """Test database connection pool under stress"""

    def test_max_connections_handling(self):
        """Verify pool handles max connections gracefully"""
        pool_size = 100
        concurrent_requests = 200

        results = {'success': 0, 'failed': 0, 'queue_time': []}

        def execute_query():
            start = time.time()
            try:
                conn = pool.get_connection(timeout=30)
                queue_time = time.time() - start
                results['queue_time'].append(queue_time)

                conn.query("SELECT 1")
                time.sleep(0.1)  # Simulate work

                pool.release_connection(conn)
                results['success'] += 1
            except TimeoutError:
                results['failed'] += 1

        # Launch concurrent requests
        with ThreadPoolExecutor(max_workers=concurrent_requests) as executor:
            futures = [executor.submit(execute_query) for _ in range(concurrent_requests)]
            for f in futures:
                f.result()

        # Assertions
        assert results['failed'] == 0, \
            f"{results['failed']} requests failed to get connection"

        avg_queue_time = sum(results['queue_time']) / len(results['queue_time'])
        p95_queue_time = sorted(results['queue_time'])[int(len(results['queue_time']) * 0.95)]

        assert p95_queue_time < 5.0, \
            f"P95 queue time {p95_queue_time}s exceeds 5s limit"

        print(f"Avg queue time: {avg_queue_time:.3f}s")
        print(f"P95 queue time: {p95_queue_time:.3f}s")

    def test_connection_leak_detection(self):
        """Verify no connection leaks under load"""
        initial_connections = pool.active_connections()

        # Execute many queries without explicit release
        for _ in range(1000):
            with pool.get_connection() as conn:  # Context manager auto-releases
                conn.query("SELECT 1")

        final_connections = pool.active_connections()

        assert final_connections <= initial_connections + 5, \
            f"Possible connection leak: {initial_connections} -> {final_connections}"
```

**b) Query Performance Under Load (4 điểm):**
```python
class TestQueryPerformance:
    """Test query performance under concurrent load"""

    CRITICAL_QUERIES = [
        ("Get user orders", """
            SELECT o.*, u.email
            FROM orders o
            JOIN users u ON o.user_id = u.id
            WHERE u.id = ?
            ORDER BY o.created_at DESC
            LIMIT 20
        """),
        ("Search products", """
            SELECT * FROM products
            WHERE name LIKE ? AND category_id = ?
            AND price BETWEEN ? AND ?
            LIMIT 50
        """),
        ("Order summary", """
            SELECT DATE(created_at), COUNT(*), SUM(total)
            FROM orders
            WHERE created_at > DATE_SUB(NOW(), INTERVAL 30 DAY)
            GROUP BY DATE(created_at)
        """),
    ]

    def test_query_performance_under_load(self):
        """All critical queries should meet SLA under load"""
        concurrent_users = 100
        queries_per_user = 50

        latencies = {name: [] for name, _ in self.CRITICAL_QUERIES}

        def user_simulation():
            for _ in range(queries_per_user):
                name, query = random.choice(self.CRITICAL_QUERIES)

                start = time.time()
                try:
                    if "?" in query:
                        # Execute with sample params
                        db.query(query, sample_params_for(name))
                    else:
                        db.query(query)

                    latency = (time.time() - start) * 1000  # ms
                    latencies[name].append(latency)
                except Exception as e:
                    latencies[name].append(float('inf'))

        # Run concurrent users
        threads = [threading.Thread(target=user_simulation)
                   for _ in range(concurrent_users)]

        for t in threads:
            t.start()
        for t in threads:
            t.join()

        # Analyze results
        for name, times in latencies.items():
            valid_times = [t for t in times if t != float('inf')]

            if not valid_times:
                pytest.fail(f"All queries failed for: {name}")

            p50 = sorted(valid_times)[len(valid_times) // 2]
            p95 = sorted(valid_times)[int(len(valid_times) * 0.95)]
            p99 = sorted(valid_times)[int(len(valid_times) * 0.99)]

            print(f"{name}: P50={p50:.1f}ms, P95={p95:.1f}ms, P99={p99:.1f}ms")

            assert p95 < 100, \
                f"{name} P95 latency {p95:.1f}ms exceeds 100ms SLA"

    def test_throughput_meets_requirement(self):
        """System should handle 100 orders/second"""
        target_ops = 100
        test_duration = 10  # seconds

        orders_created = 0
        errors = 0

        start = time.time()

        while time.time() - start < test_duration:
            try:
                create_order(random_order_data())
                orders_created += 1
            except:
                errors += 1

        actual_duration = time.time() - start
        throughput = orders_created / actual_duration

        print(f"Throughput: {throughput:.1f} orders/sec")
        print(f"Errors: {errors}")

        assert throughput >= target_ops, \
            f"Throughput {throughput:.1f} below target {target_ops}"
        assert errors / orders_created < 0.01, \
            f"Error rate {errors/orders_created:.2%} exceeds 1%"
```

**c) Deadlock Detection và Recovery (3 điểm):**
```python
class TestDeadlockHandling:
    """Test database deadlock detection and recovery"""

    def test_deadlock_detection(self):
        """Verify deadlock is detected and one transaction aborted"""
        # Setup: Two accounts
        db.execute("UPDATE accounts SET balance = 1000 WHERE id IN (1, 2)")

        deadlock_occurred = threading.Event()
        results = {'t1': None, 't2': None}

        def transaction_1():
            try:
                conn = get_connection()
                conn.begin()

                # Lock account 1
                conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
                time.sleep(0.5)  # Wait for T2 to lock account 2

                # Try to lock account 2 (T2 holds this)
                conn.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
                conn.commit()
                results['t1'] = 'success'
            except Exception as e:
                if 'deadlock' in str(e).lower():
                    deadlock_occurred.set()
                    results['t1'] = 'deadlock'
                else:
                    results['t1'] = f'error: {e}'
                conn.rollback()

        def transaction_2():
            try:
                conn = get_connection()
                conn.begin()

                # Lock account 2
                conn.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 2")
                time.sleep(0.5)  # Wait for T1 to lock account 1

                # Try to lock account 1 (T1 holds this) -> DEADLOCK
                conn.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 1")
                conn.commit()
                results['t2'] = 'success'
            except Exception as e:
                if 'deadlock' in str(e).lower():
                    deadlock_occurred.set()
                    results['t2'] = 'deadlock'
                else:
                    results['t2'] = f'error: {e}'
                conn.rollback()

        # Run concurrent transactions
        t1 = threading.Thread(target=transaction_1)
        t2 = threading.Thread(target=transaction_2)
        t1.start(); t2.start()
        t1.join(); t2.join()

        # One should succeed, one should be victim
        assert deadlock_occurred.is_set(), "Deadlock should be detected"

        statuses = [results['t1'], results['t2']]
        assert 'success' in statuses, "One transaction should succeed"
        assert 'deadlock' in statuses, "One transaction should be deadlock victim"

        print(f"T1: {results['t1']}, T2: {results['t2']}")

    def test_deadlock_retry_succeeds(self):
        """Application should retry after deadlock and eventually succeed"""
        max_retries = 3

        def transfer_with_retry(from_acc, to_acc, amount):
            for attempt in range(max_retries):
                try:
                    conn = get_connection()
                    conn.begin()
                    conn.execute(
                        "UPDATE accounts SET balance = balance - ? WHERE id = ?",
                        (amount, from_acc)
                    )
                    conn.execute(
                        "UPDATE accounts SET balance = balance + ? WHERE id = ?",
                        (amount, to_acc)
                    )
                    conn.commit()
                    return True
                except DeadlockError:
                    conn.rollback()
                    time.sleep(0.1 * (attempt + 1))  # Exponential backoff
                    continue
            return False

        # Run transfers that might deadlock
        results = []
        threads = []

        for i in range(10):
            from_acc = 1 if i % 2 == 0 else 2
            to_acc = 2 if i % 2 == 0 else 1
            t = threading.Thread(
                target=lambda: results.append(transfer_with_retry(from_acc, to_acc, 10))
            )
            threads.append(t)

        for t in threads:
            t.start()
        for t in threads:
            t.join()

        # All should eventually succeed
        assert all(results), f"Some transfers failed: {results}"
```

---

## Tổng kết

| Câu | Chủ đề | Điểm |
|-----|--------|------|
| 1 | ACID Properties Testing | 10 |
| 2 | DBUnit Test Data Management | 10 |
| 3 | SQL Injection Testing | 10 |
| 4 | Data Migration Testing | 10 |
| 5 | Index Performance Testing | 10 |
| 6 | Stored Procedure Testing | 10 |
| 7 | Schema Validation Testing | 10 |
| 8 | Trigger Testing | 10 |
| 9 | Backup/Restore Testing | 10 |
| 10 | Performance Testing | 10 |
| **Tổng** | | **100** |

---

*Thời gian ôn tập đề xuất: 4-5 giờ*
*Thực hành: Setup DBUnit với Spring Boot project, viết SQL injection test suite*
