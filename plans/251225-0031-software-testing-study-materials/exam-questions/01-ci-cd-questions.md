# Topic 1: CI/CD Testing - Câu hỏi vận dụng
**CS423 Software Testing | Final Exam Practice**

---

## Câu 1: Matrix Build Optimization (10 điểm)

**Tình huống:** Team của bạn có test suite cần chạy trên:
- 4 OS: Ubuntu, Windows, macOS, Alpine
- 5 Node.js versions: 16, 18, 20, 22, 23
- Full matrix = 20 jobs, mất 2 giờ để hoàn thành

**Yêu cầu:**
a) Tính tổng số jobs nếu chạy full matrix (2 điểm)
b) Thiết kế matrix strategy tối ưu để giảm xuống còn ~5 jobs mà vẫn đảm bảo coverage (5 điểm)
c) Giải thích tại sao strategy của bạn vẫn đảm bảo chất lượng (3 điểm)

**Đáp án mẫu:**

a) **Full matrix:** 4 × 5 = 20 jobs

b) **Optimized matrix:**
```yaml
strategy:
  fail-fast: true
  matrix:
    os: [ubuntu-latest]
    node: [18, 20, 22]
    include:
      # Critical cross-platform combinations
      - os: windows-latest
        node: 20
      - os: macos-latest
        node: 20
  # Total: 5 jobs instead of 20
```

c) **Giải thích:**
- `fail-fast: true`: Dừng sớm khi có lỗi, tiết kiệm thời gian
- Ubuntu covers 90% use cases (Linux servers)
- Node 18, 20, 22 covers LTS versions (production relevant)
- Include Windows/macOS với Node 20 (LTS) để catch platform-specific bugs
- Full matrix chỉ chạy nightly/release builds

---

## Câu 2: Cache Strategy Design (10 điểm)

**Tình huống:** Next.js monorepo có:
- npm dependencies (package-lock.json)
- Next.js build cache (.next/cache)
- Turbo cache (.turbo)
- Playwright browsers (~500MB)

Build time hiện tại: 15 phút. Target: < 5 phút.

**Yêu cầu:**
a) Thiết kế multi-layer caching strategy (5 điểm)
b) Giải thích cache key design cho mỗi layer (3 điểm)
c) Xử lý cache invalidation như thế nào? (2 điểm)

**Đáp án mẫu:**

a) **Multi-layer caching:**
```yaml
# Layer 1: Dependencies (most stable)
- name: Cache node_modules
  uses: actions/cache@v4
  with:
    path: node_modules
    key: deps-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

# Layer 2: Build cache (changes with code)
- name: Cache Next.js
  uses: actions/cache@v4
  with:
    path: .next/cache
    key: next-${{ runner.os }}-${{ hashFiles('**/*.tsx', '**/*.ts') }}
    restore-keys: next-${{ runner.os }}-

# Layer 3: Turbo (incremental)
- name: Cache Turbo
  uses: actions/cache@v4
  with:
    path: .turbo
    key: turbo-${{ github.sha }}
    restore-keys: turbo-

# Layer 4: Playwright browsers (rarely changes)
- name: Cache Playwright
  uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: playwright-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
```

b) **Cache key design:**
- Dependencies: `hashFiles('package-lock.json')` - chỉ invalidate khi deps thay đổi
- Next.js: `hashFiles('**/*.tsx')` - invalidate khi source code thay đổi
- Turbo: `github.sha` - incremental per commit
- Playwright: Theo package-lock vì version định nghĩa ở đó

c) **Cache invalidation:**
- Dùng `restore-keys` để partial cache hits
- Version prefix: `key: v2-deps-...` để force invalidate
- TTL: GitHub auto-cleanup sau 7 ngày không dùng

---

## Câu 3: Debugging Production Deployment (10 điểm)

**Tình huống:** Production deployment thành công (logs show "Deployment complete") nhưng users báo lỗi 500. Bạn cần debug.

**Yêu cầu:**
a) Liệt kê 5 bước debug theo thứ tự ưu tiên (5 điểm)
b) Viết smoke test script để verify deployment (3 điểm)
c) Thiết kế rollback automation (2 điểm)

**Đáp án mẫu:**

a) **Debug steps (theo thứ tự):**
1. **Check health endpoint:** `curl -f https://prod.example.com/health`
2. **Verify deployed version:** Compare `git sha` với API response
3. **Check container logs:** `kubectl logs -f deployment/app`
4. **Verify environment variables:** So sánh staging vs production
5. **Check database migrations:** Verify migration status ran successfully

