# Topic 7: Testing AI Systems - Comprehensive Study Guide
**CS423 Software Testing | Final Exam Preparation**

---

## 1. Key Concepts & Theory

### 1.1 Challenges of Testing AI/ML Systems

**Why AI Testing is Different:**
```
Traditional Software:
┌─────────────────────────────────────────────┐
│ Input → Deterministic Logic → Output        │
│ (Same input always produces same output)    │
└─────────────────────────────────────────────┘

AI/ML Software:
┌─────────────────────────────────────────────┐
│ Input → Probabilistic Model → Output        │
│ (Output depends on training, randomness)    │
└─────────────────────────────────────────────┘
```

**Key Challenges:**

| Challenge | Description | Testing Approach |
|-----------|-------------|------------------|
| **Non-determinism** | Same input, different outputs | Statistical validation |
| **Black Box** | Can't inspect decision logic | Interpretability tools |
| **Test Oracle** | Hard to define "correct" | Metamorphic testing |
| **Data Dependency** | Model quality = data quality | Data validation |
| **Concept Drift** | Model degrades over time | Continuous monitoring |
| **Bias** | Unfair predictions | Fairness testing |
| **Adversarial** | Vulnerable to attacks | Robustness testing |

### 1.2 ML Testing Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│                    ML TESTING LIFECYCLE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Data        Model       Training      Deployment    Production  │
│  ┌────┐     ┌────┐       ┌────┐        ┌────┐       ┌────┐     │
│  │ 🔍 │ ──> │ 🔍 │ ──>   │ 🔍 │ ──>    │ 🔍 │ ──>   │ 🔍 │     │
│  └────┘     └────┘       └────┘        └────┘       └────┘     │
│    │          │            │              │            │         │
│  Data       Schema      Training      Integration   Monitoring  │
│  Quality    Tests       Metrics       Tests         & Drift     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Types of ML Testing

| Type | Focus | When |
|------|-------|------|
| **Data Validation** | Data quality, schema | Pre-training |
| **Model Validation** | Architecture, hyperparameters | Development |
| **Training Tests** | Convergence, metrics | Training |
| **Unit Tests** | Individual components | Development |
| **Integration Tests** | Model + infrastructure | Pre-deployment |
| **Performance Tests** | Latency, throughput | Pre-production |
| **Fairness Tests** | Bias across groups | Pre-deployment |
| **Adversarial Tests** | Robustness to attacks | Pre-deployment |
| **Monitoring** | Drift, accuracy decay | Production |

---

## 2. Tools & Techniques Comparison

### 2.1 ML Testing Frameworks

| Framework | Focus | Language | Best For |
|-----------|-------|----------|----------|
| **pytest** | General unit testing | Python | Basic ML tests |
| **pytest-ml** | ML-specific assertions | Python | Fast iteration |
| **Great Expectations** | Data validation | Python | Data pipelines |
| **TFX (TensorFlow Extended)** | Full pipeline | Python | TF production |
| **MLflow** | Experiment tracking | Python | Model versioning |
| **Evidently AI** | ML monitoring | Python | Production drift |
| **Deepchecks** | Data + model testing | Python | Quick validation |

### 2.2 Framework Comparison

**pytest vs TFX:**

| Aspect | pytest | TFX |
|--------|--------|-----|
| **Speed** | Fast (30% faster unit tests) | Full pipeline (45% faster) |
| **Memory** | Low (1.3GB) | High (3.8GB) |
| **Scope** | Unit/integration | Full ML pipeline |
| **Setup** | Simple | Complex (multiple components) |
| **Best For** | Development | Production CI/CD |

**Hybrid Approach (2025 Best Practice):**
- **Development**: pytest for fast feedback
- **CI/CD**: TFX for production validation

### 2.3 Data Validation Tools

**Great Expectations:**
```python
import great_expectations as gx

# Create expectation suite
expectation_suite = gx.ExpectationSuite(
    name="training_data_suite"
)

# Define expectations
expectation_suite.add_expectation(
    gx.expectations.ExpectColumnValuesToNotBeNull(column="age")
)
expectation_suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeBetween(
        column="age", min_value=0, max_value=120
    )
)
expectation_suite.add_expectation(
    gx.expectations.ExpectColumnDistinctValuesToBeInSet(
        column="gender", value_set=["M", "F", "Other"]
    )
)

# Validate data
results = validator.validate()
```

