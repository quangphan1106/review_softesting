# Advanced Software Testing Topics: Research Report (Topics 5-9)
**CS423 Final Exam Study Materials | APPLICATION-LEVEL Focus**

---

## 5. API Testing & Mocking

### Postman Newman CLI, Monitors, Mock Servers
**Newman** automates Postman Collections in CI/CD. Key features: multiple reporters (cli, json, junit, html), file uploads, parametrization. **2025 Updates**: On-demand monitors trigger at deploy-time; Private API runners execute tests securely inside VPC; MCP support integrates with Claude, Cursor.

**Application Scenario**: Team needs to test 20+ API endpoints pre-deployment. Use Newman with Jenkins: `newman run collection.json --reporters cli,json` catches regressions before production.

**Mock Servers** host response bodies cloud-side, enabling frontend-backend parallel development. Postman mock servers have rate limits; unsuitable for load tests.

### API Contract Testing (OpenAPI/Swagger)
**Contract Testing** validates API implementations match specifications. Tools: **Postman** (spec-based automation), **Dredd** (API Blueprint testing), **Pact** (consumer-provider contracts), **Swagger Codegen** (mock generation).

**App Scenario**: REST API spec drifts during refactoring. Run Dredd in CI to validate spec compliance automatically, stopping faulty builds before they hit production.

**CI/CD Integration**: Deploy spec → generate mock → run tests → validate. Failed contracts block pipeline.

### WireMock vs Postman Comparison

| Feature | WireMock Cloud | Postman Mock |
|---------|---|---|
| Code Control | Java-centric, custom logic | UI-driven, collections |
| Response Delays | Fixed/random/chunked per stub | Fixed delay only |
| Recording/Proxying | Yes (intercept traffic) | No |
| Rate Limits | No (Performance plans) | Yes, rate-limited |
| Collaboration | Programmers | Broader teams |

**Troubleshooting**: If mocks lag under load, use WireMock; if team needs UI-driven mocks, choose Postman.

### OWASP API Top 10 Security Testing
**Broken Object Level Authorization (BOLA)**: Attacker changes ID in request (e.g., `/user/123` → `/user/456`). **Testing**: Verify non-admin users cannot access other users' data; use random GUIDs for resource IDs.

**Broken Function Level Authorization (BFLA)**: Regular user accesses admin endpoints. **Testing**: Verify HTTP method restrictions (GET OK, DELETE denied); test role-based access control (RBAC) enforces "deny-all, grant explicitly."

**SSRF/Injection**: Validate all client inputs; no trusting nested arguments in complex queries.

**CI/CD Strategy**: Integrate OWASP ASTF (API Security Testing Framework) to detect vulnerabilities pre-production. Continuous automated testing > manual pen tests (3x/year ineffective).

### GraphQL Testing
**Challenges**: Unbounded query depth (69% of public GraphQL APIs vulnerable), schema introspection exposes internals, complex nested authorization.

**Testing Tools**:
- **Altair/Insomnia**: Interactive debugging
- **GraphQL Faker**: Generate realistic fake data from SDL
- **Mock Service Worker (MSW)**: Network-level mocking for unit tests
- **Apollo Studio**: Schema registry, performance monitoring
- **Schemathesis/EvoMaster**: Security fuzzing

**App Scenario**: Frontend dev needs API before backend ready. Use GraphQL Faker + MSW to mock fields; backend team builds in parallel, swap real API on deploy.

---

## 6. AI Testing Tools

### Healenium Self-Healing
**Mechanism**: SelfHealingDriver (proxy) intercepts WebDriver calls, stores locator history (ID, CSS, XPath, text). When element fails, uses **LCS (Longest Common Subsequence) + ML** to find alternative locators.

**Application**: Test Suite breaks after UI refactor (button IDs change). Healenium auto-heals; reports show replaced locators + screenshots. Reduces maintenance 60%+.

**Limitations**: Fails on first run (no history); works on subsequent runs. Requires Selenium/Appium WebDriver interface.

### AI Test Generation: Qodo vs ChatGPT
| Aspect | Qodo | ChatGPT |
|--------|------|---------|
| Focus | Specialized test generation | General-purpose AI |
| IDE Integration | VS Code/JetBrains (in-editor) | None (web interface) |
| Edge Case Detection | Analyzes code behavior deeply | Limited context understanding |
| Language Support | Python, JavaScript, TypeScript | Any language (broad but shallow) |
| Review Requirement | Domain experts validate | Essential (accuracy unpredictable) |

