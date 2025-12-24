# Topic 4: GUI & Usability Testing - Application Questions
**CS423 Software Testing | Final Exam Practice**

---

## Hướng dẫn
- 10 câu hỏi x 10 điểm = 100 điểm
- Mỗi câu hỏi ở mức **vận dụng** (application level)
- Tập trung vào scenario thực tế, thiết kế test, troubleshooting

---

## Câu 1: Visual Regression Strategy Design (10 điểm)

### Tình huống
Công ty bạn có một ứng dụng e-commerce với 300 trang, cần triển khai visual regression testing. Yêu cầu:
- CI/CD pipeline chạy mỗi PR
- Budget hạn chế ($500/tháng cho tools)
- Team có 2 QA engineers
- Cần test trên 5 browsers (Chrome, Firefox, Safari, Edge, Mobile Chrome)

### Yêu cầu
a) Thiết kế chiến lược visual testing theo tiers (4 điểm)
b) So sánh Applitools vs Percy vs BackstopJS cho trường hợp này (3 điểm)
c) Propose giải pháp cụ thể với cost estimation (3 điểm)

### Đáp án mẫu

**a) Tiered Testing Strategy (4 điểm)**

```
┌─────────────────────────────────────────────────────────────┐
│                   TIERED VISUAL TESTING                      │
├─────────────────────────────────────────────────────────────┤
│  TIER 1: Critical (Every PR) - ~15 pages                    │
│  ├── Homepage, Login, Register                              │
│  ├── Product Detail, Cart, Checkout                         │
│  └── Payment, Order Confirmation                            │
│  → Run on: Chrome only (fastest feedback)                   │
│  → Time: ~3 minutes                                         │
├─────────────────────────────────────────────────────────────┤
│  TIER 2: Important (Nightly) - ~50 pages                    │
│  ├── Category pages, Search results                         │
│  ├── Account settings, Order history                        │
│  └── FAQ, Contact, About                                    │
│  → Run on: Chrome, Firefox, Safari                          │
│  → Time: ~15 minutes                                        │
├─────────────────────────────────────────────────────────────┤
│  TIER 3: Full Coverage (Weekly/Release) - 300 pages         │
│  ├── All pages with all variations                          │
│  └── All 5 browsers + mobile viewports                      │
│  → Time: ~45 minutes                                        │
│  → Schedule: Weekend off-hours                              │
└─────────────────────────────────────────────────────────────┘
```

**b) Tool Comparison (3 điểm)**

| Criteria | Applitools | Percy | BackstopJS |
|----------|------------|-------|------------|
| **Pricing** | $969+/mo | Custom (≈$400) | Free |
| **AI Capability** | Visual AI (best) | Smart snapshot | Pixel-diff only |
| **False Positives** | Very low | Low | High |
| **Setup Complexity** | Medium | Easy | High (self-host) |
| **Cross-browser** | Ultrafast Grid | Cloud rendering | Docker containers |
| **Team Collaboration** | Dashboard | Dashboard | Local reports |

**c) Recommended Solution (3 điểm)**

```
Recommendation: Percy + BackstopJS Hybrid

Budget Breakdown:
- Percy: $400/month (Tier 1 + Tier 2)
- BackstopJS: Free (Tier 3, self-hosted on CI runner)
- Total: $400/month (under budget)

Implementation:
1. Percy for PR testing (Tier 1):
   - 15 critical pages × 1 browser × 30 PRs/month = 450 snapshots

2. Percy for nightly (Tier 2):
   - 50 pages × 3 browsers × 30 nights = 4,500 snapshots

3. BackstopJS for weekly (Tier 3):
   - Self-hosted Docker Playwright containers
   - 300 pages × 5 browsers = 1,500 snapshots (free)

ROI: 78% reduction in visual bug escapes
```

---

## Câu 2: Cross-Browser Testing Debugging (10 điểm)

### Tình huống
Sau khi deploy, nhận report rằng checkout button không hiển thị đúng trên Safari. DevTools cho thấy:

```css
.checkout-btn {
  display: grid;
  gap: 10px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  -webkit-backdrop-filter: blur(10px);
  backdrop-filter: blur(10px);
}
```

### Yêu cầu
a) Xác định potential CSS compatibility issues (3 điểm)
b) Viết visual regression test để catch issue này (4 điểm)
c) Propose fix và prevention strategy (3 điểm)

### Đáp án mẫu

**a) CSS Compatibility Issues (3 điểm)**

```
Issue Analysis:

1. CSS Grid 'gap' Property:
   - Safari < 14.1: Requires 'grid-gap' fallback
   - Detection: gap vs grid-gap

2. Backdrop Filter:
   - Safari: Requires -webkit-backdrop-filter
   - Currently has both (OK)
   - But may not work on older Safari versions

3. Linear Gradient:
   - Safari: Generally supported
   - But specific angle syntax may differ

Tools to Verify:
- caniuse.com/css-grid
- caniuse.com/css-backdrop-filter
- BrowserStack/Sauce Labs for real device testing
```

**b) Visual Regression Test (4 điểm)**

```javascript
// Playwright + Percy for cross-browser visual test
const { test } = require('@playwright/test');
const percySnapshot = require('@percy/playwright');

test.describe('Checkout Button Cross-Browser', () => {
  const viewports = [
    { width: 1920, height: 1080, name: 'Desktop' },
    { width: 768, height: 1024, name: 'Tablet' },
    { width: 375, height: 812, name: 'Mobile' }
  ];

  for (const viewport of viewports) {
    test(`checkout button - ${viewport.name}`, async ({ page }) => {
      await page.setViewportSize({
        width: viewport.width,
        height: viewport.height
      });

      await page.goto('/cart');

      // Wait for button to be fully rendered
      await page.waitForSelector('.checkout-btn', {
        state: 'visible'
      });

      // Disable animations for consistent screenshots
      await page.addStyleTag({
        content: '*, *::before, *::after { transition: none !important; animation: none !important; }'
      });

      // Take snapshot across multiple browsers (Percy handles this)
      await percySnapshot(page, `Checkout Button - ${viewport.name}`, {
        widths: [viewport.width],
        // Percy will render on Chrome, Firefox, Safari, Edge
        browsers: ['chrome', 'firefox', 'safari', 'edge']
      });
    });
  }
});

// Applitools alternative for Visual AI
const { Eyes, Target, Configuration, BrowserType, DeviceName } = require('@applitools/eyes-playwright');

test('checkout button ultrafast grid', async ({ page }) => {
  const eyes = new Eyes();
  const config = new Configuration();

  // Configure Ultrafast Grid browsers
  config.addBrowser(1920, 1080, BrowserType.CHROME);
  config.addBrowser(1920, 1080, BrowserType.FIREFOX);
  config.addBrowser(1920, 1080, BrowserType.SAFARI);
  config.addBrowser(1920, 1080, BrowserType.EDGE_CHROMIUM);
  config.addDeviceEmulation(DeviceName.iPhone_12, ScreenOrientation.PORTRAIT);

  eyes.setConfiguration(config);

  await eyes.open(page, 'E-Commerce', 'Checkout Button');
  await page.goto('/cart');

  await eyes.check('Checkout Button', Target.region('.checkout-btn'));

  await eyes.close();
});
```

**c) Fix and Prevention (3 điểm)**

```css
/* Fixed CSS with fallbacks */
.checkout-btn {
  display: -ms-grid;          /* IE 11 */
  display: grid;
  grid-gap: 10px;             /* Safari < 14.1 fallback */
  gap: 10px;                  /* Modern browsers */

  /* Gradient with fallbacks */
  background: #667eea;        /* Fallback solid color */
  background: -webkit-linear-gradient(315deg, #667eea 0%, #764ba2 100%);
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

  /* Backdrop filter with feature detection */
  @supports (backdrop-filter: blur(10px)) or (-webkit-backdrop-filter: blur(10px)) {
    -webkit-backdrop-filter: blur(10px);
    backdrop-filter: blur(10px);
  }
}

/* Prevention Strategy */
1. Add Stylelint with browser compatibility rules:
   // .stylelintrc
   {
     "extends": "stylelint-config-standard",
     "plugins": ["stylelint-no-unsupported-browser-features"],
     "rules": {
       "plugin/no-unsupported-browser-features": [true, {
         "browsers": ["> 1%", "last 2 versions", "Safari >= 12"],
         "severity": "warning"
       }]
     }
   }

2. Add Autoprefixer to build pipeline
3. Include cross-browser visual test in PR checks
4. Use BrowserStack/Sauce Labs for real device matrix
```

---

## Câu 3: Accessibility Audit & Remediation (10 điểm)

### Tình huống
Audit WCAG 2.1 AA trên trang login cho kết quả:

```
Critical Violations:
1. Form inputs missing associated labels (3 instances)
2. Color contrast ratio 2.5:1 on error messages (req: 4.5:1)
3. No focus indicator on submit button
4. Page has no h1 heading
5. Form cannot be submitted via keyboard only
```

### Yêu cầu
a) Prioritize các violations theo impact và effort (3 điểm)
b) Viết automated accessibility test với axe-core (4 điểm)
c) Provide fixes cho mỗi violation (3 điểm)

### Đáp án mẫu

**a) Priority Matrix (3 điểm)**

```
┌────────────────────────────────────────────────────────────────┐
│                ACCESSIBILITY PRIORITY MATRIX                    │
├───────────────────────┬────────────┬───────────┬───────────────┤
│ Violation             │ Impact     │ Effort    │ Priority      │
├───────────────────────┼────────────┼───────────┼───────────────┤
│ 5. Keyboard submit    │ CRITICAL   │ Low       │ P0 (Fix now)  │
│ 1. Missing labels     │ CRITICAL   │ Low       │ P0 (Fix now)  │
│ 3. No focus indicator │ MAJOR      │ Low       │ P1 (This week)│
│ 2. Contrast 2.5:1     │ MAJOR      │ Low       │ P1 (This week)│
│ 4. No h1 heading      │ MINOR      │ Low       │ P2 (This sprint)│
└───────────────────────┴────────────┴───────────┴───────────────┘

Rationale:
- P0: Blocks screen reader/keyboard users completely
- P1: Makes content difficult to perceive/operate
- P2: Non-compliance but low user impact
```

**b) Automated Accessibility Test (4 điểm)**

