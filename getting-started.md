# Getting Started with MeshAI

**MeshAI** is a universal AI agent orchestration platform that connects agents from 15+ frameworks and providers into one unified mesh. This guide shows you exactly how to start using MeshAI to submit tasks and orchestrate AI agents.

## 🚀 Quick Start (5 minutes)

### Prerequisites

1. **Create a MeshAI Account**
   - Visit https://app.meshai.dev
   - Sign up for an account or contact support@meshai.dev for access
   
2. **Get Your API Key**
   - Login to the dashboard
   - Navigate to the "API Keys" tab
   - Click "+ Create API Key"
   - Copy and save your key securely

### Option 1: Web Dashboard (Easiest)

1. **Access the Admin Dashboard**
   - Go to: https://app.meshai.dev
   - Login with your MeshAI account

2. **Submit Your First Task**
   - Click the **"Tasks"** tab
   - Click **"+ Create Task"** button
   - Fill in your task details:
     - **Task Type**: `chat`
     - **Message**: `"Explain quantum computing in simple terms"`
     - **Capabilities**: Select `chat` and `explanation`
   - Click **"Submit Task"**
   - Watch it get routed to the best available agent!

3. **View Results**
   - See your task status update in real-time
   - View which agent handled your request
   - Check the response from the AI agent

### Option 2: API Call (For Developers)

```bash
# Submit a task via API (requires API key)
curl -X POST "https://runtime.meshai.dev/api/v1/tasks" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "task_type": "chat",
    "payload": {
      "message": "Hello MeshAI! Explain how you work."
    },
    "required_capabilities": ["chat"],
    "routing_strategy": "capability_match"
  }'
```

**Response:**
```json
{
  "task_id": "task-1234567890",
  "status": "submitted",
  "assigned_agent": "best-available-chat-agent"
}
```

## 📚 Core Concepts

### **Tasks**
A **task** is any request you want an AI agent to handle. Tasks include:
- Chat conversations
- Code generation
- Data analysis  
- Content creation
- Image processing
- Research queries

### **Capabilities** 
**Capabilities** describe what an agent can do:
- `chat` - Conversational AI
- `code-generation` - Writing code
- `analysis` - Data/text analysis
- `reasoning` - Complex reasoning tasks
- `search` - Web search and research
- `multimodal` - Text + image processing

### **Routing Strategies**
MeshAI uses different strategies to choose the best agent:
- `capability_match` - Routes to agents with required capabilities
- `performance_based` - Routes to fastest/most reliable agents
- `load_balanced` - Distributes tasks evenly across agents

## 🤖 MCP (Model Context Protocol) Integration

MeshAI provides a Model Context Protocol server that allows AI assistants like Claude to interact directly with your agents:

- **MCP Server**: https://mcp.meshai.dev
- **Documentation**: [MCP Integration Guide](mcp-integration.md)

Use the MCP server to execute tasks, query agents, and manage workflows directly from your AI conversations.

## 🛠 Usage Methods

### Method 1: Web Dashboard

**Best for:** Non-technical users, testing, monitoring

1. **Login to Dashboard**: https://app.meshai.dev
2. **Navigate to Tasks Tab**
3. **Click "+ Create Task"**
4. **Fill Task Details:**
   - Task Type (chat, analysis, code_generation, etc.)
   - Payload (your actual request/data)
   - Required Capabilities
5. **Submit and Monitor**

### Method 2: REST API

**Best for:** Applications, automation, integrations

#### Basic Task Submission
```bash
POST https://runtime.meshai.dev/api/v1/tasks
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "task_type": "analysis",
  "payload": {
    "text": "The stock market has been volatile lately",
    "analysis_type": "sentiment"
  },
  "required_capabilities": ["analysis", "reasoning"],
  "routing_strategy": "capability_match"
}
```

#### Check Task Status
```bash
GET https://runtime.meshai.dev/api/v1/tasks
Authorization: Bearer YOUR_API_KEY
```

