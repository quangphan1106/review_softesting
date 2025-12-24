# Software Testing Research Report: Topics 1-4
**Research Date:** 2025-12-25 | **Level:** Application/Practical | **Max Lines:** ~150 per topic

---

## 1. CI/CD Testing

### GitHub Actions Advanced Patterns

**Matrix Builds:**
- Use `include/exclude` strategically to avoid hardcoding combinations
- Implement upstream jobs to determine downstream job sets (e.g., path-based triggers)
- Control fail-fast behavior: `fail-fast: true` (default) cancels all jobs on failure; set `false` for full test coverage
- Pattern: Dynamic matrix outputs from one job feed into another job's matrix definition

**Caching Best Practices:**
- Use `hashFiles('package-lock.json')` to auto-invalidate cache when dependencies change
- Adopt composite cache keys: `${{ runner.os }}-turbo-${{ matrix.task }}-${{ github.head_ref || github.ref_name }}`
- Fallback strategy with `restore-keys` prevents total miss when exact hash doesn't match
- **2025 Update:** GitHub migrated cache service Feb 1, 2025—upgrade to `actions/cache@v4` or `v3` ASAP
- Only cache what matters: selective paths reduce storage overhead and restore time

**Artifacts:**
- Use `retention-days` to auto-cleanup old artifacts (default 90 days)
- Upload only necessary artifacts; avoid caching everything
- Consider caching compiled binaries to skip rebuild in subsequent runs
- Use `needs: [job-name]` for explicit job dependencies and parallel execution control

### Docker in CI/CD

**Integration Patterns:**
- Docker-in-Docker (DinD) requires careful layer caching config to avoid redundant builds
- Decouples build/test from host OS—eliminates "works on my machine" failures
- Container registries must be monitored for pull latency and error rates

**Common Failures & Debugging:**
- Transient failures: Integration tests depend on infrastructure state (Kubernetes cluster reconciliation)
- Use distributed tracing (Jaeger, Zipkin) to debug multi-service flows
- Attach ephemeral debug containers to inspect failing environments
- Log aggregation (ELK/OpenSearch/Splunk) with pipeline run IDs crucial for root cause analysis
- Resource constraints: Allocate ≥2GB RAM, 10GB disk for tracing tools

**2025 Trends:**
- AI-driven pipelines predict failures; ML subset selection (e.g., Harness) runs only relevant tests
- Real-time Slack/email alerts with verbose logging at every step

---

## 2. Performance Testing

### Tools Comparison Table

| Tool | Language | Architecture | Best For | Key Strength |
|------|----------|-------------|----------|------------|
| **JMeter** | Java/GUI | Thread-based | Wide protocol support (HTTP, JDBC, LDAP, JMS, FTP) | Legacy/enterprise systems |
| **k6** | JavaScript/CLI | Event-based | Modern APIs, CI/CD pipelines | 20+ integrations (Grafana, Datadog, etc.) |
| **Gatling** | Scala/Java | Akka-based | Massive user loads from single machine | Optimized efficiency, detailed HTML reports |
| **Locust** | Python | Event-based, distributed | Python teams, simple tests | Master-worker scaling to millions of users |

### Key Metrics & Response Time Pitfalls

**Response Time Measurement Differences:**
- JMeter/Gatling report values **close to browser timings**
- Locust reports **significantly higher** (TCP/TLS handling differences)
- k6 **under-reports unless combined metrics** used
- **Never compare raw numbers across tools**—understand their timing definitions first

**Think Time & Pacing:**
- Realistic user behavior: add think time between requests
- Pacing strategies prevent connection pool saturation
- k6: Use `sleep()` between requests; Locust: `wait_time` in task sequences

### Real-World Benchmarks

- **k6 resource efficiency:** Requires far fewer resources than JMeter for equivalent load
- **Gatling throughput:** 300k+ req/sec with efficient Akka architecture
- **Locust distributed:** Can distribute across machines for truly massive loads
- **CI/CD winner:** k6 (cloud integration) > Gatling (enterprise) > Locust (free) > JMeter (resource-hungry)

