# Topic 2: Performance Testing - Câu hỏi vận dụng
**CS423 Software Testing | Final Exam Practice**

---

## Câu 1: Test Type Selection (10 điểm)

**Tình huống:** E-commerce website chuẩn bị cho Black Friday:
- Bình thường: 1,000 concurrent users
- Black Friday dự kiến: 10,000 users (10x)
- Flash sale bắt đầu lúc 0:00, kéo dài 24 giờ

**Yêu cầu:**
a) Liệt kê các loại performance test cần thực hiện theo thứ tự (3 điểm)
b) Giải thích mục đích của từng loại test (4 điểm)
c) Xác định metrics cần monitor (3 điểm)

**Đáp án mẫu:**

a) **Thứ tự test:**
1. Load Test
2. Stress Test
3. Spike Test
4. Soak Test
5. Scalability Test

b) **Mục đích từng loại:**

| Test Type | Mục đích | Setup |
|-----------|----------|-------|
| **Load Test** | Verify baseline 1,000 users | 1,000 users, 30 min |
| **Stress Test** | Tìm breaking point | Ramp 1,000 → 15,000 |
| **Spike Test** | Flash sale simulation | 1,000 → 10,000 trong 10s |
| **Soak Test** | Memory leaks over 24h | 10,000 users, 24h |
| **Scalability Test** | Xác định resources cần scale | Gradual increase, monitor |

c) **Metrics cần monitor:**
- Response Time: P50, P90, P95, P99
- Throughput: requests/second
- Error Rate: target < 1%
- CPU Usage: < 70%
- Memory Usage: < 80%
- Database connections: Pool utilization
- Network bandwidth

---

## Câu 2: Metric Interpretation (10 điểm)

**Tình huống:** Test results show:
```
Endpoint: /api/checkout
Users: 500 concurrent
Duration: 10 minutes

Results:
- Average: 150ms
- P50 (Median): 120ms
- P90: 350ms
- P95: 800ms
- P99: 3500ms
- Max: 12000ms
- Error rate: 0.8%
- Throughput: 850 req/s
```

**Yêu cầu:**
a) Phân tích performance dựa trên data (4 điểm)
b) Xác định vấn đề tiềm ẩn (3 điểm)
c) Đề xuất giải pháp cải thiện (3 điểm)

**Đáp án mẫu:**

a) **Phân tích:**
- **P50 (120ms)**: Tốt - 50% users có trải nghiệm tốt
- **P90 (350ms)**: Acceptable - vẫn dưới 1s
- **P95 (800ms)**: Concerning - 5% users chờ gần 1s
- **P99 (3500ms)**: Problematic - 1% users chờ 3.5s (Nielsen: mất focus)
- **Gap P50-P99**: 120ms → 3500ms = 29x difference → Inconsistent performance
- **Error rate 0.8%**: Acceptable (< 1%)
- **Throughput 850 req/s**: Good capacity

b) **Vấn đề tiềm ẩn:**
1. **Tail latency cao**: P99 gấp 29x P50 → có bottleneck
2. **Bimodal distribution**: Một số requests chậm bất thường
3. **Possible causes:**
   - Database slow queries (specific records)
   - Connection pool exhaustion
   - GC pauses
   - External service timeouts
   - Cold cache misses

c) **Giải pháp:**
```
1. Database optimization:
   - EXPLAIN ANALYZE slow queries
   - Add indexes for checkout-related queries
   - Implement query caching

2. Connection pooling:
   - Increase pool size
   - Add connection timeout
   - Monitor pool utilization

3. Application level:
   - Add circuit breaker for external services
   - Implement request timeout (2s max)
   - Add async processing for non-critical operations

4. Caching:
   - Redis cache for product/price data
   - Pre-warm cache before peak hours

5. Monitoring:
   - Add detailed tracing (Jaeger/Zipkin)
   - Monitor P99 specifically
   - Alert when P99 > 2s
```

---

## Câu 3: Tool Selection (10 điểm)

**Tình huống:** Team cần performance test cho:
- REST API endpoints
- WebSocket real-time features
- GraphQL queries
- Database queries directly

Team: 5 developers (3 Python, 2 JavaScript)

**Yêu cầu:**
a) Recommend tool(s) cho từng requirement (4 điểm)
b) So sánh k6 vs JMeter cho trường hợp này (3 điểm)
c) Design test architecture (3 điểm)

**Đáp án mẫu:**

a) **Tool recommendation:**

| Requirement | Recommended Tool | Reason |
|-------------|-----------------|--------|
| REST API | k6 | JS-based, modern, efficient |
| WebSocket | k6 | Native WebSocket support |
| GraphQL | k6 | HTTP-based, easy to implement |
| Database | JMeter | JDBC Sampler built-in |

