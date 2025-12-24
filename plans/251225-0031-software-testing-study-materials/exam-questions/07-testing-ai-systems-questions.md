# Topic 7: Testing AI Systems - Application Questions
**CS423 Software Testing | Final Exam Practice**

---

## Hướng dẫn
- 10 câu hỏi x 10 điểm = 100 điểm
- Mỗi câu hỏi ở mức **vận dụng** (application level)
- Tập trung vào Testing FOR AI - kiểm thử hệ thống AI/ML

---

## Câu 1: Bias Testing Strategy (10 điểm)

### Tình huống
Công ty FinTech phát triển AI model để đánh giá credit score. Audit phát hiện:
- Male applicants: 85% approval rate
- Female applicants: 65% approval rate

Cần design và implement bias testing strategy.

### Yêu cầu
a) Identify bias types và potential sources (3 điểm)
b) Implement fairness testing với AIF360 (4 điểm)
c) Propose mitigation strategies với tradeoffs (3 điểm)

### Đáp án mẫu

**a) Bias Types & Sources (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BIAS ANALYSIS FOR CREDIT SCORING MODEL                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  OBSERVED DISPARITY                                                          │
│  ──────────────────                                                          │
│                                                                              │
│  Approval Rates:                                                             │
│  ┌──────────┬────────────┬───────────┬────────────────────────────────────┐ │
│  │ Group    │ Rate       │ Ratio     │ Assessment                          │ │
│  ├──────────┼────────────┼───────────┼────────────────────────────────────┤ │
│  │ Male     │ 85%        │ 1.0       │ Privileged group                   │ │
│  │ Female   │ 65%        │ 0.76      │ Unprivileged group                 │ │
│  └──────────┴────────────┴───────────┴────────────────────────────────────┘ │
│                                                                              │
│  Disparate Impact Ratio: 65/85 = 0.76 (< 0.80 threshold = VIOLATION)       │
│                                                                              │
│                                                                              │
│  POTENTIAL BIAS SOURCES                                                      │
│  ──────────────────────                                                      │
│                                                                              │
│  1. Historical Bias                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Root Cause: Training data reflects past discrimination               │    │
│  │ Evidence: Historical lending data from 1990-2020                     │    │
│  │ Impact: Model learns to replicate historical approval patterns       │    │
│  │                                                                      │    │
│  │ Example:                                                             │    │
│  │ - Women historically had lower income → lower approval              │    │
│  │ - Model learns income-approval correlation                          │    │
│  │ - Perpetuates historical gender wage gap                            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  2. Proxy Variables                                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Root Cause: Features that correlate with protected attribute        │    │
│  │                                                                      │    │
│  │ Suspicious features:                                                 │    │
│  │ - Job title: "Nurse" (90% female), "Engineer" (80% male)           │    │
│  │ - Industry: Healthcare vs Tech                                       │    │
│  │ - First name (encoding removes, but tokenization may leak)          │    │
│  │ - ZIP code (residential segregation patterns)                       │    │
│  │                                                                      │    │
│  │ Test: Remove feature, check if disparity decreases                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  3. Selection Bias                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Root Cause: Non-representative training data                         │    │
│  │                                                                      │    │
│  │ Check: Compare training data demographics vs population             │    │
│  │                                                                      │    │
│  │ ┌────────────────┬───────────────┬───────────────┐                  │    │
│  │ │ Demographic    │ Training Data │ Population    │                  │    │
│  │ ├────────────────┼───────────────┼───────────────┤                  │    │
│  │ │ Male           │ 70%           │ 48%           │ Over-represented │    │
│  │ │ Female         │ 30%           │ 52%           │ Under-represented│    │
│  │ └────────────────┴───────────────┴───────────────┘                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  4. Measurement Bias                                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ Root Cause: Different data quality across groups                    │    │
│  │                                                                      │    │
│  │ Example: Credit history completeness                                │    │
│  │ - Women more likely to have thin credit files                       │    │
│  │ - Missing data treated as negative signal                           │    │
│  │ - Disproportionate impact on women                                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**b) Fairness Testing with AIF360 (4 điểm)**

