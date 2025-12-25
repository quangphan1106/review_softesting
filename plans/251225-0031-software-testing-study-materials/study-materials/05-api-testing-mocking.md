# Topic 5: API Testing & Mocking - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 What is API Testing?

**Definition:** Testing application programming interfaces directly, bypassing the UI to validate business logic, data handling, security, and performance.

**Why API Testing?**
- Faster than UI testing (no rendering)
- More stable (no UI changes)
- Earlier detection of issues
- Better coverage of edge cases

### 1.2 API Testing Levels

```
┌─────────────────────────────────────────────────────────────┐
│                      API TESTING LAYERS                      │
├─────────────────────────────────────────────────────────────┤
│  Unit Tests        → Individual functions/methods            │
│  Integration Tests → API + Database/Services                 │
│  Contract Tests    → API matches specification               │
│  End-to-End Tests  → Full user flows via API                │
│  Performance Tests → Load, stress, spike                     │
│  Security Tests    → OWASP vulnerabilities                   │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 REST API Testing Fundamentals

**HTTP Methods:**
| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Update resource | No | No |
| DELETE | Remove resource | Yes | No |

**HTTP Status Codes:**
| Code | Meaning | When to Test |
|------|---------|--------------|
| 200 | OK | Success responses |
| 201 | Created | POST success |
| 204 | No Content | DELETE success |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Auth required |
| 403 | Forbidden | No permission |
| 404 | Not Found | Missing resource |
| 422 | Unprocessable | Validation error |
| 500 | Server Error | Unexpected errors |

### 1.4 Types of Mocking

| Type | Definition | Use Case |
|------|------------|----------|
| **Stub** | Fixed response, no logic | Simple tests |
| **Mock** | Verifies interactions | Behavior testing |
| **Fake** | Simplified implementation | Integration tests |
| **Spy** | Records calls, passes through | Monitoring |

---

## 2. Tools & Techniques Comparison

### 2.1 API Testing Tools Comparison

| Feature | Postman | Newman | REST Assured | Playwright |
|---------|---------|--------|--------------|------------|
| **Type** | GUI + CLI | CLI only | Code library | Framework |
| **Language** | JavaScript | JavaScript | Java | JS/TS/Python |
| **CI/CD** | Via Newman | Native | Native | Native |
| **Mocking** | Built-in | Via Postman | WireMock | msw/native |
| **Learning** | Easy | Easy | Medium | Medium |
| **Best For** | Manual + Auto | Automation | Java teams | Full-stack |

### 2.2 Postman & Newman

**Postman Features:**
- GUI for request building
- Collections for organization
- Environment variables
- Pre-request scripts
- Test scripts (JavaScript)
- Mock servers
- Monitors

**Newman CLI:**
| Command | Purpose |
|---------|---------|
| `newman run collection.json` | Basic run |
| `-e environment.json` | With environment |
| `--reporters cli,json,html` | Multiple reporters |
| `-n 10` | 10 iterations |
| `-d testdata.csv` | Data-driven testing |

### 2.3 Mock Server Tools

| Tool | Type | Best For |
|------|------|----------|
| **Postman Mock** | Cloud-hosted | Quick setup, collaboration |
| **WireMock** | Local/Cloud | Complex scenarios, recording |
| **Mock Service Worker** | Browser/Node | Frontend testing |
| **json-server** | Local | Simple REST mocks |
| **Mockoon** | Local GUI | Desktop development |

**WireMock vs Postman Mock:**

| Feature | WireMock | Postman Mock |
|---------|----------|--------------|
| Hosting | Self-hosted/Cloud | Cloud only |
| Response Delays | Fixed/random/chunked | Fixed only |
| Recording | Yes (proxy mode) | No |
| Rate Limits | No | Yes |
| Custom Logic | Java/JSON | JavaScript |
| Best For | Complex mocking | Team collaboration |

---

## 3. Practical Test Case Design

### 3.1 Postman Collection Structure

```
📁 Toolshop API Tests
├── 📁 Auth
│   ├── 🔵 POST /users/login (valid credentials)
│   ├── 🔴 POST /users/login (invalid password)
│   ├── 🔴 POST /users/login (missing email)
│   └── 🔵 POST /users/register
├── 📁 Products
│   ├── 🔵 GET /products (list all)
│   ├── 🔵 GET /products/{id}
│   ├── 🔵 GET /products?by_brand=1
│   └── 🔴 GET /products/invalid-id
├── 📁 Cart
│   ├── 🔵 POST /carts (create cart)
│   ├── 🔵 POST /carts/{id}/items
│   └── 🔴 POST /carts/{id}/items (invalid product)
└── 📁 Payment
    ├── 🔵 POST /payment/check (valid card)
    └── 🔴 POST /payment/check (invalid card)
