# CI/CD Pipeline & DevOps Testing - Concept Questions

## Question 1: Matrix Build Strategy (15 points)

### Scenario
Your team manages an e-commerce web application that must support:
- 3 browsers: Chrome, Firefox, Safari
- 3 operating systems: Windows, Ubuntu, macOS
- 2 Node.js versions: 18.x, 20.x

A full matrix would create 18 combinations. Your CI/CD budget only allows 8 parallel runners.

### Questions

**a) (5 points)** What is a matrix build strategy and why is it important for cross-platform testing?

**Expected Answer:**
- **Matrix build**: Automated strategy to run same tests across multiple environment combinations (OS, browser, runtime versions)
- **Importance**:
  - Ensures compatibility across all supported platforms
  - Catches environment-specific bugs early
  - Reduces manual testing effort
  - Provides confidence for diverse user base

**b) (5 points)** Describe TWO optimization strategies to reduce from 18 to 8 combinations while maintaining good coverage.

**Expected Answer:**

| Strategy | Description | Example |
|----------|-------------|---------|
| **Exclude invalid combinations** | Remove technically impossible or unsupported combos | Safari only runs on macOS - exclude Safari+Windows, Safari+Ubuntu |
| **Include strategic high-value combos** | Manually specify most critical combinations | Latest Node.js + most popular browser per OS |
| **Prioritize by user metrics** | Focus on combinations matching actual user distribution | If 70% users are Chrome/Windows, prioritize that |
| **Risk-based selection** | Include historically problematic combinations | Known issues with Firefox+Ubuntu → keep it |

**c) (5 points)** Explain the concept of "fail-fast" in CI/CD pipelines. When should you enable vs disable it?

**Expected Answer:**
- **Fail-fast**: Pipeline stops all parallel jobs immediately when one job fails
- **Enable when**:
  - Jobs share dependencies (one failure means others will likely fail)
  - Want fast feedback during development
  - Cost savings on compute resources
- **Disable when**:
  - Need complete test results for all platforms
  - Debugging environment-specific issues
  - Running release validation suite

---

## Question 2: CI/CD Caching Strategies (15 points)

### Scenario
Your Node.js project has:
- 500+ npm dependencies (~800MB node_modules)
- Build artifacts (~200MB)
- Test fixtures/data (~50MB)

Without caching, each pipeline run takes 15 minutes, with 10 minutes spent on npm install.

### Questions

**a) (5 points)** Explain what CI/CD caching is and list THREE types of content commonly cached.

**Expected Answer:**
- **CI/CD Caching**: Storing and reusing files between pipeline runs to speed up builds
- **Common cached content**:
  1. **Package dependencies**: node_modules, vendor/, pip packages
  2. **Build artifacts**: compiled code, transpiled files, bundled assets
  3. **Docker layers**: base images, intermediate build layers
  4. **Test fixtures**: seed data, mock files, generated test data

**b) (5 points)** What is a cache key and why is the key design critical? Provide an example of a good cache key structure.

**Expected Answer:**
- **Cache key**: Unique identifier to store/retrieve cached content
- **Why critical**:
  - Wrong key → stale cache (old dependencies)
  - Too specific key → cache never hits (wasted storage)
  - Right key → optimal balance of freshness and reuse

**Good cache key structure:**
```
{runner-os}-{dependency-type}-{hash-of-lockfile}
Example: ubuntu-npm-abc123def456
```

**Components explained:**
| Component | Purpose |
|-----------|---------|
| runner-os | Ensures OS-compatible binaries |
| dependency-type | Separates npm vs pip vs other caches |
| hash-of-lockfile | Invalidates when dependencies change |

**c) (5 points)** Describe the cache invalidation problem and TWO strategies to handle it.

**Expected Answer:**
- **Problem**: Cache becomes stale but system doesn't know to refresh it
  - Security patches not applied
  - Bug fixes not picked up
  - Incompatible versions persist

**Strategies:**
| Strategy | How it works |
|----------|--------------|
| **Hash-based invalidation** | Include file hash in key; lockfile changes → new key → fresh cache |
| **Time-based expiration** | Set max cache age (e.g., 7 days); forces periodic refresh |
| **Manual purge triggers** | Allow developers to force cache clear via commit message or UI |