```python
"""
Fairness Testing Implementation for Credit Scoring Model
Using IBM AI Fairness 360 (AIF360)
"""

import pandas as pd
import numpy as np
from aif360.datasets import BinaryLabelDataset
from aif360.metrics import BinaryLabelDatasetMetric, ClassificationMetric
from sklearn.model_selection import train_test_split

# ============ STEP 1: PREPARE DATA ============

# Load credit scoring data
df = pd.read_csv('credit_applications.csv')

# Define protected attribute
# gender: 1 = Male (privileged), 0 = Female (unprivileged)
protected_attribute = 'gender'
label = 'approved'

# Create AIF360 dataset
dataset = BinaryLabelDataset(
    df=df,
    label_names=[label],
    protected_attribute_names=[protected_attribute],
    favorable_label=1,
    unfavorable_label=0
)

# Define groups
privileged_groups = [{'gender': 1}]   # Male
unprivileged_groups = [{'gender': 0}]  # Female


# ============ STEP 2: CALCULATE BASELINE METRICS ============

def analyze_dataset_fairness(dataset, privileged_groups, unprivileged_groups):
    """Calculate dataset-level fairness metrics"""
    metric = BinaryLabelDatasetMetric(
        dataset,
        unprivileged_groups=unprivileged_groups,
        privileged_groups=privileged_groups
    )

    results = {
        'base_rate': metric.base_rate(),
        'disparate_impact': metric.disparate_impact(),
        'statistical_parity_difference': metric.statistical_parity_difference(),
        'consistency': metric.consistency()[0]
    }

    print("=" * 50)
    print("DATASET FAIRNESS METRICS")
    print("=" * 50)
    print(f"Base Rate (overall approval): {results['base_rate']:.2%}")
    print(f"Disparate Impact: {results['disparate_impact']:.3f}")
    print(f"  → Goal: 0.80 - 1.25 (4/5 rule)")
    print(f"  → Status: {'✓ PASS' if 0.80 <= results['disparate_impact'] <= 1.25 else '✗ FAIL'}")
    print(f"Statistical Parity Difference: {results['statistical_parity_difference']:.3f}")
    print(f"  → Goal: Close to 0")
    print(f"Consistency: {results['consistency']:.3f}")

    return results

baseline_metrics = analyze_dataset_fairness(dataset, privileged_groups, unprivileged_groups)


# ============ STEP 3: TRAIN MODEL AND EVALUATE ============

from sklearn.ensemble import RandomForestClassifier

def evaluate_model_fairness(model, test_dataset, privileged_groups, unprivileged_groups):
    """Calculate classification fairness metrics"""

    # Get predictions
    X_test = test_dataset.features
    y_pred = model.predict(X_test)

    # Create predictions dataset
    pred_dataset = test_dataset.copy()
    pred_dataset.labels = y_pred.reshape(-1, 1)

    # Calculate metrics
    metric = ClassificationMetric(
        test_dataset,
        pred_dataset,
        unprivileged_groups=unprivileged_groups,
        privileged_groups=privileged_groups
    )

    results = {
        'disparate_impact': metric.disparate_impact(),
        'statistical_parity_difference': metric.statistical_parity_difference(),
        'equal_opportunity_difference': metric.equal_opportunity_difference(),
        'average_odds_difference': metric.average_odds_difference(),
        'theil_index': metric.theil_index(),

        # Per-group metrics
        'TPR_privileged': metric.true_positive_rate(privileged=True),
        'TPR_unprivileged': metric.true_positive_rate(privileged=False),
        'FPR_privileged': metric.false_positive_rate(privileged=True),
        'FPR_unprivileged': metric.false_positive_rate(privileged=False),
    }

    print("\n" + "=" * 50)
    print("MODEL FAIRNESS METRICS")
    print("=" * 50)

    print(f"\n1. Disparate Impact: {results['disparate_impact']:.3f}")
    print(f"   Interpretation: Ratio of approval rates")
    print(f"   Status: {'✓ PASS' if 0.80 <= results['disparate_impact'] <= 1.25 else '✗ FAIL'}")

    print(f"\n2. Statistical Parity Difference: {results['statistical_parity_difference']:.3f}")
    print(f"   Interpretation: Difference in approval rates")
    print(f"   Goal: Close to 0 (|diff| < 0.1)")

    print(f"\n3. Equal Opportunity Difference: {results['equal_opportunity_difference']:.3f}")
    print(f"   Interpretation: TPR(unprivileged) - TPR(privileged)")
    print(f"   TPR Privileged: {results['TPR_privileged']:.2%}")
    print(f"   TPR Unprivileged: {results['TPR_unprivileged']:.2%}")
    print(f"   Goal: Close to 0")

    print(f"\n4. Average Odds Difference: {results['average_odds_difference']:.3f}")
    print(f"   Interpretation: Average of TPR and FPR differences")
    print(f"   Goal: Close to 0")

    print(f"\n5. Theil Index: {results['theil_index']:.3f}")
    print(f"   Interpretation: Inequality in benefit distribution")
    print(f"   Goal: Close to 0")

    return results

# Split and train
train_dataset, test_dataset = dataset.split([0.7])

X_train = train_dataset.features
y_train = train_dataset.labels.ravel()

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# Evaluate fairness
model_metrics = evaluate_model_fairness(
    model, test_dataset, privileged_groups, unprivileged_groups
)


# ============ STEP 4: FEATURE IMPORTANCE FOR BIAS ============

def analyze_feature_bias(model, feature_names, protected_attribute_idx):
    """Check if model relies heavily on proxy features"""
    importances = model.feature_importances_

    # Identify potential proxy features
    # (features that correlate with protected attribute)
    print("\n" + "=" * 50)
    print("FEATURE IMPORTANCE ANALYSIS")
    print("=" * 50)

    sorted_idx = np.argsort(importances)[::-1]
    for i in sorted_idx[:10]:
        print(f"{feature_names[i]}: {importances[i]:.4f}")

    # Flag if protected attribute has high importance
    protected_importance = importances[protected_attribute_idx]
    if protected_importance > 0.05:
        print(f"\n⚠️ WARNING: Protected attribute (gender) has {protected_importance:.2%} importance")
        print("   Consider removing or reviewing feature engineering")


# ============ STEP 5: AUTOMATED FAIRNESS TEST SUITE ============

import pytest

class TestModelFairness:
    """Automated fairness test suite for CI/CD integration"""

    @pytest.fixture
    def model_metrics(self, trained_model, test_data):
        return evaluate_model_fairness(
            trained_model,
            test_data,
            [{'gender': 1}],
            [{'gender': 0}]
        )

    def test_disparate_impact_threshold(self, model_metrics):
        """4/5 rule: Disparate impact must be >= 0.80"""
        assert model_metrics['disparate_impact'] >= 0.80, \
            f"Disparate impact {model_metrics['disparate_impact']:.3f} < 0.80"

    def test_equal_opportunity(self, model_metrics):
        """TPR difference should be minimal"""
        assert abs(model_metrics['equal_opportunity_difference']) < 0.10, \
            f"Equal opportunity difference {model_metrics['equal_opportunity_difference']:.3f} >= 0.10"

    def test_average_odds(self, model_metrics):
        """Average odds difference should be minimal"""
        assert abs(model_metrics['average_odds_difference']) < 0.10, \
            f"Average odds difference {model_metrics['average_odds_difference']:.3f} >= 0.10"

    def test_accuracy_parity(self, trained_model, test_data):
        """Accuracy should be similar across groups"""
        # Calculate accuracy per group
        male_data = test_data.subset([test_data.features[:, gender_idx] == 1])
        female_data = test_data.subset([test_data.features[:, gender_idx] == 0])

        male_acc = trained_model.score(male_data.features, male_data.labels.ravel())
        female_acc = trained_model.score(female_data.features, female_data.labels.ravel())

        assert abs(male_acc - female_acc) < 0.05, \
            f"Accuracy gap {abs(male_acc - female_acc):.2%} >= 5%"
```

