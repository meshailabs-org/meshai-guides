# MCP Streaming Guide - Real-time AI Agent Execution

## Overview

MeshAI supports Server-Sent Events (SSE) streaming for real-time updates during AI agent task execution. This guide explains how to use streaming capabilities with MeshAI's MCP server.

## What is SSE Streaming?

Server-Sent Events (SSE) allow the server to push real-time updates to clients over a single HTTP connection. This is perfect for:
- Long-running AI agent tasks
- Progress updates during multi-step workflows
- Real-time agent execution monitoring
- Streaming responses from language models

## Streaming Endpoint

### Standard Endpoint (JSON responses)
```
POST https://auth.meshai.dev/v1/mcp
```

### Streaming Endpoint (SSE responses)
```
POST https://auth.meshai.dev/v1/mcp/stream
```

## How It Works

### 1. Client Request
Send your MCP request to the streaming endpoint with the same format as standard requests:

```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "method": "tools/call",
  "params": {
    "name": "mesh_execute",
    "arguments": {
      "task": "Analyze this large dataset",
      "task_type": "analysis"
    }
  }
}
```

### 2. Server Response (SSE Format)
The server responds with a stream of events:

```
data: {"type": "start", "message": "Starting task execution"}

data: {"type": "progress", "message": "Loading data...", "progress": 10}

data: {"type": "progress", "message": "Analyzing patterns...", "progress": 50}

data: {"type": "result", "data": {"analysis": "..."}, "progress": 100}

data: {"jsonrpc": "2.0", "id": "123", "result": {"status": "completed"}}

data: [DONE]
```

## Event Types

### Start Event
```json
{
  "type": "start",
  "message": "Starting task execution",
  "timestamp": "2025-01-16T12:00:00Z"
}
```

### Progress Event
```json
{
  "type": "progress",
  "message": "Processing step 3 of 5",
  "progress": 60,
  "details": {
    "current_step": "data_analysis",
    "agent": "analytics-agent-01"
  }
}
```

### Result Event
```json
{
  "type": "result",
  "data": {
    // Task results
  },
  "progress": 100
}
```

### Error Event
```json
{
  "type": "error",
  "message": "Task failed: Insufficient resources",
  "code": "RESOURCE_ERROR",
  "details": {}
}
```

### Completion Event
```json
{
  "type": "complete",
  "status": "completed",
  "duration_ms": 5234
}
```

## Supported Operations

Currently, streaming is supported for:
- `mesh_execute` - Task execution with progress updates
- Future: `submit_workflow` - Multi-step workflow progress
- Future: `stream_chat` - Streaming chat responses

## Client Implementation Examples

### JavaScript/TypeScript
```javascript
const response = await fetch('https://auth.meshai.dev/v1/mcp/stream', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json',
    'Accept': 'text/event-stream'
  },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: '123',
    method: 'tools/call',
    params: {
      name: 'mesh_execute',
      arguments: {
        task: 'Analyze this data'
      }
    }
  })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;

  const chunk = decoder.decode(value);
  const lines = chunk.split('\n');

  for (const line of lines) {
    if (line.startsWith('data: ')) {
      const data = line.slice(6);
      if (data === '[DONE]') {
        console.log('Stream complete');
        break;
      }

      try {
        const event = JSON.parse(data);
        handleStreamEvent(event);
      } catch (e) {
        console.error('Failed to parse event:', e);
      }
    }
  }
}

function handleStreamEvent(event) {
  switch (event.type) {
    case 'start':
      console.log('Task started:', event.message);
      break;
    case 'progress':
      console.log(`Progress: ${event.progress}% - ${event.message}`);
      break;
    case 'result':
      console.log('Result:', event.data);
      break;
    case 'error':
      console.error('Error:', event.message);
      break;
  }
}
```

### Python
```python
import json
import requests
from sseclient import SSEClient

url = 'https://auth.meshai.dev/v1/mcp/stream'
headers = {
    'Authorization': f'Bearer {access_token}',
    'Content-Type': 'application/json',
    'Accept': 'text/event-stream'
}

payload = {
    'jsonrpc': '2.0',
    'id': '123',
    'method': 'tools/call',
    'params': {
        'name': 'mesh_execute',
        'arguments': {
            'task': 'Analyze this data'
        }
    }
}

response = requests.post(url, headers=headers, json=payload, stream=True)
client = SSEClient(response)

for event in client.events():
    if event.data == '[DONE]':
        print('Stream complete')
        break

    try:
        data = json.loads(event.data)

        if data.get('type') == 'progress':
            print(f"Progress: {data.get('progress')}% - {data.get('message')}")
        elif data.get('type') == 'result':
            print(f"Result: {data.get('data')}")
        elif data.get('type') == 'error':
            print(f"Error: {data.get('message')}")

    except json.JSONDecodeError:
        print(f"Failed to parse: {event.data}")
```

## ChatGPT Integration

ChatGPT automatically handles SSE streaming when available. When you execute a task through MeshAI:

1. ChatGPT detects streaming capability from the MCP server
2. Automatically uses the streaming endpoint for supported operations
3. Shows real-time progress updates in the chat interface
4. Handles errors and retries gracefully

## Backend Requirements

For backend services (agent-registry, mesh-runtime) to support streaming:

1. Implement `/api/v1/tasks/stream` endpoint
2. Return SSE-formatted responses
3. Include proper headers:
   ```
   Content-Type: text/event-stream
   Cache-Control: no-cache
   Connection: keep-alive
   X-Accel-Buffering: no
   ```
4. Send periodic keep-alive messages for long-running tasks

## Best Practices

### 1. Keep-Alive Messages
Send periodic messages to prevent connection timeout:
```
data: {"type": "keepalive", "timestamp": "2025-01-16T12:00:00Z"}
```

### 2. Error Handling
Always send errors as events before closing the stream:
```
data: {"type": "error", "message": "Task failed", "code": "TASK_ERROR"}
data: [DONE]
```

### 3. Progress Updates
Send regular progress updates for long tasks:
- Every 5-10 seconds for tasks > 30 seconds
- At major milestones (25%, 50%, 75%)
- When switching between agents or steps

### 4. Connection Management
- Set appropriate timeouts (2-5 minutes for long tasks)
- Handle reconnection gracefully
- Clean up resources on disconnect

## Troubleshooting

### Stream Not Working
- Verify you're using the `/v1/mcp/stream` endpoint
- Check that Accept header includes `text/event-stream`
- Ensure your client supports SSE

### Connection Drops
- Implement reconnection logic with exponential backoff
- Check for proxy/firewall timeouts
- Consider using keep-alive messages

### No Progress Updates
- Verify backend services support streaming
- Check that the operation supports streaming
- Ensure proper event formatting

## Future Enhancements

Planned streaming features:
- Streaming for workflow execution
- Real-time agent collaboration updates
- Live metrics and monitoring streams
- WebSocket support as alternative to SSE

## Support

For help with streaming integration:
- GitHub: https://github.com/orgs/meshailabs-org/discussions
- Email: support@meshai.dev
- Documentation: https://docs.meshai.dev

---

*Last Updated: January 2025*