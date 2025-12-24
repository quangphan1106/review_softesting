# Topic 2: Performance Testing - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 What is Performance Testing?

**Definition:** Non-functional testing measuring how a system performs under workload.

**Goals:**
- Validate responsiveness under expected load
- Identify bottlenecks before production
- Ensure scalability for growth
- Verify stability under stress conditions

**NOT about finding bugs** - about ensuring performance expectations are met.

### 1.2 Types of Performance Testing

| Type | Purpose | Load Pattern | Key Question |
|------|---------|--------------|--------------|
| **Load Testing** | Normal behavior | Expected users | "Does it work under normal load?" |
| **Stress Testing** | Breaking point | Beyond limits | "When does it break?" |
| **Spike Testing** | Sudden surges | Rapid increase | "How does it handle flash sales?" |
| **Soak/Endurance** | Long-term stability | Steady over time | "Memory leaks after 24 hours?" |
| **Volume Testing** | Large data sets | High data volume | "Works with 100M rows?" |
| **Scalability Testing** | Resource scaling | Gradual increase | "Does adding servers help?" |

### 1.3 Key Performance Metrics

| Metric | Definition | Target Example |
|--------|------------|----------------|
| **Response Time** | Time from request to response completion | < 200ms |
| **Throughput** | Requests per second (RPS) | > 1000 RPS |
| **Latency** | Delay before response begins | < 50ms |
| **Error Rate** | % of failed requests | < 1% |
| **Concurrency** | Simultaneous users/sessions | 500 users |
| **CPU Usage** | Processor utilization | < 70% |
| **Memory Usage** | RAM consumption | < 80% |
| **Bandwidth** | Data transferred per second | 100 Mbps |

### 1.4 Response Time Analysis

**Percentiles (Critical for Exam):**
```
- P50 (Median): 50% of requests faster than this
- P90: 90% of requests faster (important for SLAs)
- P95: 95% of requests faster
- P99: 99% of requests faster (tail latency)
- Max: Slowest request (may be outlier)
```

**Example Interpretation:**
```
P50: 100ms  → Half of users experience < 100ms
P90: 250ms  → 10% of users wait > 250ms
P99: 1200ms → 1% of users wait > 1.2s (may indicate issues)
```

**Nielsen's Response Time Limits:**
- 0.1s: Feels instantaneous
- 1.0s: User notices but flow uninterrupted
- 10s: User loses attention

---

## 2. Tools & Techniques Comparison

### 2.1 Performance Testing Tools Comparison

| Feature | JMeter | k6 | Gatling | Locust |
|---------|--------|----|---------| -------|
| **Language** | Java (GUI) | JavaScript | Scala | Python |
| **Architecture** | Thread-based | Event-based | Akka actors | Event-based |
| **Resource Usage** | High (1 thread/user) | Low | Medium | Low |
| **Max Users/Machine** | ~1000 | ~10,000+ | ~5000 | ~5000+ |
| **Protocol Support** | HTTP, JDBC, LDAP, FTP, JMS | HTTP, WebSocket, gRPC | HTTP, WebSocket | HTTP, custom |
| **CI/CD Integration** | Good | Excellent | Excellent | Good |
| **Cloud Option** | BlazeMeter | k6 Cloud | Gatling Enterprise | Locust.io |
| **Learning Curve** | Low (GUI) | Medium (code) | High (Scala) | Low (Python) |
| **Best For** | Legacy, enterprise | Modern APIs | High throughput | Python teams |

### 2.2 Tool-Specific Strengths

**Apache JMeter:**
- 20+ protocol support (most versatile)
- GUI for test design (no coding required)
- Large community, extensive plugins
- JDBC support for database testing
- Weakness: Resource-heavy, complex for CI/CD

**k6 (Grafana):**
- JavaScript-based (developer-friendly)
- Minimal resource footprint
- 20+ integrations (Grafana, Datadog, etc.)
- Checks and thresholds for SLO validation
- Weakness: HTTP-focused, less protocol diversity

**Gatling:**
- Scala DSL (expressive test scenarios)
- Efficient Akka architecture
- Beautiful HTML reports
- Excellent for high-throughput scenarios
- Weakness: Steeper learning curve

**Locust:**
- Pure Python (easy customization)
- Distributed testing with workers
- Real-time web UI
- Code-based flexibility
- Weakness: HTTP-only out of box

### 2.3 Response Time Measurement Differences

