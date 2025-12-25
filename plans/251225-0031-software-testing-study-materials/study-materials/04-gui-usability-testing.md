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

**Applitools Concepts:**
- Setup: Create `Eyes`, configure batch, API key
- Open: `eyes.open(page, 'App', 'Test', {width, height})`
- Check: `eyes.check('Name', Target.window().fully())`
- Region: `Target.region('#nav')`
- Ignore dynamic: `.ignoreRegions('.price', '.timestamp')`
- Close: `eyes.close()` / `eyes.abort()`

**Percy Concepts:**
- Snapshot: `percySnapshot(page, 'Name')`
- Multi-width: `widths: [375, 768, 1280]`
- Hide dynamic: `percyCSS: '.dynamic { visibility: hidden; }'`

### 3.2 Accessibility Testing

**axe-core Concepts:**
- Create: `new AxeBuilder({ page })`
- Filter: `.withTags(['wcag2a', 'wcag2aa'])`
- Run: `.analyze()` returns violations array
- Assert: `expect(results.violations).toEqual([])`

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

| Problem | Symptom | Solutions |
|---------|---------|-----------|
| **Anti-Aliasing False Positives** | Same content flagged different | Use Visual AI, set `matchLevel: 'Layout'` |
| **Flaky Screenshots** | Intermittent failures | Wait for animations/images, ignore dynamic regions |
| **Responsive Failures** | Layout broken at breakpoints | Test 375/768/1024/1280, check media queries, verify viewport meta |

**Key Fixes:**
- Wait for animations: `waitForFunction(() => !document.querySelector('.animating'))`
- Ignore dynamic: `.ignoreRegions('.ad-banner', '.timestamp')`

### 4.2 Usability Issue Identification

**Heuristic Evaluation:**
- 3-5 evaluators, each against 10 heuristics
- Severity: 0 (not a problem) → 4 (catastrophic)
- Prioritize: severity × frequency

**Example Issues:**
| Issue | Heuristic | Severity | Fix |
|-------|-----------|----------|-----|
| No form feedback | #1 Visibility | 3 (Major) | Add spinner + success msg |
| No undo for delete | #3 User Control | 4 (Catastrophic) | Confirmation or soft delete |

---

## 5. Toolshop Homework Connection

### 5.1 GUI Testing Applied to Toolshop

**Visual Test Concepts:**
- Wait for load: `page.waitForLoadState('networkidle')`
- Full page: `expect(page).toHaveScreenshot('name.png', {fullPage: true})`
- Disable animations: `{animations: 'disabled'}`
- Element screenshot: `expect(locator).toHaveScreenshot()`

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
**Scenario:** 200-page e-commerce site, limited budget

**Answer - Tiered Strategy:**
| Tier | Pages | Frequency | Browsers |
|------|-------|-----------|----------|
| 1 (Critical) | Homepage, Login, Checkout, Cart | Every PR | Chrome only |
| 2 (Important) | Product list, Search, Account | Daily | Chrome, FF, Safari |
| 3 (All) | Full site | Weekly/Release | All combos |

**Cost Optimization:** Layout match level, batch pages, ignore dynamic, weekly baseline review

### Question 2: Handling Dynamic Content
**Scenario:** Dashboard with real-time prices, timestamps, avatars

**Answer - Strategies:**
1. Ignore regions: `.ignoreRegions('.price', '.timestamp')`
2. Layout match: `.layout()` (focus on structure, not content)
3. Mock data: `page.route('**/api/prices', mockResponse)`
4. Freeze animations: Set `style.animation = 'none'`

### Question 3: Accessibility Testing Priority
**Scenario:** Limited time, fix order for violations

**Answer:**
| Priority | Issue | Reason |
|----------|-------|--------|
| 1 | D: Images no alt (20) | High frequency, blocks screen readers, WCAG A |
| 2 | A: Missing labels (5) | Forms unusable, WCAG A |
| 3 | C: No skip link (1) | Affects all pages, WCAG A |
| 4 | B: Low contrast footer (1) | Minor impact, WCAG AA |

---

## 7. Cheatsheet / Quick Reference

### Applitools Configuration

| Config | Purpose |
|--------|---------|
| `setApiKey()` | Authentication |
| `setBatch()` | Group tests |
| `setMatchLevel(Strict/Layout/Content)` | Comparison mode |
| `Target.window().fully()` | Full page |
| `.ignoreRegions('.dynamic')` | Ignore areas |
| `.floatingRegion('.popup', 10, 10, 10, 10)` | Allow movement |

### Playwright Visual Testing

| Option | Purpose |
|--------|---------|
| `fullPage: true` | Capture entire page |
| `animations: 'disabled'` | Freeze animations |
| `mask: [locator]` | Hide elements |
| `maxDiffPixels: 100` | Allow small differences |
| `--update-snapshots` | Update baselines |

### Accessibility Commands

| Tool | Command |
|------|---------|
| axe-core | `npx axe https://example.com` |
| Lighthouse | `npx lighthouse URL --only-categories=accessibility` |
| Pa11y | `npx pa11y https://example.com` |

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
