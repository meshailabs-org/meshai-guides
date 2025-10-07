# Evaluation Service API Reference

Complete API reference for MeshAI's Evaluation & Testing service.

**Base URL**: `https://evaluations.meshai.dev`

**Authentication**: All requests require `X-API-Key` header

```bash
X-API-Key: your_api_key_here
```

---

## Table of Contents

1. [Evaluations](#evaluations)
2. [A/B Testing](#ab-testing)
3. [Flow Tracking](#flow-tracking)
4. [Custom Metrics](#custom-metrics)
5. [Dashboard Data](#dashboard-data)

---

## Evaluations

### Run Evaluation

Evaluate an agent response against quality criteria.

**Endpoint**: `POST /v1/evaluation/run-eval`

**Request Body**:
```json
{
  "agent_id": "string (required)",
  "task_id": "string (required)",
  "prompt": "string (required)",
  "response": "string (required)",
  "expected_output": "string (optional)",
  "template": "accuracy|relevance|coherence|hallucination|custom",
  "context": {
    "grounding_docs": ["string"],
    "domain": "string",
    "custom_field": "any"
  }
}
```

**Response**:
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
  "feedback": "Detailed feedback text",
  "timestamp": "2025-10-07T18:30:00Z"
}
```

**Example**:
```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/run-eval \
  -H "X-API-Key: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "my-agent",
    "task_id": "task-001",
    "prompt": "What is machine learning?",
    "response": "Machine learning is a subset of artificial intelligence...",
    "template": "accuracy"
  }'
```

---

### Get Evaluation Results

Retrieve past evaluation results for an agent.

**Endpoint**: `GET /v1/evaluation/results/{agent_id}`

**Query Parameters**:
- `limit` (integer, optional): Number of results (default: 50, max: 500)
- `offset` (integer, optional): Pagination offset (default: 0)
- `min_score` (float, optional): Filter by minimum score
- `start_date` (ISO 8601, optional): Filter by start date
- `end_date` (ISO 8601, optional): Filter by end date

**Response**:
```json
{
  "agent_id": "my-agent",
  "total_evals": 152,
  "avg_score": 0.87,
  "pass_rate": 0.91,
  "results": [
    {
      "eval_id": "eval_abc123",
      "task_id": "task-001",
      "score": 0.95,
      "passed": true,
      "template": "accuracy",
      "timestamp": "2025-10-07T18:30:00Z"
    }
  ]
}
```

**Example**:
```bash
curl "https://evaluations.meshai.dev/v1/evaluation/results/my-agent?limit=100&min_score=0.8" \
  -H "X-API-Key: ${API_KEY}"
```

---

### Batch Evaluation

Evaluate multiple responses in one request (up to 50).

**Endpoint**: `POST /v1/evaluation/batch`

**Request Body**:
```json
{
  "evaluations": [
    {
      "agent_id": "agent-1",
      "task_id": "task-001",
      "prompt": "Question 1",
      "response": "Answer 1",
      "template": "accuracy"
    },
    {
      "agent_id": "agent-1",
      "task_id": "task-002",
      "prompt": "Question 2",
      "response": "Answer 2",
      "template": "accuracy"
    }
  ]
}
```

**Response**:
```json
{
  "total": 2,
  "successful": 2,
  "failed": 0,
  "results": [
    {
      "eval_id": "eval_abc123",
      "task_id": "task-001",
      "score": 0.95,
      "passed": true
    },
    {
      "eval_id": "eval_xyz789",
      "task_id": "task-002",
      "score": 0.88,
      "passed": true
    }
  ]
}
```

---

## A/B Testing

### Create Experiment

Create a new A/B test experiment to compare two agents.

**Endpoint**: `POST /v1/evaluation/experiments/create`

**Request Body**:
```json
{
  "name": "string (required)",
  "description": "string (optional)",
  "variant_a": "string (required, agent_id)",
  "variant_b": "string (required, agent_id)",
  "traffic_split": 0.5,
  "min_samples": 100,
  "confidence_level": 0.95,
  "metrics": ["accuracy", "latency", "cost"]
}
```

**Response**:
```json
{
  "experiment_id": "exp_xyz789",
  "status": "active",
  "created_at": "2025-10-07T18:30:00Z",
  "variant_a": "agent-gpt4",
  "variant_b": "agent-claude",
  "traffic_split": 0.5
}
```

**Example**:
```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/experiments/create \
  -H "X-API-Key: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Model Comparison",
    "variant_a": "agent-gpt4",
    "variant_b": "agent-claude",
    "traffic_split": 0.5,
    "min_samples": 100,
    "metrics": ["accuracy"]
  }'
```

---

### List Experiments

Get all active and completed experiments.

**Endpoint**: `GET /v1/evaluation/experiments/list`

**Query Parameters**:
- `status` (string, optional): Filter by status (active|completed|archived)
- `limit` (integer, optional): Number of results (default: 50)

**Response**:
```json
{
  "total": 15,
  "experiments": [
    {
      "experiment_id": "exp_xyz789",
      "name": "Model Comparison",
      "status": "active",
      "variant_a": "agent-gpt4",
      "variant_b": "agent-claude",
      "sample_count": 45,
      "created_at": "2025-10-07T18:30:00Z"
    }
  ]
}
```

---

### Get Experiment Results

Retrieve results and statistical analysis for an experiment.

**Endpoint**: `GET /v1/evaluation/experiments/{experiment_id}/results`

**Response**:
```json
{
  "experiment_id": "exp_xyz789",
  "name": "Model Comparison",
  "status": "completed",
  "winner": "agent-claude",
  "confidence": 0.95,
  "statistical_significance": true,
  "variant_a": {
    "agent_id": "agent-gpt4",
    "sample_count": 152,
    "metrics": {
      "accuracy": {
        "mean": 0.87,
        "std_dev": 0.08,
        "min": 0.65,
        "max": 0.98
      },
      "latency": {
        "mean": 1.2,
        "std_dev": 0.3
      }
    }
  },
  "variant_b": {
    "agent_id": "agent-claude",
    "sample_count": 148,
    "metrics": {
      "accuracy": {
        "mean": 0.91,
        "std_dev": 0.06,
        "min": 0.72,
        "max": 0.99
      },
      "latency": {
        "mean": 1.1,
        "std_dev": 0.25
      }
    }
  },
  "p_value": 0.023,
  "effect_size": 0.04,
  "recommendation": "Deploy variant_b (agent-claude)"
}
```

---

### Assign Task to Variant

Get the agent assignment for a task (handles traffic splitting).

**Endpoint**: `POST /v1/evaluation/experiments/{experiment_id}/assign`

**Request Body**:
```json
{
  "task_id": "string (required)"
}
```

**Response**:
```json
{
  "task_id": "task-123",
  "experiment_id": "exp_xyz789",
  "assigned_variant": "variant_a",
  "agent_id": "agent-gpt4"
}
```

---

### Stop Experiment

Stop a running experiment and get final results.

**Endpoint**: `POST /v1/evaluation/experiments/{experiment_id}/stop`

**Response**:
```json
{
  "experiment_id": "exp_xyz789",
  "status": "completed",
  "stopped_at": "2025-10-07T20:00:00Z",
  "final_sample_count": 200
}
```

---

## Flow Tracking

### Evaluate Flow Adherence

Check if an agent followed the expected execution flow.

**Endpoint**: `POST /v1/evaluation/flow-adherence`

**Request Body**:
```json
{
  "task_id": "string (required)",
  "expected_flow": ["step1", "step2", "step3"],
  "actual_flow": ["step1", "step2", "step3"],
  "metadata": {
    "agent_id": "string",
    "custom_field": "any"
  }
}
```

**Response**:
```json
{
  "success": true,
  "task_id": "task-123",
  "adherence_score": 1.0,
  "sequence_correct": true,
  "deviations": 0,
  "missed_steps": [],
  "extra_steps": [],
  "violations": []
}
```

**Example**:
```bash
curl -X POST https://evaluations.meshai.dev/v1/evaluation/flow-adherence \
  -H "X-API-Key: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": "task-123",
    "expected_flow": ["validate", "process", "respond"],
    "actual_flow": ["validate", "process", "respond"]
  }'
```

---

### Get Flow Metrics

Get flow adherence metrics for an agent over time.

**Endpoint**: `GET /v1/evaluation/flow-metrics/{agent_id}`

**Query Parameters**:
- `days` (integer, optional): Time window in days (default: 7)

**Response**:
```json
{
  "agent_id": "my-agent",
  "period_days": 7,
  "total_tasks": 523,
  "avg_adherence_score": 0.95,
  "violation_rate": 0.05,
  "common_deviations": [
    {
      "deviation": "skipped_validation",
      "count": 12,
      "percentage": 0.023
    }
  ]
}
```

---

## Custom Metrics

### Register Custom Metric

Define a custom evaluation metric with your own scoring logic.

**Endpoint**: `POST /v1/evaluation/metrics/register`

**Request Body**:
```json
{
  "metric_id": "business_value",
  "name": "Business Value Score",
  "description": "Measures business impact of response",
  "scoring_function": "def score(response, context):\n    # Your Python code\n    return {'score': 0.9, 'passed': True}",
  "weight": 2.0,
  "threshold": 0.7
}
```

**Response**:
```json
{
  "metric_id": "business_value",
  "status": "registered",
  "created_at": "2025-10-07T18:30:00Z"
}
```

---

### List Custom Metrics

Get all registered custom metrics.

**Endpoint**: `GET /v1/evaluation/metrics/list`

**Response**:
```json
{
  "total": 5,
  "metrics": [
    {
      "metric_id": "business_value",
      "name": "Business Value Score",
      "weight": 2.0,
      "threshold": 0.7,
      "created_at": "2025-10-07T18:30:00Z"
    }
  ]
}
```

---

### Evaluate with Custom Metrics

Run evaluation using custom metrics.

**Endpoint**: `POST /v1/evaluation/custom-eval`

**Request Body**:
```json
{
  "agent_id": "my-agent",
  "task_id": "task-001",
  "metrics": ["business_value", "customer_satisfaction"],
  "data": {
    "response": "Agent response text",
    "customer_tier": "enterprise",
    "custom_context": "any"
  }
}
```

**Response**:
```json
{
  "eval_id": "eval_custom_123",
  "scores": {
    "business_value": 0.92,
    "customer_satisfaction": 0.88
  },
  "overall_score": 0.90,
  "passed": true
}
```

---

## Dashboard Data

### Get Summary Statistics

Get aggregate statistics for the dashboard.

**Endpoint**: `GET /v1/evaluation/dashboard/summary`

**Query Parameters**:
- `agent_id` (string, optional): Filter by agent
- `days` (integer, optional): Time window (default: 7)

**Response**:
```json
{
  "period_days": 7,
  "total_evaluations": 1523,
  "avg_score": 0.89,
  "pass_rate": 0.92,
  "trend": "improving",
  "top_agents": [
    {
      "agent_id": "agent-claude",
      "avg_score": 0.93,
      "eval_count": 456
    }
  ],
  "score_distribution": {
    "0.9-1.0": 678,
    "0.8-0.9": 512,
    "0.7-0.8": 256,
    "below_0.7": 77
  }
}
```

---

## Error Responses

All endpoints return standard error responses:

**400 Bad Request**:
```json
{
  "error": "Invalid request",
  "detail": "Missing required field: agent_id"
}
```

**401 Unauthorized**:
```json
{
  "error": "Authentication failed",
  "detail": "Invalid or missing API key"
}
```

**404 Not Found**:
```json
{
  "error": "Resource not found",
  "detail": "Agent 'my-agent' does not exist"
}
```

**429 Too Many Requests**:
```json
{
  "error": "Rate limit exceeded",
  "detail": "Maximum 1000 requests per minute",
  "retry_after": 42
}
```

**500 Internal Server Error**:
```json
{
  "error": "Internal server error",
  "detail": "Evaluation service temporarily unavailable"
}
```

---

## Rate Limits

- **Evaluation requests**: 1000 per minute per account
- **Batch evaluations**: 100 per minute per account
- **Experiment operations**: 100 per minute per account

Contact support@meshai.dev to request higher limits.

---

## SDKs

### Python

```bash
pip install meshai-sdk
```

```python
from meshai import MeshAIClient

client = MeshAIClient(api_key="YOUR_API_KEY")
result = client.evaluations.run(...)
```

### Node.js

```bash
npm install @meshai/sdk
```

```javascript
const { MeshAIClient } = require('@meshai/sdk');

const client = new MeshAIClient({ apiKey: 'YOUR_API_KEY' });
const result = await client.evaluations.run({...});
```

### cURL

All examples in this documentation use cURL and can be run from any terminal.

---

## Support

- **API Status**: [status.meshai.dev](https://status.meshai.dev)
- **Documentation**: [docs.meshai.dev](https://docs.meshai.dev)
- **Support Email**: support@meshai.dev
