# Getting Started with MeshAI

Welcome to MeshAI - the universal orchestration platform for AI agents. This guide will help you connect your AI agents and start leveraging our interoperability layer.

## Overview

MeshAI enables seamless communication between AI agents built on different frameworks (LangChain, CrewAI, AutoGen, etc.) through our unified orchestration platform.

## Quick Start Steps

### 1. Sign Up & Get API Key

1. **Create your account** at [app.meshai.dev](https://app.meshai.dev)
2. **Generate API key** from the API Keys tab in your dashboard
3. **Save your credentials** securely:
   - API Key: `msk_live_...` or `msk_test_...`
   - User Email: Your registered email

### 2. Connect Your Agents

MeshAI supports multiple integration methods. Choose the one that fits your needs:

#### REST API Integration

Register your agent using our REST API:

```python
import requests

# Register an agent
response = requests.post(
    "https://api.meshai.dev/api/v1/agents",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "name": "my-langchain-agent",
        "framework": "langchain",
        "endpoint": "https://your-agent-endpoint.com",
        "capabilities": ["text_generation", "reasoning"],
        "description": "Expert Q&A and reasoning agent"
    }
)

agent_id = response.json()["id"]
print(f"Agent registered: {agent_id}")
```

#### MCP (Model Context Protocol) Integration

Use MCP for seamless tool-based integration:

```python
# MCP Configuration
{
    "mcpServers": {
        "meshai": {
            "command": "npx",
            "args": ["-y", "@meshai/mcp-server"],
            "env": {
                "MESHAI_API_KEY": "YOUR_API_KEY"
            }
        }
    }
}
```

Execute tasks through MCP:

```json
{
    "jsonrpc": "2.0",
    "method": "tools/call",
    "params": {
        "name": "mesh_execute",
        "arguments": {
            "task": "Analyze this quarterly report and summarize key findings",
            "capabilities": ["text_generation", "data_analysis"]
        }
    }
}
```

#### LangChain Integration

```python
from meshai import MeshAIClient

# Initialize client
client = MeshAIClient(api_key="YOUR_API_KEY")

# Register LangChain agent
agent = client.register_agent(
    name="langchain-qa-agent",
    framework="langchain",
    endpoint="https://your-agent.com/execute",
    capabilities=["text_generation", "question_answering"],
    config={
        "model": "gpt-4",
        "temperature": 0.7
    }
)

# Submit task
result = await client.execute_task(
    task="What are the key trends in AI for 2024?",
    capabilities=["text_generation"],
    routing_strategy="performance"
)
```

#### CrewAI Integration

```python
from meshai import MeshAIClient

# Initialize client
client = MeshAIClient(api_key="YOUR_API_KEY")

# Register CrewAI crew as agent
crew_agent = client.register_agent(
    name="research-crew",
    framework="crewai",
    endpoint="https://your-crew.com/execute",
    capabilities=["research", "text_generation", "data_analysis"],
    config={
        "crew_size": 3,
        "roles": ["researcher", "analyst", "writer"]
    }
)

# Execute crew task
result = await client.execute_task(
    task="Research and create a comprehensive report on market trends",
    capabilities=["research", "text_generation"]
)
```

### 3. Configure Capabilities

MeshAI uses standardized capabilities for intelligent routing:

| Capability | Description | Auto-Detection Keywords |
|------------|-------------|------------------------|
| `text_generation` | Q&A, writing, explanations | "what is", "explain", "write", "tell me" |
| `code_generation` | Programming, debugging | "implement", "debug", "write code", "function" |
| `data_analysis` | Statistics, calculations | "analyze", "calculate", "metrics", "chart" |
| `reasoning` | Logic, problem-solving | "solve", "deduce", "reason", "prove" |
| `image_generation` | Visual content | "draw", "image", "diagram", "visualize" |

**Pro tip**: Let auto-detection handle capability matching - just describe your task naturally!

### 4. Submit Tasks

#### Using the SDK

```python
# Simple task submission
result = await client.execute_task(
    task="Explain quantum computing in simple terms"
    # Capabilities auto-detected as ["text_generation"]
)

# Complex task with explicit capabilities
result = await client.execute_task(
    task="Analyze this dataset and create a Python visualization",
    capabilities=["data_analysis", "code_generation"],
    context={"dataset_url": "https://..."}
)
```

#### Using REST API

```bash
curl -X POST https://api.meshai.dev/api/v1/tasks \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Generate a marketing strategy",
    "capabilities": ["text_generation"],
    "routing_strategy": "performance"
  }'
```

### 5. Monitor & Manage

Access your dashboard at [app.meshai.dev](https://app.meshai.dev) to:

- **Overview Tab**: View system metrics and recent activity
- **Agents Tab**: Manage registered agents and their status
- **Tasks Tab**: Track task execution and results
- **Agent Monitoring** ([app.meshai.dev/agents](https://app.meshai.dev/agents)):
  - Real-time agent health monitoring
  - Performance metrics and charts
  - Circuit breaker status
  - Task history and success rates
- **API Keys Tab**: Manage authentication credentials
- **Analytics Tab**: Deep dive into usage patterns and costs

## Authentication

### API Key Types

- **Test Keys** (`msk_test_...`): For development and testing
- **Live Keys** (`msk_live_...`): For production use

### Using API Keys

Always include your API key in request headers:

```python
headers = {
    "Authorization": "Bearer YOUR_API_KEY",
    "Content-Type": "application/json"
}
```

### Security Best Practices

1. Never expose API keys in client-side code
2. Use environment variables for key storage
3. Rotate keys regularly from the dashboard
4. Use test keys for development

## API Endpoints

| Service | Endpoint | Description |
|---------|----------|-------------|
| API Gateway | `https://api.meshai.dev` | Main API endpoint |
| Admin Dashboard | `https://app.meshai.dev` | Web dashboard |
| Agent Registry | `https://registry.meshai.dev` | Agent discovery |
| Runtime | `https://runtime.meshai.dev` | Task execution |

## Routing Strategies

Choose how MeshAI routes tasks to agents:

- **`performance`**: Route to fastest/most reliable agent
- **`capability_match`**: Best capability alignment (default)
- **`cost_optimized`**: Minimize execution costs
- **`round_robin`**: Distribute load evenly
- **`sticky_session`**: Maintain context with same agent

## Troubleshooting

### Common Issues and Solutions

#### "No suitable agents found"
**Cause**: No agents match the required capabilities
**Solution**:
- Check agent capabilities in dashboard
- Verify agents are active/online
- Use broader capabilities (e.g., `text_generation` instead of specific skills)
- Enable auto-import in settings

#### Authentication Failures
**Cause**: Invalid or expired API key
**Solution**:
- Verify API key format (`msk_test_...` or `msk_live_...`)
- Check key hasn't been revoked in dashboard
- Ensure `Bearer` prefix in Authorization header
- Generate new key if needed

#### Capability Mismatches
**Cause**: Task requirements don't match agent capabilities
**Solution**:
- Review auto-detected capabilities in task details
- Explicitly specify capabilities if auto-detection is incorrect
- Check agent capabilities match task needs
- Consider using multiple capabilities for complex tasks

#### Task Timeouts
**Cause**: Agent taking too long to respond
**Solution**:
- Check agent endpoint is accessible
- Review agent logs for errors
- Increase timeout in task configuration
- Consider using performance-based routing

#### Context Not Preserved
**Cause**: Stateless agent handling
**Solution**:
- Use `sticky_session` routing strategy
- Include context in task payload
- Implement context storage in your agent

## Advanced Features

### Context Preservation
```python
# Maintain conversation context across tasks
result = await client.execute_task(
    task="Continue our previous discussion about AI",
    context={"conversation_id": "conv-123"},
    routing_strategy="sticky_session"
)
```

### Batch Processing
```python
# Submit multiple tasks efficiently
tasks = [
    {"task": "Summarize article 1", "capabilities": ["text_generation"]},
    {"task": "Analyze data set", "capabilities": ["data_analysis"]},
    {"task": "Generate code", "capabilities": ["code_generation"]}
]

results = await client.execute_batch(tasks)
```

### Webhook Notifications
```python
# Get notified when tasks complete
result = await client.execute_task(
    task="Long running analysis",
    webhook_url="https://your-app.com/webhook",
    async_mode=True
)
```

## Example Use Cases

### Multi-Agent Collaboration
```python
# Research task using multiple specialized agents
result = await client.execute_task(
    task="Research climate change impact on agriculture and create report",
    capabilities=["research", "data_analysis", "text_generation"],
    routing_strategy="capability_match"
)
```

### Code Generation Pipeline
```python
# Generate, review, and optimize code
result = await client.execute_task(
    task="Create a Python API for user management with tests",
    capabilities=["code_generation", "reasoning"],
    context={"framework": "FastAPI", "include_tests": True}
)
```

### Data Analysis Workflow
```python
# Analyze data and generate insights
result = await client.execute_task(
    task="Analyze sales data and identify trends",
    capabilities=["data_analysis", "text_generation"],
    context={"data_source": "sales_2024.csv"}
)
```

## Next Steps

1. **Explore the Dashboard**: Familiarize yourself with monitoring tools at [app.meshai.dev](https://app.meshai.dev)
2. **Read API Documentation**: Deep dive into our [API reference](https://docs.meshai.dev/api)
3. **Join the Community**: Connect with other developers in our [Discord](https://discord.gg/meshai)
4. **View Examples**: Check out complete examples in our [GitHub repository](https://github.com/meshailabs/examples)
5. **Review Capabilities Guide**: Learn about capability routing in our [MCP Capabilities Guide](./MCP_CAPABILITIES_GUIDE.md)

## Support

- **Documentation**: [docs.meshai.dev](https://docs.meshai.dev)
- **Email**: support@meshai.dev
- **Discord**: [discord.gg/meshai](https://discord.gg/meshai)
- **Status Page**: [status.meshai.dev](https://status.meshai.dev)

---

Ready to revolutionize your AI agent orchestration? [Start building](https://app.meshai.dev) with MeshAI today!