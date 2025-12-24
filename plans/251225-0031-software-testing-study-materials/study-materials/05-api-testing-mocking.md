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
```bash
# Run collection
newman run collection.json

# With environment
newman run collection.json -e environment.json

# Multiple reporters
newman run collection.json --reporters cli,json,html

# With iterations
newman run collection.json -n 10

# With data file
newman run collection.json -d testdata.csv
```

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

### 3.2 Postman Test Scripts

**Basic Test:**
```javascript
// Status code check
pm.test("Status is 200", function() {
    pm.response.to.have.status(200);
});

// JSON validation
pm.test("Response has products", function() {
    const response = pm.response.json();
    pm.expect(response.data).to.be.an("array");
    pm.expect(response.data.length).to.be.greaterThan(0);
});

// Schema validation
const schema = {
    type: "object",
    required: ["id", "name", "price"],
    properties: {
        id: { type: "string" },
        name: { type: "string" },
        price: { type: "number", minimum: 0 }
    }
};

pm.test("Schema is valid", function() {
    pm.response.to.have.jsonSchema(schema);
});
```

**Pre-request Script (Auth):**
```javascript
// Get auth token and set as variable
const loginRequest = {
    url: pm.environment.get("base_url") + "/users/login",
    method: 'POST',
    header: { 'Content-Type': 'application/json' },
    body: {
        mode: 'raw',
        raw: JSON.stringify({
            email: pm.environment.get("email"),
            password: pm.environment.get("password")
        })
    }
};

pm.sendRequest(loginRequest, function(err, response) {
    const token = response.json().access_token;
    pm.environment.set("auth_token", token);
});
```

### 3.3 Contract Testing with OpenAPI

**OpenAPI Spec Example:**
```yaml
openapi: 3.0.0
info:
  title: Toolshop API
  version: 1.0.0
paths:
  /products:
    get:
      summary: List products
      parameters:
        - name: by_brand
          in: query
          schema:
            type: integer
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
```

**Contract Validation (Dredd):**
```bash
# Install Dredd
npm install -g dredd

# Run contract tests
dredd api-spec.yaml http://localhost:3000

# With hooks
dredd api-spec.yaml http://localhost:3000 --hookfiles=./hooks.js
```

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

**Problem 1: Authentication Failures**
```
Symptom: 401 Unauthorized on protected endpoints

Debug Steps:
1. Verify token is being set:
   console.log(pm.environment.get("auth_token"))

2. Check token format:
   Authorization: Bearer <token>

3. Verify token hasn't expired:
   Decode JWT and check exp claim

4. Check token scope/permissions:
   Some endpoints require admin token

Solution:
- Refresh token in pre-request script
- Use collection-level auth
- Add token refresh logic
```

**Problem 2: Response Mismatch**
```
Symptom: Test passes in Postman, fails in Newman

Causes:
- Environment variables not exported
- Timing issues (async operations)
- Different base URLs

Solutions:
1. Export environment with current values
2. Add delays between requests:
   pm.setTimeout(() => {}, 1000)
3. Verify environment file paths
```

**Problem 3: Flaky Tests**
```
Symptom: Tests pass/fail randomly

Causes:
- Race conditions
- Shared test data
- External dependencies

Solutions:
1. Use unique test data:
   const email = `test_${Date.now()}@example.com`

2. Chain requests with pm.sendRequest
3. Add retry logic:
   pm.test.retry(3, function() { ... })
```

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

**BOLA Test Example:**
```javascript
// Test: User A shouldn't access User B's data
pm.test("Cannot access other user's order", function() {
    // Logged in as User A, trying User B's order ID
    pm.response.to.have.status(403);
});
```

---

## 5. Toolshop Homework Connection

### 5.1 From API Testing Homework (HW08)

**Test Environment:**
- Application: ToolShop Practice Software Testing (Sprint 5 with bugs)
- Base URL: https://api-with-bugs.practicesoftwaretesting.com
- Documentation: /api/documentation

