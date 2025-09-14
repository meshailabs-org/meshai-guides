# MeshAI MCP (Model Context Protocol) Integration Guide

## Overview

MeshAI provides seamless integration with Anthropic's Model Context Protocol (MCP), enabling AI assistants like Claude to interact directly with your MeshAI agents and workflows. This integration allows you to leverage MeshAI's multi-agent orchestration capabilities directly from your AI conversations.

## What is MCP?

The Model Context Protocol (MCP) is an open standard that enables AI models to securely connect with external data sources and tools. Through MCP, AI assistants can:
- Access real-time data from various sources
- Execute actions in external systems
- Maintain context across different tools and services

## MeshAI MCP Server

The MeshAI MCP server (`https://mcp.meshai.dev/v1/mcp`) provides a Model Context Protocol interface that enables AI assistants like Claude Code, Google Gemini CLI, and OpenAI's ChatGPT to interact with your MeshAI infrastructure. This integration allows you to:

- **Execute Agent Tasks**: Run any registered MeshAI agent directly from your AI conversation
- **Orchestrate Workflows**: Trigger complex multi-agent workflows through simple natural language commands
- **Monitor Operations**: Check task status, view results, and track system performance
- **Manage Resources**: List agents, query capabilities, and control task execution
- **Batch Processing**: Execute multiple tasks in parallel for improved efficiency

The MCP server acts as a secure gateway to your MeshAI deployment, requiring API key authentication to ensure only authorized access to your agents and workflows.

## Getting Started

### Prerequisites

- MeshAI account with API access
- API key from the MeshAI dashboard
- Compatible AI assistant (Claude Code, Google Gemini CLI, or OpenAI ChatGPT)

### MeshAI MCP Server Access

The MeshAI MCP server is deployed and available at `https://mcp.meshai.dev`. You don't need to install any packages - simply configure your AI assistant to connect to our hosted service.

### Obtaining Your API Key

