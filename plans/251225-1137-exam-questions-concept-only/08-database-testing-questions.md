# Database Testing - Concept Questions

## Question 1: ACID Properties Testing (15 points)

### Scenario
ToolShop's order processing system must maintain data integrity. When a customer places an order:
1. Inventory count decreases
2. Order record created
3. Payment record created
4. Customer points updated

### Questions

**a) (5 points)** Explain each ACID property and why it matters for ToolShop.

**Expected Answer:**

| Property | Definition | ToolShop Application |
|----------|------------|---------------------|
| **Atomicity** | All operations complete or none do | If payment fails, inventory must not decrease |
| **Consistency** | Database moves from one valid state to another | Total inventory + sold items = original stock |
| **Isolation** | Concurrent transactions don't interfere | Two customers buying last item handled correctly |
| **Durability** | Committed transactions survive crashes | Confirmed order exists after server restart |

**Why each matters:**

```
Atomicity violation example:
  1. ✅ Inventory decreased (-1)
  2. ✅ Order created
  3. ❌ Payment failed (external service)
  4. ❌ Points not updated

  Result without atomicity: Inventory lost, no payment received
  Result with atomicity: All changes rolled back, customer can retry
```

**b) (5 points)** How would you test Atomicity for the order processing flow?

**Expected Answer:**

**Test scenarios:**

| Scenario | Setup | Action | Expected |
|----------|-------|--------|----------|
| **Complete success** | Valid order data | Process order | All 4 operations succeed |
| **Payment failure** | Invalid payment | Process order | All changes rolled back |
| **Inventory shortage** | Stock = 0 | Process order | Transaction aborted, no partial updates |
| **Database crash mid-transaction** | Kill DB during write | Restart DB | Either fully committed or fully rolled back |

**Verification approach:**
```
Before transaction:
  - Record inventory count: 10
  - Record order count: 50
  - Record payment count: 100

Execute transaction that will fail at step 3

After transaction:
  - Verify inventory count: 10 (unchanged)
  - Verify order count: 50 (unchanged)
  - Verify payment count: 100 (unchanged)
  - All or nothing!
```

**c) (5 points)** How would you test Isolation with concurrent transactions?

**Expected Answer:**

**Concurrency test scenarios:**

| Scenario | Description | Expected Behavior |
|----------|-------------|-------------------|
| **Lost update** | Two users update same inventory | Both updates reflected correctly |
| **Dirty read** | Read uncommitted data | Only committed data visible |
| **Non-repeatable read** | Value changes between reads | Consistent within transaction |
| **Phantom read** | New rows appear in query | Query results stable within transaction |

**Test design for "last item" scenario:**
```
Setup: Product X has inventory = 1

Thread 1:                    Thread 2:
─────────────────────────────────────────────
Start transaction            Start transaction
Check inventory (1)          Check inventory (1)
Reserve item                 Reserve item
                            ← One must fail!
Commit                       Commit (or rollback)

Expected result:
  - Only ONE order created
  - Inventory = 0
  - No overselling
```

**Isolation level testing:**

| Isolation Level | Test For |
|-----------------|----------|
| **Read Uncommitted** | Dirty reads possible |
| **Read Committed** | No dirty reads, but phantom possible |
| **Repeatable Read** | No non-repeatable reads |
| **Serializable** | Full isolation (but slowest) |

---

## Question 2: Data Integrity Testing (15 points)

### Scenario
ToolShop's database has these relationships:
- Orders belong to Customers (foreign key)
- OrderItems belong to Orders (foreign key)
- Products have Categories (foreign key)
- Inventory tracks Product stock (foreign key)

### Questions

**a) (5 points)** What types of data integrity should be tested?

**Expected Answer:**

| Integrity Type | Description | Test Example |
|----------------|-------------|--------------|
| **Entity integrity** | Primary key is unique and not null | Cannot create two orders with same ID |
| **Referential integrity** | Foreign keys reference valid records | Cannot create order for non-existent customer |
| **Domain integrity** | Values within allowed range/type | Price cannot be negative |
| **User-defined integrity** | Business rules enforced | Cannot order more than available inventory |

**b) (5 points)** How would you test referential integrity constraints?

**Expected Answer:**

