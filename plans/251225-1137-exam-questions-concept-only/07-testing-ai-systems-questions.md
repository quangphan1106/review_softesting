# Testing AI/ML Systems - Concept Questions

## Question 1: AI System Testing Challenges (15 points)

### Scenario
ToolShop is implementing an AI-powered product recommendation system that suggests products based on browsing history, purchase patterns, and similar user behavior.

### Questions

**a) (5 points)** Why is testing AI/ML systems different from testing traditional software?

**Expected Answer:**

| Aspect | Traditional Software | AI/ML Systems |
|--------|---------------------|---------------|
| **Behavior** | Deterministic (same input → same output) | Probabilistic (output may vary) |
| **Correctness** | Clear pass/fail criteria | Acceptable range of outcomes |
| **Test oracle** | Known expected output | No single "correct" answer |
| **Failure modes** | Crashes, wrong values | Subtle degradation, bias |
| **Debugging** | Trace code path | Model is "black box" |
| **Test data** | Specific cases | Statistical distribution matters |

**Key challenges:**
```
Traditional: f(x) = y  (always)
ML Model:   f(x) ≈ y  (with probability p)

Question: When p=0.85, is that passing or failing?
Answer: Depends on business context and baseline
```

**b) (5 points)** What types of testing are specific to AI/ML systems?

**Expected Answer:**

| Test Type | Purpose | Example |
|-----------|---------|---------|
| **Model accuracy testing** | Verify prediction quality | Measure precision/recall on test set |
| **Bias/fairness testing** | Detect discriminatory patterns | Test across demographic groups |
| **Robustness testing** | Handle edge cases gracefully | Adversarial inputs, noisy data |
| **Drift testing** | Detect model degradation | Compare current vs baseline performance |
| **Explainability testing** | Verify reasoning can be explained | Feature importance makes sense |
| **A/B testing** | Compare model versions | New model vs production model |
| **Integration testing** | Model works in system | API response format, latency |

**c) (5 points)** What is the "oracle problem" in AI testing and how can it be addressed?

**Expected Answer:**

**Oracle problem:** No definitive way to know the "correct" output for many AI predictions.

**Example:**
```
Product recommendation system returns:
  User browsing hammers → Recommends: [nails, screwdriver, drill]

Is this "correct"?
  - Any of these could be reasonable
  - User might want something unrelated
  - "Correct" depends on user's actual intent
```

**Approaches to address:**

| Approach | Description | Limitation |
|----------|-------------|------------|
| **Human labeling** | Experts judge output quality | Expensive, subjective |
| **A/B testing** | Compare conversion/engagement | Delayed feedback, can't test edge cases |
| **Metamorphic testing** | If input changes X, output should change Y | Only catches certain bugs |
| **Statistical testing** | Output distribution matches expected | Doesn't catch individual errors |
| **Proxy metrics** | Use measurable outcomes (clicks, sales) | May not capture full quality |

---

## Question 2: Bias and Fairness Testing (20 points)

### Scenario
ToolShop's recommendation system shows different products to different users. Concerns have been raised that:
- Expensive products are shown more to users from wealthy zip codes
- Gender-stereotyped recommendations (tools for men, home decor for women)

### Questions

**a) (5 points)** What is algorithmic bias and why does it occur?

**Expected Answer:**

**Algorithmic bias:** Systematic and unfair discrimination in AI system outputs based on protected characteristics.

**Causes:**

| Cause | Description | Example |
|-------|-------------|---------|
| **Training data bias** | Historical data reflects past discrimination | Previous sales skewed by marketing targeting |
| **Sampling bias** | Non-representative training data | Over-represented demographic in training |
| **Measurement bias** | Proxy variables encode bias | Zip code correlates with race |
| **Aggregation bias** | One model for diverse populations | Model optimized for majority group |
| **Feedback loops** | Biased predictions reinforce bias | Show cheap products → users buy cheap → model learns to show cheap |

**Visualization of feedback loop:**
```
Biased initial state
        │
        ▼
┌───────────────────┐
│ Model recommends  │
│ based on bias     │───────┐
└───────────────────┘       │
        ▲                   │
        │                   ▼
        │           ┌───────────────┐
        │           │ Users interact│
        │           │ with biased   │
        │           │ recommendations│
        │           └───────┬───────┘
        │                   │
        │                   ▼
┌───────────────────┐       │
│ Model retrained   │◀──────┘
│ on biased data    │
└───────────────────┘
     (Bias amplified)
```

**b) (5 points)** What fairness metrics should be measured and how?

**Expected Answer:**

**Key fairness metrics:**