**TensorFlow Data Validation:**
```python
import tensorflow_data_validation as tfdv

# Generate statistics
stats = tfdv.generate_statistics_from_csv(data_path)

# Infer schema
schema = tfdv.infer_schema(stats)

# Validate new data against schema
anomalies = tfdv.validate_statistics(new_stats, schema)

# Visualize
tfdv.visualize_statistics(stats)
```

---

## 3. Practical Test Case Design

### 3.1 ML Model Unit Tests

```python
import pytest
import numpy as np

class TestModelPredictions:
    @pytest.fixture
    def model(self):
        """Load trained model"""
        from model import load_model
        return load_model("model_v1.pkl")

    def test_output_shape(self, model):
        """Verify output dimensions"""
        input_data = np.random.randn(10, 50)  # 10 samples, 50 features
        predictions = model.predict(input_data)
        assert predictions.shape == (10,)

    def test_output_range(self, model):
        """Verify predictions are in valid range"""
        input_data = np.random.randn(100, 50)
        predictions = model.predict(input_data)
        assert np.all(predictions >= 0)
        assert np.all(predictions <= 1)  # For probability outputs

    def test_determinism(self, model):
        """Same input should produce same output"""
        np.random.seed(42)
        input_data = np.random.randn(5, 50)

        pred1 = model.predict(input_data)
        pred2 = model.predict(input_data)

        np.testing.assert_array_almost_equal(pred1, pred2)

    def test_edge_cases(self, model):
        """Test boundary inputs"""
        # All zeros
        zero_input = np.zeros((1, 50))
        assert model.predict(zero_input) is not None

        # Very large values
        large_input = np.ones((1, 50)) * 1e6
        assert model.predict(large_input) is not None

        # Negative values
        neg_input = np.ones((1, 50)) * -1
        assert model.predict(neg_input) is not None
```

### 3.2 Model Accuracy Tests

```python
class TestModelAccuracy:
    @pytest.fixture
    def test_data(self):
        """Load test dataset"""
        from data import load_test_data
        return load_test_data()

    def test_accuracy_threshold(self, model, test_data):
        """Model should meet minimum accuracy"""
        X_test, y_test = test_data
        accuracy = model.score(X_test, y_test)
        assert accuracy >= 0.85, f"Accuracy {accuracy} below threshold 0.85"

    def test_precision_recall(self, model, test_data):
        """Check precision and recall balance"""
        from sklearn.metrics import precision_score, recall_score

        X_test, y_test = test_data
        y_pred = model.predict(X_test)

        precision = precision_score(y_test, y_pred)
        recall = recall_score(y_test, y_pred)

        assert precision >= 0.80, f"Precision {precision} too low"
        assert recall >= 0.80, f"Recall {recall} too low"

    def test_no_regression(self, model, test_data):
        """New model shouldn't be worse than baseline"""
        baseline_accuracy = 0.82  # From previous model
        X_test, y_test = test_data
        current_accuracy = model.score(X_test, y_test)

        assert current_accuracy >= baseline_accuracy - 0.02, \
            f"Regression detected: {current_accuracy} < {baseline_accuracy - 0.02}"
```

### 3.3 Test Cases for AI Systems

**TC-ML-001: Data Schema Validation**
- **Input**: New training data batch
- **Test**: Validate against expected schema
- **Pass**: All columns present, types correct

**TC-ML-002: Model Output Consistency**
- **Input**: Same input 100 times
- **Test**: Check prediction variance
- **Pass**: Variance < threshold (e.g., 0.01)

**TC-ML-003: Performance Degradation**
- **Input**: Batch of 10,000 predictions
- **Test**: Measure latency P95
- **Pass**: P95 < 100ms

**TC-ML-004: Fairness Across Groups**
- **Input**: Test data split by demographic
- **Test**: Compare accuracy across groups
- **Pass**: Max accuracy difference < 5%

**TC-ML-005: Adversarial Robustness**
- **Input**: Original + perturbation
- **Test**: Prediction consistency
- **Pass**: < 10% predictions changed

---

## 4. Bias & Fairness Testing

### 4.1 Types of Bias

| Bias Type | Description | Example |
|-----------|-------------|---------|
| **Selection Bias** | Non-representative data | Medical data from one hospital |
| **Reporting Bias** | Unusual events over-represented | News-trained models |
| **Historical Bias** | Past discrimination in data | Hiring data with gender gap |
| **Measurement Bias** | Flawed data collection | Low-quality images for some groups |
| **Aggregation Bias** | Model assumes homogeneity | Same model for all demographics |