b) **k6 vs JMeter comparison:**

| Aspect | k6 | JMeter |
|--------|-----|--------|
| Language | JavaScript | Java (GUI) |
| Resource/user | ~1KB | ~1MB (thread-based) |
| Max users/machine | 10,000+ | ~1,000 |
| WebSocket | Native | Plugin required |
| GraphQL | Easy (HTTP) | Possible |
| Database | Not supported | JDBC Sampler |
| CI/CD integration | Excellent | Good |
| Team fit | ✓ (2 JS devs) | Partial |

**Recommendation:** k6 for API/WebSocket + JMeter for Database only

c) **Test architecture:**
```
┌─────────────────────────────────────────────────────────┐
│                    Test Infrastructure                   │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────────────┐      ┌─────────────────┐         │
│   │   k6 Scripts    │      │  JMeter Tests   │         │
│   │  - REST API     │      │  - Database     │         │
│   │  - WebSocket    │      │  - JDBC queries │         │
│   │  - GraphQL      │      └────────┬────────┘         │
│   └────────┬────────┘               │                   │
│            │                        │                   │
│   ┌────────▼────────────────────────▼────────┐         │
│   │              CI/CD Pipeline              │         │
│   │  (GitHub Actions / Jenkins)              │         │
│   └────────────────────┬─────────────────────┘         │
│                        │                                │
│   ┌────────────────────▼─────────────────────┐         │
│   │           Results Aggregation            │         │
│   │  - Grafana Dashboard                     │         │
│   │  - InfluxDB for time-series              │         │
│   │  - Slack alerts                          │         │
│   └──────────────────────────────────────────┘         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Câu 4: k6 Script Design (10 điểm)

**Tình huống:** E-commerce checkout flow:
1. Browse products (GET /products) - think time 3s
2. View product (GET /products/{id}) - think time 5s
3. Add to cart (POST /cart) - think time 2s
4. Checkout (POST /checkout) - no think time

SLO: P95 < 500ms, Error rate < 1%

**Yêu cầu:**
a) Viết k6 script với stages (ramp up, steady, ramp down) (5 điểm)
b) Implement thresholds (2 điểm)
c) Add checks và custom metrics (3 điểm)

**Đáp án mẫu:**

```javascript
import http from 'k6/http';
import { check, sleep, group } from 'k6';
import { Counter, Trend } from 'k6/metrics';

// Custom metrics
const checkoutDuration = new Trend('checkout_duration');
const checkoutErrors = new Counter('checkout_errors');

export const options = {
  // a) Stages: ramp up → steady → stress → ramp down
  stages: [
    { duration: '2m', target: 100 },   // Ramp up to 100 users
    { duration: '5m', target: 100 },   // Steady state
    { duration: '2m', target: 200 },   // Stress test
    { duration: '1m', target: 0 },     // Ramp down
  ],

  // b) Thresholds matching SLO
  thresholds: {
    'http_req_duration': ['p(95)<500'],           // P95 < 500ms
    'http_req_failed': ['rate<0.01'],             // Error rate < 1%
    'checkout_duration': ['p(95)<1000'],          // Checkout P95 < 1s
    'checkout_errors': ['count<10'],              // Max 10 checkout errors
    'checks': ['rate>0.95'],                      // 95% checks pass
  },
};

const BASE_URL = 'https://api.example.com';

export default function () {
  // c) Checks and custom metrics

  group('Browse Products', () => {
    const res = http.get(`${BASE_URL}/products`);

    check(res, {
      'browse status 200': (r) => r.status === 200,
      'browse has products': (r) => r.json().data.length > 0,
      'browse time < 200ms': (r) => r.timings.duration < 200,
    });

    sleep(3); // Think time
  });

  group('View Product', () => {
    const productId = Math.floor(Math.random() * 100) + 1;
    const res = http.get(`${BASE_URL}/products/${productId}`);

    check(res, {
      'product status 200': (r) => r.status === 200,
      'product has details': (r) => r.json().name !== undefined,
    });

    sleep(5); // Think time
  });

  group('Add to Cart', () => {
    const payload = JSON.stringify({
      productId: 123,
      quantity: 1,
    });

    const params = {
      headers: { 'Content-Type': 'application/json' },
    };

    const res = http.post(`${BASE_URL}/cart`, payload, params);

    check(res, {
      'cart status 200': (r) => r.status === 200,
      'cart item added': (r) => r.json().success === true,
    });

    sleep(2); // Think time
  });

  group('Checkout', () => {
    const start = Date.now();

    const payload = JSON.stringify({
      paymentMethod: 'credit_card',
      shippingAddress: '123 Test St',
    });

    const params = {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer test-token',
      },
    };

    const res = http.post(`${BASE_URL}/checkout`, payload, params);

    // Custom metrics
    const duration = Date.now() - start;
    checkoutDuration.add(duration);

    const success = check(res, {
      'checkout status 200': (r) => r.status === 200,
      'checkout confirmed': (r) => r.json().orderId !== undefined,
      'checkout time < 1s': (r) => r.timings.duration < 1000,
    });

    if (!success) {
      checkoutErrors.add(1);
    }

    // No think time after checkout
  });
}