**c) Mitigation Strategies with Tradeoffs (3 điểm)**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BIAS MITIGATION STRATEGIES                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STRATEGY 1: PRE-PROCESSING (Reweighing)                                     │
│  ───────────────────────────────────────                                     │
│                                                                              │
│  Approach: Assign weights to training samples                               │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ from aif360.algorithms.preprocessing import Reweighing               │    │
│  │                                                                      │    │
│  │ rw = Reweighing(                                                     │    │
│  │     unprivileged_groups=[{'gender': 0}],                            │    │
│  │     privileged_groups=[{'gender': 1}]                               │    │
│  │ )                                                                    │    │
│  │ balanced_dataset = rw.fit_transform(training_dataset)               │    │
│  │                                                                      │    │
│  │ # Train model on balanced data                                      │    │
│  │ model.fit(balanced_dataset.features,                                │    │
│  │           balanced_dataset.labels,                                  │    │
│  │           sample_weight=balanced_dataset.instance_weights)          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Tradeoffs:                                                                 │
│  ✓ Doesn't change model architecture                                       │
│  ✓ Interpretable (can explain weights)                                      │
│  ✗ May reduce overall accuracy (1-3% typical)                              │
│  ✗ Doesn't guarantee fairness at prediction time                           │
│                                                                              │
│                                                                              │
│  STRATEGY 2: IN-PROCESSING (Fairness Constraints)                           │
│  ─────────────────────────────────────────────────                           │
│                                                                              │
│  Approach: Add fairness constraint to loss function                         │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ from aif360.algorithms.inprocessing import PrejudiceRemover          │    │
│  │                                                                      │    │
│  │ # eta controls fairness-accuracy tradeoff                           │    │
│  │ # Higher eta = more fairness, less accuracy                         │    │
│  │ model = PrejudiceRemover(                                            │    │
│  │     sensitive_attr='gender',                                         │    │
│  │     eta=1.0  # Start with 1.0, tune based on results                │    │
│  │ )                                                                    │    │
│  │ model.fit(training_dataset)                                          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Tradeoffs:                                                                 │
│  ✓ Optimizes fairness during training                                       │
│  ✓ Single model (no post-processing needed)                                │
│  ✗ Requires custom model implementation                                     │
│  ✗ Accuracy-fairness tradeoff more severe (3-8% accuracy drop)            │
│  ✗ Hyperparameter tuning complex                                            │
│                                                                              │
│                                                                              │
│  STRATEGY 3: POST-PROCESSING (Threshold Calibration)                        │
│  ────────────────────────────────────────────────────                        │
│                                                                              │
│  Approach: Different decision thresholds per group                          │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ from aif360.algorithms.postprocessing import CalibratedEqOddsPostprocessing│
│  │                                                                      │    │
│  │ calibrator = CalibratedEqOddsPostprocessing(                         │    │
│  │     unprivileged_groups=[{'gender': 0}],                            │    │
│  │     privileged_groups=[{'gender': 1}],                              │    │
│  │     cost_constraint='fnr'  # or 'fpr', 'weighted'                   │    │
│  │ )                                                                    │    │
│  │                                                                      │    │
│  │ calibrator.fit(test_dataset, predictions_dataset)                   │    │
│  │ fair_predictions = calibrator.predict(test_dataset)                 │    │
│  │                                                                      │    │
│  │ # Results:                                                           │    │
│  │ # Male threshold: 0.50 (standard)                                   │    │
│  │ # Female threshold: 0.42 (adjusted for equal opportunity)          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  Tradeoffs:                                                                 │
│  ✓ No retraining needed                                                     │
│  ✓ Guarantees fairness at prediction time                                   │
│  ✓ Minimal accuracy impact (1-2%)                                           │
│  ✗ May face legal challenges (different treatment)                         │
│  ✗ Harder to explain to stakeholders                                        │
│  ✗ Requires storing group membership                                        │
│                                                                              │
│                                                                              │
│  RECOMMENDED APPROACH                                                        │
│  ────────────────────                                                        │
│                                                                              │
│  Hybrid Strategy:                                                           │
│  1. Pre-processing: Reweighing (address training data imbalance)           │
│  2. In-processing: Add fairness regularization (eta=0.5, moderate)         │
│  3. Post-processing: Calibration as safety net                              │
│                                                                              │
│  Expected Results:                                                          │
│  ┌────────────────────┬───────────┬───────────┬───────────┐                │
│  │ Metric             │ Before    │ After     │ Change    │                │
│  ├────────────────────┼───────────┼───────────┼───────────┤                │
│  │ Disparate Impact   │ 0.76      │ 0.85      │ +0.09 ✓   │                │
│  │ Overall Accuracy   │ 87%       │ 84%       │ -3%       │                │
│  │ Male Approval      │ 85%       │ 82%       │ -3%       │                │
│  │ Female Approval    │ 65%       │ 70%       │ +5% ✓     │                │
│  └────────────────────┴───────────┴───────────┴───────────┘                │
│                                                                              │
│  Legal Considerations:                                                       │
│  - Document all fairness testing and mitigation steps                       │
│  - Consult legal on differential thresholding                               │
│  - Regular audits (quarterly) for drift                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Câu 2: Test Oracle Problem for ML (10 điểm)

