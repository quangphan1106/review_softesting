# Topic 4: GUI & Usability Testing - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 What is GUI Testing?

**Definition:** Testing the Graphical User Interface to ensure visual elements function correctly and appear as expected.

**Scope:**
- Visual appearance (colors, fonts, layout)
- Functional behavior (clicks, inputs, navigation)
- Responsiveness (different screen sizes)
- Accessibility (screen readers, keyboard navigation)

### 1.2 Types of GUI Testing

| Type | Purpose | Approach |
|------|---------|----------|
| **Manual Testing** | Exploratory, UX validation | Human testers |
| **Functional Testing** | UI behavior verification | Selenium, Playwright |
| **Visual Regression** | Detect unintended UI changes | Applitools, Percy |
| **Cross-browser** | Consistency across browsers | BrowserStack, Sauce Labs |
| **Responsive** | Mobile/tablet layouts | DevTools, physical devices |
| **Accessibility** | WCAG compliance | axe, Pa11y |

### 1.3 Visual Regression Testing Concept

```
┌──────────────────────────────────────────────────────────────────┐
│                    VISUAL REGRESSION WORKFLOW                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│   Baseline      New Build        Comparison       Result          │
│   ┌─────┐       ┌─────┐          ┌─────┐         ┌─────┐        │
│   │ 📷  │       │ 📷  │    →     │ 🔍  │    →    │ ✓/✗ │        │
│   └─────┘       └─────┘          └─────┘         └─────┘        │
│   (Known        (New             (Diff           (Pass/          │
│    Good)        Screenshot)       Analysis)       Fail)          │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Key Concepts:**
- **Baseline**: Known good state (approved screenshot)
- **Checkpoint**: New screenshot from current build
- **Diff**: Visual comparison algorithm
- **Threshold**: Acceptable difference percentage

### 1.4 Usability Testing Fundamentals

**Nielsen's 10 Usability Heuristics:**

1. **Visibility of System Status** - User knows what's happening
2. **Match Real World** - Familiar language/concepts
3. **User Control & Freedom** - Undo, redo, escape
4. **Consistency & Standards** - Follow platform conventions
5. **Error Prevention** - Prevent mistakes before they happen
6. **Recognition vs Recall** - Make options visible
7. **Flexibility & Efficiency** - Shortcuts for experts
8. **Aesthetic & Minimalist** - No irrelevant information
9. **Error Recovery** - Clear error messages with solutions
10. **Help & Documentation** - Easy to find when needed

---

## 2. Tools & Techniques Comparison

### 2.1 Visual Regression Testing Tools

| Tool | AI Capability | Integration | Pricing | Best For |
|------|--------------|-------------|---------|----------|
| **Applitools Eyes** | Advanced Visual AI | All frameworks | $969+/mo | Enterprise, AI-powered |
| **Percy (BrowserStack)** | Smart snapshot | CI/CD native | Custom | DevOps teams |
| **Chromatic** | Storybook native | Component-level | Free tier | React/Vue components |
| **BackstopJS** | None (pixel-diff) | Self-hosted | Free | Simple projects |
| **Playwright** | Built-in screenshots | Native | Free | Playwright users |

### 2.2 Applitools Eyes Deep Dive

**Architecture:**
```
Test Code → SDK → Eyes Server → Visual AI → Results Dashboard
     ↓
  Screenshot
  captured
     ↓
  Uploaded
     ↓
  AI analysis
     ↓
  Human review
```

**Key Features:**
- **Visual AI**: 10+ years training on 4B screens
- **Ultrafast Grid**: Run once, render on 70+ browser/device combos
- **Root Cause Analysis**: Pinpoints exact CSS/DOM changes
- **Smart Content Handling**: Ignores dynamic content (ads, timestamps)

**AI vs Pixel-Perfect Comparison:**
```
Pixel-Perfect:
- Exact pixel match required
- False positives from anti-aliasing
- Fails on font rendering differences
- High maintenance burden

Visual AI (Applitools):
- Human-like perception
- Ignores rendering noise
- Detects meaningful changes
- 78% reduced maintenance (Peloton case study)
```

### 2.3 Cross-Browser Testing Strategies

**Traditional Approach:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Chrome    │     │   Firefox   │     │   Safari    │
│   Test      │     │   Test      │     │   Test      │
│   (5 min)   │     │   (5 min)   │     │   (5 min)   │
└─────────────┘     └─────────────┘     └─────────────┘
Total: 15 minutes + infrastructure cost
```

