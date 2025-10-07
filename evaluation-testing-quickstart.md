# Evaluation & Testing Quickstart

Get started with MeshAI's automated evaluation and A/B testing in under 5 minutes.

## Overview

MeshAI's Evaluation & Testing service helps you:
- ✅ Automatically score agent responses for accuracy and quality
- ✅ Run A/B tests to compare different agents or models
- ✅ Track workflow adherence to ensure agents follow expected steps
- ✅ Create custom metrics for domain-specific evaluation

## Prerequisites

- MeshAI account with API key
- At least one registered agent in your account
- (Optional) Python 3.8+ for SDK examples

## Quick Start

### 1. Get Your API Key

1. Visit [app.meshai.dev](https://app.meshai.dev)
2. Navigate to **Settings** → **API Keys**
3. Click **Generate New Key**
4. Copy your API key (you'll need this for all API calls)

### 2. Run Your First Evaluation

#### Using cURL (Any Platform)

```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/run-eval \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "my-agent",
    "task_id": "task-001",
    "prompt": "What is the capital of France?",
    "response": "The capital of France is Paris.",
    "expected_output": "Paris",
    "template": "accuracy"
  }'
```

**Expected Response:**
```json
{
  "success": true,
  "eval_id": "eval_abc123",
  "score": 0.95,
  "passed": true,
  "metrics": {
    "accuracy": 0.95,
    "relevance": 0.92,
    "coherence": 0.98
  },
  "feedback": "Excellent response, accurate and complete"
}
```

#### Using Python SDK

```python
from meshai import MeshAIClient

# Initialize client
client = MeshAIClient(api_key="YOUR_API_KEY")

# Run evaluation
result = client.evaluations.run(
    agent_id="my-agent",
    task_id="task-001",
    prompt="What is the capital of France?",
    response="The capital of France is Paris.",
    expected_output="Paris",
    template="accuracy"
)

print(f"Score: {result.score}")  # 0.95
print(f"Passed: {result.passed}")  # True
```

### 3. View Results in Dashboard

1. Visit [app.meshai.dev/evaluations](https://app.meshai.dev/evaluations)
2. See your evaluation history, scores, and trends
3. Filter by agent, date range, or score threshold

### 4. Run an A/B Test

Compare two agents to see which performs better:

```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/experiments/create \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "GPT-4 vs Claude Comparison",
    "description": "Test which model gives better responses",
    "variant_a": "agent-gpt4",
    "variant_b": "agent-claude",
    "traffic_split": 0.5,
    "min_samples": 100,
    "metrics": ["accuracy", "latency"]
  }'
```

**Response:**
```json
{
  "experiment_id": "exp_xyz789",
  "status": "active",
  "created_at": "2025-10-07T18:30:00Z"
}
```

### 5. Get Experiment Results

After collecting enough samples:

```bash
curl "https://evaluations.meshai.dev/v1/evaluation/experiments/exp_xyz789/results" \
  -H "X-API-Key: YOUR_API_KEY"
```

**Response:**
```json
{
  "experiment_id": "exp_xyz789",
  "status": "completed",
  "winner": "agent-claude",
  "confidence": 0.95,
  "variant_a": {
    "agent_id": "agent-gpt4",
    "avg_score": 0.87,
    "sample_count": 152
  },
  "variant_b": {
    "agent_id": "agent-claude",
    "avg_score": 0.91,
    "sample_count": 148
  },
  "statistical_significance": true
}
```

## Common Evaluation Templates

### 1. **Accuracy** - How correct is the response?
```json
{
  "template": "accuracy",
  "expected_output": "Paris"
}
```

### 2. **Relevance** - Is the response on-topic?
```json
{
  "template": "relevance",
  "context": {"topic": "geography"}
}
```

### 3. **Coherence** - Is the response well-structured?
```json
{
  "template": "coherence"
}
```

### 4. **Hallucination Detection** - Is the agent making things up?
```json
{
  "template": "hallucination",
  "grounding_docs": ["Document 1 text...", "Document 2 text..."]
}
```

## Workflow Adherence Tracking

Ensure your agent follows the expected execution flow:

```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/flow-adherence \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": "task-002",
    "expected_flow": [
      "validate_input",
      "call_external_api",
      "process_response",
      "format_output"
    ],
    "actual_flow": [
      "validate_input",
      "call_external_api",
      "process_response",
      "format_output"
    ]
  }'
```

**Response:**
```json
{
  "adherence_score": 1.0,
  "sequence_correct": true,
  "deviations": 0,
  "missed_steps": [],
  "extra_steps": []
}
```

## What's Next?

### Learn More
- [Complete Evaluation Guide](./evaluation-testing-guide.md) - Deep dive into all features
- [API Reference](./evaluation-api-reference.md) - Complete API documentation
- [A/B Testing Best Practices](./ab-testing-guide.md) - Statistical testing tips

### Advanced Features
- **Custom Metrics**: Define domain-specific scoring functions
- **Batch Evaluation**: Evaluate multiple responses at once
- **Automated Alerts**: Get notified when scores drop
- **Cost Tracking**: Monitor evaluation costs per agent

## Troubleshooting

### "Invalid API Key" Error
- Verify your API key is correct
- Check that the key hasn't been revoked in [app.meshai.dev/settings/api-keys](https://app.meshai.dev/settings/api-keys)

### "Agent Not Found" Error
- Ensure your agent is registered: `GET https://api.meshai.dev/v1/agents`
- Check the agent_id matches exactly (case-sensitive)

### Low Scores
- Review the `feedback` field for specific issues
- Check `metrics` object for detailed breakdowns
- Compare against similar successful evaluations

## Support

- **Documentation**: [docs.meshai.dev](https://docs.meshai.dev)
- **API Status**: [status.meshai.dev](https://status.meshai.dev)
- **Community**: [community.meshai.dev](https://community.meshai.dev)
- **Email**: support@meshai.dev

---

**Next Steps**: Once you're comfortable with basic evaluations, explore [A/B testing](./ab-testing-guide.md) to systematically improve your agents!