```javascript
// Playwright + axe-core integration
const { test, expect } = require('@playwright/test');
const AxeBuilder = require('@axe-core/playwright').default;

test.describe('Login Page Accessibility', () => {

  test('should have no WCAG 2.1 AA violations', async ({ page }) => {
    await page.goto('/login');

    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa'])
      .analyze();

    // Output violations for debugging
    if (accessibilityScanResults.violations.length > 0) {
      console.log('Accessibility violations:');
      accessibilityScanResults.violations.forEach(violation => {
        console.log(`- ${violation.id}: ${violation.description}`);
        console.log(`  Impact: ${violation.impact}`);
        console.log(`  Nodes: ${violation.nodes.length}`);
      });
    }

    expect(accessibilityScanResults.violations).toEqual([]);
  });

  test('should have proper form labels', async ({ page }) => {
    await page.goto('/login');

    // Check specific rule
    const results = await new AxeBuilder({ page })
      .withRules(['label'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('should have sufficient color contrast', async ({ page }) => {
    await page.goto('/login');

    // Trigger error state
    await page.fill('#email', 'invalid');
    await page.click('#submit');
    await page.waitForSelector('.error-message');

    const results = await new AxeBuilder({ page })
      .withRules(['color-contrast'])
      .analyze();

    expect(results.violations).toEqual([]);
  });

  test('should be keyboard navigable', async ({ page }) => {
    await page.goto('/login');

    // Tab through all interactive elements
    await page.keyboard.press('Tab');
    const firstFocused = await page.evaluate(() =>
      document.activeElement?.getAttribute('id')
    );
    expect(firstFocused).toBe('email');

    await page.keyboard.press('Tab');
    const secondFocused = await page.evaluate(() =>
      document.activeElement?.getAttribute('id')
    );
    expect(secondFocused).toBe('password');

    await page.keyboard.press('Tab');
    const thirdFocused = await page.evaluate(() =>
      document.activeElement?.getAttribute('id')
    );
    expect(thirdFocused).toBe('submit');

    // Should be able to submit via Enter
    await page.fill('#email', 'test@example.com');
    await page.fill('#password', 'password123');
    await page.keyboard.press('Enter');

    // Verify form submitted (navigation occurred or success message)
    await expect(page).toHaveURL(/dashboard|home/);
  });

  test('should have visible focus indicators', async ({ page }) => {
    await page.goto('/login');

    // Focus on submit button
    await page.focus('#submit');

    // Check focus styles
    const focusStyles = await page.$eval('#submit', el => {
      const styles = window.getComputedStyle(el);
      return {
        outline: styles.outline,
        boxShadow: styles.boxShadow,
        border: styles.border
      };
    });

    // At least one focus indicator should be visible
    const hasFocusIndicator =
      focusStyles.outline !== 'none' ||
      focusStyles.boxShadow !== 'none' ||
      focusStyles.border !== 'none';

    expect(hasFocusIndicator).toBe(true);
  });
});
```

**c) Fixes for Each Violation (3 điểm)**

```html
<!-- BEFORE (Broken) -->
<form>
  <input type="email" id="email" placeholder="Email">
  <input type="password" id="password" placeholder="Password">
  <span class="error-message" style="color: #ff6b6b;">Invalid credentials</span>
  <div onclick="submitForm()">Login</div>
</form>

<!-- AFTER (Fixed) -->
<form onsubmit="handleSubmit(event)">
  <!-- Fix 4: Add h1 heading -->
  <h1>Login to Your Account</h1>

  <!-- Fix 1: Add proper labels -->
  <label for="email">Email Address</label>
  <input
    type="email"
    id="email"
    name="email"
    aria-required="true"
    aria-describedby="email-error"
  >

  <label for="password">Password</label>
  <input
    type="password"
    id="password"
    name="password"
    aria-required="true"
  >

  <!-- Fix 2: Improve contrast (4.5:1 minimum) -->
  <span
    class="error-message"
    id="email-error"
    role="alert"
    style="color: #c53030; background: #fff5f5;"
  >
    Invalid credentials
  </span>

  <!-- Fix 3 & 5: Use button, add focus styles -->
  <button
    type="submit"
    id="submit"
    class="submit-btn"
  >
    Login
  </button>
</form>

<style>
/* Fix 3: Focus indicator */
.submit-btn:focus {
  outline: 3px solid #4299e1;
  outline-offset: 2px;
}

/* Also ensure focus-visible for keyboard only */
.submit-btn:focus:not(:focus-visible) {
  outline: none;
}

.submit-btn:focus-visible {
  outline: 3px solid #4299e1;
  outline-offset: 2px;
}

/* Fix 2: Error message contrast */
.error-message {
  color: #c53030;        /* Contrast ratio: 7.1:1 with white */
  background: #fff5f5;
  padding: 8px 12px;
  border-radius: 4px;
}
</style>

<script>
// Fix 5: Keyboard submission support
function handleSubmit(event) {
  event.preventDefault();
  // Form submission logic
  submitForm();
}
</script>
```

---

## Câu 4: Responsive Design Testing (10 điểm)

### Tình huống
Product Manager yêu cầu verify responsive behavior của product grid:
- Desktop (1920px): 4 columns
- Tablet (768px): 2 columns
- Mobile (375px): 1 column

Tester phát hiện grid bị vỡ ở 900px (3 columns không đều).

### Yêu cầu
a) Thiết kế test cases cho responsive behavior (3 điểm)
b) Viết automated test với Playwright để verify grid columns (4 điểm)
c) Debug và propose fix cho 900px breakpoint issue (3 điểm)

### Đáp án mẫu

**a) Test Case Design (3 điểm)**

```
TC-RESP-001: Desktop Layout (1920px)
├── Precondition: Browser width = 1920px
├── Steps: Navigate to product listing
├── Expected: 4 product columns, equal width
└── Verify: Column count, spacing, alignment

TC-RESP-002: Tablet Layout (768px)
├── Precondition: Browser width = 768px
├── Steps: Navigate to product listing
├── Expected: 2 product columns
└── Verify: Columns equal width, no overflow

TC-RESP-003: Mobile Layout (375px)
├── Precondition: Browser width = 375px
├── Steps: Navigate to product listing
├── Expected: 1 product column, full width
└── Verify: No horizontal scroll, images scale

TC-RESP-004: Breakpoint Boundary Testing
├── Test at: 767px, 768px, 769px (tablet boundary)
├── Test at: 374px, 375px, 376px (mobile boundary)
├── Test at: 899px, 900px, 901px (gap in design)
└── Expected: Graceful transition at each breakpoint

TC-RESP-005: Intermediate Widths
├── Test at: 1024px, 1280px, 1440px
├── Expected: Layout appropriate for width
└── Verify: No awkward gaps or overlaps

TC-RESP-006: Orientation Change
├── Start: Portrait 768x1024
├── Action: Rotate to landscape 1024x768
├── Expected: Layout adapts without refresh
└── Verify: No content cut off
```

**b) Automated Responsive Test (4 điểm)**

```javascript
const { test, expect } = require('@playwright/test');

test.describe('Product Grid Responsive Layout', () => {

  const breakpoints = [
    { name: 'Desktop', width: 1920, height: 1080, expectedColumns: 4 },
    { name: 'Laptop', width: 1280, height: 800, expectedColumns: 4 },
    { name: 'Tablet Landscape', width: 1024, height: 768, expectedColumns: 3 },
    { name: 'Tablet Portrait', width: 768, height: 1024, expectedColumns: 2 },
    { name: 'Mobile', width: 375, height: 812, expectedColumns: 1 },
    { name: 'Problem Width', width: 900, height: 600, expectedColumns: 3 },
  ];

  for (const bp of breakpoints) {
    test(`should display ${bp.expectedColumns} columns at ${bp.name} (${bp.width}px)`, async ({ page }) => {
      await page.setViewportSize({ width: bp.width, height: bp.height });
      await page.goto('/products');

      // Wait for products to load
      await page.waitForSelector('.product-card');

      // Get computed grid columns
      const gridColumns = await page.$eval('.product-grid', el => {
        const style = window.getComputedStyle(el);
        const gridTemplateColumns = style.gridTemplateColumns;
        // Count number of column values (e.g., "300px 300px 300px" = 3)
        return gridTemplateColumns.split(' ').length;
      });

      expect(gridColumns).toBe(bp.expectedColumns);
    });
  }

  test('should have equal column widths at all breakpoints', async ({ page }) => {
    const widths = [1920, 768, 375];

    for (const width of widths) {
      await page.setViewportSize({ width, height: 800 });
      await page.goto('/products');
      await page.waitForSelector('.product-card');

      const cardWidths = await page.$$eval('.product-card', cards =>
        cards.slice(0, 4).map(card => card.getBoundingClientRect().width)
      );

      // All cards should be same width (within 1px tolerance)
      const firstWidth = cardWidths[0];
      cardWidths.forEach((w, i) => {
        expect(Math.abs(w - firstWidth)).toBeLessThan(2);
      });
    }
  });

  test('should not have horizontal scroll on mobile', async ({ page }) => {
    await page.setViewportSize({ width: 375, height: 812 });
    await page.goto('/products');

    const hasHorizontalScroll = await page.evaluate(() => {
      return document.documentElement.scrollWidth > document.documentElement.clientWidth;
    });

    expect(hasHorizontalScroll).toBe(false);
  });

  test('visual regression at all breakpoints', async ({ page }) => {
    const breakpoints = [1920, 1280, 1024, 900, 768, 375];

    for (const width of breakpoints) {
      await page.setViewportSize({ width, height: 800 });
      await page.goto('/products');
      await page.waitForSelector('.product-card');

      // Wait for images
      await page.waitForFunction(() =>
        Array.from(document.images).every(img => img.complete)
      );

      await expect(page).toHaveScreenshot(`products-${width}px.png`, {
        fullPage: true,
        animations: 'disabled'
      });
    }
  });
});
```

**c) Debug and Fix 900px Issue (3 điểm)**

```css
/* PROBLEM: Current CSS */
.product-grid {
  display: grid;
  gap: 20px;
}

@media (min-width: 1200px) {
  .product-grid { grid-template-columns: repeat(4, 1fr); }
}

@media (min-width: 768px) and (max-width: 1199px) {
  .product-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (max-width: 767px) {
  .product-grid { grid-template-columns: 1fr; }
}

/* ISSUE AT 900px:
   - Falls into 768-1199px range → 2 columns
   - But screen is wide enough for 3 columns
   - Creates awkward spacing
*/

/* SOLUTION 1: Add intermediate breakpoint */
.product-grid {
  display: grid;
  gap: 20px;
  grid-template-columns: 1fr;  /* Mobile default */
}

@media (min-width: 576px) {
  .product-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 900px) {
  .product-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (min-width: 1200px) {
  .product-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

/* SOLUTION 2: Auto-fit (more flexible) */
.product-grid {
  display: grid;
  gap: 20px;
  grid-template-columns: repeat(
    auto-fit,
    minmax(280px, 1fr)
  );
}

/* This auto-calculates columns based on:
   - Minimum card width: 280px
   - Maximum card width: 1fr (equal share)

   At 900px: floor(900 / 300) = 3 columns
   At 768px: floor(768 / 300) = 2 columns
   At 375px: floor(375 / 300) = 1 column
*/
```

---

## Câu 5: Handling Dynamic Content in Visual Testing (10 điểm)

### Tình huống
Dashboard có nhiều dynamic content:
- Real-time stock prices (thay đổi mỗi giây)
- User avatar (different per user)
- Timestamps ("5 minutes ago")
- Chart data (live updates)
- Notification badges (random count)