**Ultrafast Grid Approach:**
```
┌─────────────┐     ┌──────────────────────────────────┐
│   Local     │ →   │        Ultrafast Grid            │
│   Test      │     │  ┌─────┐ ┌─────┐ ┌─────┐        │
│   (5 min)   │     │  │ Chr │ │ FF  │ │ Saf │  ...   │
└─────────────┘     │  └─────┘ └─────┘ └─────┘        │
                    │       (30 seconds total)          │
                    └──────────────────────────────────┘
Total: 5.5 minutes, no infrastructure
```

---

## 3. Practical Test Case Design

### 3.1 Visual Regression Test Implementation

**Playwright + Applitools Example:**
```javascript
const { test } = require('@playwright/test');
const { Eyes, Target, Configuration, BatchInfo } = require('@applitools/eyes-playwright');

let eyes;

test.beforeAll(async () => {
  eyes = new Eyes();
  const config = new Configuration();
  config.setBatch(new BatchInfo('Toolshop Visual Tests'));
  config.setApiKey(process.env.APPLITOOLS_API_KEY);
  eyes.setConfiguration(config);
});

test('homepage visual check', async ({ page }) => {
  await eyes.open(page, 'Toolshop', 'Homepage', { width: 1920, height: 1080 });
  await page.goto('https://practicesoftwaretesting.com');

  // Full page screenshot
  await eyes.check('Homepage', Target.window().fully());

  // Specific region
  await eyes.check('Navigation', Target.region('#nav'));

  // Ignore dynamic content
  await eyes.check('Products', Target.region('.products')
    .ignoreRegions('.price', '.timestamp'));

  await eyes.close();
});

test.afterAll(async () => {
  await eyes.abort();
});
```

**Percy + Playwright Example:**
```javascript
const { test } = require('@playwright/test');
const percySnapshot = require('@percy/playwright');

test('homepage visual check', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  // Take Percy snapshot
  await percySnapshot(page, 'Homepage');

  // With options
  await percySnapshot(page, 'Homepage Mobile', {
    widths: [375, 768, 1280],
    percyCSS: '.dynamic-content { visibility: hidden; }'
  });
});
```

### 3.2 Accessibility Testing

**axe-core Integration:**
```javascript
const { test, expect } = require('@playwright/test');
const AxeBuilder = require('@axe-core/playwright').default;

test('accessibility check', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa'])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

**Common WCAG Violations:**
| Issue | Impact | Fix |
|-------|--------|-----|
| Missing alt text | Images inaccessible | Add descriptive alt |
| Low contrast | Hard to read | Increase contrast ratio |
| No focus indicator | Keyboard nav broken | Add :focus styles |
| Missing labels | Form controls unclear | Add label elements |

### 3.3 Test Cases for GUI Testing

**TC-GUI-001: Visual Regression Baseline**
- **Objective**: Establish baseline for homepage
- **Steps**: Navigate to homepage, capture screenshot
- **Expected**: Screenshot saved as baseline
- **Tool**: Applitools/Percy

**TC-GUI-002: Cross-Browser Consistency**
- **Objective**: Verify UI consistent across browsers
- **Steps**: Run visual test on Chrome, Firefox, Safari
- **Expected**: No visual differences
- **Tool**: Ultrafast Grid

**TC-GUI-003: Responsive Design**
- **Objective**: Verify mobile layout
- **Steps**: Test at 375px, 768px, 1024px widths
- **Expected**: Layout adapts correctly
- **Tool**: Playwright viewport setting

**TC-GUI-004: Accessibility Compliance**
- **Objective**: WCAG 2.1 AA compliance
- **Steps**: Run axe-core audit
- **Expected**: Zero critical violations
- **Tool**: axe-core

**TC-GUI-005: Dynamic Content Handling**
- **Objective**: Ignore timestamps, ads in comparison
- **Steps**: Configure ignore regions, run visual test
- **Expected**: Only meaningful changes flagged
- **Tool**: Applitools ignore regions

---

## 4. Troubleshooting Scenarios

### 4.1 Common Visual Testing Issues

**Problem 1: False Positives from Anti-Aliasing**
```
Symptom: Same content flagged as different
Root Cause: Font rendering differs across OS/browsers