### 4.2 Fairness Metrics

| Metric | Definition | Goal |
|--------|------------|------|
| **Demographic Parity** | Equal positive rate across groups | P(Ŷ=1\|A=0) = P(Ŷ=1\|A=1) |
| **Equalized Odds** | Equal TPR and FPR across groups | TPR, FPR equal |
| **Equal Opportunity** | Equal TPR across groups | TPR equal |
| **Predictive Parity** | Equal precision across groups | Precision equal |
| **Individual Fairness** | Similar individuals get similar predictions | Distance-based |

### 4.3 Fairness Testing with AIF360

```python
from aif360.datasets import BinaryLabelDataset
from aif360.metrics import BinaryLabelDatasetMetric, ClassificationMetric
from aif360.algorithms.preprocessing import Reweighing

# Load data
dataset = BinaryLabelDataset(
    df=df,
    label_names=['hired'],
    protected_attribute_names=['gender'],
    favorable_label=1,
    unfavorable_label=0
)

# Calculate disparate impact
metric = BinaryLabelDatasetMetric(
    dataset,
    unprivileged_groups=[{'gender': 0}],  # Female
    privileged_groups=[{'gender': 1}]      # Male
)

print(f"Disparate Impact: {metric.disparate_impact()}")
# Goal: Between 0.8 and 1.25

# Calculate equalized odds
predictions_dataset = dataset.copy()
predictions_dataset.labels = model.predict(X)

clf_metric = ClassificationMetric(
    dataset,
    predictions_dataset,
    unprivileged_groups=[{'gender': 0}],
    privileged_groups=[{'gender': 1}]
)

print(f"Equal Opportunity Difference: {clf_metric.equal_opportunity_difference()}")
# Goal: Close to 0

# Mitigation: Reweighing
rw = Reweighing(
    unprivileged_groups=[{'gender': 0}],
    privileged_groups=[{'gender': 1}]
)
dataset_transformed = rw.fit_transform(dataset)
```

---

## 5. Adversarial Testing

### 5.1 Types of Adversarial Attacks

| Attack Type | Description | Target |
|-------------|-------------|--------|
| **Evasion** | Fool model at inference | Predictions |
| **Poisoning** | Corrupt training data | Model |
| **Model Extraction** | Steal model parameters | IP |
| **Model Inversion** | Recover training data | Privacy |
| **Backdoor** | Hidden trigger for wrong prediction | Security |

### 5.2 Adversarial Testing Techniques

**FGSM (Fast Gradient Sign Method):**
```python
import torch

def fgsm_attack(image, epsilon, gradient):
    """Generate adversarial example"""
    # Get sign of gradient
    sign = gradient.sign()

    # Create perturbed image
    perturbed = image + epsilon * sign

    # Clamp to valid range
    perturbed = torch.clamp(perturbed, 0, 1)

    return perturbed

# Usage
model.eval()
image.requires_grad = True
output = model(image)
loss = criterion(output, label)
loss.backward()

adversarial_image = fgsm_attack(image, epsilon=0.03, gradient=image.grad)
```

**Testing Robustness:**
```python
def test_adversarial_robustness(model, test_data, epsilon=0.03):
    """Test model robustness to adversarial examples"""
    original_correct = 0
    adversarial_correct = 0

    for image, label in test_data:
        # Original prediction
        original_pred = model.predict(image)
        if original_pred == label:
            original_correct += 1

        # Adversarial prediction
        adversarial_image = generate_adversarial(image, epsilon)
        adversarial_pred = model.predict(adversarial_image)
        if adversarial_pred == label:
            adversarial_correct += 1

    original_accuracy = original_correct / len(test_data)
    adversarial_accuracy = adversarial_correct / len(test_data)
    robustness = adversarial_accuracy / original_accuracy

    print(f"Original Accuracy: {original_accuracy:.2%}")
    print(f"Adversarial Accuracy: {adversarial_accuracy:.2%}")
    print(f"Robustness: {robustness:.2%}")

    assert robustness >= 0.70, "Model is too vulnerable to adversarial attacks"
```

---

## 6. LLM Evaluation (2025)

### 6.1 LLM Evaluation Challenges

**Static Benchmarks Saturating:**
- MMLU, MMLU-Pro reaching ceiling
- Models memorize test data (GSM8K evidence)
- Benchmarks don't reflect real-world performance

**2025 Trends:**
- Dynamic/interactive evaluations
- AgentBench: Multi-turn reasoning
- Humanity's Last Exam (HLE): Expert-level questions
- Custom domain-specific evaluations

