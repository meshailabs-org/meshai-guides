# ChatGPT OAuth Integration with MeshAI

This guide explains how to configure ChatGPT to connect to MeshAI's MCP server using OAuth 2.1 authentication with full JSON-RPC 2.0 support.

## Prerequisites

- MeshAI account (sign up at https://meshai.dev)
- ChatGPT Plus or Team subscription
- Access to ChatGPT's MCP server settings

## Quick Setup (Recommended)

### Simple Method

1. Go to ChatGPT **Settings** → **Personalization** → **Connected MCP servers**
2. Click **"Add MCP Server"**
3. Enter Server URL: `https://auth.meshai.dev`
4. Complete OAuth login with your MeshAI credentials (same as app.meshai.dev)
5. Done! ChatGPT handles the OAuth configuration automatically

## Features Available

Once connected, ChatGPT can:
- **Search** - Search for agents, documentation, and workflows
- **Fetch** - Get detailed information about specific resources
- **Execute Tasks** - Run tasks through MeshAI's intelligent agent routing
- **List Agents** - Browse available AI agents and their capabilities
- **Stream Responses** - Receive real-time updates for long-running tasks (SSE)

## Manual Setup (Alternative)

### 1. Access MCP Settings

1. Open ChatGPT Settings
2. Navigate to **Personalization** → **Connected MCP servers**
3. Click **"Add MCP Server"**

### 2. Configure MCP Server

Enter these settings:

- **Server Name**: MeshAI Agent Platform
- **Server URL**: `https://auth.meshai.dev`
- **Description**: Connect to MeshAI's multi-agent orchestration platform

### 3. OAuth Configuration (if prompted)

If ChatGPT asks for manual OAuth configuration:

- **Authorization URL**: `https://auth.meshai.dev/oauth/authorize`
- **Token URL**: `https://auth.meshai.dev/oauth/token`
- **MCP Endpoint**: `https://auth.meshai.dev/v1/mcp`
- **Scope**: `mcp:read mcp:execute`

### 4. Complete Authorization

1. Save the connector configuration
2. ChatGPT will redirect you to MeshAI's login page at `https://auth.meshai.dev/oauth/login`
3. Log in with your MeshAI account credentials (the same ones you use at app.meshai.dev)
4. Approve the authorization request
5. You'll be redirected back to ChatGPT

> **Note**: Use your actual MeshAI account credentials. The OAuth system should authenticate against the same user database as app.meshai.dev.

## OAuth Endpoints Reference

MeshAI implements OAuth 2.1 with the following endpoints:

- **Discovery**: `https://auth.meshai.dev/.well-known/oauth-authorization-server`
- **Authorization**: `https://auth.meshai.dev/oauth/authorize`
- **Token**: `https://auth.meshai.dev/oauth/token`
- **UserInfo**: `https://auth.meshai.dev/oauth/userinfo`
- **Revocation**: `https://auth.meshai.dev/oauth/revoke`
- **Introspection**: `https://auth.meshai.dev/oauth/introspect`

## Supported Scopes

- `openid` - OpenID Connect support
- `profile` - User profile information
- `email` - User email address
- `mcp:read` - Read access to MeshAI agents and tasks
- `mcp:execute` - Execute tasks using MeshAI agents
- `mcp:manage` - Manage agent configurations (admin only)

## Using the MeshAI Connector

Once configured, you can use MeshAI capabilities in ChatGPT:

### Available MCP Tools

1. **mesh_execute** - Execute tasks using MeshAI's multi-agent orchestration
   - **Parameters:**
     - `task` (required): Task description
     - `capabilities` (optional): Required capabilities (e.g., text_generation, code_generation, data_analysis, reasoning)
     - `agents` (optional): Specific agent IDs to use
     - `workflow` (optional): Workflow type
     - `context` (optional): Additional context

2. **list_agents** - List available AI agents in the MeshAI registry
   - **Parameters:**
     - `capabilities` (optional): Filter by capabilities
     - `framework` (optional): Filter by framework (LangChain, CrewAI, AutoGen)

3. **submit_workflow** - Submit multi-agent workflows for execution
   - **Parameters:**
     - `workflow_type` (required): Type of workflow to execute
     - `payload` (required): Workflow-specific payload data
     - `agents` (optional): List of specific agents to use

### Example Usage

1. **Execute a Task**:
   ```
   Use mesh_execute to analyze this code for security vulnerabilities
   ```

2. **List Available Agents**:
   ```
   Use list_agents to show me agents with code_generation capability
   ```

3. **Submit a Workflow**:
   ```
   Use submit_workflow to run a data analysis pipeline
   ```

### Real-time Streaming

MeshAI supports Server-Sent Events (SSE) for real-time updates during task execution. ChatGPT will automatically show progress updates when running long tasks.

## Troubleshooting

### "Error creating connector"
- Make sure you're using `https://auth.meshai.dev` as the server URL
- Try removing any existing MeshAI connectors and re-adding
- Ensure you have a valid MeshAI account (sign up at https://meshai.dev)

### Authorization Failed
- Verify you're using your MeshAI account credentials (same as app.meshai.dev)
- Make sure your account is active and verified
- Check that you're not using demo or test credentials

### "fetch action not found" or Similar Errors
- The connector should automatically discover available actions
- If you see this error, try disconnecting and reconnecting the MCP server
- Ensure you're using the latest version of the connector

### Connection Issues
- Verify the server URL is exactly: `https://auth.meshai.dev`
- Check your internet connection
- Try logging into app.meshai.dev to verify your credentials work

### Token Expiration
- Access tokens expire after 60 minutes
- ChatGPT automatically refreshes tokens using the refresh token
- If issues persist, disconnect and reconnect the MCP server

## Security Notes

- OAuth tokens are securely stored by ChatGPT
- All communication uses HTTPS/TLS encryption
- Tokens can be revoked at any time through MeshAI's admin panel
- Each token is scoped to specific permissions

## Support

For assistance with the MeshAI OAuth integration:
- GitHub Community: https://github.com/orgs/meshailabs-org/discussions
- Support: support@meshai.dev
- Status: https://status.meshai.dev

## Next Steps

After successful integration, explore MeshAI's capabilities:
- Browse available agents and their capabilities
- Execute multi-agent workflows
- Monitor task execution in real-time
- Create custom agent configurations (with appropriate permissions)