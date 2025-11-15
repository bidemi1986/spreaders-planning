N8N Infrastructure & MCP Integration Architecture
==================================================

## Overview

This document outlines the complete infrastructure needed to support n8n workflow automation triggered via Model Context Protocol (MCP) from user prompts. It covers all supporting microservices, provisioning systems, deployment automation, and MCP integration layers.

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                     FRONTEND LAYER                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│  │   Web App    │  │  Mobile App  │  │   AI Chat    │         │
│  │  (React)     │  │  (RN/Swift)  │  │  Interface   │         │
│  └──────────────┘  └──────────────┘  └──────────────┘         │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY LAYER                           │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │  Kong API Gateway / AWS API Gateway                       │ │
│  │  - Authentication & Authorization                         │ │
│  │  - Rate Limiting                                          │ │
│  │  - Request Routing                                        │ │
│  │  - Load Balancing                                         │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   MCP ORCHESTRATION LAYER                        │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │              MCP Server (Main Entry Point)                │ │
│  │  - Tool Discovery & Registration                          │ │
│  │  - Request Parsing & Validation                           │ │
│  │  - Response Formatting                                    │ │
│  │  - Error Handling                                         │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                 CORE MICROSERVICES LAYER                         │
│                                                                   │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │  AI Planner        │  │  N8N Orchestrator  │                │
│  │  Service           │  │  Service           │                │
│  │  (Python/FastAPI)  │  │  (Node.js/NestJS)  │                │
│  └────────────────────┘  └────────────────────┘                │
│                                                                   │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │  Workflow Registry │  │  Provisioning      │                │
│  │  Service           │  │  Service           │                │
│  │  (Node.js/Express) │  │  (Go)              │                │
│  └────────────────────┘  └────────────────────┘                │
│                                                                   │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │  Credential Vault  │  │  Execution Monitor │                │
│  │  Service           │  │  Service           │                │
│  │  (Go/Vault)        │  │  (Python/FastAPI)  │                │
│  └────────────────────┘  └────────────────────┘                │
│                                                                   │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │  User Management   │  │  Campaign          │                │
│  │  Service           │  │  Management        │                │
│  │  (Node.js)         │  │  Service           │                │
│  └────────────────────┘  └────────────────────┘                │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                   N8N EXECUTION LAYER                            │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           N8N Instance Manager (Kubernetes)              │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │  │
│  │  │  N8N     │  │  N8N     │  │  N8N     │              │  │
│  │  │ Pod 1    │  │ Pod 2    │  │ Pod N    │              │  │
│  │  │(User A)  │  │(User B)  │  │(User X)  │              │  │
│  │  └──────────┘  └──────────┘  └──────────┘              │  │
│  │                                                            │  │
│  │  ┌──────────────────────────────────────┐               │  │
│  │  │   Shared N8N Pool (Free Tier)        │               │  │
│  │  │   ┌────┐ ┌────┐ ┌────┐ ┌────┐       │               │  │
│  │  │   │N8N │ │N8N │ │N8N │ │N8N │       │               │  │
│  │  │   └────┘ └────┘ └────┘ └────┘       │               │  │
│  │  └──────────────────────────────────────┘               │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                     DATA & STORAGE LAYER                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │ PostgreSQL │  │  MongoDB   │  │   Redis    │               │
│  │(Workflows, │  │(Execution  │  │  (Cache,   │               │
│  │ Users)     │  │ Logs)      │  │  Sessions) │               │
│  └────────────┘  └────────────┘  └────────────┘               │
│                                                                   │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │    S3      │  │ TimescaleDB│  │ RabbitMQ   │               │
│  │  (Media,   │  │ (Metrics)  │  │ (Message   │               │
│  │  Assets)   │  │            │  │  Queue)    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│              MONITORING & OBSERVABILITY LAYER                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐               │
│  │ Prometheus │  │  Grafana   │  │    ELK     │               │
│  │ (Metrics)  │  │(Dashboard) │  │  (Logs)    │               │
│  └────────────┘  └────────────┘  └────────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

---

## Core Microservices (Required)

### 1. MCP Server Service

**Purpose**: Main entry point that exposes all system capabilities via Model Context Protocol.

**Responsibilities**:
- Register all available MCP tools/functions
- Parse incoming MCP requests from AI agents
- Route requests to appropriate microservices
- Format responses in MCP standard format
- Handle authentication & authorization
- Provide tool discovery endpoint

**Technology Stack**: Node.js/TypeScript, Express/Fastify

**MCP Tools Exposed**:

```typescript
// Campaign Management Tools
{
  "name": "campaign.createFromPrompt",
  "description": "Create campaign from natural language prompt",
  "inputSchema": {
    "type": "object",
    "properties": {
      "prompt": { "type": "string" },
      "userId": { "type": "string" }
    },
    "required": ["prompt", "userId"]
  }
}

// Workflow Discovery Tools
{
  "name": "workflow.search",
  "description": "Search available n8n workflows",
  "inputSchema": {
    "type": "object",
    "properties": {
      "requirements": { "type": "array" },
      "platforms": { "type": "array" },
      "category": { "type": "string" }
    }
  }
}

// Execution Tools
{
  "name": "workflow.execute",
  "description": "Execute selected n8n workflow",
  "inputSchema": {
    "type": "object",
    "properties": {
      "workflowId": { "type": "string" },
      "parameters": { "type": "object" },
      "userId": { "type": "string" }
    },
    "required": ["workflowId", "userId"]
  }
}

// Credential Tools
{
  "name": "credential.connect",
  "description": "Connect user credential to platform",
  "inputSchema": {
    "type": "object",
    "properties": {
      "platform": { "type": "string" },
      "credentialType": { "type": "string" },
      "userId": { "type": "string" }
    },
    "required": ["platform", "userId"]
  }
}

// Monitoring Tools
{
  "name": "execution.getStatus",
  "description": "Get workflow execution status",
  "inputSchema": {
    "type": "object",
    "properties": {
      "executionId": { "type": "string" }
    },
    "required": ["executionId"]
  }
}
```

**API Structure**:
```typescript
// MCP Server Implementation
import { MCPServer } from '@modelcontextprotocol/sdk';

class SpreadersMCPServer extends MCPServer {
  constructor() {
    super({
      name: "spreaders-automation",
      version: "1.0.0"
    });
    
    this.registerTools();
  }
  
  private registerTools() {
    // Register all MCP tools
    this.addTool({
      name: "campaign.createFromPrompt",
      description: "Create marketing campaign from natural language",
      handler: this.handleCampaignCreate.bind(this)
    });
    
    this.addTool({
      name: "workflow.search",
      description: "Search n8n workflow templates",
      handler: this.handleWorkflowSearch.bind(this)
    });
    
    // ... register all other tools
  }
  
  private async handleCampaignCreate(params: any) {
    // 1. Call AI Planner Service
    const plan = await this.aiPlannerService.analyzeCampaign(params.prompt);
    
    // 2. Select workflows
    const workflows = await this.registryService.selectWorkflows(plan);
    
    // 3. Return plan to AI agent
    return {
      campaignPlan: plan,
      selectedWorkflows: workflows,
      nextStep: "credential_collection"
    };
  }
}
```

**Deployment**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcp-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mcp-server
  template:
    metadata:
      labels:
        app: mcp-server
    spec:
      containers:
      - name: mcp-server
        image: spreaders/mcp-server:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

---

### 2. AI Planner Service

**Purpose**: Analyze user prompts and create campaign execution plans.

**Responsibilities**:
- Natural language understanding (NLU)
- Extract campaign requirements (platforms, goals, timeline)
- Generate structured campaign plans
- Map requirements to workflow capabilities
- Suggest optimizations

**Technology Stack**: Python, FastAPI, OpenAI API, LangChain

**Core Functions**:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from langchain import PromptTemplate, LLMChain
from langchain.chat_models import ChatOpenAI

app = FastAPI()

class CampaignPrompt(BaseModel):
    prompt: str
    userId: str
    context: dict = {}

class CampaignPlan(BaseModel):
    campaign_type: str
    objectives: list[str]
    platforms: list[str]
    content_requirements: list[str]
    timeline: dict
    budget: dict = {}
    target_audience: dict
    required_workflows: list[str]
    confidence: float

@app.post("/analyze")
async def analyze_campaign(prompt: CampaignPrompt) -> CampaignPlan:
    """
    Analyze user prompt and generate campaign plan
    """
    # 1. Use LLM to extract structured data
    llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    analysis_prompt = PromptTemplate(
        input_variables=["prompt"],
        template="""
        Analyze this marketing campaign request and extract:
        - Campaign type (product_launch, promotion, brand_awareness, etc.)
        - Objectives (what user wants to achieve)
        - Platforms mentioned (social media, email, ads, etc.)
        - Content needs (images, videos, copy, etc.)
        - Timeline (when to start/end)
        - Target audience
        - Budget (if mentioned)
        
        User request: {prompt}
        
        Return as JSON.
        """
    )
    
    chain = LLMChain(llm=llm, prompt=analysis_prompt)
    result = chain.run(prompt=prompt.prompt)
    
    # 2. Parse and validate
    plan = parse_campaign_plan(result)
    
    # 3. Map to workflow requirements
    plan.required_workflows = map_to_workflows(plan)
    
    # 4. Calculate confidence score
    plan.confidence = calculate_confidence(plan, prompt.prompt)
    
    return plan

def map_to_workflows(plan: CampaignPlan) -> list[str]:
    """
    Map campaign requirements to workflow capabilities
    """
    required_workflows = []
    
    # Content generation needed?
    if "images" in plan.content_requirements:
        required_workflows.append("content_generation_image")
    if "videos" in plan.content_requirements:
        required_workflows.append("content_generation_video")
    
    # Social media publishing?
    if any(p in plan.platforms for p in ["instagram", "facebook", "tiktok"]):
        required_workflows.append("social_publishing")
    
    # Email campaign?
    if "email" in plan.platforms:
        required_workflows.append("email_campaign")
    
    # Paid ads?
    if "ads" in plan.platforms or "advertising" in plan.objectives:
        required_workflows.append("ad_campaign")
    
    # Analytics tracking
    required_workflows.append("analytics_tracking")
    
    return required_workflows

