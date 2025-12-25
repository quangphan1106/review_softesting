# Performance Testing - Concept Questions

## Question 1: Performance Test Types Selection (15 points)

### Scenario
ToolShop e-commerce platform has the following requirements:
- Normal traffic: 500 concurrent users
- Black Friday peak: 5,000 concurrent users (10x normal)
- API response time SLA: P95 < 500ms
- System must run 24/7 with 99.9% uptime

### Questions

**a) (5 points)** Define FOUR types of performance tests and when each should be used.

**Expected Answer:**

| Test Type | Purpose | When to Use |
|-----------|---------|-------------|
| **Load Testing** | Verify system handles expected load | Before releases, validate SLAs under normal conditions |
| **Stress Testing** | Find breaking point, test beyond expected load | Capacity planning, identify failure modes |
| **Spike Testing** | Test sudden traffic surges | Flash sale scenarios, viral marketing events |
| **Soak/Endurance Testing** | Find issues over extended time | Memory leaks, resource exhaustion, 24/7 systems |

**b) (5 points)** For ToolShop's Black Friday scenario, which test type is MOST appropriate and why?

**Expected Answer:**

**Spike Testing** is most appropriate because:
- Black Friday creates sudden 10x traffic surge (500 → 5,000 users)
- Need to verify system can handle rapid scale-up
- Tests auto-scaling response time
- Validates graceful degradation under extreme load

**Test design:**
```
Users
5000 │         ┌─────────┐
     │         │  Peak   │
     │        /│  Period │\
 500 │───────/ │         │ \───────
     │      ↑  │         │  ↑
     └──────┴──┴─────────┴──┴────── Time
        Ramp    Hold (30m)   Ramp
        Up                   Down
```

**c) (5 points)** Explain the difference between Soak Testing and Stress Testing with examples.

**Expected Answer:**

| Aspect | Stress Testing | Soak Testing |
|--------|---------------|--------------|
| **Goal** | Find breaking point | Find time-based issues |
| **Duration** | Short (minutes) | Long (hours/days) |
| **Load pattern** | Increasing until failure | Constant at expected load |
| **Finds** | Max capacity, failure modes | Memory leaks, resource exhaustion |
| **Example** | Ramp from 500 to 10,000 users until errors | Run 500 users for 72 hours continuously |

**Soak Test Issues Found:**
- Memory leaks: RAM usage grows over time
- Connection pool exhaustion: DB connections not released
- Log file growth: Disk space fills up
- Certificate expiration: Certs expire mid-operation

---

## Question 2: Performance Metrics Analysis (20 points)

### Scenario
After running a load test on ToolShop's checkout API, you get these results:

| Metric | Value |
|--------|-------|
| Total Requests | 100,000 |
| Failed Requests | 500 |
| Min Response Time | 45ms |
| Max Response Time | 15,200ms |
| Average Response Time | 320ms |
| P50 (Median) | 180ms |
| P90 | 450ms |
| P95 | 890ms |
| P99 | 3,200ms |

### Questions

**a) (5 points)** Explain what P50, P90, P95, and P99 percentiles mean and why they're more useful than average.

**Expected Answer:**

**Percentile meanings:**
- **P50 (Median)**: 50% of requests faster than this value
- **P90**: 90% of requests faster than this value
- **P95**: 95% of requests faster than this value
- **P99**: 99% of requests faster than this value

**Why more useful than average:**
- Average hides outliers (320ms avg but P99 is 3,200ms!)
- Real user experience varies widely
- SLAs should use percentiles, not averages
- Identifies tail latency problems affecting subset of users

**Visual representation:**
```
Response Time Distribution:

Fast │█████████████████████████     │ 50% of users
     │                         ████ │ 40% of users (50-90th)
     │                             █│ 5% of users (90-95th)
     │                              █ 4% of users (95-99th)
Slow │                               ░ 1% of users (99th+)
     └──────────────────────────────────────
       P50    P90   P95       P99    Max
       180ms  450ms 890ms   3200ms  15200ms
```

**b) (5 points)** Based on the metrics, does the system meet the SLA (P95 < 500ms)? Explain your analysis.

**Expected Answer:**

**No, the system FAILS the SLA.**