**⚠️ CRITICAL FOR EXAM:**
```
Tool comparison test results (same endpoint):
- JMeter:  P50=45ms, P95=120ms
- k6:      P50=42ms, P95=110ms  (under-reports unless combined metrics)
- Gatling: P50=48ms, P95=125ms
- Locust:  P50=85ms, P95=280ms  (significantly higher - TCP handling)
```

**Why Differences?**
- Different timing measurement points
- TCP/TLS handling variations
- Connection pooling strategies
- Warm-up handling

**Best Practice:** Never compare raw numbers across tools. Benchmark within same tool.

---

## 3. Practical Test Case Design

### 3.1 Load Test Scenario Design

**Scenario: E-commerce Checkout Flow**
```
User Journey:
1. Browse products (GET /products) - 3 seconds think time
2. View product details (GET /products/{id}) - 5 seconds
3. Add to cart (POST /cart) - 2 seconds
4. View cart (GET /cart) - 3 seconds
5. Checkout (POST /checkout) - varies
```

**JMeter Test Plan Structure:**
```
Test Plan
├── Thread Group (Users: 100, Ramp-up: 60s, Loop: 10)
│   ├── HTTP Request: Browse Products
│   ├── Constant Timer: 3000ms
│   ├── HTTP Request: Product Details
│   ├── Constant Timer: 5000ms
│   ├── HTTP Request: Add to Cart
│   ├── Constant Timer: 2000ms
│   ├── HTTP Request: View Cart
│   ├── Constant Timer: 3000ms
│   ├── HTTP Request: Checkout
│   └── Response Assertion
├── View Results Tree
├── Summary Report
└── Aggregate Report
```

### 3.2 k6 Script Example

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Steady state
    { duration: '2m', target: 200 },  // Stress
    { duration: '1m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% under 500ms
    http_req_failed: ['rate<0.01'],    // Error rate < 1%
  },
};

export default function () {
  // Browse products
  let res = http.get('https://api.example.com/products');
  check(res, { 'browse status 200': (r) => r.status === 200 });
  sleep(3);

  // View details
  res = http.get('https://api.example.com/products/123');
  check(res, { 'details status 200': (r) => r.status === 200 });
  sleep(5);

  // Add to cart
  res = http.post('https://api.example.com/cart', JSON.stringify({
    productId: 123,
    quantity: 1
  }), { headers: { 'Content-Type': 'application/json' } });
  check(res, { 'cart status 200': (r) => r.status === 200 });
  sleep(2);
}
```

### 3.3 Locust Script Example

```python
from locust import HttpUser, task, between

class ShopUser(HttpUser):
    wait_time = between(1, 5)  # Random wait 1-5 seconds

    def on_start(self):
        """Login on start"""
        self.client.post("/login", {
            "email": "test@example.com",
            "password": "welcome01"
        })

    @task(3)  # Weight: 3x more likely
    def browse_products(self):
        self.client.get("/products")

    @task(2)
    def view_product(self):
        self.client.get("/products/123")

    @task(1)
    def checkout(self):
        self.client.post("/cart", json={"productId": 123})
        self.client.post("/checkout")
```

### 3.4 Test Cases for Performance Testing

**TC-PERF-001: Load Test - Normal Load**
- **Objective**: Verify system handles expected load
- **Setup**: 100 concurrent users, 5-minute duration
- **Metrics**: Response time P95 < 500ms, Error rate < 1%
- **Pass Criteria**: All thresholds met

**TC-PERF-002: Stress Test - Breaking Point**
- **Objective**: Find system limits
- **Setup**: Ramp from 100 to 1000 users, 10% increment/minute
- **Metrics**: Track when response time > 2s or errors > 5%
- **Pass Criteria**: Document breaking point

**TC-PERF-003: Spike Test - Flash Sale**
- **Objective**: Verify sudden load handling
- **Setup**: 50 → 500 users in 10 seconds, hold 2 minutes
- **Metrics**: Recovery time, error rate during spike
- **Pass Criteria**: System recovers within 30 seconds

**TC-PERF-004: Soak Test - Memory Leaks**
- **Objective**: Detect memory leaks
- **Setup**: 100 users, 24-hour duration
- **Metrics**: Memory trend, response time degradation
- **Pass Criteria**: No memory growth > 10%

---

## 4. Troubleshooting Scenarios

### 4.1 Common Performance Issues

**Problem 1: Response Time Degradation Under Load**
```
Symptom: P95 increases from 200ms to 2000ms at 100 users
Root Causes:
- Database connection pool exhausted
- Single-threaded bottleneck
- Memory pressure (GC overhead)
- Network bandwidth saturation