@app.post("/suggest-improvements")
async def suggest_improvements(plan: CampaignPlan):
    """
    Suggest improvements to campaign plan
    """
    llm = ChatOpenAI(model="gpt-4")
    
    suggestions = llm.predict(f"""
    Given this campaign plan, suggest 3-5 improvements:
    {plan.dict()}
    
    Consider:
    - Missing platforms that could help
    - Content types that perform well
    - Timing optimizations
    - Budget allocation
    """)
    
    return {"suggestions": suggestions}
```

**MCP Integration**:
```python
@app.post("/mcp/tools/campaign.analyze")
async def mcp_analyze_campaign(request: dict):
    """
    MCP-compatible endpoint
    """
    try:
        result = await analyze_campaign(
            CampaignPrompt(**request['params'])
        )
        return {
            "content": [
                {
                    "type": "text",
                    "text": f"Campaign plan created with {result.confidence*100}% confidence"
                },
                {
                    "type": "resource",
                    "resource": {
                        "uri": f"campaign://plans/{result.campaign_id}",
                        "mimeType": "application/json",
                        "text": result.json()
                    }
                }
            ]
        }
    except Exception as e:
        return {"error": str(e)}
```

---

### 3. Workflow Registry Service

**Purpose**: Manage catalog of available n8n workflow templates.

**Responsibilities**:
- Store workflow template definitions
- Provide search/discovery API
- Version management
- Workflow validation
- Usage analytics
- Rating/feedback system

**Technology Stack**: Node.js, PostgreSQL, Elasticsearch

**Database Schema**:

```sql
-- Workflow Templates Table
CREATE TABLE workflow_templates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_id VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    category VARCHAR(50),
    version VARCHAR(20),
    template_json JSONB NOT NULL,
    required_inputs JSONB,
    required_credentials JSONB,
    supported_platforms TEXT[],
    capabilities TEXT[],
    tags TEXT[],
    estimated_execution_time INTEGER, -- seconds
    success_rate DECIMAL(5,2),
    usage_count INTEGER DEFAULT 0,
    rating DECIMAL(3,2),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Workflow Categories Table
CREATE TABLE workflow_categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    icon VARCHAR(50),
    parent_category_id INTEGER REFERENCES workflow_categories(id)
);

-- Workflow Executions Table
CREATE TABLE workflow_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_template_id UUID REFERENCES workflow_templates(id),
    user_id UUID NOT NULL,
    campaign_id UUID,
    n8n_execution_id VARCHAR(100),
    status VARCHAR(20), -- pending, running, completed, failed
    input_data JSONB,
    output_data JSONB,
    error_message TEXT,
    execution_time INTEGER,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Workflow Dependencies Table
CREATE TABLE workflow_dependencies (
    id SERIAL PRIMARY KEY,
    workflow_id UUID REFERENCES workflow_templates(id),
    depends_on_workflow_id UUID REFERENCES workflow_templates(id),
    dependency_type VARCHAR(50) -- prerequisite, data_source, enhancement
);

-- Create indexes
CREATE INDEX idx_workflows_category ON workflow_templates(category);
CREATE INDEX idx_workflows_platforms ON workflow_templates USING GIN(supported_platforms);
CREATE INDEX idx_workflows_capabilities ON workflow_templates USING GIN(capabilities);
CREATE INDEX idx_workflows_tags ON workflow_templates USING GIN(tags);
CREATE INDEX idx_executions_user ON workflow_executions(user_id);
CREATE INDEX idx_executions_status ON workflow_executions(status);
```

**API Implementation**:

```typescript
import { FastifyInstance } from 'fastify';
import { Pool } from 'pg';

export class WorkflowRegistryService {
  private db: Pool;
  
  constructor(db: Pool) {
    this.db = db;
  }
  
  async searchWorkflows(criteria: {
    platforms?: string[];
    capabilities?: string[];
    category?: string;
    keywords?: string;
  }) {
    let query = `
      SELECT * FROM workflow_templates
      WHERE is_active = true
    `;
    
    const params: any[] = [];
    let paramIndex = 1;
    
    if (criteria.platforms?.length) {
      query += ` AND supported_platforms && $${paramIndex}`;
      params.push(criteria.platforms);
      paramIndex++;
    }
    
    if (criteria.capabilities?.length) {
      query += ` AND capabilities && $${paramIndex}`;
      params.push(criteria.capabilities);
      paramIndex++;
    }
    
    if (criteria.category) {
      query += ` AND category = $${paramIndex}`;
      params.push(criteria.category);
      paramIndex++;
    }
    
    if (criteria.keywords) {
      query += ` AND (
        name ILIKE $${paramIndex} OR 
        description ILIKE $${paramIndex} OR
        $${paramIndex} = ANY(tags)
      )`;
      params.push(`%${criteria.keywords}%`);
    }
    
    query += ` ORDER BY success_rate DESC, usage_count DESC`;
    
    const result = await this.db.query(query, params);
    return result.rows;
  }
  
