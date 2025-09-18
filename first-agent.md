# Creating Your First AI Agent with MeshAI

This guide will walk you through creating and deploying your first AI agent using the MeshAI platform. We'll cover multiple approaches using different SDKs and frameworks.

## Prerequisites

- MeshAI account with API key (get one at [admin.meshai.dev](https://admin.meshai.dev))
- Node.js 16+ or Python 3.8+ installed
- Basic understanding of async/await programming

## Quick Start: 5-Minute Agent

### Option 1: Using JavaScript/TypeScript SDK

#### Installation

```bash
npm install @meshailabs/sdk
```

#### Create Your First Agent

```typescript
// first-agent.ts
import { MeshClient } from '@meshailabs/sdk';

async function createMyFirstAgent() {
  // Initialize the MeshAI client
  const client = await MeshClient.createAndInitialize({
    apiKey: process.env.MESHAI_API_KEY // Set your API key
  });

  // Execute a simple task
  const result = await client.quickExecute(
    'What are the key benefits of using multi-agent AI systems?',
    ['text-generation', 'analysis']
  );

  console.log('Agent Response:', result.result);
  
  // Clean up
  client.disconnect();
}

// Run the agent
createMyFirstAgent().catch(console.error);
```

#### Run Your Agent

```bash
# Set your API key
export MESHAI_API_KEY="your-api-key-here"

# Run with Node.js
node first-agent.js

# Or with TypeScript
px tsx first-agent.ts
```

### Option 2: Using Python SDK

#### Installation

```bash
pip install meshai-sdk
```

#### Create Your First Agent

```python
# first_agent.py
import asyncio
from meshai import MeshClient

async def create_my_first_agent():
    # Initialize the MeshAI client
    client = MeshClient(
        api_key="your-api-key-here"  # Or use os.environ['MESHAI_API_KEY']
    )
    
    # Execute a simple task
    result = await client.quick_execute(
        "What are the key benefits of using multi-agent AI systems?",
        required_capabilities=["text-generation", "analysis"]
    )
    
    print(f"Agent Response: {result['result']}")

# Run the agent
if __name__ == "__main__":
    asyncio.run(create_my_first_agent())
```

#### Run Your Agent

```bash
# Set your API key
export MESHAI_API_KEY="your-api-key-here"

# Run the agent
python first_agent.py
```

## Provider-Specific Agents

### OpenAI Agent (GPT-4)

```python
# openai_agent.py
from meshai.adapters import OpenAIMeshAgent
import os

class GPT4Agent(OpenAIMeshAgent):
    def __init__(self):
        super().__init__(
            model="gpt-4-turbo-preview",
            agent_id="gpt4-assistant",
            name="GPT-4 Assistant",
            capabilities=[
                "text-generation",
                "code-generation",
                "analysis",
                "creative-writing",
                "translation"
            ],
            openai_api_key=os.environ.get("OPENAI_API_KEY"),
            temperature=0.7,
            max_tokens=2000
        )
    
    async def process_with_functions(self, task):
        """Process task with OpenAI function calling"""
        functions = [
            {
                "name": "analyze_sentiment",
                "description": "Analyze the sentiment of text",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "text": {"type": "string"},
                        "language": {"type": "string"}
                    }
                }
            }
        ]
        
        return await self.complete_with_functions(
            messages=[{"role": "user", "content": task["input"]}],
            functions=functions
        )

# JavaScript/TypeScript version
```

```typescript
// openai-agent.ts
import { OpenAIMeshAgent } from '@meshailabs/sdk/adapters';

class GPT4Agent extends OpenAIMeshAgent {
  constructor() {
    super({
      model: 'gpt-4-turbo-preview',
      agentId: 'gpt4-assistant',
      name: 'GPT-4 Assistant',
      capabilities: [
        'text-generation',
        'code-generation',
        'analysis',
        'creative-writing',
        'translation'
      ],
      openaiApiKey: process.env.OPENAI_API_KEY,
      temperature: 0.7,
      maxTokens: 2000
    });
  }

  async processWithTools(task: any) {
    const tools = [{
      type: 'function',
      function: {
        name: 'analyze_data',
        description: 'Analyze structured data',
        parameters: {
          type: 'object',
          properties: {
            data: { type: 'array' },
            operation: { type: 'string' }
          }
        }
      }
    }];

    return await this.completeWithTools({
      messages: [{ role: 'user', content: task.input }],
      tools
    });
  }
}

// Usage
async function main() {
  const agent = new GPT4Agent();
  await agent.register(); // Register with MeshAI
  
  const result = await agent.execute({
    input: 'Generate a Python function to calculate fibonacci numbers',
    type: 'code-generation'
  });
  
  console.log('Generated code:', result);
}
```

### Anthropic Agent (Claude 3)

```python
# anthropic_agent.py
from meshai.adapters import AnthropicMeshAgent
import os

class Claude3Agent(AnthropicMeshAgent):
    def __init__(self):
        super().__init__(
            model="claude-3-opus-20240229",  # or claude-3-sonnet/haiku
            agent_id="claude3-analyst",
            name="Claude 3 Analyst",
            capabilities=[
                "deep-analysis",
                "research",
                "code-review",
                "academic-writing",
                "data-interpretation"
            ],
            anthropic_api_key=os.environ.get("ANTHROPIC_API_KEY"),
            max_tokens=4000
        )
    
    async def analyze_with_artifacts(self, content, artifact_type="document"):
        """Use Claude's artifact generation capabilities"""
        prompt = f"""Please analyze the following and create a {artifact_type}:
        
        {content}
        
        Provide structured output with clear sections.
        """
        
        response = await self.complete(
            messages=[{"role": "user", "content": prompt}],
            system="You are an expert analyst. Create detailed, well-structured artifacts."
        )
        
        return response

# JavaScript/TypeScript version
```

```typescript
// anthropic-agent.ts
import { AnthropicMeshAgent } from '@meshailabs/sdk/adapters';

class Claude3Agent extends AnthropicMeshAgent {
  constructor() {
    super({
      model: 'claude-3-opus-20240229', // or claude-3-sonnet-20240229
      agentId: 'claude3-analyst',
      name: 'Claude 3 Analyst',
      capabilities: [
        'deep-analysis',
        'research',
        'code-review',
        'academic-writing',
        'data-interpretation'
      ],
      anthropicApiKey: process.env.ANTHROPIC_API_KEY,
      maxTokens: 4000
    });
  }

  async performDeepAnalysis(topic: string, context?: any) {
    const systemPrompt = `You are an expert analyst with deep knowledge across multiple domains.
    Provide comprehensive, nuanced analysis with attention to detail.`;
    
    const result = await this.complete({
      messages: [{
        role: 'user',
        content: `Perform a deep analysis of: ${topic}\n\nContext: ${JSON.stringify(context)}`
      }],
      system: systemPrompt,
      temperature: 0.3 // Lower temperature for analytical tasks
    });
    
    return result;
  }
}

// Usage example
async function main() {
  const agent = new Claude3Agent();
  await agent.register();
  
  const analysis = await agent.performDeepAnalysis(
    'Impact of AI on software development',
    { focus: 'productivity', timeframe: '5 years' }
  );
  
  console.log('Analysis:', analysis);
}
```

### Google AI Agent (Gemini)

```python
# google_agent.py
from meshai.adapters import GoogleMeshAgent
import os

class GeminiAgent(GoogleMeshAgent):
    def __init__(self):
        super().__init__(
            model="gemini-pro",  # or gemini-pro-vision for multimodal
            agent_id="gemini-creative",
            name="Gemini Creative Assistant",
            capabilities=[
                "creative-writing",
                "brainstorming",
                "multimodal-analysis",
                "translation",
                "summarization"
            ],
            google_api_key=os.environ.get("GOOGLE_API_KEY"),
            generation_config={
                "temperature": 0.9,
                "top_p": 0.95,
                "top_k": 40,
                "max_output_tokens": 2048
            }
        )
    
    async def process_multimodal(self, text, image_path=None):
        """Process text with optional image for multimodal analysis"""
        if image_path:
            # Use gemini-pro-vision for image analysis
            self.model = "gemini-pro-vision"
            return await self.analyze_image_with_text(text, image_path)
        else:
            return await self.generate(text)
    
    async def creative_brainstorm(self, topic, num_ideas=5):
        """Generate creative ideas on a topic"""
        prompt = f"""Generate {num_ideas} creative and innovative ideas about: {topic}
        
        For each idea provide:
        1. A catchy title
        2. Brief description (2-3 sentences)
        3. Potential impact
        4. Implementation difficulty (Easy/Medium/Hard)
        """
        
        return await self.generate(
            prompt,
            generation_config={"temperature": 0.95}  # Higher creativity
        )
```

```typescript
// google-agent.ts
import { GoogleMeshAgent } from '@meshailabs/sdk/adapters';

class GeminiAgent extends GoogleMeshAgent {
  constructor() {
    super({
      model: 'gemini-pro', // or gemini-pro-vision
      agentId: 'gemini-creative',
      name: 'Gemini Creative Assistant',
      capabilities: [
        'creative-writing',
        'brainstorming',
        'multimodal-analysis',
        'translation',
        'summarization'
      ],
      googleApiKey: process.env.GOOGLE_API_KEY,
      generationConfig: {
        temperature: 0.9,
        topP: 0.95,
        topK: 40,
        maxOutputTokens: 2048
      }
    });
  }

  async processWithSafetySettings(content: string) {
    const safetySettings = [
      {
        category: 'HARM_CATEGORY_HARASSMENT',
        threshold: 'BLOCK_MEDIUM_AND_ABOVE'
      },
      {
        category: 'HARM_CATEGORY_HATE_SPEECH',
        threshold: 'BLOCK_MEDIUM_AND_ABOVE'
      }
    ];

    return await this.generate({
      contents: [{ parts: [{ text: content }] }],
      safetySettings,
      generationConfig: this.generationConfig
    });
  }

  async multimodalAnalysis(text: string, imageData?: Buffer) {
    if (!imageData) {
      return await this.generate(text);
    }

    // Switch to vision model for image analysis
    this.model = 'gemini-pro-vision';
    
    return await this.analyzeWithImage({
      text,
      image: {
        mimeType: 'image/jpeg',
        data: imageData.toString('base64')
      }
    });
  }
}
```

## Advanced Agent: Task-Specific Implementation

### Building a Customer Support Agent

Let's create a more sophisticated agent that can handle customer support tasks:

```typescript
// customer-support-agent.ts
import { MeshClient, RoutingStrategy, TaskStatus } from '@meshailabs/sdk';

class CustomerSupportAgent {
  private client: MeshClient;
  private conversationId: string;

  constructor(apiKey: string) {
    this.client = new MeshClient({
      apiKey,
      defaultRoutingStrategy: RoutingStrategy.CAPABILITY_MATCH,
      timeout: 30000
    });
    this.conversationId = `support-${Date.now()}`;
  }

  async initialize() {
    await this.client.initialize();
    console.log('Customer Support Agent initialized');
  }

  async handleInquiry(customerMessage: string) {
    // Create a task with context preservation
    const task = this.client.createTask(
      {
        message: customerMessage,
        type: 'customer_inquiry'
      },
      {
        taskType: 'support',
        requiredCapabilities: [
          'customer-service',
          'sentiment-analysis',
          'problem-solving'
        ],
        preserveContext: true,
        conversationId: this.conversationId,
        maxRetries: 2
      }
    );

    // Execute with real-time monitoring
    const result = await this.client.execute(task, {
      onProgress: (status, details) => {
        console.log(`Processing: ${status}`, details);
      }
    });

    return result.result;
  }

  async escalateToHuman(reason: string) {
    const task = this.client.createTask(
      {
        action: 'escalate',
        reason,
        conversationId: this.conversationId
      },
      {
        taskType: 'escalation',
        requiredCapabilities: ['ticket-creation', 'notification'],
        routingStrategy: RoutingStrategy.PERFORMANCE_BASED
      }
    );

    return await this.client.execute(task);
  }

  async getConversationSummary() {
    const history = await this.client.getTaskHistory(this.conversationId);
    return history;
  }

  disconnect() {
    this.client.disconnect();
  }
}

// Example usage
async function main() {
  const agent = new CustomerSupportAgent(process.env.MESHAI_API_KEY!);
  await agent.initialize();

  // Handle customer inquiries
  const responses = [
    await agent.handleInquiry("I can't log into my account"),
    await agent.handleInquiry("I've tried resetting my password but haven't received the email"),
    await agent.handleInquiry("This has been happening for 2 days now")
  ];

  responses.forEach((response, index) => {
    console.log(`Response ${index + 1}:`, response);
  });

  // Get conversation summary
  const summary = await agent.getConversationSummary();
  console.log('Conversation Summary:', summary);

  // Check if escalation is needed
  if (responses.some(r => r.includes('escalate'))) {
    await agent.escalateToHuman('Customer issue unresolved after multiple attempts');
  }

  agent.disconnect();
}

main().catch(console.error);
```

## Multi-Provider Orchestration

### Combining Multiple AI Providers

```typescript
// multi-provider-orchestration.ts
import { MeshClient } from '@meshailabs/sdk';
import { OpenAIMeshAgent } from '@meshailabs/sdk/adapters/openai';
import { AnthropicMeshAgent } from '@meshailabs/sdk/adapters/anthropic';
import { GoogleMeshAgent } from '@meshailabs/sdk/adapters/google';

class MultiProviderOrchestrator {
  private client: MeshClient;
  private agents: Map<string, any>;

  constructor() {
    this.client = new MeshClient({
      apiKey: process.env.MESHAI_API_KEY
    });
    this.agents = new Map();
  }

  async initialize() {
    await this.client.initialize();

    // Register different provider agents
    const openaiAgent = new OpenAIMeshAgent({
      model: 'gpt-4-turbo-preview',
      agentId: 'openai-coder',
      capabilities: ['code-generation', 'debugging']
    });

    const anthropicAgent = new AnthropicMeshAgent({
      model: 'claude-3-opus-20240229',
      agentId: 'anthropic-reviewer',
      capabilities: ['code-review', 'analysis']
    });

    const googleAgent = new GoogleMeshAgent({
      model: 'gemini-pro',
      agentId: 'google-documenter',
      capabilities: ['documentation', 'summarization']
    });

    // Register all agents
    await Promise.all([
      this.client.registerAgent(openaiAgent.toRegistration()),
      this.client.registerAgent(anthropicAgent.toRegistration()),
      this.client.registerAgent(googleAgent.toRegistration())
    ]);

    this.agents.set('coder', openaiAgent);
    this.agents.set('reviewer', anthropicAgent);
    this.agents.set('documenter', googleAgent);
  }

  async developFeature(requirements: string) {
    console.log('Starting multi-provider development workflow...');

    // Step 1: Generate code with OpenAI
    const codeGenTask = this.client.createTask(
      { requirements, action: 'generate_code' },
      { requiredCapabilities: ['code-generation'] }
    );
    const code = await this.client.execute(codeGenTask);
    console.log('Code generated by OpenAI GPT-4');

    // Step 2: Review code with Anthropic Claude
    const reviewTask = this.client.createTask(
      { code: code.result, action: 'review_code' },
      { requiredCapabilities: ['code-review'] }
    );
    const review = await this.client.execute(reviewTask);
    console.log('Code reviewed by Anthropic Claude 3');

    // Step 3: Generate documentation with Google Gemini
    const docTask = this.client.createTask(
      { code: code.result, review: review.result, action: 'document' },
      { requiredCapabilities: ['documentation'] }
    );
    const documentation = await this.client.execute(docTask);
    console.log('Documentation created by Google Gemini');

    return {
      code: code.result,
      review: review.result,
      documentation: documentation.result
    };
  }
}

// Usage
async function main() {
  const orchestrator = new MultiProviderOrchestrator();
  await orchestrator.initialize();

  const result = await orchestrator.developFeature(
    'Create a REST API endpoint for user authentication with JWT'
  );

  console.log('Complete feature development:', result);
}

main().catch(console.error);
```

## Framework-Specific Agents

### LangChain Agent

```python
# langchain_agent.py
from meshai.adapters import LangChainMeshAgent
from langchain.tools import Tool
from langchain.agents import initialize_agent, AgentType

# Create a MeshAI-compatible LangChain agent
class MyLangChainAgent(LangChainMeshAgent):
    def __init__(self):
        super().__init__(
            agent_id="langchain-analyst",
            name="Data Analyst Agent",
            capabilities=["data-analysis", "visualization", "reporting"]
        )
        
        # Define LangChain tools
        self.tools = [
            Tool(
                name="analyze_data",
                func=self.analyze_data,
                description="Analyze datasets and extract insights"
            ),
            Tool(
                name="create_visualization",
                func=self.create_visualization,
                description="Create data visualizations"
            )
        ]
        
    async def analyze_data(self, query: str):
        # Your data analysis logic here
        return f"Analysis complete for: {query}"
    
    async def create_visualization(self, data: str):
        # Your visualization logic here
        return f"Visualization created for: {data}"

# Register and use the agent
async def main():
    agent = MyLangChainAgent()
    
    # Register with MeshAI
    from meshai import MeshRegistry
    registry = MeshRegistry()
    await registry.register_agent(agent)
    
    # Agent is now available for orchestration
    print(f"Agent {agent.agent_id} registered and ready")

```

### CrewAI Agent

```python
# crewai_agent.py
from meshai.adapters import CrewAIMeshAgent
from crewai import Agent, Task, Crew

class MyCrewAgent(CrewAIMeshAgent):
    def __init__(self):
        # Define CrewAI agents
        researcher = Agent(
            role='Research Analyst',
            goal='Gather and analyze information',
            backstory='Expert at finding and synthesizing information'
        )
        
        writer = Agent(
            role='Content Writer',
            goal='Create compelling content',
            backstory='Skilled at creating engaging content'
        )
        
        # Create crew
        crew = Crew(
            agents=[researcher, writer],
            verbose=True
        )
        
        super().__init__(
            crew=crew,
            agent_id="crewai-content-team",
            name="Content Creation Crew",
            capabilities=["research", "content-creation", "writing"]
        )

# Use the crew agent
async def main():
    agent = MyCrewAgent()
    
    # Execute a task
    result = await agent.execute_task({
        "input": "Write a blog post about AI agent orchestration",
        "context": {"tone": "professional", "length": "1000 words"}
    })
    
    print(f"Result: {result}")
```

## Deploying Your Agent

### Step 1: Test Locally

```bash
# Test your agent locally
npm test  # or python -m pytest
```

### Step 2: Register Your Agent

```typescript
// register-agent.ts
import { MeshClient } from '@meshailabs/sdk';

async function registerAgent() {
  const client = await MeshClient.createAndInitialize();
  
  const agentInfo = await client.registerAgent({
    id: 'my-first-agent',
    name: 'My First Agent',
    framework: 'custom',
    capabilities: ['text-generation', 'analysis'],
    endpoint: 'https://my-agent.example.com/execute',
    metadata: {
      version: '1.0.0',
      author: 'Your Name'
    }
  });
  
  console.log('Agent registered:', agentInfo);
}

registerAgent().catch(console.error);
```

### Step 3: Monitor Your Agent

```typescript
// monitor-agent.ts
import { MeshClient } from '@meshailabs/sdk';

async function monitorAgent() {
  const client = await MeshClient.createAndInitialize();
  const agentId = 'my-first-agent';
  
  // Check health
  const health = await client.checkAgentHealth(agentId);
  console.log('Agent health:', health);
  
  // Get metrics
  const metrics = await client.getAgentMetrics(agentId);
  console.log('Agent metrics:', metrics);
  
  // Send heartbeat
  setInterval(async () => {
    await client.sendHeartbeat(agentId);
    console.log('Heartbeat sent');
  }, 30000); // Every 30 seconds
}

monitorAgent().catch(console.error);
```

## Best Practices

### 1. Error Handling

```typescript
import { MeshError, TaskExecutionError, TimeoutError } from '@meshailabs/sdk';

try {
  const result = await client.execute(task);
} catch (error) {
  if (error instanceof TimeoutError) {
    console.error('Task timed out, retrying...');
    // Implement retry logic
  } else if (error instanceof TaskExecutionError) {
    console.error('Task failed:', error.taskId, error.message);
    // Handle task failure
  } else {
    console.error('Unexpected error:', error);
  }
}
```

### 2. Context Management

```typescript
// Maintain context across interactions
const conversationId = `session-${Date.now()}`;

const tasks = [
  { input: "What is MeshAI?" },
  { input: "How does it work?" },
  { input: "Can you give me an example?" }
];

for (const taskData of tasks) {
  const task = client.createTask(taskData.input, {
    preserveContext: true,
    conversationId
  });
  
  const result = await client.execute(task);
  console.log(result.result);
}
```

### 3. Performance Optimization

```typescript
// Use appropriate routing strategies
const strategies = {
  simple: RoutingStrategy.ROUND_ROBIN,           // For simple tasks
  complex: RoutingStrategy.CAPABILITY_MATCH,     // For specialized tasks
  critical: RoutingStrategy.PERFORMANCE_BASED,   // For important tasks
  contextual: RoutingStrategy.STICKY_SESSION,    // For conversational tasks
  budget: RoutingStrategy.COST_OPTIMIZED        // For cost-sensitive tasks
};
```

## Troubleshooting

### Common Issues

1. **Authentication Error**
   ```
   Error: Invalid API key
   ```
   **Solution**: Verify your API key is correct and active in the admin dashboard.

2. **No Agents Available**
   ```
   Error: No agents found matching capabilities
   ```
   **Solution**: Check that agents with required capabilities are registered and active.

3. **Timeout Issues**
   ```
   Error: Task execution timed out
   ```
   **Solution**: Increase timeout or optimize task complexity.

### Debug Mode

```typescript
// Enable debug logging
const client = new MeshClient({
  apiKey: process.env.MESHAI_API_KEY,
  logLevel: 'debug'
});
```

## Next Steps

1. **Explore Advanced Features**
   - [WebSocket real-time monitoring](./websocket-guide.md)
   - [Multi-agent workflows](./multi-agent-workflows.md)
   - [Custom routing strategies](./routing-strategies.md)

2. **Integrate with Your Stack**
   - [REST API integration](./rest-api-guide.md)
   - [Webhook setup](./webhooks.md)
   - [Event-driven architecture](./events.md)

3. **Scale Your Agents**
   - [Performance optimization](./performance.md)
   - [Load balancing](./load-balancing.md)
   - [High availability](./high-availability.md)

## Resources

- **Documentation**: [docs.meshai.dev](https://docs.meshai.dev)
- **API Reference**: [api.meshai.dev/docs](https://api.meshai.dev/docs)
- **GitHub**: [github.com/meshailabs-org](https://github.com/meshailabs-org)
- **Support**: [support.meshai.dev](https://support.meshai.dev)
- **Discord Community**: [discord.gg/meshai](https://discord.gg/meshai)

## Summary

You've successfully created and deployed your first AI agent with MeshAI! You've learned:

- How to quickly create agents using JavaScript/TypeScript or Python SDKs
- Building provider-specific agents (OpenAI, Anthropic, Google)
- Orchestrating multiple AI providers for complex workflows
- Integrating with popular frameworks like LangChain and CrewAI
- Best practices for error handling, context management, and performance
- How to deploy, register, and monitor your agents

Continue exploring MeshAI's capabilities to build powerful multi-agent AI systems that can tackle complex tasks through intelligent orchestration.

---

*Last updated: September 2025*