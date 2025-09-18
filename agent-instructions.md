# Agent Instructions Guide

## Overview

Instructions are a critical component of agent behavior in MeshAI, defining how agents should conduct themselves, make decisions, and handle various scenarios. Following the three-component agent model (Model, Tools, Instructions), MeshAI now provides first-class support for comprehensive agent instructions.

## Why Instructions Matter

Based on best practices from leading AI providers:
- **Behavioral Consistency**: Instructions ensure agents behave predictably across different tasks
- **Task Boundaries**: Clear guidelines prevent agents from exceeding their intended scope
- **Decision Frameworks**: Instructions provide criteria for handling edge cases and ambiguous situations
- **Output Quality**: Structured instructions lead to more consistent, higher-quality outputs
- **Safety & Compliance**: Built-in guardrails ensure agents operate within acceptable parameters

## Instruction Structure

MeshAI supports both simple string instructions and structured instruction objects:

### Simple String Format
```javascript
const agent = {
  id: "customer-service-agent",
  name: "Customer Service Assistant",
  instructions: "You are a helpful customer service agent. Always be polite, professional, and solution-oriented. Never share sensitive customer data or make promises beyond company policy."
}
```

### Structured Format
```javascript
const agent = {
  id: "data-analyst-agent",
  name: "Data Analysis Expert",
  instructions: {
    guidelines: [
      "Provide data-driven insights with supporting evidence",
      "Always cite data sources and methodologies",
      "Express uncertainty when confidence is low",
      "Maintain objectivity in analysis"
    ],
    taskBoundaries: [
      "DO analyze data patterns and trends",
      "DO provide statistical summaries",
      "DO NOT make financial recommendations",
      "DO NOT access external data without permission"
    ],
    outputFormat: "Present findings as: 1) Executive Summary, 2) Key Findings, 3) Supporting Data, 4) Recommendations",
    edgeCaseHandling: [
      "If data is incomplete, clearly state limitations",
      "If multiple interpretations exist, present all viable options",
      "If requested analysis is outside scope, suggest alternatives"
    ],
    guardrails: [
      "Never process personally identifiable information",
      "Ensure all calculations are reproducible",
      "Flag potential biases in data or methodology"
    ]
  }
}
```

## Implementation Examples

### 1. OpenAI Agent with Instructions

```javascript
import { MeshClient } from '@meshailabs/sdk';

const client = await MeshClient.createAndInitialize({
  apiKey: process.env.MESHAI_API_KEY
});

// Register an OpenAI agent with comprehensive instructions
const openaiAgent = await client.registerAgent({
  id: "research-assistant",
  name: "Research Assistant",
  framework: "openai",
  capabilities: ["research", "summarization", "analysis"],
  endpoint: "https://api.openai.com/v1/assistants",

  instructions: {
    guidelines: [
      "You are an expert research assistant specializing in technology trends",
      "Provide comprehensive, well-structured research summaries",
      "Always verify information from multiple sources",
      "Maintain an objective, analytical tone"
    ],
    taskBoundaries: [
      "DO conduct thorough research on requested topics",
      "DO provide citations and references",
      "DO NOT speculate without evidence",
      "DO NOT access unauthorized data sources"
    ],
    outputFormat: "Structure responses with: Abstract, Key Findings, Detailed Analysis, Citations",
    edgeCaseHandling: [
      "If information is conflicting, present all viewpoints",
      "If topic is too broad, suggest focused subtopics",
      "If sources are unreliable, note credibility concerns"
    ],
    guardrails: [
      "Avoid bias in research presentation",
      "Flag potential misinformation",
      "Respect intellectual property rights"
    ]
  }
});
```

### 2. Anthropic Claude Agent with Instructions

```javascript
// Register a Claude agent with specific behavioral instructions
const claudeAgent = await client.registerAgent({
  id: "code-reviewer",
  name: "Code Review Expert",
  framework: "anthropic",
  capabilities: ["code_review", "best_practices", "security_analysis"],
  endpoint: "https://api.anthropic.com/v1/messages",

  instructions: {
    guidelines: [
      "You are a senior software engineer conducting thorough code reviews",
      "Focus on code quality, maintainability, and security",
      "Provide constructive feedback with specific examples",
      "Suggest improvements, not just identify problems"
    ],
    taskBoundaries: [
      "DO review code for bugs, security issues, and best practices",
      "DO suggest specific improvements with examples",
      "DO NOT modify code without explicit permission",
      "DO NOT approve code with critical security vulnerabilities"
    ],
    outputFormat: "Review format: 1) Summary, 2) Critical Issues, 3) Suggestions, 4) Positive Aspects",
    edgeCaseHandling: [
      "If code purpose is unclear, request clarification",
      "If multiple solutions exist, explain tradeoffs",
      "If security risk is critical, escalate immediately"
    ],
    guardrails: [
      "Never expose sensitive credentials in reviews",
      "Maintain professional, constructive tone",
      "Follow OWASP security guidelines"
    ]
  }
});
```

### 3. Task-Specific Instructions

You can override agent instructions for specific tasks:

```javascript
// Execute a task with custom instructions
const result = await client.execute({
  taskType: "analyze_user_feedback",
  input: "Analyze the sentiment and key themes in our latest user feedback",

  // Task-specific instructions override agent defaults
  instructions: {
    taskGuidelines: [
      "Focus on actionable insights",
      "Categorize feedback by feature area",
      "Identify top 3 improvement opportunities"
    ],
    constraints: [
      "Analysis must be completed within 2 minutes",
      "Use only the provided feedback data",
      "Maintain user anonymity"
    ],
    outputRequirements: "JSON format with: sentiment_score, themes[], recommendations[]",
    prioritization: [
      "Critical bugs first",
      "Feature requests by frequency",
      "UI/UX improvements last"
    ]
  },

  requiredCapabilities: ["sentiment_analysis", "text_summarization"]
});
```

