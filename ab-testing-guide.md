# A/B Testing Guide for AI Agents

Learn how to use MeshAI's A/B testing to systematically improve your AI agents through data-driven experimentation.

## What is A/B Testing for AI Agents?

A/B testing (also called split testing) allows you to compare two different agents or model configurations to see which performs better on your specific use case. Instead of guessing which approach works best, you let real data decide.

### Why A/B Test Your Agents?

- **Choose the right model**: GPT-4 vs Claude vs Gemini - which is best for YOUR use case?
- **Optimize prompts**: Test different system prompts to maximize accuracy
- **Reduce costs**: Find the cheapest model that meets quality standards
- **Improve continuously**: Always be testing incremental improvements
- **Make data-driven decisions**: Replace gut feelings with statistical evidence

## Quick Start

### 1. Create an Experiment

```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/experiments/create \
  -H "X-API-Key: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "GPT-4 vs Claude-3 Comparison",
    "description": "Which model gives better customer support responses?",
    "variant_a": "agent-gpt4",
    "variant_b": "agent-claude3",
    "traffic_split": 0.5,
    "min_samples": 100,
    "confidence_level": 0.95,
    "metrics": ["accuracy", "latency", "cost"]
  }'
```

**Response**:
```json
{
  "experiment_id": "exp_abc123",
  "status": "active",
  "created_at": "2025-10-07T18:30:00Z"
}
```

### 2. Route Traffic Through the Experiment

For each task, ask which variant to use:

```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/experiments/exp_abc123/assign \
  -H "X-API-Key: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"task_id": "task-001"}'
```

**Response**:
```json
{
  "assigned_variant": "variant_a",
  "agent_id": "agent-gpt4"
}
```

### 3. Send Task to Assigned Agent

Use the assigned agent_id to process the task:

```python
# Get assignment
assignment = client.experiments.assign_variant(
    experiment_id="exp_abc123",
    task_id="task-001"
)

# Use the assigned agent
result = run_agent(
    agent_id=assignment.agent_id,
    task=task_data
)

# Log the evaluation
client.evaluations.run(
    agent_id=assignment.agent_id,
    task_id="task-001",
    prompt=task_data.prompt,
    response=result.response,
    template="accuracy"
)
```

### 4. Check Results

After collecting enough samples (min_samples parameter):

```bash
curl "https://evaluations.meshai.dev/v1/evaluation/experiments/exp_abc123/results" \
  -H "X-API-Key: ${API_KEY}"
```

**Response**:
```json
{
  "experiment_id": "exp_abc123",
  "status": "completed",
  "winner": "agent-claude3",
  "confidence": 0.97,
  "statistical_significance": true,
  "recommendation": "Deploy variant_b (agent-claude3)",

  "variant_a": {
    "agent_id": "agent-gpt4",
    "sample_count": 152,
    "metrics": {
      "accuracy": {"mean": 0.87, "std_dev": 0.08},
      "latency": {"mean": 1.2, "std_dev": 0.3},
      "cost": {"mean": 0.05, "std_dev": 0.01}
    }
  },

  "variant_b": {
    "agent_id": "agent-claude3",
    "sample_count": 148,
    "metrics": {
      "accuracy": {"mean": 0.91, "std_dev": 0.06},
      "latency": {"mean": 1.1, "std_dev": 0.25},
      "cost": {"mean": 0.03, "std_dev": 0.005}
    }
  },

  "p_value": 0.012,
  "effect_size": 0.04
}
```

**Winner**: Claude-3 has higher accuracy (0.91 vs 0.87), lower latency (1.1s vs 1.2s), and 40% lower cost!

## Best Practices

### 1. Define Success Metrics Upfront

Choose 1-3 key metrics before starting:

| Metric | When to Use |
|--------|------------|
| **Accuracy** | Correctness is critical (medical, legal, financial) |
| **Latency** | Speed matters (chatbots, real-time systems) |
| **Cost** | Budget constraints |
| **User Satisfaction** | End-user experience is key |
| **Hallucination Rate** | Factual accuracy is critical |

```json
{
  "metrics": ["accuracy", "cost"],
  "weights": {
    "accuracy": 2.0,  // Twice as important as cost
    "cost": 1.0
  }
}
```

### 2. Choose the Right Sample Size

**Minimum samples needed** depends on expected effect size:

| Expected Difference | Minimum Samples per Variant |
|-------------------|---------------------------|
| Large (>10%) | 100 |
| Medium (5-10%) | 300 |
| Small (2-5%) | 1000 |
| Tiny (<2%) | 5000+ |

**Formula** (for 95% confidence):
```
n = (z² × σ²) / d²
```
Where:
- z = 1.96 (for 95% confidence)
- σ = standard deviation
- d = minimum detectable difference

**MeshAI automatically calculates** when you have statistical significance!

### 3. Use Appropriate Traffic Splitting

| Scenario | Recommended Split |
|----------|------------------|
| **Testing new experimental feature** | 90/10 (keep most traffic on proven variant) |
| **Comparing similar models** | 50/50 (equal split) |
| **Multi-armed bandit** | Dynamic (start 50/50, gradually favor winner) |

```json
{
  "traffic_split": 0.1,  // 10% to variant_b
  "adaptive": true       // Gradually shift traffic to winner
}
```