b) **Smoke test script:**
```yaml
- name: Post-deployment smoke test
  run: |
    # Health check
    curl -f https://prod.example.com/health || exit 1

    # Version verification
    DEPLOYED_SHA=$(curl -s https://prod.example.com/api/version | jq -r .sha)
    if [ "$DEPLOYED_SHA" != "${{ github.sha }}" ]; then
      echo "Version mismatch! Expected: ${{ github.sha }}, Got: $DEPLOYED_SHA"
      exit 1
    fi

    # Critical API check
    curl -f https://prod.example.com/api/products | jq '.data | length > 0' || exit 1

    echo "All smoke tests passed!"
```

c) **Rollback automation:**
```yaml
- name: Auto-rollback on failure
  if: failure()
  run: |
    echo "Deployment failed, initiating rollback..."
    kubectl rollout undo deployment/app

    # Wait for rollback
    kubectl rollout status deployment/app --timeout=5m

    # Notify team
    curl -X POST $SLACK_WEBHOOK -d '{"text":"Production rollback executed!"}'
```

---

## Câu 4: Secret Management (10 điểm)

**Tình huống:** CI/CD pipeline cần access:
- Database credentials
- API keys (Stripe, SendGrid)
- AWS credentials
- Private npm registry token

**Yêu cầu:**
a) Thiết kế secret management strategy (4 điểm)
b) Xử lý dynamic secrets (generated at runtime) (3 điểm)
c) Audit và rotation policy (3 điểm)

**Đáp án mẫu:**

a) **Secret management strategy:**
```yaml
# Organization secrets (shared across repos)
# - NPM_TOKEN
# - AWS_ACCESS_KEY_ID

# Repository secrets (repo-specific)
# - DATABASE_URL
# - STRIPE_SECRET_KEY

# Environment secrets (per environment)
# - PROD_DATABASE_URL (production environment)
# - STAGING_DATABASE_URL (staging environment)

jobs:
  deploy:
    environment: production  # Requires approval
    steps:
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Use secrets
        env:
          DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}
        run: npm run migrate
```

b) **Dynamic secrets handling:**
```yaml
- name: Generate dynamic token
  id: token
  run: |
    # Generate JWT for deployment
    TOKEN=$(python scripts/generate_deploy_token.py)
    echo "::add-mask::$TOKEN"  # Mask in logs
    echo "token=$TOKEN" >> $GITHUB_OUTPUT

- name: Use dynamic token
  env:
    DEPLOY_TOKEN: ${{ steps.token.outputs.token }}
  run: ./deploy.sh
```

c) **Audit và rotation:**
- **Audit**: Enable GitHub audit log, review monthly
- **Rotation policy**:
  - API keys: 90 days
  - Database passwords: 30 days
  - AWS credentials: Use IAM roles + OIDC instead
- **Alert**: Set up Dependabot secret scanning
- **Review**: Quarterly access review

---

## Câu 5: Multi-Environment Pipeline (10 điểm)

**Tình huống:** Cần deploy lên 4 environments:
1. Development (auto-deploy on push)
2. Staging (auto-deploy on PR merge)
3. UAT (manual approval required)
4. Production (manual approval + scheduled window)

**Yêu cầu:**
a) Thiết kế workflow với proper gates (5 điểm)
b) Implement environment-specific configurations (3 điểm)
c) Notification strategy (2 điểm)

**Đáp án mẫu:**

a) **Workflow với gates:**
```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [develop]
  pull_request:
    types: [closed]
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [uat, production]

jobs:
  # Development - Auto deploy
  deploy-dev:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: development
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh development

  # Staging - On PR merge to main
  deploy-staging:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh staging

  # UAT - Manual approval
  deploy-uat:
    if: github.event.inputs.environment == 'uat'
    runs-on: ubuntu-latest
    environment:
      name: uat
      url: https://uat.example.com
    steps:
      - uses: actions/checkout@v4
      - run: ./deploy.sh uat

  # Production - Manual + scheduled
  deploy-production:
    if: github.event.inputs.environment == 'production'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    # Only deploy during maintenance window (weekdays 2-4 AM)
    steps:
      - name: Check deployment window
        run: |
          HOUR=$(date +%H)
          DAY=$(date +%u)
          if [ $DAY -gt 5 ] || [ $HOUR -lt 2 ] || [ $HOUR -gt 4 ]; then
            echo "Outside deployment window!"
            exit 1
          fi
      - uses: actions/checkout@v4
      - run: ./deploy.sh production
```

