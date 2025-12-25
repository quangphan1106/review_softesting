# API Testing - Concept Questions

## Question 1: REST API Testing Fundamentals (15 points)

### Scenario
ToolShop has these API endpoints:
- `GET /api/products` - List all products
- `GET /api/products/{id}` - Get product details
- `POST /api/cart` - Add item to cart
- `PUT /api/cart/{id}` - Update cart item quantity
- `DELETE /api/cart/{id}` - Remove item from cart
- `POST /api/orders` - Create order (requires authentication)

### Questions

**a) (5 points)** Explain REST principles and how they apply to ToolShop's API design.

**Expected Answer:**

**REST (Representational State Transfer) principles:**

| Principle | Meaning | ToolShop Application |
|-----------|---------|---------------------|
| **Stateless** | Each request contains all info needed | No server session; auth token sent with each request |
| **Client-Server** | Separation of concerns | Frontend (React) independent from API |
| **Uniform Interface** | Consistent API patterns | All endpoints follow same URL/verb conventions |
| **Resource-based** | URLs represent resources | `/products`, `/cart`, `/orders` are resources |
| **HTTP Verbs** | Actions via standard methods | GET=read, POST=create, PUT=update, DELETE=remove |

**HTTP methods mapping:**

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Read resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Update/Replace | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

**b) (5 points)** What should you test for each HTTP method? List test cases for GET /api/products.

**Expected Answer:**

**Test categories for GET /api/products:**

| Category | Test Case | Expected Result |
|----------|-----------|-----------------|
| **Happy path** | Get products without params | 200 OK, array of products |
| **Pagination** | Get page 2 with limit 10 | 200 OK, correct subset |
| **Filtering** | Filter by category=tools | 200 OK, only tools returned |
| **Sorting** | Sort by price ascending | 200 OK, lowest price first |
| **Empty result** | Filter with no matches | 200 OK, empty array (not 404) |
| **Invalid params** | Invalid sort field | 400 Bad Request |
| **Security** | Request without auth (if required) | 401 Unauthorized |
| **Performance** | Large dataset | Response < SLA time |

**Response structure validation:**
```
Verify:
- Status code (200, 400, 404, etc.)
- Response headers (Content-Type: application/json)
- Response body schema
- Correct data values
- Pagination metadata (total, page, limit)
```

**c) (5 points)** Explain the difference between positive and negative API testing with examples.

**Expected Answer:**

**Positive testing:** Verify API works correctly with valid inputs

| Endpoint | Positive Test |
|----------|---------------|
| POST /api/cart | Add valid product with valid quantity |
| PUT /api/cart/{id} | Update existing item to valid quantity |
| POST /api/orders | Create order with valid address and payment |

**Negative testing:** Verify API handles invalid inputs correctly

| Endpoint | Negative Test | Expected |
|----------|---------------|----------|
| POST /api/cart | Add product with quantity = 0 | 400 Bad Request |
| POST /api/cart | Add non-existent product ID | 404 Not Found |
| PUT /api/cart/{id} | Update with negative quantity | 400 Bad Request |
| DELETE /api/cart/{id} | Delete non-existent item | 404 Not Found |
| POST /api/orders | Create order without auth | 401 Unauthorized |

**Boundary testing:**

| Field | Invalid Values to Test |
|-------|----------------------|
| Quantity | 0, -1, MAX_INT+1, non-numeric |
| Product ID | Empty, non-existent, malformed |
| Price | Negative, zero (for payment) |
| Email | Invalid format, too long |

---

## Question 2: API Authentication Testing (15 points)

### Scenario
ToolShop uses JWT (JSON Web Token) authentication:
- `POST /api/auth/login` - Get access token (15 min expiry)
- `POST /api/auth/refresh` - Refresh token (7 day expiry)
- Protected endpoints require `Authorization: Bearer <token>` header

### Questions

**a) (5 points)** Explain JWT authentication and its components.

**Expected Answer:**

**JWT Structure:**
```
header.payload.signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header (base64)
eyJ1c2VySWQiOiIxMjMiLCJyb2xlIjoiY3VzdG9tZXIi...  ← Payload (base64)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c    ← Signature
```

**Components:**

| Part | Contains | Purpose |
|------|----------|---------|
| **Header** | Algorithm (HS256), token type | Parsing instructions |
| **Payload** | User ID, role, expiry time | User identity claims |
| **Signature** | HMAC of header+payload | Verify token integrity |

**Token flow:**
```
1. User sends credentials → POST /api/auth/login
2. Server validates, creates JWT → Returns access_token + refresh_token
3. Client stores tokens → localStorage or httpOnly cookie
4. Client sends requests → Authorization: Bearer <access_token>
5. Server validates signature → Extracts user from payload
6. Token expires → Use refresh_token to get new access_token
```

**b) (5 points)** List security test cases for JWT authentication.