  async selectOptimalWorkflows(requirements: any) {
    // 1. Search for matching workflows
    const candidates = await this.searchWorkflows({
      platforms: requirements.platforms,
      capabilities: requirements.required_workflows,
      category: requirements.campaign_type
    });
    
    // 2. Score each workflow
    const scored = candidates.map(workflow => ({
      ...workflow,
      score: this.calculateWorkflowScore(workflow, requirements)
    }));
    
    // 3. Sort by score
    scored.sort((a, b) => b.score - a.score);
    
    // 4. Select optimal combination (avoiding duplicates)
    const selected = this.selectOptimalCombination(scored, requirements);
    
    // 5. Check for dependencies
    const withDependencies = await this.resolveDependencies(selected);
    
    return withDependencies;
  }
  
  private calculateWorkflowScore(workflow: any, requirements: any): number {
    let score = 0;
    
    // Platform match (high weight)
    const platformMatch = workflow.supported_platforms.filter(
      p => requirements.platforms.includes(p)
    ).length / requirements.platforms.length;
    score += platformMatch * 40;
    
    // Capability match
    const capabilityMatch = workflow.capabilities.filter(
      c => requirements.required_workflows.includes(c)
    ).length / requirements.required_workflows.length;
    score += capabilityMatch * 30;
    
    // Success rate
    score += (workflow.success_rate / 100) * 20;
    
    // Usage popularity (normalized)
    score += Math.min(workflow.usage_count / 1000, 1) * 10;
    
    return score;
  }
  
  private async resolveDependencies(workflows: any[]) {
    const workflowIds = workflows.map(w => w.id);
    
    const dependencies = await this.db.query(`
      SELECT * FROM workflow_dependencies
      WHERE workflow_id = ANY($1)
    `, [workflowIds]);
    
    // Add prerequisite workflows
    for (const dep of dependencies.rows) {
      if (dep.dependency_type === 'prerequisite') {
        const prereq = await this.getWorkflowById(dep.depends_on_workflow_id);
        if (!workflowIds.includes(prereq.id)) {
          workflows.unshift(prereq); // Add at beginning
        }
      }
    }
    
    return workflows;
  }
}

// Fastify Routes
export async function registerRoutes(fastify: FastifyInstance) {
  const registry = new WorkflowRegistryService(fastify.pg);
  
  fastify.post('/workflows/search', async (request, reply) => {
    const results = await registry.searchWorkflows(request.body);
    return results;
  });
  
  fastify.post('/workflows/select', async (request, reply) => {
    const selected = await registry.selectOptimalWorkflows(request.body);
    return selected;
  });
  
  fastify.get('/workflows/:id', async (request, reply) => {
    const { id } = request.params;
    const workflow = await registry.getWorkflowById(id);
    return workflow;
  });
}
```

---

### 4. Provisioning Service

**Purpose**: Dynamically provision and manage n8n instances for users.

**Responsibilities**:
- Spin up new n8n instances (Docker/Kubernetes)
- Assign instances to users/teams
- Scale instances based on load
- Health monitoring
- Resource cleanup
- Cost optimization

**Technology Stack**: Go, Kubernetes Client, Docker SDK

**Core Implementation**:

```go
package provisioning

import (
    "context"
    "fmt"
    
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
)

type ProvisioningService struct {
    k8sClient *kubernetes.Clientset
    namespace string
}

type N8NInstanceConfig struct {
    UserID      string
    TierLevel   string // free, pro, enterprise
    Resources   ResourceConfig
    Persistence bool
}

type ResourceConfig struct {
    CPURequest    string
    CPULimit      string
    MemoryRequest string
    MemoryLimit   string
    StorageSize   string
}

func (s *ProvisioningService) ProvisionN8NInstance(
    ctx context.Context, 
    config N8NInstanceConfig,
) (*N8NInstance, error) {
    
    // 1. Determine resource allocation based on tier
    resources := s.getResourcesForTier(config.TierLevel)
    
    // 2. Create namespace if needed
    namespace := fmt.Sprintf("n8n-%s", config.UserID)
    if err := s.ensureNamespace(ctx, namespace); err != nil {
        return nil, err
    }
    
    // 3. Create ConfigMap for n8n configuration
    if err := s.createN8NConfigMap(ctx, namespace, config); err != nil {
        return nil, err
    }
    
    // 4. Create PersistentVolumeClaim if persistence enabled
    if config.Persistence {
        if err := s.createPVC(ctx, namespace, resources.StorageSize); err != nil {
            return nil, err
        }
    }
    
    // 5. Create Deployment
    deployment := s.buildN8NDeployment(namespace, config, resources)
    _, err := s.k8sClient.AppsV1().Deployments(namespace).Create(
        ctx, 
        deployment, 
        metav1.CreateOptions{},
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create deployment: %w", err)
    }
    
    // 6. Create Service
    service := s.buildN8NService(namespace, config)
    _, err = s.k8sClient.CoreV1().Services(namespace).Create(
        ctx,
        service,
        metav1.CreateOptions{},
    )
    if err != nil {
        return nil, fmt.Errorf("failed to create service: %w", err)
    }
    
    // 7. Create Ingress for external access
    ingress := s.buildN8NIngress(namespace, config)
    if err := s.createIngress(ctx, ingress); err != nil {
        return nil, err
    }
    
    // 8. Wait for pod to be ready
    if err := s.waitForPodReady(ctx, namespace, config.UserID); err != nil {
        return nil, err
    }
    
    // 9. Initialize n8n (create admin user, set up webhooks)
    instance := &N8NInstance{
        UserID:    config.UserID,
        Namespace: namespace,
        URL:       fmt.Sprintf("https://n8n-%s.spreaders.io", config.UserID),
        Status:    "ready",
    }
    
    if err := s.initializeN8N(ctx, instance); err != nil {
        return nil, err
    }
    
    // 10. Store instance metadata in database
    if err := s.saveInstanceMetadata(instance); err != nil {
        return nil, err
    }
    
    return instance, nil
}