### Tình huống
Image classification model cho product defect detection. Challenge: Không có single "correct" answer.
- Model confidence: 92% defective
- Human inspectors disagree 15% of the time

### Yêu cầu
a) Design multi-layer test oracle (4 điểm)
b) Implement metamorphic testing (3 điểm)
c) Design differential testing approach (3 điểm)

### Đáp án mẫu

**a) Multi-layer Test Oracle (4 điểm)**

```python
"""
Multi-Layer Test Oracle for Defect Detection Model
"""

from dataclasses import dataclass
from typing import Tuple, Optional
import numpy as np

@dataclass
class OracleResult:
    verdict: str  # PASS, FAIL, REVIEW
    confidence: float
    reason: str
    layer: str

class DefectDetectionOracle:
    """
    Multi-layer oracle combining statistical, metamorphic,
    and reference-based approaches
    """

    def __init__(self, model, reference_set, confidence_threshold=0.85):
        self.model = model
        self.reference_set = reference_set  # Expert-labeled images
        self.confidence_threshold = confidence_threshold

    def evaluate(self, image, image_id=None) -> OracleResult:
        """
        Main oracle function - applies layers sequentially
        """
        prediction = self.model.predict(image)

        # Layer 1: Confidence-based oracle
        result = self._confidence_oracle(prediction)
        if result.verdict != "CONTINUE":
            return result

        # Layer 2: Metamorphic testing oracle
        result = self._metamorphic_oracle(image, prediction)
        if result.verdict != "CONTINUE":
            return result

        # Layer 3: Reference set oracle (if applicable)
        if image_id and image_id in self.reference_set:
            result = self._reference_oracle(image_id, prediction)
            return result

        # Layer 4: Ensemble oracle
        result = self._ensemble_oracle(image, prediction)
        return result

    def _confidence_oracle(self, prediction) -> OracleResult:
        """
        Layer 1: Statistical Oracle based on prediction confidence

        Rules:
        - confidence >= 0.95: High confidence → PASS
        - confidence <= 0.50: Very low confidence → FAIL
        - 0.50 < confidence < 0.85: Uncertain → REVIEW
        - 0.85 <= confidence < 0.95: Good → CONTINUE to next layer
        """
        confidence = prediction.confidence

        if confidence >= 0.95:
            return OracleResult(
                verdict="PASS",
                confidence=confidence,
                reason="High confidence prediction",
                layer="confidence"
            )
        elif confidence <= 0.50:
            return OracleResult(
                verdict="FAIL",
                confidence=confidence,
                reason="Very low confidence - model uncertain",
                layer="confidence"
            )
        elif confidence < 0.85:
            return OracleResult(
                verdict="REVIEW",
                confidence=confidence,
                reason=f"Medium confidence ({confidence:.2%}) - human review needed",
                layer="confidence"
            )
        else:
            return OracleResult(
                verdict="CONTINUE",
                confidence=confidence,
                reason="Proceed to metamorphic testing",
                layer="confidence"
            )

    def _metamorphic_oracle(self, image, original_prediction) -> OracleResult:
        """
        Layer 2: Metamorphic Testing Oracle

        Metamorphic Relations:
        1. Rotation invariance: Defect class same for rotated image
        2. Brightness invariance: Slight brightness change shouldn't affect class
        3. Flip invariance: Horizontal flip shouldn't change defect detection
        """
        transformations = [
            ("rotation_90", self._rotate(image, 90)),
            ("rotation_180", self._rotate(image, 180)),
            ("brightness_up", self._adjust_brightness(image, 1.1)),
            ("brightness_down", self._adjust_brightness(image, 0.9)),
            ("horizontal_flip", self._flip(image)),
        ]

        violations = []
        for name, transformed_image in transformations:
            transformed_pred = self.model.predict(transformed_image)

            # Check if class changed
            if transformed_pred.label != original_prediction.label:
                violations.append({
                    "transformation": name,
                    "original_label": original_prediction.label,
                    "transformed_label": transformed_pred.label,
                    "original_confidence": original_prediction.confidence,
                    "transformed_confidence": transformed_pred.confidence
                })

        if len(violations) >= 3:
            return OracleResult(
                verdict="FAIL",
                confidence=0.0,
                reason=f"Failed {len(violations)}/5 metamorphic tests: {violations}",
                layer="metamorphic"
            )
        elif len(violations) >= 1:
            return OracleResult(
                verdict="REVIEW",
                confidence=original_prediction.confidence,
                reason=f"Failed {len(violations)}/5 metamorphic tests",
                layer="metamorphic"
            )
        else:
            return OracleResult(
                verdict="CONTINUE",
                confidence=original_prediction.confidence,
                reason="Passed all metamorphic tests",
                layer="metamorphic"
            )

    def _reference_oracle(self, image_id, prediction) -> OracleResult:
        """
        Layer 3: Reference Set Oracle

        Compare against expert-labeled reference images
        """
        expected_label = self.reference_set[image_id]["label"]
        expected_confidence = self.reference_set[image_id].get("min_confidence", 0.80)

        if prediction.label != expected_label:
            return OracleResult(
                verdict="FAIL",
                confidence=0.0,
                reason=f"Mismatch with reference: expected {expected_label}, got {prediction.label}",
                layer="reference"
            )
        elif prediction.confidence < expected_confidence:
            return OracleResult(
                verdict="REVIEW",
                confidence=prediction.confidence,
                reason=f"Below reference confidence: {prediction.confidence:.2%} < {expected_confidence:.2%}",
                layer="reference"
            )
        else:
            return OracleResult(
                verdict="PASS",
                confidence=prediction.confidence,
                reason="Matches reference set",
                layer="reference"
            )

    def _ensemble_oracle(self, image, primary_prediction) -> OracleResult:
        """
        Layer 4: Ensemble/Differential Oracle

        Use multiple models and check agreement
        """
        # Load ensemble models (different architectures)
        models = [
            self.model,  # Primary (e.g., ResNet)
            self._load_model("efficientnet"),
            self._load_model("vit"),
        ]

        predictions = [m.predict(image) for m in models]
        labels = [p.label for p in predictions]

        # Majority voting
        from collections import Counter
        vote_counts = Counter(labels)
        majority_label, majority_count = vote_counts.most_common(1)[0]

        agreement = majority_count / len(models)

        if agreement >= 0.67:  # 2/3 agree
            if primary_prediction.label == majority_label:
                return OracleResult(
                    verdict="PASS",
                    confidence=agreement,
                    reason=f"Ensemble agreement: {agreement:.0%}",
                    layer="ensemble"
                )
            else:
                return OracleResult(
                    verdict="FAIL",
                    confidence=agreement,
                    reason=f"Primary model disagrees with ensemble majority",
                    layer="ensemble"
                )
        else:
            return OracleResult(
                verdict="REVIEW",
                confidence=agreement,
                reason=f"Low ensemble agreement: {agreement:.0%}",
                layer="ensemble"
            )

    # Helper methods for transformations
    @staticmethod
    def _rotate(image, degrees):
        from PIL import Image
        return image.rotate(degrees)

    @staticmethod
    def _adjust_brightness(image, factor):
        from PIL import ImageEnhance
        enhancer = ImageEnhance.Brightness(image)
        return enhancer.enhance(factor)

    @staticmethod
    def _flip(image):
        from PIL import Image
        return image.transpose(Image.FLIP_LEFT_RIGHT)

    def _load_model(self, model_name):
        # Load alternative model for ensemble
        pass
```