**Scenario**: New payment module needs tests. Qodo maps behavior, generates edge cases (negative amounts, decimal precision). ChatGPT provides scaffold; human reviews for business logic.

**2025 Stat**: 84% devs adopted AI tools; 41% of commits AI-assisted.

### Test Oracle Problem
**Challenge**: AI systems produce probabilistic outputs—defining "correct" is hard. E.g., image classification: is 92% vs 91% accuracy pass/fail?

**Solutions**:
1. **Dynamic Oracle Generation**: ML learns expected behavior from historical data
2. **Differential Testing**: Compare 2 systems solving same task
3. **Intramorphic Testing**: Modify system slightly; relate original vs modified outputs
4. **Statistical Oracles**: Accept ranges (e.g., confidence ±3%)
5. **Pseudo-Oracles**: Separately written program validates outputs

**App Usage**: LLM test scenario. Cannot assert exact text. Use oracle measuring semantic similarity (embeddings-based) instead.

### Applitools Eyes Visual AI
**Technology**: 10+ years training on 4B app screens. Detects pixel-level + smart content changes (ignores ads, dates, IDs intelligently).

**Features**: Matches baseline images, identifies visual regressions, handles dynamic content. Ultrafast Grid parallelizes cross-browser tests; 10x speedup reported.

**Application**: E-commerce site redesign. Baseline all pages with Eyes. Every PR auto-compares; team reviews diffs visually. Reduced visual bugs by 78%, maintenance by 78% (Peloton case study).

**Integration**: Works with Cypress, Selenium, Playwright, Appium.

---

## 7. Testing AI Systems

### ML Model Testing Frameworks
**PyTest-ML 2.0 vs TensorFlow Extended (TFX)**:
- **PyTest-ML**: 30% faster unit tests, low memory (1.3GB), flexible
- **TFX**: 45% faster full pipelines, production-grade validation, 3.8GB memory

**Best Practice (2025)**: Hybrid—PyTest for dev, TFX for CI/CD production pipelines.

**TensorFlow Test Tools**:
- `tf.test.TestCase`: Extends unittest with TF-specific assertions
- `TensorFlow Data Validation`: Data integrity checks
- `TensorFlow Model Analysis`: In-depth metrics

**Application**: Training model with noisy data. Use TF Data Validation to flag outliers; MLflow pytest plugin automates regression detection on each training run.

### Bias & Fairness Testing
**Common Biases**:
- **Reporting Bias**: Dataset reflects unusual events (news bias)
- **Selection Bias**: Non-representative sampling
- **Historical Bias**: Old data encodes past inequities

**Testing Tools**:
- **AI Fairness 360 (AIF360)**: Comprehensive metrics + mitigation algorithms
- **Aequitas**: Bias auditing per population subgroups
- **FairML**: Input ranking to quantify feature dependence

**App Scenario**: Hiring model biased against female candidates. Slice data by gender; run Aequitas; apply MinDiff loss to balance prediction distributions. Re-test fairness metrics post-mitigation.

### Adversarial Testing Robustness
**Attack Types**: Evasion (mislead model), data poisoning (corrupt training), Byzantine (distributed attacks), model extraction (steal weights).

**Metrics**:
- **Transferability**: Adversarial examples fool multiple models?
- **Success Rate**: % of adversarial examples that deceive?
- **UAR (Unforeseen Attack Robustness)**: OpenAI metric for unanticipated attacks

**Techniques**: FGSM (fast gradients), APSO (particle swarm), adversarial training (iterate: generate examples → retrain).

**Application**: Computer vision classifier for autonomous vehicles. Adversarially test: 1-pixel perturbations, adversarial patches. Ensure robustness before deployment.

### LLM Evaluation & Benchmarking (2025)
**Static Benchmarks Saturating**: MMLU, MMLU-Pro reach ceiling; models memorize test data (GSM8K math problem evidence).

**2025 Trends**: Dynamic/interactive evals replace static. **AgentBench** tests multi-turn reasoning (OS tasks, web shopping, code execution). **Humanity's Last Exam (HLE)** expert-level reasoning; most models <30% accuracy.