func (s *ProvisioningService) buildN8NDeployment(
    namespace string,
    config N8NInstanceConfig,
    resources ResourceConfig,
) *appsv1.Deployment {
    
    replicas := int32(1)
    
    return &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      fmt.Sprintf("n8n-%s", config.UserID),
            Namespace: namespace,
            Labels: map[string]string{
                "app":     "n8n",
                "user":    config.UserID,
                "tier":    config.TierLevel,
                "managed": "spreaders",
            },
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: &replicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{
                    "app":  "n8n",
                    "user": config.UserID,
                },
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{
                        "app":  "n8n",
                        "user": config.UserID,
                    },
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  "n8n",
                            Image: "n8nio/n8n:latest",
                            Ports: []corev1.ContainerPort{
                                {
                                    ContainerPort: 5678,
                                    Name:          "http",
                                },
                            },
                            Env: []corev1.EnvVar{
                                {
                                    Name:  "N8N_BASIC_AUTH_ACTIVE",
                                    Value: "true",
                                },
                                {
                                    Name:  "N8N_HOST",
                                    Value: fmt.Sprintf("n8n-%s.spreaders.io", config.UserID),
                                },
                                {
                                    Name:  "WEBHOOK_URL",
                                    Value: fmt.Sprintf("https://n8n-%s.spreaders.io", config.UserID),
                                },
                                {
                                    Name:  "N8N_ENCRYPTION_KEY",
                                    ValueFrom: &corev1.EnvVarSource{
                                        SecretKeyRef: &corev1.SecretKeySelector{
                                            LocalObjectReference: corev1.LocalObjectReference{
                                                Name: "n8n-secrets",
                                            },
                                            Key: "encryption-key",
                                        },
                                    },
                                },
                            },
                            Resources: corev1.ResourceRequirements{
                                Requests: corev1.ResourceList{
                                    corev1.ResourceCPU:    resource.MustParse(resources.CPURequest),
                                    corev1.ResourceMemory: resource.MustParse(resources.MemoryRequest),
                                },
                                Limits: corev1.ResourceList{
                                    corev1.ResourceCPU:    resource.MustParse(resources.CPULimit),
                                    corev1.ResourceMemory: resource.MustParse(resources.MemoryLimit),
                                },
                            },
                            VolumeMounts: []corev1.VolumeMount{
                                {
                                    Name:      "n8n-data",
                                    MountPath: "/home/node/.n8n",
                                },
                            },
                        },
                    },
                    Volumes: []corev1.Volume{
                        {
                            Name: "n8n-data",
                            VolumeSource: corev1.VolumeSource{
                                PersistentVolumeClaim: &corev1.PersistentVolumeClaimVolumeSource{
                                    ClaimName: fmt.Sprintf("n8n-pvc-%s", config.UserID),
                                },
                            },
                        },
                    },
                },
            },
        },
    }
}

func (s *ProvisioningService) getResourcesForTier(tier string) ResourceConfig {
    switch tier {
    case "free":
        return ResourceConfig{
            CPURequest:    "100m",
            CPULimit:      "250m",
            MemoryRequest: "256Mi",
            MemoryLimit:   "512Mi",
            StorageSize:   "5Gi",
        }
    case "pro":
        return ResourceConfig{
            CPURequest:    "250m",
            CPULimit:      "1000m",
            MemoryRequest: "512Mi",
            MemoryLimit:   "2Gi",
            StorageSize:   "20Gi",
        }
    case "enterprise":
        return ResourceConfig{
            CPURequest:    "1000m",
            CPULimit:      "4000m",
            MemoryRequest: "2Gi",
            MemoryLimit:   "8Gi",
            StorageSize:   "100Gi",
        }
    default:
        return s.getResourcesForTier("free")
    }
}