### 4. Avoid Common Pitfalls

#### ❌ Stopping Too Early
```
"We ran 20 samples and variant A looks better!"
```
**Problem**: Not statistically significant. Random chance could explain the difference.

**Solution**: Wait for `statistical_significance: true` in results.

#### ❌ P-Hacking
```
"Let's keep testing until we get the result we want!"
```
**Problem**: Running multiple tests increases false positives.

**Solution**: Define success criteria before starting. Stick to it.

#### ❌ Ignoring Sample Bias
```
"All our test data is from enterprise customers."
```
**Problem**: Results may not generalize to all users.

**Solution**: Ensure test data represents real production distribution.

#### ❌ Testing Too Many Things
```
"Let's test 5 different models at once!"
```
**Problem**: Harder to achieve significance. Confusing results.

**Solution**: Test ONE change at a time. Then iterate.

### 5. Understand Statistical Significance

**P-value < 0.05** means:
- Less than 5% chance the difference is random
- 95% confident the winner is truly better

**Confidence Level**:
- 0.90 (90%) = Less strict, faster results
- 0.95 (95%) = Standard for most experiments ✅
- 0.99 (99%) = Very strict, requires more samples

**Effect Size**:
- Small (0.2) = Noticeable but minor improvement
- Medium (0.5) = Moderate improvement
- Large (0.8+) = Substantial improvement

## Advanced Techniques

### Multi-Armed Bandit

Instead of fixed traffic splits, automatically shift traffic to the winning variant:

```json
{
  "strategy": "thompson_sampling",
  "exploration_rate": 0.1,
  "min_samples": 50
}
```

This **balances exploration** (trying both) with **exploitation** (favoring the winner).

### Sequential Testing

Stop the experiment as soon as you reach significance (instead of waiting for min_samples):

```json
{
  "sequential": true,
  "alpha": 0.05,
  "power": 0.8
}
```

**Benefit**: Faster results when difference is large.

### Multi-Metric Optimization

Optimize for multiple metrics with weights:

```json
{
  "metrics": {
    "accuracy": {"weight": 2.0, "threshold": 0.8},
    "latency": {"weight": 1.0, "threshold": 2.0},
    "cost": {"weight": 1.5, "threshold": 0.1}
  },
  "optimization": "pareto"  // Find best tradeoff
}
```

## Real-World Examples

### Example 1: Reducing Costs

**Goal**: Find the cheapest model that maintains 90%+ accuracy.

```python
experiment = client.experiments.create(
    name="GPT-4 vs GPT-3.5 Cost Optimization",
    variant_a="agent-gpt4",      # Expensive but accurate
    variant_b="agent-gpt3.5",    # Cheaper but lower quality?
    metrics=["accuracy", "cost"],
    min_samples=500,
    acceptance_criteria={
        "accuracy": {"min": 0.90},   # Must maintain 90% accuracy
        "cost": {"optimize": "min"}   # Minimize cost
    }
)
```

**Result**: If GPT-3.5 achieves 91% accuracy at 1/10th the cost, it wins!

### Example 2: Improving Response Quality

**Goal**: Improve customer satisfaction with better prompts.

```python
experiment = client.experiments.create(
    name="Standard vs Enhanced System Prompt",
    variant_a="agent-standard-prompt",
    variant_b="agent-enhanced-prompt",
    metrics=["user_satisfaction", "accuracy"],
    min_samples=300
)
```

### Example 3: Balancing Speed and Quality

**Goal**: Faster responses without sacrificing accuracy.

```python
experiment = client.experiments.create(
    name="Speed vs Quality Tradeoff",
    variant_a="agent-claude-sonnet",  # Faster
    variant_b="agent-claude-opus",    # More accurate
    metrics=["accuracy", "latency"],
    acceptance_criteria={
        "accuracy": {"min": 0.85},
        "latency": {"max": 3.0}  # Must respond within 3 seconds
    }
)
```

## Dashboard Monitoring

View real-time experiment progress at:
**[app.meshai.dev/evaluations/experiments](https://app.meshai.dev/evaluations/experiments)**

Features:
- Live sample counts and scores
- Statistical significance indicator
- Confidence intervals visualization
- Cost comparison charts
- Winner recommendation

## When to Stop an Experiment

✅ **Safe to stop when**:
- `statistical_significance: true`
- Sample count ≥ `min_samples`
- Winner is clearly better across all key metrics

⚠️ **Keep running when**:
- Results are close (p-value > 0.05)
- High variance in scores
- Not enough samples yet
- Confidence level not reached

❌ **Stop and redesign when**:
- Both variants perform poorly
- No clear winner after 10x min_samples
- Business requirements changed

## Next Steps

1. **Start simple**: Compare two models on one metric
2. **Iterate**: Once you have a winner, test improvements
3. **Automate**: Integrate experiments into your CI/CD pipeline
4. **Scale**: Run multiple experiments in parallel

## Resources

- [API Reference](./evaluation-api-reference.md) - Complete API documentation
- [Quickstart Guide](./evaluation-testing-quickstart.md) - Get started in 5 minutes
- [Statistical Testing Primer](./statistical-testing-primer.md) - Understanding p-values and confidence

## Support

Questions? Email us at support@meshai.dev or visit [app.meshai.dev/support](https://app.meshai.dev/support)