b) **Environment-specific configs:**
```yaml
# Use environment files
- name: Load environment config
  run: |
    if [ "${{ github.event.inputs.environment }}" == "production" ]; then
      cp .env.production .env
    else
      cp .env.${{ github.event.inputs.environment }} .env
    fi

# Or use environment secrets
env:
  API_URL: ${{ vars.API_URL }}  # Environment variable
  API_KEY: ${{ secrets.API_KEY }}  # Environment secret
```

c) **Notification strategy:**
```yaml
- name: Notify on success
  if: success()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "✅ Deployed to ${{ github.event.inputs.environment }}",
        "blocks": [
          {
            "type": "section",
            "text": {
              "type": "mrkdwn",
              "text": "*Deployment Successful*\n• Env: ${{ github.event.inputs.environment }}\n• SHA: ${{ github.sha }}\n• By: ${{ github.actor }}"
            }
          }
        ]
      }

- name: Notify on failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "❌ Deployment FAILED to ${{ github.event.inputs.environment }}! @oncall"
      }
```

---

## Câu 6: Docker Multi-Stage Build (10 điểm)

**Tình huống:** Node.js application có:
- Development dependencies: 500MB
- Production dependencies: 50MB
- Source code: 10MB
- Current image size: 1.2GB

Target: < 200MB production image

**Yêu cầu:**
a) Viết multi-stage Dockerfile (5 điểm)
b) Optimize layer caching (3 điểm)
c) Security hardening (2 điểm)

**Đáp án mẫu:**

a) **Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app

# Security: Non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy only necessary files
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

USER nextjs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

b) **Layer caching optimization:**
```dockerfile
# Order matters! Least changing → Most changing

# 1. System deps (rarely changes)
RUN apk add --no-cache libc6-compat

# 2. Package files (changes when deps update)
COPY package*.json ./

# 3. Install deps (cached if package.json unchanged)
RUN npm ci

# 4. Source code (changes frequently)
COPY . .

# 5. Build (runs on every code change)
RUN npm run build
```

c) **Security hardening:**
```dockerfile
# Use specific version, not latest
FROM node:20.10.0-alpine3.18

# Non-root user
USER nextjs

# Read-only filesystem
# (set at runtime: docker run --read-only)

# No shell access
RUN rm /bin/sh

# Scan for vulnerabilities
# docker scout cves myimage:latest

# Minimal base image
FROM gcr.io/distroless/nodejs20-debian12
```

---

## Câu 7: Flaky Test Handling (10 điểm)

**Tình huống:** Test suite có 500 tests, 10% flaky (pass/fail randomly). CI/CD fails intermittently.

**Yêu cầu:**
a) Identify flaky tests strategy (3 điểm)
b) Fix common flaky test causes (4 điểm)
c) CI/CD retry strategy (3 điểm)

**Đáp án mẫu:**

a) **Identify flaky tests:**
```yaml
# Run tests multiple times to identify flaky
- name: Detect flaky tests
  run: |
    for i in {1..5}; do
      npm test -- --json > results_$i.json
    done

    # Compare results
    python scripts/find_flaky.py results_*.json > flaky_tests.txt

# Quarantine flaky tests
- name: Run stable tests
  run: npm test -- --testPathIgnorePatterns="flaky/"

- name: Run flaky tests (allow failure)
  continue-on-error: true
  run: npm test -- --testPathPattern="flaky/"
```

b) **Fix common causes:**
```javascript
// Problem 1: Race conditions
// BAD
test('async data', async () => {
  fetchData();
  expect(data).toBeDefined(); // May fail
});

// GOOD
test('async data', async () => {
  await fetchData();
  expect(data).toBeDefined();
});

// Problem 2: Shared state
// BAD - tests affect each other
let counter = 0;
test('increment', () => { counter++; expect(counter).toBe(1); });

// GOOD - isolated state
beforeEach(() => { counter = 0; });

// Problem 3: Time-dependent
// BAD
expect(result.createdAt).toBe(new Date());

// GOOD
expect(result.createdAt).toBeInstanceOf(Date);
// Or use frozen time
jest.useFakeTimers().setSystemTime(new Date('2024-01-01'));

// Problem 4: Network dependencies
// Use test containers or mocks
jest.mock('./api', () => ({
  fetchData: () => Promise.resolve({ data: 'test' })
}));
```

c) **CI/CD retry strategy:**
```yaml
- name: Run tests with retry
  uses: nick-fields/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    retry_on: error
    command: npm test

# Or built-in Jest retry
# jest.config.js
module.exports = {
  testRetries: 2,
  reporters: [
    'default',
    ['jest-junit', { outputDirectory: 'reports' }]
  ]
};
```