Debug Steps:
1. Monitor DB connections: SHOW PROCESSLIST
2. Check thread count: jstack <pid>
3. Analyze GC logs: -XX:+PrintGCDetails
4. Monitor network: netstat -an | grep ESTABLISHED
```

**Problem 2: High Error Rate**
```
Symptom: 5%+ HTTP 500 errors at peak load
Root Causes:
- Timeout configuration too low
- Out of memory errors
- Database deadlocks
- External service failures

Debug Steps:
1. Check error logs for stack traces
2. Increase timeout temporarily
3. Monitor resource usage (CPU, memory)
4. Add circuit breakers for external calls
```

**Problem 3: JMeter Memory Issues**
```
Symptom: JMeter crashes with OutOfMemoryError
Root Causes:
- Too many listeners collecting results
- View Results Tree enabled
- Insufficient heap allocation

Solutions:
1. Disable View Results Tree in load test
2. Increase heap: JVM_ARGS="-Xms1g -Xmx4g"
3. Use Simple Data Writer instead of listeners
4. Run in non-GUI mode: jmeter -n -t test.jmx
```

### 4.2 Metric Interpretation

**Reading JMeter Summary Report:**
```
Label       | # Samples | Average | Min | Max  | Std Dev | Error% | Throughput
------------|-----------|---------|-----|------|---------|--------|------------
Homepage    | 1000      | 125     | 45  | 890  | 112     | 0.10%  | 85.2/sec
Product API | 1000      | 89      | 32  | 456  | 78      | 0.20%  | 95.4/sec
Checkout    | 500       | 345     | 120 | 2340 | 289     | 1.20%  | 42.1/sec
```

**Analysis:**
- Homepage: Good performance, low variance
- Product API: Excellent, consistent
- Checkout: **Problem!** High max (2340ms), high std dev, 1.2% errors

---

## 5. Toolshop Homework Connection

### 5.1 From Performance Testing Homework (HW07)

**Test Environment:**
- Application: ToolShop Practice Software Testing
- URL: https://with-bugs.practicesoftwaretesting.com
- Tool: Apache JMeter

**Test Scenarios Executed:**

**Scenario 1: Load Test (100 Users)**
```
Thread Group: 100 users
Ramp-up: 60 seconds
Duration: 5 minutes
Target: Homepage, Product List, Product Detail

Results:
- Average Response Time: 245ms
- Throughput: 156.3 requests/sec
- Error Rate: 0.5%
- P95: 520ms
```

**Scenario 2: Stress Test (200 Users)**
```
Thread Group: 200 users
Ramp-up: 30 seconds
Duration: 3 minutes

Results:
- Breaking point at ~180 users
- Response time degraded to 1.8s
- Error rate increased to 3.2%
```

**Scenario 3: Spike Test**
```
Pattern: 50 → 200 users in 5 seconds
Hold: 2 minutes
Recovery: 1 minute

Results:
- Peak error rate: 8% during spike
- Recovery time: 45 seconds
- System stabilized at 200 users
```

### 5.2 JMeter Test Plan from Homework

```xml
<!-- Simplified representation -->
<TestPlan>
  <ThreadGroup>
    <numThreads>200</numThreads>
    <rampUp>30</rampUp>
    <duration>180</duration>
  </ThreadGroup>

  <HTTPSampler>
    <path>/products</path>
    <method>GET</method>
  </HTTPSampler>

  <UniformRandomTimer>
    <range>2000</range>
    <offset>1000</offset>
  </UniformRandomTimer>

  <ResponseAssertion>
    <testString>200</testString>
  </ResponseAssertion>