| Metric | Definition | Application |
|--------|------------|-------------|
| **Demographic parity** | Same positive outcome rate across groups | Equal recommendation of premium products |
| **Equal opportunity** | Same true positive rate across groups | Same discovery rate for relevant products |
| **Equalized odds** | Same TPR and FPR across groups | Same error rates for all demographics |
| **Individual fairness** | Similar individuals get similar treatment | Similar browsing patterns → similar recommendations |
| **Calibration** | Prediction confidence matches actual rates | 80% confidence = 80% correct across groups |

**Measurement approach:**
```
For each protected group:
1. Collect model predictions
2. Calculate metric per group
3. Compare across groups
4. Flag if difference > threshold

Example - Demographic Parity:
Group A (Male):    Premium recommendations: 30%
Group B (Female):  Premium recommendations: 18%
Disparity ratio: 18/30 = 0.6
Threshold: > 0.8 required
Status: ❌ FAIL (gender bias detected)
```

**c) (5 points)** How would you design tests to detect gender bias in product recommendations?

**Expected Answer:**

**Test design approach:**

| Phase | Activity |
|-------|----------|
| **1. Define scope** | Which protected attributes to test (gender, age, location) |
| **2. Create test users** | Synthetic profiles differing only in protected attribute |
| **3. Control variables** | Same browsing history, purchase patterns |
| **4. Collect recommendations** | Get recommendations for each test user |
| **5. Analyze patterns** | Compare recommendation sets across groups |
| **6. Statistical testing** | Determine if differences are significant |

**Test user matrix:**
```
Profile Template: Interested in home improvement

Test User 1: Male, Age 35, Urban
Test User 2: Female, Age 35, Urban
Test User 3: Male, Age 35, Suburban
Test User 4: Female, Age 35, Suburban
...

All users have IDENTICAL:
- Browsing history (same products viewed)
- Purchase history (same purchases)
- Session behavior (same click patterns)

ONLY difference: Protected attribute being tested
```

**Analysis criteria:**
```
If Female users receive:
- Fewer power tool recommendations
- More decorative item recommendations
- Lower average price recommendations
- Different product categories

Then bias is indicated → Investigate model features
```

**d) (5 points)** What remediation strategies exist for biased AI systems?

**Expected Answer:**

| Stage | Strategy | Description |
|-------|----------|-------------|
| **Pre-processing** | Data balancing | Ensure training data represents all groups |
| **Pre-processing** | Feature removal | Remove or transform biased features |
| **In-processing** | Fairness constraints | Add fairness terms to optimization |
| **In-processing** | Adversarial debiasing | Train to maximize accuracy while minimizing group predictability |
| **Post-processing** | Threshold adjustment | Different decision thresholds per group |
| **Post-processing** | Output calibration | Adjust outputs to ensure fairness |

**Trade-off consideration:**
```
Accuracy vs Fairness Trade-off:

           │ Higher Accuracy
           │       ▲
           │      /│\
           │     / │ \
           │    /  │  \
           │   /   │   \
           │  /    │    \
           │ /     │     \
           │/      │      \
           └───────────────────▶
             Higher Fairness

Decision: Business/ethical judgment on acceptable balance
```

---

## Question 3: Model Performance Testing (15 points)

### Scenario
ToolShop's recommendation model needs to be evaluated before deployment. The model predicts whether a user will purchase a recommended product.

### Questions

**a) (5 points)** Explain precision, recall, and F1 score. When would you prioritize each?

**Expected Answer:**

**Definitions:**

| Metric | Formula | Meaning |
|--------|---------|---------|
| **Precision** | TP / (TP + FP) | Of items recommended, how many were good? |
| **Recall** | TP / (TP + FN) | Of good items, how many were recommended? |
| **F1 Score** | 2 × (P × R) / (P + R) | Harmonic mean of precision and recall |

**Confusion matrix context:**
```
                    Predicted
                  Yes      No
Actual    Yes    TP       FN
          No     FP       TN

Precision = TP / (TP + FP)  → "Don't recommend junk"
Recall    = TP / (TP + FN)  → "Don't miss good items"
```

**When to prioritize:**