Solutions:
1. Use Visual AI (Applitools) - handles automatically
2. Set threshold: `matchLevel: 'Strict'` vs `'Layout'`
3. Use CSS font smoothing: `-webkit-font-smoothing: antialiased`
```

**Problem 2: Flaky Screenshots**
```
Symptom: Tests fail intermittently
Root Causes:
- Animations not complete
- Lazy-loaded images
- Dynamic content (ads, timestamps)

Solutions:
1. Wait for animations:
   await page.waitForFunction(() => !document.querySelector('.animating'))

2. Wait for images:
   await page.waitForFunction(() =>
     Array.from(document.images).every(img => img.complete)
   )

3. Ignore dynamic regions:
   await eyes.check('Page', Target.window()
     .ignoreRegions('.ad-banner', '.timestamp'))
```

**Problem 3: Mobile Responsiveness Failures**
```
Symptom: Layout broken at specific breakpoints
Root Causes:
- Missing media queries
- Fixed width elements
- Viewport meta tag missing

Debug Steps:
1. Test at exact breakpoints (375, 768, 1024, 1280)
2. Check CSS media queries
3. Verify viewport: <meta name="viewport" content="width=device-width">
4. Use Chrome DevTools Device Mode
```

### 4.2 Usability Issue Identification

**Heuristic Evaluation Process:**
```
1. Select 3-5 evaluators (Nielsen's recommendation)
2. Each evaluates against 10 heuristics
3. Document issues with severity rating:
   - 0: Not a problem
   - 1: Cosmetic
   - 2: Minor
   - 3: Major
   - 4: Catastrophic
4. Consolidate findings
5. Prioritize fixes by severity × frequency
```

**Example Issues Found:**
```
Issue: Form submission with no feedback
Heuristic Violated: #1 (Visibility of System Status)
Severity: 3 (Major)
Recommendation: Add loading spinner and success message

Issue: No undo for delete action
Heuristic Violated: #3 (User Control & Freedom)
Severity: 4 (Catastrophic)
Recommendation: Add confirmation dialog or soft delete
```

---

## 5. Toolshop Homework Connection

### 5.1 GUI Testing Applied to Toolshop

**Visual Regression Test Scenarios:**
```javascript
// Homepage visual test
test('toolshop homepage visual', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');
  await page.waitForLoadState('networkidle');

  // Full page
  await expect(page).toHaveScreenshot('homepage.png', {
    fullPage: true,
    animations: 'disabled'
  });
});

// Product card consistency
test('product card visual', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');
  const productCard = page.locator('.product-card').first();

  await expect(productCard).toHaveScreenshot('product-card.png');
});
```

### 5.2 Usability Issues Found in Toolshop

| Page | Issue | Heuristic | Severity |
|------|-------|-----------|----------|
| Login | No password visibility toggle | #7 Flexibility | 2 |
| Cart | No quantity update feedback | #1 Visibility | 3 |
| Checkout | Error messages unclear | #9 Error Recovery | 3 |
| Search | No "no results" message | #1 Visibility | 2 |
| Product | Image doesn't zoom | #7 Flexibility | 1 |

**Recommended Fixes:**
1. Add password show/hide button
2. Add "Item updated" toast notification
3. Rewrite error messages with specific guidance
4. Add "No products found" with suggestions
5. Implement image zoom on hover/click

---

## 6. Application-Level Exam Questions

### Question 1: Visual Testing Strategy
**Scenario:** E-commerce site with 200 pages needs visual regression testing in CI/CD. Budget is limited.

**Question:** Design a visual testing strategy balancing coverage and cost.

**Answer:**
```
Tiered Strategy:

Tier 1: Critical Pages (Every PR)
- Homepage, Login, Checkout, Cart
- Use Applitools AI (avoid false positives)
- Run on primary browser (Chrome) only

Tier 2: Important Pages (Daily)
- Product listing, Search results, Account
- Run on Chrome, Firefox, Safari
- Use Ultrafast Grid

Tier 3: All Pages (Weekly/Release)
- Full site coverage
- All browser/device combinations
- Schedule during off-hours

Cost Optimization:
- Use layout match level (not strict pixel)
- Batch similar pages together
- Ignore known dynamic regions
- Review and approve baselines weekly
```

### Question 2: Handling Dynamic Content
**Scenario:** Dashboard has real-time data (prices, timestamps, user avatars) that changes constantly.

**Question:** How would you implement visual testing?

**Answer:**
```javascript
// Strategy: Ignore dynamic, test layout

