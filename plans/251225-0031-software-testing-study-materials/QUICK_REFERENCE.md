# Quick Reference: Topics 5-9 Cheat Sheet

## Topic 5: API Testing & Mocking - Quick Lookup

### Tool Decision Matrix
| Need | Tool | Reason |
|------|------|--------|
| Contract validation in CI | Dredd | Validates API spec compliance automatically |
| Complex mock responses | WireMock | Recording + proxying + network faults |
| Team collaboration | Postman Mock | UI-friendly, collections-based |
| Load test mocking | WireMock Cloud | No rate limits, dedicated capacity |
| GraphQL schema mocking | GraphQL Faker | SDL-based auto-generation |

### Key Concepts
- Newman: Postman CLI runner; use in Jenkins/GitHub Actions
- BOLA: Attacker changes ID in request → test: verify authorization checks
- BFLA: User accesses admin endpoints → test: HTTP method restrictions
- GraphQL 69% vulnerability: unbounded query depth + schema introspection

### 2025 Updates
- Postman Private API Runners execute inside VPC (no VPN workaround)
- MCP support integrates Newman with Claude, Cursor

---

## Topic 6: AI Testing Tools - Quick Lookup

### Healenium Self-Healing
**Algorithm**: LCS (Longest Common Subsequence) + ML gradient-boosted priorities
**When**: Fails on run N, heals on run N+1 (learns from history)
**Benefit**: Maintenance ↓ 60%; works with Selenium + Appium

### Qodo vs ChatGPT
| Factor | Qodo | ChatGPT |
|--------|------|---------|
| Setup | IDE extension (in-editor) | Web browser interface |
| Edge Cases | Deep code analysis | Limited context |
| Accuracy | 82/100 for test gen | Requires review |
| Speed | Faster (IDE integrated) | Slower (context switches) |
| Cost | Free tier available | Free/paid tiers |

### Test Oracle Problem
**Challenge**: AI outputs probabilistic → can't define "correct"
**Solution**: Differential testing (compare 2 systems) OR statistical oracles (accept ranges)

### Applitools Eyes
**Trained on**: 4 billion app screens
**Speedup**: 10x cross-browser (Ultrafast Grid parallel)
**Intelligence**: Ignores dynamic content (ads, dates, IDs)

---

## Topic 7: Testing AI Systems - Quick Lookup

### ML Framework Hybrid (2025 Best Practice)
```
Development: PyTest-ML (30% faster, low memory)
    ↓
Production CI/CD: TensorFlow Extended (45% faster pipelines)
```

### Bias Testing Checklist
1. Slice data by sensitive attributes (gender, race, age)
2. Calculate metrics per slice (AIF360 library)
3. Identify imbalance (prediction distribution varies 5%+?)
4. Mitigate: MinDiff loss function OR data augmentation
5. Re-test fairness post-fix

### Adversarial Robustness Metrics
- **Transferability**: Does adversarial example fool Model A + Model B?
- **Success Rate**: % of adversarial examples that fool?
- **UAR (OpenAI)**: Robustness against unforeseen attacks

### LLM Evaluation (2025 Shift)
- OLD: MMLU, MMLU-Pro (saturated; models memorize)
- NEW: AgentBench (multi-turn tasks), HLE (expert reasoning)
- METHOD: Automated (BLEU, ROUGE) + Human review (fluency, correctness)

---

## Topic 8: Database Testing - Quick Lookup

### ACID Test Scenarios (Quick Reference)

| Property | Test Approach | Pass Criteria |
|----------|---|---|
| **Atomicity** | Simulate crash mid-transaction | Both ops commit OR both rollback |
| **Consistency** | Verify constraints before/after | SUM(balances) unchanged |
| **Isolation** | Run concurrent reads + writes | No dirty reads across transactions |
| **Durability** | Kill process after commit | Data persists after restart |

### DBUnit Spring Annotations
```
@DatabaseSetup("/data/users.xml")  → Populate before test
@DatabaseTearDown → Clean after
@ExpectedDatabase(assertion=NON_STRICT) → Verify final state
```

### Index Optimization Thresholds
- Rebuild if fragmentation **>30%**
- Use Query Store for before/after A/B testing
- Covering indexes: include all needed columns (avoid table access)

### Data Migration 3-Phase
1. **Pre**: Validate source row counts, data types
2. **During**: Handle NULL, duplicates, transformations
3. **Post**: 100% verification (not sampling) + test rollback

### SQL Injection Testing Tools Priority
1. **SQLmap**: Full automation, 6 injection types
2. **Burp Suite**: Web proxy + Intruder fuzzing
3. **OWASP ZAP**: Free, pre-configured payloads
4. Integrate into CI/CD on each code push

---

## Topic 9: Domain Testing & Data Generation - Quick Lookup

### ECP + BVA Combination Rule
```
Equivalence Classes:  valid, too-short, too-long
Boundary Values:      min-1, min, max, max+1
Combined Test Set:    1 representative per class + 4 boundaries = minimal coverage
```

### Faker vs Mockaroo Decision
| Use Case | Choice | Why |
|----------|--------|-----|
| Unit test data generation | **Faker** | Code integration; fine control |
| Quick CSV/SQL seed file | **Mockaroo** | No-code web UI; 100K rows instant |
| Relationship integrity (Denver→Colorado) | **Mockaroo** | Built-in smart pairing |
| Complex data logic | **Faker** | Programmatic control |

### Pairwise Testing ROI
- **Exhaustive**: 4000 test cases
- **Conventional**: 24 test cases
- **Pairwise**: 6 test cases (still catches 95%+ defects)
- **Formula**: All parameter pairs covered, not all combinations

### Advanced: N-wise Testing
- 3-wise, 4-wise catch rarer multi-factor bugs
- Exponential cost; use for critical systems only

---

## Cross-Topic Integration Patterns

### API Security + Database Security
```
Topic 5 (API Testing) → Newman + OWASP ASTF
Topic 8 (Database)   → SQLmap in CI/CD
Combined: Test API endpoint + Database injection in single pipeline
```

### AI Tools + AI System Testing
```
Topic 6: Healenium auto-fixes UI locators
Topic 7: Bias testing detects discriminatory UI layouts
Risk: If Healenium trained data is skewed → perpetuates bias
Mitigation: Audit Healenium's locator selection for bias
```

### Domain Testing + Data Generation
```
Topic 9: Design test cases with Pairwise (6 cases)
Topic 9: Generate data for those 6 cases with Faker
Result: Minimal test data, maximum coverage
```

---

## Exam Day Memory Joggers

**Topic 5**: "Newman-Dredd-WireMock-GraphQL" (tools in order of complexity)
**Topic 6**: "Healenium-Qodo-Oracle-Applitools" (tools across AI testing spectrum)
**Topic 7**: "PyTest-Bias-Adversarial-LLM" (testing layers for AI systems)
**Topic 8**: "ACID-DBUnit-Index-Migration-SQLi" (5 database concerns)
**Topic 9**: "ECP-BVA-Faker-Pairwise" (4 data/design techniques)

---

## Unresolved Questions (Deep Dive Research)

1. AIF360 fairness metrics: What thresholds pass in hiring vs lending vs healthcare?
2. LLM evaluation: When human judges disagree, which opinion is "ground truth"?
3. Petabyte migrations: Does 100% verification scale to massive datasets practically?
4. GraphQL vs REST: Specific gaps in automated security testing tooling?
5. Adversarial training: At what cost/robustness tradeoff is it ROI-positive?

---

**Last Updated**: December 25, 2025
**Use**: Quick lookup during exam prep or as cheat sheet reference
