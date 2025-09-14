# MeshAI MCP (Model Context Protocol) Integration Guide

## Overview

MeshAI provides seamless integration with Anthropic's Model Context Protocol (MCP), enabling AI assistants like Claude to interact directly with your MeshAI agents and workflows. This integration allows you to leverage MeshAI's multi-agent orchestration capabilities directly from your AI conversations.

## What is MCP?

The Model Context Protocol (MCP) is an open standard that enables AI models to securely connect with external data sources and tools. Through MCP, AI assistants can:
- Access real-time data from various sources
- Execute actions in external systems
- Maintain context across different tools and services

## MeshAI MCP Server

The MeshAI MCP server (`mcp.meshai.dev`) provides a bridge between AI assistants and your MeshAI agents, allowing you to:

- **Execute Agent Tasks**: Run any registered MeshAI agent directly from your AI conversation
- **Orchestrate Workflows**: Trigger complex multi-agent workflows through simple commands
- **Access Real-time Data**: Query agent status, execution history, and performance metrics
- **Manage Resources**: Create, update, and monitor agents across different frameworks

## Installation

### Prerequisites

- Node.js 18+ or Python 3.8+
- MeshAI account with API access
- Claude Desktop app (for Claude integration)

### Install MeshAI MCP Server

#### Option 1: Using npm (Node.js)
```bash
npm install -g @meshai/mcp-server
```

#### Option 2: Using pip (Python)
```bash
pip install meshai-mcp-server
```

#### Option 3: Using npx (No installation)
```bash
npx @meshai/mcp-server
```

## Configuration

### For Claude Desktop

1. Open Claude Desktop settings
2. Navigate to Developer > Model Context Protocol
3. Add the MeshAI MCP server configuration:

```json
{
  "mcpServers": {
    "meshai": {
      "command": "npx",
      "args": ["@meshai/mcp-server"],
      "env": {
        "MESHAI_API_KEY": "your-api-key-here",
        "MESHAI_API_URL": "https://api.meshai.dev"
      }
    }
  }
}
```

### Environment Variables

Configure the MCP server with these environment variables:

- `MESHAI_API_KEY`: Your MeshAI API key (required)
- `MESHAI_API_URL`: MeshAI API endpoint (default: `https://api.meshai.dev`)
- `MESHAI_RUNTIME_URL`: Runtime endpoint (default: `https://runtime.meshai.dev`)
- `MCP_SERVER_PORT`: Server port for standalone mode (default: `3000`)

## Usage Examples

### Basic Agent Execution

Once configured, you can interact with MeshAI agents directly in your conversation:

```
"Execute my data analysis agent to process the latest sales report"
```

The MCP server will:
1. Identify the appropriate agent
2. Execute the task
3. Return results directly in the conversation

### Multi-Agent Workflows

Trigger complex workflows involving multiple agents:

```
"Run my content pipeline: research trending topics, generate article drafts, and optimize for SEO"
```

### Query Agent Status

Check the status of your agents and executions:

```
"Show me the status of all my active agents"
"What's the execution history for my customer service agent?"
```

## Available MCP Tools

The MeshAI MCP server exposes the following tools:

### `mesh_execute`
Execute a task using MeshAI's multi-agent orchestration.

**Parameters:**
- `task` (required): Task description
- `capabilities` (optional): Array of required capabilities
- `framework` (optional): Preferred framework (langchain, crewai, autogen)
- `timeout` (optional): Execution timeout in seconds

**Example:**
```javascript
{
  "task": "Analyze customer feedback and generate insights report",
  "capabilities": ["sentiment-analysis", "report-generation"],
  "framework": "langchain",
  "timeout": 300
}
```

### `mesh_list_agents`
List all available agents in your MeshAI workspace.

**Parameters:**
- `status` (optional): Filter by status (active, inactive, all)
- `framework` (optional): Filter by framework

### `mesh_get_agent`
Get detailed information about a specific agent.

**Parameters:**
- `agent_id` (required): The agent's unique identifier

### `mesh_create_workflow`
Create a new multi-agent workflow.

**Parameters:**
- `name` (required): Workflow name
- `description` (required): Workflow description
- `agents` (required): Array of agent IDs
- `steps` (required): Workflow steps configuration

## Advanced Features