func (s *ProvisioningService) ScaleN8NInstance(
    ctx context.Context,
    userID string,
    newTier string,
) error {
    // Update deployment resources
    namespace := fmt.Sprintf("n8n-%s", userID)
    deployment, err := s.k8sClient.AppsV1().Deployments(namespace).Get(
        ctx,
        fmt.Sprintf("n8n-%s", userID),
        metav1.GetOptions{},
    )
    if err != nil {
        return err
    }
    
    resources := s.getResourcesForTier(newTier)
    deployment.Spec.Template.Spec.Containers[0].Resources = corev1.ResourceRequirements{
        Requests: corev1.ResourceList{
            corev1.ResourceCPU:    resource.MustParse(resources.CPURequest),
            corev1.ResourceMemory: resource.MustParse(resources.MemoryRequest),
        },
        Limits: corev1.ResourceList{
            corev1.ResourceCPU:    resource.MustParse(resources.CPULimit),
            corev1.ResourceMemory: resource.MustParse(resources.MemoryLimit),
        },
    }
    
    _, err = s.k8sClient.AppsV1().Deployments(namespace).Update(
        ctx,
        deployment,
        metav1.UpdateOptions{},
    )
    
    return err
}

func (s *ProvisioningService) DeprovisionN8NInstance(
    ctx context.Context,
    userID string,
) error {
    namespace := fmt.Sprintf("n8n-%s", userID)
    
    // 1. Backup data before deletion
    if err := s.backupInstanceData(ctx, userID); err != nil {
        return err
    }
    
    // 2. Delete deployment
    deletePolicy := metav1.DeletePropagationForeground
    if err := s.k8sClient.AppsV1().Deployments(namespace).Delete(
        ctx,
        fmt.Sprintf("n8n-%s", userID),
        metav1.DeleteOptions{
            PropagationPolicy: &deletePolicy,
        },
    ); err != nil {
        return err
    }
    
    // 3. Delete service
    if err := s.k8sClient.CoreV1().Services(namespace).Delete(
        ctx,
        fmt.Sprintf("n8n-service-%s", userID),
        metav1.DeleteOptions{},
    ); err != nil {
        return err
    }
    
    // 4. Delete PVC (after grace period)
    // Note: Usually keep PVCs for a period in case of restore
    
    // 5. Update database
    if err := s.markInstanceDeprovisioned(userID); err != nil {
        return err
    }
    
    return nil
}
```

---

### 5. N8N Orchestrator Service

**Purpose**: Interface with n8n instances to deploy and execute workflows.

**Responsibilities**:
- Deploy workflow templates to n8n instances
- Configure workflow parameters
- Trigger workflow executions
- Monitor execution status
- Retrieve execution results
- Handle workflow errors

**Technology Stack**: Node.js, N8N API Client, NestJS

**Implementation**:

```typescript
import { Injectable } from '@nestjs/common';
import axios, { AxiosInstance } from 'axios';

@Injectable()
export class N8NOrchestratorService {
  private n8nClients: Map<string, AxiosInstance> = new Map();
  
  /**
   * Get or create N8N API client for user
   */
  private getN8NClient(userInstanceUrl: string, apiKey: string): AxiosInstance {
    if (!this.n8nClients.has(userInstanceUrl)) {
      const client = axios.create({
        baseURL: userInstanceUrl,
        headers: {
          'X-N8N-API-KEY': apiKey,
          'Content-Type': 'application/json',
        },
      });
      this.n8nClients.set(userInstanceUrl, client);
    }
    return this.n8nClients.get(userInstanceUrl)!;
  }
  
  /**
   * Deploy workflow template to user's N8N instance
   */
  async deployWorkflow(params: {
    userId: string;
    workflowTemplate: any;
    configuration: any;
  }) {
    // 1. Get user's N8N instance details
    const instance = await this.getInstance(params.userId);
    const client = this.getN8NClient(instance.url, instance.apiKey);
    
    // 2. Clone and configure workflow
    const workflow = this.configureWorkflow(
      params.workflowTemplate,
      params.configuration
    );
    
    // 3. Create workflow in N8N
    const response = await client.post('/workflows', {
      name: `${params.configuration.campaign_id}_${workflow.name}`,
      nodes: workflow.nodes,
      connections: workflow.connections,
      settings: workflow.settings,
      active: false, // Initially inactive
    });
    
    // 4. Store workflow metadata
    await this.saveWorkflowDeployment({
      userId: params.userId,
      campaignId: params.configuration.campaign_id,
      workflowTemplateId: params.workflowTemplate.id,
      n8nWorkflowId: response.data.id,
      configuration: params.configuration,
    });
    
    return {
      workflowId: response.data.id,
      status: 'deployed',
      url: `${instance.url}/workflow/${response.data.id}`,
    };
  }
  
  /**
   * Configure workflow with user-specific parameters
   */
  private configureWorkflow(template: any, config: any): any {
    const workflow = JSON.parse(JSON.stringify(template.template_json));
    
    // Replace placeholders in nodes
    workflow.nodes = workflow.nodes.map(node => {
      // Inject credentials
      if (node.credentials) {
        Object.keys(node.credentials).forEach(credType => {
          const credId = config.credentials[credType];
          if (credId) {
            node.credentials[credType] = { id: credId };
          }
        });
      }
      
      // Inject parameters
      if (node.parameters) {
        node.parameters = this.injectParameters(
          node.parameters,
          config.parameters
        );
      }
      
      return node;
    });
    
    return workflow;
  }
  