**APIs Tested:**

**API 1: User Authentication**
```
Endpoints: /users/login, /users/register
Test Cases: 22
Pass Rate: 63.64%
Bugs Found: 8
- SQL injection vulnerability (CRITICAL)
- Wrong HTTP status codes (MAJOR)
- Missing field validation (MAJOR)
```

**API 2: Product Search & Filter**
```
Endpoints: /products, /products/search
Test Cases: 32
Pass Rate: 100%
Bugs Found: 0
```

**API 3: Payment Processing**
```
Endpoint: /payment/check
Test Cases: 25
Pass Rate: 60%
Bugs Found: 10
- Missing field validation
- Invalid format acceptance
```

### 5.2 Key Test Results

**Overall Statistics:**
- Total Test Cases: 79
- Passed: 61 (77.22%)
- Failed: 18
- Critical Bugs: 1
- Major Bugs: 17

**Bug Categories:**
| Category | Count |
|----------|-------|
| Missing Field Validation | 9 |
| Wrong HTTP Status Codes | 6 |
| SQL Injection Vulnerability | 1 |
| Invalid Format Acceptance | 2 |

**Example Bug Report:**
```
BUG-001: SQL Injection in Login
Severity: CRITICAL
Endpoint: POST /users/login
Input: {"email": "' OR '1'='1", "password": "test"}
Expected: 400 Bad Request
Actual: 200 OK (bypasses authentication)
Impact: Complete authentication bypass
```

---

## 6. Application-Level Exam Questions

### Question 1: API Test Design
**Scenario:** Design tests for a new "Wishlist" API:
- POST /wishlist/items (add item)
- GET /wishlist (get all items)
- DELETE /wishlist/items/{id} (remove item)

**Question:** List 10 test cases covering positive, negative, and edge cases.

**Answer:**
```
Positive Tests:
1. Add valid item to wishlist → 201 Created
2. Get wishlist with items → 200 OK, items returned
3. Delete existing item → 204 No Content
4. Add multiple items → All appear in GET

Negative Tests:
5. Add item without auth → 401 Unauthorized
6. Add non-existent product → 404 Not Found
7. Delete non-existent item → 404 Not Found
8. Add duplicate item → 409 Conflict (or 200 if allowed)

Edge Cases:
9. Empty wishlist GET → 200 OK, empty array
10. Add item at max wishlist size → 400 or 201

Boundary Tests:
11. Product ID at boundary (0, max_int) → appropriate response
12. Very long product name → handled correctly
```

### Question 2: Mock Server Selection
**Scenario:** Team needs to mock a payment gateway for testing. Requirements:
- Simulate delays (1-5 seconds)
- Record actual API calls for debugging
- Run in CI/CD pipeline

**Question:** Which mock tool would you recommend?

**Answer:**
```
Recommendation: WireMock

Justification:
1. Response Delays: Fixed/random/chunked delays supported
   {"fixedDelayMilliseconds": 2000}

2. Recording: Proxy mode records actual traffic
   wiremock --record-mappings --proxy-all="https://payment.api.com"

3. CI/CD: Docker image available
   docker run -d wiremock/wiremock:latest

Why not Postman Mock:
- Fixed delays only (no random)
- No recording capability
- Rate-limited on free tier

Configuration Example:
{
  "request": {
    "method": "POST",
    "url": "/payment/process"
  },
  "response": {
    "status": 200,
    "body": "{\"success\": true}",
    "fixedDelayMilliseconds": 2000
  }
}
```

### Question 3: Security Test Design
**Scenario:** API returns user data including email, phone, and password hash. How would you test for excessive data exposure?