// Summary handler for custom reporting
export function handleSummary(data) {
  return {
    'stdout': textSummary(data, { indent: ' ', enableColors: true }),
    'summary.json': JSON.stringify(data),
    'summary.html': htmlReport(data),
  };
}
```

---

## Câu 5: JMeter Test Plan (10 điểm)

**Tình huống:** Design JMeter test plan cho:
- 200 concurrent users
- Ramp up: 60 seconds
- Duration: 5 minutes
- Endpoints: Login, Product List, Add to Cart

**Yêu cầu:**
a) Vẽ test plan structure (4 điểm)
b) Configure assertions (3 điểm)
c) Design result listeners (3 điểm)

**Đáp án mẫu:**

a) **Test Plan Structure:**
```
Test Plan: E-commerce Load Test
├── User Defined Variables
│   ├── BASE_URL = https://api.example.com
│   ├── USERS = 200
│   └── RAMP_UP = 60
│
├── HTTP Request Defaults
│   ├── Server: ${BASE_URL}
│   └── Content-Type: application/json
│
├── Thread Group (Users: 200, Ramp-up: 60s, Duration: 300s)
│   │
│   ├── HTTP Request: Login
│   │   ├── Path: /auth/login
│   │   ├── Method: POST
│   │   ├── Body: {"email":"test@test.com","password":"pass"}
│   │   └── Response Assertion: Status 200
│   │
│   ├── JSON Extractor: Extract Token
│   │   └── Variable: auth_token
│   │
│   ├── HTTP Header Manager
│   │   └── Authorization: Bearer ${auth_token}
│   │
│   ├── Constant Timer: 2000ms
│   │
│   ├── HTTP Request: Product List
│   │   ├── Path: /products
│   │   ├── Method: GET
│   │   └── Response Assertion: Contains "data"
│   │
│   ├── Uniform Random Timer: 1000-3000ms
│   │
│   └── HTTP Request: Add to Cart
│       ├── Path: /cart
│       ├── Method: POST
│       ├── Body: {"productId": 123, "quantity": 1}
│       └── Response Assertion: Status 200
│
├── Listeners
│   ├── View Results Tree (disable in load test)
│   ├── Summary Report
│   ├── Aggregate Report
│   └── Backend Listener (InfluxDB)
│
└── Assertions
    └── Duration Assertion: < 2000ms
```

b) **Assertions configuration:**
```xml
<!-- Response Assertion -->
<ResponseAssertion>
  <stringProp name="Assertion.test_type">2</stringProp>
  <stringProp name="Assertion.test_string">200</stringProp>
  <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
</ResponseAssertion>

<!-- JSON Assertion -->
<JSONPathAssertion>
  <stringProp name="JSON_PATH">$.data</stringProp>
  <stringProp name="EXPECTED_VALUE"></stringProp>
  <boolProp name="JSONVALIDATION">true</boolProp>
  <boolProp name="EXPECT_NULL">false</boolProp>
</JSONPathAssertion>

<!-- Duration Assertion -->
<DurationAssertion>
  <stringProp name="DurationAssertion.duration">2000</stringProp>
</DurationAssertion>

<!-- Size Assertion -->
<SizeAssertion>
  <stringProp name="Assertion.test_type">4</stringProp>
  <stringProp name="SizeAssertion.size">1000</stringProp>
</SizeAssertion>
```

c) **Result listeners:**
```xml
<!-- Summary Report (lightweight) -->
<SummaryReport>
  <stringProp name="filename">summary.csv</stringProp>
</SummaryReport>

<!-- Aggregate Report -->
<AggregateReport>
  <stringProp name="filename">aggregate.csv</stringProp>
</AggregateReport>

<!-- Backend Listener for real-time monitoring -->
<BackendListener>
  <stringProp name="classname">org.apache.jmeter.visualizers.backend.influxdb.InfluxdbBackendListenerClient</stringProp>
  <elementProp name="arguments">
    <collectionProp name="Arguments.arguments">
      <elementProp name="influxdbUrl">
        <stringProp name="Argument.value">http://influxdb:8086/write?db=jmeter</stringProp>
      </elementProp>
      <elementProp name="application">
        <stringProp name="Argument.value">ecommerce</stringProp>
      </elementProp>
    </collectionProp>
  </elementProp>