| SLA Requirement | Actual Value | Status |
|-----------------|--------------|--------|
| P95 < 500ms | 890ms | ❌ FAIL |

**Analysis:**
- P95 is 890ms, which is 78% above the 500ms target
- Even P90 (450ms) is borderline
- Gap between P90 and P95 suggests performance cliff
- Error rate: 0.5% (500/100,000) - acceptable but needs investigation
- Max response (15,200ms) indicates severe outliers

**Root cause hypotheses:**
1. Database query timeout at scale
2. Thread pool exhaustion
3. External service dependency slowdown
4. Garbage collection pauses

**c) (5 points)** Calculate the error rate and throughput from the data. Are these acceptable?

**Expected Answer:**

**Calculations:**

| Metric | Formula | Value |
|--------|---------|-------|
| Error Rate | Failed / Total × 100% | 500 / 100,000 × 100% = **0.5%** |
| Throughput | Depends on test duration | Need test duration to calculate |

**Acceptability assessment:**

Error rate of 0.5%:
- ⚠️ Borderline for e-commerce checkout
- Industry standard: <0.1% for critical paths
- 500 failed checkouts = lost revenue + customer frustration
- Needs investigation: Are errors consistent or spiking?

**Error pattern analysis needed:**
```
If errors distributed evenly → Systemic issue
If errors clustered at high load → Capacity issue
If errors at start only → Warmup issue
If errors random → Flaky dependency
```

**d) (5 points)** What is "throughput" and how does it relate to response time? Describe the typical relationship.

**Expected Answer:**

**Throughput**: Number of requests processed per unit time (e.g., requests/second, transactions/minute)

**Relationship with response time:**

```
                    Throughput vs Response Time

Response ▲
Time     │                          ╱
         │                        ╱
         │                     ╱
         │                  ╱
         │            ╱───╱
         │     ╱─────╱
         │────╱
         └──────────────────────────▶ Throughput
              A        B        C

A = Optimal zone (linear scaling)
B = Degradation zone (response time rising)
C = Saturation zone (throughput drops, response time spikes)
```

**Key concepts:**
| Zone | Behavior | Cause |
|------|----------|-------|
| **Linear** | Throughput ↑, response time stable | System has headroom |
| **Degradation** | Throughput ↑, response time ↑ | Resources getting constrained |
| **Saturation** | Throughput plateaus/drops, response time spikes | System overwhelmed |

**Little's Law**: Concurrent Users = Throughput × Response Time

---

## Question 3: Performance Testing Tools Comparison (15 points)

### Scenario
Your team needs to select a performance testing tool. Options:
- k6 (JavaScript-based, developer-friendly)
- JMeter (GUI-based, enterprise standard)
- Gatling (Scala-based, CI/CD friendly)

### Questions

**a) (5 points)** Compare k6 and JMeter across FOUR criteria relevant to ToolShop's needs.

**Expected Answer:**

| Criteria | k6 | JMeter |
|----------|-----|--------|
| **Scripting** | JavaScript (developer-friendly) | XML/GUI (tester-friendly) |
| **CI/CD Integration** | Excellent (CLI-first, lightweight) | Moderate (requires setup, heavy) |
| **Protocol Support** | HTTP/WebSocket/gRPC | HTTP, JDBC, JMS, SOAP, more |
| **Resource Usage** | Low (Go-based, efficient) | High (Java-based, memory hungry) |
| **Learning Curve** | Easy for developers | Steeper, GUI complex |
| **Distributed Testing** | k6 Cloud or manual | Built-in distributed mode |
| **Community** | Growing, modern | Mature, extensive plugins |

**b) (5 points)** What factors should influence the tool selection for ToolShop?

**Expected Answer:**

| Factor | Consideration | Recommendation |
|--------|---------------|----------------|
| **Team skills** | Developers know JavaScript | Favors k6 |
| **Test types needed** | HTTP APIs, browser simulation | Both can handle |
| **CI/CD integration** | GitHub Actions pipeline | Favors k6 (lighter) |
| **Protocol requirements** | REST APIs, WebSocket for real-time | Both support |
| **Budget** | Open source preferred | Both free (k6 Cloud optional) |
| **Existing infrastructure** | No existing setup | Either viable |
| **Test complexity** | Data-driven, scenarios | k6 flexible scripting |
| **Reporting needs** | Integrate with dashboards | k6 outputs to various formats |