Visual tests liên tục fail do dynamic content.

### Yêu cầu
a) Phân loại dynamic content và strategies để handle (3 điểm)
b) Implement visual test với Applitools xử lý dynamic content (4 điểm)
c) Design mock strategy để ổn định tests (3 điểm)

### Đáp án mẫu

**a) Dynamic Content Classification (3 điểm)**

```
┌──────────────────────────────────────────────────────────────────┐
│                DYNAMIC CONTENT STRATEGIES                         │
├─────────────────┬──────────────────┬──────────────────────────────┤
│ Content Type    │ Strategy         │ Implementation               │
├─────────────────┼──────────────────┼──────────────────────────────┤
│ Stock Prices    │ Ignore Region    │ .ignoreRegions('.stock-price')│
│ User Avatar     │ Mock/Replace     │ Intercept API, return fixed  │
│ Timestamps      │ Ignore or Mock   │ Mock Date.now() or ignore    │
│ Chart Data      │ Layout Match     │ MatchLevel.Layout            │
│ Notification    │ Hide/Remove      │ CSS: display:none or remove  │
│ Carousel        │ Freeze State     │ Stop animation, set index    │
│ Loading States  │ Wait for Stable  │ waitForLoadState('networkidle')│
│ Ads/Banners     │ Ignore Region    │ .ignoreRegions('.ad-container')│
└─────────────────┴──────────────────┴──────────────────────────────┘
```

**b) Applitools Implementation (4 điểm)**

```javascript
const { test } = require('@playwright/test');
const { Eyes, Target, Configuration, MatchLevel } = require('@applitools/eyes-playwright');

test.describe('Dashboard Visual Tests', () => {
  let eyes;

  test.beforeEach(async ({ page }) => {
    eyes = new Eyes();
    const config = new Configuration();
    config.setApiKey(process.env.APPLITOOLS_API_KEY);
    config.setBatch({ name: 'Dashboard Tests' });
    eyes.setConfiguration(config);

    // Mock dynamic APIs before navigation
    await page.route('**/api/user/profile', route => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          name: 'Test User',
          avatar: 'https://example.com/default-avatar.png'
        })
      });
    });

    await page.route('**/api/stocks', route => {
      route.fulfill({
        status: 200,
        contentType: 'application/json',
        body: JSON.stringify({
          AAPL: 150.00,
          GOOGL: 2800.00,
          MSFT: 300.00
        })
      });
    });
  });

  test('dashboard with dynamic content handling', async ({ page }) => {
    await eyes.open(page, 'Dashboard', 'Main Dashboard');

    await page.goto('/dashboard');

    // Wait for all content to load
    await page.waitForLoadState('networkidle');

    // Freeze timestamp display
    await page.evaluate(() => {
      const now = new Date('2024-01-15T10:00:00Z');
      Date.now = () => now.getTime();
    });

    // Stop animations and transitions
    await page.addStyleTag({
      content: `
        *, *::before, *::after {
          animation: none !important;
          transition: none !important;
        }
        .carousel { transform: translateX(0) !important; }
      `
    });

    // Hide notification badges (unpredictable count)
    await page.evaluate(() => {
      document.querySelectorAll('.notification-badge').forEach(el => {
        el.style.display = 'none';
      });
    });

    // Full page check with smart regions
    await eyes.check('Dashboard', Target.window()
      .fully()
      .ignoreRegions(
        '.stock-ticker',      // Real-time prices
        '.timestamp',         // Relative times
        '.ad-banner',         // Promotional content
        '.loading-spinner'    // Loading states
      )
      .layoutRegions(
        '.chart-container',   // Chart structure, not data
        '.feed-container'     // Feed layout, not content
      )
      .floatingRegion('.tooltip', 10, 10, 10, 10)  // Allow tooltip movement
    );

    // Specific component checks
    await eyes.check('Navigation', Target.region('.nav-header'));

    await eyes.check('Sidebar', Target.region('.sidebar')
      .ignoreRegions('.user-avatar')
    );

    // Chart with layout matching (verify axes, legends, not data points)
    await eyes.check('Performance Chart', Target.region('.chart-container')
      .matchLevel(MatchLevel.Layout)
    );

    await eyes.close();
  });

  test.afterEach(async () => {
    await eyes.abort();
  });
});
```

**c) Mock Strategy Design (3 điểm)**

```javascript
// test/fixtures/visual-test-fixtures.js

/**
 * Visual Test Mock Strategy
 *
 * Goal: Ensure deterministic visual tests by controlling all dynamic content
 */

// 1. API Response Mocks
const mockResponses = {
  user: {
    name: 'Test User',
    avatar: '/assets/test-avatar.png',
    notifications: 5
  },
  stocks: {
    AAPL: { price: 150.00, change: +2.5 },
    GOOGL: { price: 2800.00, change: -1.2 }
  },
  chart: {
    labels: ['Jan', 'Feb', 'Mar'],
    values: [100, 120, 115]
  }
};

// 2. Setup Mock Routes
async function setupVisualTestMocks(page) {
  // Mock all API endpoints
  await page.route('**/api/**', async route => {
    const url = route.request().url();

    if (url.includes('/user')) {
      return route.fulfill({ json: mockResponses.user });
    }
    if (url.includes('/stocks')) {
      return route.fulfill({ json: mockResponses.stocks });
    }
    if (url.includes('/chart')) {
      return route.fulfill({ json: mockResponses.chart });
    }

    // Let other requests through
    return route.continue();
  });

  // Mock browser APIs
  await page.addInitScript(() => {
    // Fixed date/time
    const fixedDate = new Date('2024-01-15T10:00:00Z');
    window.Date = class extends Date {
      constructor(...args) {
        if (args.length === 0) return fixedDate;
        return super(...args);
      }
      static now() { return fixedDate.getTime(); }
    };

    // Disable random for consistent renders
    Math.random = () => 0.5;
  });
}

// 3. Content Stabilization Utilities
async function stabilizeForScreenshot(page) {
  await page.evaluate(() => {
    // Disable all animations
    document.querySelectorAll('*').forEach(el => {
      el.style.animation = 'none';
      el.style.transition = 'none';
    });

    // Replace dynamic images with placeholders
    document.querySelectorAll('img[data-dynamic]').forEach(img => {
      img.src = '/assets/placeholder.png';
    });

    // Set carousel to first slide
    document.querySelectorAll('.carousel').forEach(carousel => {
      carousel.style.transform = 'translateX(0)';
    });

    // Hide loading indicators
    document.querySelectorAll('.skeleton, .loading').forEach(el => {
      el.style.display = 'none';
    });
  });

  // Wait for images to load
  await page.waitForFunction(() =>
    Array.from(document.images).every(img => img.complete)
  );
}

// 4. Usage in Tests
test('stable dashboard visual test', async ({ page }) => {
  await setupVisualTestMocks(page);
  await page.goto('/dashboard');
  await page.waitForLoadState('networkidle');
  await stabilizeForScreenshot(page);

  // Now safe to take screenshots
  await expect(page).toHaveScreenshot('dashboard.png');
});
```

---

## Câu 6: Usability Heuristic Evaluation (10 điểm)

### Tình huống
Được giao evaluate trang checkout theo Nielsen's 10 Heuristics. Quan sát thấy:

1. Sau click "Place Order", không có loading indicator
2. Error message: "Error 500"
3. Không có nút "Back" để sửa thông tin
4. Form fields không có placeholders hoặc hints
5. Cùng field "Phone" có 3 formats khác nhau ở 3 trang

### Yêu cầu
a) Map mỗi issue vào heuristic bị vi phạm và severity rating (4 điểm)
b) Viết test cases để detect các usability issues này (3 điểm)
c) Recommend fixes theo priority (3 điểm)

### Đáp án mẫu

