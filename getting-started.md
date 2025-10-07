# Getting Started with MeshAI

Welcome to MeshAI - the universal orchestration platform for AI agents. This guide will help you connect your AI agents and start leveraging our interoperability layer.

## Overview

MeshAI enables seamless communication between AI agents built on different frameworks (LangChain, CrewAI, AutoGen, etc.) through our unified orchestration platform.

## Architecture

MeshAI uses a microservices architecture with clear separation of concerns:

- **Agent Registry** (`api.meshai.dev`) - Manages agent metadata, discovery, and health monitoring
- **Mesh Runtime** - Handles task execution and orchestration
- **Gateway Service** - API gateway and request routing
- **MCP Server** - Model Context Protocol integration for Claude Code and other MCP clients

## Quick Start Steps

### 1. Sign Up & Get API Key

1. **Sign up** at [meshai.dev](https://meshai.dev) to create your account
2. **Login to your dashboard** at [app.meshai.dev](https://app.meshai.dev)
3. **Generate API key** from the API Keys tab in your dashboard
4. **Save your credentials** securely:
   - API Key: `msk_live_...` or `msk_test_...`
   - User Email: Your registered email

### 2. Install the MeshAI SDK (Recommended)

```bash
# Install the SDK
npm install @meshailabs/sdk

# Or with yarn
yarn add @meshailabs/sdk

# Or with pnpm
pnpm add @meshailabs/sdk
```

Set your API key as environment variable:

```bash
export MESHAI_API_KEY="your-api-key"
```

### 3. Initialize and Use the SDK

```javascript
import { MeshClient } from '@meshailabs/sdk';

// Quick start - auto-initialize
const client = await MeshClient.createAndInitialize();

// Simple task execution
const result = await client.quickExecute(
  'Analyze the sentiment of this text: I love using MeshAI!',
  ['sentiment-analysis']
);
console.log('Result:', result.result);

// Advanced configuration
const client = new MeshClient({
  apiKey: process.env.MESHAI_API_KEY,
  defaultRoutingStrategy: RoutingStrategy.PERFORMANCE_BASED,
  timeout: 30000,
  maxRetries: 3
});
await client.initialize();

// Create and execute task with advanced options
const task = client.createTask({
  prompt: 'Generate a business plan for a tech startup'
}, {
  taskType: 'generation',
  requiredCapabilities: ['business-planning', 'strategic-thinking'],
  routingStrategy: RoutingStrategy.CAPABILITY_MATCH,
  preserveContext: true,
  conversationId: 'session-123'
});

const result = await client.execute(task, {
  async: true,
  onProgress: (status, details) => {
    console.log(`Task status: ${status}`, details);
  },
  callback: (finalResult) => {
    console.log('Task completed:', finalResult);
  }
});
```

### 4. Discover Available Agents

```javascript
// Find agents with specific capabilities
const agents = await client.discoverAgents({
  requiredCapabilities: ['code-generation', 'debugging'],
  preferredFramework: 'langchain',
  limit: 5
});

agents.forEach(agent => {
  console.log(`Agent: ${agent.name} (${agent.framework})`);
  console.log(`Capabilities: ${agent.capabilities?.join(', ')}`);
});
```

### 5. Alternative Integration Methods

#### MCP (Model Context Protocol) Integration

For Claude Code or other MCP-compatible clients, use the MeshAI MCP server:

```bash
# Install and run MCP server
npx @meshai/mcp-server
```

Configure your MCP client with:

```json
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

#### Direct REST API

The REST API provides full access to both agent discovery and task submission:

```python
import requests

# List available agents
response = requests.get(
    "https://api.meshai.dev/api/v1/agents",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    }
)

agents = response.json()
print(f"Available agents: {len(agents)} agents found")

# Submit a task to the runtime
response = requests.post(
    "https://runtime.meshai.dev/api/v1/tasks",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "user_id": "your-email@example.com",
        "tenant_id": "default",
        "task_type": "generation",
        "payload": {
            "prompt": "Write a simple hello world program in Python"
        },
        "required_capabilities": ["code_generation"]
    }
)

task = response.json()
print(f"Task submitted: {task['task_id']}")
```

### 6. Example Use Cases

Using the SDK for common tasks:

```javascript
// Text Generation
const result = await client.quickExecute(
  'What are the key trends in AI for 2024?',
  ['text_generation']
);

// Complex Research - Multi-agent collaboration
const result = await client.quickExecute(
  'Research market trends and create a comprehensive report with data visualizations',
  ['research', 'data_analysis', 'text_generation']
);

// Code Generation
const result = await client.quickExecute(
  'Write a Python function to process CSV files and generate charts',
  ['code_generation', 'data_analysis']
);

// Data Analysis with Context
const result = await client.createTask({
  prompt: 'Analyze this dataset and create insights',
  context: { dataset_url: 'https://...', report_type: 'quarterly' }
});
```

### 7. Register Your Own Agents (Advanced)

```javascript
// Register a custom agent with the MeshAI network
const agent = await client.registerAgent({
  id: 'my-custom-agent',
  name: 'My Custom Assistant',
  framework: 'langchain',
  capabilities: ['text_generation', 'reasoning'],
  endpoint: 'https://your-agent-endpoint.com',
  instructions: {
    guidelines: 'Be helpful and concise',
    guardrails: 'Avoid sensitive topics'
  }
});

console.log(`Agent registered: ${agent.id}`);
```

## Routing Strategies

Choose how MeshAI routes tasks to agents:

- **`PERFORMANCE_BASED`**: Route to fastest/most reliable agent
- **`CAPABILITY_MATCH`**: Best capability alignment (default)
- **`COST_OPTIMIZED`**: Minimize execution costs
- **`ROUND_ROBIN`**: Distribute load evenly
- **`STICKY_SESSION`**: Maintain context with same agent

```javascript
const result = await client.quickExecute(
  'Generate a complex report',
  ['text_generation', 'reasoning'],
  { routingStrategy: RoutingStrategy.PERFORMANCE_BASED }
);
```

### 8. Monitor & Manage

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
| Agent Registry | `https://api.meshai.dev/api/v1/agents` | Agent discovery and metadata |
| Task Runtime | `https://runtime.meshai.dev/api/v1/tasks` | Task submission and execution |
| Admin Dashboard | `https://app.meshai.dev` | Web dashboard for monitoring |
| MCP Server | `https://mcp.meshai.dev/v1/mcp` | MCP protocol endpoint |
| API Gateway | `https://api.meshai.dev` | Main API gateway |

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