**Evaluation Methods**:
- **Automated**: BLEU, ROUGE, Perplexity (quick but misses nuance)
- **Human**: Coherence, relevance, fluency (captures quality but slow)
- **Balanced**: Hybrid approach

**App Scenario**: RAG chatbot evaluation. Use automated metrics (ROUGE for retrieval) + human review (fluency/correctness). Safety eval: test harmful request refusal.

---

## 8. Database Testing

### ACID Properties Testing Scenarios
**Atomicity (All or Nothing)**: Bank transfer: debit Alice, credit Bob. If Bob credit fails → rollback Alice's debit.
- **Test**: Simulate crash mid-transaction; verify both ops rollback or both commit.

**Consistency (Data Integrity)**: Sum(Alice, Bob) = $200 before/after.
- **Test**: Run concurrent transactions; verify final sum unchanged.

**Isolation (No Interference)**: Transaction X reads data modified by Transaction Y mid-flight?
- **Test**: Run concurrent reads/writes; test isolation levels (read committed, snapshot). Verify no dirty reads.

**Durability (Permanent)**: Power failure after commit—data survives?
- **Test**: Commit transaction → kill process → restart → verify data persists.

**Real-World**: MongoDB multi-document transactions guarantee ACID across sharded clusters. Test with 2-phase commit scenarios.

### DBUnit Advanced Usage
**@DatabaseSetup/@DatabaseTearDown**: Populate before test, clean after.
```java
@DatabaseSetup("/data/users.xml")
public void testDeleteUser() { ... }
@DatabaseTearDown(type=DatabaseOperation.DELETE_ALL, value="/data/users.xml")
```

**@ExpectedDatabase**: Verify final DB state matches expected XML.
```java
@ExpectedDatabase(assertion=NON_STRICT, value="/expected.xml")
public void testInsert() { ... }
```

**Advanced**: Custom loaders, Spring integration, Docker-based multi-database testing.

**Application**: Integration test suite. Setup test data via XML → run CRUD ops → assert final state matches expected. Docker-ized PostgreSQL ensures reproducibility.

### Index Performance Testing
**Strategies**:
- **Covering Indexes**: All columns needed in index (avoid table access)
- **Composite Indexes**: Multiple columns (order matters for query patterns)
- **Partial Indexes**: Index only active rows (filter on "active=true")

**Testing Approach**: Clear cache (`CHECKPOINT`, `DBCC DROPCLEANBUFFERS`) → measure query time before/after index → A/B test via Query Store.

**Threshold**: Rebuild if fragmentation >30%; reorganize if <30%.

**App Scenario**: Slow report query on 100M row table. Analyze execution plan; add composite index on (date, status, user_id). Test with realistic data volumes; measure latency improvement.

### Data Migration Testing Strategy
**3 Phases**:
1. **Pre-Migration**: Validate source data (row counts, data types)
2. **Migration**: Map & transform data; handle NULL/duplicates
3. **Post-Migration**: Reconcile source ↔ target; test rollback

**Validation Approaches**:
- **Sampling**: Check random subset (risk of missing errors)
- **100% Verification**: Automated tools independent of migration tool (safer)

**Best Practice**: Dry-run migration → test data reconciliation → plan rollback → execute with monitoring. Test edge cases (null, max length, special chars).

**Tool Integration**: Lineage tracking (column-level), SQL dialect translation, cross-database diffing accelerate testing.

### SQL Injection Testing Techniques & Tools
**5 Exploitation Methods**:
1. **Union Operator**: Combine queries (info leakage)
2. **Boolean-Based**: Conditional true/false (blind attacks)
3. **Error-Based**: Force DB errors (info extraction)
4. **Out-of-Band**: Exfiltrate via DNS/HTTP callback
5. **Time-Delay**: Sleep queries to infer conditions

**Top Tools**:
- **SQLmap**: Automates all 6 injection types; fine-grained control
- **Burp Suite**: Web proxy intercept + Intruder fuzzing
- **OWASP ZAP**: Free, pre-configured payloads
- **Invicti**: Enterprise continuous scanning

**Best Practice**: Integrate SQLmap into CI/CD; test on each code push. Input validation + parameterized queries prevent 95%+ of SQLi. Regular pen testing recommended.

---

## 9. Domain Testing & Data Generation

