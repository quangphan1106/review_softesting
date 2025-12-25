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

| Step | Action | Endpoint | Think Time |
|------|--------|----------|------------|
| 1 | Browse products | GET /products | 3s |
| 2 | View details | GET /products/{id} | 5s |
| 3 | Add to cart | POST /cart | 2s |
| 4 | View cart | GET /cart | 3s |
| 5 | Checkout | POST /checkout | - |

**JMeter Test Plan Structure:**
- Thread Group: Users (100), Ramp-up (60s), Loop (10)
- HTTP Requests: Each step above
- Timers: Constant timer between requests (think time)
- Response Assertion: Verify status codes
- Listeners: Summary Report, Aggregate Report

### 3.2 k6 Script Concepts

**Key Components:**
- `stages`: Define ramp-up/down pattern (duration + target VUs)
- `thresholds`: Pass/fail criteria (e.g., `p(95)<500`, `rate<0.01`)
- `http.get/post`: Make HTTP requests
- `check()`: Validate response
- `sleep()`: Think time between actions

**Example Pattern:**
```
Ramp up (2m → 100 users) → Steady (5m) → Stress (2m → 200) → Ramp down (1m → 0)
```

### 3.3 Locust Script Concepts

**Key Components:**
- `HttpUser`: Define user behavior class
- `wait_time = between(1, 5)`: Random think time
- `on_start()`: Login/setup before tests
- `@task(weight)`: Define task with weight (3 = 3x more likely)
- `self.client.get/post`: Make HTTP calls

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

| Problem | Symptom | Root Causes | Debug Steps |
|---------|---------|-------------|-------------|
| **Response Time Degradation** | P95: 200ms → 2000ms at load | DB pool exhausted, thread bottleneck, GC overhead | Check DB connections, thread count, GC logs |
| **High Error Rate** | 5%+ HTTP 500 at peak | Timeout too low, OOM, deadlocks | Check error logs, increase timeout, monitor resources |
| **JMeter OOM** | OutOfMemoryError | Too many listeners, View Results Tree | Disable listeners, increase heap, run non-GUI mode |

**JMeter Optimization Tips:**
- Run non-GUI: `jmeter -n -t test.jmx`
- Increase heap: `JVM_ARGS="-Xms1g -Xmx4g"`
- Disable View Results Tree during load tests

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

**Test Environment:** ToolShop (https://with-bugs.practicesoftwaretesting.com) using JMeter

| Scenario | Config | Key Results |
|----------|--------|-------------|
| **Load Test** | 100 users, 60s ramp, 5min | Avg: 245ms, P95: 520ms, Errors: 0.5% |
| **Stress Test** | 200 users, 30s ramp | Breaking point: ~180 users, RT: 1.8s, Errors: 3.2% |
| **Spike Test** | 50→200 users in 5s | Peak errors: 8%, Recovery: 45s |

### 5.2 JMeter Test Plan Concepts

**Key Elements:**
- Thread Group: numThreads (VUs), rampUp (seconds), duration
- HTTPSampler: path, method (GET/POST)
- Timer: UniformRandomTimer (range, offset) for think time
- ResponseAssertion: Verify status code

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
**Scenario:** Avg=150ms, P50=120ms, P95=800ms, P99=3500ms, Errors=0.8%

**Question:** Is this acceptable?

**Answer:**
- P50 good, P95 concerning (nearly 1s), P99 problematic (3.5s)
- Large gap P50→P99 = inconsistent performance
- Indicates: resource contention, slow DB queries for some requests
- **NOT acceptable** - investigate P99 spikes, check DB, fix before production

### Question 3: Tool Selection
**Scenario:** Need to test REST API + WebSocket + Database queries

**Question:** Which tool(s)?

**Answer:**
| Option | Tool(s) | Pros | Cons |
|--------|---------|------|------|
| 1 | JMeter only | All 3 supported | Resource-heavy |
| 2 | k6 + JMeter | k6 for API/WS, JMeter for JDBC | 2 tools |

**Recommendation:** k6 for API/WebSocket (CI/CD friendly), JMeter only if need JDBC

---

## 7. Cheatsheet / Quick Reference

### JMeter Quick Reference

| Command/Element | Purpose |
|-----------------|---------|
| `jmeter -n -t test.jmx -l results.jtl` | Run non-GUI mode |
| `jmeter -g results.jtl -o report/` | Generate HTML report |
| `JVM_ARGS="-Xms1g -Xmx4g"` | Increase heap |
| Thread Group | Define virtual users, ramp-up, duration |
| HTTP/JDBC Sampler | Make requests |
| Timer (Constant/Random) | Think time |
| Assertion | Validate response |
| Listener | Collect results (Summary, Aggregate) |

### k6 Quick Reference

| Element | Syntax/Example |
|---------|----------------|
| Run | `k6 run script.js` |
| VUs | `vus: 100` |
| Duration | `duration: '5m'` |
| Stages | `stages: [{duration: '1m', target: 100}]` |
| Threshold | `http_req_duration: ['p(95)<500']` |
| Request | `http.get(url)`, `http.post(url, body)` |
| Check | `check(res, {'ok': r => r.status === 200})` |
| Think time | `sleep(seconds)` |

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

## 8. Artillery & Core Web Vitals (2025 Update)

### 8.1 Artillery

**What is Artillery?**
- Node.js-based load testing, YAML config
- Built-in Playwright support for browser testing
- Cloud-native (AWS/Azure integration)

**Key Concepts:**
- `config.target`: Base URL
- `config.phases`: Load pattern (duration, arrivalRate)
- `scenarios.flow`: User journey steps (get, post, think)

**Commands:** `artillery run test.yml`, `artillery report report.json`

**Artillery vs k6:**

| Aspect | Artillery | k6 |
|--------|-----------|-----|
| Config | YAML + JS | JavaScript |
| Browser | Playwright built-in | Extension needed |
| Learning | Easier (YAML) | More flexible |

### 8.2 Core Web Vitals Testing

**What are Core Web Vitals?**
- Google's UX metrics, affects SEO rankings
- Measured in Chrome, Lighthouse, PageSpeed

**Metrics:**

| Metric | Measures | Good | Poor |
|--------|----------|------|------|
| **LCP** (Largest Contentful Paint) | Loading | <2.5s | >4s |
| **FID** (First Input Delay) | Interactivity | <100ms | >300ms |
| **CLS** (Cumulative Layout Shift) | Stability | <0.1 | >0.25 |
| **INP** (Interaction to Next Paint) | Responsiveness | <200ms | >500ms |

**Testing Methods:**
- Playwright: Use PerformanceObserver API to measure LCP, CLS
- Lighthouse CI: `lhci autorun --collect.url=https://example.com`
- Set assertions in `lighthouserc.js` for CI/CD integration

---

## 9. Key Takeaways for Exam

1. **Test Types**: Know the difference between load, stress, spike, soak
2. **Metrics**: Focus on percentiles (P95, P99), not averages
3. **Tools**: JMeter for versatility, k6 for modern APIs, Locust for Python, Artillery for YAML
4. **Response Time**: Tools measure differently - never compare across tools
5. **Troubleshooting**: Check DB connections, memory, network first
6. **Nielsen's Limits**: 0.1s (instant), 1s (noticeable), 10s (abandon)
7. **Core Web Vitals**: LCP <2.5s, FID <100ms, CLS <0.1, INP <200ms
8. **Artillery**: YAML-based, Playwright support, cloud-native

---

*Study time recommended: 3-4 hours*
*Practice: Set up JMeter or k6 and run a load test against a sample API*