**b) Metamorphic Testing Implementation (3 điểm)**

```python
"""
Metamorphic Testing for Image Classification
"""

import pytest
import numpy as np
from PIL import Image
import cv2

class MetamorphicRelations:
    """
    Metamorphic Relations for Defect Detection

    Key Insight: We don't know the "correct" output, but we know
    relationships between inputs and outputs that should hold.
    """

    @staticmethod
    def rotation_invariance(model, image, max_degree_change=180):
        """
        MR1: Rotation Invariance

        For defect detection, rotation shouldn't change classification
        (defect is defect regardless of orientation)
        """
        original_pred = model.predict(image)

        for degree in [90, 180, 270]:
            rotated = image.rotate(degree)
            rotated_pred = model.predict(rotated)

            assert rotated_pred.label == original_pred.label, \
                f"Rotation {degree}° changed prediction: {original_pred.label} → {rotated_pred.label}"

    @staticmethod
    def scale_invariance(model, image, scale_range=(0.8, 1.2)):
        """
        MR2: Scale Invariance (within reasonable bounds)

        Slight zoom shouldn't change classification
        """
        original_pred = model.predict(image)
        h, w = image.size

        for scale in [0.9, 1.1]:
            new_h, new_w = int(h * scale), int(w * scale)
            scaled = image.resize((new_w, new_h))
            # Center crop back to original size
            cropped = scaled.crop((
                (new_w - w) // 2,
                (new_h - h) // 2,
                (new_w + w) // 2,
                (new_h + h) // 2
            ))
            scaled_pred = model.predict(cropped)

            assert scaled_pred.label == original_pred.label, \
                f"Scale {scale}x changed prediction"

    @staticmethod
    def brightness_invariance(model, image, adjustment_range=0.1):
        """
        MR3: Brightness Invariance

        Minor brightness changes shouldn't affect classification
        """
        from PIL import ImageEnhance
        original_pred = model.predict(image)

        for factor in [0.9, 1.1]:
            enhancer = ImageEnhance.Brightness(image)
            adjusted = enhancer.enhance(factor)
            adjusted_pred = model.predict(adjusted)

            assert adjusted_pred.label == original_pred.label, \
                f"Brightness {factor}x changed prediction"

    @staticmethod
    def additive_invariance(model, good_image, defect_overlay):
        """
        MR4: Additive Relation

        Adding defect to good product should change classification
        """
        good_pred = model.predict(good_image)
        assert good_pred.label == "good", "Baseline should be classified as good"

        # Overlay defect pattern
        combined = Image.blend(good_image, defect_overlay, alpha=0.5)
        combined_pred = model.predict(combined)

        assert combined_pred.label == "defective", \
            "Adding defect should change classification to defective"

    @staticmethod
    def monotonic_relation(model, images_by_severity):
        """
        MR5: Monotonic Confidence

        Defect confidence should increase with defect severity
        """
        confidences = []
        for severity, image in sorted(images_by_severity.items()):
            pred = model.predict(image)
            if pred.label == "defective":
                confidences.append((severity, pred.confidence))

        # Check monotonicity
        for i in range(1, len(confidences)):
            prev_severity, prev_conf = confidences[i-1]
            curr_severity, curr_conf = confidences[i]

            assert curr_conf >= prev_conf - 0.05, \
                f"Confidence should increase with severity: " \
                f"severity {prev_severity}={prev_conf:.2%} vs {curr_severity}={curr_conf:.2%}"


# ============ PYTEST INTEGRATION ============

@pytest.fixture
def defect_model():
    from model import load_defect_model
    return load_defect_model()

@pytest.fixture
def test_images():
    return {
        "good": Image.open("test_data/good_product.jpg"),
        "minor_defect": Image.open("test_data/minor_scratch.jpg"),
        "major_defect": Image.open("test_data/major_crack.jpg"),
    }

class TestMetamorphicRelations:
    """Metamorphic test suite for CI/CD"""

    def test_rotation_invariance(self, defect_model, test_images):
        for name, image in test_images.items():
            MetamorphicRelations.rotation_invariance(defect_model, image)

    def test_brightness_invariance(self, defect_model, test_images):
        for name, image in test_images.items():
            MetamorphicRelations.brightness_invariance(defect_model, image)

    def test_scale_invariance(self, defect_model, test_images):
        for name, image in test_images.items():
            MetamorphicRelations.scale_invariance(defect_model, image)

    def test_confidence_monotonicity(self, defect_model):
        images_by_severity = {
            0: Image.open("test_data/severity_0.jpg"),
            1: Image.open("test_data/severity_1.jpg"),
            2: Image.open("test_data/severity_2.jpg"),
            3: Image.open("test_data/severity_3.jpg"),
        }
        MetamorphicRelations.monotonic_relation(defect_model, images_by_severity)
```

