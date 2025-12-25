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

**Great Expectations Concepts:**
- Create expectation suite with rules
- Common expectations: `NotBeNull`, `ValuesToBeBetween`, `DistinctValuesToBeInSet`
- Run `validator.validate()` to check data

**TensorFlow Data Validation Concepts:**
- Generate statistics: `tfdv.generate_statistics_from_csv()`
- Infer schema from data: `tfdv.infer_schema(stats)`
- Validate new data: `tfdv.validate_statistics(new_stats, schema)`
- Detect anomalies automatically

---

## 3. Practical Test Case Design

### 3.1 ML Model Unit Tests

**Key Test Types:**
| Test | What to Check | Assert |
|------|---------------|--------|
| Output shape | Dimensions correct | `predictions.shape == (N,)` |
| Output range | Values valid | `0 <= predictions <= 1` (probabilities) |
| Determinism | Same input → same output | `assert_array_almost_equal(pred1, pred2)` |
| Edge cases | Zeros, large, negative values | Returns without error |

### 3.2 Model Accuracy Tests

**Key Metrics:**
| Test | Threshold | Assert |
|------|-----------|--------|
| Accuracy | ≥85% | `model.score(X_test, y_test) >= 0.85` |
| Precision | ≥80% | `precision_score(y_test, y_pred) >= 0.80` |
| Recall | ≥80% | `recall_score(y_test, y_pred) >= 0.80` |
| No regression | ≥baseline - 2% | Current accuracy ≥ previous model |

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

**Key Concepts:**
- Define protected groups: `unprivileged_groups=[{'gender': 0}]`
- Calculate metrics: `BinaryLabelDatasetMetric`, `ClassificationMetric`
- Goal: Disparate Impact 0.8-1.25, Equal Opportunity Difference ~0

**Mitigation Strategies:**
- Pre-processing: `Reweighing` to balance samples
- In-processing: `PrejudiceRemover` with fairness constraint
- Post-processing: `CalibratedEqOddsPostprocessing` to adjust thresholds

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
- Formula: `adversarial = original + epsilon * gradient.sign()`
- Typical epsilon: 0.03
- Clamp to valid range [0, 1]

**Robustness Testing:**
- Metric: `robustness = adversarial_accuracy / original_accuracy`
- Target: ≥70% robustness
- If robustness <70% → model too vulnerable

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

### 6.3 LLM Testing Concepts

**Key Test Types:**
| Test | Method | Metric |
|------|--------|--------|
| LLM-as-Judge | Use GPT-4 to evaluate | Helpfulness score |
| Factual Accuracy | Compare to expected | % semantically similar |
| Safety | Check refusal of harmful | Refusal rate |

---

## 7. LLM Evaluation Frameworks (2025)

### 7.1 OpenAI Evals

**What is OpenAI Evals?**
- Framework for evaluating LLMs and LLM systems
- API released April 2025 for programmatic evaluation
- Supports basic (ground-truth) and model-graded evals

**Core Concepts:**
```
┌─────────────────────────────────────────────────────────────┐
│                    OPENAI EVALS STRUCTURE                    │
├─────────────────────────────────────────────────────────────┤
│  Eval              → Defines task + testing criteria         │
│  Run               → Executes eval against model            │
│  data_source_config→ Schema for test data                   │
│  testing_criteria  → Graders (how to score outputs)         │
└─────────────────────────────────────────────────────────────┘
```

**Types of Evals:**

| Type | Description | Use Case |
|------|-------------|----------|
| **Basic (Ground-truth)** | Compare output to known answer | Multiple choice, factual QA |
| **Model-graded** | LLM judges output quality | Open-ended, subjective tasks |
| **Code-graded** | Custom logic evaluates | Structured outputs, JSON |

**Setup:** `pip install openai>=1.20.0` + set API key

**Basic Eval Concepts:**
- Create eval with `data_source_config` (schema) + `testing_criteria` (graders)
- `string_match`: Compare output to expected
- `llm_grader`: Use LLM to score (1-5) with custom prompt
- Run eval: `client.evals.runs.create(eval_id, model, data_source)`

### 7.2 DeepEval Framework

**What is DeepEval?**
- Open-source LLM evaluation framework (pytest for LLMs)
- 800k+ daily evaluations, 14+ metrics
- Supports RAG, agents, chatbots

**Key Metrics:**

| Metric | Purpose | Formula |
|--------|---------|---------|
| **Hallucination** | Factual accuracy | Claims supported / Total |
| **Faithfulness** | Grounded in context | Same as hallucination |
| **Answer Relevancy** | Addresses question | Score 0-1 |
| **Contextual Precision** | Relevant chunks ranked high | Precision@K |
| **Contextual Recall** | All relevant retrieved | Retrieved / Total relevant |

**Usage Concepts:**
- Create `LLMTestCase(input, actual_output, context)`
- Define metrics with threshold: `HallucinationMetric(threshold=0.7)`
- Run: `evaluate([test_cases], [metrics])`
- CI/CD: Use `assert_test(test_case, metrics)` with pytest

### 7.3 Ragas (RAG Assessment)

**What is Ragas?**
- Evaluation framework specifically for RAG pipelines
- Reference-free evaluation (no ground-truth needed)
- Works with production traces

**Core RAG Metrics:**