### Equivalence Class Partitioning (Advanced)
**Concept**: Group inputs into partitions; test one representative per partition. Reduces test cases 10:1+.

**Edge Case Handling**: Combine with Boundary Value Analysis. E.g., password 8-20 chars:
- Classes: valid, too short, too long
- Boundaries: 7, 8, 20, 21 chars

**Advanced**: Grey-box testing identifies internal partitions unknown to black-box. If function branches on 1-6 vs 7-12 internally, create 4 classes (not 3) to test both paths.

**Limitation**: Poorly defined partitions miss defects. Requires domain expertise; unsuitable for complex integrated scenarios.

**Application**: Age field (1-120). Classes: child (1-12), teen (13-17), adult (18-120), invalid (<0, >200). Test one per class + boundaries (0, 1, 120, 121).

### Boundary Value Analysis (Advanced)
**Core Values**: Min, just above min, nominal, just below max, max, just outside max.

**Example**: Input 1-100 → test: 0, 1, 50, 99, 100, 101.

**Modern Extensions**:
- Non-numeric: Character lengths, dates, enums
- Machine Learning: Test extreme distributions (highly skewed, missing data)
- Environmental: Max concurrent users, disk space, memory

**Best Practice**: Combine exploratory testing (creative break attempts) with BVA (systematic edge testing). Test reverse workflows, rapid clicking, data combinations.

**Limitation**: Doesn't catch interdependent variables (age limit depends on gender). Requires additional logic testing.

### Faker & Mockaroo Comparison

| Aspect | Faker | Mockaroo |
|--------|-------|----------|
| Type | Code library (Node/Python) | Web UI tool |
| Setup | Programmatic integration | No-code browser |
| Output | Via code | CSV, JSON, SQL, Excel |
| Mock API | No | Yes (built-in) |
| Data Relationships | Manual setup | Limited |
| Best For | Dev integration | Quick no-code generation |
| Cost | Free | Free (1K rows), $50-500/year paid |

**Application**: Unit test needs user data. Use Faker.js in Jest. Integration test needs DB seed. Use Mockaroo to generate 10K realistic rows → import SQL.

**Advanced**: Mockaroo ensures realistic relationships (Denver → Colorado). Faker requires manual pairing.

### Combinatorial (Pairwise) Testing
**Concept**: Test all **pairs** of parameter values. Reduces exhaustive combos 4000→6 test cases; still catches 95%+ defects (most bugs involve ≤2 factors).

**Tools**:
- **PICT** (Microsoft): Text config → test cases
- **ACTS** (NIST): Advanced algorithms
- **Pairwiser**: Fastest, visualizer for coverage
- **Hexawise**: Cloud-based

**Example**:
- Browser (Chrome, Firefox, Safari): 3
- OS (Win, Mac, Linux): 3
- Screen size (800×600, 1920×1080): 2
- Exhaustive: 18 cases → Pairwise: 6 cases

**Advanced**: N-wise testing (3-way, 4-way) captures rarer multi-factor bugs; exponential growth but practical for critical systems.

**Application**: Feature with 5+ params. Use Pairwiser to generate minimal test suite covering all pairs → run faster than exhaustive without sacrificing quality.

---

## Key Takeaways for Exam

1. **API Testing**: Newman + Dredd for contracts; WireMock for complex stubs; OWASP ASTF for security; GraphQL needs specialized tools (MSW, Faker).
2. **AI Tools**: Healenium reduces maintenance; Qodo > ChatGPT for code testing; Applitools Eyes visual regression; Oracle Problem solved via differential/dynamic approaches.
3. **Testing AI**: ML frameworks vary (PyTest vs TFX); fairness + adversarial robustness essential; LLM evals shifting from static to dynamic.
4. **DB Testing**: ACID scenarios testable via transactions + concurrent ops; DBUnit automates setup/teardown; indexes need fragmentation monitoring; migration testing requires 100% verification for safety.
5. **Domain Testing**: ECP + BVA = comprehensive coverage; Faker/Mockaroo for data; Pairwise reduces test explosion while maintaining quality.

---

## Unresolved Questions

1. How to quantify "acceptable" fairness metrics (AIF360) thresholds per domain?
2. Best practices for LLM evaluation when ground truth is subjective?
3. Real-world experience: Does 100% migration verification scale to petabyte-sized data?
4. GraphQL security testing automation maturity vs REST (gap analysis)?