---

## Question 3: Secrets Management in CI/CD (15 points)

### Scenario
Your pipeline needs to access:
- Database credentials for integration tests
- API keys for third-party services (payment, email)
- Docker registry credentials for image push
- Cloud provider credentials for deployment

### Questions

**a) (5 points)** Why should secrets NEVER be hardcoded in code or CI/CD configuration files?

**Expected Answer:**
- **Security risks**:
  - Version control history exposes secrets permanently
  - Forked repositories inherit secrets
  - Logs may print secret values accidentally
  - Anyone with repo access gets production credentials
- **Compliance violations**: PCI-DSS, HIPAA, SOC2 require secret protection
- **Breach impact**: Single exposed key can compromise entire infrastructure

**b) (5 points)** Compare ENVIRONMENT VARIABLES vs SECRET MANAGEMENT SERVICES for storing secrets.

**Expected Answer:**

| Aspect | Environment Variables | Secret Management Services |
|--------|----------------------|---------------------------|
| **Security** | Moderate - visible in process list | High - encrypted at rest, access logged |
| **Rotation** | Manual, requires redeploy | Automatic rotation support |
| **Audit** | Limited logging | Full audit trail |
| **Access control** | Binary (have or don't) | Granular, role-based |
| **Examples** | CI/CD platform secrets | HashiCorp Vault, AWS Secrets Manager |
| **Best for** | Simple projects, dev/test | Production, compliance-required |

**c) (5 points)** Explain the principle of "least privilege" for CI/CD secrets and give TWO examples.

**Expected Answer:**
- **Least privilege**: Grant minimum permissions needed for specific task, nothing more

**Examples:**

1. **Database credentials**:
   - Test pipeline: READ-ONLY access to test database
   - Deploy pipeline: READ-WRITE but only to specific tables
   - NOT: Full admin access to production database

2. **Cloud provider credentials**:
   - Build job: Only permission to push to container registry
   - Deploy job: Only permission to update specific service
   - NOT: Full cloud account administrator access

---

## Question 4: Multi-Environment Deployment Pipeline (20 points)

### Scenario
Your company has 4 environments:
1. **Development**: Continuous deployment on every commit
2. **Staging**: Weekly deployment, mirrors production
3. **UAT**: Manual trigger, customer validation
4. **Production**: Approval-gated, blue-green deployment

### Questions

**a) (5 points)** Draw a pipeline flow diagram showing the stages and gates between environments.

**Expected Answer:**

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Build  │────▶│  Test   │────▶│   Dev   │────▶│ Staging │────▶│   UAT   │────▶ Production
│  Stage  │     │  Stage  │     │ Deploy  │     │ Deploy  │     │ Deploy  │
└─────────┘     └─────────┘     └────┬────┘     └────┬────┘     └────┬────┘
                                     │               │               │
                              [Auto on commit] [Weekly schedule] [Manual trigger]
                                                                     │
                                                              ┌──────▼──────┐
                                                              │  Approval   │
                                                              │    Gate     │
                                                              └──────┬──────┘
                                                                     ▼
                                                              ┌─────────────┐
                                                              │ Production  │
                                                              │   Deploy    │
                                                              └─────────────┘
```

**b) (5 points)** What is a "deployment gate" and list THREE types of gates commonly used.

**Expected Answer:**
- **Deployment gate**: Checkpoint that must pass before deployment proceeds

**Types of gates:**

| Gate Type | Description | Example |
|-----------|-------------|---------|
| **Automated quality gate** | Tests/metrics must pass threshold | >80% code coverage, 0 critical vulnerabilities |
| **Manual approval gate** | Human must review and approve | Product owner approves UAT → Prod |
| **Time-based gate** | Deployment only during allowed windows | Only deploy to Prod on weekdays 9AM-5PM |
| **Health check gate** | Previous environment must be healthy | Staging must be green before UAT |

**c) (5 points)** Explain blue-green deployment. What are its advantages and disadvantages?

**Expected Answer:**

**Blue-Green Deployment:**
- Maintain two identical production environments (Blue and Green)
- One serves live traffic, other is idle
- Deploy new version to idle environment
- Switch traffic after validation
- Keep old environment for instant rollback

```
          ┌──────────────┐
          │ Load Balancer│
          └──────┬───────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
   ┌─────────┐       ┌─────────┐
   │  Blue   │       │  Green  │
   │  (v1)   │       │  (v2)   │
   │ [LIVE]  │       │ [IDLE]  │
   └─────────┘       └─────────┘
        │                 │
        └─────┬───────────┘
              ▼ Switch!
