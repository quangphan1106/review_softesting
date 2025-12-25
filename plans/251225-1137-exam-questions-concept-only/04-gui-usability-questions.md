# GUI Testing & Usability - Concept Questions

## Question 1: Visual Regression Testing (15 points)

### Scenario
ToolShop is planning to redesign their product cards. You need to set up visual regression testing to catch unintended visual changes.

### Questions

**a) (5 points)** What is visual regression testing and how does it differ from functional testing?

**Expected Answer:**

**Visual regression testing:** Automated comparison of screenshots to detect unintended visual changes between versions.

**Comparison with functional testing:**

| Aspect | Functional Testing | Visual Regression Testing |
|--------|-------------------|--------------------------|
| **Verifies** | Behavior works correctly | UI looks correct |
| **Catches** | Logic errors, broken features | Layout shifts, color changes, missing elements |
| **Method** | Assert on DOM/data | Compare screenshots pixel-by-pixel |
| **Example** | "Add to cart button adds item" | "Add to cart button is green and 44px tall" |
| **Output** | Pass/Fail | Baseline image vs Current image diff |

**Why both are needed:**
- Functional: Button works → passes
- Visual: Button moved 50px left → functional still passes, visual catches it

**b) (5 points)** Describe THREE approaches to visual regression testing and their trade-offs.

**Expected Answer:**

| Approach | How It Works | Pros | Cons |
|----------|--------------|------|------|
| **Pixel-by-pixel comparison** | Exact pixel match between screenshots | Simple, catches all changes | Sensitive to anti-aliasing, fonts, sub-pixel rendering |
| **Perceptual diff** | AI-based comparison ignoring minor variations | Tolerates rendering differences | May miss subtle changes, slower |
| **DOM-based snapshot** | Compare serialized DOM structure | Fast, deterministic | Misses CSS-only changes, positioning |

**Tolerance strategies:**

| Strategy | Use When |
|----------|----------|
| **Strict (0% tolerance)** | Critical branding elements, logos |
| **Moderate (0.1-1%)** | General UI, accounts for rendering |
| **Region-based** | Dynamic content areas excluded |

**c) (5 points)** What are the key challenges in implementing visual regression testing and how would you address them?

**Expected Answer:**

| Challenge | Description | Solution |
|-----------|-------------|----------|
| **Dynamic content** | Timestamps, ads, random data | Mask dynamic regions, use consistent test data |
| **Cross-browser differences** | Same page renders differently | Separate baselines per browser |
| **Animation timing** | Screenshots during animations | Wait for animations, disable animations in tests |
| **Environment inconsistency** | Different CI runners, fonts | Containerized testing, font embedding |
| **Baseline management** | When to update baselines | Review process, intentional vs accidental |
| **False positives** | Minor changes cause failures | Tunable thresholds, smart diffing |

**Baseline workflow:**
```
Developer makes change
         │
         ▼
┌─────────────────────┐
│ Visual test detects │
│    difference       │
└──────────┬──────────┘
           │
    ┌──────┴──────┐
    ▼             ▼
Intentional   Unintentional
    │             │
    ▼             ▼
Update       Fix the bug
baseline     Re-run test
```

---

## Question 2: Accessibility Testing (20 points)

### Scenario
ToolShop must comply with WCAG 2.1 AA standards. The checkout form has:
- Input fields for personal information
- Dropdown for country selection
- Radio buttons for shipping options
- Submit button

### Questions

**a) (5 points)** Explain WCAG and its four core principles (POUR).

**Expected Answer:**

**WCAG**: Web Content Accessibility Guidelines - international standards for web accessibility.

**POUR Principles:**

| Principle | Meaning | Examples |
|-----------|---------|----------|
| **Perceivable** | Users can perceive content | Alt text for images, captions for video, sufficient color contrast |
| **Operable** | Users can interact with UI | Keyboard navigation, no seizure-triggering content, sufficient time limits |
| **Understandable** | Users can understand content | Readable text, predictable navigation, error identification |
| **Robust** | Works with assistive technologies | Valid HTML, ARIA labels, compatible with screen readers |

**Conformance levels:**
```
Level A   ── Minimum accessibility (must-have)
Level AA  ── Standard accessibility (most laws require)
Level AAA ── Enhanced accessibility (aspirational)
```

**b) (5 points)** What are ARIA attributes and why are they important? Give THREE examples for form elements.

**Expected Answer:**

**ARIA:** Accessible Rich Internet Applications - attributes that provide additional semantics for assistive technologies.

**Why important:**
- HTML alone doesn't convey all UI semantics
- Dynamic content (SPAs) need extra context
- Custom components lack native accessibility

