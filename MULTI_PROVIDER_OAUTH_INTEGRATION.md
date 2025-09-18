# Multi-Provider OAuth Integration Guide

## Overview

MeshAI's unified authentication service provides OAuth 2.1 authentication with JSON-RPC 2.0 support for multiple AI providers (ChatGPT, Claude, and others) to securely access MeshAI's agent orchestration platform through the MCP (Model Context Protocol).

## Quick Start

### For ChatGPT Users

1. **Add MeshAI MCP Server to ChatGPT**:
   - Go to ChatGPT Settings → Personalization → Connected MCP servers
   - Click "Add MCP Server"
   - Enter URL: `https://auth.meshai.dev`
   - Complete OAuth authentication with your MeshAI credentials

2. **Available Tools**:
   - `search` - Search for agents, docs, and workflows
   - `fetch` - Get detailed information about resources
   - `mesh_execute` - Run tasks through MeshAI agents
   - `list_agents` - Browse available agents
   - `submit_workflow` - Execute complex workflows

3. **Streaming Support**:
   - Real-time updates via Server-Sent Events (SSE)
   - Progress tracking for long-running tasks
   - Automatic streaming for supported operations

That's it! ChatGPT can now orchestrate AI agents through MeshAI with full streaming support.

## Architecture

### OAuth Server
- **Base URL**: `https://auth.meshai.dev`
- **OAuth Discovery**: `https://auth.meshai.dev/.well-known/oauth-authorization-server`
- **MCP Endpoint**: `https://auth.meshai.dev/v1/mcp`

### Authentication Flow

```
1. AI Provider discovers OAuth configuration
2. User authorizes with MeshAI credentials
3. Provider receives access token
4. All MCP requests include Bearer token
5. MeshAI validates and routes to appropriate agents
```

## Provider-Specific Setup

### ChatGPT

**Option 1: Simple Setup (Recommended)**
- Server URL: `https://auth.meshai.dev`
- ChatGPT handles OAuth automatically

**Option 2: Manual Configuration**
- Authorization URL: `https://auth.meshai.dev/oauth/authorize`
- Token URL: `https://auth.meshai.dev/oauth/token`
- MCP Server: `https://auth.meshai.dev/v1/mcp`
- Scopes: `mcp:read mcp:execute`

### Claude (Coming Soon)

Claude will support automatic MCP discovery:
- Server URL: `https://auth.meshai.dev`
- OAuth handled automatically via discovery

### Custom AI Applications

For developers integrating their own AI applications:

```javascript
// OAuth Configuration
const oauth = {
  discovery: 'https://auth.meshai.dev/.well-known/oauth-authorization-server',
  clientId: 'YOUR_CLIENT_ID',
  clientSecret: 'YOUR_CLIENT_SECRET',
  scopes: ['mcp:read', 'mcp:execute']
}

// MCP Request Example
fetch('https://auth.meshai.dev/v1/mcp', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    method: 'mesh_execute',
    params: {
      task: 'Analyze this data...',
      capabilities: ['analysis']
    }
  })
})
```

## Available MCP Tools

### search
Search across agents, documentation, and workflows.

```json
{
  "name": "search",
  "arguments": {
    "query": "Your search query",
    "type": "all|agents|docs|workflows"
  }
}
```

### fetch
Get detailed information about a specific resource.

```json
{
  "name": "fetch",
  "arguments": {
    "id": "resource-id",
    "type": "agent|task|workflow"  // optional, defaults to agent
  }
}
```

### mesh_execute
Execute tasks using MeshAI's intelligent agent routing.

```json
{
  "name": "mesh_execute",
  "arguments": {
    "task": "Your task description",
    "task_type": "chat|analysis|code|creative",
    "capabilities": ["required", "capabilities"]
  }
}
```

### list_agents
Discover available agents and their capabilities.

```json
{
  "name": "list_agents",
  "arguments": {
    "framework": "langchain|crewai|autogen",
    "capabilities": ["filter", "by", "capability"]
  }
}
```

### submit_workflow
Run complex multi-agent workflows.

```json
{
  "name": "submit_workflow",
  "arguments": {
    "workflow_type": "sequential|parallel",
    "payload": {
      "steps": [...]
    },
    "agents": ["agent_ids"]
  }
}
```

## OAuth Scopes

| Scope | Description | Use Case |
|-------|-------------|----------|
| `mcp:read` | Read agent information | Browse available agents |
| `mcp:execute` | Execute tasks | Run agent tasks |
| `mcp:manage` | Manage configurations | Admin operations |

Most users only need `mcp:read` and `mcp:execute`.

## Security

### For Users
- Uses your existing MeshAI account
- No separate passwords needed
- Tokens expire after 1 hour
- Automatic refresh for long sessions

### For Developers
- Client credentials kept secure
- All traffic over HTTPS
- Zero-trust validation
- Rate limiting enforced

## Troubleshooting

### "Error creating connector" in ChatGPT
- Ensure you're using `https://auth.meshai.dev` as the server URL
- Remove any duplicate MCP server configurations
- Try logging out and back into MeshAI

### "401 Unauthorized" during login
- Verify you have a MeshAI account (sign up at https://meshai.dev)
- Check your email/password are correct
- Ensure your account is active

### "403 Insufficient scope" errors
- The provider needs to request `mcp:execute` scope
- Re-authorize if scopes have changed
- Contact support if issue persists

### Token expired
- Tokens auto-refresh in most providers
- Manual refresh: Use refresh_token grant
- Re-authenticate if refresh fails

## How Multi-Provider Support Works

MeshAI uses a **centralized OAuth approach** where:

1. **Single Authentication Point**: All AI providers authenticate through `auth.meshai.dev`
2. **Unified Token Management**: One token system works across all providers
3. **Consistent API**: Same MCP methods available to all providers
4. **Provider Isolation**: Each provider gets unique client credentials
5. **User Context**: Your tasks are isolated by user account

This means you can:
- Use multiple AI assistants with one MeshAI account
- Switch between ChatGPT, Claude, etc. seamlessly
- Access the same agents from any provider
- Maintain consistent state across providers

## Supported Providers

### Currently Available
- ✅ **ChatGPT** - Full MCP integration via custom connectors
- ✅ **Custom Applications** - Any OAuth 2.1 compatible client

### Coming Soon
- 🚧 **Claude** - Native MCP support planned
- 🚧 **Gemini** - Integration in development
- 🚧 **Local LLMs** - Self-hosted integration

## Getting Help

- **Support Email**: support@meshai.dev
- **GitHub Community**: https://github.com/orgs/meshailabs-org/discussions
- **Status Page**: https://status.meshai.dev

## Next Steps

1. **Sign up** for MeshAI at https://meshai.dev
2. **Configure** your AI provider using the instructions above
3. **Start orchestrating** AI agents with simple commands
4. **Build workflows** for complex multi-agent tasks

---

*Last Updated: January 2025*