| Metric | Prioritize When | Example |
|--------|-----------------|---------|
| **Precision** | False positives are costly | Spam filter (don't mark legit email as spam) |
| **Recall** | False negatives are costly | Fraud detection (don't miss fraud) |
| **F1** | Need balance | General recommendation systems |

**b) (5 points)** What is a confusion matrix and how do you interpret it?

**Expected Answer:**

**Confusion matrix breakdown:**
```
                          Predicted Class
                    Positive        Negative
                ┌─────────────┬─────────────┐
    Positive    │     TP      │     FN      │
Actual          │ (Hit)       │ (Miss)      │
Class           ├─────────────┼─────────────┤
    Negative    │     FP      │     TN      │
                │ (False alarm)│ (Correct   │
                │             │  rejection) │
                └─────────────┴─────────────┘
```

**Interpretation guide:**

| Cell | Meaning | Impact |
|------|---------|--------|
| **True Positive (TP)** | Correctly recommended relevant product | Good! |
| **True Negative (TN)** | Correctly didn't recommend irrelevant | Good! |
| **False Positive (FP)** | Recommended irrelevant product | Annoys user, wastes opportunity |
| **False Negative (FN)** | Failed to recommend relevant product | Missed sale opportunity |

**Example for ToolShop:**
```
Model tested on 1000 user-product pairs:

                    Recommended
                  Yes      No
Actual    Yes    150      50     (200 relevant)
Interest  No     100     700     (800 irrelevant)

Precision = 150/(150+100) = 60%  (40% of recommendations were bad)
Recall    = 150/(150+50) = 75%   (25% of good products missed)
Accuracy  = (150+700)/1000 = 85% (overall correct)
```

**c) (5 points)** How do you evaluate a recommendation system specifically?

**Expected Answer:**

**Recommendation-specific metrics:**

| Metric | Description | Formula/Method |
|--------|-------------|----------------|
| **Hit Rate** | Proportion of users with at least one good rec | Users with hit / Total users |
| **Mean Reciprocal Rank (MRR)** | Position of first relevant item | 1/rank averaged across users |
| **NDCG** | Quality considering position | Weighted by position in list |
| **Coverage** | Variety of items recommended | Unique items / Total catalog |
| **Diversity** | Dissimilarity within recommendations | Intra-list distance |
| **Novelty** | Recommending non-obvious items | Inverse popularity weighting |

**Offline vs Online evaluation:**

| Type | Method | Measures |
|------|--------|----------|
| **Offline** | Historical data holdout | Prediction accuracy |
| **Online A/B** | Real user traffic | Business outcomes (clicks, purchases) |
| **User studies** | Qualitative feedback | Satisfaction, trust |

**ToolShop example:**
```
Recommendation quality metrics:

Top-5 Hit Rate: 72% (72% of users have at least 1 relevant item in top 5)
MRR: 0.45 (first relevant item typically at position 2-3)
Catalog Coverage: 35% (only 35% of products ever recommended)
                  → May indicate popularity bias

Action: Increase coverage to surface long-tail products
```

---

## Question 4: Model Robustness Testing (15 points)

### Scenario
The recommendation model should handle unexpected inputs gracefully:
- New products with no history
- Users with minimal browsing history
- Unusual browsing patterns
- Adversarial attacks (manipulated input)

### Questions

**a) (5 points)** What is robustness testing for ML models?

**Expected Answer:**

**Robustness testing:** Verifying ML model performs acceptably under unexpected, noisy, or adversarial conditions.

**Types of robustness:**

| Type | Description | Test Approach |
|------|-------------|---------------|
| **Input robustness** | Handle unusual inputs | Edge cases, missing data, out-of-distribution |
| **Noise robustness** | Tolerate data quality issues | Add noise to inputs, test performance |
| **Adversarial robustness** | Resist manipulation | Craft inputs designed to fool model |
| **Distribution shift** | Handle changing data | Test on data from different time/population |

**b) (5 points)** How would you test the model with edge cases and unusual inputs?

**Expected Answer:**

**Edge case categories:**

| Category | Examples | Expected Behavior |
|----------|----------|-------------------|
| **Cold start - user** | New user, no history | Generic popular recommendations |
| **Cold start - item** | New product, no ratings | Use content-based features |
| **Sparse data** | User with 1-2 interactions | Don't overfit to limited data |
| **Heavy user** | Thousands of interactions | Scale gracefully |
| **Contradictory signals** | Viewed but never bought | Handle mixed signals |
| **Missing features** | Product without category | Fallback behavior |

**Test scenarios:**
```
Scenario: New user cold start
Given: User has just registered
When: Request recommendations
Then:
  - Should return recommendations (not empty)
  - Should be popular/trending items
  - Should not crash or error
  - Response time still acceptable

Scenario: Missing product data
Given: Product has no description or category
When: Product is candidate for recommendation
Then:
  - Should still evaluate product
  - Should use available features only
  - Should not cause errors
```

**c) (5 points)** What are adversarial attacks on ML systems and how do you test for them?

**Expected Answer:**

**Adversarial attacks on recommendation systems:**

| Attack | Method | Goal |
|--------|--------|------|
| **Shilling attack** | Create fake users/reviews | Boost or bury products |
| **Profile pollution** | Inject fake interactions | Manipulate recommendations |
| **Data poisoning** | Corrupt training data | Degrade model performance |
| **Model extraction** | Query model repeatedly | Steal proprietary model |