</BackendListener>

<!-- Simple Data Writer (for post-analysis) -->
<ResultCollector>
  <stringProp name="filename">results.jtl</stringProp>
</ResultCollector>
```

---

## Câu 6: Locust Test Design (10 điểm)

**Tình huống:** Python team cần load test cho:
- User registration
- Login
- Profile update
- Logout

Weight distribution: Browse (60%), Register (10%), Profile (20%), Logout (10%)

**Yêu cầu:**
a) Viết Locust test class với weights (5 điểm)
b) Add custom events và metrics (3 điểm)
c) Configure distributed testing (2 điểm)

**Đáp án mẫu:**

a) **Locust test với weights:**
```python
from locust import HttpUser, task, between, events
from locust.runners import MasterRunner
import time
import json

class EcommerceUser(HttpUser):
    wait_time = between(1, 5)
    host = "https://api.example.com"

    def on_start(self):
        """Called when user starts - login"""
        response = self.client.post("/auth/login", json={
            "email": f"user_{self.environment.runner.user_count}@test.com",
            "password": "password123"
        })
        if response.status_code == 200:
            self.token = response.json().get("access_token")
            self.headers = {"Authorization": f"Bearer {self.token}"}
        else:
            self.token = None
            self.headers = {}

    def on_stop(self):
        """Called when user stops - logout"""
        if self.token:
            self.client.post("/auth/logout", headers=self.headers)

    @task(6)  # Weight 60%
    def browse_products(self):
        """Browse products - most common action"""
        with self.client.get("/products", headers=self.headers,
                            catch_response=True) as response:
            if response.status_code == 200:
                products = response.json().get("data", [])
                if len(products) > 0:
                    response.success()
                else:
                    response.failure("No products returned")
            else:
                response.failure(f"Status {response.status_code}")

    @task(1)  # Weight 10%
    def register_user(self):
        """Register new user"""
        unique_email = f"newuser_{time.time()}@test.com"
        response = self.client.post("/auth/register", json={
            "email": unique_email,
            "password": "Password123!",
            "firstName": "Test",
            "lastName": "User"
        })
        # Don't fail on 409 (already exists)
        if response.status_code not in [200, 201, 409]:
            response.failure(f"Registration failed: {response.status_code}")

    @task(2)  # Weight 20%
    def update_profile(self):
        """Update user profile"""
        if not self.token:
            return

        with self.client.put("/profile", headers=self.headers,
                            json={"firstName": "Updated"},
                            catch_response=True) as response:
            if response.status_code == 200:
                response.success()
            elif response.status_code == 401:
                # Token expired, refresh
                self.on_start()
            else:
                response.failure(f"Profile update failed: {response.status_code}")

    @task(1)  # Weight 10%
    def view_order_history(self):
        """View order history"""
        if not self.token:
            return

        self.client.get("/orders", headers=self.headers)
```

b) **Custom events và metrics:**
```python
from locust import events
import logging

# Custom metrics
checkout_times = []
failed_checkouts = 0

@events.request.add_listener
def on_request(request_type, name, response_time, response_length,
               response, context, exception, **kwargs):
    """Track custom metrics per request"""
    if name == "/checkout":
        checkout_times.append(response_time)
        if exception or response.status_code != 200:
            global failed_checkouts
            failed_checkouts += 1

@events.test_stop.add_listener
def on_test_stop(environment, **kwargs):
    """Report custom metrics at end of test"""
    if checkout_times:
        avg_checkout = sum(checkout_times) / len(checkout_times)
        p95_checkout = sorted(checkout_times)[int(len(checkout_times) * 0.95)]

        logging.info(f"Checkout Avg: {avg_checkout:.2f}ms")
        logging.info(f"Checkout P95: {p95_checkout:.2f}ms")
        logging.info(f"Failed Checkouts: {failed_checkouts}")

@events.init.add_listener
def on_locust_init(environment, **kwargs):
    """Initialize - runs once at start"""
    if isinstance(environment.runner, MasterRunner):
        logging.info("Running as master node")
    else:
        logging.info("Running as worker node")

# Custom wait time with exponential backoff on failure
class AdaptiveUser(HttpUser):
    def wait_time(self):
        if hasattr(self, 'last_request_failed') and self.last_request_failed:
            return 10  # Wait longer after failure
        return between(1, 5)()
```

c) **Distributed testing config:**
```bash
# Master node
locust -f locustfile.py --master --expect-workers=4