**a) Heuristic Mapping & Severity (4 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        HEURISTIC EVALUATION REPORT                           │
├────┬──────────────────────────┬─────────────────────────┬─────────┬─────────┤
│ #  │ Issue                    │ Heuristic Violated      │ Severity│ Impact  │
├────┼──────────────────────────┼─────────────────────────┼─────────┼─────────┤
│ 1  │ No loading indicator     │ H1: Visibility of       │ 3-Major │ User    │
│    │ after "Place Order"      │ System Status           │         │ thinks  │
│    │                          │                         │         │ it failed│
├────┼──────────────────────────┼─────────────────────────┼─────────┼─────────┤
│ 2  │ "Error 500" message      │ H9: Help Users          │ 4-Critical│ User  │
│    │                          │ Recognize, Diagnose,    │         │ cannot  │
│    │                          │ and Recover from Errors │         │ recover │
├────┼──────────────────────────┼─────────────────────────┼─────────┼─────────┤
│ 3  │ No "Back" button         │ H3: User Control        │ 3-Major │ Must    │
│    │                          │ and Freedom             │         │ restart │
│    │                          │                         │         │ checkout│
├────┼──────────────────────────┼─────────────────────────┼─────────┼─────────┤
│ 4  │ No placeholders/hints    │ H6: Recognition         │ 2-Minor │ Users   │
│    │                          │ Rather Than Recall      │         │ confused│
├────┼──────────────────────────┼─────────────────────────┼─────────┼─────────┤
│ 5  │ Inconsistent phone       │ H4: Consistency         │ 3-Major │ Data    │
│    │ format across pages      │ and Standards           │         │ errors  │
└────┴──────────────────────────┴─────────────────────────┴─────────┴─────────┘

Severity Scale:
0 = Not a usability problem
1 = Cosmetic problem only
2 = Minor usability problem
3 = Major usability problem (important to fix)
4 = Usability catastrophe (imperative to fix)
```

**b) Usability Test Cases (3 điểm)**

```javascript
const { test, expect } = require('@playwright/test');

test.describe('Checkout Usability Tests', () => {

  // H1: Visibility of System Status
  test('should show loading indicator when placing order', async ({ page }) => {
    await page.goto('/checkout');
    await fillCheckoutForm(page);

    // Start order submission
    const submitButton = page.locator('[data-test="place-order"]');
    await submitButton.click();

    // Loading indicator should appear within 100ms
    const loadingIndicator = page.locator('.loading-indicator, .spinner, [role="progressbar"]');
    await expect(loadingIndicator).toBeVisible({ timeout: 500 });

    // Button should be disabled to prevent double-submit
    await expect(submitButton).toBeDisabled();
  });

  // H9: Error Recovery
  test('should show user-friendly error messages', async ({ page }) => {
    await page.goto('/checkout');

    // Mock server error
    await page.route('**/api/orders', route =>
      route.fulfill({ status: 500 })
    );

    await fillCheckoutForm(page);
    await page.click('[data-test="place-order"]');

    // Error message should be user-friendly
    const errorMessage = page.locator('.error-message');
    await expect(errorMessage).toBeVisible();

    // Should NOT contain technical jargon
    const errorText = await errorMessage.textContent();
    expect(errorText).not.toContain('500');
    expect(errorText).not.toContain('Error');
    expect(errorText).not.toContain('exception');

    // Should provide actionable guidance
    expect(errorText).toMatch(/try again|contact support|please check/i);
  });

  // H3: User Control and Freedom
  test('should allow user to go back and edit information', async ({ page }) => {
    await page.goto('/checkout');
    await fillCheckoutForm(page);

    // Navigate to review step
    await page.click('[data-test="continue-to-review"]');
    await expect(page.locator('.review-step')).toBeVisible();

    // Back button should exist
    const backButton = page.locator('[data-test="back-button"], .back-link, button:has-text("Back")');
    await expect(backButton).toBeVisible();

    // Clicking back should preserve form data
    await backButton.click();
    await expect(page.locator('#email')).toHaveValue('test@example.com');
  });

  // H6: Recognition Rather Than Recall
  test('should have placeholders or hints for form fields', async ({ page }) => {
    await page.goto('/checkout');

    const requiredFields = ['#email', '#phone', '#address', '#card-number'];

    for (const fieldSelector of requiredFields) {
      const field = page.locator(fieldSelector);

      // Check for placeholder OR associated label with hint
      const placeholder = await field.getAttribute('placeholder');
      const ariaDescribedBy = await field.getAttribute('aria-describedby');

      const hasHint = placeholder || ariaDescribedBy;
      expect(hasHint, `Field ${fieldSelector} should have placeholder or hint`).toBeTruthy();
    }
  });

  // H4: Consistency and Standards
  test('should have consistent phone format across pages', async ({ page }) => {
    const phoneFormats = [];

    // Check phone field on checkout page
    await page.goto('/checkout');
    phoneFormats.push(await getPhoneFieldFormat(page, '#phone'));

    // Check phone field on account page
    await page.goto('/account');
    phoneFormats.push(await getPhoneFieldFormat(page, '#phone'));

    // Check phone field on contact page
    await page.goto('/contact');
    phoneFormats.push(await getPhoneFieldFormat(page, '#phone'));

    // All formats should be identical
    expect(new Set(phoneFormats).size).toBe(1);
  });
});

async function fillCheckoutForm(page) {
  await page.fill('#email', 'test@example.com');
  await page.fill('#name', 'Test User');
  await page.fill('#address', '123 Test St');
}

async function getPhoneFieldFormat(page, selector) {
  const field = page.locator(selector);
  const placeholder = await field.getAttribute('placeholder') || '';
  const pattern = await field.getAttribute('pattern') || '';
  return { placeholder, pattern };
}
```

**c) Prioritized Recommendations (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────┐
│                    REMEDIATION ROADMAP                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PHASE 1: Critical (This Week) - Severity 4                     │
│  ────────────────────────────────────────                       │
│  Issue #2: User-friendly error messages                         │
│                                                                  │
│  Implementation:                                                 │
│  ```javascript                                                   │
│  const errorMessages = {                                         │
│    500: 'Something went wrong. Please try again or contact      │
│          support at help@example.com',                           │
│    400: 'Please check your information and try again',          │
│    401: 'Please log in to continue',                            │
│    default: 'An unexpected error occurred. Please refresh.'     │
│  };                                                              │
│  ```                                                             │
│                                                                  │
│  PHASE 2: Major (This Sprint) - Severity 3                      │
│  ────────────────────────────────────────                       │
│  Issue #1: Loading indicator                                     │
│  Issue #3: Back button                                           │
│  Issue #5: Phone format consistency                              │
│                                                                  │
│  Implementation:                                                 │
│  - Add loading spinner component                                 │
│  - Add "Edit" links to review step                               │
│  - Create shared phone input component with mask                 │
│                                                                  │
│  PHASE 3: Minor (Next Sprint) - Severity 2                      │
│  ────────────────────────────────────────                       │
│  Issue #4: Placeholders and hints                                │
│                                                                  │
│  Implementation:                                                 │
│  - Add descriptive placeholders                                  │
│  - Add helper text for complex fields                            │
│  - Consider inline validation hints                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

Effort vs Impact Matrix:
                    HIGH IMPACT
                         │
   [#2 Error Msg]        │    [#1 Loading]
   Quick Win             │    Strategic
                         │
   ─────────────────────┼─────────────────────
                         │
   [#4 Placeholders]     │    [#5 Phone Format]
   Fill-In               │    Major Project
                         │
                    LOW IMPACT
         LOW EFFORT              HIGH EFFORT
```

---

## Câu 7: Visual Testing Pipeline Design (10 điểm)

### Tình huống
Team cần integrate visual testing vào CI/CD pipeline:
- GitHub Actions for CI
- 50 critical pages cần test
- 3 environments: dev, staging, prod
- Cần review process cho visual changes

### Yêu cầu
a) Design CI/CD workflow với visual testing gates (4 điểm)
b) Write GitHub Actions workflow file (3 điểm)
c) Design review process cho visual diffs (3 điểm)

### Đáp án mẫu

**a) CI/CD Workflow Design (4 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    VISUAL TESTING CI/CD PIPELINE                         │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │   Code   │───>│   Unit   │───>│  Build   │───>│  Deploy  │          │
│  │   Push   │    │  Tests   │    │   App    │    │   Dev    │          │
│  └──────────┘    └──────────┘    └──────────┘    └────┬─────┘          │
│                                                        │                 │
│                                                        ▼                 │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    VISUAL TESTING STAGE                          │   │
│  ├─────────────────────────────────────────────────────────────────┤   │
│  │                                                                   │   │
│  │   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │   │
│  │   │ Screenshot   │───>│  Compare to  │───>│   Report     │      │   │
│  │   │ Capture      │    │   Baseline   │    │   Diffs      │      │   │
│  │   └──────────────┘    └──────────────┘    └───────┬──────┘      │   │
│  │                                                    │             │   │
│  │                       ┌────────────────────────────┼────────┐   │   │
│  │                       │                            ▼        │   │   │
│  │                       │    ┌──────────────────────────┐    │   │   │
│  │                       │    │      DIFF DETECTED?      │    │   │   │
│  │                       │    └────────────┬─────────────┘    │   │   │
│  │                       │                 │                   │   │   │
│  │                       │    ┌────────────┴────────────┐     │   │   │
│  │                       │    │                         │     │   │   │
│  │                       │    ▼                         ▼     │   │   │
│  │                 ┌──────────────┐              ┌──────────┐│   │   │
│  │                 │ No Changes   │              │ Changes  ││   │   │
│  │                 │ Auto-Approve │              │ Detected ││   │   │
│  │                 └──────┬───────┘              └────┬─────┘│   │   │
│  │                        │                           │      │   │   │
│  │                        │                           ▼      │   │   │
│  │                        │                    ┌────────────┐│   │   │
│  │                        │                    │ Request    ││   │   │
│  │                        │                    │ Review     ││   │   │
│  │                        │                    └─────┬──────┘│   │   │
│  │                        │                          │       │   │   │
│  │                        │         ┌────────────────┼───┐   │   │   │
│  │                        │         │                ▼   │   │   │   │
│  │                        │         │  ┌──────────────┐  │   │   │   │
│  │                        │         │  │ APPROVED?    │  │   │   │   │
│  │                        │         │  └───────┬──────┘  │   │   │   │
│  │                        │         │          │         │   │   │   │
│  │                        ▼         │   ┌──────┴──────┐  │   │   │   │
│  │                 ┌──────────┐     │   │             │  │   │   │   │
│  │                 │ PASS     │◄────│───┤ YES  │  NO  │  │   │   │   │
│  │                 └────┬─────┘     │   │             │  │   │   │   │
│  │                      │           │   └─────────────┘  │   │   │   │
│  │                      │           │         │          │   │   │   │
│  │                      │           │         ▼          │   │   │   │
│  │                      │           │  ┌──────────────┐  │   │   │   │
│  │                      │           │  │ BLOCK PR     │  │   │   │   │
│  │                      │           │  └──────────────┘  │   │   │   │
│  │                      │           │                    │   │   │   │
│  └──────────────────────│───────────┴────────────────────┘   │   │   │
│                         │                                     │   │   │
│                         ▼                                     │   │   │
│                  ┌──────────────┐                             │   │   │
│                  │ Deploy to    │                             │   │   │
│                  │ Staging      │                             │   │   │
│                  └──────┬───────┘                             │   │   │
│                         │                                     │   │   │
│                         ▼                                     │   │   │
│                  ┌──────────────┐                             │   │   │
│                  │ E2E Tests    │                             │   │   │
│                  │ + Visual     │                             │   │   │
│                  └──────┬───────┘                             │   │   │
│                         │                                     │   │   │
│                         ▼                                     │   │   │
│                  ┌──────────────┐                             │   │   │
│                  │ Deploy to    │                             │   │   │
│                  │ Production   │                             │   │   │
│                  └──────────────┘                             │   │   │
│                                                               │   │   │
└───────────────────────────────────────────────────────────────┘───┘───┘
```

**b) GitHub Actions Workflow (3 điểm)**

```yaml
# .github/workflows/visual-testing.yml
name: Visual Testing Pipeline

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

env:
  PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}
  APPLITOOLS_API_KEY: ${{ secrets.APPLITOOLS_API_KEY }}

jobs:
  visual-tests:
    runs-on: ubuntu-latest

    services:
      # App runs locally for testing
      app:
        image: node:18
        ports:
          - 3000:3000

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build application
        run: npm run build

      - name: Start application
        run: |
          npm run start &
          npx wait-on http://localhost:3000 --timeout 60000

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Run visual tests with Percy
        run: npx percy exec -- npx playwright test visual-tests/
        env:
          PERCY_PARALLEL_NONCE: ${{ github.run_id }}
          PERCY_PARALLEL_TOTAL: 1

      - name: Run cross-browser visual tests with Applitools
        run: npx playwright test ultrafast-grid-tests/
        env:
          APPLITOOLS_BATCH_NAME: "PR #${{ github.event.pull_request.number }}"
          APPLITOOLS_BRANCH_NAME: ${{ github.head_ref }}
          APPLITOOLS_PARENT_BRANCH_NAME: ${{ github.base_ref }}

      - name: Check visual test results
        id: visual-check
        run: |
          # Percy finalize and check status
          percy_status=$(npx percy exec:status)
          echo "percy_status=$percy_status" >> $GITHUB_OUTPUT

          if [ "$percy_status" == "pending_approval" ]; then
            echo "::warning::Visual changes detected - review required"
            echo "needs_review=true" >> $GITHUB_OUTPUT
          fi

      - name: Comment on PR with visual diff links
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const { percy_status, needs_review } = process.env;

            let comment = '## Visual Testing Results\n\n';

            if (needs_review === 'true') {
              comment += ':warning: **Visual changes detected!**\n\n';
              comment += 'Please review the visual diffs:\n';
              comment += '- [Percy Dashboard](https://percy.io/...)\n';
              comment += '- [Applitools Dashboard](https://eyes.applitools.com/...)\n\n';
              comment += 'Approve the changes in the dashboard to unblock this PR.';
            } else {
              comment += ':white_check_mark: **No visual changes detected.**';
            }

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });

      - name: Upload test artifacts
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: visual-test-results
          path: |
            test-results/
            playwright-report/

  deploy-staging:
    needs: visual-tests
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com

    steps:
      - name: Deploy to Staging
        run: |
          # Deployment logic here
          echo "Deploying to staging..."

  visual-tests-staging:
    needs: deploy-staging
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run visual tests against staging
        run: |
          BASE_URL=https://staging.example.com npx percy exec -- npx playwright test visual-tests/
        env:
          PERCY_TOKEN: ${{ secrets.PERCY_TOKEN }}
          PERCY_BRANCH: staging