### 6.2 LLM Evaluation Approaches

| Approach | Metrics | Best For |
|----------|---------|----------|
| **Automated** | BLEU, ROUGE, Perplexity | Quick iteration |
| **Human** | Coherence, relevance, fluency | Quality assessment |
| **LLM-as-Judge** | GPT-4 evaluation | Scalable quality |
| **Behavioral** | Task completion rate | Functional testing |
| **Safety** | Refusal rate, harmful outputs | Alignment |

### 6.3 LLM Testing Example

```python
from langchain.evaluation import load_evaluator
from langchain.chat_models import ChatOpenAI

# LLM-as-Judge evaluation
evaluator = load_evaluator("criteria", criteria="helpfulness")

def evaluate_response(question, response):
    """Evaluate LLM response quality"""
    result = evaluator.evaluate_strings(
        prediction=response,
        input=question,
    )
    return result["score"], result["reasoning"]

# Behavioral testing
def test_factual_accuracy(llm, qa_pairs):
    """Test factual correctness"""
    correct = 0
    for question, expected_answer in qa_pairs:
        response = llm.invoke(question)
        # Use fuzzy matching or semantic similarity
        if is_semantically_similar(response, expected_answer):
            correct += 1
    return correct / len(qa_pairs)

# Safety testing
def test_harmful_refusal(llm, harmful_prompts):
    """Test refusal of harmful requests"""
    refusals = 0
    for prompt in harmful_prompts:
        response = llm.invoke(prompt)
        if contains_refusal(response):
            refusals += 1
    return refusals / len(harmful_prompts)
```

---

## 7. Application-Level Exam Questions

### Question 1: Bias Testing Strategy
**Scenario:** Hiring AI model shows 80% accuracy for male candidates but 65% for female candidates.

**Question:** How would you test and mitigate this bias?

**Answer:**
```python
# 1. Quantify bias
from aif360.metrics import ClassificationMetric

# Split data by gender
metric = ClassificationMetric(
    test_data,
    predictions,
    unprivileged_groups=[{'gender': 'F'}],
    privileged_groups=[{'gender': 'M'}]
)

disparate_impact = metric.disparate_impact()  # Should be 0.8-1.25
statistical_parity = metric.statistical_parity_difference()

# 2. Identify bias sources
# - Training data imbalance (check gender distribution)
# - Feature correlation (check if features proxy for gender)
# - Historical bias (past hiring data)

# 3. Mitigation strategies
# Pre-processing: Reweigh samples
from aif360.algorithms.preprocessing import Reweighing
rw = Reweighing(unprivileged_groups, privileged_groups)
balanced_data = rw.fit_transform(training_data)

# In-processing: Add fairness constraint
from aif360.algorithms.inprocessing import PrejudiceRemover
model = PrejudiceRemover(eta=1.0)  # eta = fairness weight

# Post-processing: Calibrate thresholds
from aif360.algorithms.postprocessing import CalibratedEqOddsPostprocessing
calibrator = CalibratedEqOddsPostprocessing(unprivileged_groups, privileged_groups)
fair_predictions = calibrator.fit_predict(dataset, predictions)

# 4. Validation
# - Retest with balanced test set
# - Check disparate impact improved
# - Verify accuracy still acceptable
```

### Question 2: Test Oracle for Image Classifier
**Scenario:** Image classification model for product defects. How do you define "correct" output?

**Question:** Design a test oracle approach.

**Answer:**
```
Approach: Multi-layer oracle with thresholds

1. Statistical Oracle:
   - Accept prediction if confidence > 0.85
   - Flag for review if 0.50 < confidence < 0.85
   - Reject if confidence < 0.50

2. Metamorphic Testing:
   - Rotation: Same class for rotated images
   - Brightness: Same class for slight brightness changes
   - Cropping: Same class if defect visible

3. Differential Testing:
   - Compare with human-labeled ground truth
   - Use ensemble of models (majority vote)

4. Reference Oracle:
   - Small set of expert-labeled images
   - Must achieve 95%+ accuracy on reference set

Implementation:
```python
def oracle(prediction, image, reference_set=None):
    # Layer 1: Confidence check
    if prediction.confidence < 0.50:
        return "REJECT", "Low confidence"
    elif prediction.confidence < 0.85:
        return "REVIEW", "Medium confidence"

    # Layer 2: Metamorphic check
    rotated = rotate(image, 90)
    rotated_pred = model.predict(rotated)
    if rotated_pred.label != prediction.label:
        return "REVIEW", "Failed rotation invariance"

    # Layer 3: Reference check (if applicable)
    if image in reference_set:
        if prediction.label != reference_set[image]:
            return "FAIL", "Incorrect vs reference"

    return "PASS", "All checks passed"