**Test scenarios for foreign key constraints:**

| Constraint | Test Action | Expected Result |
|------------|-------------|-----------------|
| **Insert with valid FK** | Create order for existing customer | Success |
| **Insert with invalid FK** | Create order for non-existent customer (ID=9999) | Error: FK violation |
| **Update to invalid FK** | Change order's customer_id to invalid | Error: FK violation |
| **Delete referenced record** | Delete customer with orders | Error or cascade (depends on rule) |

**ON DELETE behavior testing:**

| Rule | Behavior | Test |
|------|----------|------|
| **RESTRICT** | Cannot delete if referenced | Delete customer with orders → Error |
| **CASCADE** | Delete children when parent deleted | Delete category → Products deleted |
| **SET NULL** | Set FK to null when parent deleted | Delete customer → Order.customer_id = NULL |
| **SET DEFAULT** | Set FK to default when parent deleted | Delete category → Product.category = 'Uncategorized' |

**c) (5 points)** Design tests for business rule constraints (check constraints, triggers).

**Expected Answer:**

**ToolShop business rule tests:**

| Rule | Implementation | Test |
|------|----------------|------|
| **Price > 0** | CHECK constraint | Insert product with price = -5 → Error |
| **Quantity ≥ 0** | CHECK constraint | Update inventory to -1 → Error |
| **Order total matches items** | Trigger | Insert order, verify trigger updates total |
| **Audit timestamp** | Trigger | Update record, verify updated_at changed |

**Test approach for triggers:**
```
Scenario: Order total calculation trigger

Setup:
  Create order #123

Action:
  Insert order_item (product=$10, quantity=2)
  Insert order_item (product=$15, quantity=1)

Verify:
  Order #123 total = (10×2) + (15×1) = $35
  Trigger should auto-calculate this

Edge cases:
  - Delete order_item → Total recalculated
  - Update quantity → Total recalculated
  - Empty order → Total = 0
```

---

## Question 3: Query Performance Testing (20 points)

### Scenario
ToolShop's product search is slow. The query searches across:
- Product name (text search)
- Category (exact match)
- Price range (between)
- Availability (boolean)

Database has 1 million products.

### Questions

**a) (5 points)** What metrics should be measured for query performance?

**Expected Answer:**

| Metric | Description | Target Example |
|--------|-------------|----------------|
| **Execution time** | Total query duration | < 100ms for simple queries |
| **Rows examined** | Rows scanned vs returned | Ratio < 10:1 |
| **Index usage** | Whether indexes are used | Query uses index (not full scan) |
| **Buffer/cache hits** | Data from memory vs disk | > 95% cache hit rate |
| **Lock wait time** | Time waiting for locks | < 10ms |

**b) (5 points)** Explain how indexes work and when they should be used.

**Expected Answer:**

**Index concepts:**

| Concept | Description |
|---------|-------------|
| **B-tree index** | Balanced tree for range/equality queries |
| **Hash index** | Direct lookup, equality only |
| **Composite index** | Multiple columns in one index |
| **Covering index** | Contains all query columns |
| **Full-text index** | Text search optimization |

**When to use indexes:**

| Scenario | Index Recommendation |
|----------|---------------------|
| **Primary key lookups** | Always indexed (automatic) |
| **Foreign key joins** | Index FK columns |
| **WHERE clause columns** | Index frequently filtered columns |
| **ORDER BY columns** | Index if sorting large result sets |
| **Aggregation (GROUP BY)** | Index grouped columns |

**Trade-offs:**
```
Indexes speed up: SELECT (reads)
Indexes slow down: INSERT, UPDATE, DELETE (writes)

Rule of thumb:
  - Read-heavy tables: More indexes
  - Write-heavy tables: Fewer indexes
  - Balance based on workload
```

**c) (5 points)** How would you identify and fix a slow query?

**Expected Answer:**

**Diagnostic steps:**

| Step | Action | Tool/Method |
|------|--------|-------------|
| **1. Identify slow queries** | Find queries taking > threshold | Slow query log |
| **2. Analyze execution plan** | See how query is executed | EXPLAIN or EXPLAIN ANALYZE |
| **3. Check index usage** | Verify indexes are used | Execution plan shows "Index Scan" vs "Seq Scan" |
| **4. Examine data volume** | Rows scanned vs returned | Execution plan metrics |
| **5. Test optimization** | Apply fix, re-measure | Before/after comparison |