# Worker nodes (run on different machines)
locust -f locustfile.py --worker --master-host=192.168.1.100

# Docker Compose for distributed testing
# docker-compose.yml
version: '3'
services:
  master:
    image: locustio/locust
    ports:
      - "8089:8089"
    volumes:
      - ./:/mnt/locust
    command: -f /mnt/locust/locustfile.py --master -H http://target:8080

  worker:
    image: locustio/locust
    volumes:
      - ./:/mnt/locust
    command: -f /mnt/locust/locustfile.py --worker --master-host master
    deploy:
      replicas: 4  # 4 worker nodes
```

---

## Câu 7: Bottleneck Analysis (10 điểm)

**Tình huống:** Load test results:
```
At 100 users:  P95 = 200ms, CPU = 30%, Memory = 40%
At 200 users:  P95 = 250ms, CPU = 45%, Memory = 50%
At 300 users:  P95 = 400ms, CPU = 55%, Memory = 60%
At 400 users:  P95 = 1500ms, CPU = 60%, Memory = 65%
At 500 users:  P95 = 5000ms, CPU = 62%, Memory = 68%
```

**Yêu cầu:**
a) Phân tích và xác định bottleneck (4 điểm)
b) Đề xuất root cause investigation (3 điểm)
c) Thiết kế giải pháp scale (3 điểm)

**Đáp án mẫu:**

a) **Bottleneck Analysis:**
```
Observation:
- Response time tăng đột biến từ 300→400 users (400ms → 1500ms)
- CPU và Memory tăng linear, không saturate
- CPU chỉ 62% ở 500 users nhưng P95 = 5000ms

Conclusion:
- KHÔNG phải CPU/Memory bottleneck
- Likely bottleneck: Connection-related
  - Database connection pool
  - External service connections
  - Thread pool exhaustion
```

b) **Root cause investigation:**
```python
# 1. Check database connections
"""
SHOW PROCESSLIST;
-- Look for: waiting connections, long-running queries

SHOW VARIABLES LIKE 'max_connections';
SHOW STATUS LIKE 'Threads_connected';
"""

# 2. Check application thread pool
"""
# Java: Thread dump
jstack <pid> > thread_dump.txt
# Look for: BLOCKED, WAITING threads

# Node.js: Check event loop lag
const blocked = require('blocked-at')
blocked((time, stack) => {
  console.log(`Blocked for ${time}ms`)
})
"""

# 3. Monitor connection pool
"""
HikariCP metrics:
- hikaricp.connections.active
- hikaricp.connections.pending
- hikaricp.connections.timeout

If pending > 0 and active = max → Pool exhausted
"""

# 4. External service check
"""
# Check latency to external services
curl -w "@curl-format.txt" https://external-api.com/health

# Verify circuit breaker status
GET /actuator/health/circuitbreakers
"""
```

c) **Scaling solution:**
```
1. Immediate fixes (short-term):
   ┌─────────────────────────────────────────┐
   │ Connection Pool Tuning                   │
   │ - Increase max connections: 10 → 50     │
   │ - Reduce connection timeout: 30s → 10s  │
   │ - Add connection validation             │
   └─────────────────────────────────────────┘

2. Application changes (medium-term):
   ┌─────────────────────────────────────────┐
   │ Async Processing                         │
   │ - Convert sync DB calls to async        │
   │ - Add message queue for heavy tasks     │
   │ - Implement connection pooling properly │
   └─────────────────────────────────────────┘

3. Infrastructure (long-term):
   ┌─────────────────────────────────────────┐
   │ Horizontal Scaling                       │
   │                                          │
   │   Load Balancer                          │
   │        │                                 │
   │   ┌────┴────┬────────┐                  │
   │   │         │        │                  │
   │  App1     App2     App3                 │
   │   │         │        │                  │
   │   └────┬────┴────────┘                  │
   │        │                                 │
   │   Read Replicas ←── Primary DB          │
   │                                          │
   └─────────────────────────────────────────┘
```

---

## Câu 8: SLO Definition (10 điểm)

**Tình huống:** Define SLOs cho e-commerce platform:
- Homepage
- Product Search
- Checkout
- Payment

**Yêu cầu:**
a) Define SLOs cho mỗi endpoint (4 điểm)
b) Calculate error budget (3 điểm)
c) Design alerting strategy (3 điểm)

**Đáp án mẫu:**

a) **SLO Definitions:**

| Endpoint | Availability | Latency (P95) | Error Rate |
|----------|-------------|---------------|------------|
| Homepage | 99.9% | < 200ms | < 0.1% |
| Search | 99.5% | < 500ms | < 0.5% |
| Checkout | 99.95% | < 1000ms | < 0.05% |
| Payment | 99.99% | < 2000ms | < 0.01% |

**Justification:**
- Homepage: High traffic, must be fast, less critical if fails
- Search: Can tolerate some latency, less business impact
- Checkout: Critical path, revenue impact
- Payment: Most critical, money involved

b) **Error Budget Calculation:**
```
Formula: Error Budget = (1 - SLO) × Time Period

