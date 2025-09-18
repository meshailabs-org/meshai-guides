# Claude Desktop MCP Integration Guide

This guide explains how to connect Claude Desktop to the MeshAI MCP Server.

## Prerequisites

- Claude Desktop installed on your computer
- Claude Desktop Pro, Max, Team, or Enterprise plan (for custom connectors)
- Node.js installed (Claude Desktop uses its built-in Node runtime)
- A MeshAI account (you'll be prompted to authenticate on first connection)

## Setup Instructions

### 1. Open Claude Desktop Settings

1. Launch Claude Desktop
2. Go to **Settings** > **Developer** > **Edit Config**

### 2. Configure the MCP Connection

Add the following configuration to your `claude_desktop_config.json` file:

```json
{
  "isUsingBuiltInNodeForMcp": true,
  "mcpServers": {
    "meshai": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://auth.meshai.dev/"
      ]
    }
  }
}
```

### 3. Save and Restart

1. Save the configuration file
2. Completely restart Claude Desktop (quit and relaunch)

### 4. Authenticate with MeshAI

On first connection, mcp-remote will:
1. Detect that authentication is required
2. Open your browser to the MeshAI OAuth login page
3. After successful login, return you to Claude Desktop
4. Store your authentication token for future sessions

The authentication process uses OAuth 2.1 with PKCE for security.

### 5. Verify Connection

After authentication, you should see the MeshAI tools available in Claude Desktop:

1. Starting a new conversation
2. Looking for the tools icon (usually in the bottom right)
3. You should see three MeshAI tools:
   - **MeshAI Task Executor** - Execute tasks using MeshAI's multi-agent orchestration
   - **List MeshAI Agents** - List available AI agents in the MeshAI registry
   - **Submit Agent Workflow** - Submit multi-agent workflows for execution

## Available Tools

### 1. MeshAI Task Executor (`mesh_execute`)
Execute a task using MeshAI's intelligent agent routing.

**Parameters:**
- `task` (required): Task description
- `capabilities` (optional): Required capabilities (e.g., text_generation, code_generation)
- `agents` (optional): Specific agent IDs to use
- `workflow` (optional): Workflow type
- `context` (optional): Additional context

**Example:**
```
Execute task: "Analyze this code for security vulnerabilities"
```

### 2. List MeshAI Agents (`list_agents`)
List available AI agents in the MeshAI registry.

**Parameters:**
- `capabilities` (optional): Filter by capabilities
- `framework` (optional): Filter by framework (LangChain, CrewAI, AutoGen)

**Example:**
```
List agents with capability: "code_generation"
```

### 3. Submit Agent Workflow (`submit_workflow`)
Submit a multi-agent workflow for execution.

**Parameters:**
- `workflow_type` (required): Type of workflow to execute
- `payload` (required): Workflow-specific payload data
- `agents` (optional): List of specific agents to use

## Troubleshooting

### Tools Not Appearing

1. **Check configuration syntax**: Ensure the JSON is valid
2. **Verify URL**: The server URL should be `https://auth.meshai.dev/`
3. **Restart Claude Desktop**: A full restart is required after configuration changes
4. **Check logs**: Look for errors in Claude Desktop's developer console

### Connection Errors

If you see connection errors:

1. Verify your internet connection
2. Check if the MeshAI server is accessible: `curl https://auth.meshai.dev/`
3. Ensure `mcp-remote` can be installed via npx
4. Check if mcp-remote opened a browser for authentication

### Authentication Issues

If authentication fails or doesn't trigger:

1. **Browser didn't open**: mcp-remote should automatically open your browser for authentication
2. **Token expired**: Delete the stored token and restart Claude Desktop to re-authenticate
3. **Invalid credentials**: Ensure you have a valid MeshAI account
4. **Callback failed**: Check that your browser can reach `https://auth.meshai.dev/oauth/callback`

The MeshAI MCP server uses OAuth 2.1 authentication:
- **Claude Desktop**: Uses mcp-remote which handles OAuth flow automatically
- **ChatGPT**: Direct OAuth authentication through OpenAI
- **Other clients**: Standard OAuth 2.1 with PKCE

## Technical Details

### How It Works

1. Claude Desktop uses the `npx` command to run `mcp-remote`
2. `mcp-remote` acts as a bridge between Claude's stdio communication and the HTTP server
3. On first connection, mcp-remote detects the 401 response and initiates OAuth flow
4. After successful authentication, mcp-remote stores the token and includes it in subsequent requests
5. Tools are executed on the server and results are returned to Claude Desktop

### Security

- Authentication uses OAuth 2.1 with PKCE for security
- Tokens are stored securely by mcp-remote
- All tool executions are logged on the server
- The server implements rate limiting and security monitoring
- Sessions expire after 24 hours and require re-authentication

## Configuration File Locations

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%/Claude/claude_desktop_config.json`
- **Linux**: `~/.config/Claude/claude_desktop_config.json`

## Support

For issues or questions:
- Check the [MeshAI Documentation](https://docs.meshai.dev)
- Contact support at support@meshai.dev

## Version Information

- Current Server Version: 1.6.7
- Protocol Version: 2025-06-18
- MCP Specification: [Model Context Protocol](https://modelcontextprotocol.io)