```

**c) Review Process Design (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    VISUAL DIFF REVIEW PROCESS                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. AUTOMATED TRIAGE                                                     │
│  ───────────────────                                                     │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ When diff detected:                                                │  │
│  │                                                                    │  │
│  │ a) Auto-approve if:                                               │  │
│  │    - Only timestamp/date changes                                  │  │
│  │    - Known dynamic content regions                                │  │
│  │    - < 0.1% pixel difference                                      │  │
│  │                                                                    │  │
│  │ b) Request review if:                                             │  │
│  │    - Structural changes (layout shift)                            │  │
│  │    - Color changes                                                │  │
│  │    - New elements added/removed                                   │  │
│  │    - > 0.1% pixel difference                                      │  │
│  │                                                                    │  │
│  │ c) Auto-reject if:                                                │  │
│  │    - Entire page blank/error                                      │  │
│  │    - Critical component missing                                   │  │
│  │    - Layout completely broken                                     │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  2. REVIEWER ASSIGNMENT                                                  │
│  ─────────────────────                                                   │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ Based on changed components:                                       │  │
│  │                                                                    │  │
│  │ - Header/Footer changes    → @design-team                        │  │
│  │ - Checkout flow changes    → @checkout-owners + @design-team     │  │
│  │ - Product pages changes    → @catalog-team                       │  │
│  │ - All other changes        → @frontend-team                      │  │
│  │                                                                    │  │
│  │ CODEOWNERS integration:                                           │  │
│  │ # .github/CODEOWNERS                                              │  │
│  │ /visual-baselines/checkout/* @checkout-owners @design-team       │  │
│  │ /visual-baselines/product/*  @catalog-team                       │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  3. REVIEW WORKFLOW                                                      │
│  ─────────────────                                                       │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                                                                    │  │
│  │   [PR Created] ────> [Visual Test Run] ────> [Diff Detected]     │  │
│  │                                                      │            │  │
│  │                                                      ▼            │  │
│  │                              ┌─────────────────────────────────┐  │  │
│  │                              │ Slack/Teams Notification        │  │  │
│  │                              │ "Visual changes in PR #123"     │  │  │
│  │                              │ [Review Now] button             │  │  │
│  │                              └───────────────┬─────────────────┘  │  │
│  │                                              │                    │  │
│  │                                              ▼                    │  │
│  │                              ┌─────────────────────────────────┐  │  │
│  │                              │ Reviewer opens dashboard:       │  │  │
│  │                              │ - Side-by-side comparison       │  │  │
│  │                              │ - Diff highlighting             │  │  │
│  │                              │ - Component-level breakdown     │  │  │
│  │                              └───────────────┬─────────────────┘  │  │
│  │                                              │                    │  │
│  │                         ┌────────────────────┼────────────────┐   │  │
│  │                         │                    │                │   │  │
│  │                         ▼                    ▼                ▼   │  │
│  │                 ┌────────────┐       ┌────────────┐   ┌──────────┐│  │
│  │                 │  APPROVE   │       │  REQUEST   │   │  REJECT  ││  │
│  │                 │ (Expected) │       │  CHANGES   │   │(Bug Found)│  │
│  │                 └─────┬──────┘       └─────┬──────┘   └─────┬────┘│  │
│  │                       │                    │                │     │  │
│  │                       ▼                    ▼                ▼     │  │
│  │              ┌────────────┐       ┌────────────┐    ┌────────────┐│  │
│  │              │ Update     │       │ Comment on │    │ Block PR   ││  │
│  │              │ Baseline   │       │ PR with    │    │ Create Bug ││  │
│  │              │ Unblock PR │       │ Details    │    │ Ticket     ││  │
│  │              └────────────┘       └────────────┘    └────────────┘│  │
│  │                                                                    │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  4. SLA & ESCALATION                                                     │
│  ──────────────────                                                      │
│  - Review SLA: 4 hours during business hours                            │
│  - If no review after 4h: Escalate to tech lead                         │
│  - If no review after 8h: Auto-escalate to engineering manager          │
│  - Critical pages (checkout, payment): 2 hour SLA                        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 8: Mobile Responsive Testing (10 điểm)

### Tình huống
Nhận report từ users mobile về các issues:
- Images không hiển thị đúng trên iPhone
- Touch targets quá nhỏ trên Android
- Horizontal scroll xuất hiện unexpectedly
- Text overlap khi rotate screen

### Yêu cầu
a) Design test matrix cho mobile responsive testing (3 điểm)
b) Write Playwright tests cho mobile-specific issues (4 điểm)
c) Propose fixes và prevention measures (3 điểm)

### Đáp án mẫu

**a) Mobile Test Matrix (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MOBILE RESPONSIVE TEST MATRIX                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DEVICE COVERAGE                                                             │
│  ───────────────                                                             │
│  ┌─────────────────┬─────────────┬────────────────┬───────────────────────┐ │
│  │ Device          │ Viewport    │ Pixel Ratio    │ Priority              │ │
│  ├─────────────────┼─────────────┼────────────────┼───────────────────────┤ │
│  │ iPhone 14 Pro   │ 393×852     │ 3x             │ HIGH (market share)   │ │
│  │ iPhone SE       │ 375×667     │ 2x             │ HIGH (smallest iPhone)│ │
│  │ iPhone 14 Pro Max│ 430×932    │ 3x             │ MEDIUM                │ │
│  │ Samsung S21     │ 360×800     │ 3x             │ HIGH (Android leader) │ │
│  │ Pixel 7         │ 412×915     │ 2.625x         │ MEDIUM                │ │
│  │ Samsung Tab S8  │ 800×1280    │ 2x             │ MEDIUM (tablet)       │ │
│  │ iPad Pro 12.9   │ 1024×1366   │ 2x             │ MEDIUM (tablet)       │ │
│  └─────────────────┴─────────────┴────────────────┴───────────────────────┘ │
│                                                                              │
│  ORIENTATION COMBINATIONS                                                    │
│  ────────────────────────                                                    │
│  - Portrait → Landscape → Portrait (rotation test)                          │
│  - Start in Landscape (some users preference)                               │
│  - Split screen mode (Android)                                              │
│                                                                              │
│  INTERACTION MODES                                                           │
│  ─────────────────                                                           │
│  - Touch (tap, long-press, swipe)                                           │
│  - Keyboard (on-screen keyboard)                                            │
│  - Voice input (accessibility)                                              │
│  - Screen reader (VoiceOver, TalkBack)                                      │
│                                                                              │
│  TEST SCENARIOS PER DEVICE                                                   │
│  ─────────────────────────                                                   │
│  1. Image Loading                                                            │
│     - [ ] Images load at correct resolution (srcset)                        │
│     - [ ] Lazy loading works                                                │
│     - [ ] WebP/AVIF fallback for older devices                              │
│                                                                              │
│  2. Touch Targets (WCAG 2.5.5)                                              │
│     - [ ] All touch targets ≥ 44×44 CSS pixels                              │
│     - [ ] Adequate spacing between targets (≥8px)                           │
│     - [ ] No overlapping touch areas                                        │
│                                                                              │
│  3. Horizontal Scroll                                                        │
│     - [ ] No horizontal overflow at any breakpoint                          │
│     - [ ] Tables scroll within container, not page                          │
│     - [ ] Images constrained to viewport                                    │
│                                                                              │
│  4. Orientation Change                                                       │
│     - [ ] Layout adapts correctly                                           │
│     - [ ] Form state preserved                                              │
│     - [ ] No content cut off                                                │
│     - [ ] Media queries trigger appropriately                               │
│                                                                              │
│  5. Text/Typography                                                          │
│     - [ ] Minimum font size 16px for inputs                                 │
│     - [ ] Line height adequate for touch                                    │
│     - [ ] No text truncation without ellipsis                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Playwright Mobile Tests (4 điểm)**

```javascript
const { test, expect, devices } = require('@playwright/test');

// Test on multiple devices
const deviceList = [
  'iPhone 14 Pro',
  'iPhone SE',
  'Pixel 7',
  'Galaxy S21',
];