#### Get All Available Agents
```bash
GET https://api.meshai.dev/api/v1/agents
Authorization: Bearer YOUR_API_KEY
```

### Method 3: Python Client

**Best for:** Python applications, data science, automation

```python
import requests
import json

class MeshAIClient:
    def __init__(self, api_key):
        self.api_key = api_key
        self.runtime_url = "https://runtime.meshai.dev"
        self.registry_url = "https://api.meshai.dev"
        self.headers = {
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        }
    
    def submit_task(self, task_type, payload, capabilities=None, routing="capability_match"):
        """Submit a task to MeshAI"""
        task_data = {
            "task_type": task_type,
            "payload": payload,
            "required_capabilities": capabilities or ["chat"],
            "routing_strategy": routing
        }
        
        response = requests.post(
            f"{self.runtime_url}/api/v1/tasks", 
            json=task_data,
            headers=self.headers
        )
        return response.json()
    
    def get_tasks(self):
        """Get all tasks"""
        response = requests.get(
            f"{self.runtime_url}/api/v1/tasks",
            headers=self.headers
        )
        return response.json()
    
    def get_agents(self):
        """Get all registered agents"""
        response = requests.get(
            f"{self.registry_url}/api/v1/agents",
            headers=self.headers
        )
        return response.json()

# Usage example
client = MeshAIClient(api_key="YOUR_API_KEY")

# Submit a chat task
result = client.submit_task(
    task_type="chat",
    payload={"message": "Explain machine learning in simple terms"},
    capabilities=["chat", "explanation"]
)
print(f"Task submitted: {result['task_id']}")

# Submit a code generation task
result = client.submit_task(
    task_type="code_generation", 
    payload={
        "requirements": "Create a Python function to calculate Fibonacci numbers",
        "language": "python"
    },
    capabilities=["code-generation", "python"]
)
print(f"Code task: {result['task_id']}")
```

### Method 4: JavaScript/Web Apps

**Best for:** Web applications, frontend integrations

```javascript
class MeshAIClient {
    constructor(apiKey) {
        this.apiKey = apiKey;
        this.runtimeUrl = 'https://runtime.meshai.dev';
        this.registryUrl = 'https://api.meshai.dev';
        this.headers = {
            'Authorization': `Bearer ${apiKey}`,
            'Content-Type': 'application/json'
        };
    }
    
    async submitTask(taskType, payload, capabilities = ["chat"], routing = "capability_match") {
        const taskData = {
            task_type: taskType,
            payload: payload,
            required_capabilities: capabilities,
            routing_strategy: routing
        };
        
        try {
            const response = await fetch(`${this.runtimeUrl}/api/v1/tasks`, {
                method: 'POST',
                headers: this.headers,
                body: JSON.stringify(taskData)
            });
            
            return await response.json();
        } catch (error) {
            console.error('Error submitting task:', error);
            throw error;
        }
    }
    
    async getTasks() {
        const response = await fetch(`${this.runtimeUrl}/api/v1/tasks`, {
            headers: this.headers
        });
        return await response.json();
    }
}

// Usage
const client = new MeshAIClient('YOUR_API_KEY');

// Submit task
const result = await client.submitTask("chat", {
    message: "What are the benefits of AI orchestration?"
}, ["chat", "explanation"]);

console.log('Task result:', result);
```

## 🎯 Common Use Cases

### 1. Customer Support Automation

```python
def handle_customer_query(user_message, context, api_key):
    client = MeshAIClient(api_key)
    
    # Determine appropriate capabilities
    if "code" in user_message.lower() or "api" in user_message.lower():
        capabilities = ["code-generation", "debugging", "technical-support"]
    elif "billing" in user_message.lower():
        capabilities = ["data-analysis", "calculation", "billing"]
    else:
        capabilities = ["chat", "customer-service"]
    
    # Submit to MeshAI
    result = client.submit_task(
        task_type="customer_support",
        payload={
            "message": user_message,
            "context": context,
            "priority": "normal"
        },
        capabilities=capabilities
    )
    
    return result

# Usage
response = handle_customer_query(
    "My API is returning 401 errors", 
    {"user_id": "12345", "plan": "enterprise"}
)
```