## Best Practices

### 1. Layered Instructions
Combine agent-level and task-level instructions for flexibility:

```javascript
// Agent has general instructions
const agent = {
  instructions: "General customer service guidelines..."
};

// Task can add specific requirements
const task = {
  instructions: "For this VIP customer interaction..."
};
```

### 2. Clear Boundaries
Always specify what agents should and shouldn't do:

```javascript
instructions: {
  taskBoundaries: [
    "DO provide factual information",
    "DO NOT make medical diagnoses",
    "DO NOT share personal opinions"
  ]
}
```

### 3. Structured Output Requirements
Define expected output formats for consistency:

```javascript
instructions: {
  outputFormat: `
    Return JSON with:
    {
      "summary": "Brief overview",
      "details": "Comprehensive analysis",
      "confidence": 0.0-1.0,
      "sources": ["citation1", "citation2"]
    }
  `
}
```

### 4. Guardrails for Safety
Include safety measures in instructions:

```javascript
instructions: {
  guardrails: [
    "Never process credit card numbers",
    "Redact all PII from outputs",
    "Flag potentially harmful content",
    "Maintain audit trail of decisions"
  ]
}
```

## Multi-Agent Orchestration with Instructions

When orchestrating multiple agents, instructions help coordinate behavior:

```javascript
// Manager agent with orchestration instructions
const managerAgent = {
  id: "orchestrator",
  instructions: {
    guidelines: [
      "Coordinate specialist agents for complex tasks",
      "Route tasks based on agent capabilities and current load",
      "Synthesize outputs from multiple agents"
    ],
    taskBoundaries: [
      "DO decompose complex tasks into subtasks",
      "DO NOT execute tasks directly",
      "DO ensure all subtasks are completed before final response"
    ]
  }
};

// Specialist agents with focused instructions
const specialistAgents = [
  {
    id: "data-agent",
    instructions: "Focus solely on data extraction and transformation..."
  },
  {
    id: "analysis-agent",
    instructions: "Perform statistical analysis on provided data..."
  },
  {
    id: "visualization-agent",
    instructions: "Create clear, insightful visualizations..."
  }
];
```

## Dynamic Instruction Updates

Instructions can be updated based on context or user preferences:

```javascript
// Update agent instructions based on user preferences
const userPreferences = await getUserPreferences(userId);

const personalizedAgent = await client.updateAgent(agentId, {
  instructions: {
    guidelines: [
      ...baseGuidelines,
      `Communicate in ${userPreferences.language}`,
      `Use ${userPreferences.formalityLevel} tone`
    ],
    outputFormat: userPreferences.preferredFormat
  }
});
```

## Monitoring Instruction Compliance

Track how well agents follow instructions:

```javascript
// Log instruction adherence
const executionResult = await client.execute(task);

// Check if output matches expected format
const complianceCheck = validateOutputFormat(
  executionResult.result,
  agent.instructions.outputFormat
);

// Track metrics
await client.updateAgentMetrics(agentId, {
  instructionCompliance: complianceCheck.score,
  violations: complianceCheck.violations
});
```

## Common Instruction Patterns

### Research Agent
```javascript
instructions: {
  guidelines: [
    "Provide comprehensive, unbiased research",
    "Cite all sources with links",
    "Highlight conflicting information"
  ],
  outputFormat: "Academic paper structure with abstract, findings, and references"
}
```

### Customer Service Agent
```javascript
instructions: {
  guidelines: [
    "Be empathetic and solution-focused",
    "Follow company policies strictly",
    "Escalate complex issues appropriately"
  ],
  guardrails: [
    "Never share customer data across accounts",
    "Don't make promises beyond authority level"
  ]
}
```

### Code Generation Agent
```javascript
instructions: {
  guidelines: [
    "Write clean, maintainable code",
    "Include comprehensive comments",
    "Follow language-specific best practices"
  ],
  taskBoundaries: [
    "DO NOT include hardcoded credentials",
    "DO NOT use deprecated APIs"
  ]
}
```

### Data Analysis Agent
```javascript
instructions: {
  guidelines: [
    "Provide statistical significance for all findings",
    "Clearly state assumptions and limitations",
    "Use appropriate visualizations"
  ],
  outputFormat: "Include: summary statistics, visualizations, interpretations, recommendations"
}
```

## Migration Guide

For existing agents without instructions, add them progressively:

```javascript
// Step 1: Add basic instructions
await client.updateAgent(agentId, {
  instructions: "Basic behavioral guidelines..."
});

// Step 2: Expand to structured format
await client.updateAgent(agentId, {
  instructions: {
    guidelines: ["Enhanced guidelines..."],
    taskBoundaries: ["Clear boundaries..."]
  }
});

// Step 3: Add task-specific overrides
const task = {
  instructions: {
    taskGuidelines: ["Specific for this task..."]
  }
};
```

## Conclusion

Instructions are essential for creating reliable, predictable AI agents. By clearly defining guidelines, boundaries, and expectations, you ensure that agents operate safely and effectively within your system. MeshAI's comprehensive instruction support enables you to build sophisticated multi-agent systems with confidence.

## Next Steps

- Review the [First Agent Guide](./first-agent.md) to see instructions in action
- Explore [Advanced Routing Strategies](./routing-strategies.md) that leverage instructions
- Check the [API Reference](https://api.meshai.dev/docs) for complete instruction schema
- Join our [Discord Community](https://discord.gg/meshai) for best practices and support