**Form examples:**

| Element | ARIA Attribute | Purpose |
|---------|---------------|---------|
| **Required field** | `aria-required="true"` | Indicates field is mandatory |
| **Error state** | `aria-invalid="true"` + `aria-describedby="error-msg"` | Links error message to field |
| **Field description** | `aria-describedby="hint-text"` | Associates helper text with input |
| **Loading button** | `aria-busy="true"` | Indicates processing state |
| **Expandable section** | `aria-expanded="false"` | Indicates collapsed/expanded state |

**Proper vs improper usage:**

```
BAD:  <span class="error">Please enter email</span>
      (Screen reader doesn't know it relates to email field)

GOOD: <input id="email" aria-describedby="email-error" aria-invalid="true">
      <span id="email-error" role="alert">Please enter email</span>
      (Screen reader announces error in context)
```

**c) (5 points)** Design an accessibility test checklist for ToolShop's checkout form.

**Expected Answer:**

**Accessibility test checklist:**

| Category | Test Item | Pass Criteria |
|----------|-----------|---------------|
| **Keyboard Navigation** | Tab through all fields | Logical order, visible focus |
| | Submit with Enter key | Form submits correctly |
| | Escape closes modals | Returns focus appropriately |
| **Screen Reader** | Labels announced | Every input has accessible name |
| | Errors announced | `role="alert"` or live region |
| | Instructions read | Helper text associated |
| **Visual** | Color contrast | 4.5:1 for text, 3:1 for UI |
| | Focus indicator | Visible focus state |
| | Error indication | Not color-only (icon + text) |
| **Semantics** | Fieldsets for groups | Related fields grouped |
| | Legends for groups | Group purpose described |
| | Button type | Submit button has `type="submit"` |

**d) (5 points)** Explain the difference between automated and manual accessibility testing.

**Expected Answer:**

| Aspect | Automated Testing | Manual Testing |
|--------|-------------------|----------------|
| **What it checks** | Technical standards (alt text exists, contrast ratio) | Usability (alt text is meaningful, flow makes sense) |
| **Coverage** | ~30% of WCAG criteria | ~100% of WCAG criteria |
| **Speed** | Fast, runs in CI/CD | Slow, requires human judgment |
| **Consistency** | Same results every run | Varies by tester expertise |
| **Examples** | Missing labels, insufficient contrast | Confusing instructions, poor reading order |

**Testing pyramid for accessibility:**
```
           /\
          /  \  Manual audit (quarterly)
         /    \ Real user testing
        /──────\
       /        \ Semi-automated
      /          \ Keyboard testing
     /            \ Screen reader spot checks
    /──────────────\
   /                \ Automated
  /                  \ axe-core, Lighthouse
 /                    \ WAVE, Pa11y
/──────────────────────\
```

**Best practice:** Automated catches ~30% of issues; essential to also do manual testing.

---

## Question 3: Responsive Design Testing (15 points)

### Scenario
ToolShop targets these devices:
- Mobile: 320px - 767px
- Tablet: 768px - 1023px
- Desktop: 1024px+

Product listing shows:
- Mobile: 1 column, stacked filters
- Tablet: 2 columns, collapsible filters
- Desktop: 4 columns, sidebar filters

### Questions

**a) (5 points)** What breakpoints would you test and why?

**Expected Answer:**

**Test breakpoints:**

| Breakpoint | Width | Why Test |
|------------|-------|----------|
| **Smallest mobile** | 320px | iPhone SE, edge case |
| **Typical mobile** | 375px | iPhone X/12/13 |
| **Mobile-tablet boundary** | 767px | Just before layout change |
| **Tablet start** | 768px | Layout change trigger |
| **Typical tablet** | 820px | iPad portrait |
| **Tablet-desktop boundary** | 1023px | Just before layout change |
| **Desktop start** | 1024px | Desktop layout trigger |
| **Large desktop** | 1440px | Common large monitor |
| **Extra large** | 1920px | Full HD, scaling test |

**Why boundary testing matters:**
```
Content behavior at breakpoints:

Width:  320   767 768   1023 1024   1920
        │      │  │       │   │      │
Layout: ├───1 col──┼──2 col──┼──4 col──┤
        │      │  │       │   │      │
        Mobile │  Tablet   │  Desktop │
               ↑           ↑
        Boundary: high risk of bugs
```

**b) (5 points)** What elements should be tested specifically for responsive behavior?

**Expected Answer:**