For 30-day month (43,200 minutes):

Homepage (99.9%):
- Allowed downtime = 0.1% × 43,200 = 43.2 minutes/month
- Daily budget = 1.44 minutes

Search (99.5%):
- Allowed downtime = 0.5% × 43,200 = 216 minutes/month
- Daily budget = 7.2 minutes

Checkout (99.95%):
- Allowed downtime = 0.05% × 43,200 = 21.6 minutes/month
- Daily budget = 0.72 minutes (43 seconds)

Payment (99.99%):
- Allowed downtime = 0.01% × 43,200 = 4.32 minutes/month
- Daily budget = 0.144 minutes (8.6 seconds)

Error Budget Remaining (example):
┌─────────────────────────────────────────────┐
│ Endpoint   │ Budget │ Used  │ Remaining     │
├─────────────────────────────────────────────┤
│ Homepage   │ 43.2m  │ 12m   │ 31.2m (72%)  │
│ Search     │ 216m   │ 45m   │ 171m (79%)   │
│ Checkout   │ 21.6m  │ 18m   │ 3.6m (17%) ⚠️│
│ Payment    │ 4.32m  │ 1m    │ 3.32m (77%)  │
└─────────────────────────────────────────────┘
```

c) **Alerting Strategy:**
```yaml
# Prometheus alerting rules
groups:
  - name: slo-alerts
    rules:
      # Burn rate alerts (multi-window)
      - alert: HomepageHighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{endpoint="homepage",status=~"5.."}[1h]))
            /
            sum(rate(http_requests_total{endpoint="homepage"}[1h]))
          ) > 0.001 * 14.4  # 14.4x burn rate
        for: 5m
        labels:
          severity: critical
          endpoint: homepage

      - alert: CheckoutLatencyHigh
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{endpoint="checkout"}[5m]))
            by (le)
          ) > 1.0
        for: 5m
        labels:
          severity: critical
          endpoint: checkout

      # Error budget alerts
      - alert: ErrorBudgetLow
        expr: |
          slo_error_budget_remaining_ratio < 0.25
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Error budget below 25% for {{ $labels.endpoint }}"

# Alert routing
route:
  receiver: default
  routes:
    - match:
        severity: critical
        endpoint: payment
      receiver: pagerduty-critical
      repeat_interval: 5m

    - match:
        severity: critical
      receiver: slack-critical
      repeat_interval: 15m

    - match:
        severity: warning
      receiver: slack-warning
      repeat_interval: 1h
```

---

## Câu 9: Test Data Management (10 điểm)

**Tình huống:** Performance test cần:
- 1 million products in database
- 100,000 users
- Realistic order history
- Fresh data cho mỗi test run

**Yêu cầu:**
a) Design data generation strategy (4 điểm)
b) Implement data cleanup (3 điểm)
c) Handle test isolation (3 điểm)

**Đáp án mẫu:**

a) **Data generation strategy:**
```python
from faker import Faker
import asyncio
import asyncpg

fake = Faker()

async def generate_test_data():
    """Generate realistic test data"""
    conn = await asyncpg.connect('postgresql://localhost/testdb')

    # 1. Generate products in batches
    print("Generating 1M products...")
    products = []
    for i in range(1_000_000):
        products.append((
            f"product_{i}",
            fake.catch_phrase(),
            round(fake.pyfloat(min_value=1, max_value=1000), 2),
            fake.random_int(min=0, max=1000),
            fake.random_int(min=1, max=50)  # category_id
        ))

        if len(products) >= 10000:  # Batch insert
            await conn.executemany("""
                INSERT INTO products (sku, name, price, stock, category_id)
                VALUES ($1, $2, $3, $4, $5)
            """, products)
            products = []
            print(f"Inserted {i+1} products")

    # 2. Generate users
    print("Generating 100K users...")
    users = []
    for i in range(100_000):
        users.append((
            fake.email(),
            fake.sha256(),  # password hash
            fake.name(),
            fake.date_of_birth(minimum_age=18, maximum_age=80)
        ))

        if len(users) >= 5000:
            await conn.executemany("""
                INSERT INTO users (email, password_hash, name, dob)
                VALUES ($1, $2, $3, $4)
            """, users)
            users = []

    # 3. Generate order history
    print("Generating order history...")
    for user_id in range(1, 100_001):
        orders_count = fake.random_int(min=0, max=50)
        for _ in range(orders_count):
            await conn.execute("""
                INSERT INTO orders (user_id, product_id, quantity, total, created_at)
                VALUES ($1, $2, $3, $4, $5)
            """,
            user_id,
            fake.random_int(min=1, max=1_000_000),
            fake.random_int(min=1, max=5),
            round(fake.pyfloat(min_value=10, max_value=5000), 2),
            fake.date_time_this_year()
            )

    await conn.close()
    print("Data generation complete!")