  /**
   * Execute workflow
   */
  async executeWorkflow(params: {
    userId: string;
    workflowId: string;
    inputData?: any;
  }) {
    const instance = await this.getInstance(params.userId);
    const client = this.getN8NClient(instance.url, instance.apiKey);
    
    // 1. Activate workflow if not active
    await client.patch(`/workflows/${params.workflowId}`, {
      active: true,
    });
    
    // 2. Execute workflow
    const response = await client.post(
      `/workflows/${params.workflowId}/execute`,
      {
        data: params.inputData || {},
      }
    );
    
    // 3. Store execution metadata
    await this.saveExecution({
      userId: params.userId,
      workflowId: params.workflowId,
      n8nExecutionId: response.data.executionId,
      inputData: params.inputData,
      status: 'running',
    });
    
    return {
      executionId: response.data.executionId,
      status: 'running',
    };
  }
  
  /**
   * Get execution status
   */
  async getExecutionStatus(params: {
    userId: string;
    executionId: string;
  }) {
    const instance = await this.getInstance(params.userId);
    const client = this.getN8NClient(instance.url, instance.apiKey);
    
    const response = await client.get(`/executions/${params.executionId}`);
    
    // Update database
    await this.updateExecutionStatus({
      executionId: params.executionId,
      status: response.data.finished ? 'completed' : 'running',
      output: response.data.data,
      error: response.data.error,
    });
    
    return {
      executionId: params.executionId,
      status: response.data.finished ? 'completed' : 'running',
      success: !response.data.error,
      output: response.data.data,
      error: response.data.error,
      executionTime: response.data.executionTime,
    };
  }
  
  /**
   * Execute workflow chain (multiple workflows in sequence)
   */
  async executeWorkflowChain(params: {
    userId: string;
    workflows: Array<{
      workflowId: string;
      inputData?: any;
      waitForCompletion?: boolean;
    }>;
  }) {
    const results = [];
    
    for (const workflow of params.workflows) {
      // Prepare input (may include output from previous workflow)
      const inputData = workflow.inputData || {};
      if (results.length > 0) {
        inputData._previousOutput = results[results.length - 1].output;
      }
      
      // Execute workflow
      const execution = await this.executeWorkflow({
        userId: params.userId,
        workflowId: workflow.workflowId,
        inputData,
      });
      
      // Wait for completion if required
      if (workflow.waitForCompletion !== false) {
        const result = await this.waitForCompletion(
          params.userId,
          execution.executionId
        );
        results.push(result);
        
        // Stop chain if workflow failed
        if (!result.success) {
          return {
            status: 'failed',
            completedWorkflows: results.length,
            totalWorkflows: params.workflows.length,
            results,
          };
        }
      } else {
        results.push(execution);
      }
    }
    
    return {
      status: 'completed',
      completedWorkflows: results.length,
      totalWorkflows: params.workflows.length,
      results,
    };
  }
  