| Element | Mobile Test | Tablet Test | Desktop Test |
|---------|-------------|-------------|--------------|
| **Navigation** | Hamburger menu works | Partial menu visible | Full menu visible |
| **Images** | Appropriate size, fast load | Medium resolution | Full resolution |
| **Typography** | Readable without zoom (16px min) | Proportional scaling | Optimal line length |
| **Touch targets** | 44x44px minimum | Touch-friendly | Can be smaller |
| **Forms** | Proper keyboard types, autocomplete | Two-column possible | Multi-column layout |
| **Tables** | Horizontal scroll or card layout | Partial columns | Full table |
| **Modals** | Full-screen takeover | Centered, sized | Sized appropriately |

**Critical responsive tests:**

| Test | What Could Break |
|------|------------------|
| **Content overflow** | Text/images breaking container |
| **Touch target overlap** | Buttons too close together |
| **Hidden content** | Desktop-only content inaccessible |
| **Scroll behavior** | Horizontal scroll appearing unexpectedly |
| **Input usability** | Keyboard covering form fields |

**c) (5 points)** Compare testing responsive design via viewport resizing vs device emulation vs real devices.

**Expected Answer:**

| Method | Fidelity | Speed | Cost | Best For |
|--------|----------|-------|------|----------|
| **Viewport resizing** | Low | Fast | Free | Quick layout checks |
| **Device emulation** | Medium | Fast | Free | Development testing |
| **Real devices** | High | Slow | High | Final validation |

**What each catches:**

| Issue | Viewport | Emulator | Real Device |
|-------|----------|----------|-------------|
| Layout breaks | ✅ | ✅ | ✅ |
| Touch interactions | ❌ | Partial | ✅ |
| Performance issues | ❌ | Partial | ✅ |
| Gesture behavior | ❌ | ❌ | ✅ |
| Orientation changes | Partial | ✅ | ✅ |
| Browser-specific bugs | ❌ | Partial | ✅ |

---

## Question 4: Usability Heuristics (15 points)

### Scenario
You're evaluating ToolShop's checkout flow against Nielsen's usability heuristics.

Current issues observed:
- No progress indicator on multi-step checkout
- Error messages say "Invalid input" without specifics
- No way to undo removing item from cart
- Loading states unclear

### Questions

**a) (5 points)** List Nielsen's 10 usability heuristics.

**Expected Answer:**

| # | Heuristic | Description |
|---|-----------|-------------|
| 1 | **Visibility of system status** | System informs users what's happening |
| 2 | **Match between system and real world** | Use familiar language and concepts |
| 3 | **User control and freedom** | Support undo, redo, exit |
| 4 | **Consistency and standards** | Follow platform conventions |
| 5 | **Error prevention** | Prevent errors before they happen |
| 6 | **Recognition rather than recall** | Minimize memory load |
| 7 | **Flexibility and efficiency** | Accelerators for expert users |
| 8 | **Aesthetic and minimalist design** | Avoid irrelevant information |
| 9 | **Help users recognize, diagnose, recover from errors** | Clear error messages |
| 10 | **Help and documentation** | Provide searchable help |

**b) (5 points)** Map the observed issues to specific heuristics and suggest fixes.

**Expected Answer:**

| Issue | Violated Heuristic | Fix |
|-------|-------------------|-----|
| **No progress indicator** | #1: Visibility of system status | Add step indicator (Step 2 of 4: Shipping) |
| **Vague error messages** | #9: Error recognition/recovery | "Please enter a valid email address (example: user@domain.com)" |
| **No undo for cart removal** | #3: User control and freedom | Add "Undo" button with timer, or confirmation dialog |
| **Unclear loading states** | #1: Visibility of system status | Add loading spinner with message "Processing payment..." |

**Visual improvements:**
```
BEFORE (No progress indicator):
┌──────────────────────┐
│     Checkout         │
│                      │
│  [Shipping Form]     │
│                      │
└──────────────────────┘

AFTER (Clear progress):
┌──────────────────────┐
│     Checkout         │
│ ○───●───○───○       │
│ Cart Ship Pay  Done  │
│                      │
│  [Shipping Form]     │
│  Step 2 of 4         │
└──────────────────────┘
```

**c) (5 points)** How would you conduct a heuristic evaluation? Describe the process.

**Expected Answer:**

**Heuristic evaluation process:**

| Phase | Activities |
|-------|------------|
| **1. Preparation** | Define scope (which flows), recruit 3-5 evaluators, provide materials |
| **2. Training** | Brief evaluators on heuristics, product context |
| **3. Individual evaluation** | Each evaluator independently reviews interface |
| **4. Documentation** | Record issues with severity, heuristic violated, location |
| **5. Consolidation** | Aggregate findings, remove duplicates |
| **6. Prioritization** | Rate by severity, assign to backlog |