### 2. Content Creation Pipeline

```python
def create_blog_post(topic, api_key):
    client = MeshAIClient(api_key)
    
    # Step 1: Research the topic
    research = client.submit_task("research", {
        "topic": topic,
        "depth": "comprehensive",
        "sources": ["web", "academic"]
    }, ["search", "research"])
    
    # Step 2: Create outline
    outline = client.submit_task("planning", {
        "research_data": research,
        "content_type": "blog_post",
        "audience": "developers"
    }, ["planning", "writing"])
    
    # Step 3: Write content
    content = client.submit_task("writing", {
        "outline": outline,
        "tone": "professional",
        "length": "1500_words"
    }, ["writing", "creativity"])
    
    # Step 4: Generate code examples
    code = client.submit_task("code_generation", {
        "context": content,
        "language": "python",
        "examples": 3
    }, ["code-generation", "documentation"])
    
    return {
        "content": content,
        "code": code,
        "outline": outline
    }
```

### 3. Data Analysis Workflow

```python
def analyze_data(dataset, api_key):
    client = MeshAIClient(api_key)
    
    # Statistical analysis
    stats = client.submit_task("analysis", {
        "data": dataset,
        "analysis_types": ["descriptive", "correlation", "trend"]
    }, ["analysis", "statistics"])
    
    # Generate visualizations
    charts = client.submit_task("visualization", {
        "data": stats,
        "chart_types": ["line", "bar", "scatter"],
        "library": "matplotlib"
    }, ["code-generation", "visualization"])
    
    # Business insights
    insights = client.submit_task("business_analysis", {
        "analysis": stats,
        "context": "quarterly_review"
    }, ["reasoning", "business-analysis"])
    
    return {
        "statistics": stats,
        "visualizations": charts,
        "insights": insights
    }
```

## 🔧 Adding Your Own Agents

**Note:** To register agents, you need an API key from your MeshAI dashboard.

### Register an OpenAI Agent

```python
agent_data = {
    "id": "my-chatgpt-agent",
    "name": "My ChatGPT Assistant",
    "framework": "openai-assistants",
    "provider": "OpenAI", 
    "capabilities": ["chat", "code-generation", "function-calling"],
    "endpoint": "https://api.openai.com/v1/chat/completions",
    "metadata": {
        "model": "gpt-4",
        "api_key": "your-api-key-here"
    }
}

response = requests.post(
    "https://api.meshai.dev/api/v1/agents",
    headers={
        "Authorization": f"Bearer {YOUR_API_KEY}",
        "Content-Type": "application/json"
    },
    json=agent_data
)
```

### Register a Claude Agent

```python
claude_agent = {
    "id": "my-claude-agent",
    "name": "Claude Reasoning Assistant", 
    "framework": "anthropic-claude",
    "provider": "Anthropic",
    "capabilities": ["reasoning", "analysis", "writing", "safety"],
    "endpoint": "https://api.anthropic.com/v1/messages",
    "metadata": {
        "model": "claude-3-sonnet-20240229",
        "api_key": "your-anthropic-key"
    }
}
```

### Register a Custom Agent

```python
custom_agent = {
    "id": "my-custom-agent",
    "name": "My Specialized Agent",
    "framework": "custom",
    "provider": "My Company",
    "capabilities": ["domain-specific", "custom-analysis"],
    "endpoint": "https://my-api.com/agent",
    "metadata": {
        "version": "1.0.0",
        "specialization": "financial_analysis"
    }
}
```

## 📊 Monitoring and Management

### Check Task Status