  /**
   * Wait for workflow execution to complete
   */
  private async waitForCompletion(
    userId: string,
    executionId: string,
    timeoutMs: number = 300000 // 5 minutes default
  ) {
    const startTime = Date.now();
    
    while (Date.now() - startTime < timeoutMs) {
      const status = await this.getExecutionStatus({ userId, executionId });
      
      if (status.status === 'completed') {
        return status;
      }
      
      // Wait 2 seconds before checking again
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
    
    throw new Error(`Execution ${executionId} timed out after ${timeoutMs}ms`);
  }
}
```

---

### 6. Credential Vault Service

**Purpose**: Securely store and manage user credentials for third-party platforms.

**Responsibilities**:
- Store OAuth tokens, API keys
- Encryption at rest
- Credential refresh (OAuth)
- Audit logging
- Access control
- Integration with n8n credential system

**Technology Stack**: Go, HashiCorp Vault, PostgreSQL

**Implementation**:

```go
package credentials

import (
    "context"
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "encoding/base64"
    "fmt"
    
    vault "github.com/hashicorp/vault/api"
)

type CredentialVaultService struct {
    vaultClient *vault.Client
    db          *Database
    cipher      cipher.AEAD
}

type Credential struct {
    ID           string
    UserID       string
    Platform     string
    CredType     string // oauth, api_key, basic_auth
    EncryptedData string
    ExpiresAt    *time.Time
    CreatedAt    time.Time
    UpdatedAt    time.Time
}

func NewCredentialVaultService(vaultAddr, vaultToken string) (*CredentialVaultService, error) {
    config := vault.DefaultConfig()
    config.Address = vaultAddr
    
    client, err := vault.NewClient(config)
    if err != nil {
        return nil, err
    }
    
    client.SetToken(vaultToken)
    
    return &CredentialVaultService{
        vaultClient: client,
    }, nil
}

func (s *CredentialVaultService) StoreCredential(
    ctx context.Context,
    credential Credential,
) error {
    // 1. Encrypt credential data
    encrypted, err := s.encrypt(credential.EncryptedData)
    if err != nil {
        return fmt.Errorf("encryption failed: %w", err)
    }
    
    // 2. Store in Vault
    path := fmt.Sprintf("secret/data/spreaders/users/%s/credentials/%s",
        credential.UserID,
        credential.Platform,
    )
    
    data := map[string]interface{}{
        "data": map[string]interface{}{
            "credential_type": credential.CredType,
            "encrypted_data":  encrypted,
            "expires_at":      credential.ExpiresAt,
        },
    }
    
    _, err = s.vaultClient.Logical().Write(path, data)
    if err != nil {
        return fmt.Errorf("vault write failed: %w", err)
    }
    
    // 3. Store metadata in PostgreSQL
    err = s.db.StoreCredentialMetadata(ctx, credential)
    if err != nil {
        return fmt.Errorf("db write failed: %w", err)
    }
    
    // 4. Audit log
    s.auditLog(ctx, "credential_stored", credential.UserID, credential.Platform)
    
    return nil
}

func (s *CredentialVaultService) GetCredential(
    ctx context.Context,
    userID, platform string,
) (*Credential, error) {
    // 1. Get from Vault
    path := fmt.Sprintf("secret/data/spreaders/users/%s/credentials/%s",
        userID,
        platform,
    )
    
    secret, err := s.vaultClient.Logical().Read(path)
    if err != nil {
        return nil, fmt.Errorf("vault read failed: %w", err)
    }
    
    if secret == nil {
        return nil, fmt.Errorf("credential not found")
    }
    
    data := secret.Data["data"].(map[string]interface{})
    
    // 2. Decrypt
    decrypted, err := s.decrypt(data["encrypted_data"].(string))
    if err != nil {
        return nil, fmt.Errorf("decryption failed: %w", err)
    }
    
    // 3. Audit log
    s.auditLog(ctx, "credential_accessed", userID, platform)
    
    return &Credential{
        UserID:       userID,
        Platform:     platform,
        CredType:     data["credential_type"].(string),
        EncryptedData: decrypted,
    }, nil
}

func (s *CredentialVaultService) RefreshOAuthToken(
    ctx context.Context,
    userID, platform string,
) error {
    // 1. Get existing credential
    cred, err := s.GetCredential(ctx, userID, platform)
    if err != nil {
        return err
    }
    
    // 2. Parse OAuth data
    var oauthData OAuthCredential
    if err := json.Unmarshal([]byte(cred.EncryptedData), &oauthData); err != nil {
        return err
    }
    
    // 3. Refresh token
    newToken, err := s.refreshToken(platform, oauthData.RefreshToken)
    if err != nil {
        return fmt.Errorf("token refresh failed: %w", err)
    }
    
    // 4. Update stored credential
    oauthData.AccessToken = newToken.AccessToken
    oauthData.RefreshToken = newToken.RefreshToken
    oauthData.ExpiresAt = time.Now().Add(time.Duration(newToken.ExpiresIn) * time.Second)
    
    updatedData, _ := json.Marshal(oauthData)
    cred.EncryptedData = string(updatedData)
    
    err = s.StoreCredential(ctx, *cred)
    if err != nil {
        return err
    }
    
    return nil
}

func (s *CredentialVaultService) DeleteCredential(
    ctx context.Context,
    userID, platform string,
) error {
    // 1. Delete from Vault
    path := fmt.Sprintf("secret/data/spreaders/users/%s/credentials/%s",
        userID,
        platform,
    )
    
    _, err := s.vaultClient.Logical().Delete(path)
    if err != nil {
        return err
    }
    
    // 2. Delete metadata from DB
    err = s.db.DeleteCredentialMetadata(ctx, userID, platform)
    if err != nil {
        return err
    }
    
    // 3. Audit log
    s.auditLog(ctx, "credential_deleted", userID, platform)
    
    return nil
}

func (s *CredentialVaultService) MapToN8NCredential(
    ctx context.Context,
    userID, platform string,
) (string, error) {
    // 1. Get credential from vault
    cred, err := s.GetCredential(ctx, userID, platform)
    if err != nil {
        return "", err
    }
    
    // 2. Create credential in user's N8N instance
    n8nCred := s.convertToN8NFormat(platform, cred)
    
    // 3. Call N8N API to create credential
    credentialID, err := s.createN8NCredential(userID, n8nCred)
    if err != nil {
        return "", err
    }
    
    return credentialID, nil
}

func (s *CredentialVaultService) encrypt(plaintext string) (string, error) {
    nonce := make([]byte, s.cipher.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return "", err
    }
    
    ciphertext := s.cipher.Seal(nonce, nonce, []byte(plaintext), nil)
    return base64.StdEncoding.EncodeToString(ciphertext), nil
}

func (s *CredentialVaultService) decrypt(ciphertext string) (string, error) {
    data, err := base64.StdEncoding.DecodeString(ciphertext)
    if err != nil {
        return "", err
    }
    
    nonceSize := s.cipher.NonceSize()
    nonce, ciphertext := data[:nonceSize], data[nonceSize:]
    
    plaintext, err := s.cipher.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return "", err
    }
    
    return string(plaintext), nil
}
```

---

## (Continuing in next file due to length limits...)

The infrastructure requires **13 core microservices** total, with Kubernetes orchestration, comprehensive monitoring, and MCP integration throughout. Would you like me to continue with the remaining services (Execution Monitor, User Management, Campaign Management) and deployment configurations?