```
┌─────────────────────────────────────────────────────────────┐
│                    RAGAS METRICS                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  RETRIEVAL QUALITY              GENERATION QUALITY          │
│  ┌─────────────────┐            ┌─────────────────┐         │
│  │Context Precision│            │  Faithfulness   │         │
│  │Context Recall   │            │Answer Relevancy │         │
│  └─────────────────┘            └─────────────────┘         │
│                                                              │
│  Faithfulness = Claims supported by context / Total claims  │
│  Answer Relevancy = How well answer addresses question      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**Usage:** `pip install ragas`, create Dataset with question/answer/contexts/ground_truth

**Metrics:**
| Metric | Score | Meaning |
|--------|-------|---------|
| Faithfulness | 1.0 | No hallucination |
| Faithfulness | 0.85 | 15% claims unsupported |
| Context Precision | 0.9 | 90% top chunks relevant |
| Context Recall | 0.8 | 80% needed info retrieved |

### 7.4 Prompt Testing & Promptfoo

**Why Prompt Testing?**
- Prompts are code → need version control + testing
- Small changes can significantly impact output
- Non-deterministic → need statistical validation

**Promptfoo Overview:**
- CLI tool for testing prompts, agents, RAGs
- Red teaming, security testing
- Compare models (GPT, Claude, Gemini, Llama)

**Setup:** `npm install -g promptfoo` or `pip install promptfoo`

**Config (YAML):**
- `prompts`: List of prompt templates
- `providers`: Models to test (gpt-4o, claude-3-sonnet)
- `tests`: Input vars + assertions

**Assertion Types:**
| Type | Description |
|------|-------------|
| `contains` | Output contains string |
| `equals` | Exact match |
| `llm-rubric` | LLM judges quality |
| `cost` | API cost threshold |
| `latency` | Response time |

**Commands:** `promptfoo eval`, `promptfoo view`

**Red Teaming:** Plugins for harmful content, PII, prompt-injection

### 7.5 LLM Evaluation Comparison

| Framework | Best For | Metrics | CI/CD |
|-----------|----------|---------|-------|
| **OpenAI Evals** | OpenAI models | Basic, model-graded | API |
| **DeepEval** | General LLMs | 14+ metrics | pytest |
| **Ragas** | RAG pipelines | Retrieval + generation | Python |
| **Promptfoo** | Prompt testing | Custom assertions | CLI |

**Choosing the Right Tool:**
```
Use OpenAI Evals when:
└── Testing OpenAI models primarily
└── Need model-graded evaluation
└── Want API-first approach

Use DeepEval when:
└── Need comprehensive metrics suite
└── Want pytest integration
└── Testing RAG, agents, chatbots

Use Ragas when:
└── Focused on RAG pipeline quality
└── Need reference-free evaluation
└── Want production trace analysis

Use Promptfoo when:
└── Comparing multiple models/prompts
└── Need red teaming/security testing
└── Want declarative YAML config
```

---

## 8. Application-Level Exam Questions (Updated)

### Question 1: Bias Testing Strategy
**Scenario:** Hiring AI: 80% accuracy (male) vs 65% (female)

**Answer:**
1. **Quantify:** Disparate impact (goal: 0.8-1.25), statistical parity difference
2. **Identify sources:** Data imbalance, feature proxy, historical bias
3. **Mitigate:** Pre-processing (Reweighing), In-processing (PrejudiceRemover), Post-processing (CalibratedEqOdds)
4. **Validate:** Retest, verify disparate impact improved, accuracy still acceptable

### Question 2: Test Oracle for Image Classifier
**Scenario:** Product defect classification - how to define "correct"?

**Answer - Multi-layer Oracle:**
| Layer | Method | Threshold |
|-------|--------|-----------|
| Statistical | Confidence check | Accept >0.85, Review 0.5-0.85, Reject <0.5 |
| Metamorphic | Rotation/brightness invariance | Same class for transforms |
| Reference | Expert-labeled set | ≥95% accuracy |

### Question 3: LLM Evaluation Strategy
**Scenario:** RAG chatbot evaluation plan

**Answer - Dimensions:**
| Dimension | Metrics |
|-----------|---------|
| Retrieval | Recall@K, Precision@K, MRR |
| Response | Factual accuracy, Relevance (LLM-judge), Completeness |
| Safety | Refusal rate, Hallucination, PII handling |
| UX | Latency, Coherence, Escalation rate |

---

## 8. Cheatsheet / Quick Reference

### ML Testing Commands

| Command | Purpose |
|---------|---------|
| `assert_model_accuracy(model, X, y, min=0.85)` | Check accuracy threshold |
| `assert_feature_importance(model, X, threshold=0.05)` | Verify feature relevance |
| `assert_no_distribution_shift(X_train, X_test)` | Check data drift |

### Fairness Metrics Quick Reference

| Metric | Goal |
|--------|------|
| `disparate_impact()` | 0.8-1.25 |
| `statistical_parity_difference()` | ~0 |
| `equal_opportunity_difference()` | ~0 |
| `average_odds_difference()` | ~0 |

### Adversarial Quick Reference

- FGSM: `adversarial = original + 0.03 * gradient.sign()`
- Robustness: `adversarial_accuracy / original_accuracy` (goal: >70%)

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

## 10. Key Takeaways for Exam

1. **Test Oracle**: Use statistical, metamorphic, differential approaches
2. **Fairness**: AIF360 for metrics and mitigation; disparate impact 0.8-1.25
3. **Adversarial**: FGSM for attack generation; 70%+ robustness target
4. **Data Validation**: Great Expectations or TFX before training
5. **LLM Evaluation**: Hybrid (automated + human + LLM-as-judge)
6. **Bias Types**: Selection, reporting, historical, measurement
7. **OpenAI Evals**: Basic (ground-truth) + Model-graded; API-first
8. **DeepEval**: pytest for LLMs; 14+ metrics; Hallucination = Claims supported / Total
9. **Ragas**: RAG evaluation; Faithfulness + Context Precision/Recall
10. **Promptfoo**: Prompt testing CLI; red teaming; compare models

---

*Study time recommended: 4-5 hours*
*Practice: Implement fairness testing with AIF360 on a sample dataset*