### Custom Agent Registration

Register custom agents through MCP:

```python
# Example: Registering a custom agent via MCP
{
  "tool": "mesh_register_agent",
  "parameters": {
    "name": "Custom Data Processor",
    "framework": "langchain",
    "capabilities": ["data-processing", "validation"],
    "endpoint": "https://my-agent.example.com/process",
    "authentication": {
      "type": "api_key",
      "key": "agent-specific-key"
    }
  }
}
```

### Streaming Responses

For long-running tasks, the MCP server supports streaming responses:

```javascript
{
  "tool": "mesh_execute_stream",
  "parameters": {
    "task": "Generate comprehensive market analysis",
    "stream": true
  }
}
```

### Context Preservation

The MCP server maintains context across interactions:

```javascript
{
  "tool": "mesh_continue_task",
  "parameters": {
    "task_id": "previous-task-id",
    "additional_instructions": "Focus on European markets"
  }
}
```

## Security & Authentication

### API Key Management

1. Generate API keys from the MeshAI dashboard
2. Store keys securely using environment variables
3. Never commit API keys to version control

### Rate Limiting

The MCP server implements rate limiting to prevent abuse:
- Default: 100 requests per minute
- Burst: 500 requests per hour
- Enterprise plans have higher limits

### Data Privacy

- All communications are encrypted using TLS 1.3
- Data is processed in compliance with GDPR and SOC2
- No conversation data is stored without explicit consent

## Troubleshooting

### Common Issues

#### Connection Failed
```
Error: Failed to connect to MeshAI MCP server
```
**Solution**: Verify your API key and network connectivity

#### Agent Not Found
```
Error: No suitable agent found for task
```
**Solution**: Ensure agents are registered and active in your MeshAI workspace

#### Timeout Errors
```
Error: Task execution timeout
```
**Solution**: Increase timeout parameter or optimize agent performance

### Debug Mode

Enable debug logging for troubleshooting:

```bash
MESHAI_DEBUG=true npx @meshai/mcp-server
```

### Health Check

Verify server health:

```bash
curl https://mcp.meshai.dev/health
```

## Best Practices

1. **Agent Specialization**: Create focused agents for specific tasks
2. **Capability Mapping**: Clearly define agent capabilities for optimal routing
3. **Error Handling**: Implement robust error handling in custom agents
4. **Monitoring**: Use MeshAI dashboard to monitor agent performance
5. **Version Control**: Version your agent configurations and workflows

## API Reference

### REST Endpoints

The MCP server also exposes REST endpoints for programmatic access:

```
GET  /api/v1/agents          - List agents
POST /api/v1/execute         - Execute task
GET  /api/v1/tasks/{id}      - Get task status
POST /api/v1/workflows       - Create workflow
GET  /api/v1/health          - Health check
```

### WebSocket Support

For real-time updates:

```javascript
const ws = new WebSocket('wss://mcp.meshai.dev/ws');
ws.on('message', (data) => {
  const update = JSON.parse(data);
  console.log('Task update:', update);
});
```

## Integration Examples

### Python Integration
```python
from meshai_mcp import MCPClient

client = MCPClient(api_key="your-api-key")
result = client.execute(
    task="Analyze sentiment of customer reviews",
    capabilities=["sentiment-analysis"]
)
print(result)
```

### Node.js Integration
```javascript
const { MCPClient } = require('@meshai/mcp-client');

const client = new MCPClient({
  apiKey: process.env.MESHAI_API_KEY
});

const result = await client.execute({
  task: 'Generate product descriptions',
  capabilities: ['content-generation']
});
```

## Support & Resources

- **Documentation**: https://docs.meshai.dev/mcp
- **GitHub**: https://github.com/meshailabs/mcp-server
- **Discord Community**: https://discord.gg/meshai
- **Support Email**: support@meshai.dev

## Changelog

### Version 1.0.0 (2024-01-15)
- Initial release with core MCP functionality
- Support for agent execution and workflow orchestration
- Claude Desktop integration

### Version 1.1.0 (Coming Soon)
- Streaming response support
- Enhanced context preservation
- Custom agent registration via MCP
- Performance improvements

---

*For the latest updates and additional resources, visit [docs.meshai.dev](https://docs.meshai.dev)*