1. Log in to the [MeshAI Admin Dashboard](https://app.meshai.dev)
2. Navigate to the API Keys section
3. Create a new API key or use an existing one
4. Copy the key for configuration

## Configuration

### Direct API Access

The MeshAI MCP server exposes a REST API at `https://mcp.meshai.dev`. You can interact with it directly using HTTP requests:

```bash
curl -X POST https://mcp.meshai.dev/api/v1/execute \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Analyze customer feedback",
    "capabilities": ["sentiment-analysis"]
  }'
```

### For Claude Code

You can connect Claude Code to the MeshAI MCP server using the following command:

```bash
claude mcp add meshai-mcp https://mcp.meshai.dev/v1/mcp -t http -H "Authorization: Bearer YOUR_API_KEY"
```

Replace `YOUR_API_KEY` with your actual API key from the MeshAI dashboard.

### For Google Gemini CLI

Gemini CLI supports MCP servers through its `settings.json` configuration. You can add the MeshAI MCP server in two ways:

#### Option 1: Using CLI Command
```bash
gemini mcp add meshai-mcp
```

Then configure it with:
- URL: `https://mcp.meshai.dev/v1/mcp`
- Transport: HTTP
- Headers: Authorization Bearer token

#### Option 2: Manual Configuration
Edit your `settings.json` file (located in your project or user config directory):

```json
{
  "mcpServers": {
    "meshai": {
      "httpUrl": "https://mcp.meshai.dev/v1/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      },
      "timeout": 30000,
      "trust": true
    }
  }
}
```

**Note**: Gemini CLI currently supports HTTP transport. SSE transport support for real-time updates is coming soon.

### For OpenAI ChatGPT

OpenAI fully adopted MCP in 2025 and supports it across their platform including ChatGPT desktop app, Agents SDK, and Responses API.

#### ChatGPT Pro/Business/Enterprise Custom Connectors

If you have ChatGPT Pro, Business, Enterprise, or Edu access, you can add MeshAI as a custom connector:

1. **Access Custom Connectors**:
   - Click on your name at the bottom left of ChatGPT
   - Select "Settings"
   - Navigate to "Connectors"
   - If you don't see a "Create" button at the top:
     - Go to "Advanced Settings"
     - Enable "Developer Mode"
     - Return to the Connectors section

2. **Create Custom Connector**:
   - Click the "Create" button at the top of the Connectors page

3. **Configure MeshAI MCP Server**:
   - **Name**: MeshAI Agent Platform
   - **URL**: `https://mcp.meshai.dev/v1/mcp`
   - **Transport**: HTTP
   - **Authentication**: Bearer Token
   - **Token**: `YOUR_API_KEY`

4. **Enable the Connector**:
   - Save the configuration
   - Enable the MeshAI connector in your ChatGPT session
   - You can now use natural language to interact with your agents

#### OpenAI Agents SDK (Python)

For developers using the OpenAI Agents SDK:

```python
from openai_agents import Agent
from openai_agents.mcp import MCPServer

# Configure MeshAI MCP server
mcp_server = MCPServer(
    url="https://mcp.meshai.dev/v1/mcp",
    headers={"Authorization": "Bearer YOUR_API_KEY"},
    transport="http"
)

# Create agent with MCP tools
agent = Agent(
    model="gpt-4",
    mcp_servers=[mcp_server]
)

# Execute tasks through MeshAI
response = agent.run(
    "Use MeshAI to analyze customer sentiment from the latest reviews"
)
```

#### OpenAI Responses API

For API integrations using the Responses API:

```python
import openai

client = openai.OpenAI()

response = client.responses.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Analyze market trends using MeshAI"}
    ],
    mcp_servers=[
        {
            "server_url": "https://mcp.meshai.dev/v1/mcp",
            "headers": {"Authorization": "Bearer YOUR_API_KEY"}
        }
    ]
)
```

### Features Available
Once connected, Claude Code, Gemini CLI, and OpenAI ChatGPT will have access to all MeshAI tools:
- Execute tasks across your agent network
- Query and manage agents
- Create and monitor workflows
- Check system status and metrics
- Batch execute multiple tasks
- Access system performance data

### Transport Method

**Current Implementation**: The MeshAI MCP server currently uses HTTP transport (`-t http`) for stateless request-response communication.

**Coming Soon**: SSE (Server-Sent Events) transport for real-time, bidirectional communication as described in [Anthropic's remote MCP server documentation](https://support.anthropic.com/en/articles/11503834-building-custom-connectors-via-remote-mcp-servers). This will enable:
- Persistent connections for real-time updates
- Streaming responses for long-running tasks
- Bidirectional event-driven communication
- Reduced latency for interactive workflows

### Authentication

All requests to the MCP server require authentication using your MeshAI API key:

- **Header**: `Authorization: Bearer YOUR_API_KEY`
- **Base URL**: `https://mcp.meshai.dev`
- **API Endpoint**: `https://api.meshai.dev`
- **Runtime Endpoint**: `https://runtime.meshai.dev`

## Usage Examples

### With Claude Code

Once you've connected Claude Code to MeshAI MCP, you can use natural language to interact with your agents:

**Execute a task:**
```
You: "Use MeshAI to analyze the sentiment of our latest customer reviews"
Claude: I'll execute that task using MeshAI's sentiment analysis agents...
[Uses mesh_execute tool]
```

**Check agent status:**
```
You: "Show me all active MeshAI agents with data analysis capabilities"
Claude: Let me list the agents with those specifications...
[Uses list_agents tool with filters]
```

### With Gemini CLI

After configuring MeshAI MCP in Gemini CLI, you can interact with your agents:

**Execute a task:**
```bash
gemini "Using MeshAI, generate a summary of today's market trends"
# Gemini will use the mesh_execute tool via MCP
```

**List agents:**
```bash
gemini "List all available MeshAI agents for text generation"
# Uses list_agents tool with capability filter
```

**Check task status:**
```bash
gemini "Check the status of MeshAI task abc-123"
# Uses get_task_status tool
```

**Batch operations:**
```bash
gemini "Execute these three tasks in parallel using MeshAI: 
1. Analyze customer sentiment
2. Generate weekly report
3. Update dashboard metrics"
# Uses batch_execute tool
```

### With OpenAI ChatGPT

After configuring MeshAI as a custom connector, you can interact with your agents using natural language:

**Execute a task:**
```
You: "Using MeshAI, analyze the sentiment of our customer reviews from this quarter"
ChatGPT: I'll use MeshAI's sentiment analysis capabilities to process your customer reviews...
[Uses mesh_execute tool via MCP connector]
```

**List agents:**
```
You: "Show me all available MeshAI agents that can handle code generation tasks"
ChatGPT: Let me query your MeshAI agents with code generation capabilities...
[Uses list_agents tool with capability filter]
```

**Monitor workflows:**
```
You: "Check the status of my MeshAI data processing workflow"
ChatGPT: I'll check your workflow status using MeshAI...
[Uses get_task_status and list_active_tasks tools]
```

**Batch operations:**
```
You: "Use MeshAI to run these tasks in parallel: sentiment analysis, report generation, and data visualization"
ChatGPT: I'll execute these three tasks simultaneously using MeshAI's batch processing...
[Uses batch_execute tool]
```

### Common Use Cases

**Monitor tasks:**
```
"What tasks are currently running in MeshAI?"
# Uses list_active_tasks tool
```

**Run workflows:**
```
"Execute the customer feedback workflow in MeshAI"
# Uses execute_workflow tool
```

**System health:**
```
"Show me MeshAI system statistics"
# Uses get_system_stats tool
```

## Direct API Usage Examples

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

The MeshAI MCP server exposes the following tools through the Model Context Protocol:

### Task Execution

#### `mesh_execute`
Execute a task using MeshAI's multi-agent orchestration.

**Parameters:**
- `task` (required): Task description
- `capabilities` (optional): Array of required capabilities

**Example:**
```json
{
  "task": "Analyze customer feedback and generate insights report",
  "capabilities": ["sentiment-analysis", "report-generation"]
}
```

#### `batch_execute`
Execute multiple tasks in parallel for improved efficiency.

**Parameters:**
- `tasks` (required): Array of tasks to execute
  - Each task includes: `task` (string) and optional `capabilities` (array)

### Agent Management

#### `list_agents`
List available agents in the MeshAI system.

**Parameters:**
- `status` (optional): Filter by status (active, inactive, all)
- `framework` (optional): Filter by framework (langchain, crewai, autogen)
- `capabilities` (optional): Filter by capabilities array

#### `get_agent_details`
Get detailed information about a specific agent.

**Parameters:**
- `agent_id` (required): The agent's unique identifier

### Workflow Management

#### `list_workflows`
List available workflows in the MeshAI system.

**Parameters:**
- `category` (optional): Filter by workflow category
- `tags` (optional): Filter by tags array

#### `execute_workflow`
Execute a pre-defined workflow.

**Parameters:**
- `workflow_id` (required): Workflow ID
- `parameters` (optional): Workflow parameters object

### Task Monitoring

#### `get_task_status`
Check the status of a submitted task.

**Parameters:**
- `task_id` (required): Task ID

#### `get_task_result`
Retrieve the result of a completed task.

**Parameters:**
- `task_id` (required): Task ID

#### `cancel_task`
Cancel a running task.

**Parameters:**
- `task_id` (required): Task ID

#### `list_active_tasks`
List currently running tasks.

**Parameters:**
- `limit` (optional): Maximum number of tasks to return (default: 10)

### System Information

#### `get_system_stats`
Get system performance metrics and statistics.

**Parameters:**
- `include_agents` (optional): Include agent statistics (default: true)
- `include_tasks` (optional): Include task statistics (default: true)

#### `list_capabilities`
List all available capabilities in the system.

**Parameters:** None

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

```bash
curl -X POST https://mcp.meshai.dev/api/v1/execute \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: text/event-stream" \
  -d '{
    "task": "Generate comprehensive market analysis",
    "stream": true
  }'
```

### Context Preservation

The MCP server maintains context across interactions:

```bash
curl -X POST https://mcp.meshai.dev/api/v1/continue \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "task_id": "previous-task-id",
    "additional_instructions": "Focus on European markets"
  }'
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

#### Claude Code / Gemini CLI / ChatGPT Connection Failed
```
Error: Failed to connect to MeshAI MCP server
```
**Solution**: 
1. Verify your API key is correct
2. Ensure you're using the correct command format
3. Check that your API key has the necessary permissions

#### Authentication Error
```
Error: 401 Unauthorized
```
**Solution**: 
1. Create a new API key in the MeshAI dashboard
2. Make sure to include the "Bearer" prefix in the Authorization header
3. Check that your API key hasn't expired

#### Agent Not Found
```
Error: No suitable agent found for task
```
**Solution**: 
1. Ensure agents are registered and active in your MeshAI workspace
2. Check that agents have the required capabilities
3. Verify agents are not at capacity

#### Timeout Errors
```
Error: Task execution timeout
```
**Solution**: 
1. Increase timeout parameter in your task request
2. Check agent performance and availability
3. Consider breaking large tasks into smaller ones

#### MCP Tools Not Available
```
Error: MCP tools not showing in Claude Code, Gemini CLI, or ChatGPT
```
**Solution for Claude Code**:
1. Restart Claude Code after adding the MCP server
2. Check the connection status with: `claude mcp list`
3. Verify the server URL is correct: `https://mcp.meshai.dev/v1/mcp`

**Solution for Gemini CLI**:
1. Verify your `settings.json` configuration
2. Check connection with: `gemini mcp list`
3. Ensure the `httpUrl` field is set correctly
4. Verify your API key has proper permissions

**Solution for OpenAI ChatGPT**:
1. Ensure you have ChatGPT Pro, Business, Enterprise, or Edu access
2. Verify the custom connector is properly configured with the correct URL
3. Check that the Bearer token is set correctly in the connector settings
4. Try disabling and re-enabling the MeshAI connector
5. For Agents SDK: Verify the MCPServer configuration and import statements

### Debug Mode

Enable debug logging by adding the debug parameter to your API requests:

```bash
curl -X POST https://mcp.meshai.dev/api/v1/execute?debug=true \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"task": "Test task"}'
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
import requests
import os

api_key = os.environ.get('MESHAI_API_KEY')
headers = {
    'Authorization': f'Bearer {api_key}',
    'Content-Type': 'application/json'
}

response = requests.post(
    'https://mcp.meshai.dev/api/v1/execute',
    headers=headers,
    json={
        'task': 'Analyze sentiment of customer reviews',
        'capabilities': ['sentiment-analysis']
    }
)

result = response.json()
print(result)
```

### Node.js Integration
```javascript
const fetch = require('node-fetch');

const apiKey = process.env.MESHAI_API_KEY;

const response = await fetch('https://mcp.meshai.dev/api/v1/execute', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    task: 'Generate product descriptions',
    capabilities: ['content-generation']
  })
});

const result = await response.json();
console.log(result);
```

## Support & Resources

- **Documentation**: https://docs.meshai.dev/mcp
- **MCP Server**: https://mcp.meshai.dev
- **API Status**: https://mcp.meshai.dev/status
- **Discord Community**: https://discord.gg/meshai
- **Support Email**: support@meshai.dev

## Changelog

### Version 1.0.0 (2024-01-15)
- Initial release with core MCP functionality
- Support for agent execution and workflow orchestration
- Claude Desktop integration

### Version 1.1.0 (Coming Soon)
- SSE (Server-Sent Events) transport support for real-time communication
- Streaming response support for long-running tasks
- Enhanced context preservation across sessions
- Custom agent registration via MCP
- Performance improvements
- Full compatibility with Anthropic's remote MCP server specification

---

*For the latest updates and additional resources, visit [docs.meshai.dev](https://docs.meshai.dev)*