**Recommendation**: k6 for ToolShop because:
- Developer-centric team
- Modern JavaScript approach
- Excellent CI/CD integration
- Lower resource footprint

**c) (5 points)** Explain what "virtual users" means in performance testing and how it differs from "requests per second".

**Expected Answer:**

**Virtual Users (VUs):**
- Simulated users executing test scenarios
- Each VU maintains state (cookies, session, context)
- VU count = concurrent active users

**Requests Per Second (RPS):**
- Raw throughput measurement
- Number of HTTP requests completed per second
- Doesn't account for user behavior/think time

**Key differences:**

| Aspect | Virtual Users | Requests Per Second |
|--------|--------------|---------------------|
| **Represents** | Concurrent users | System throughput |
| **Includes think time** | Yes | No |
| **Session state** | Maintained | Not relevant |
| **Real-world accuracy** | Higher | Lower |
| **Resource correlation** | User behavior patterns | Raw capacity |

**Relationship:**
```
RPS = VUs × (Requests per iteration / (Iteration time + Think time))

Example:
- 100 VUs
- 5 requests per shopping flow
- 10 seconds per flow + 2 seconds think time
- RPS = 100 × (5 / 12) = ~42 RPS
```

---

## Question 4: Performance Test Design (15 points)

### Scenario
Design a performance test for ToolShop's checkout flow:
1. Browse products (2-3 pages)
2. Add item to cart
3. View cart
4. Enter shipping info
5. Process payment
6. Confirmation page

### Questions

**a) (5 points)** Define a realistic test scenario with appropriate pacing and think times.

**Expected Answer:**

**User Journey with Timing:**

| Step | Action | Think Time | Rationale |
|------|--------|------------|-----------|
| 1 | Browse home page | 3-5s | User scans products |
| 2 | Browse category page | 5-10s | User compares items |
| 3 | View product detail | 10-15s | User reads description |
| 4 | Add to cart | 2-3s | Quick action |
| 5 | View cart | 5-8s | User reviews items |
| 6 | Enter shipping info | 30-60s | User types address |
| 7 | Process payment | 20-30s | User enters card |
| 8 | Confirmation | 3-5s | User screenshots |

**Total flow time**: 78-136 seconds (realistic user behavior)

**Think time importance:**
- Simulates real user pauses
- Prevents unrealistic server hammering
- More accurate resource usage patterns
- Use random variation (gaussian distribution)

**b) (5 points)** What is "ramp-up" and why is it important? Describe an appropriate ramp-up pattern.

**Expected Answer:**

**Ramp-up**: Gradual increase of virtual users over time (not instant spike)

**Importance:**
- Prevents sudden shock to system
- Identifies performance degradation points
- More realistic traffic pattern
- Allows system warmup (JIT compilation, cache population)
- Easier to correlate issues with specific load levels

**Appropriate pattern for ToolShop (500 VU target):**

```
VUs
500│                    ┌───────────────────────────┐
   │                   /                             \
400│                  /                               \
   │                 /                                 \
300│                /                                   \
   │               /                                     \
200│              /                                       \
   │             /                                         \
100│            /                                           \
   │           /                                             \
 0 │──────────/                                               \────
   └──────────────────────────────────────────────────────────────▶
   0        5min      15min                    45min      50min  Time

   Ramp-up    |         Steady State          |    Ramp-down
```

**c) (5 points)** How would you establish baseline metrics and define pass/fail criteria?

**Expected Answer:**

**Baseline establishment:**
1. Run test with minimal load (10 VUs)
2. Measure "ideal" response times
3. Run with expected production load
4. Record metrics over multiple runs
5. Calculate statistical baselines

**Pass/Fail criteria structure:**

| Metric | Threshold | Severity | Action |
|--------|-----------|----------|--------|
| P95 Response Time | < 500ms | Critical | Fail test |
| Error Rate | < 0.1% | Critical | Fail test |
| P99 Response Time | < 2000ms | Warning | Review |
| Throughput | > 100 RPS | Critical | Fail test |
| CPU Usage | < 80% | Warning | Investigate |
| Memory Usage | < 85% | Warning | Investigate |