**Execution plan interpretation:**
```
Bad plan (full table scan):
  Seq Scan on products  (cost=0.00..35000.00 rows=1000000)
    Filter: (category = 'tools')
  → Scanning all 1M rows!

Good plan (index usage):
  Index Scan using idx_category on products (cost=0.43..8.45 rows=500)
    Index Cond: (category = 'tools')
  → Only reading matching rows via index
```

**Common fixes:**

| Problem | Fix |
|---------|-----|
| **Missing index** | Create index on filter columns |
| **Wrong index** | Create composite index matching query |
| **Too many columns** | Select only needed columns |
| **N+1 queries** | Use JOIN or batch fetching |
| **Unoptimized JOIN** | Ensure join columns indexed |

**d) (5 points)** What is database load testing and how would you perform it?

**Expected Answer:**

**Database load testing:** Verifying database performance under expected and peak load.

**Test types:**

| Type | Purpose | Method |
|------|---------|--------|
| **Throughput test** | Max queries/second | Gradually increase load |
| **Concurrency test** | Handle parallel connections | Multiple threads querying |
| **Stress test** | Find breaking point | Increase until failure |
| **Endurance test** | Long-term stability | Sustained load over hours |

**Approach for ToolShop:**
```
Test scenario: Product search under load

Metrics to collect:
  - Queries per second
  - Response time percentiles (p50, p95, p99)
  - Error rate
  - Connection pool utilization
  - CPU/memory/disk I/O on database server

Load pattern:
  Baseline:  50 concurrent users
  Expected:  200 concurrent users
  Peak:      500 concurrent users
  Stress:    1000+ concurrent users (find limit)

Pass criteria:
  - p95 response time < 200ms at expected load
  - Error rate < 0.1%
  - No connection pool exhaustion
```

---

## Question 4: Database Migration Testing (15 points)

### Scenario
ToolShop needs to migrate the database:
1. Add new column: `products.weight` (for shipping calculation)
2. Rename column: `customers.phone` → `customers.mobile`
3. Split table: `addresses` extracted from `customers`

### Questions

**a) (5 points)** What risks are involved in database migrations?

**Expected Answer:**

| Risk | Description | Mitigation |
|------|-------------|------------|
| **Data loss** | Columns dropped or data corrupted | Backup before migration, verify row counts |
| **Downtime** | Service unavailable during migration | Use online migration techniques |
| **Application incompatibility** | Code expects old schema | Coordinate deploy with migration |
| **Performance degradation** | Large table locks | Batch operations, off-peak timing |
| **Rollback difficulty** | Can't undo destructive changes | Test rollback procedure |
| **Constraint violations** | New constraints on existing bad data | Clean data before migration |

**b) (5 points)** How would you test the column rename migration?

**Expected Answer:**

**Migration testing strategy:**

| Phase | Test |
|-------|------|
| **Pre-migration** | Verify current state (column exists, data present) |
| **Migration execution** | Run migration, capture any errors |
| **Post-migration schema** | Verify new column exists, old doesn't |
| **Post-migration data** | All data preserved in new column |
| **Application test** | Code works with new schema |
| **Rollback test** | Can revert if needed |

**Detailed tests for rename:**
```
Before migration:
  - Count: SELECT COUNT(*) FROM customers WHERE phone IS NOT NULL
  - Sample: SELECT id, phone FROM customers LIMIT 10

Run migration:
  - ALTER TABLE customers RENAME COLUMN phone TO mobile

After migration:
  - Schema: DESCRIBE customers (mobile exists, phone doesn't)
  - Count: SELECT COUNT(*) FROM customers WHERE mobile IS NOT NULL
    → Should equal before count
  - Sample: SELECT id, mobile FROM customers WHERE id IN (sampled ids)
    → Values should match before values

Application:
  - Update API reads mobile correctly
  - Search by mobile works
  - Create customer with mobile works
```

**c) (5 points)** What is the "expand and contract" pattern for zero-downtime migrations?

**Expected Answer:**

**Expand and Contract pattern:** Gradual migration allowing old and new code to coexist.