---

## Câu 8: Artifact Management (10 điểm)

**Tình huống:** Pipeline generates:
- Test reports (HTML, 5MB)
- Coverage reports (lcov, 2MB)
- Build artifacts (Docker image, 500MB)
- Screenshots on failure (PNG, variable)

**Yêu cầu:**
a) Design artifact storage strategy (4 điểm)
b) Implement conditional artifact upload (3 điểm)
c) Artifact retention policy (3 điểm)

**Đáp án mẫu:**

a) **Artifact storage strategy:**
```yaml
# Small artifacts: GitHub Artifacts
- uses: actions/upload-artifact@v4
  with:
    name: test-reports
    path: reports/

# Large artifacts: External storage
- name: Push to S3
  run: |
    aws s3 cp coverage/ s3://artifacts-bucket/coverage/${{ github.sha }}/ --recursive

# Docker images: Container registry
- name: Push image
  run: |
    docker tag app:latest ghcr.io/${{ github.repository }}:${{ github.sha }}
    docker push ghcr.io/${{ github.repository }}:${{ github.sha }}
```

b) **Conditional upload:**
```yaml
# Always upload test reports
- uses: actions/upload-artifact@v4
  if: always()
  with:
    name: test-results
    path: test-results/

# Only upload screenshots on failure
- uses: actions/upload-artifact@v4
  if: failure()
  with:
    name: failure-screenshots
    path: screenshots/
    retention-days: 7

# Only upload coverage on main branch
- uses: actions/upload-artifact@v4
  if: github.ref == 'refs/heads/main'
  with:
    name: coverage
    path: coverage/
```

c) **Retention policy:**
```yaml
# Short retention for debug artifacts
- uses: actions/upload-artifact@v4
  with:
    name: debug-logs
    path: logs/
    retention-days: 3

# Medium retention for test reports
- uses: actions/upload-artifact@v4
  with:
    name: test-reports
    path: reports/
    retention-days: 30

# Long retention for release artifacts
- uses: actions/upload-artifact@v4
  if: startsWith(github.ref, 'refs/tags/')
  with:
    name: release-${{ github.ref_name }}
    path: dist/
    retention-days: 90

# Cleanup old artifacts
- name: Delete old artifacts
  uses: actions/github-script@v7
  with:
    script: |
      const artifacts = await github.rest.actions.listArtifactsForRepo({
        owner: context.repo.owner,
        repo: context.repo.repo,
      });

      const OLD_DAYS = 7;
      const cutoff = Date.now() - (OLD_DAYS * 24 * 60 * 60 * 1000);

      for (const artifact of artifacts.data.artifacts) {
        if (new Date(artifact.created_at) < cutoff) {
          await github.rest.actions.deleteArtifact({
            owner: context.repo.owner,
            repo: context.repo.repo,
            artifact_id: artifact.id,
          });
        }
      }
```

---

## Câu 9: Branch Protection & PR Workflow (10 điểm)

**Tình huống:** Team cần enforce:
- All PRs must pass CI
- At least 2 reviewers approval
- No direct push to main
- Auto-merge when ready

**Yêu cầu:**
a) Configure branch protection rules (4 điểm)
b) Design PR validation workflow (3 điểm)
c) Auto-merge strategy (3 điểm)

**Đáp án mẫu:**

a) **Branch protection rules (via GitHub API/UI):**
```yaml
# .github/branch-protection.yml (for reference)
branches:
  main:
    protection:
      required_status_checks:
        strict: true
        contexts:
          - build
          - test
          - lint
      required_pull_request_reviews:
        required_approving_review_count: 2
        dismiss_stale_reviews: true
        require_code_owner_reviews: true
      restrictions:
        users: []
        teams: [maintainers]
      enforce_admins: true
      allow_force_pushes: false
      allow_deletions: false
```

b) **PR validation workflow:**
```yaml
name: PR Validation

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Lint PR title
        uses: amannn/action-semantic-pull-request@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Check PR size
        run: |
          ADDITIONS=$(gh pr view ${{ github.event.number }} --json additions -q .additions)
          if [ $ADDITIONS -gt 500 ]; then
            echo "PR too large! Please split into smaller PRs."
            exit 1
          fi

      - name: Require linked issue
        run: |
          BODY=$(gh pr view ${{ github.event.number }} --json body -q .body)
          if ! echo "$BODY" | grep -qE "(Fixes|Closes|Resolves) #[0-9]+"; then
            echo "PR must reference an issue!"
            exit 1
          fi

  build:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

  test:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

c) **Auto-merge strategy:**
```yaml
name: Auto Merge