**Expected Answer:**

| Category | Test Case | Expected Result |
|----------|-----------|-----------------|
| **Token absence** | Request without token | 401 Unauthorized |
| **Invalid token** | Malformed token string | 401 Unauthorized |
| **Expired token** | Token past expiry time | 401 Unauthorized |
| **Wrong signature** | Modified payload, old signature | 401 Unauthorized |
| **Algorithm none** | Token with alg:none | 401 Unauthorized (security check) |
| **Role escalation** | Customer token accessing admin route | 403 Forbidden |
| **Refresh token reuse** | Use same refresh token twice | 401 (after first use invalidates it) |
| **Cross-user access** | User A's token accessing User B's data | 403 Forbidden |

**Token manipulation tests:**
```
Attack: Modify payload to change userId
Step 1: Decode payload (base64)
Step 2: Change userId: "123" → "456"
Step 3: Re-encode (signature now invalid)
Result: Server should reject (signature mismatch)
```

**c) (5 points)** How would you test the token refresh flow?

**Expected Answer:**

**Refresh flow test cases:**

| Scenario | Test Steps | Expected |
|----------|------------|----------|
| **Happy path** | Valid refresh token → /refresh | New access token returned |
| **Expired refresh** | Expired refresh token → /refresh | 401 Unauthorized |
| **Used refresh** | Same refresh token second time | 401 (token rotation) |
| **Invalid refresh** | Malformed refresh token | 401 Unauthorized |
| **Access still valid** | Refresh while access token works | New tokens (should work) |

**Token lifecycle testing:**
```
Timeline test:
─────────────────────────────────────────────────────────▶ Time
0m     5m     10m    15m    20m    25m
│      │      │      │      │      │
├──────────────────────┤  Access token valid (15m)
│  API calls work      │  API calls fail (401)
                       │
                       └─▶ Refresh token must be used here

├──────────────────────────────────────────────────────────┤
                        Refresh token valid (7 days)
```

---

## Question 3: API Error Handling (15 points)

### Scenario
ToolShop's API should return standardized error responses:
```
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      {"field": "quantity", "message": "Must be positive integer"}
    ]
  }
}
```

### Questions

**a) (5 points)** What HTTP status codes should be used for different error types?

**Expected Answer:**

| Status Code | Meaning | When to Use |
|-------------|---------|-------------|
| **400** | Bad Request | Invalid input, validation errors |
| **401** | Unauthorized | Missing or invalid authentication |
| **403** | Forbidden | Valid auth but insufficient permissions |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method for endpoint |
| **409** | Conflict | Resource state conflict (e.g., duplicate) |
| **422** | Unprocessable Entity | Valid syntax but semantic errors |
| **429** | Too Many Requests | Rate limit exceeded |
| **500** | Internal Server Error | Server-side error (bug) |
| **502** | Bad Gateway | Upstream service failure |
| **503** | Service Unavailable | Server overloaded or maintenance |

**Common mistakes to test:**

| Scenario | Wrong Response | Correct Response |
|----------|---------------|------------------|
| User not logged in | 403 Forbidden | 401 Unauthorized |
| Resource deleted | 500 Error | 404 Not Found |
| Validation failed | 500 Error | 400 Bad Request |
| User can't access other's data | 404 Not Found | 403 Forbidden (or 404 for security) |

**b) (5 points)** How should you test error response structure and content?

**Expected Answer:**

**Error response validation:**

| Aspect | Test |
|--------|------|
| **Structure** | Response matches error schema |
| **Code** | Error code is documented value |
| **Message** | Human-readable, helpful |
| **Details** | Field-specific errors included |
| **No sensitive data** | Stack traces not exposed |
| **Consistency** | Same format across all endpoints |

**Error detail requirements:**

| Field Error | Good Response | Bad Response |
|-------------|---------------|--------------|
| Invalid email | "Please enter valid email format" | "Validation failed" |
| Quantity too high | "Maximum quantity is 99" | "Invalid quantity" |
| Required field | "Email is required" | "Missing field" |

**Security considerations:**
```
BAD (exposes internals):
{
  "error": {
    "message": "SQL Error: SELECT * FROM users WHERE id='123'--"
    "stack": "at Database.query (db.js:45)..."
  }
}

GOOD (safe):
{
  "error": {
    "code": "DATABASE_ERROR",
    "message": "Unable to process request. Please try again."
  }
}
```

**c) (5 points)** What is error rate testing and how would you implement it?

**Expected Answer:**

**Error rate testing:** Measuring frequency of errors under various conditions.

**Implementation approach:**

| Metric | Calculation | Threshold |
|--------|-------------|-----------|
| **Error rate** | (Error responses / Total requests) × 100% | < 1% |
| **4xx rate** | Client errors / Total | < 5% (depends on usage) |
| **5xx rate** | Server errors / Total | < 0.1% |