**Testing for adversarial robustness:**
```
Shilling Attack Test:
1. Establish baseline recommendations
2. Inject fake users who all like product X
3. Re-request recommendations
4. Measure if product X ranking increased unfairly

Expected: Robust model should:
- Detect anomalous patterns
- Limit impact of suspicious users
- Not dramatically shift from baseline
```

**Defense testing:**

| Defense | Test Method |
|---------|-------------|
| **Rate limiting** | Verify max queries per user enforced |
| **Anomaly detection** | Inject suspicious patterns, verify flagged |
| **Input validation** | Send malformed inputs, verify rejection |
| **Freshness controls** | Verify old/manipulated data discounted |

---

## Question 5: Model Monitoring and Drift (15 points)

### Scenario
After deploying the recommendation model, ToolShop needs to monitor its ongoing performance.

### Questions

**a) (5 points)** What is model drift and what types exist?

**Expected Answer:**

**Model drift:** Degradation of model performance over time due to changes in data or environment.

**Types of drift:**

| Type | Description | Example |
|------|-------------|---------|
| **Data drift** | Input distribution changes | Seasonal shopping patterns change |
| **Concept drift** | Relationship between input and output changes | Product preferences change after pandemic |
| **Feature drift** | Meaning of features changes | "Popular" category redefined |
| **Label drift** | Target variable distribution changes | More impulse purchases than before |

**Visualization:**
```
Model trained:    Jan 2024
Deployed:         Feb 2024
Drift detected:   May 2024

       Accuracy
100% ─┬────────────────────────────────
      │▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░░░░░░░
 90% ─┤            ▓▓▓▓▓▓▓▓░░░░░░░░
      │                    ▓▓▓▓▓▓▓▓
 80% ─┤                            ▓▓▓
      │                               ▓ ← Drift!
      └──────────────────────────────────▶
        Feb    Mar    Apr    May    Jun
```

**b) (5 points)** What metrics should be monitored for a deployed recommendation model?

**Expected Answer:**

**Monitoring categories:**

| Category | Metrics | Alert Threshold |
|----------|---------|-----------------|
| **Performance** | Latency, throughput, errors | P99 > 200ms, error rate > 1% |
| **Quality** | CTR, conversion rate, engagement | >10% drop from baseline |
| **Fairness** | Disparity across groups | Ratio < 0.8 |
| **Data** | Input distribution, null rates | Significant distribution shift |
| **Model** | Prediction distribution, confidence | Average confidence dropping |

**Dashboard design:**
```
┌─────────────────────────────────────────────────────────┐
│ Model Health Dashboard                                   │
├────────────────────┬────────────────────────────────────┤
│ Prediction Latency │ [Chart: p50=45ms, p99=120ms]  ✅   │
├────────────────────┼────────────────────────────────────┤
│ Click-Through Rate │ [Chart: 3.2% ↓ 0.1%]          ✅   │
├────────────────────┼────────────────────────────────────┤
│ Conversion Rate    │ [Chart: 1.1% ↓ 0.3%]          ⚠️   │
├────────────────────┼────────────────────────────────────┤
│ Feature Drift      │ [Alert: browsing_pattern]    ❌   │
├────────────────────┼────────────────────────────────────┤
│ Fairness Score     │ [Chart: 0.92]                 ✅   │
└────────────────────┴────────────────────────────────────┘
```

**c) (5 points)** How do you detect and respond to model drift?

**Expected Answer:**

**Drift detection methods:**

| Method | Approach | Best For |
|--------|----------|----------|
| **Statistical tests** | KS test, chi-square on input distributions | Data drift |
| **Performance monitoring** | Track accuracy/quality metrics | Concept drift |
| **Prediction distribution** | Monitor output distribution changes | Both |
| **Page-Hinkley test** | Sequential change detection | Gradual drift |
| **Control charts** | Statistical process control | Continuous monitoring |

**Response strategy:**
```
Drift Detected
      │
      ▼
┌─────────────────┐
│  Assess Impact  │
│  Is performance │
│  significantly  │
│  degraded?      │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
   No        Yes
    │         │
    ▼         ▼
 Monitor   Immediate
 closely   action needed
              │
    ┌─────────┴─────────┐
    ▼                   ▼
 Retrain          Rollback to
 model            previous model
    │
    ▼
 Deploy updated model
 with validation
```

**Prevention and mitigation:**
- Regular scheduled retraining (weekly/monthly)
- Online learning for gradual adaptation
- Ensemble with freshness weighting
- Shadow mode testing before full deployment

---

## Grading Summary

| Question | Topic | Points |
|----------|-------|--------|
| Q1 | AI Testing Challenges | 15 |
| Q2 | Bias and Fairness Testing | 20 |
| Q3 | Model Performance Testing | 15 |
| Q4 | Robustness Testing | 15 |
| Q5 | Monitoring and Drift | 15 |
| **Total** | | **80** |