on:
  pull_request:
    types: [labeled]
  check_suite:
    types: [completed]

jobs:
  auto-merge:
    if: contains(github.event.pull_request.labels.*.name, 'automerge')
    runs-on: ubuntu-latest
    steps:
      - name: Wait for checks
        uses: fountainhead/action-wait-for-check@v1.1.0
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          checkName: build
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Auto merge
        uses: pascalgn/automerge-action@v0.15.6
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          MERGE_METHOD: squash
          MERGE_COMMIT_MESSAGE: pull-request-title
          MERGE_DELETE_BRANCH: true
```

---

## Câu 10: Monitoring & Alerting (10 điểm)

**Tình huống:** Cần monitor CI/CD pipeline health:
- Build success rate
- Average build time
- Flaky test rate
- Deployment frequency

**Yêu cầu:**
a) Collect metrics from pipeline (4 điểm)
b) Define SLOs và alert thresholds (3 điểm)
c) Dashboard design (3 điểm)

**Đáp án mẫu:**

a) **Collect metrics:**
```yaml
- name: Collect build metrics
  if: always()
  run: |
    # Push metrics to Prometheus/Grafana
    cat << EOF | curl -X POST http://pushgateway:9091/metrics/job/ci
    # HELP ci_build_duration_seconds Build duration
    # TYPE ci_build_duration_seconds gauge
    ci_build_duration_seconds{repo="${{ github.repository }}", branch="${{ github.ref_name }}", status="${{ job.status }}"} ${{ github.run_number }}

    # HELP ci_build_status Build status (1=success, 0=failure)
    # TYPE ci_build_status gauge
    ci_build_status{repo="${{ github.repository }}"} $([[ "${{ job.status }}" == "success" ]] && echo 1 || echo 0)
    EOF

- name: Track test results
  if: always()
  run: |
    TOTAL=$(jq '.numTotalTests' test-results.json)
    PASSED=$(jq '.numPassedTests' test-results.json)
    FAILED=$(jq '.numFailedTests' test-results.json)

    curl -X POST http://pushgateway:9091/metrics/job/tests -d "
    ci_tests_total $TOTAL
    ci_tests_passed $PASSED
    ci_tests_failed $FAILED
    "
```

b) **SLOs và alert thresholds:**
```yaml
# SLO Definitions
SLOs:
  build_success_rate:
    target: 95%
    window: 7 days
    alert_threshold: 90%

  build_duration_p95:
    target: 10 minutes
    window: 24 hours
    alert_threshold: 15 minutes

  deployment_frequency:
    target: 1/day
    window: 7 days
    alert_threshold: 3/week

  test_flake_rate:
    target: < 5%
    window: 7 days
    alert_threshold: 10%

# Alert rules (Prometheus)
groups:
  - name: ci-alerts
    rules:
      - alert: BuildSuccessRateLow
        expr: |
          sum(rate(ci_build_status{status="success"}[7d])) /
          sum(rate(ci_build_status[7d])) < 0.90
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Build success rate below 90%"
```

c) **Dashboard design (Grafana panels):**
```
┌─────────────────────────────────────────────────────────────┐
│                    CI/CD Health Dashboard                    │
├─────────────────┬─────────────────┬─────────────────────────┤
│  Build Success  │  Avg Build Time │  Deployment Frequency   │
│     95.2%       │    8m 32s       │      1.5/day           │
│   ✅ Target: 95%│  ✅ Target: 10m │   ✅ Target: 1/day     │
├─────────────────┴─────────────────┴─────────────────────────┤
│                  Build Duration Trend (7 days)               │
│  [Line chart showing build times over past week]            │
├─────────────────────────────────────────────────────────────┤
│                  Test Results Breakdown                      │
│  Passed: 485 | Failed: 10 | Flaky: 5 (1%)                  │
├─────────────────────────────────────────────────────────────┤
│                  Recent Deployments                          │
│  • prod: 2h ago by @john (v1.2.3) ✅                        │
│  • staging: 4h ago by @jane (v1.2.4-beta) ✅                │
│  • dev: 30m ago by @bob (feature-x) ✅                      │
└─────────────────────────────────────────────────────────────┘
```

---

*Tổng: 10 câu × 10 điểm = 100 điểm*
*Thời gian làm bài đề xuất: 120 phút*