**Test scenarios:**

| Scenario | Method | Expected Error Rate |
|----------|--------|---------------------|
| **Normal load** | 100 RPS valid requests | < 0.1% 5xx |
| **Stress test** | 1000 RPS (10x normal) | < 1% 5xx |
| **Invalid input flood** | Many bad requests | 100% 4xx (expected) |
| **Dependency failure** | Database down | Graceful 503 |

**Monitoring approach:**
```
Track per endpoint:
┌─────────────────────────────────────────┐
│ Endpoint: POST /api/orders              │
├──────────────────────────────┬──────────┤
│ Total requests (1hr)         │ 10,000   │
│ 2xx Success                  │ 9,850    │
│ 4xx Client errors            │ 140      │
│ 5xx Server errors            │ 10       │
│ Error rate                   │ 0.10%    │
│ Status                       │ ✅ OK    │
└──────────────────────────────┴──────────┘
```

---

## Question 4: API Contract Testing (15 points)

### Scenario
ToolShop has separate frontend and backend teams. The product API contract states:
```
GET /api/products/{id}
Response 200:
{
  "id": integer,
  "name": string (1-100 chars),
  "price": number (positive),
  "category": enum ["tools", "clothing", "accessories"],
  "inStock": boolean
}
```

### Questions

**a) (5 points)** What is API contract testing and why is it important?

**Expected Answer:**

**API contract:** Formal agreement on request/response format between API provider and consumers.

**Contract testing:** Verifying both sides adhere to agreed contract.

**Why important:**

| Benefit | Explanation |
|---------|-------------|
| **Catch integration issues early** | Before deployment, not production |
| **Enable parallel development** | Frontend/backend work independently |
| **Prevent breaking changes** | Detect schema changes that break consumers |
| **Documentation as code** | Contract serves as living documentation |
| **Faster feedback** | No need for full environment integration |

**What can break:**

| Breaking Change | Impact |
|-----------------|--------|
| Field renamed | Frontend can't find data |
| Field removed | Frontend crashes on missing data |
| Type changed | Parsing errors |
| New required field | Old requests fail validation |

**b) (5 points)** What aspects of the API response should contract tests verify?

**Expected Answer:**

**Contract verification checklist:**

| Aspect | What to Verify | Example |
|--------|----------------|---------|
| **Status code** | Correct HTTP status | 200 for success |
| **Content-Type** | Correct media type | application/json |
| **Required fields** | All required fields present | id, name, price |
| **Field types** | Correct data types | price is number, not string |
| **Field constraints** | Value ranges, formats | price > 0, name ≤ 100 chars |
| **Enum values** | Only allowed values | category in [tools, clothing, accessories] |
| **Nullable fields** | Null handling correct | inStock can be null or must exist |
| **Array structure** | Array items match schema | products[] each match Product schema |

**Schema validation example:**
```
Expected:
{
  "id": integer (required),
  "name": string (required, 1-100 chars),
  "price": number (required, positive),
  "category": enum (required),
  "inStock": boolean (required)
}

Actual (FAIL):
{
  "id": "123",         ← Wrong type (string, not integer)
  "name": "Hammer",
  "price": -5.99,      ← Constraint violation (negative)
  "category": "TOOLS", ← Wrong case (TOOLS vs tools)
  "stock": true        ← Wrong field name (stock vs inStock)
}
```

**c) (5 points)** Explain consumer-driven contract testing (CDCT).

**Expected Answer:**

**Consumer-Driven Contract Testing:** Consumers (API clients) define their expectations; providers verify they meet those expectations.

**Flow:**
```
┌─────────────┐     1. Define contract     ┌─────────────┐
│   Frontend  │ ─────────────────────────▶ │   Contract  │
│  (Consumer) │     "I expect this schema" │    File     │
└─────────────┘                            └──────┬──────┘
                                                  │
                                           2. Share│
                                                  ▼
┌─────────────┐     3. Verify against      ┌─────────────┐
│   Backend   │ ◀───────────────────────── │   Contract  │
│  (Provider) │     "Do I fulfill this?"   │   Broker    │
└─────────────┘                            └─────────────┘
```

**Benefits vs traditional testing:**

| Aspect | Traditional | CDCT |
|--------|-------------|------|
| **Who defines** | Provider | Consumer |
| **Focuses on** | All API features | What consumer actually uses |
| **Detects** | Provider bugs | Breaking changes for consumers |
| **When runs** | Integration phase | Provider CI pipeline |

**CDCT tools:** Pact, Spring Cloud Contract

**Example scenario:**
- Frontend only uses `id`, `name`, `price` from Product
- Contract only specifies those three fields
- Backend can add `description` field safely (no consumer uses it)
- Backend can't remove `price` (consumer contract requires it)

---

## Question 5: API Testing Tools & Practices (20 points)