```

### Question 3: LLM Evaluation Strategy
**Scenario:** RAG chatbot for customer support needs evaluation.

**Question:** Design comprehensive evaluation plan.

**Answer:**
```
Evaluation Dimensions:

1. Retrieval Quality:
   - Recall@K: % of relevant docs retrieved
   - Precision@K: % of retrieved docs relevant
   - MRR: Mean Reciprocal Rank

2. Response Quality:
   - Factual Accuracy: Cross-reference with knowledge base
   - Relevance: Answer addresses question (LLM-as-judge)
   - Completeness: All aspects answered

3. Safety & Guardrails:
   - Refusal rate for out-of-scope queries
   - Hallucination detection
   - PII handling

4. User Experience:
   - Response latency
   - Conversation coherence
   - Escalation rate to human

Test Suite:
```python
def evaluate_rag_chatbot(chatbot, test_set):
    results = {
        "retrieval_recall": [],
        "answer_relevance": [],
        "factual_accuracy": [],
        "safety": [],
        "latency": []
    }

    for query, expected_docs, expected_answer in test_set:
        # Retrieval evaluation
        retrieved = chatbot.retrieve(query)
        recall = len(set(retrieved) & set(expected_docs)) / len(expected_docs)
        results["retrieval_recall"].append(recall)

        # Response evaluation
        start = time.time()
        response = chatbot.respond(query)
        latency = time.time() - start
        results["latency"].append(latency)

        # LLM-as-judge for relevance
        relevance = evaluate_relevance(query, response)
        results["answer_relevance"].append(relevance)

        # Factual check against knowledge base
        accuracy = check_facts(response, knowledge_base)
        results["factual_accuracy"].append(accuracy)

    # Aggregate metrics
    return {k: np.mean(v) for k, v in results.items()}
```

---

## 8. Cheatsheet / Quick Reference

### ML Testing Commands

```python
# pytest-ml assertions
import pytest_ml

# Model accuracy
pytest_ml.assert_model_accuracy(model, X_test, y_test, min_accuracy=0.85)

# Feature importance
pytest_ml.assert_feature_importance(model, X, threshold=0.05)

# Distribution shift
pytest_ml.assert_no_distribution_shift(X_train, X_test, p_value=0.05)
```

### Fairness Metrics Quick Reference

```python
from aif360.metrics import ClassificationMetric

metric = ClassificationMetric(dataset, predictions,
    unprivileged_groups=[{'attr': 0}],
    privileged_groups=[{'attr': 1}])

# Common metrics
metric.disparate_impact()           # Goal: 0.8-1.25
metric.statistical_parity_difference()  # Goal: ~0
metric.equal_opportunity_difference()   # Goal: ~0
metric.average_odds_difference()        # Goal: ~0
```

### Adversarial Testing Quick Reference

```python
# FGSM attack
epsilon = 0.03
perturbation = epsilon * gradient.sign()
adversarial = original + perturbation

# Robustness metrics
robustness = adversarial_accuracy / original_accuracy
# Goal: > 70%
```

### ML Testing Checklist

- [ ] Data validation (schema, types, ranges)
- [ ] Data quality (missing values, outliers)
- [ ] Model output shape and type
- [ ] Model determinism
- [ ] Accuracy threshold
- [ ] No regression from baseline
- [ ] Fairness across groups
- [ ] Adversarial robustness
- [ ] Performance (latency, throughput)
- [ ] Drift monitoring

---

## 9. Key Takeaways for Exam

1. **Test Oracle**: Use statistical, metamorphic, differential approaches
2. **Fairness**: AIF360 for metrics and mitigation; disparate impact 0.8-1.25
3. **Adversarial**: FGSM for attack generation; 70%+ robustness target
4. **Data Validation**: Great Expectations or TFX before training
5. **LLM Evaluation**: Hybrid (automated + human + LLM-as-judge)
6. **Bias Types**: Selection, reporting, historical, measurement
7. **Framework Choice**: pytest for dev, TFX for production

---

*Study time recommended: 4-5 hours*
*Practice: Implement fairness testing with AIF360 on a sample dataset*