---

## 3. Automation Testing (Web + Desktop)

### Selenium vs Playwright vs Cypress (2025)

| Aspect | Selenium | Playwright | Cypress |
|--------|----------|-----------|---------|
| **Speed** | Slower (WebDriver) | Fastest (DevTools Protocol) | Fast (runs in browser) |
| **Browsers** | All (IE, Safari, etc.) | Chromium, Firefox, WebKit | Chrome-family + Firefox |
| **Languages** | Java, Python, C#, Ruby, JS | Java, Python, JS, .NET | JS/TS only |
| **Stability** | Explicit waits (flaky) | Auto-wait (40% less flaky) | Auto-wait, native handling |
| **Architecture** | External WebDriver API | Direct browser protocol | Runs inside browser |
| **Debugging** | Standard tools | Trace viewer, inspector | Time-travel debugging |
| **CI/CD** | All pipelines | GitHub Actions, Azure, Docker | GitHub, Bitbucket, GitLab |
| **Mobile** | Appium integration | Emulation only | Responsive emulation |

### Advanced Patterns & Anti-Patterns

**Page Object Model (POM) 2025:**
- Encapsulate selectors in POM classes to reduce maintenance
- Use fluent APIs for chainable interactions: `page.visit().login().search()`
- Avoid storing element references; always query fresh to prevent stale element exceptions

**Handling Dynamic Elements:**
- Playwright/Cypress auto-wait; Selenium requires explicit `WebDriverWait(driver, 10)`
- iframes: Playwright/Cypress require `.frameLocator()` or `.frames[0]` navigation
- Captchas: Use `playwright.waitForPopup()` for new windows; for image captchas, consider integration with OCR or manual bypass in test env
- Shadow DOM: Playwright has `.locator()` pierce, Selenium requires JS execution

**Test Flakiness Solutions:**
- Wait for network idle instead of hard sleeps: `.waitForLoadState('networkidle')`
- Retry logic for transient failures (external API delays)
- Run tests in parallel with isolated data to avoid resource contention

### Desktop Automation (Pywinauto)

**When to use:** Windows GUI automation (legacy apps, native dialogs)
- Pattern: `app = Application().connect(path='notepad.exe')` → interact with windows
- Limitation: Windows-only; macOS/Linux require AutoIt or native solutions
- Real-world: Testing dialog flows, file saves, system integrations

---

## 4. GUI & Usability Testing

### Visual Regression Testing Tools

| Tool | AI Capability | Integration | Pricing | Best For |
|------|--------------|-------------|---------|----------|
| **Applitools Eyes** | Smart pixel-diff (ignores dynamic content) | Selenium, Playwright, Cypress, Appium | $969+/mo | AI-powered, cross-browser rendering |
| **Percy (BrowserStack)** | AI visual testing | Works with CI/CD natively | Custom | Seamless CI integration, UI regression |
| **Pixel-diff tools** | None (strict comparison) | Manual integration | Free | Pixel-perfect validation |

### Applitools AI Capabilities (2025)

- **Visual AI:** Replicates "human eye"—ignores rendering differences, font shifts, dynamic content
- **Ultrafast Grid:** Run tests once locally, instant render across browser/device/viewport matrix
- **Automated Maintenance:** Groups similar differences across test suite; verify multiple checkpoints at once
- **Smart Content Handling:** Distinguishes true regressions from expected changes (timestamp updates, etc.)
- **Framework Coverage:** Integrates with Selenium, Cypress, Playwright, TestCafe, Appium, PDFs/images

### Cross-Browser Testing Strategies

**Traditional Approach:**
- Test on multiple OS/browser combinations locally—slow, expensive, infrastructure-heavy

**Modern Approach (2025):**
- Single test run locally → instant rendering on Ultrafast Grid (seconds vs minutes)
- Run once, test on 20+ browser/device combos automatically
- Better ROI than maintaining test lab or using legacy services