**Phases:**

| Phase | Database State | Code State |
|-------|----------------|------------|
| **Expand** | Add new column/table | Write to both old and new |
| **Migrate** | Copy data to new structure | Read from new, write to both |
| **Contract** | Remove old structure | Only use new structure |

**Example: Adding `products.weight`**
```
Phase 1 - Expand:
  Database: Add weight column (nullable)
  Code: Write weight when available, null otherwise

Phase 2 - Migrate:
  Background job: Populate weight from product catalog
  Code: Read weight, handle null gracefully

Phase 3 - Contract:
  Database: Add NOT NULL constraint (once all populated)
  Code: Remove null handling (weight always exists)
```

**Benefits:**
- No downtime (always backward compatible)
- Rollback possible between phases
- Risk isolated to each phase

---

## Question 5: Database Security Testing (15 points)

### Scenario
ToolShop stores sensitive customer data: names, emails, addresses, payment methods.

### Questions

**a) (5 points)** What SQL injection vulnerabilities should be tested?

**Expected Answer:**

**SQL injection types:**

| Type | Description | Example |
|------|-------------|---------|
| **Classic injection** | Manipulate WHERE clause | `' OR '1'='1` |
| **Union-based** | Extract data from other tables | `' UNION SELECT password FROM users--` |
| **Blind injection** | Infer data from timing/behavior | `' AND 1=1--` vs `' AND 1=2--` |
| **Second-order** | Stored input executed later | Inject in registration, trigger in report |

**Test approach:**
```
Input field: Product search

Test payloads:
  ' OR '1'='1
  '; DROP TABLE products;--
  ' UNION SELECT username, password FROM users--
  1; WAITFOR DELAY '0:0:5'--

Expected result:
  - All payloads should be rejected or sanitized
  - No SQL errors exposed to user
  - Query should not be modified
  - Parameterized queries should prevent all injection
```

**b) (5 points)** How should sensitive data be protected in the database?

**Expected Answer:**

**Data protection strategies:**

| Strategy | Description | Application |
|----------|-------------|-------------|
| **Encryption at rest** | Data encrypted on disk | Full disk or column-level encryption |
| **Encryption in transit** | Data encrypted during transmission | TLS/SSL connections |
| **Column-level encryption** | Specific columns encrypted | Credit card numbers, SSN |
| **Hashing** | One-way transformation | Passwords (with salt) |
| **Masking** | Partial data exposure | Show only last 4 digits of card |
| **Tokenization** | Replace sensitive with token | Payment card tokenization |

**Password storage test:**
```
Verify passwords are properly hashed:

Test:
  1. Create user with password "TestPassword123"
  2. Query password directly: SELECT password FROM users WHERE id = X

Expected:
  - Password is hashed (not plaintext)
  - Hash looks like: $2b$12$... (bcrypt) or similar
  - Same password for two users produces different hashes (salted)

Verify:
  - Length appropriate for algorithm (bcrypt = 60 chars)
  - Cannot derive original password from hash
```

**c) (5 points)** What access control tests should be performed?

**Expected Answer:**

**Access control test areas:**

| Area | Test |
|------|------|
| **Authentication** | Verify database requires valid credentials |
| **User privileges** | Each user has minimum required permissions |
| **Role separation** | Admin vs read-only vs application accounts |
| **Object permissions** | Tables, views, procedures access controlled |
| **Row-level security** | Users only see their own data |

**Test scenarios:**
```
Scenario: Application database user permissions

Test:
  Connect as application user (not admin)

Verify CAN:
  - SELECT from application tables
  - INSERT/UPDATE/DELETE in application tables
  - Execute stored procedures

Verify CANNOT:
  - DROP any table
  - CREATE or ALTER tables
  - Access system tables
  - GRANT permissions to others
  - Access other databases

Scenario: Row-level security

Test:
  Connect as Customer A
  SELECT * FROM orders

Expected:
  - Only Customer A's orders returned
  - Cannot see Customer B's orders
```

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | ACID Properties Testing | 15 |
| Q2 | Data Integrity Testing | 15 |
| Q3 | Query Performance Testing | 20 |
| Q4 | Migration Testing | 15 |
| Q5 | Security Testing | 15 |
| **Total** | | **80** |