test.describe('Mobile Responsive Tests', () => {

  for (const deviceName of deviceList) {
    test.describe(`${deviceName}`, () => {
      test.use({ ...devices[deviceName] });

      // Issue 1: Images not displaying correctly
      test('images should load with correct resolution', async ({ page }) => {
        await page.goto('/products');
        await page.waitForLoadState('networkidle');

        const images = page.locator('img.product-image');
        const count = await images.count();

        for (let i = 0; i < count; i++) {
          const img = images.nth(i);

          // Check image loaded
          const isLoaded = await img.evaluate(el => el.complete && el.naturalHeight > 0);
          expect(isLoaded).toBe(true);

          // Check srcset attribute exists for responsive images
          const srcset = await img.getAttribute('srcset');
          expect(srcset).toBeTruthy();

          // Verify correct size loaded based on DPR
          const currentSrc = await img.evaluate(el => el.currentSrc);
          const dpr = await page.evaluate(() => window.devicePixelRatio);

          // For 2x/3x displays, should load larger image
          if (dpr >= 2) {
            expect(currentSrc).toMatch(/@2x|_large|w_\d{3,}/);
          }
        }
      });

      // Issue 2: Touch targets too small
      test('touch targets should be at least 44x44 pixels', async ({ page }) => {
        await page.goto('/');

        const interactiveElements = page.locator('a, button, input, select, [role="button"], [onclick]');
        const count = await interactiveElements.count();

        const tooSmall = [];

        for (let i = 0; i < count; i++) {
          const element = interactiveElements.nth(i);

          // Skip hidden elements
          if (!await element.isVisible()) continue;

          const box = await element.boundingBox();
          if (!box) continue;

          // WCAG 2.5.5: 44x44 CSS pixels minimum
          if (box.width < 44 || box.height < 44) {
            const text = await element.textContent();
            const tagName = await element.evaluate(el => el.tagName);
            tooSmall.push({
              element: `${tagName}: "${text?.trim().substring(0, 20)}"`,
              width: box.width,
              height: box.height
            });
          }
        }

        expect(tooSmall, `Touch targets too small: ${JSON.stringify(tooSmall, null, 2)}`).toEqual([]);
      });

      // Issue 3: Horizontal scroll
      test('should not have horizontal scroll', async ({ page }) => {
        const testPages = ['/', '/products', '/cart', '/checkout'];

        for (const url of testPages) {
          await page.goto(url);
          await page.waitForLoadState('networkidle');

          const hasHorizontalScroll = await page.evaluate(() => {
            return document.documentElement.scrollWidth > document.documentElement.clientWidth;
          });

          expect(hasHorizontalScroll, `Horizontal scroll detected on ${url}`).toBe(false);

          // Also check for overflow elements
          const overflowElements = await page.evaluate(() => {
            const elements = [];
            document.querySelectorAll('*').forEach(el => {
              const rect = el.getBoundingClientRect();
              if (rect.right > window.innerWidth + 5) { // 5px tolerance
                elements.push({
                  tag: el.tagName,
                  class: el.className,
                  right: rect.right,
                  windowWidth: window.innerWidth
                });
              }
            });
            return elements;
          });

          expect(overflowElements, `Elements overflow viewport on ${url}`).toEqual([]);
        }
      });

      // Issue 4: Text overlap on rotation
      test('should handle orientation change without text overlap', async ({ page }) => {
        await page.goto('/products');

        // Portrait mode
        await page.setViewportSize({ width: 393, height: 852 });
        await page.waitForLoadState('networkidle');

        // Capture text positions in portrait
        const portraitLayout = await getTextLayout(page);

        // Rotate to landscape
        await page.setViewportSize({ width: 852, height: 393 });
        await page.waitForTimeout(500); // Allow reflow

        // Check for text overlap
        const hasOverlap = await page.evaluate(() => {
          const textElements = document.querySelectorAll('p, h1, h2, h3, span, a, button');
          const rects = Array.from(textElements)
            .filter(el => el.offsetParent !== null)
            .map(el => el.getBoundingClientRect());

          // Check for overlapping rectangles
          for (let i = 0; i < rects.length; i++) {
            for (let j = i + 1; j < rects.length; j++) {
              const a = rects[i];
              const b = rects[j];

              // Check if rectangles overlap
              const overlaps = !(
                a.right < b.left ||
                a.left > b.right ||
                a.bottom < b.top ||
                a.top > b.bottom
              );

              // Small overlap (< 5px) is acceptable
              if (overlaps) {
                const overlapArea = Math.max(0, Math.min(a.right, b.right) - Math.max(a.left, b.left)) *
                                   Math.max(0, Math.min(a.bottom, b.bottom) - Math.max(a.top, b.top));
                if (overlapArea > 25) { // More than 5x5px overlap
                  return true;
                }
              }
            }
          }
          return false;
        });

        expect(hasOverlap).toBe(false);

        // Rotate back to portrait
        await page.setViewportSize({ width: 393, height: 852 });
        await page.waitForTimeout(500);

        // Verify layout restored
        const restoredLayout = await getTextLayout(page);
        expect(restoredLayout.elementCount).toBe(portraitLayout.elementCount);
      });
    });
  }
});

async function getTextLayout(page) {
  return await page.evaluate(() => {
    const textElements = document.querySelectorAll('p, h1, h2, h3, span');
    return {
      elementCount: textElements.length,
      positions: Array.from(textElements)
        .filter(el => el.offsetParent !== null)
        .slice(0, 10)
        .map(el => {
          const rect = el.getBoundingClientRect();
          return { top: rect.top, left: rect.left, width: rect.width };
        })
    };
  });
}
```

**c) Fixes and Prevention (3 điểm)**

```css
/* ====== FIXES FOR REPORTED ISSUES ====== */

/* Issue 1: Images not displaying correctly on iPhone */
/* Problem: Fixed width images, missing srcset, no WebP fallback */

/* BEFORE */
img.product-image {
  width: 300px;
  height: 300px;
}

/* AFTER */
img.product-image {
  width: 100%;
  height: auto;
  max-width: 100%;
  object-fit: cover;
  aspect-ratio: 1 / 1;
}

/* HTML with srcset */
/*
<picture>
  <source
    type="image/webp"
    srcset="product-300.webp 300w,
            product-600.webp 600w,
            product-900.webp 900w"
    sizes="(max-width: 375px) 100vw, (max-width: 768px) 50vw, 300px"
  >
  <img
    src="product-300.jpg"
    srcset="product-300.jpg 300w,
            product-600.jpg 600w,
            product-900.jpg 900w"
    sizes="(max-width: 375px) 100vw, (max-width: 768px) 50vw, 300px"
    alt="Product name"
    loading="lazy"
    class="product-image"
  >
</picture>
*/


/* Issue 2: Touch targets too small on Android */
/* Problem: Buttons < 44px, links too close together */

/* BEFORE */
.btn-small {
  padding: 4px 8px;
  font-size: 12px;
}

/* AFTER */
.btn-small {
  /* Minimum 44x44 touch target (WCAG 2.5.5) */
  min-height: 44px;
  min-width: 44px;
  padding: 10px 16px;
  font-size: 14px;
  /* Add spacing between adjacent buttons */
  margin: 4px;
}

/* For inline links, use padding to increase touch area */
a {
  /* Visual padding without affecting layout */
  position: relative;
}

a::before {
  content: '';
  position: absolute;
  top: -10px;
  right: -10px;
  bottom: -10px;
  left: -10px;
}


/* Issue 3: Horizontal scroll appearing unexpectedly */
/* Problem: Fixed-width elements, images overflowing, tables */

/* Global prevention */
html, body {
  overflow-x: hidden;
  max-width: 100vw;
}

/* Constrain all elements */
* {
  max-width: 100%;
  box-sizing: border-box;
}

/* Images */
img {
  max-width: 100%;
  height: auto;
}

/* Tables */
.table-container {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
  max-width: 100%;
}

table {
  min-width: 100%;
}

/* Fixed-position elements */
.sidebar {
  width: 100%;
  max-width: 100vw;
}


/* Issue 4: Text overlap when rotating screen */
/* Problem: Absolute positioning, fixed heights, missing media queries */

/* BEFORE */
.product-card {
  position: absolute;
  height: 400px;
}

.product-title {
  position: absolute;
  top: 350px;
}

/* AFTER */
.product-card {
  position: relative;
  display: flex;
  flex-direction: column;
  height: auto;
  min-height: 200px;
}

.product-title {
  position: relative;
  margin-top: auto;
  /* Allow text to wrap and grow */
  overflow-wrap: break-word;
  word-wrap: break-word;
  hyphens: auto;
}

/* Orientation-specific adjustments */
@media (orientation: landscape) {
  .product-card {
    flex-direction: row;
    min-height: 150px;
  }

  .product-image {
    width: 40%;
    flex-shrink: 0;
  }

  .product-info {
    width: 60%;
    padding: 10px;
  }
}

/* ====== PREVENTION MEASURES ====== */

/*
1. Stylelint rules:
   - plugin/no-unsupported-browser-features
   - selector-max-specificity (avoid !important)
   - unit-no-unknown (prevent wrong units)

2. CI/CD checks:
   - Mobile viewport tests in PR pipeline
   - Lighthouse mobile audit score > 90
   - Touch target size automated check

3. Design system:
   - Minimum touch target: 44x44px variable
   - Standard breakpoints only
   - Tested component library

4. Code review checklist:
   - [ ] Tested on real devices
   - [ ] No fixed widths for containers
   - [ ] Images have srcset
   - [ ] Touch targets verified
*/
```

---

## Câu 9: Applitools Ultrafast Grid Configuration (10 điểm)

### Tình huống
Công ty cần test visual consistency across:
- 5 browsers: Chrome, Firefox, Safari, Edge, IE11
- 3 viewports: Desktop (1920), Tablet (768), Mobile (375)
- 2 operating systems: Windows, Mac

Current approach: Run Playwright tests 30 times (5×3×2). Takes 2 hours.

### Yêu cầu
a) Design Ultrafast Grid configuration để optimize (3 điểm)
b) Write test code với configuration (4 điểm)
c) Calculate time/cost savings (3 điểm)

### Đáp án mẫu

**a) Ultrafast Grid Configuration Design (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ULTRAFAST GRID OPTIMIZATION                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  BEFORE (Traditional Sequential Testing)                                     │
│  ────────────────────────────────────────                                    │
│                                                                              │
│   Test Run 1: Chrome Desktop Windows     → 4 min                            │
│   Test Run 2: Chrome Desktop Mac         → 4 min                            │
│   Test Run 3: Chrome Tablet Windows      → 4 min                            │
│   ...                                                                        │
│   Test Run 30: IE11 Mobile Windows       → 4 min                            │
│   ───────────────────────────────────────────────                           │
│   TOTAL: 30 runs × 4 min = 120 minutes (2 hours)                            │
│   Infrastructure: 30 browser instances                                       │
│                                                                              │
│                                                                              │
│  AFTER (Ultrafast Grid)                                                      │
│  ──────────────────────                                                      │
│                                                                              │
│   ┌─────────────────┐                                                       │
│   │   Local Test    │ → Captures DOM/CSS (4 min)                            │
│   │   (1 browser)   │                                                       │
│   └────────┬────────┘                                                       │
│            │                                                                 │
│            ▼                                                                 │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      APPLITOOLS CLOUD                                │   │
│   │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │   │
│   │  │ Chrome  │ │ Firefox │ │ Safari  │ │  Edge   │ │  IE11   │       │   │
│   │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘       │   │
│   │       │           │           │           │           │              │   │
│   │  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐       │   │
│   │  │1920│768│375│1920│768│375│1920│768│375│1920│768│375│1920│768│375│   │
│   │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘       │   │
│   │                                                                      │   │
│   │  Parallel rendering: ~30 seconds                                    │   │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   TOTAL: 4.5 minutes (96% faster)                                           │
│   Infrastructure: 1 browser locally                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Test Code with Configuration (4 điểm)**

```javascript
const { test } = require('@playwright/test');
const {
  Eyes,
  Target,
  Configuration,
  BatchInfo,
  BrowserType,
  DeviceName,
  ScreenOrientation,
  VisualGridRunner
} = require('@applitools/eyes-playwright');

// Create a runner with concurrency
const runner = new VisualGridRunner({ testConcurrency: 10 });