// 1. Define ignore regions
await eyes.check('Dashboard', Target.window()
  .ignoreRegions('.real-time-price', '.timestamp', '.user-avatar')
  .layout()  // Focus on layout, not exact content
);

// 2. Use data-test attributes for stable regions
await eyes.check('Charts', Target.region('[data-test="chart-container"]'));

// 3. Mock dynamic data for consistency
await page.route('**/api/prices', route => {
  route.fulfill({
    body: JSON.stringify({ price: 100.00 })
  });
});

// 4. Freeze animations
await page.evaluate(() => {
  document.querySelectorAll('*').forEach(el => {
    el.style.animation = 'none';
    el.style.transition = 'none';
  });
});
```

### Question 3: Accessibility Testing Priority
**Scenario:** Limited time to fix accessibility issues before audit.

**Question:** Prioritize these violations:
- A: Missing form labels (5 instances)
- B: Low color contrast on footer (1 instance)
- C: No skip-to-content link (1 instance)
- D: Images without alt text (20 instances)

**Answer:**
```
Priority Order (WCAG severity + user impact):

1. D: Images without alt text (Critical)
   - 20 instances = high frequency
   - Screen reader users completely blocked
   - WCAG Level A violation

2. A: Missing form labels (Critical)
   - 5 instances on forms = high impact
   - Forms unusable for screen reader users
   - WCAG Level A violation

3. C: No skip-to-content link (Major)
   - Keyboard users must tab through nav repeatedly
   - WCAG Level A violation
   - Single fix affects all pages

4. B: Low color contrast footer (Minor)
   - Only footer affected
   - WCAG Level AA violation (less critical)
   - Visual users can still see (just harder)

Quick wins: Fix labels and alt text with tools
Long-term: Implement design system with accessibility built-in
```

---

## 7. Cheatsheet / Quick Reference

### Applitools Configuration

```javascript
// Configuration options
const config = new Configuration();
config.setApiKey(process.env.APPLITOOLS_API_KEY);
config.setBatch(new BatchInfo('Batch Name'));
config.setMatchLevel(MatchLevel.Strict);  // or Layout, Content, Exact
config.setIgnoreCaret(true);
config.setHideCaret(true);
config.setHideScrollbars(true);

// Check options
await eyes.check('Name', Target.window()
  .fully()                    // Full page
  .layout()                   // Layout match level
  .strict()                   // Strict match level
  .ignoreRegions('.dynamic')  // Ignore specific regions
  .floatingRegion('.popup', 10, 10, 10, 10)  // Allow movement
);
```

### Playwright Visual Testing

```javascript
// Built-in screenshot comparison
await expect(page).toHaveScreenshot('name.png', {
  fullPage: true,
  animations: 'disabled',
  mask: [page.locator('.dynamic')],
  maxDiffPixels: 100,  // Allow small differences
  threshold: 0.2       // 20% difference threshold
});

// Element screenshot
await expect(locator).toHaveScreenshot('element.png');

// Update baselines
// npx playwright test --update-snapshots
```

### Accessibility Testing Commands

```bash
# Run axe-core
npx axe https://example.com

# Lighthouse accessibility audit
npx lighthouse https://example.com --only-categories=accessibility

# Pa11y CLI
npx pa11y https://example.com
```

### Usability Heuristics Quick Reference

| # | Heuristic | Key Question |
|---|-----------|--------------|
| 1 | Visibility | Does user know system status? |
| 2 | Match Real World | Is language familiar? |
| 3 | User Control | Can user undo/escape? |
| 4 | Consistency | Does it follow standards? |
| 5 | Error Prevention | Are mistakes prevented? |
| 6 | Recognition | Are options visible? |
| 7 | Flexibility | Are there shortcuts? |
| 8 | Aesthetic | Is design minimal? |
| 9 | Error Recovery | Are errors clear/fixable? |
| 10 | Help | Is help available? |

---

## 8. Key Takeaways for Exam

1. **Visual AI**: Applitools > pixel-diff (78% less maintenance)
2. **Ultrafast Grid**: Run once, render on 70+ combos
3. **False Positives**: Use layout match level for dynamic content
4. **Accessibility**: WCAG 2.1 Level AA is standard target
5. **Heuristics**: Know Nielsen's 10 for usability evaluation
6. **Responsive**: Test at 375, 768, 1024, 1280 breakpoints
7. **Dynamic Content**: Use ignore regions, mock data, disable animations

---

*Study time recommended: 2-3 hours*
*Practice: Set up visual regression testing for a sample project*