**Threshold types:**
- **Hard fail**: Test fails, block deployment
- **Soft fail**: Warning, require review
- **Trend-based**: Compare to previous run (>10% degradation = investigate)

---

## Question 5: Analyzing Performance Problems (15 points)

### Scenario
During load testing, you observe:
- Response times normal up to 200 VUs
- At 300 VUs, P95 jumps from 400ms to 2,000ms
- At 400 VUs, errors start appearing (HTTP 503)
- Server CPU: 45%, Memory: 60%

### Questions

**a) (5 points)** What does this pattern suggest? List THREE possible root causes.

**Expected Answer:**

**Pattern suggests:** Resource exhaustion at application layer (not CPU/memory)

**Possible root causes:**

| Root Cause | Evidence | Investigation |
|------------|----------|---------------|
| **Database connection pool exhaustion** | Low CPU/memory but high latency | Check pool size, connection wait time |
| **Thread pool saturation** | Requests queuing | Check thread count, queue depth |
| **External service bottleneck** | App waiting on dependencies | Check external API latencies |
| **Lock contention** | Threads waiting on shared resources | Check lock metrics, deadlocks |
| **Network saturation** | Bandwidth limit reached | Check network I/O metrics |

**Why NOT CPU/memory:**
- CPU at 45% = plenty of compute headroom
- Memory at 60% = no memory pressure
- Issue is likely I/O bound or concurrency limited

**b) (5 points)** Explain the concept of "resource contention" and how it manifests in performance tests.

**Expected Answer:**

**Resource contention:** Multiple processes/threads competing for limited shared resources

**Manifestation patterns:**

```
Performance Cliff Pattern:

Latency ▲           │     ×
        │           │    × ×
        │           │   ×   ×
        │           │  ×     ×
        │           │ ×
        │───────────×──────────────
        │× × × × × ×│
        └───────────┴──────────────▶ VUs
                    ↑
              Contention Point
```

**Types of contention:**

| Resource | Symptom | Solution |
|----------|---------|----------|
| **Database connections** | Connection timeout errors | Increase pool, optimize queries |
| **Thread pool** | Request queuing, increased latency | Increase threads, async processing |
| **File handles** | "Too many open files" | Increase limits, close properly |
| **Network sockets** | Connection refused | Increase backlog, connection reuse |
| **Locks/Mutexes** | Increased wait time | Reduce lock scope, use read/write locks |

**c) (5 points)** Describe a systematic approach to diagnosing this performance issue.

**Expected Answer:**

**Diagnostic approach (OSI-style, layer by layer):**

```
┌─────────────────────────────────────────────┐
│ 1. INFRASTRUCTURE LAYER                      │
│    □ CPU, Memory, Disk I/O, Network I/O     │
│    □ If all normal → move to next layer     │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│ 2. PLATFORM LAYER                           │
│    □ Container limits, Pod resources        │
│    □ Load balancer connections              │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│ 3. APPLICATION LAYER                        │
│    □ Thread pool size and utilization       │
│    □ Connection pool stats                  │
│    □ GC pauses, heap usage                  │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│ 4. DATABASE LAYER                           │
│    □ Query execution time                   │
│    □ Lock waits, deadlocks                  │
│    □ Connection pool at DB side             │
└─────────────────────────────────────────────┘
                    ▼
┌─────────────────────────────────────────────┐
│ 5. EXTERNAL DEPENDENCIES                    │
│    □ Third-party API latency                │
│    □ Cache hit rates                        │
│    □ Message queue depth                    │
└─────────────────────────────────────────────┘
```

**Specific for this scenario:**
1. ✅ Infrastructure: CPU/Memory normal → not the bottleneck
2. Check application thread pool metrics
3. Check database connection pool (likely culprit)
4. Profile slow requests to find wait time breakdown

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | Performance Test Types | 15 |
| Q2 | Metrics Analysis | 20 |
| Q3 | Tool Comparison | 15 |
| Q4 | Test Design | 15 |
| Q5 | Problem Analysis | 15 |
| **Total** | | **80** |