```

### 3.2 Postman Test Concepts

**Common Assertions:**
| Assertion | Syntax |
|-----------|--------|
| Status code | `pm.response.to.have.status(200)` |
| JSON array | `pm.expect(json.data).to.be.an("array")` |
| Length > 0 | `pm.expect(json.data.length).to.be.greaterThan(0)` |
| Schema | `pm.response.to.have.jsonSchema(schema)` |

**Pre-request Auth Pattern:**
1. Build login request with `pm.environment.get("base_url")`
2. Send via `pm.sendRequest(request, callback)`
3. Extract token: `response.json().access_token`
4. Store: `pm.environment.set("auth_token", token)`

### 3.3 Contract Testing with OpenAPI

**OpenAPI Spec Concepts:**
- Define paths, methods, parameters, responses
- Schema validation with `$ref` to component schemas
- Response codes with expected content types

**Contract Validation (Dredd):**
- `dredd api-spec.yaml http://localhost:3000` - Run contract tests
- Validates API implementation matches spec
- Use `--hookfiles` for custom setup/teardown

### 3.4 Test Cases for API Testing

**TC-API-001: Valid Login**
- **Endpoint**: POST /users/login
- **Input**: `{"email": "customer@test.com", "password": "welcome01"}`
- **Expected**: 200 OK, token returned
- **Validation**: Response contains `access_token`

**TC-API-002: Invalid Password**
- **Endpoint**: POST /users/login
- **Input**: `{"email": "customer@test.com", "password": "wrong"}`
- **Expected**: 401 Unauthorized
- **Validation**: Error message present

**TC-API-003: Product Filter by Brand**
- **Endpoint**: GET /products?by_brand=1
- **Expected**: 200 OK, filtered products
- **Validation**: All products have brand_id = 1

**TC-API-004: Create Cart Item**
- **Endpoint**: POST /carts/{cart_id}/items
- **Input**: `{"product_id": "123", "quantity": 2}`
- **Expected**: 201 Created
- **Validation**: Item added to cart

**TC-API-005: Payment Validation**
- **Endpoint**: POST /payment/check
- **Input**: `{"method": "Credit Card", "account_number": "4111111111111111"}`
- **Expected**: 200 OK, success: true
- **Validation**: Payment accepted

---

## 4. Troubleshooting Scenarios

### 4.1 Common API Testing Issues

| Problem | Symptoms | Solutions |
|---------|----------|-----------|
| **Auth Failures** | 401 on protected endpoints | Verify token set, check format `Bearer <token>`, check expiry |
| **Response Mismatch** | Passes Postman, fails Newman | Export env with values, add delays, verify paths |
| **Flaky Tests** | Random pass/fail | Unique test data `test_${Date.now()}@example.com`, retry logic |

**Debug Tips:**
- Log token: `console.log(pm.environment.get("auth_token"))`
- Use pre-request to refresh tokens
- Chain requests with `pm.sendRequest`

### 4.2 API Security Testing

**OWASP API Top 10 Testing:**

| Vulnerability | Test Approach |
|--------------|---------------|
| **BOLA** (Broken Object Level Auth) | Access other users' data by changing IDs |
| **Broken Auth** | Test weak passwords, token expiry |
| **Excessive Data Exposure** | Check response for sensitive data |
| **Rate Limiting** | Send rapid requests, check for limiting |
| **BFLA** (Broken Function Level Auth) | Access admin endpoints as regular user |
| **Mass Assignment** | Send extra fields in POST/PUT |
| **Security Misconfiguration** | Check headers, CORS |
| **Injection** | SQL/NoSQL injection in parameters |
| **Improper Asset Management** | Find undocumented endpoints |
| **Logging & Monitoring** | Verify security events logged |

**BOLA Test:** User A accessing User B's data should return 403

---

## 5. Toolshop Homework Connection

### 5.1 From API Testing Homework (HW08)

**Test Environment:** ToolShop API (Sprint 5 with bugs)

| API | Endpoints | TCs | Pass Rate | Bugs |
|-----|-----------|-----|-----------|------|
| User Auth | /users/login, /register | 22 | 63.64% | 8 (incl. SQL injection CRITICAL) |
| Product Search | /products, /search | 32 | 100% | 0 |
| Payment | /payment/check | 25 | 60% | 10 |

### 5.2 Key Test Results

**Overall:** 79 TCs, 77.22% pass, 1 critical + 17 major bugs

**Bug Categories:** Validation (9), Wrong status codes (6), SQL injection (1), Format acceptance (2)

**Example Bug:** SQL Injection - `{"email": "' OR '1'='1"}` bypasses auth (200 OK instead of 400)

---

## 6. Application-Level Exam Questions

### Question 1: API Test Design
**Scenario:** Wishlist API: POST /items, GET /, DELETE /items/{id}

**Answer - 10 Test Cases:**
| Type | Test Case | Expected |
|------|-----------|----------|
| Positive | Add valid item | 201 |
| Positive | Get with items | 200, items returned |
| Positive | Delete existing | 204 |
| Negative | Add without auth | 401 |
| Negative | Add non-existent product | 404 |
| Negative | Delete non-existent | 404 |
| Negative | Add duplicate | 409 or 200 |
| Edge | Empty wishlist GET | 200, empty array |
| Edge | Max wishlist size | 400 or 201 |
| Boundary | Product ID at min/max | Appropriate response |