```

| Advantages | Disadvantages |
|------------|---------------|
| Zero-downtime deployment | Double infrastructure cost |
| Instant rollback capability | Database schema changes complex |
| Full production testing before switch | Session handling between versions |
| Reduced deployment risk | Requires sophisticated routing |

**d) (5 points)** How would you handle database migrations in a blue-green deployment?

**Expected Answer:**

**Strategy: Backward-Compatible Migrations**

1. **Expand phase** (before switch):
   - Add new columns/tables
   - Keep old columns, don't remove yet
   - New code handles both old and new schema

2. **Migrate phase** (after switch confirmed):
   - Background jobs migrate data
   - Monitor for issues

3. **Contract phase** (after stabilization):
   - Remove old columns/tables
   - Separate deployment cycle

**Key principle**: Never make breaking schema changes in same deployment as code changes

---

## Question 5: Pipeline Optimization (15 points)

### Scenario
Your current pipeline metrics:
- Average duration: 45 minutes
- Failure rate: 15%
- Queue wait time: 10 minutes (only 4 runners)
- Most failures: Flaky UI tests (60% of failures)

### Questions

**a) (5 points)** Identify THREE bottlenecks in this pipeline and propose solutions.

**Expected Answer:**

| Bottleneck | Impact | Solution |
|------------|--------|----------|
| **Long duration (45 min)** | Slow feedback, developer context switching | Parallelize test suites, cache dependencies |
| **Flaky UI tests (60% failures)** | False negatives, wasted reruns, developer distrust | Implement retry logic, fix flaky tests, use test quarantine |
| **Queue wait (10 min)** | Added delay before even starting | Add more runners or use auto-scaling |
| **High failure rate (15%)** | Blocked merges, wasted compute | Address flaky tests, add pre-commit hooks |

**b) (5 points)** What is "test quarantine" and how does it help with flaky tests?

**Expected Answer:**
- **Test quarantine**: Temporarily isolating flaky tests from blocking pipeline

**How it works:**
1. Identify consistently flaky tests (fail >X% without code changes)
2. Move to separate "quarantined" test suite
3. Run quarantined tests but don't fail pipeline
4. Track quarantine metrics
5. Fix or delete tests, then un-quarantine

**Benefits:**
- Pipeline reliability increases immediately
- Developers not blocked by unrelated failures
- Creates visibility and accountability for flaky tests
- Prevents normalization of "just rerun it" culture

**c) (5 points)** Explain the concept of "shift-left testing" and how it relates to CI/CD optimization.

**Expected Answer:**

**Shift-left testing**: Moving testing activities earlier in the development lifecycle

```
Traditional:  Code → Build → Deploy → Test → Fix
                                        ↑ Problems found late
                                        (expensive to fix)

Shift-left:   Lint → Unit Test → Code → Build → Integration Test → Deploy
              ↑     ↑
              Problems found early (cheap to fix)
```

**Relation to CI/CD optimization:**

| Practice | Stage | Benefit |
|----------|-------|---------|
| **Pre-commit hooks** | Before commit | Catch issues in seconds, not pipeline |
| **Fast unit tests first** | Early pipeline | Quick feedback, fail fast |
| **Static analysis** | Build stage | Find bugs without execution |
| **Incremental testing** | Test stage | Only test affected code |

**Result**: Faster pipelines, earlier feedback, cheaper fixes

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | Matrix Build Strategy | 15 |
| Q2 | CI/CD Caching | 15 |
| Q3 | Secrets Management | 15 |
| Q4 | Multi-Environment Deployment | 20 |
| Q5 | Pipeline Optimization | 15 |
| **Total** | | **80** |