**Mobile Testing:**
- Playwright/Cypress: Emulate mobile viewports, responsive layout checks only
- True device testing: Requires Appium (Android/iOS) or cloud services (BrowserStack, SauceLabs)
- Usability heuristics: Test touch targets (48px minimum), readability on small screens, portrait/landscape

### Usability Testing for Test Design

**Heuristics:**
- Visibility: User can identify page state without guessing (visual feedback on actions)
- Control: User can undo/cancel actions; no traps
- Learnability: New users don't struggle with basic tasks
- Error prevention: Confirm destructive actions; validate input early
- Performance: Page loads in <3s, interactions instant

**Test Design Application:**
- Verify error messages are visible and actionable
- Test form validation on invalid data (email format, password strength)
- Confirm loading spinners appear during async operations
- Check accessibility: ARIA labels, keyboard navigation, color contrast (WCAG 2.1 AA)

---

## Summary: Choosing Tools for CS423 Exam Scenarios

| Scenario | Tool Choice | Why |
|----------|------------|-----|
| Need matrix tests across 5 OSes | GitHub Actions + Playwright | Native matrix, fast execution |
| Compare response time under load | k6 vs Gatling | Accurate metrics, compare carefully |
| Legacy browser support required | Selenium + Appium | Multi-language, 20+ year track record |
| UI regression in CI/CD | Applitools + Playwright | AI ignores fluff, fast feedback |
| Windows GUI automation | Pywinauto | Only Windows-capable option |
| Flaky test debugging | Playwright + trace viewer | Time-travel debugging, exact timeline |

---

## Research Sources

- [GitHub Actions Matrix Build Optimization - SmartScope](https://smartscope.blog/en/Infrastructure/github-actions-matrix-build-optimization/)
- [GitHub Actions Advanced Workflows - LearnXOps](https://www.learnxops.com/github-actions-advanced-workflows/)
- [GitHub Actions Cache Repository](https://github.com/actions/cache)
- [Top 5 Load Testing Tools 2025 - TestLeaf](https://www.testleaf.com/blog/5-best-load-testing-tools-in-2025/)
- [k6 vs Other Performance Testing Tools - QACRAFT](https://qacraft.com/k6-vs-other-performance-testing-tools/)
- [Playwright vs Selenium vs Cypress: Best 2025 Comparison - ThinkSys](https://thinksys.com/qa-testing/playwright-vs-selenium-vs-cypress/)
- [Cypress vs Playwright vs Selenium: Which Is Best for 2025 - Royal Cyber](https://www.royalcyber.com/blogs/test-automation/cypress-vs-playwright-vs-selenium/)
- [Applitools Eyes - AI-Powered Visual Testing](https://applitools.com/platform/eyes/)
- [What is Visual Regression Testing - Applitools](https://applitools.com/blog/visual-regression-testing/)
- [How to Debug CI/CD Pipelines - FreeCodeCamp](https://www.freecodecamp.org/news/how-to-debug-cicd-pipelines-handbook/)
- [Docker in CI/CD Pipelines Guide - MOSS](https://moss.sh/deployment/docker-in-ci-cd-pipelines-guide/)
- [Why Your CI/CD Pipeline is Failing - CodeGenitor/Medium](https://medium.com/technology-hits/why-your-ci-cd-pipeline-is-failing-and-how-to-fix-it-cd6b452ebe5d/)

---

## Unresolved Questions for Further Study

1. **Think time algorithms:** How do realistic think times vary by user persona (power user vs casual)? Any industry benchmarks?
2. **Flakiness root causes:** Beyond network/timing, what causes Selenium flakiness vs Playwright in identical scenarios?
3. **Applitools pricing ROI:** At $969+/mo, when does visual regression justify cost vs manual cross-browser testing?
4. **Mobile automation trade-off:** When does Appium overhead (setup, infrastructure) justify value vs responsive design testing alone?
5. **CI/CD failure patterns:** Frequency data on Docker vs code issues—what % of pipeline failures are environmental vs test bugs?