</TestPlan>
```

---

## 6. Application-Level Exam Questions

### Question 1: Choosing Test Type
**Scenario:** An e-commerce site expects 10,000 users during Black Friday (10x normal traffic). The system currently handles 1,000 users.

**Question:** Which performance test types should be performed and in what order?

**Answer:**
1. **Load Test** (first): Verify current baseline at 1,000 users
2. **Stress Test**: Find breaking point between 1,000-10,000
3. **Spike Test**: Simulate sudden 10x increase at sale start
4. **Scalability Test**: Determine resources needed for 10,000 users
5. **Soak Test**: Ensure stability over Black Friday duration (24h)

**Justification:**
- Load test establishes baseline for comparison
- Stress test identifies where to focus optimization
- Spike test validates auto-scaling configuration
- Scalability test informs infrastructure planning
- Soak test catches memory leaks under sustained load

### Question 2: Metric Interpretation
**Scenario:** Test results show:
- Average: 150ms
- P50: 120ms
- P95: 800ms
- P99: 3500ms
- Error rate: 0.8%

**Question:** Is this performance acceptable? What does it indicate?

**Answer:**
```
Analysis:
- P50 (120ms) is good - half of users have good experience
- P95 (800ms) is concerning - 5% wait nearly 1 second
- P99 (3500ms) is problematic - 1% wait 3.5 seconds
- Large gap between average and P99 indicates:
  - Inconsistent performance
  - Possible resource contention
  - Database slow queries for certain requests

Recommendation:
- Investigate specific requests causing P99 spikes
- Check database query performance
- Review connection pooling
- NOT acceptable for production without fixes
```

### Question 3: Tool Selection
**Scenario:** Team needs to performance test:
- REST API endpoints
- WebSocket real-time features
- Database queries directly

**Question:** Which tool(s) would you recommend?

**Answer:**
```
Option 1: JMeter (comprehensive)
- Pros: Supports all three (HTTP, WebSocket plugin, JDBC)
- Cons: Resource-heavy, complex setup

Option 2: k6 + JMeter hybrid
- k6: REST API + WebSocket (efficient, modern)
- JMeter: Database testing (JDBC sampler)
- Pros: Best of both worlds
- Cons: Two tools to maintain

Recommendation: k6 for API/WebSocket (CI/CD friendly),
JMeter only for database testing if needed.
```

---

## 7. Cheatsheet / Quick Reference

### JMeter Quick Reference

```
# Non-GUI mode (production tests)
jmeter -n -t test.jmx -l results.jtl

# Generate HTML report
jmeter -g results.jtl -o report/

# Increase heap
export JVM_ARGS="-Xms1g -Xmx4g"

# Key Elements:
- Thread Group: Virtual users
- Sampler: HTTP Request, JDBC Request
- Timer: Constant, Gaussian, Uniform Random
- Assertion: Response, JSON, Duration
- Listener: Summary Report, Aggregate Report
```

### k6 Quick Reference

```javascript
// Run test
// k6 run script.js

// Key options
export const options = {
  vus: 100,                    // Virtual users
  duration: '5m',              // Test duration
  stages: [                    // Ramping pattern
    { duration: '1m', target: 100 },
    { duration: '3m', target: 100 },
    { duration: '1m', target: 0 },
  ],
  thresholds: {                // Pass/fail criteria
    http_req_duration: ['p(95)<500'],
    http_req_failed: ['rate<0.01'],
  },
};

// Key functions
import http from 'k6/http';
import { check, sleep } from 'k6';

http.get(url);
http.post(url, body, params);
check(response, { 'status 200': (r) => r.status === 200 });
sleep(seconds);
```

### Performance Testing Checklist

- [ ] Define SLOs (P95 < 500ms, error < 1%)
- [ ] Set up realistic test data
- [ ] Configure think times
- [ ] Run baseline load test
- [ ] Execute stress test to find limits
- [ ] Perform spike test for surge handling
- [ ] Run soak test for memory leaks
- [ ] Analyze results (percentiles, not averages)
- [ ] Document findings and recommendations

### Key Formulas

```
Throughput = Requests / Time
Concurrent Users = Throughput × Response Time
Little's Law: L = λW (Users = Arrival Rate × Wait Time)

Example:
- Target: 1000 req/sec
- Avg response: 200ms
- Required concurrent users: 1000 × 0.2 = 200
```

---

## 8. Key Takeaways for Exam

1. **Test Types**: Know the difference between load, stress, spike, soak
2. **Metrics**: Focus on percentiles (P95, P99), not averages
3. **Tools**: JMeter for versatility, k6 for modern APIs, Locust for Python
4. **Response Time**: Tools measure differently - never compare across tools
5. **Troubleshooting**: Check DB connections, memory, network first
6. **Nielsen's Limits**: 0.1s (instant), 1s (noticeable), 10s (abandon)
7. **Toolshop**: Know the specific results from HW07 experiments

---

*Study time recommended: 3-4 hours*
*Practice: Set up JMeter or k6 and run a load test against a sample API*