**c) Differential Testing (3 điểm)**

```python
"""
Differential Testing for ML Models
Compare multiple implementations to find bugs
"""

from typing import List, Dict, Any
import numpy as np

class DifferentialTester:
    """
    Compare predictions across multiple models/versions
    to identify potential bugs or inconsistencies
    """

    def __init__(self, models: List[Any], model_names: List[str]):
        assert len(models) == len(model_names)
        self.models = models
        self.model_names = model_names

    def compare_predictions(self, image) -> Dict:
        """Compare all models on single input"""
        predictions = {}
        for model, name in zip(self.models, self.model_names):
            pred = model.predict(image)
            predictions[name] = {
                "label": pred.label,
                "confidence": pred.confidence
            }

        # Calculate agreement
        labels = [p["label"] for p in predictions.values()]
        unique_labels = set(labels)

        return {
            "predictions": predictions,
            "agreement": len(labels) - len(unique_labels) + 1,
            "unanimous": len(unique_labels) == 1,
            "disagreement_models": self._find_disagreement(predictions)
        }

    def _find_disagreement(self, predictions: Dict) -> List[str]:
        """Find which models disagree"""
        labels = [(name, pred["label"]) for name, pred in predictions.items()]
        from collections import Counter
        label_counts = Counter([l for _, l in labels])

        # Find minority predictions
        majority_label = label_counts.most_common(1)[0][0]
        return [name for name, label in labels if label != majority_label]

    def run_test_suite(self, test_images: List) -> Dict:
        """Run differential tests on multiple images"""
        results = {
            "total": len(test_images),
            "unanimous": 0,
            "disagreement": 0,
            "disagreement_cases": []
        }

        for idx, image in enumerate(test_images):
            comparison = self.compare_predictions(image)

            if comparison["unanimous"]:
                results["unanimous"] += 1
            else:
                results["disagreement"] += 1
                results["disagreement_cases"].append({
                    "image_idx": idx,
                    "predictions": comparison["predictions"],
                    "disagreeing_models": comparison["disagreement_models"]
                })

        results["agreement_rate"] = results["unanimous"] / results["total"]
        return results


# ============ VERSION COMPARISON ============

class VersionDifferentialTester:
    """
    Compare model versions to detect regression
    """

    def __init__(self, current_model, baseline_model):
        self.current = current_model
        self.baseline = baseline_model

    def find_regressions(self, test_set, labels):
        """Find cases where current model is worse than baseline"""
        regressions = []

        for idx, (image, true_label) in enumerate(zip(test_set, labels)):
            current_pred = self.current.predict(image)
            baseline_pred = self.baseline.predict(image)

            current_correct = current_pred.label == true_label
            baseline_correct = baseline_pred.label == true_label

            if baseline_correct and not current_correct:
                regressions.append({
                    "image_idx": idx,
                    "true_label": true_label,
                    "baseline_prediction": baseline_pred.label,
                    "current_prediction": current_pred.label,
                    "baseline_confidence": baseline_pred.confidence,
                    "current_confidence": current_pred.confidence
                })

        return {
            "total_regressions": len(regressions),
            "regression_rate": len(regressions) / len(test_set),
            "cases": regressions
        }
```

