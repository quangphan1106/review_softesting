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
```yaml
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:      # Manual trigger
```

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

**Matrix Strategy:**
```yaml
strategy:
  fail-fast: false  # Don't cancel other jobs on failure
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [18, 20, 22]
    exclude:
      - os: macos-latest
        node: 18
    include:
      - os: ubuntu-latest
        node: 22
        experimental: true
```

**Caching Best Practices:**
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

**Artifacts:**
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage/
    retention-days: 7

- uses: actions/download-artifact@v4
  with:
    name: coverage-report
```

### 2.3 Docker in CI/CD

**Multi-stage Build Example:**
```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-slim
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**Docker Compose for Testing:**
```yaml
services:
  app:
    build: .
    depends_on:
      - db
      - redis
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: test
  redis:
    image: redis:7
```

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

**Problem 1: Flaky Tests**
```
Symptom: Tests pass locally but fail intermittently in CI
Root Causes:
- Race conditions in async code
- Shared state between tests
- External service dependencies
- Time-based tests (timezone issues)

Solution:
- Add retry logic: `jest --bail --maxWorkers=1`
- Use test containers for DB
- Mock external services
- Use fixed timestamps in tests
```

**Problem 2: Cache Corruption**
```
Symptom: Build fails after cache restore
Root Causes:
- Incompatible cache across branches
- Partial cache write (timeout)
- Lock file changed but cache key unchanged

Solution:
- Include branch in cache key
- Use hashFiles() for dependencies
- Add cache version: key-v2-${{ hashFiles('...') }}
```

**Problem 3: Docker Build Failures**
```
Symptom: Image build times out or fails
Root Causes:
- No layer caching configured
- Large context (include/exclude not set)
- Base image unavailable

Solution:
- Configure BuildKit cache: DOCKER_BUILDKIT=1
- Use .dockerignore
- Pin base image versions
```

**Problem 4: Secret Exposure**
```
Symptom: Secrets visible in logs
Root Causes:
- Echoing secret values
- Debug mode enabled
- Third-party action logging

Solution:
- Never echo secrets
- Use `add-mask` for dynamic secrets
- Audit third-party actions
```

### 4.2 Debugging Pipeline Failures

**Step-by-Step Debug Process:**
1. Check job logs for exact error message
2. Identify failing step (expand in UI)
3. Look for environment differences
4. Run locally with same configuration
5. Add debug output: `ACTIONS_STEP_DEBUG=true`
6. Use SSH debug action for interactive debugging

**Useful Debug Commands:**
```yaml
- name: Debug environment
  run: |
    echo "OS: ${{ runner.os }}"
    echo "Node: $(node --version)"
    echo "NPM: $(npm --version)"
    pwd && ls -la
    env | sort
```

---

## 5. Toolshop Homework Connection

### 5.1 From Automation Testing Homework (HW06)

The Toolshop project uses CI/CD for automated test execution:

**Pipeline Configuration:**
```yaml
name: Automation Tests
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        browser: [chrome, firefox, edge]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run Selenium tests
        run: pytest tests/ --browser=${{ matrix.browser }}
      - uses: actions/upload-artifact@v4
        with:
          name: test-results-${{ matrix.browser }}
          path: reports/
```

**Key Integration Points:**
- Cross-browser testing via matrix strategy
- Artifact upload for test reports
- Parallel execution across 3 browsers
- Centralized result collection

### 5.2 From Performance Testing Homework (HW07)

Performance tests integrated into CI/CD:

```yaml
jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install JMeter
        run: |
          wget https://dlcdn.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
          tar -xzf apache-jmeter-5.6.3.tgz
      - name: Run load test
        run: ./apache-jmeter-5.6.3/bin/jmeter -n -t tests/loadtest.jmx -l results.jtl
      - name: Analyze results
        run: python scripts/analyze_perf.py results.jtl
```

---

## 6. Application-Level Exam Questions

### Question 1: Matrix Build Optimization
**Scenario:** Your team has a test suite that needs to run on 4 OS (Ubuntu, Windows, macOS, Alpine) × 5 Node versions (16, 18, 20, 22, 23). Full matrix = 20 jobs, taking 2 hours.

**Question:** How would you optimize this for faster feedback while maintaining coverage?

**Answer:**
```yaml
strategy:
  fail-fast: true
  matrix:
    os: [ubuntu-latest]
    node: [18, 20, 22]
    include:
      # Add critical combinations only
      - os: windows-latest
        node: 20
      - os: macos-latest
        node: 20
  # Total: 5 jobs instead of 20
  # Critical coverage maintained
  # Run full matrix on release branches only
```

**Key Points:**
- Use `fail-fast: true` for early failure detection
- Primary OS covers most cases
- Include specific combinations for cross-platform issues
- Schedule full matrix for nightly/release builds

### Question 2: Cache Strategy Design
**Scenario:** Your Next.js app has:
- npm dependencies (package-lock.json)
- Next.js build cache (.next/cache)
- Turbo monorepo cache

**Question:** Design a caching strategy to minimize build time.

**Answer:**
```yaml
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: node_modules
    key: deps-${{ hashFiles('package-lock.json') }}

- name: Cache Next.js
  uses: actions/cache@v4
  with:
    path: .next/cache
    key: next-${{ hashFiles('**/*.js', '**/*.jsx') }}
    restore-keys: next-

- name: Cache Turbo
  uses: actions/cache@v4
  with:
    path: .turbo
    key: turbo-${{ github.sha }}
    restore-keys: turbo-
```

**Key Points:**
- Separate caches for different purposes
- Use `hashFiles()` for content-based invalidation
- Use `restore-keys` for partial cache hits
- SHA-based key for turbo (incremental)

### Question 3: Debugging Failed Deployment
**Scenario:** Production deployment fails silently. Logs show success but users report errors.

**Question:** What debugging steps would you take?

**Answer:**
1. **Add smoke tests post-deployment:**
```yaml
- name: Smoke test
  run: |
    curl -f https://prod.example.com/health || exit 1
    curl -f https://prod.example.com/api/version | jq .
```

2. **Compare environment variables:**
```yaml
- name: Verify env
  run: |
    echo "Expected: v${{ github.sha }}"
    curl https://prod.example.com/api/version
```

3. **Check deployment logs:**
- Verify correct artifact deployed
- Check container startup logs
- Verify database migrations ran

4. **Add rollback automation:**
```yaml
- name: Rollback on failure
  if: failure()
  run: |
    kubectl rollout undo deployment/app
```

---

## 7. Cheatsheet / Quick Reference

### GitHub Actions Syntax

```yaml
# Workflow structure
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test

# Environment variables
env:
  NODE_ENV: test

# Secrets
${{ secrets.API_KEY }}

# Conditionals
if: github.event_name == 'push'
if: success()
if: failure()
if: always()

# Job dependencies
needs: [job1, job2]

# Matrix
strategy:
  matrix:
    node: [18, 20]

# Caching
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

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
