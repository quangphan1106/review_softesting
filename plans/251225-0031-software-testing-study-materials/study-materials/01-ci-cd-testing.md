# Topic 1: CI/CD Testing - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 CI/CD Fundamentals

**Continuous Integration (CI):**
- Developers frequently merge code changes into a central repository
- Automated builds and tests run on every merge
- Goal: Detect integration issues early (fail fast)

**Continuous Delivery/Deployment (CD):**
- Continuous Delivery: Code always in deployable state, manual release approval
- Continuous Deployment: Automatic release to production after passing tests
- Goal: Reduce time-to-market, minimize manual intervention

### 1.2 Testing in CI/CD Pipeline Stages

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   COMMIT    │───>│    BUILD    │───>│    TEST     │───>│   DEPLOY    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
      │                  │                  │                  │
   Lint &             Compile          Unit Tests          Staging/
   Format              Code           Integration          Production
                                         E2E
```

**Stage 1 - Commit:**
- Linting (ESLint, Pylint)
- Code formatting (Prettier, Black)
- Static analysis (SonarQube)
- Pre-commit hooks

**Stage 2 - Build:**
- Compile source code
- Dependency resolution
- Container image creation
- Artifact generation

**Stage 3 - Test:**
- Unit tests (fastest, run first)
- Integration tests (API, DB connections)
- E2E tests (UI flows, slowest)
- Performance tests (load, stress)

**Stage 4 - Deploy:**
- Staging environment validation
- Smoke tests post-deployment
- Production monitoring
- Rollback procedures

### 1.3 GitHub Actions Architecture

**Components:**
- **Workflow**: YAML file defining automation (`.github/workflows/`)
- **Job**: Group of steps running on same runner
- **Step**: Individual task (action or shell command)
- **Runner**: Machine executing jobs (GitHub-hosted or self-hosted)
- **Action**: Reusable unit of code

**Event Triggers:**
- `push` / `pull_request` - code changes
- `schedule` (cron) - periodic runs
- `workflow_dispatch` - manual trigger

---

## 2. Tools & Techniques Comparison

### 2.1 CI/CD Platform Comparison

| Feature | GitHub Actions | GitLab CI | Jenkins | CircleCI |
|---------|---------------|-----------|---------|----------|
| **Hosting** | Cloud (free tier) | Cloud/Self-hosted | Self-hosted | Cloud |
| **Config** | YAML | YAML | Groovy/UI | YAML |
| **Marketplace** | 15,000+ actions | Templates | 1800+ plugins | Orbs |
| **Container** | Native Docker | Native Docker | Plugin-based | Native |
| **Matrix Builds** | Native | Native | Plugins | Native |
| **Caching** | Built-in | Built-in | Plugins | Built-in |
| **Secrets** | Encrypted | Encrypted | Credentials | Context |

### 2.2 GitHub Actions Advanced Patterns

**Matrix Strategy Concepts:**
- Run same job across multiple configurations (OS × Node versions)
- `fail-fast: false` - don't cancel other jobs on failure
- `exclude` - skip specific combinations
- `include` - add specific combinations with extra variables

**Caching Concepts:**
- Cache path: dependencies folder (`node_modules`, `~/.npm`)
- Key: `runner.os` + `hashFiles('package-lock.json')`
- `restore-keys`: fallback keys for partial cache hit

**Artifacts:**
- Upload: save files between jobs (`coverage/`, `reports/`)
- Download: retrieve in subsequent jobs
- `retention-days`: auto-cleanup after N days

### 2.3 Docker in CI/CD

**Multi-stage Build Concept:**
- Stage 1 (builder): Install deps, compile code
- Stage 2 (production): Copy only built artifacts → smaller image
- Key: `COPY --from=builder` to copy from previous stage

**Docker Compose for Testing:**
- Define multiple services: app, db, redis
- `depends_on`: startup order control
- Isolated environment for integration tests

---

## 3. Practical Test Case Design

### 3.1 Pipeline Test Strategy

| Test Type | Trigger | Duration | Purpose |
|-----------|---------|----------|---------|
| Unit | Every commit | < 2 min | Fast feedback |
| Integration | PR merge | < 10 min | API contracts |
| E2E | Nightly/Release | < 30 min | User flows |
| Performance | Weekly/Release | Variable | Load testing |
| Security | PR/Daily | < 15 min | Vulnerability scan |

### 3.2 Test Cases for CI/CD Pipeline

**TC-CI-001: Pipeline Trigger on Push**
- **Precondition**: Code pushed to main branch
- **Steps**: Monitor workflow execution
- **Expected**: All jobs triggered, tests run, artifacts uploaded
- **Pass Criteria**: Exit code 0, all checks green

**TC-CI-002: Matrix Build Coverage**
- **Precondition**: Matrix with 3 OS × 3 Node versions
- **Steps**: Push code, observe job creation
- **Expected**: 9 parallel jobs created
- **Pass Criteria**: All combinations tested

**TC-CI-003: Cache Hit/Miss Behavior**
- **Precondition**: First run (cache miss)
- **Steps**: Run pipeline twice
- **Expected**: Second run uses cache, faster execution
- **Pass Criteria**: Cache restored message in logs

**TC-CI-004: Secret Injection**
- **Precondition**: API_KEY secret configured
- **Steps**: Access ${{ secrets.API_KEY }} in workflow
- **Expected**: Secret available but masked in logs
- **Pass Criteria**: Value accessible, "***" in output

**TC-CI-005: Artifact Upload/Download**
- **Precondition**: Test generates coverage report
- **Steps**: Upload artifact, download in subsequent job
- **Expected**: File available in download job
- **Pass Criteria**: File content matches

---

## 4. Troubleshooting Scenarios

### 4.1 Common CI/CD Failures

| Problem | Symptoms | Root Causes | Solutions |
|---------|----------|-------------|-----------|
| **Flaky Tests** | Pass locally, fail in CI | Race conditions, shared state, external deps | Retry logic, test containers, mock services |
| **Cache Corruption** | Build fails after restore | Incompatible cache, partial write | Include branch in key, use `hashFiles()`, version keys |
| **Docker Build Fail** | Timeout or error | No layer cache, large context | Enable BuildKit, use `.dockerignore`, pin versions |
| **Secret Exposure** | Secrets in logs | Echo secrets, debug mode | Never echo, use `add-mask`, audit 3rd-party actions |

### 4.2 Debugging Pipeline Failures

**Debug Process:**
1. Check job logs for exact error message
2. Identify failing step (expand in UI)
3. Look for environment differences (OS, versions)
4. Run locally with same configuration
5. Enable debug: `ACTIONS_STEP_DEBUG=true`
6. Add env dump step: print OS, versions, `pwd`, `env | sort`

---

## 5. Toolshop Homework Connection

### 5.1 Automation Testing (HW06) Integration

**Pipeline Concept:**
- Trigger: push/PR to main
- Matrix: run across chrome, firefox, edge (parallel)
- Steps: checkout → setup Python → install deps → run pytest → upload artifacts

**Key Points:**
- Cross-browser testing via matrix
- Artifact upload for test reports per browser
- Parallel execution = faster feedback

### 5.2 Performance Testing (HW07) Integration

**Pipeline Concept:**
- Install JMeter in CI environment
- Run load test: `jmeter -n -t loadtest.jmx -l results.jtl`
- Analyze results with Python script

---

## 6. Application-Level Exam Questions

### Question 1: Matrix Build Optimization
**Scenario:** 4 OS × 5 Node versions = 20 jobs, taking 2 hours.

**Question:** How to optimize for faster feedback?

**Answer:**
- Use `fail-fast: true` - stop early on failure
- Primary OS (ubuntu) with 3 Node versions (18, 20, 22) = 3 jobs
- `include` only critical cross-platform: windows+node20, macos+node20
- Total: 5 jobs instead of 20
- Full matrix only on release/nightly builds

### Question 2: Cache Strategy Design
**Scenario:** Next.js app with npm deps, .next/cache, Turbo cache.

**Question:** Design caching strategy to minimize build time.

**Answer:**
| Cache | Path | Key Strategy |
|-------|------|--------------|
| node_modules | `node_modules` | `hashFiles('package-lock.json')` |
| Next.js | `.next/cache` | `hashFiles('**/*.js')` + restore-keys |
| Turbo | `.turbo` | `github.sha` (incremental) |

Key: Separate caches, content-based invalidation, restore-keys for fallback

### Question 3: Debugging Failed Deployment
**Scenario:** Logs show success but users report errors.

**Answer:**
1. **Smoke tests**: `curl -f /health`, `curl /api/version`
2. **Verify env**: Compare deployed SHA vs expected
3. **Check logs**: Container startup, DB migrations
4. **Rollback**: `if: failure()` → `kubectl rollout undo`

---

## 7. Cheatsheet / Quick Reference

### GitHub Actions Quick Syntax

| Element | Syntax | Purpose |
|---------|--------|---------|
| Workflow | `name: CI` + `on: [push]` | Define pipeline |
| Job | `jobs: build: runs-on: ubuntu-latest` | Group of steps |
| Step | `- uses: action@v4` or `- run: cmd` | Individual task |
| Env var | `env: NODE_ENV: test` | Set environment |
| Secret | `${{ secrets.API_KEY }}` | Access encrypted value |
| Conditional | `if: success()` / `failure()` / `always()` | Control flow |
| Dependency | `needs: [job1, job2]` | Job ordering |
| Matrix | `strategy: matrix: node: [18,20]` | Multi-config |
| Cache | `actions/cache@v4` + `key: hashFiles()` | Speed up builds |

### Common Actions

| Action | Purpose |
|--------|---------|
| `actions/checkout@v4` | Clone repository |
| `actions/setup-node@v4` | Install Node.js |
| `actions/cache@v4` | Cache dependencies |
| `actions/upload-artifact@v4` | Upload files |
| `actions/download-artifact@v4` | Download files |
| `docker/build-push-action@v5` | Build Docker image |

### CI/CD Best Practices Checklist

- [ ] Fast feedback (unit tests < 2 min)
- [ ] Parallel execution where possible
- [ ] Caching for dependencies
- [ ] Matrix for cross-platform testing
- [ ] Secrets management (never in code)
- [ ] Artifact retention policy
- [ ] Failure notifications
- [ ] Rollback automation
- [ ] Security scanning
- [ ] Performance monitoring

---

## 8. Key Takeaways for Exam

1. **Pipeline Design**: Understand stage ordering (lint → build → test → deploy)
2. **Matrix Builds**: Know when to use include/exclude for optimization
3. **Caching**: Use `hashFiles()` for dependency-based invalidation
4. **Docker**: Multi-stage builds reduce image size
5. **Debugging**: Always add smoke tests post-deployment
6. **Secrets**: Never echo, use add-mask for dynamic values
7. **2025 Updates**: GitHub cache v4, BuildKit for Docker

---

*Study time recommended: 2-3 hours*
*Practice: Set up a GitHub Actions workflow for a sample project*