**Severity rating scale:**

| Rating | Definition | Action |
|--------|------------|--------|
| 0 | Not a usability problem | None |
| 1 | Cosmetic only | Fix if time permits |
| 2 | Minor problem | Low priority fix |
| 3 | Major problem | Important to fix |
| 4 | Usability catastrophe | Must fix before release |

**Evaluation template:**
```
Issue #: 001
Location: Checkout - Payment step
Heuristic: #9 Error recognition
Severity: 3 (Major)
Description: When credit card is declined, message
             shows "Transaction failed" without
             explaining why or what to do next.
Recommendation: Show specific reason and next steps:
                "Card declined. Please check card
                 details or try a different payment method."
```

---

## Question 5: Form Usability Testing (15 points)

### Scenario
ToolShop's registration form has these fields:
- Email (required)
- Password (required, 8+ chars, 1 uppercase, 1 number)
- Confirm password (required)
- Full name (required)
- Phone (optional)
- Terms checkbox (required)

Users are abandoning the registration with 40% completion rate.

### Questions

**a) (5 points)** What usability issues might cause high form abandonment?

**Expected Answer:**

| Potential Issue | Why It Causes Abandonment |
|-----------------|--------------------------|
| **Password requirements not shown upfront** | Users fail validation repeatedly |
| **Unclear required vs optional** | Users unsure what to fill |
| **No inline validation** | Errors only shown on submit |
| **Confusing error messages** | Users don't know how to fix |
| **Too many fields** | Feels like too much effort |
| **Poor mobile experience** | Hard to complete on phone |
| **No progress indication** | Users unsure how much remains |
| **Lost data on error** | Browser back loses work |

**Form completion funnel analysis:**
```
100% │████████████████████████████████
     │ Start form (view)
 80% │████████████████████████████
     │ Begin filling (email)
 60% │████████████████████████
     │ Password issues (-20%)
 50% │██████████████████████
     │ Additional fields (-10%)
 40% │████████████████
     │ Complete
     └─────────────────────────────▶
```

**b) (5 points)** Design improvements to increase completion rate.

**Expected Answer:**

| Improvement | Implementation |
|-------------|----------------|
| **Progressive disclosure** | Split into steps: Email → Password → Profile |
| **Inline validation** | Validate each field on blur, show checkmarks |
| **Password strength indicator** | Visual bar showing strength as user types |
| **Smart defaults** | Auto-detect country from locale |
| **Reduce friction** | Social login options (Google, Apple) |
| **Mobile optimization** | Appropriate keyboard types, large touch targets |
| **Show requirements upfront** | Display password rules before user types |
| **Optional field clarity** | "(Optional)" label, visually distinct |

**Password requirements display:**
```
Password:
┌────────────────────────────┐
│ ●●●●●●●●                   │
└────────────────────────────┘
Requirements:
  ✅ At least 8 characters
  ✅ One uppercase letter
  ❌ One number
  ▓▓▓░░ Medium strength
```

**c) (5 points)** What metrics would you track to measure form usability improvement?

**Expected Answer:**

| Metric | What It Measures | Target |
|--------|------------------|--------|
| **Completion rate** | Users who submit / Users who start | >70% |
| **Time to complete** | Average duration | <2 minutes |
| **Error rate** | Submissions with errors / Total attempts | <20% |
| **Field drop-off** | Which field causes abandonment | Identify problem fields |
| **Correction rate** | Fields edited multiple times | <10% per field |
| **Help engagement** | Clicks on help text/tooltips | Low is good |

**Field-level analytics:**
```
Field           Start   Complete  Drop-off  Avg Time  Error Rate
─────────────────────────────────────────────────────────────────
Email           100%    95%       5%        8s        3%
Password        95%     70%       25%       45s       35% ←──Problem!
Confirm         70%     68%       2%        5s        15%
Full Name       68%     65%       3%        6s        2%
Phone           65%     64%       1%        10s       1%
Terms           64%     40%       24%       3s        0% ←──Problem!
```

**Root cause analysis:**
- Password: 35% error rate → requirements unclear
- Terms: 24% drop-off → Users won't agree, or checkbox hard to find

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | Visual Regression Testing | 15 |
| Q2 | Accessibility Testing | 20 |
| Q3 | Responsive Design Testing | 15 |
| Q4 | Usability Heuristics | 15 |
| Q5 | Form Usability Testing | 15 |
| **Total** | | **80** |