asyncio.run(generate_test_data())
```

b) **Data cleanup:**
```python
# cleanup.py
import asyncpg

async def cleanup_test_data():
    """Clean up test data after performance test"""
    conn = await asyncpg.connect('postgresql://localhost/testdb')

    # Option 1: Truncate (fast, resets auto-increment)
    await conn.execute("""
        TRUNCATE orders, users, products CASCADE;
    """)

    # Option 2: Delete with condition (preserve some data)
    await conn.execute("""
        DELETE FROM orders WHERE created_at > NOW() - INTERVAL '1 day';
        DELETE FROM users WHERE email LIKE 'test_%';
        DELETE FROM products WHERE sku LIKE 'product_%';
    """)

    # Option 3: Restore from snapshot
    # pg_restore -d testdb baseline_snapshot.dump

    await conn.close()

# CI/CD integration
"""
# .github/workflows/performance.yml
jobs:
  performance-test:
    steps:
      - name: Restore test database
        run: |
          pg_restore -d testdb baseline.dump

      - name: Generate test data
        run: python generate_data.py

      - name: Run performance tests
        run: k6 run loadtest.js

      - name: Cleanup
        if: always()
        run: python cleanup.py
"""
```

c) **Test isolation:**
```python
# Test isolation strategies

# 1. Separate database per test
class PerformanceTest:
    def setup(self):
        self.db_name = f"perf_test_{uuid.uuid4().hex[:8]}"
        create_database(self.db_name)
        load_schema(self.db_name)
        generate_data(self.db_name)

    def teardown(self):
        drop_database(self.db_name)

# 2. Transaction rollback (for smaller tests)
@pytest.fixture
def db_session():
    session = create_session()
    session.begin()
    yield session
    session.rollback()

# 3. Docker containers per test
# docker-compose.test.yml
"""
services:
  db:
    image: postgres:15
    tmpfs:
      - /var/lib/postgresql/data  # RAM-based for speed
    environment:
      POSTGRES_DB: testdb
"""

# 4. Namespace isolation (Kubernetes)
"""
kubectl create namespace perf-test-${RUN_ID}
helm install testdb postgres-chart -n perf-test-${RUN_ID}
# ... run tests ...
kubectl delete namespace perf-test-${RUN_ID}
"""
```

---

## Câu 10: Performance Test in CI/CD (10 điểm)

**Tình huống:** Integrate performance testing into CI/CD:
- Run on every PR (lightweight)
- Run nightly (comprehensive)
- Block deployment if SLO violated

**Yêu cầu:**
a) Design CI/CD workflow (4 điểm)
b) Define pass/fail criteria (3 điểm)
c) Implement reporting (3 điểm)

**Đáp án mẫu:**

a) **CI/CD Workflow:**
```yaml
# .github/workflows/performance.yml
name: Performance Tests

on:
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'  # Nightly at 2 AM
  workflow_dispatch:
    inputs:
      test_type:
        type: choice
        options: [smoke, load, stress]

jobs:
  # Lightweight test on every PR
  smoke-test:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Start test environment
        run: docker-compose up -d

      - name: Wait for services
        run: ./scripts/wait-for-healthy.sh

      - name: Run smoke performance test
        run: |
          k6 run --vus 10 --duration 1m \
            --out json=results.json \
            tests/smoke.js

      - name: Check thresholds
        run: |
          node scripts/check-thresholds.js results.json \
            --p95 500 \
            --error-rate 1

      - name: Upload results
        uses: actions/upload-artifact@v4
        with:
          name: smoke-results
          path: results.json

  # Comprehensive nightly test
  load-test:
    if: github.event_name == 'schedule' || github.event.inputs.test_type == 'load'
    runs-on: ubuntu-latest
    environment: performance
    steps:
      - uses: actions/checkout@v4

      - name: Setup k6
        run: |
          sudo apt-get install -y gnupg
          curl -s https://dl.k6.io/key.gpg | sudo apt-key add -
          echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install -y k6

      - name: Run load test
        run: |
          k6 run --vus 100 --duration 10m \
            --out influxdb=http://influxdb:8086/k6 \
            --out json=results.json \
            tests/load.js

      - name: Generate report
        run: node scripts/generate-report.js results.json > report.html

      - name: Check SLO compliance
        id: slo-check
        run: |
          PASS=$(node scripts/check-slo.js results.json)
          echo "slo_passed=$PASS" >> $GITHUB_OUTPUT

      - name: Block deployment if SLO failed
        if: steps.slo-check.outputs.slo_passed == 'false'
        run: |
          echo "::error::SLO violation detected! Deployment blocked."
          exit 1

      - name: Notify on failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {"text": "⚠️ Performance test failed! SLO violated."}
