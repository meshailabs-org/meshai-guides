# ChatGPT OAuth Integration with MeshAI

This guide explains how to configure ChatGPT to connect to MeshAI's MCP server using OAuth 2.1 authentication.

## Prerequisites

- MeshAI account (sign up at https://meshai.dev)
- ChatGPT Plus or Team subscription
- Access to ChatGPT's MCP server settings

## Quick Setup (Recommended)

### Simple Method

1. Go to ChatGPT **Settings** → **Personalization** → **Connected MCP servers**
2. Click **"Add MCP Server"**
3. Enter Server URL: `https://auth.meshai.dev`
4. Complete OAuth login with your MeshAI credentials
5. Done! ChatGPT handles the OAuth configuration automatically

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

1. **List Available Agents**:
   ```
   Show me all available MeshAI agents
   ```

2. **Execute Tasks**:
   ```
   Use MeshAI to analyze this document with the best suitable agent
   ```

3. **Check Task Status**:
   ```
   Check the status of my MeshAI tasks
   ```

## Troubleshooting

### Authorization Failed
- Verify the Client ID is correct: `mcp_VAO7jtrlgh92QkeWSCJqLw`
- Ensure you're using valid MeshAI credentials
- Check that the redirect URI matches ChatGPT's callback URL

### Connection Issues
- Verify the MCP Server URL: `https://mcp.meshai.dev/v1/mcp`
- Ensure OAuth endpoints are accessible
- Check ChatGPT's connector status

### Token Expiration
- Access tokens expire after 60 minutes
- ChatGPT should automatically refresh using the refresh token
- If issues persist, re-authorize the connector

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