test.describe('Cross-Browser Visual Tests', () => {
  let eyes;
  let configuration;

  test.beforeAll(() => {
    // Create configuration with all browser/viewport combinations
    configuration = new Configuration();

    // API Key
    configuration.setApiKey(process.env.APPLITOOLS_API_KEY);

    // Batch info for grouping results
    configuration.setBatch(new BatchInfo('Cross-Browser Test Suite'));

    // ========== BROWSER CONFIGURATIONS ==========

    // Desktop Browsers (1920x1080)
    configuration.addBrowser(1920, 1080, BrowserType.CHROME);
    configuration.addBrowser(1920, 1080, BrowserType.FIREFOX);
    configuration.addBrowser(1920, 1080, BrowserType.SAFARI);
    configuration.addBrowser(1920, 1080, BrowserType.EDGE_CHROMIUM);
    configuration.addBrowser(1920, 1080, BrowserType.IE_11);

    // Tablet Browsers (768x1024)
    configuration.addBrowser(768, 1024, BrowserType.CHROME);
    configuration.addBrowser(768, 1024, BrowserType.FIREFOX);
    configuration.addBrowser(768, 1024, BrowserType.SAFARI);
    configuration.addBrowser(768, 1024, BrowserType.EDGE_CHROMIUM);

    // Mobile Browsers (375x812)
    configuration.addBrowser(375, 812, BrowserType.CHROME);
    configuration.addBrowser(375, 812, BrowserType.FIREFOX);
    configuration.addBrowser(375, 812, BrowserType.SAFARI);

    // Real Device Emulation
    configuration.addDeviceEmulation(DeviceName.iPhone_14, ScreenOrientation.PORTRAIT);
    configuration.addDeviceEmulation(DeviceName.Pixel_7, ScreenOrientation.PORTRAIT);
    configuration.addDeviceEmulation(DeviceName.iPad_Pro_12_9, ScreenOrientation.LANDSCAPE);

    // ========== VISUAL SETTINGS ==========

    // Match level (Layout for dynamic content, Strict for pixel-perfect)
    configuration.setMatchLevel('Layout');

    // Ignore caret in text fields
    configuration.setIgnoreCaret(true);

    // Hide scrollbars
    configuration.setHideScrollbars(true);

    // Wait before capture
    configuration.setWaitBeforeScreenshots(100);
  });

  test.beforeEach(async ({ page }) => {
    eyes = new Eyes(runner);
    eyes.setConfiguration(configuration);
  });

  test('homepage visual consistency', async ({ page }) => {
    await eyes.open(page, 'E-Commerce App', 'Homepage');

    await page.goto('https://example.com');
    await page.waitForLoadState('networkidle');

    // Full page capture - will render on all 15+ configurations
    await eyes.check('Homepage Full', Target.window()
      .fully()
      .ignoreRegions('.dynamic-banner', '.timestamp')
      .layoutRegions('.product-carousel')
    );

    // Navigation component
    await eyes.check('Navigation', Target.region('nav.main-nav'));

    // Footer component
    await eyes.check('Footer', Target.region('footer'));

    await eyes.close(false); // false = don't throw on diff
  });

  test('product listing page', async ({ page }) => {
    await eyes.open(page, 'E-Commerce App', 'Product Listing');

    await page.goto('https://example.com/products');
    await page.waitForLoadState('networkidle');

    // Wait for images
    await page.waitForFunction(() =>
      Array.from(document.images).every(img => img.complete)
    );

    await eyes.check('Product Grid', Target.window()
      .fully()
      .ignoreRegions('.price', '.stock-count')
      .layoutRegions('.product-grid')
    );

    await eyes.close(false);
  });

  test('checkout flow', async ({ page }) => {
    await eyes.open(page, 'E-Commerce App', 'Checkout');

    await page.goto('https://example.com/checkout');

    // Step 1: Shipping
    await eyes.check('Checkout - Shipping', Target.window().fully());

    // Fill form and proceed
    await page.fill('#name', 'Test User');
    await page.fill('#address', '123 Test St');
    await page.click('[data-test="continue"]');

    // Step 2: Payment
    await page.waitForSelector('.payment-step');
    await eyes.check('Checkout - Payment', Target.window()
      .fully()
      .ignoreRegions('.card-input') // Sensitive field
    );

    await eyes.close(false);
  });

  test.afterEach(async () => {
    await eyes.abort();
  });

  test.afterAll(async () => {
    // Get all results
    const results = await runner.getAllTestResults(false);
    console.log(results.toString());
  });
});
```

**c) Time/Cost Savings Calculation (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    TIME & COST ANALYSIS                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TIME SAVINGS                                                                │
│  ────────────                                                                │
│                                                                              │
│  Traditional Approach:                                                       │
│  ┌──────────────────────────────────────────────────────────────┐           │
│  │ Per test: 4 minutes                                          │           │
│  │ Configurations: 5 browsers × 3 viewports × 2 OS = 30        │           │
│  │ Total per test: 30 × 4 min = 120 minutes                    │           │
│  │                                                              │           │
│  │ For 50 pages:                                                │           │
│  │ Sequential: 50 × 120 min = 6,000 minutes = 100 hours        │           │
│  │ With parallelization (10 machines): 100 / 10 = 10 hours     │           │
│  └──────────────────────────────────────────────────────────────┘           │
│                                                                              │
│  Ultrafast Grid Approach:                                                    │
│  ┌──────────────────────────────────────────────────────────────┐           │
│  │ Per test (local): 4 minutes                                  │           │
│  │ Cloud rendering: ~30 seconds (all 30 configs parallel)      │           │
│  │ Total per test: 4.5 minutes                                 │           │
│  │                                                              │           │
│  │ For 50 pages:                                                │           │
│  │ Sequential: 50 × 4.5 min = 225 minutes = 3.75 hours         │           │
│  │ With parallelization (5 workers): 225 / 5 = 45 minutes      │           │
│  └──────────────────────────────────────────────────────────────┘           │
│                                                                              │
│  Time Saved: 10 hours → 45 minutes = 92% reduction                          │
│                                                                              │
│                                                                              │
│  COST SAVINGS                                                                │
│  ────────────                                                                │
│                                                                              │
│  Traditional Approach (Monthly):                                             │
│  ┌──────────────────────────────────────────────────────────────┐           │
│  │ BrowserStack/Sauce Labs:                                     │           │
│  │ - 30 parallel sessions needed                                │           │
│  │ - Enterprise plan: ~$3,000/month                            │           │
│  │                                                              │           │
│  │ CI/CD compute (10 machines × 10 hours × 30 runs/month):     │           │
│  │ - GitHub Actions: 3,000 minutes × $0.008 = $24/run          │           │
│  │ - Monthly: $24 × 30 = $720                                  │           │
│  │                                                              │           │
│  │ QA time (reviewing false positives):                        │           │
│  │ - 5 hours/week × $50/hour = $1,000/month                    │           │
│  │                                                              │           │
│  │ TOTAL: ~$4,720/month                                        │           │
│  └──────────────────────────────────────────────────────────────┘           │
│                                                                              │
│  Ultrafast Grid Approach (Monthly):                                          │
│  ┌──────────────────────────────────────────────────────────────┐           │
│  │ Applitools:                                                  │           │
│  │ - Growth plan: $969/month (unlimited browsers)              │           │
│  │                                                              │           │
│  │ CI/CD compute (1 machine × 45 min × 30 runs/month):         │           │
│  │ - GitHub Actions: 1,350 minutes × $0.008 = $10.80/run       │           │
│  │ - Monthly: $10.80 × 30 = $324                               │           │
│  │                                                              │           │
│  │ QA time (reduced false positives with Visual AI):           │           │
│  │ - 1 hour/week × $50/hour = $200/month                       │           │
│  │                                                              │           │
│  │ TOTAL: ~$1,493/month                                        │           │
│  └──────────────────────────────────────────────────────────────┘           │
│                                                                              │
│  Cost Saved: $4,720 → $1,493 = 68% reduction ($3,227/month)                 │
│                                                                              │
│                                                                              │
│  SUMMARY                                                                     │
│  ───────                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐           │
│  │                          Before      After      Savings      │           │
│  │  ─────────────────────────────────────────────────────────   │           │
│  │  Test Time (per run)    10 hours    45 min     92%          │           │
│  │  Monthly Cost           $4,720      $1,493     68%          │           │
│  │  Browser Instances      30          1          97%          │           │
│  │  False Positives        High        Low        78%          │           │
│  │  Maintenance Effort     10 hr/wk    2 hr/wk    80%          │           │
│  └──────────────────────────────────────────────────────────────┘           │
│                                                                              │
│  ROI: $3,227/month × 12 = $38,724/year saved                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 10: Comprehensive GUI Testing Strategy (10 điểm)

### Tình huống
Bạn là QA Lead cho dự án redesign website banking. Requirements:
- 80 screens cần test
- WCAG 2.1 AA compliance required
- Support 3 browsers (Chrome, Safari, Firefox)
- 3 breakpoints (desktop, tablet, mobile)
- 2 themes (light, dark)
- Release every 2 weeks

### Yêu cầu
a) Design comprehensive GUI testing strategy (4 điểm)
b) Create test execution plan với priorities (3 điểm)
c) Define success metrics và KPIs (3 điểm)

### Đáp án mẫu

**a) Comprehensive GUI Testing Strategy (4 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BANKING WEBSITE GUI TESTING STRATEGY                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. TESTING LAYERS                                                           │
│  ─────────────────                                                           │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Layer 1: Component Testing (Design System)                          │   │
│   │  ─────────────────────────────────────────                           │   │
│   │  Tool: Storybook + Chromatic                                        │   │
│   │  Scope: Buttons, Forms, Cards, Modals, Tables                       │   │
│   │  Coverage: All component states, themes, sizes                       │   │
│   │  Frequency: Every commit to component library                       │   │
│   │  Owner: Frontend developers                                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                         │                                                    │
│                         ▼                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Layer 2: Page-Level Visual Regression                               │   │
│   │  ───────────────────────────────────────                             │   │
│   │  Tool: Applitools Ultrafast Grid                                    │   │
│   │  Scope: 80 screens × 3 browsers × 3 viewports × 2 themes           │   │
│   │        = 1,440 visual checkpoints                                   │   │
│   │  Frequency: Every PR (critical pages), nightly (all pages)         │   │
│   │  Owner: QA engineers                                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                         │                                                    │
│                         ▼                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Layer 3: Accessibility Testing                                      │   │
│   │  ──────────────────────────────                                      │   │
│   │  Tool: axe-core + manual audits                                     │   │
│   │  Standard: WCAG 2.1 AA                                              │   │
│   │  Scope: All pages, all interactive elements                         │   │
│   │  Frequency: Every PR (automated), monthly (manual audit)           │   │
│   │  Owner: QA + UX designer                                            │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                         │                                                    │
│                         ▼                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Layer 4: Usability Testing                                          │   │
│   │  ─────────────────────────                                           │   │
│   │  Method: Heuristic evaluation + user testing                        │   │
│   │  Scope: Critical flows (login, transfer, payments)                  │   │
│   │  Frequency: Before major releases                                   │   │
│   │  Owner: UX researcher + Product                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│                                                                              │
│  2. TEST ENVIRONMENT MATRIX                                                  │
│  ──────────────────────────                                                  │
│                                                                              │
│   ┌─────────────────┬──────────────────────────────────────────────────┐    │
│   │ Dimension       │ Configurations                                    │    │
│   ├─────────────────┼──────────────────────────────────────────────────┤    │
│   │ Browsers        │ Chrome (latest), Safari (latest), Firefox (latest)│    │
│   │ Viewports       │ Desktop (1920×1080), Tablet (768×1024),          │    │
│   │                 │ Mobile (375×812)                                  │    │
│   │ Themes          │ Light mode, Dark mode                             │    │
│   │ Languages       │ English, Vietnamese (if applicable)              │    │
│   │ Accessibility   │ Standard mode, High contrast, Screen reader     │    │
│   └─────────────────┴──────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  3. PAGE CATEGORIZATION                                                      │
│  ────────────────────────                                                    │
│                                                                              │
│   Critical (15 pages) - Test on every PR:                                   │
│   - Login, Registration, Password Reset                                     │
│   - Dashboard, Account Summary                                              │
│   - Transfer, Bill Payment, Card Management                                │
│   - Profile, Settings, Logout                                               │
│                                                                              │
│   Important (30 pages) - Test nightly:                                      │
│   - Transaction History, Statements                                         │
│   - Notifications, Messages                                                 │
│   - Loan Applications, Investment                                           │
│                                                                              │
│   Standard (35 pages) - Test weekly/pre-release:                           │
│   - Help, FAQ, Contact                                                      │
│   - Terms, Privacy, About                                                   │
│   - Marketing pages                                                         │
│                                                                              │
│                                                                              │
│  4. AUTOMATION FRAMEWORK                                                     │
│  ───────────────────────                                                     │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                      │   │
│   │   tests/                                                             │   │
│   │   ├── visual/                                                        │   │
│   │   │   ├── critical/           # Run every PR                        │   │
│   │   │   │   ├── login.spec.ts                                         │   │
│   │   │   │   ├── dashboard.spec.ts                                     │   │
│   │   │   │   └── transfer.spec.ts                                      │   │
│   │   │   ├── nightly/            # Run every night                     │   │
│   │   │   └── weekly/             # Run before release                  │   │
│   │   ├── accessibility/                                                 │   │
│   │   │   ├── wcag-audit.spec.ts                                        │   │
│   │   │   └── keyboard-nav.spec.ts                                      │   │
│   │   ├── responsive/                                                    │   │
│   │   │   └── breakpoint-tests.spec.ts                                  │   │
│   │   └── themes/                                                        │   │
│   │       └── dark-mode.spec.ts                                         │   │
│   │                                                                      │   │
│   │   fixtures/                                                          │   │
│   │   ├── visual-test-setup.ts    # Applitools configuration           │   │
│   │   └── mock-data.ts            # Consistent test data                │   │
│   │                                                                      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Test Execution Plan (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    2-WEEK SPRINT TEST EXECUTION PLAN                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DAY 1-3: Development Phase                                                  │
│  ─────────────────────────                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Trigger: Every PR/Commit                                             │    │
│  │ Tests:                                                               │    │
│  │   ✓ Component visual tests (Chromatic) - 5 min                      │    │
│  │   ✓ Critical page visual tests (15 pages × 3 browsers) - 10 min     │    │
│  │   ✓ Accessibility scan (axe-core) - 3 min                           │    │
│  │ Gate: Must pass to merge                                            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 4-8: Integration Phase                                                  │
│  ─────────────────────────                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Trigger: Nightly at 2 AM                                             │    │
│  │ Tests:                                                               │    │
│  │   ✓ Full visual regression (80 pages × full matrix) - 45 min        │    │
│  │   ✓ Cross-browser consistency check - 20 min                        │    │
│  │   ✓ Theme switching tests (light/dark) - 15 min                     │    │
│  │   ✓ Responsive breakpoint tests - 20 min                            │    │
│  │ Report: Slack notification, dashboard update                         │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 9: Pre-Release Testing                                                  │
│  ─────────────────────────                                                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Trigger: Release candidate tagged                                    │    │
│  │ Tests:                                                               │    │
│  │   ✓ Complete visual baseline comparison - 1 hour                    │    │
│  │   ✓ Full WCAG 2.1 AA audit - 2 hours                               │    │
│  │   ✓ Heuristic evaluation (critical flows) - 2 hours                │    │
│  │   ✓ Real device testing (BrowserStack) - 1 hour                    │    │
│  │ Gate: All critical issues resolved                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  DAY 10: Release Day                                                         │
│  ──────────────────                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Trigger: Production deployment                                       │    │
│  │ Tests:                                                               │    │
│  │   ✓ Smoke visual tests (critical pages) - 10 min                    │    │
│  │   ✓ Production accessibility scan - 5 min                           │    │
│  │   ✓ Cross-browser spot check - 15 min                               │    │
│  │ Rollback criteria: Any critical visual regression                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│                                                                              │
│  PRIORITY MATRIX                                                             │
│  ───────────────                                                             │
│                                                                              │
│   Priority 1 (Block release):                                               │
│   - Authentication flows broken                                             │
│   - Payment/transfer functionality affected                                 │
│   - WCAG Level A violations                                                 │
│   - Complete page not rendering                                             │
│                                                                              │
│   Priority 2 (Fix before next release):                                     │
│   - Visual inconsistencies on critical pages                                │
│   - WCAG Level AA violations                                                │
│   - Cross-browser rendering issues                                          │
│   - Responsive layout breaks                                                │
│                                                                              │
│   Priority 3 (Backlog):                                                     │
│   - Minor visual differences                                                │
│   - Non-critical page issues                                                │
│   - Enhancement suggestions                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**c) Success Metrics and KPIs (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    GUI TESTING SUCCESS METRICS & KPIs                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. QUALITY METRICS                                                          │
│  ──────────────────                                                          │
│                                                                              │
│   ┌─────────────────────┬─────────────┬─────────────┬──────────────────┐    │
│   │ Metric              │ Target      │ Current     │ Measurement      │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Visual Bug Escape   │ < 2/release │ Baseline TBD│ Bugs found in    │    │
│   │ Rate                │             │             │ production       │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ WCAG Compliance     │ 100% AA     │ Audit score │ axe-core report  │    │
│   │ Score               │             │             │                  │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Cross-Browser       │ 100%        │ Visual test │ Applitools       │    │
│   │ Consistency         │             │ pass rate   │ dashboard        │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Visual Test         │ > 95%       │ Calculate   │ Passed / Total   │    │
│   │ Pass Rate           │             │             │ tests            │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ False Positive      │ < 5%        │ Track       │ Incorrect diffs  │    │
│   │ Rate                │             │             │ / Total diffs    │    │
│   └─────────────────────┴─────────────┴─────────────┴──────────────────┘    │
│                                                                              │
│                                                                              │
│  2. EFFICIENCY METRICS                                                       │
│  ─────────────────────                                                       │
│                                                                              │
│   ┌─────────────────────┬─────────────┬─────────────┬──────────────────┐    │
│   │ Metric              │ Target      │ Current     │ Measurement      │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Visual Test         │ < 30 min    │ CI logs     │ Total test time  │    │
│   │ Execution Time      │             │             │ in pipeline      │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Review Turnaround   │ < 4 hours   │ Track       │ Diff detected to │    │
│   │ Time                │             │             │ approved/rejected│    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Test Coverage       │ 100%        │ Calculate   │ Pages tested /   │    │
│   │ (Pages)             │ critical    │             │ Total pages      │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Automation Rate     │ > 90%       │ Track       │ Automated tests  │    │
│   │                     │             │             │ / All GUI tests  │    │
│   ├─────────────────────┼─────────────┼─────────────┼──────────────────┤    │
│   │ Maintenance Hours   │ < 2 hr/week │ Track       │ Time spent on    │    │
│   │                     │             │             │ test maintenance │    │
│   └─────────────────────┴─────────────┴─────────────┴──────────────────┘    │
│                                                                              │
│                                                                              │
│  3. BUSINESS IMPACT METRICS                                                  │
│  ──────────────────────────                                                  │
│                                                                              │
│   ┌─────────────────────┬─────────────┬─────────────────────────────────┐   │
│   │ Metric              │ Target      │ Business Impact                  │   │
│   ├─────────────────────┼─────────────┼─────────────────────────────────┤   │
│   │ Customer complaints │ 0 UI-related│ Customer satisfaction, trust    │   │
│   │ (UI/UX issues)      │ complaints  │                                 │   │
│   ├─────────────────────┼─────────────┼─────────────────────────────────┤   │
│   │ Accessibility       │ 0 violations│ Legal compliance, inclusivity   │   │
│   │ Lawsuits/Complaints │             │                                 │   │
│   ├─────────────────────┼─────────────┼─────────────────────────────────┤   │
│   │ Time to Market      │ < 2 weeks   │ Competitive advantage           │   │
│   │ (Release cycle)     │             │                                 │   │
│   ├─────────────────────┼─────────────┼─────────────────────────────────┤   │
│   │ Hotfix Rate         │ < 1/month   │ Stability, customer trust       │   │
│   │ (UI-related)        │             │                                 │   │
│   └─────────────────────┴─────────────┴─────────────────────────────────┘   │
│                                                                              │
│                                                                              │
│  4. DASHBOARD & REPORTING                                                    │
│  ────────────────────────                                                    │
│                                                                              │
│   Weekly Report includes:                                                    │
│   - Visual test pass/fail trend                                             │
│   - Accessibility score trend                                               │
│   - Top 5 pages with most visual changes                                    │
│   - Cross-browser issue summary                                             │
│   - Maintenance effort log                                                  │
│                                                                              │
│   Monthly Executive Report:                                                  │
│   - Visual bug escape rate vs target                                        │
│   - WCAG compliance status                                                  │
│   - ROI analysis (bugs caught vs cost)                                      │
│   - Customer feedback related to UI                                         │
│   - Recommendations for improvement                                         │
│                                                                              │
│                                                                              │
│  5. CONTINUOUS IMPROVEMENT                                                   │
│  ─────────────────────────                                                   │
│                                                                              │
│   Quarterly Review:                                                          │
│   - Analyze false positive patterns                                         │
│   - Update baseline review process                                          │
│   - Evaluate new tools/techniques                                           │
│   - Adjust test coverage based on risk                                      │
│   - Training on new accessibility guidelines                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tổng kết

| Câu | Topic | Điểm | Độ khó |
|-----|-------|------|--------|
| 1 | Visual Regression Strategy | 10 | Medium |
| 2 | Cross-Browser Debugging | 10 | Hard |
| 3 | Accessibility Audit | 10 | Medium |
| 4 | Responsive Design Testing | 10 | Medium |
| 5 | Dynamic Content Handling | 10 | Hard |
| 6 | Usability Heuristic Evaluation | 10 | Medium |
| 7 | Visual Testing Pipeline | 10 | Hard |
| 8 | Mobile Responsive Testing | 10 | Medium |
| 9 | Applitools Ultrafast Grid | 10 | Hard |
| 10 | Comprehensive Strategy | 10 | Hard |

**Tổng: 100 điểm**

---

*Created for CS423 Software Testing Final Exam Preparation*
*Focus: Application-level questions with practical scenarios*