### Question 2: Mock Server Selection
**Scenario:** Mock payment gateway with delays, recording, CI/CD support

**Answer:** WireMock
- Delays: `{"fixedDelayMilliseconds": 2000}` (fixed/random/chunked)
- Recording: `--record-mappings --proxy-all`
- CI/CD: Docker image available
- Why not Postman Mock: Fixed delays only, no recording, rate-limited

### Question 3: Security Test Design
**Scenario:** Test for excessive data exposure

**Answer - Checks:**
- No password/ssn in response: `.not.have.property("password")`
- Phone masked: `/\*{6}\d{4}/`
- Email partial: `/^.{2}\*+@/`
- Admin endpoints return 403 for regular users
- No stack traces in error messages
- Non-sequential IDs (prevent enumeration)

---

## 7. Cheatsheet / Quick Reference

### Postman Test Syntax

| Category | Syntax |
|----------|--------|
| Status | `pm.response.to.have.status(200)` |
| JSON | `pm.response.to.be.json` |
| Header | `pm.response.to.have.header("Content-Type")` |
| Property | `pm.expect(json).to.have.property("id")` |
| Array | `pm.expect(json.items).to.be.an("array")` |
| Comparison | `pm.expect(json.count).to.be.greaterThan(0)` |
| Time | `pm.expect(pm.response.responseTime).to.be.below(200)` |
| Set var | `pm.environment.set("key", value)` |
| Get var | `pm.environment.get("key")` |
| Chain | `pm.sendRequest(req, callback)` |

### Newman Commands

| Command | Purpose |
|---------|---------|
| `newman run collection.json` | Basic run |
| `-e env.json` | With environment |
| `--reporters cli,html,json` | Multiple reporters |
| `-n 10` | Iterations |
| `-d data.csv` | Data file |
| `--bail` | Stop on error |
| `--delay-request 1000` | Delay between requests |

### Common HTTP Headers

| Type | Header |
|------|--------|
| Request | `Content-Type: application/json`, `Authorization: Bearer <token>` |
| Response | `X-RateLimit-Remaining`, `Cache-Control` |

### API Testing Checklist

- [ ] Positive tests (happy path)
- [ ] Negative tests (invalid input)
- [ ] Boundary tests (min/max values)
- [ ] Authentication tests
- [ ] Authorization tests (BOLA, BFLA)
- [ ] Rate limiting tests
- [ ] Response schema validation
- [ ] Error message validation
- [ ] Performance (response time)
- [ ] Security (injection, data exposure)

---

## 8. GraphQL Testing (2025 Update)

### 8.1 GraphQL vs REST

| Aspect | REST | GraphQL |
|--------|------|---------|
| Endpoint | Multiple (/users, /products) | Single (/graphql) |
| Data fetching | Fixed response | Client specifies fields |
| Versioning | URL (/v1, /v2) | Schema evolution |
| Over-fetching | Common | Eliminated |

### 8.2 GraphQL Query Testing

**Query Concepts:**
- POST to single endpoint with `query` and `variables` in body
- Check: `json.data.user` exists, `json.errors` undefined
- Specify only needed fields (no over-fetching)

### 8.3 GraphQL Mutation Testing

**Mutation Concepts:**
- `mutation CreateUser($input: CreateUserInput!)` defines operation
- Variables passed separately: `{input: {name, email, password}}`
- Assert: `json.data.createUser.id` exists

### 8.4 GraphQL Security Testing

| Vulnerability | Test | Expected |
|--------------|------|----------|
| **Introspection** | Query `__schema` | Should fail in prod |
| **Depth attack** | Deeply nested queries | Should be rejected |
| **Batching** | Multiple queries in one request | Should be limited |
| **Injection** | Special chars in variables | Should be sanitized |

### 8.5 GraphQL Testing Tools

| Tool | Purpose |
|------|---------|
| **Postman** | Manual testing with GraphQL support |
| **Insomnia** | Alternative to Postman |
| **Apollo Studio** | Schema management + testing |
| **graphql-test** | Pytest plugin for GraphQL |

---

## 9. Key Takeaways for Exam

1. **HTTP Methods**: Know GET/POST/PUT/PATCH/DELETE purposes
2. **Status Codes**: 2xx success, 4xx client error, 5xx server error
3. **Postman Tests**: Use pm.response and pm.expect
4. **Newman**: CLI for CI/CD integration
5. **Contract Testing**: OpenAPI/Swagger validation
6. **Mocking**: WireMock > Postman Mock for complex scenarios
7. **Security**: BOLA, BFLA are top API vulnerabilities
8. **GraphQL**: Single endpoint, client specifies fields, test introspection + depth attacks

---

*Study time recommended: 3-4 hours*
*Practice: Create a Postman collection for a sample API*
