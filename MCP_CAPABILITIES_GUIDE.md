# MCP Capabilities Guide

## Overview

The MeshAI MCP (Model Context Protocol) integration now supports intelligent capability detection and specification for task routing. This ensures tasks are matched with agents that have the appropriate capabilities, improving task success rates and performance.

## Changes from Previous Version

- **Old behavior**: Tasks defaulted to `["general"]` capability when not specified
- **New behavior**: Tasks default to `["text_generation"]` with intelligent auto-detection based on task content

## Capability Types

MeshAI supports the following capability types for agent matching:

| Capability | Description | Example Tasks |
|------------|-------------|---------------|
| `text_generation` | General text creation, Q&A, explanations | "What is...", "Explain...", "Tell me about..." |
| `code_generation` | Programming, scripting, debugging | "Write a function...", "Debug this code...", "Implement..." |
| `data_analysis` | Statistics, calculations, data processing | "Analyze...", "Calculate the average...", "Create a chart..." |
| `reasoning` | Logic, problem-solving, mathematical proofs | "Solve this puzzle...", "Prove that...", "What's the logic behind..." |
| `image_generation` | Visual content creation | "Generate an image...", "Draw a diagram...", "Create an illustration..." |

## Using the mesh_execute Tool

### Method 1: Explicit Capabilities

Specify the exact capabilities your task requires:

```json
{
  "task": "What is another name for rectangle?",
  "capabilities": ["text_generation"]
}
```

### Method 2: Auto-Detection (Recommended)

Let the system intelligently detect capabilities from your task:

```json
{
  "task": "What is another name for rectangle?"
}
// Auto-detects: ["text_generation"]
```

### Method 3: Multiple Capabilities

For complex tasks requiring multiple capabilities:

```json
{
  "task": "Analyze this dataset and write a Python script to visualize it",
  "capabilities": ["data_analysis", "code_generation"]
}
```

## Auto-Detection Patterns

The gateway uses pattern matching to intelligently detect required capabilities:

### Text Generation Patterns
- Questions: "what is", "what are", "who", "when", "where", "why", "how"
- Instructions: "explain", "describe", "tell me", "summarize"
- Creation: "write", "create a story", "translate", "define"
- Lookups: "another name for", "synonym"

### Code Generation Patterns
- Programming: "write code", "implement", "function", "class", "algorithm"
- Debugging: "debug", "fix code", "refactor", "optimize code"
- Development: "script", "api", "endpoint", "database query", "sql"

### Data Analysis Patterns
- Analysis: "analyze", "calculate", "compute", "statistics"
- Aggregation: "average", "sum", "count"
- Visualization: "graph", "chart", "visualize"
- Data handling: "report", "csv", "excel", "dataset", "metrics"

### Reasoning Patterns
- Problem-solving: "solve", "reason", "logic", "puzzle", "problem"
- Inference: "deduce", "infer", "conclude", "prove"
- Mathematics: "theorem", "mathematical", "equation", "formula"

### Image Generation Patterns
- Visual: "image", "picture", "photo", "visual"
- Creation: "draw", "sketch", "diagram", "illustration"
- AI generation: "generate image", "dall-e"

## Examples

### Example 1: Simple Question
```python
# MCP Request
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "mesh_execute",
    "arguments": {
      "task": "What is the capital of France?"
    }
  }
}

# System behavior:
# - Auto-detects: ["text_generation"]
# - Routes to text generation agents
```

### Example 2: Code Task
```python
# MCP Request
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "mesh_execute",
    "arguments": {
      "task": "Write a Python function to calculate factorial"
    }
  }
}

# System behavior:
# - Auto-detects: ["code_generation"]
# - Routes to code generation agents
```

### Example 3: Complex Task with Explicit Capabilities
```python
# MCP Request
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "mesh_execute",
    "arguments": {
      "task": "Analyze sales data and create visualizations",
      "capabilities": ["data_analysis", "code_generation"]
    }
  }
}

# System behavior:
# - Uses provided capabilities
# - Routes to agents with both capabilities
```

### Example 4: With Specific Agents
```python
# MCP Request
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "mesh_execute",
    "arguments": {
      "task": "Generate a marketing report",
      "capabilities": ["text_generation"],
      "agents": ["agent-123", "agent-456"]  # Optional: specific agent IDs
    }
  }
}

# System behavior:
# - Uses specified agents if available
# - Falls back to capability matching if agents unavailable
```

## Implementation Details

### Gateway Processing (mcp_gateway_complete.py)

1. Receives MCP tool call for `mesh_execute`
2. Checks if `capabilities` field is provided
3. If not provided, calls `auto_detect_capabilities(task)`
4. Forwards to mesh runtime with detected/provided capabilities

### Mesh Runtime Processing (main_production.py)

1. Receives task with capabilities
2. If no capabilities provided, defaults to `["text_generation"]`
3. Stores capabilities in task database
4. Routes task to agents matching required capabilities

### Auto-Detection Logic

```python
def auto_detect_capabilities(task_text: str) -> List[str]:
    """Auto-detect required capabilities based on task content"""
    task_lower = task_text.lower()
    capabilities = []

    # Check patterns and add matching capabilities
    if any(pattern in task_lower for pattern in text_patterns):
        capabilities.append("text_generation")

    if any(pattern in task_lower for pattern in code_patterns):
        capabilities.append("code_generation")

    # ... check other patterns ...

    # Default to text_generation if no matches
    if not capabilities:
        capabilities = ["text_generation"]

    return capabilities
```

## Best Practices

1. **Let auto-detection handle most cases** - The system is designed to intelligently detect capabilities
2. **Specify capabilities for ambiguous tasks** - When a task could match multiple patterns
3. **Use multiple capabilities for complex tasks** - Tasks can require multiple agent capabilities
4. **Use specific agent IDs sparingly** - Only when you need specific agents, not capability-based routing

## Troubleshooting

### Task routed to wrong agent type
- Check if task text matches expected patterns
- Consider explicitly specifying capabilities
- Review agent capabilities in registry

### No agents found for task
- Verify agents with required capabilities are registered
- Check if capabilities are too specific
- Consider using more general capabilities

### Auto-detection not working as expected
- Task text may not match any patterns
- Consider adding explicit capabilities
- Default fallback is `["text_generation"]`

## Migration from Previous Version

If you have existing code using the MCP integration:

1. **No changes required** - Auto-detection handles most cases
2. **Update explicit "general" capabilities** to more specific types
3. **Test task routing** to ensure expected behavior

## API Changes

### Tool Schema Update
```json
{
  "name": "mesh_execute",
  "inputSchema": {
    "properties": {
      "task": {"type": "string", "description": "Task description"},
      "capabilities": {  // NEW FIELD
        "type": "array",
        "items": {"type": "string"},
        "description": "Required capabilities (auto-detected if not specified)"
      },
      "agents": {"type": "array", "items": {"type": "string"}},
      "workflow": {"type": "string"},
      "context": {"type": "object"}
    }
  }
}
```

## Version History

- **v2.0** (Current): Added capabilities field and auto-detection
- **v1.0**: Basic MCP integration with "general" default

## Support

For issues or questions about capability routing:
1. Check agent capabilities in admin dashboard
2. Review task auto-detection patterns above
3. Use explicit capabilities for testing
4. Contact support with task ID for investigation