```python
# Get all tasks
tasks = client.get_tasks()
print(f"Total tasks: {len(tasks)}")

# Check recent tasks
for task in tasks[-5:]:
    print(f"Task {task['task_id']}: {task['status']} - {task['task_type']}")

# Filter by status
active_tasks = [t for t in tasks if t['status'] in ['submitted', 'running']]
completed_tasks = [t for t in tasks if t['status'] == 'completed']
```

### Monitor Agents

```python
# Get all registered agents
agents = client.get_agents()
print(f"Total agents: {len(agents)}")

# Check agent health
active_agents = [a for a in agents if a['status'] == 'active']
print(f"Active agents: {len(active_agents)}")

# Group by provider
from collections import defaultdict
providers = defaultdict(list)
for agent in agents:
    providers[agent['provider']].append(agent['name'])

for provider, agent_names in providers.items():
    print(f"{provider}: {len(agent_names)} agents")
```

## 🌟 Best Practices

### 1. Capability Matching
- **Be Specific**: Use precise capabilities like `["python", "data-analysis"]` instead of just `["coding"]`
- **Multiple Options**: List multiple capabilities to give MeshAI routing flexibility
- **Context Matters**: Include domain-specific capabilities like `["financial-analysis", "medical-text"]`

### 2. Task Structure
```python
# Good task structure
task = {
    "task_type": "analysis",  # Clear category
    "payload": {
        "data": "your_data_here",
        "format": "json",        # Specify expected output format
        "context": "financial",  # Provide domain context
        "requirements": ["summary", "insights", "recommendations"]
    },
    "required_capabilities": ["financial-analysis", "reasoning", "report-generation"],
    "routing_strategy": "performance_based"  # Choose appropriate strategy
}
```

### 3. Error Handling
```python
def safe_submit_task(client, task_type, payload, capabilities):
    try:
        result = client.submit_task(task_type, payload, capabilities)
        return result
    except requests.exceptions.RequestException as e:
        print(f"Network error: {e}")
        return None
    except Exception as e:
        print(f"Unexpected error: {e}")
        return None

# Usage with retry logic
import time

def submit_with_retry(client, task_data, max_retries=3):
    for attempt in range(max_retries):
        result = safe_submit_task(client, **task_data)
        if result:
            return result
        
        print(f"Retry {attempt + 1}/{max_retries}")
        time.sleep(2 ** attempt)  # Exponential backoff
    
    return None
```

## 🚀 Next Steps

1. **✅ Try the Web Dashboard**: Start with the visual interface
2. **✅ Test API Calls**: Use curl or Postman to test the API
3. **✅ Build a Simple Client**: Create a Python script using the examples
4. **✅ Register Your Agents**: Add your existing AI services to MeshAI
5. **✅ Create Workflows**: Chain multiple tasks together
6. **✅ Monitor Performance**: Use the dashboard to track your agents and tasks
7. **✅ Scale to Production**: Integrate MeshAI into your applications

## 📞 Support

- **Admin Dashboard**: https://app.meshai.dev
- **API Documentation**: https://api.meshai.dev/docs
- **MCP Server**: https://mcp.meshai.dev
- **GitHub**: https://github.com/meshailabs-org
- **Discord Community**: https://discord.gg/meshai
- **Email Support**: support@meshai.dev

## 🎯 Key Benefits

- **🔄 Multi-Provider**: Use ChatGPT, Claude, Gemini, and custom agents together
- **🎯 Smart Routing**: Automatically route tasks to the best agent
- **📊 Unified Monitoring**: See all your AI agents in one dashboard
- **🔗 Framework Agnostic**: Works with LangChain, CrewAI, and any REST API
- **💰 Cost Optimization**: Route simple tasks to cheaper models
- **🔒 No Lock-in**: Switch providers without changing your code

Start with the web dashboard, then move to API integration as you scale! 🚀

## Legacy SDK Documentation

For advanced users building framework adapters, see:
- [Technical Specification](../overview/meshai_tech_spec.md)
- [Framework Adapters](../development/meshai_sdk_architecture.md)
- [Security & Authentication](../development/meshai_security_auth.md)