### Scenario
Your team needs to implement comprehensive API testing for ToolShop using Postman and automated CI/CD integration.

### Questions

**a) (5 points)** What features should an API testing tool provide?

**Expected Answer:**

| Feature | Purpose | Example |
|---------|---------|---------|
| **Request builder** | Construct HTTP requests easily | Set headers, body, params |
| **Environment variables** | Switch between environments | DEV, STAGING, PROD URLs |
| **Collection organization** | Group related tests | Product API, Cart API folders |
| **Assertions** | Validate responses | Status code, JSON schema |
| **Data-driven testing** | Run same test with different data | CSV/JSON data files |
| **Pre/post scripts** | Setup and teardown | Generate auth token, cleanup data |
| **Test chaining** | Pass data between requests | Create product → use ID in cart |
| **CI/CD integration** | Automated execution | CLI runner, exit codes |
| **Reporting** | Test results visualization | HTML reports, metrics |
| **Mock servers** | Simulate API responses | Frontend development without backend |

**b) (5 points)** How would you structure API test collections for ToolShop?

**Expected Answer:**

**Collection hierarchy:**
```
ToolShop API Tests/
├── 🔐 Auth/
│   ├── Login - Valid credentials
│   ├── Login - Invalid password
│   ├── Login - Non-existent user
│   ├── Refresh token - Valid
│   └── Refresh token - Expired
│
├── 📦 Products/
│   ├── List products - Default pagination
│   ├── List products - Filter by category
│   ├── Get product - Valid ID
│   └── Get product - Invalid ID
│
├── 🛒 Cart/
│   ├── Add to cart - Valid product
│   ├── Add to cart - Invalid quantity
│   ├── Update quantity - Valid
│   ├── Remove item - Existing
│   └── Remove item - Non-existing
│
├── 📋 Orders/
│   ├── Create order - Valid (authenticated)
│   ├── Create order - Unauthenticated
│   └── Get order history
│
└── 🔄 E2E Flows/
    ├── Complete checkout flow
    └── Guest checkout flow
```

**Environment structure:**

| Variable | Dev | Staging | Prod |
|----------|-----|---------|------|
| base_url | localhost:3000 | staging.toolshop.com | api.toolshop.com |
| test_user | dev@test.com | staging@test.com | (no test user) |
| timeout | 30s | 10s | 5s |

**c) (5 points)** What is the difference between API smoke tests, regression tests, and integration tests?

**Expected Answer:**

| Type | Purpose | Scope | When Run | Duration |
|------|---------|-------|----------|----------|
| **Smoke tests** | Verify API is alive | Critical endpoints only | Every deploy | < 1 min |
| **Regression tests** | Verify nothing broke | All endpoints, scenarios | Daily/PR | 10-30 min |
| **Integration tests** | Verify systems work together | Cross-service flows | Nightly | 30+ min |

**Smoke test examples:**
```
1. GET /health → 200 OK
2. GET /api/products → 200 OK, returns array
3. POST /api/auth/login → 200 OK with valid creds
4. POST /api/orders (auth) → 201 Created

If any fail → Deployment blocked
```

**Regression test coverage:**
- All endpoints, all HTTP methods
- Positive and negative cases
- Edge cases and boundaries
- Error responses

**Integration test scope:**
- Full user journeys (browse → cart → checkout)
- Third-party services (payment gateway)
- Database state changes
- Message queue flows

**d) (5 points)** How do you handle test data and dependencies between API tests?

**Expected Answer:**

**Dependency handling strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Pre-request scripts** | Create required data before test | Create user before login test |
| **Test chaining** | Pass data via variables | Save created order ID for later tests |
| **Fixtures/Factories** | API calls to create test data | Populate products via admin API |
| **Database seeding** | Direct DB setup before tests | Bulk data for performance tests |
| **Independent tests** | Each test creates own data | Parallel execution support |

**Variable passing between requests:**
```
Request 1: POST /api/auth/login
  Response: { "token": "abc123" }
  Post-script: Set variable "auth_token" = response.token

Request 2: POST /api/cart
  Header: Authorization: Bearer {{auth_token}}

Request 3: POST /api/orders
  Body uses: {{cart_id}} from Request 2
```

**Cleanup strategies:**

| Strategy | When | How |
|----------|------|-----|
| **Post-test cleanup** | After each test | Delete created resources |
| **Pre-test cleanup** | Before each test | Ensure clean state |
| **Scheduled cleanup** | Nightly | Delete test data older than X days |
| **Isolated environments** | Per test run | Spin up/down test database |

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | REST API Fundamentals | 15 |
| Q2 | Authentication Testing | 15 |
| Q3 | Error Handling | 15 |
| Q4 | Contract Testing | 15 |
| Q5 | Tools & Practices | 20 |
| **Total** | | **80** |