```

b) **Pass/Fail Criteria:**
```javascript
// scripts/check-slo.js
const fs = require('fs');

const results = JSON.parse(fs.readFileSync(process.argv[2]));

const SLO = {
  http_req_duration: {
    p95: 500,      // ms
    p99: 1000,     // ms
  },
  http_req_failed: {
    rate: 0.01,    // 1%
  },
  checks: {
    rate: 0.95,    // 95% checks pass
  },
  throughput: {
    min: 100,      // requests/second
  }
};

let passed = true;
const violations = [];

// Check latency
if (results.metrics.http_req_duration.p95 > SLO.http_req_duration.p95) {
  violations.push(`P95 latency ${results.metrics.http_req_duration.p95}ms > ${SLO.http_req_duration.p95}ms`);
  passed = false;
}

// Check error rate
if (results.metrics.http_req_failed.rate > SLO.http_req_failed.rate) {
  violations.push(`Error rate ${(results.metrics.http_req_failed.rate * 100).toFixed(2)}% > 1%`);
  passed = false;
}

// Check throughput
const throughput = results.metrics.http_reqs.count / results.state.testRunDurationMs * 1000;
if (throughput < SLO.throughput.min) {
  violations.push(`Throughput ${throughput.toFixed(1)} req/s < ${SLO.throughput.min} req/s`);
  passed = false;
}

if (!passed) {
  console.error('SLO Violations:');
  violations.forEach(v => console.error(`  - ${v}`));
}

console.log(passed ? 'true' : 'false');
process.exit(passed ? 0 : 1);
```

c) **Reporting:**
```javascript
// scripts/generate-report.js
const fs = require('fs');

const results = JSON.parse(fs.readFileSync(process.argv[2]));

const report = `
<!DOCTYPE html>
<html>
<head>
  <title>Performance Test Report</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 20px; }
    .metric { display: inline-block; margin: 10px; padding: 20px; border: 1px solid #ccc; border-radius: 8px; }
    .metric.pass { border-color: green; background: #e8f5e9; }
    .metric.fail { border-color: red; background: #ffebee; }
    .value { font-size: 24px; font-weight: bold; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
    th { background: #f5f5f5; }
  </style>
</head>
<body>
  <h1>Performance Test Report</h1>
  <p>Date: ${new Date().toISOString()}</p>
  <p>Duration: ${(results.state.testRunDurationMs / 1000 / 60).toFixed(1)} minutes</p>

  <h2>Key Metrics</h2>
  <div class="metric ${results.metrics.http_req_duration.p95 < 500 ? 'pass' : 'fail'}">
    <div>P95 Latency</div>
    <div class="value">${results.metrics.http_req_duration.p95.toFixed(0)}ms</div>
    <div>Target: < 500ms</div>
  </div>

  <div class="metric ${results.metrics.http_req_failed.rate < 0.01 ? 'pass' : 'fail'}">
    <div>Error Rate</div>
    <div class="value">${(results.metrics.http_req_failed.rate * 100).toFixed(2)}%</div>
    <div>Target: < 1%</div>
  </div>

  <h2>Detailed Results</h2>
  <table>
    <tr><th>Metric</th><th>Min</th><th>Avg</th><th>P50</th><th>P90</th><th>P95</th><th>P99</th><th>Max</th></tr>
    <tr>
      <td>Response Time (ms)</td>
      <td>${results.metrics.http_req_duration.min.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.avg.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.med.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.p90.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.p95.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.p99.toFixed(0)}</td>
      <td>${results.metrics.http_req_duration.max.toFixed(0)}</td>
    </tr>
  </table>

  <h2>Throughput</h2>
  <p>Total requests: ${results.metrics.http_reqs.count}</p>
  <p>Requests/second: ${(results.metrics.http_reqs.count / results.state.testRunDurationMs * 1000).toFixed(1)}</p>

</body>
</html>
`;

console.log(report);
```

---

*Tổng: 10 câu × 10 điểm = 100 điểm*
*Thời gian làm bài đề xuất: 120 phút*