---

## Câu 3-10: Tóm tắt các câu còn lại

## Câu 3: Adversarial Robustness Testing (10 điểm)
- Implement FGSM attack generation
- Design robustness test suite
- Calculate robustness metrics

## Câu 4: LLM Evaluation Strategy (10 điểm)
- Design RAG evaluation pipeline
- Implement LLM-as-Judge
- Handle hallucination detection

## Câu 5: Data Validation Testing (10 điểm)
- Great Expectations implementation
- Schema drift detection
- Training data quality gates

## Câu 6: Model Drift Detection (10 điểm)
- Implement drift monitoring with Evidently AI
- Design alerting thresholds
- Retrain triggers

## Câu 7: ML Pipeline Testing (10 điểm)
- TFX pipeline testing
- Integration tests for ML workflow
- Reproducibility testing

## Câu 8: Fairness Testing for NLP (10 điểm)
- Sentiment analysis bias detection
- Multilingual fairness
- Toxicity testing

## Câu 9: Model Performance Testing (10 điểm)
- Inference latency testing
- Throughput optimization
- Memory footprint testing

## Câu 10: Comprehensive AI Testing Strategy (10 điểm)
- Design end-to-end testing framework
- CI/CD integration for ML
- Monitoring and observability

---

## Tổng kết

| Câu | Topic | Điểm | Độ khó |
|-----|-------|------|--------|
| 1 | Bias Testing Strategy | 10 | Hard |
| 2 | Test Oracle Problem | 10 | Hard |
| 3 | Adversarial Robustness | 10 | Hard |
| 4 | LLM Evaluation | 10 | Medium |
| 5 | Data Validation | 10 | Medium |
| 6 | Drift Detection | 10 | Medium |
| 7 | ML Pipeline Testing | 10 | Medium |
| 8 | NLP Fairness | 10 | Hard |
| 9 | Performance Testing | 10 | Medium |
| 10 | Comprehensive Strategy | 10 | Hard |

**Tổng: 100 điểm**

---

*Created for CS423 Software Testing Final Exam Preparation*
*Focus: Testing FOR AI - testing AI/ML systems*
