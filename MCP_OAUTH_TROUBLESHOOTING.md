# MCP OAuth Configuration Troubleshooting Guide

## Overview

When configuring MCP servers that use OAuth authentication (like OpenAI's ChatGPT), you may encounter authentication issues when trying to integrate with MeshAI. This guide helps troubleshoot common problems.

## Common Issue: "Error creating connector"

### Problem
When trying to configure OpenAI's MCP server, you may encounter the error message: **"Error creating connector"**. This typically occurs when:
- The MCP server cannot establish a connection
- OAuth authentication fails to initialize
- There's a conflict between multiple MCP servers

### Example Configuration That May Cause Issues
```json
{
  "mcpServers": {
    "openai": {
      "command": "npx",
      "args": ["-y", "@openai/mcp-server"],
      "env": {
        // OAuth configuration
      }
    },
    "meshai": {
      "command": "npx",
      "args": ["-y", "meshai-mcp-server"],
      "env": {
        "MESHAI_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

## Why This Happens

1. **OAuth Flow Interruption**: MCP servers that require OAuth need to complete an authentication flow, which can conflict with other MCP servers running simultaneously.

2. **Token Storage**: OAuth tokens need to be stored and refreshed, which may not work correctly in a multi-server MCP environment.

3. **Callback URL Issues**: OAuth providers need to redirect back to your application, which can be problematic with MCP's architecture.

## Root Causes of "Error creating connector"

1. **Incompatible MCP Server Versions**: The OpenAI MCP server may require specific versions or configurations not compatible with your environment.

2. **OAuth Initialization Failure**: The OAuth flow cannot start properly, possibly due to:
   - Missing OAuth client configuration
   - Incorrect callback URLs
   - Browser-based authentication not available in your environment

3. **Multiple MCP Servers Conflict**: Running MeshAI and OpenAI MCP servers simultaneously can cause connection conflicts.

## Solutions

### Option 1: Use MeshAI as Your Single MCP Server (Recommended)

Instead of connecting directly to multiple MCP servers, use MeshAI as your unified interface:

```json
{
  "mcpServers": {
    "meshai": {
      "command": "npx",
      "args": ["-y", "meshai-mcp-server"],
      "env": {
        "MESHAI_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

Then, MeshAI will route your tasks to the appropriate agents (including OpenAI) without needing direct OAuth configuration.

### Option 2: Sequential Configuration (Not Simultaneous)

If you need both servers, configure them separately and use them at different times:

1. First, configure and authenticate with OpenAI's MCP server
2. Complete the OAuth flow and save the tokens
3. Then switch to MeshAI's MCP server for task routing

### Option 3: Use API Keys Instead of OAuth

For services that support both OAuth and API keys, prefer API key authentication:

```json
{
  "mcpServers": {
    "meshai": {
      "command": "npx",
      "args": ["-y", "meshai-mcp-server"],
      "env": {
        "MESHAI_API_KEY": "YOUR_MESHAI_KEY",
        "OPENAI_API_KEY": "YOUR_OPENAI_KEY"  // If MeshAI supports this
      }
    }
  }
}
```

## Best Practices

1. **Use MeshAI for Orchestration**: Let MeshAI handle the complexity of connecting to multiple AI services. This is what it's designed for.

2. **Avoid Multiple OAuth Flows**: Having multiple MCP servers that each require OAuth will likely cause conflicts.

3. **Check Token Expiration**: If you've previously authenticated, OAuth tokens may have expired and need refreshing.

## Specific Fix for "Error creating connector"

Since OpenAI's MCP server uses OAuth and this conflicts with other MCP servers, the best approach is:

### Immediate Solution
Remove the OpenAI MCP server from your configuration and use only MeshAI:

```json
{
  "mcpServers": {
    "meshai": {
      "command": "npx",
      "args": ["-y", "meshai-mcp-server"],
      "env": {
        "MESHAI_API_KEY": "YOUR_API_KEY"
      }
    }
    // Remove the "openai" server configuration
  }
}
```

### Why This Works
- MeshAI acts as a universal router to all AI services
- You don't need direct connections to individual AI providers
- MeshAI handles authentication and routing internally
- Avoids OAuth conflicts and connector errors

## Debugging Steps

1. **Check Logs**: Look for authentication errors in your MCP client logs
   ```bash
   # Check your application's console or log files
   ```

2. **Verify API Keys**: Ensure your MeshAI API key is valid
   ```bash
   curl -H "Authorization: Bearer YOUR_API_KEY" https://api.meshai.dev/api/v1/health
   ```

3. **Test MeshAI MCP Server Alone**: First verify MeshAI works without other MCP servers
   ```json
   {
     "mcpServers": {
       "meshai": {
         "command": "npx",
         "args": ["-y", "meshai-mcp-server"],
         "env": {
           "MESHAI_API_KEY": "YOUR_API_KEY"
         }
       }
     }
   }
   ```

4. **Contact Support**: If issues persist, reach out with:
   - Your configuration (without API keys)
   - Error messages
   - Steps to reproduce

## Alternative Approach: Direct API Integration

If MCP integration is proving difficult, you can always use MeshAI's REST API directly:

```python
import requests

response = requests.post(
    "https://api.meshai.dev/api/v1/tasks",
    headers={
        "Authorization": "Bearer YOUR_API_KEY",
        "Content-Type": "application/json"
    },
    json={
        "task": "Your task here",
        # MeshAI will route to OpenAI or other providers as needed
    }
)
```

## Current Limitations

- **OAuth + MCP**: OAuth flows in MCP environments are still evolving and may have compatibility issues
- **Multiple MCP Servers**: Running multiple MCP servers simultaneously can cause conflicts
- **Token Management**: Automated token refresh may not work correctly in all scenarios

## Future Improvements

We're working on:
1. Better OAuth support in the MeshAI MCP server
2. Unified authentication management
3. Automatic token refresh handling
4. Improved multi-server coordination

## Need Help?

- **Documentation**: [MCP Capabilities Guide](https://github.com/meshailabs-org/meshai-guides/blob/main/MCP_CAPABILITIES_GUIDE.md)
- **Support**: support@meshai.dev
- **Discord**: [Join our community](https://discord.gg/meshai)

---

*Note: MCP (Model Context Protocol) is still in development, and OAuth integration patterns are evolving. This guide will be updated as better solutions become available.*