**Answer:**
```javascript
// Test: Response should not include sensitive fields
pm.test("No sensitive data exposed", function() {
    const response = pm.response.json();

    // Should NOT be in response
    pm.expect(response).to.not.have.property("password");
    pm.expect(response).to.not.have.property("password_hash");
    pm.expect(response).to.not.have.property("ssn");

    // Phone should be masked
    if (response.phone) {
        pm.expect(response.phone).to.match(/\*{6}\d{4}/);
    }

    // Email should be partial
    if (response.email) {
        pm.expect(response.email).to.match(/^.{2}\*+@/);
    }
});

// Test: Admin endpoint shouldn't be accessible
pm.test("Admin data not accessible to regular user", function() {
    pm.response.to.have.status(403);
});

// Additional checks:
// - Verify response headers (no server info leaked)
// - Check error messages don't reveal stack traces
// - Ensure IDs are not sequential (prevent enumeration)
```

---

## 7. Cheatsheet / Quick Reference

### Postman Test Syntax

```javascript
// Response checks
pm.response.to.have.status(200);
pm.response.to.have.status("OK");
pm.response.to.be.ok;
pm.response.to.be.json;
pm.response.to.have.header("Content-Type", "application/json");

// JSON checks
const json = pm.response.json();
pm.expect(json.name).to.equal("Test");
pm.expect(json.items).to.be.an("array");
pm.expect(json.count).to.be.greaterThan(0);
pm.expect(json).to.have.property("id");

// Response time
pm.expect(pm.response.responseTime).to.be.below(200);

// Environment variables
pm.environment.set("key", value);
pm.environment.get("key");
pm.globals.set("key", value);
pm.collectionVariables.set("key", value);

// Chained requests
pm.sendRequest(request, function(err, response) {
    // Handle response
});
```

### Newman Commands

```bash
# Basic run
newman run collection.json

# With environment
newman run collection.json -e env.json

# Reporters
newman run collection.json --reporters cli,html,json

# Iterations
newman run collection.json -n 10

# Data file
newman run collection.json -d data.csv

# Bail on error
newman run collection.json --bail

# Delay between requests
newman run collection.json --delay-request 1000

# Export results
newman run collection.json --reporter-html-export report.html
```

### Common HTTP Headers

```
# Request Headers
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
X-Request-ID: uuid

# Response Headers to Check
Content-Type: application/json
X-RateLimit-Remaining: 99
Cache-Control: no-cache
```

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

**Basic Query Test:**
```javascript
// Postman test for GraphQL
const query = `
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
      posts {
        title
      }
    }
  }
`;

pm.sendRequest({
  url: pm.environment.get('graphql_url'),
  method: 'POST',
  header: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer ' + pm.environment.get('token')
  },
  body: {
    mode: 'raw',
    raw: JSON.stringify({
      query: query,
      variables: { id: "123" }
    })
  }
}, (err, res) => {
  pm.test('User query returns data', () => {
    const json = res.json();
    pm.expect(json.data.user).to.not.be.null;
    pm.expect(json.data.user.name).to.be.a('string');
    pm.expect(json.errors).to.be.undefined;
  });
});
```

### 8.3 GraphQL Mutation Testing

```javascript
const mutation = `
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

const variables = {
  input: {
    name: "John Doe",
    email: "john@example.com",
    password: "Pass1234!"
  }
};

// Test mutation
pm.test('Mutation creates user', () => {
  const json = pm.response.json();
  pm.expect(json.data.createUser.id).to.exist;
  pm.expect(json.data.createUser.name).to.eql("John Doe");
});
```

### 8.4 GraphQL Security Testing

**Common Vulnerabilities:**

| Vulnerability | Test Approach |
|--------------|---------------|
| **Introspection** | Query `__schema` - should be disabled in prod |
| **Depth attack** | Send deeply nested queries |
| **Batching attack** | Send multiple queries in one request |
| **Injection** | Test special chars in variables |

**Introspection Test:**
```graphql
# Should fail in production
query IntrospectionQuery {
  __schema {
    types {
      name
    }
  }
}
```

**Depth Attack Test:**
```graphql
# Malicious deep query - should be rejected
query DeepAttack {
  user(id: "1") {
    posts {
      author {
        posts {
          author {
            posts {
              # ... recursively
            }
          }
        }
      }
    }
  }
}
```

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
