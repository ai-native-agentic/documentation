# UC5: Enterprise AI Ops Platform
## Implementation-Ready Specification

**Version:** 1.0  
**Date:** 2026-03-10  
**Status:** MVP Ready  
**Target Audience:** Engineering Leadership, Platform Architects, Product Management

---

## 1. Executive Summary

### The Problem
Enterprises struggle to deploy AI agents at scale across their organization. Challenges include:
- Fragmented communication channels (Slack, Teams, email, SMS)
- No isolation between teams or departments
- Zero visibility into AI costs vs business value
- Manual agent deployment and maintenance
- Security concerns with shared agent memory
- No ROI measurement framework

### The Solution
Enterprise AI Ops Platform is a multi-tenant SaaS that unifies AI agent deployment, execution, and economic tracking across enterprise communication channels. Built on proven open-source components with $19K/8hr economic validation.

### Core Value Propositions
1. **Unified Gateway**: Deploy agents to 22+ channels (Slack, Teams, email) from single control plane
2. **Secure Isolation**: Container-based per-tenant agent execution with OS-level security
3. **Computer Use Automation**: Agents automate desktop/browser tasks via openmanus
4. **Self-Learning**: Continuous improvement through DSPy optimization and PEFT fine-tuning
5. **ROI Transparency**: Real-time cost tracking with GDPVal benchmark comparison
6. **Enterprise Auth**: SSO, RBAC, audit logs, SOC2-ready architecture

### Business Model
- **Starter**: $500/mo (5 agents, 10K requests/day)
- **Professional**: $2,500/mo (25 agents, 100K requests/day, SSO)
- **Enterprise**: $10K+/mo (unlimited, 99.9% SLA, custom integrations)
- **Target ACV**: $50K-$500K per customer
- **NDR Target**: 130%+ (expansion through agent proliferation)

### MVP Timeline
**2 weeks** to production-ready MVP with core features deployed.

---

## 2. Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        ENTERPRISE USERS                              │
│  Slack │ Teams │ Email │ SMS │ Discord │ Telegram │ Custom Channels │
└────────┬────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    OPENCLAW GATEWAY (TypeScript)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ChannelPlugin │  │ChannelPlugin │  │ChannelPlugin │  (22+ types) │
│  │   (Slack)    │  │   (Teams)    │  │   (Email)    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         └──────────────────┴──────────────────┘                      │
│                            │                                         │
│                    ┌───────▼────────┐                                │
│                    │ Message Router │                                │
│                    └───────┬────────┘                                │
└────────────────────────────┼─────────────────────────────────────────┘
                             │ HTTP/gRPC
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              ENTERPRISE GATEWAY (Python FastAPI)                     │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Request Handler                                              │   │
│  │  • Tenant identification (JWT/API key)                       │   │
│  │  • RBAC permission check                                     │   │
│  │  • Quota enforcement (requests/day, tokens/day)              │   │
│  │  • Request routing                                           │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                             │                                        │
│         ┌───────────────────┼───────────────────┐                   │
│         ▼                   ▼                   ▼                    │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐             │
│  │   Tenant    │   │    Agent     │   │   Economic   │             │
│  │  Manager    │   │ Orchestrator │   │   Tracker    │             │
│  └─────────────┘   └──────┬───────┘   └──────────────┘             │
└────────────────────────────┼─────────────────────────────────────────┘
                             │ Container Spawn
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              NANOCLAW ORCHESTRATOR (TypeScript)                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Container Pool Manager                                       │   │
│  │  • Per-tenant container isolation                            │   │
│  │  • CLAUDE.md memory isolation                                │   │
│  │  • Resource limits (CPU, memory, disk)                       │   │
│  │  • Mount security (whitelist only)                           │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                             │                                        │
│         ┌───────────────────┼───────────────────┐                   │
│         ▼                   ▼                   ▼                    │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐               │
│  │ Container A │   │ Container B │   │ Container C │  (isolated)   │
│  │ Tenant: 001 │   │ Tenant: 002 │   │ Tenant: 003 │               │
│  └─────┬───────┘   └─────┬───────┘   └─────┬───────┘               │
└────────┼─────────────────┼─────────────────┼─────────────────────────┘
         │                 │                 │
         └─────────────────┴─────────────────┘
                           │ A2A Protocol
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 OPENMANUS EXECUTOR (Python)                          │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Agent Execution Engine                                       │   │
│  │  • Computer use (keyboard, mouse, screenshot)                │   │
│  │  • Browser automation (browser_use_tool)                     │   │
│  │  • Code execution (bash, file ops)                           │   │
│  │  • Web search, MCP tools                                     │   │
│  │  • Planning & ask_human                                      │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                             │                                        │
│         ┌───────────────────┼───────────────────┐                   │
│         ▼                   ▼                   ▼                    │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐               │
│  │ Computer    │   │  Browser    │   │    Code     │               │
│  │    Use      │   │    Use      │   │  Execution  │               │
│  └─────────────┘   └─────────────┘   └─────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│          SELF-LEARNING PIPELINE (Python)                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Continuous Improvement                                       │   │
│  │  • DSPy prompt optimization                                  │   │
│  │  • PEFT fine-tuning (LoRA)                                   │   │
│  │  • Quality evaluation (LLM-as-judge)                         │   │
│  │  • Feedback loop (economic signals)                          │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│          ECONOMIC TRACKING (Python)                                  │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ ROI Dashboard                                                │   │
│  │  • Cost tracking (LLM API calls)                             │   │
│  │  • Income tracking (task value from GDPVal)                  │   │
│  │  • Quality scoring (0-100%, threshold 0.6)                   │   │
│  │  • ROI metrics (compute_roi, compute_pay_rate)               │   │
│  │  • Sustainability score                                      │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Layer | Technology | Project Source |
|-------|-----------|----------------|
| Gateway | TypeScript, pnpm, Express | openclaw |
| Orchestration | Python 3.10+, FastAPI, asyncio | NEW (enterprise-gateway) |
| Container Isolation | TypeScript, Docker/Apple Container | nanoclaw |
| Agent Execution | Python, MetaGPT, A2A protocol | openmanus |
| Self-Learning | Python, DSPy, PEFT, PyTorch | ai-native-self-learning-agents |
| Economic Tracking | Python, JSONL ledger | ai-native-agentic-economic-sdk |
| Benchmarking | Python, GDPVal (220 tasks) | ClawWork |
| Auth & RBAC | PostgreSQL, JWT, Stripe | ai-native-self-learning-agents |

---

## 3. Component Responsibilities

### 3.1 OpenClaw Gateway (TypeScript)

**Location**: `/home/lunark/projects/ai-native-agentic-org/openclaw/`

**Responsibilities**:
- Multi-channel message ingestion (Slack, Teams, email, SMS, Discord, Telegram, etc.)
- Channel-specific protocol handling (OAuth, webhooks, polling)
- Message normalization to common format
- Response delivery to appropriate channel
- Enterprise features: SSO, custom domains, audit logs

**Key Interfaces**:
```typescript
// packages/core/src/channels/base.ts
export interface ChannelPlugin {
  id: string;
  name: string;
  initialize(config: ChannelConfig): Promise<void>;
  sendMessage(channelId: string, message: Message): Promise<void>;
  onMessage(handler: MessageHandler): void;
}

// Enterprise extension
export interface EnterpriseChannelPlugin extends ChannelPlugin {
  validateSSO(token: string): Promise<SSOUser>;
  auditLog(event: AuditEvent): Promise<void>;
}
```

**Integration Point**:
- Forwards normalized messages to Enterprise Gateway via HTTP/gRPC
- Receives responses and delivers to original channel

### 3.2 Enterprise Gateway (Python FastAPI) - NEW

**Location**: `/home/lunark/projects/ai-native-agentic-org/enterprise-gateway/` (to be created)

**Responsibilities**:
- Tenant identification and authentication (JWT, API keys)
- RBAC permission checks (VIEWER/OPERATOR/ADMIN)
- Quota enforcement (requests/day, tokens/day, max_agents)
- Agent orchestration (spawn containers, execute tasks)
- Economic tracking integration
- Response aggregation and delivery

**Key Interfaces**:
```python
# enterprise_gateway/main.py
from fastapi import FastAPI, Depends, HTTPException
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from typing import Optional
import asyncio

app = FastAPI(title="Enterprise AI Ops Gateway")
security = HTTPBearer()

class EnterpriseRequest(BaseModel):
    channel_id: str
    tenant_id: str
    user_id: str
    message: str
    context: Optional[dict] = None

class EnterpriseResponse(BaseModel):
    request_id: str
    response: str
    cost_usd: float
    income_usd: float
    roi: float
    execution_time_ms: int

@app.post("/v1/execute", response_model=EnterpriseResponse)
async def execute_agent_task(
    request: EnterpriseRequest,
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    # Authenticate and authorize
    tenant = await authenticate_tenant(credentials.credentials)
    await check_permission(tenant.id, request.user_id, Permission.EXECUTE_AGENT)
    
    # Enforce quotas
    await enforce_quotas(tenant)
    
    # Orchestrate execution
    gateway = EnterpriseGateway(tenant)
    response = await gateway.handle_request(request)
    
    return response
```

**Core Logic**:
```python
# enterprise_gateway/orchestrator.py
from dataclasses import dataclass
from typing import Optional
import httpx
import asyncio

@dataclass
class TaskResult:
    response: str
    tokens_used: int
    execution_time_ms: int
    quality_score: float
    cost_usd: float

class EnterpriseGateway:
    def __init__(self, tenant: Tenant):
        self.tenant = tenant
        self.nanoclaw_client = NanoclawClient()
        self.openmanus_client = OpenManusA2AClient()
        self.economic_tracker = EconomicTracker(tenant.id)
        
    async def handle_request(self, request: EnterpriseRequest) -> EnterpriseResponse:
        start_time = asyncio.get_event_loop().time()
        
        # 1. Spawn isolated container for this tenant
        container = await self.nanoclaw_client.spawn_container(
            tenant_id=self.tenant.id,
            agent_id=request.context.get("agent_id", "default"),
            memory_limit_mb=self.tenant.quotas.max_storage_mb
        )
        
        try:
            # 2. Execute task in isolated container via A2A protocol
            task_result = await self.openmanus_client.invoke_agent(
                container_id=container.id,
                query=request.message,
                session_id=request.context.get("session_id")
            )
            
            # 3. Track economics
            self.economic_tracker.track_llm_call(
                tokens_used=task_result.tokens_used,
                model_name=self.tenant.config.get("model", "claude-sonnet-4"),
                cost_per_token=0.000003  # $3/MTok
            )
            
            # 4. Evaluate quality and add income if threshold met
            if task_result.quality_score >= 0.6:
                task_value = await self._estimate_task_value(request.message)
                self.economic_tracker.add_work_income(
                    task_value=task_value,
                    evaluation_score=task_result.quality_score
                )
            
            # 5. Compute ROI
            metrics = self.economic_tracker.compute_roi()
            
            execution_time_ms = int((asyncio.get_event_loop().time() - start_time) * 1000)
            
            return EnterpriseResponse(
                request_id=str(uuid.uuid4()),
                response=task_result.response,
                cost_usd=task_result.cost_usd,
                income_usd=task_value if task_result.quality_score >= 0.6 else 0.0,
                roi=metrics["roi"],
                execution_time_ms=execution_time_ms
            )
            
        finally:
            # 6. Cleanup container
            await self.nanoclaw_client.destroy_container(container.id)
    
    async def _estimate_task_value(self, message: str) -> float:
        # Use ClawWork GDPVal benchmark to estimate task value
        # Map message to profession/task category, return BLS wage equivalent
        # Simplified: $50/hr average knowledge worker rate
        estimated_time_hours = 0.25  # 15 minutes average
        return 50.0 * estimated_time_hours
```

### 3.3 Nanoclaw Orchestrator (TypeScript) - EXTENDED

**Location**: `/home/lunark/projects/ai-native-agentic-org/nanoclaw/`

**New Module**: `src/enterprise/orchestrator.ts`

**Responsibilities**:
- Container lifecycle management (spawn, execute, destroy)
- Per-tenant memory isolation (separate CLAUDE.md)
- Resource limits enforcement (CPU, memory, disk)
- Mount security (whitelist only, no home dir)
- Container pool optimization (warm containers)

**Implementation**:
```typescript
// src/enterprise/orchestrator.ts
import { spawn } from 'child_process';
import { promises as fs } from 'fs';
import path from 'path';
import crypto from 'crypto';

export interface ContainerConfig {
  tenantId: string;
  agentId: string;
  memoryLimitMb: number;
  cpuLimit?: number;
  allowedMounts?: string[];
}

export interface Container {
  id: string;
  tenantId: string;
  agentId: string;
  status: 'starting' | 'running' | 'stopped';
  createdAt: Date;
}

export class NanoclawOrchestrator {
  private containers: Map<string, Container> = new Map();
  private baseDir = '/home/lunark/projects/ai-native-agentic-org/nanoclaw';
  
  async spawnContainer(config: ContainerConfig): Promise<Container> {
    const containerId = crypto.randomUUID();
    const containerDir = path.join(this.baseDir, 'containers', containerId);
    
    // Create isolated directory structure
    await fs.mkdir(containerDir, { recursive: true });
    await fs.mkdir(path.join(containerDir, 'memory'), { recursive: true });
    await fs.mkdir(path.join(containerDir, 'workspace'), { recursive: true });
    
    // Create tenant-specific CLAUDE.md
    const claudeMemory = `# Agent Memory for Tenant ${config.tenantId}
Agent ID: ${config.agentId}
Created: ${new Date().toISOString()}

## Context
This is an isolated memory space for tenant ${config.tenantId}.
No cross-tenant data should be accessible.

## Capabilities
- Computer use (keyboard, mouse, screenshot)
- Browser automation
- Code execution (sandboxed)
- File operations (within workspace only)

## Restrictions
- No access to other tenant data
- No home directory mounting
- Resource limits: ${config.memoryLimitMb}MB memory
`;
    
    await fs.writeFile(
      path.join(containerDir, 'memory', 'CLAUDE.md'),
      claudeMemory
    );
    
    // Prepare Docker/Apple Container command
    const containerCmd = this.buildContainerCommand(containerId, config);
    
    // Spawn container
    const container: Container = {
      id: containerId,
      tenantId: config.tenantId,
      agentId: config.agentId,
      status: 'starting',
      createdAt: new Date()
    };
    
    this.containers.set(containerId, container);
    
    // Start container process
    await this.startContainerProcess(containerCmd);
    
    container.status = 'running';
    
    return container;
  }
  
  private buildContainerCommand(containerId: string, config: ContainerConfig): string {
    const containerDir = path.join(this.baseDir, 'containers', containerId);
    
    // Use Docker for Linux, Apple Container for macOS
    if (process.platform === 'darwin') {
      // Apple Container (macOS)
      return `sandbox-exec -f ${this.baseDir}/sandbox.sb \\
        -D CONTAINER_ID=${containerId} \\
        -D MEMORY_LIMIT=${config.memoryLimitMb} \\
        -D WORKSPACE=${containerDir}/workspace \\
        -D MEMORY=${containerDir}/memory \\
        /usr/bin/python3 ${this.baseDir}/../openmanus/src/openmanus/main.py`;
    } else {
      // Docker (Linux)
      return `docker run -d \\
        --name nanoclaw-${containerId} \\
        --memory ${config.memoryLimitMb}m \\
        --cpus ${config.cpuLimit || 1} \\
        --network none \\
        -v ${containerDir}/workspace:/workspace:rw \\
        -v ${containerDir}/memory:/memory:ro \\
        -e TENANT_ID=${config.tenantId} \\
        -e AGENT_ID=${config.agentId} \\
        openmanus:latest`;
    }
  }
  
  private async startContainerProcess(cmd: string): Promise<void> {
    return new Promise((resolve, reject) => {
      const proc = spawn('sh', ['-c', cmd]);
      
      proc.on('error', reject);
      
      // Wait for container to be ready (simplified)
      setTimeout(() => resolve(), 2000);
    });
  }
  
  async executeInContainer(containerId: string, task: any): Promise<any> {
    const container = this.containers.get(containerId);
    if (!container || container.status !== 'running') {
      throw new Error(`Container ${containerId} not running`);
    }
    
    // Send task to container via IPC (HTTP API to openmanus)
    const response = await fetch(`http://localhost:${this.getContainerPort(containerId)}/execute`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(task)
    });
    
    return response.json();
  }
  
  async destroyContainer(containerId: string): Promise<void> {
    const container = this.containers.get(containerId);
    if (!container) return;
    
    // Stop container
    if (process.platform === 'darwin') {
      // Kill process group
      spawn('pkill', ['-f', `CONTAINER_ID=${containerId}`]);
    } else {
      // Stop Docker container
      spawn('docker', ['stop', `nanoclaw-${containerId}`]);
      spawn('docker', ['rm', `nanoclaw-${containerId}`]);
    }
    
    // Cleanup filesystem
    const containerDir = path.join(this.baseDir, 'containers', containerId);
    await fs.rm(containerDir, { recursive: true, force: true });
    
    this.containers.delete(containerId);
  }
  
  private getContainerPort(containerId: string): number {
    // Simple port mapping: hash container ID to port range 10000-20000
    const hash = crypto.createHash('md5').update(containerId).digest('hex');
    return 10000 + (parseInt(hash.substring(0, 4), 16) % 10000);
  }
}

// HTTP API for Enterprise Gateway to call
import express from 'express';

const app = express();
app.use(express.json());

const orchestrator = new NanoclawOrchestrator();

app.post('/containers/spawn', async (req, res) => {
  try {
    const container = await orchestrator.spawnContainer(req.body);
    res.json(container);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.post('/containers/:id/execute', async (req, res) => {
  try {
    const result = await orchestrator.executeInContainer(req.params.id, req.body);
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.delete('/containers/:id', async (req, res) => {
  try {
    await orchestrator.destroyContainer(req.params.id);
    res.json({ success: true });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(8080, () => {
  console.log('Nanoclaw Orchestrator API listening on port 8080');
});
```

### 3.4 OpenManus A2A Client (Python) - NEW

**Location**: `/home/lunark/projects/ai-native-agentic-org/openmanus/src/openmanus/enterprise/`

**New Module**: `a2a_client.py`

**Responsibilities**:
- A2A protocol client for invoking openmanus agents
- AgentCard registration for enterprise skills
- Task execution with computer use, browser automation, code execution
- Result aggregation and quality scoring

**Implementation**:
```python
# openmanus/src/openmanus/enterprise/a2a_client.py
from typing import Optional, Dict, Any
import httpx
import asyncio
from dataclasses import dataclass
from enum import Enum

class ToolType(Enum):
    COMPUTER_USE = "computer_use"
    BROWSER_USE = "browser_use"
    CODE_EXECUTION = "bash"
    FILE_OPS = "file_operators"
    WEB_SEARCH = "web_search"
    PLANNING = "planning"
    ASK_HUMAN = "ask_human"

@dataclass
class AgentSkill:
    name: str
    description: str
    tools: list[ToolType]
    parameters: Dict[str, Any]

@dataclass
class AgentResponse:
    response: str
    tokens_used: int
    execution_time_ms: int
    quality_score: float
    cost_usd: float
    tools_used: list[str]

class OpenManusA2AClient:
    def __init__(self, base_url: str = "http://localhost:8000"):
        self.base_url = base_url
        self.client = httpx.AsyncClient(timeout=300.0)  # 5 min timeout
        
    async def register_enterprise_skills(self, skills: list[AgentSkill]) -> None:
        """Register enterprise-specific agent skills via A2A protocol"""
        agent_card = {
            "name": "Enterprise AI Ops Agent",
            "version": "1.0.0",
            "capabilities": [
                {
                    "name": skill.name,
                    "description": skill.description,
                    "tools": [tool.value for tool in skill.tools],
                    "parameters": skill.parameters
                }
                for skill in skills
            ]
        }
        
        response = await self.client.post(
            f"{self.base_url}/a2a/register",
            json=agent_card
        )
        response.raise_for_status()
        
    async def invoke_agent(
        self,
        container_id: str,
        query: str,
        session_id: Optional[str] = None,
        tools: Optional[list[ToolType]] = None
    ) -> AgentResponse:
        """Invoke agent in isolated container via A2A protocol"""
        
        # Default to all tools if not specified
        if tools is None:
            tools = [
                ToolType.COMPUTER_USE,
                ToolType.BROWSER_USE,
                ToolType.CODE_EXECUTION,
                ToolType.FILE_OPS,
                ToolType.WEB_SEARCH,
                ToolType.PLANNING
            ]
        
        request_payload = {
            "query": query,
            "session_id": session_id or f"session-{container_id}",
            "tools": [tool.value for tool in tools],
            "container_id": container_id,
            "max_iterations": 10,
            "enable_computer_use": ToolType.COMPUTER_USE in tools,
            "enable_browser_use": ToolType.BROWSER_USE in tools
        }
        
        start_time = asyncio.get_event_loop().time()
        
        # Execute via openmanus ManusExecutor
        response = await self.client.post(
            f"{self.base_url}/execute",
            json=request_payload
        )
        response.raise_for_status()
        
        result = response.json()
        execution_time_ms = int((asyncio.get_event_loop().time() - start_time) * 1000)
        
        # Extract metrics
        tokens_used = result.get("tokens_used", 0)
        tools_used = result.get("tools_used", [])
        
        # Compute quality score (LLM-as-judge)
        quality_score = await self._evaluate_quality(query, result["response"])
        
        # Compute cost (simplified: $3/MTok for Claude Sonnet 4)
        cost_usd = (tokens_used / 1_000_000) * 3.0
        
        return AgentResponse(
            response=result["response"],
            tokens_used=tokens_used,
            execution_time_ms=execution_time_ms,
            quality_score=quality_score,
            cost_usd=cost_usd,
            tools_used=tools_used
        )
    
    async def _evaluate_quality(self, query: str, response: str) -> float:
        """Evaluate response quality using LLM-as-judge (0-100 scale)"""
        
        evaluation_prompt = f"""Evaluate the quality of this AI agent response on a scale of 0-100.

Query: {query}

Response: {response}

Criteria:
- Accuracy: Does it correctly address the query?
- Completeness: Does it cover all aspects?
- Clarity: Is it well-structured and understandable?
- Actionability: Can the user act on this response?

Return ONLY a number between 0 and 100."""

        eval_response = await self.client.post(
            f"{self.base_url}/evaluate",
            json={"prompt": evaluation_prompt}
        )
        eval_response.raise_for_status()
        
        try:
            score = float(eval_response.json()["score"])
            return min(max(score / 100.0, 0.0), 1.0)  # Normalize to 0-1
        except (ValueError, KeyError):
            return 0.5  # Default to 50% if evaluation fails
    
    async def close(self):
        await self.client.aclose()
```

### 3.5 Self-Learning Pipeline Integration

**Location**: `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/`

**Integration Point**: `src/agents/economic_feedback_agent.py`

**Responsibilities**:
- Collect execution traces from enterprise gateway
- DSPy prompt optimization based on quality scores
- PEFT fine-tuning for tenant-specific patterns
- Continuous improvement loop

**Extension**:
```python
# ai-native-self-learning-agents/src/agents/enterprise_learning_agent.py
from typing import List, Dict, Any
import dspy
from peft import LoraConfig, get_peft_model
import torch
from .economic_feedback_agent import EconomicFeedbackAgent

class EnterpriseLearningAgent(EconomicFeedbackAgent):
    """Extends EconomicFeedbackAgent with enterprise-specific learning"""
    
    def __init__(self, tenant_id: str):
        super().__init__()
        self.tenant_id = tenant_id
        self.execution_traces: List[Dict[str, Any]] = []
        
    async def collect_trace(self, request: str, response: str, quality_score: float, roi: float):
        """Collect execution trace for learning"""
        self.execution_traces.append({
            "request": request,
            "response": response,
            "quality_score": quality_score,
            "roi": roi,
            "timestamp": datetime.now().isoformat()
        })
        
        # Trigger optimization if we have enough data
        if len(self.execution_traces) >= 100:
            await self.optimize_prompts()
            
    async def optimize_prompts(self):
        """Use DSPy to optimize prompts based on execution traces"""
        
        # Filter high-quality traces (score >= 0.8)
        high_quality_traces = [
            t for t in self.execution_traces
            if t["quality_score"] >= 0.8
        ]
        
        if len(high_quality_traces) < 20:
            return  # Not enough data
        
        # Prepare DSPy training data
        training_data = [
            dspy.Example(
                question=t["request"],
                answer=t["response"]
            ).with_inputs("question")
            for t in high_quality_traces
        ]
        
        # Optimize using BootstrapFewShot
        from dspy.teleprompt import BootstrapFewShot
        
        optimizer = BootstrapFewShot(
            metric=self._quality_metric,
            max_bootstrapped_demos=10,
            max_labeled_demos=5
        )
        
        # Compile optimized program
        optimized_program = optimizer.compile(
            student=self.agent_program,
            trainset=training_data
        )
        
        # Save optimized prompts for this tenant
        await self._save_optimized_prompts(optimized_program)
        
        # Clear traces
        self.execution_traces = []
        
    def _quality_metric(self, example, prediction, trace=None):
        """Metric for DSPy optimization"""
        # Simplified: check if prediction is similar to gold answer
        from difflib import SequenceMatcher
        similarity = SequenceMatcher(None, example.answer, prediction.answer).ratio()
        return similarity
    
    async def _save_optimized_prompts(self, program):
        """Save tenant-specific optimized prompts"""
        prompts_path = f"/var/lib/enterprise-ai-ops/tenants/{self.tenant_id}/prompts.json"
        # Save logic here
        pass
```

### 3.6 Economic Tracking Integration

**Location**: `/home/lunark/projects/ai-native-agentic-org/ai-native-agentic-economic-sdk/`

**Integration**: Direct import in Enterprise Gateway

**Usage**:
```python
# enterprise_gateway/economics.py
from economic_sdk import EconomicTracker

class EnterpriseEconomicTracker:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id
        self.tracker = EconomicTracker(
            ledger_path=f"/var/lib/enterprise-ai-ops/tenants/{tenant_id}/economics.jsonl"
        )
        
    def track_llm_call(self, tokens_used: int, model_name: str, cost_per_token: float):
        """Track LLM API cost"""
        self.tracker.track_llm_call(
            tokens_used=tokens_used,
            model_name=model_name,
            cost_per_token=cost_per_token
        )
        
    def add_work_income(self, task_value: float, evaluation_score: float):
        """Add income if quality threshold met (>= 0.6)"""
        if evaluation_score >= 0.6:
            self.tracker.add_work_income(
                task_value=task_value,
                evaluation_score=evaluation_score
            )
    
    def compute_roi(self) -> Dict[str, float]:
        """Compute ROI metrics"""
        return {
            "roi": self.tracker.compute_roi(),
            "pay_rate": self.tracker.compute_pay_rate(),
            "sustainability": self.tracker.compute_sustainability_score()
        }
    
    def get_daily_summary(self) -> Dict[str, Any]:
        """Get daily cost/income summary"""
        # Read JSONL ledger and aggregate by day
        # Return: { "date": "2026-03-10", "cost": 45.23, "income": 125.00, "roi": 176% }
        pass
```

### 3.7 ClawWork Benchmark Integration

**Location**: `/home/lunark/projects/ai-native-agentic-org/ClawWork/`

**Integration**: Task value estimation using GDPVal

**Usage**:
```python
# enterprise_gateway/gdpval.py
from clawwork.gdpval import GDPValBenchmark, TaskCategory

class TaskValueEstimator:
    def __init__(self):
        self.benchmark = GDPValBenchmark()
        
    async def estimate_task_value(self, message: str) -> float:
        """Estimate task value using GDPVal benchmark"""
        
        # Classify task into one of 44 professions
        category = await self._classify_task(message)
        
        # Get BLS wage for this profession
        hourly_wage = self.benchmark.get_hourly_wage(category)
        
        # Estimate time required (simplified: use message length as proxy)
        estimated_hours = self._estimate_time(message)
        
        return hourly_wage * estimated_hours
    
    async def _classify_task(self, message: str) -> TaskCategory:
        """Classify task into GDPVal category using LLM"""
        # Use LLM to map message to one of 44 professions
        # Return TaskCategory enum
        pass
    
    def _estimate_time(self, message: str) -> float:
        """Estimate time required in hours"""
        # Simplified heuristic: 
        # - Short query (<100 chars): 0.1 hours (6 min)
        # - Medium query (100-500 chars): 0.25 hours (15 min)
        # - Long query (>500 chars): 0.5 hours (30 min)
        if len(message) < 100:
            return 0.1
        elif len(message) < 500:
            return 0.25
        else:
            return 0.5
```

---

## 4. Multi-Tenancy Model

### 4.1 Tenant Tiers

| Tier | Price/mo | Agents | Requests/day | Tokens/day | Storage | Support | SLA |
|------|----------|--------|--------------|------------|---------|---------|-----|
| Starter | $500 | 5 | 10,000 | 10M | 10GB | Email | 95% |
| Professional | $2,500 | 25 | 100,000 | 100M | 100GB | Slack | 99% |
| Enterprise | $10,000+ | Unlimited | Unlimited | Unlimited | 1TB+ | Dedicated | 99.9% |

### 4.2 Tenant Data Model

**Location**: `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/src/api/tenant_manager.py`

**Schema** (PostgreSQL):
```sql
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    tier VARCHAR(50) NOT NULL CHECK (tier IN ('FREE', 'STARTER', 'PROFESSIONAL', 'ENTERPRISE')),
    stripe_customer_id VARCHAR(255),
    stripe_subscription_id VARCHAR(255),
    status VARCHAR(50) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'cancelled')),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE tenant_quotas (
    tenant_id UUID PRIMARY KEY REFERENCES tenants(id) ON DELETE CASCADE,
    requests_per_minute INTEGER NOT NULL DEFAULT 10,
    requests_per_day INTEGER NOT NULL DEFAULT 10000,
    max_agents INTEGER NOT NULL DEFAULT 5,
    max_tokens_per_day BIGINT NOT NULL DEFAULT 10000000,
    max_storage_mb INTEGER NOT NULL DEFAULT 10240,
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE tenant_usage (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    requests_count INTEGER NOT NULL DEFAULT 0,
    tokens_used BIGINT NOT NULL DEFAULT 0,
    storage_used_mb INTEGER NOT NULL DEFAULT 0,
    cost_usd DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    income_usd DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    UNIQUE(tenant_id, date)
);

CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    email VARCHAR(255) NOT NULL UNIQUE,
    role VARCHAR(50) NOT NULL CHECK (role IN ('VIEWER', 'OPERATOR', 'ADMIN')),
    sso_provider VARCHAR(50),
    sso_user_id VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE permissions (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    permission VARCHAR(100) NOT NULL,
    PRIMARY KEY (user_id, permission)
);

CREATE INDEX idx_tenant_usage_date ON tenant_usage(tenant_id, date);
CREATE INDEX idx_users_tenant ON users(tenant_id);
```

### 4.3 RBAC Model

**Roles**:
- **VIEWER**: Read-only access to dashboards and logs
- **OPERATOR**: Execute agents, view results, no config changes
- **ADMIN**: Full access including tenant config, user management, billing

**Permissions**:
```python
# ai-native-self-learning-agents/src/api/rbac.py
from enum import Enum

class Permission(Enum):
    # Agent operations
    EXECUTE_AGENT = "execute_agent"
    VIEW_AGENT_RESULTS = "view_agent_results"
    CREATE_AGENT = "create_agent"
    UPDATE_AGENT = "update_agent"
    DELETE_AGENT = "delete_agent"
    
    # Tenant management
    VIEW_TENANT_CONFIG = "view_tenant_config"
    UPDATE_TENANT_CONFIG = "update_tenant_config"
    MANAGE_USERS = "manage_users"
    MANAGE_BILLING = "manage_billing"
    
    # Analytics
    VIEW_ANALYTICS = "view_analytics"
    EXPORT_DATA = "export_data"

class Role(Enum):
    VIEWER = "VIEWER"
    OPERATOR = "OPERATOR"
    ADMIN = "ADMIN"

ROLE_PERMISSIONS = {
    Role.VIEWER: [
        Permission.VIEW_AGENT_RESULTS,
        Permission.VIEW_TENANT_CONFIG,
        Permission.VIEW_ANALYTICS
    ],
    Role.OPERATOR: [
        Permission.EXECUTE_AGENT,
        Permission.VIEW_AGENT_RESULTS,
        Permission.VIEW_TENANT_CONFIG,
        Permission.VIEW_ANALYTICS
    ],
    Role.ADMIN: list(Permission)  # All permissions
}

async def check_permission(tenant_id: str, user_id: str, permission: Permission) -> bool:
    """Check if user has permission"""
    user = await get_user(user_id)
    if user.tenant_id != tenant_id:
        return False
    
    role = Role(user.role)
    return permission in ROLE_PERMISSIONS[role]
```

### 4.4 Quota Enforcement

```python
# enterprise_gateway/quotas.py
from datetime import date, datetime
from typing import Optional
import asyncio

class QuotaEnforcer:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id
        self.redis_client = redis.Redis()  # For rate limiting
        
    async def enforce_quotas(self, tenant: Tenant) -> None:
        """Enforce all quota limits"""
        
        # 1. Check requests per minute (rate limiting)
        rpm_key = f"quota:rpm:{self.tenant_id}:{datetime.now().strftime('%Y%m%d%H%M')}"
        current_rpm = await self.redis_client.incr(rpm_key)
        await self.redis_client.expire(rpm_key, 60)
        
        if current_rpm > tenant.quotas.requests_per_minute:
            raise QuotaExceededError(f"Rate limit exceeded: {tenant.quotas.requests_per_minute} req/min")
        
        # 2. Check requests per day
        usage = await self.get_daily_usage(date.today())
        if usage.requests_count >= tenant.quotas.requests_per_day:
            raise QuotaExceededError(f"Daily request limit exceeded: {tenant.quotas.requests_per_day}")
        
        # 3. Check tokens per day
        if usage.tokens_used >= tenant.quotas.max_tokens_per_day:
            raise QuotaExceededError(f"Daily token limit exceeded: {tenant.quotas.max_tokens_per_day}")
        
        # 4. Check storage
        if usage.storage_used_mb >= tenant.quotas.max_storage_mb:
            raise QuotaExceededError(f"Storage limit exceeded: {tenant.quotas.max_storage_mb}MB")
        
        # 5. Check max agents
        active_agents = await self.count_active_agents()
        if active_agents >= tenant.quotas.max_agents:
            raise QuotaExceededError(f"Max agents limit exceeded: {tenant.quotas.max_agents}")
    
    async def get_daily_usage(self, date: date) -> TenantUsage:
        """Get usage for specific date"""
        # Query tenant_usage table
        pass
    
    async def count_active_agents(self) -> int:
        """Count currently running agents for this tenant"""
        # Query nanoclaw orchestrator
        pass

class QuotaExceededError(Exception):
    pass
```

---

## 5. Agent Isolation Architecture

### 5.1 Container Isolation Strategy

**Goal**: Ensure zero cross-tenant data leakage while maintaining performance.

**Approach**:
1. **OS-level isolation**: Docker containers (Linux) or Apple Container (macOS)
2. **Memory isolation**: Separate CLAUDE.md per tenant, no shared memory
3. **Filesystem isolation**: Dedicated workspace directory, no home dir mounting
4. **Network isolation**: No network access by default (whitelist only)
5. **Resource limits**: CPU, memory, disk quotas enforced at container level

### 5.2 Security Model

**Threat Model**:
- Malicious tenant attempting to access other tenant data
- Agent attempting to escape container
- Agent attempting to exfiltrate data via network
- Resource exhaustion attacks (CPU, memory, disk)

**Mitigations**:

| Threat | Mitigation |
|--------|-----------|
| Cross-tenant data access | Separate containers, no shared volumes |
| Container escape | Seccomp profiles, AppArmor/SELinux, read-only root filesystem |
| Network exfiltration | Network isolation (--network none), whitelist egress |
| Resource exhaustion | CPU/memory/disk limits, container lifecycle timeouts |
| Privilege escalation | Non-root user in container, drop capabilities |

**Container Security Profile**:
```yaml
# nanoclaw/security/container-profile.yaml
apiVersion: v1
kind: SecurityContext
spec:
  runAsNonRoot: true
  runAsUser: 1000
  readOnlyRootFilesystem: true
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
  seccompProfile:
    type: RuntimeDefault
```

### 5.3 Memory Isolation

**CLAUDE.md Structure**:
```markdown
# Agent Memory for Tenant {tenant_id}

## Tenant Context
- Tenant ID: {tenant_id}
- Agent ID: {agent_id}
- Tier: {tier}
- Created: {timestamp}

## Conversation History
{last 500 lines of conversation}

## Learned Patterns
{tenant-specific optimized prompts from DSPy}

## Restrictions
- No access to other tenant data
- No home directory access
- Network: whitelist only
- Storage: {max_storage_mb}MB limit
```

**Isolation Verification**:
```python
# nanoclaw/tests/test_isolation.py
import pytest
import asyncio

async def test_cross_tenant_isolation():
    """Verify tenants cannot access each other's data"""
    orchestrator = NanoclawOrchestrator()
    
    # Spawn two containers for different tenants
    container_a = await orchestrator.spawnContainer({
        "tenantId": "tenant-a",
        "agentId": "agent-1",
        "memoryLimitMb": 512
    })
    
    container_b = await orchestrator.spawnContainer({
        "tenantId": "tenant-b",
        "agentId": "agent-1",
        "memoryLimitMb": 512
    })
    
    # Write secret to container A
    await orchestrator.executeInContainer(container_a.id, {
        "action": "write_file",
        "path": "/workspace/secret.txt",
        "content": "SECRET_DATA_A"
    })
    
    # Attempt to read from container B
    result = await orchestrator.executeInContainer(container_b.id, {
        "action": "read_file",
        "path": f"/containers/{container_a.id}/workspace/secret.txt"
    })
    
    # Should fail (file not found or permission denied)
    assert result["status"] == "error"
    assert "not found" in result["message"].lower() or "permission denied" in result["message"].lower()
```

---

## 6. Computer Use & Automation

### 6.1 OpenManus Capabilities

**Computer Use** (via `computer_use_tool.py`):
- Keyboard input simulation
- Mouse movement and clicks
- Screenshot capture
- Window management
- Clipboard operations

**Browser Use** (via `browser_use_tool.py`):
- Full browser automation (Playwright)
- Form filling
- Navigation
- Element interaction
- JavaScript execution

**Code Execution** (via `bash_tool.py`):
- Sandboxed shell execution
- File operations
- Git operations
- Package installation (within container)

### 6.2 Enterprise Use Cases

| Use Case | Tools Used | Example |
|----------|-----------|---------|
| Data entry automation | Computer Use + Browser Use | Fill CRM forms from spreadsheet |
| Report generation | Code Execution + File Ops | Generate PDF reports from database |
| Web scraping | Browser Use + Code Execution | Extract competitor pricing data |
| Testing automation | Computer Use + Browser Use | E2E testing of web applications |
| Document processing | Code Execution + File Ops | OCR invoices, extract structured data |
| Email automation | Browser Use + Code Execution | Respond to support tickets |

### 6.3 Safety Constraints

**Blocked Operations**:
```python
# openmanus/src/openmanus/tools/safety.py
BLOCKED_COMMANDS = [
    r"rm\s+-rf\s+/",  # Recursive delete root
    r":()\{\s*:\|\:&\s*\};:",  # Fork bomb
    r"curl.*\|\s*sh",  # Pipe to shell
    r"wget.*\|\s*sh",
    r"dd\s+if=/dev/zero",  # Disk fill
    r"mkfs\.",  # Format filesystem
    r"shutdown",
    r"reboot",
    r"init\s+0",
]

BLOCKED_BROWSER_ACTIONS = [
    "navigate to file://",  # Local file access
    "execute javascript: eval(",  # Arbitrary JS execution
    "download .exe",  # Executable downloads
]

def validate_command(cmd: str) -> bool:
    """Validate command is safe to execute"""
    for pattern in BLOCKED_COMMANDS:
        if re.search(pattern, cmd, re.IGNORECASE):
            raise UnsafeCommandError(f"Blocked command pattern: {pattern}")
    return True
```

---

## 7. Self-Learning Pipeline

### 7.1 Learning Loop

```
Execution → Quality Evaluation → Trace Collection → Optimization → Deployment
     ↑                                                                  │
     └──────────────────────────────────────────────────────────────────┘
```

### 7.2 DSPy Optimization

**Trigger**: Every 100 executions per tenant

**Process**:
1. Filter high-quality traces (score >= 0.8)
2. Prepare DSPy training data
3. Run BootstrapFewShot optimization
4. Save optimized prompts to tenant-specific storage
5. Reload prompts in next agent execution

**Metrics**:
- Quality improvement: Track average quality score before/after optimization
- ROI improvement: Track ROI before/after optimization
- Convergence: Stop optimization when improvement < 5%

### 7.3 PEFT Fine-Tuning

**Trigger**: Every 1,000 executions per tenant (optional, Enterprise tier only)

**Process**:
1. Collect high-quality execution traces
2. Prepare fine-tuning dataset (input/output pairs)
3. Fine-tune using LoRA (Low-Rank Adaptation)
4. Evaluate on held-out test set
5. Deploy if quality improvement >= 10%

**Implementation**:
```python
# ai-native-self-learning-agents/src/training/peft_trainer.py
from peft import LoraConfig, get_peft_model, TaskType
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

class PEFTTrainer:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id
        self.base_model = "anthropic/claude-3-sonnet"  # Placeholder
        
    async def fine_tune(self, traces: list[dict]) -> str:
        """Fine-tune model using PEFT (LoRA)"""
        
        # Prepare dataset
        dataset = self._prepare_dataset(traces)
        
        # Load base model
        model = AutoModelForCausalLM.from_pretrained(self.base_model)
        tokenizer = AutoTokenizer.from_pretrained(self.base_model)
        
        # Configure LoRA
        lora_config = LoraConfig(
            task_type=TaskType.CAUSAL_LM,
            r=8,  # Low-rank dimension
            lora_alpha=32,
            lora_dropout=0.1,
            target_modules=["q_proj", "v_proj"]
        )
        
        # Apply LoRA
        model = get_peft_model(model, lora_config)
        
        # Train
        trainer = Trainer(
            model=model,
            args=TrainingArguments(
                output_dir=f"/var/lib/enterprise-ai-ops/tenants/{self.tenant_id}/models",
                num_train_epochs=3,
                per_device_train_batch_size=4,
                learning_rate=3e-4
            ),
            train_dataset=dataset
        )
        
        trainer.train()
        
        # Save adapter weights
        model_path = f"/var/lib/enterprise-ai-ops/tenants/{self.tenant_id}/models/lora-adapter"
        model.save_pretrained(model_path)
        
        return model_path
    
    def _prepare_dataset(self, traces: list[dict]):
        """Convert traces to training dataset"""
        # Format: [{"input": request, "output": response}, ...]
        pass
```

---

## 8. Economic ROI Tracking

### 8.1 Cost Tracking

**Sources**:
- LLM API calls (tokens × cost per token)
- Infrastructure costs (container runtime, storage)
- Human-in-the-loop costs (ask_human tool usage)

**Implementation**:
```python
# enterprise_gateway/economics.py (extended)
class CostTracker:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id
        
    async def track_execution_cost(self, execution: TaskResult) -> float:
        """Track total cost of execution"""
        
        # 1. LLM API cost
        llm_cost = (execution.tokens_used / 1_000_000) * 3.0  # $3/MTok
        
        # 2. Infrastructure cost (simplified: $0.01 per container-minute)
        container_minutes = execution.execution_time_ms / 60000
        infra_cost = container_minutes * 0.01
        
        # 3. Human-in-the-loop cost (if ask_human was used)
        human_cost = 0.0
        if "ask_human" in execution.tools_used:
            human_cost = 5.0  # $5 per human intervention
        
        total_cost = llm_cost + infra_cost + human_cost
        
        # Record in ledger
        await self._record_cost(total_cost, {
            "llm_cost": llm_cost,
            "infra_cost": infra_cost,
            "human_cost": human_cost
        })
        
        return total_cost
    
    async def _record_cost(self, total: float, breakdown: dict):
        """Record cost in JSONL ledger"""
        # Append to economics.jsonl
        pass
```

### 8.2 Income Tracking

**Sources**:
- Task value estimation (GDPVal benchmark)
- Quality-adjusted income (only if score >= 0.6)

**Implementation**:
```python
# enterprise_gateway/economics.py (extended)
class IncomeTracker:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id
        self.gdpval = TaskValueEstimator()
        
    async def track_execution_income(self, request: str, quality_score: float) -> float:
        """Track income from execution"""
        
        # Estimate task value using GDPVal
        task_value = await self.gdpval.estimate_task_value(request)
        
        # Apply quality threshold (only count if >= 0.6)
        if quality_score >= 0.6:
            income = task_value * quality_score  # Quality-adjusted
        else:
            income = 0.0
        
        # Record in ledger
        await self._record_income(income, {
            "task_value": task_value,
            "quality_score": quality_score,
            "quality_adjusted": income
        })
        
        return income
    
    async def _record_income(self, total: float, breakdown: dict):
        """Record income in JSONL ledger"""
        # Append to economics.jsonl
        pass
```

### 8.3 ROI Dashboard

**Metrics**:
- **ROI**: (Income - Cost) / Cost × 100%
- **Pay Rate**: Income / Execution Time ($/hour equivalent)
- **Sustainability Score**: Income / Cost (must be > 1.0 for sustainability)
- **Quality Rate**: % of executions with score >= 0.6

**Implementation**:
```python
# enterprise_gateway/dashboard.py
from fastapi import APIRouter
from datetime import date, timedelta

router = APIRouter()

@router.get("/tenants/{tenant_id}/roi/daily")
async def get_daily_roi(tenant_id: str, days: int = 30):
    """Get daily ROI metrics for last N days"""
    
    end_date = date.today()
    start_date = end_date - timedelta(days=days)
    
    metrics = []
    for d in range(days):
        current_date = start_date + timedelta(days=d)
        usage = await get_daily_usage(tenant_id, current_date)
        
        roi = ((usage.income_usd - usage.cost_usd) / usage.cost_usd * 100) if usage.cost_usd > 0 else 0
        
        metrics.append({
            "date": current_date.isoformat(),
            "cost": float(usage.cost_usd),
            "income": float(usage.income_usd),
            "roi": roi,
            "requests": usage.requests_count,
            "tokens": usage.tokens_used
        })
    
    return {
        "tenant_id": tenant_id,
        "period": f"{start_date} to {end_date}",
        "metrics": metrics,
        "summary": {
            "total_cost": sum(m["cost"] for m in metrics),
            "total_income": sum(m["income"] for m in metrics),
            "average_roi": sum(m["roi"] for m in metrics) / len(metrics),
            "total_requests": sum(m["requests"] for m in metrics)
        }
    }

@router.get("/tenants/{tenant_id}/roi/agents")
async def get_agent_roi(tenant_id: str):
    """Get ROI breakdown by agent"""
    # Query economics ledger grouped by agent_id
    # Return: [{"agent_id": "...", "cost": ..., "income": ..., "roi": ...}, ...]
    pass

@router.get("/tenants/{tenant_id}/roi/benchmark")
async def get_benchmark_comparison(tenant_id: str):
    """Compare tenant ROI to ClawWork GDPVal benchmark"""
    
    # ClawWork benchmark: $19,915.68 in 8 hours = $2,489/hour
    benchmark_pay_rate = 2489.0
    
    # Get tenant pay rate
    tenant_metrics = await get_daily_roi(tenant_id, days=30)
    total_income = tenant_metrics["summary"]["total_income"]
    total_time_hours = tenant_metrics["summary"]["total_requests"] * 0.25  # Assume 15 min per request
    tenant_pay_rate = total_income / total_time_hours if total_time_hours > 0 else 0
    
    return {
        "tenant_id": tenant_id,
        "tenant_pay_rate": tenant_pay_rate,
        "benchmark_pay_rate": benchmark_pay_rate,
        "performance_ratio": tenant_pay_rate / benchmark_pay_rate,
        "status": "above_benchmark" if tenant_pay_rate > benchmark_pay_rate else "below_benchmark"
    }
```

---

## 9. Integration Points

### 9.1 OpenClaw → Enterprise Gateway

**Protocol**: HTTP REST API

**Endpoint**: `POST /v1/execute`

**Request**:
```json
{
  "channel_id": "slack-C12345",
  "tenant_id": "tenant-uuid",
  "user_id": "user-uuid",
  "message": "Generate Q4 sales report",
  "context": {
    "agent_id": "sales-agent",
    "session_id": "session-123"
  }
}
```

**Response**:
```json
{
  "request_id": "req-uuid",
  "response": "I've generated the Q4 sales report. Here are the key findings...",
  "cost_usd": 0.15,
  "income_usd": 12.50,
  "roi": 8233.33,
  "execution_time_ms": 4500
}
```

**OpenClaw Integration Code**:
```typescript
// openclaw/packages/enterprise/src/gateway-client.ts
import axios from 'axios';

export class EnterpriseGatewayClient {
  private baseUrl: string;
  private apiKey: string;
  
  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }
  
  async executeAgent(request: {
    channelId: string;
    tenantId: string;
    userId: string;
    message: string;
    context?: any;
  }): Promise<any> {
    const response = await axios.post(
      `${this.baseUrl}/v1/execute`,
      {
        channel_id: request.channelId,
        tenant_id: request.tenantId,
        user_id: request.userId,
        message: request.message,
        context: request.context
      },
      {
        headers: {
          'Authorization': `Bearer ${this.apiKey}`,
          'Content-Type': 'application/json'
        },
        timeout: 300000  // 5 min
      }
    );
    
    return response.data;
  }
}

// Usage in channel plugin
export class SlackEnterprisePlugin extends SlackPlugin {
  private gatewayClient: EnterpriseGatewayClient;
  
  async onMessage(message: SlackMessage) {
    // Extract tenant from Slack workspace ID
    const tenantId = await this.getTenantIdFromWorkspace(message.team_id);
    
    // Execute via enterprise gateway
    const result = await this.gatewayClient.executeAgent({
      channelId: message.channel,
      tenantId: tenantId,
      userId: message.user,
      message: message.text,
      context: {
        session_id: message.thread_ts || message.ts
      }
    });
    
    // Send response back to Slack
    await this.sendMessage(message.channel, result.response);
  }
}
```

### 9.2 Enterprise Gateway → Nanoclaw Orchestrator

**Protocol**: HTTP REST API

**Endpoints**:
- `POST /containers/spawn` - Spawn new container
- `POST /containers/:id/execute` - Execute task in container
- `DELETE /containers/:id` - Destroy container

**Client Code** (already shown in section 3.2):
```python
# enterprise_gateway/clients/nanoclaw.py
import httpx

class NanoclawClient:
    def __init__(self, base_url: str = "http://localhost:8080"):
        self.base_url = base_url
        self.client = httpx.AsyncClient()
    
    async def spawn_container(self, tenant_id: str, agent_id: str, memory_limit_mb: int):
        response = await self.client.post(
            f"{self.base_url}/containers/spawn",
            json={
                "tenantId": tenant_id,
                "agentId": agent_id,
                "memoryLimitMb": memory_limit_mb
            }
        )
        response.raise_for_status()
        return response.json()
    
    async def execute_in_container(self, container_id: str, task: dict):
        response = await self.client.post(
            f"{self.base_url}/containers/{container_id}/execute",
            json=task
        )
        response.raise_for_status()
        return response.json()
    
    async def destroy_container(self, container_id: str):
        response = await self.client.delete(
            f"{self.base_url}/containers/{container_id}"
        )
        response.raise_for_status()
```

### 9.3 Nanoclaw → OpenManus

**Protocol**: A2A (Agent-to-Agent) over HTTP

**Endpoint**: `POST /execute` (within container)

**Request**:
```json
{
  "query": "Generate Q4 sales report",
  "session_id": "session-123",
  "tools": ["computer_use", "browser_use", "code_execution"],
  "max_iterations": 10,
  "enable_computer_use": true,
  "enable_browser_use": true
}
```

**Response**:
```json
{
  "response": "I've generated the Q4 sales report...",
  "tokens_used": 50000,
  "tools_used": ["browser_use", "code_execution"],
  "iterations": 5,
  "status": "success"
}
```

### 9.4 Enterprise Gateway → Self-Learning Pipeline

**Protocol**: Async message queue (Redis Streams or RabbitMQ)

**Message Format**:
```json
{
  "tenant_id": "tenant-uuid",
  "agent_id": "sales-agent",
  "request": "Generate Q4 sales report",
  "response": "I've generated...",
  "quality_score": 0.85,
  "roi": 8233.33,
  "timestamp": "2026-03-10T14:30:00Z"
}
```

**Consumer** (Self-Learning Agent):
```python
# ai-native-self-learning-agents/src/workers/learning_worker.py
import asyncio
import redis.asyncio as redis

async def learning_worker():
    """Background worker for continuous learning"""
    
    r = await redis.Redis()
    
    while True:
        # Read from stream
        messages = await r.xread(
            {"learning:traces": "$"},
            count=10,
            block=5000
        )
        
        for stream, msg_list in messages:
            for msg_id, data in msg_list:
                # Process trace
                tenant_id = data[b"tenant_id"].decode()
                agent = EnterpriseLearningAgent(tenant_id)
                
                await agent.collect_trace(
                    request=data[b"request"].decode(),
                    response=data[b"response"].decode(),
                    quality_score=float(data[b"quality_score"]),
                    roi=float(data[b"roi"])
                )
                
                # Acknowledge
                await r.xack("learning:traces", "learning-group", msg_id)

if __name__ == "__main__":
    asyncio.run(learning_worker())
```

---

## 10. Data Flow Walkthrough

### End-to-End Flow: Slack Message → Agent Execution → Response

**Step 1: User sends message in Slack**
```
User: "@AIAgent Generate Q4 sales report"
  ↓
Slack API → Webhook → OpenClaw Slack Plugin
```

**Step 2: OpenClaw normalizes and routes**
```typescript
// openclaw/packages/channels/slack/src/plugin.ts
async onMessage(slackMessage: SlackMessage) {
  const normalized = {
    channelId: slackMessage.channel,
    tenantId: await this.getTenantId(slackMessage.team_id),
    userId: slackMessage.user,
    message: slackMessage.text.replace(/@AIAgent\s+/, ''),
    context: { session_id: slackMessage.thread_ts }
  };
  
  // Forward to Enterprise Gateway
  const result = await this.gatewayClient.executeAgent(normalized);
  
  // Send response back to Slack
  await this.slack.chat.postMessage({
    channel: slackMessage.channel,
    text: result.response,
    thread_ts: slackMessage.thread_ts
  });
}
```

**Step 3: Enterprise Gateway authenticates and authorizes**
```python
# enterprise_gateway/main.py
@app.post("/v1/execute")
async def execute_agent_task(request: EnterpriseRequest, credentials: HTTPAuthorizationCredentials):
    # 1. Authenticate tenant
    tenant = await authenticate_tenant(credentials.credentials)
    
    # 2. Check RBAC permission
    has_permission = await check_permission(
        tenant.id,
        request.user_id,
        Permission.EXECUTE_AGENT
    )
    if not has_permission:
        raise HTTPException(403, "Permission denied")
    
    # 3. Enforce quotas
    try:
        await enforce_quotas(tenant)
    except QuotaExceededError as e:
        raise HTTPException(429, str(e))
    
    # 4. Orchestrate execution
    gateway = EnterpriseGateway(tenant)
    response = await gateway.handle_request(request)
    
    return response
```

**Step 4: Spawn isolated container**
```python
# enterprise_gateway/orchestrator.py
async def handle_request(self, request: EnterpriseRequest):
    # Spawn container via Nanoclaw
    container = await self.nanoclaw_client.spawn_container(
        tenant_id=self.tenant.id,
        agent_id=request.context.get("agent_id", "default"),
        memory_limit_mb=self.tenant.quotas.max_storage_mb
    )
```

**Step 5: Nanoclaw creates isolated environment**
```typescript
// nanoclaw/src/enterprise/orchestrator.ts
async spawnContainer(config: ContainerConfig): Promise<Container> {
    const containerId = crypto.randomUUID();
    const containerDir = path.join(this.baseDir, 'containers', containerId);
    
    // Create isolated directory
    await fs.mkdir(containerDir, { recursive: true });
    
    // Create tenant-specific CLAUDE.md
    await fs.writeFile(
        path.join(containerDir, 'memory', 'CLAUDE.md'),
        `# Agent Memory for Tenant ${config.tenantId}...`
    );
    
    // Start Docker container
    await this.startContainerProcess(this.buildContainerCommand(containerId, config));
    
    return { id: containerId, tenantId: config.tenantId, status: 'running', ... };
}
```

**Step 6: Execute task via OpenManus A2A**
```python
# enterprise_gateway/orchestrator.py (continued)
    task_result = await self.openmanus_client.invoke_agent(
        container_id=container.id,
        query=request.message,
        session_id=request.context.get("session_id"),
        tools=[ToolType.BROWSER_USE, ToolType.CODE_EXECUTION]
    )
```

**Step 7: OpenManus executes with computer use**
```python
# openmanus/src/openmanus/executor.py
async def execute(self, query: str, tools: list[str]):
    # Plan task
    plan = await self.planner.create_plan(query)
    
    # Execute steps
    for step in plan.steps:
        if step.tool == "browser_use":
            # Open browser, navigate, extract data
            result = await self.browser_tool.execute(step.params)
        elif step.tool == "code_execution":
            # Generate Python code, execute in sandbox
            result = await self.bash_tool.execute(step.params)
    
    # Aggregate results
    return self.aggregate_results(results)
```

**Step 8: Track economics**
```python
# enterprise_gateway/orchestrator.py (continued)
    # Track cost
    self.economic_tracker.track_llm_call(
        tokens_used=task_result.tokens_used,
        model_name="claude-sonnet-4",
        cost_per_token=0.000003
    )
    
    # Track income (if quality >= 0.6)
    if task_result.quality_score >= 0.6:
        task_value = await self._estimate_task_value(request.message)
        self.economic_tracker.add_work_income(
            task_value=task_value,
            evaluation_score=task_result.quality_score
        )
    
    # Compute ROI
    metrics = self.economic_tracker.compute_roi()
```

**Step 9: Cleanup and return**
```python
# enterprise_gateway/orchestrator.py (continued)
    finally:
        # Destroy container
        await self.nanoclaw_client.destroy_container(container.id)
    
    return EnterpriseResponse(
        request_id=str(uuid.uuid4()),
        response=task_result.response,
        cost_usd=task_result.cost_usd,
        income_usd=task_value,
        roi=metrics["roi"],
        execution_time_ms=execution_time_ms
    )
```

**Step 10: Response delivered to Slack**
```typescript
// openclaw/packages/channels/slack/src/plugin.ts (continued)
  await this.slack.chat.postMessage({
    channel: slackMessage.channel,
    text: result.response,
    thread_ts: slackMessage.thread_ts
  });
```

**Step 11: Async learning (background)**
```python
# enterprise_gateway/orchestrator.py (fire and forget)
    await self.publish_learning_trace({
        "tenant_id": self.tenant.id,
        "request": request.message,
        "response": task_result.response,
        "quality_score": task_result.quality_score,
        "roi": metrics["roi"]
    })
```

---

## 11. Security & Compliance

### 11.1 SOC 2 Readiness

**Type II Controls**:

| Control | Implementation |
|---------|---------------|
| Access Control | RBAC with VIEWER/OPERATOR/ADMIN roles |
| Encryption at Rest | PostgreSQL TDE, encrypted container volumes |
| Encryption in Transit | TLS 1.3 for all HTTP/gRPC, mTLS for inter-service |
| Audit Logging | All API calls logged with user, timestamp, action |
| Data Isolation | Container-based multi-tenancy, no shared volumes |
| Backup & Recovery | Daily PostgreSQL backups, 30-day retention |
| Incident Response | Automated alerts for quota violations, failed auth |
| Vulnerability Management | Weekly dependency scans (Snyk), monthly pen tests |

**Audit Log Schema**:
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(255),
    ip_address INET,
    user_agent TEXT,
    request_body JSONB,
    response_status INTEGER,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_tenant_date ON audit_logs(tenant_id, created_at);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id, created_at);
```

### 11.2 GDPR Compliance

**Data Subject Rights**:

| Right | Implementation |
|-------|---------------|
| Right to Access | `GET /tenants/{id}/data-export` (JSON export) |
| Right to Erasure | `DELETE /tenants/{id}` (cascade delete all data) |
| Right to Portability | Data export in JSON format |
| Right to Rectification | `PATCH /users/{id}` (update personal data) |

**Data Retention**:
- Active tenant data: Retained indefinitely while subscription active
- Cancelled tenant data: 90-day grace period, then deleted
- Audit logs: 7 years retention (compliance requirement)
- Economic ledgers: 7 years retention (tax compliance)

**Data Processing Agreement (DPA)**:
- Standard Contractual Clauses (SCCs) for EU customers
- Data Processing Addendum (DPA) signed at contract time
- Sub-processor list published and updated quarterly

### 11.3 SSO Integration

**Supported Providers**:
- Okta
- Azure AD
- Google Workspace
- OneLogin
- Generic SAML 2.0

**Implementation**:
```python
# ai-native-self-learning-agents/src/api/sso.py
from fastapi import APIRouter, HTTPException
import httpx
from jose import jwt

router = APIRouter()

@router.post("/auth/sso/saml")
async def saml_callback(saml_response: str):
    """Handle SAML SSO callback"""
    
    # Validate SAML response
    user_info = validate_saml_response(saml_response)
    
    # Find or create user
    user = await get_or_create_sso_user(
        email=user_info["email"],
        sso_provider="saml",
        sso_user_id=user_info["subject"]
    )
    
    # Generate JWT
    token = jwt.encode(
        {
            "sub": str(user.id),
            "tenant_id": str(user.tenant_id),
            "role": user.role,
            "exp": datetime.utcnow() + timedelta(hours=24)
        },
        settings.JWT_SECRET,
        algorithm="HS256"
    )
    
    return {"access_token": token, "token_type": "bearer"}

@router.post("/auth/sso/oidc")
async def oidc_callback(code: str, state: str):
    """Handle OIDC SSO callback (Okta, Azure AD, Google)"""
    
    # Exchange code for token
    token_response = await httpx.post(
        settings.OIDC_TOKEN_ENDPOINT,
        data={
            "grant_type": "authorization_code",
            "code": code,
            "redirect_uri": settings.OIDC_REDIRECT_URI,
            "client_id": settings.OIDC_CLIENT_ID,
            "client_secret": settings.OIDC_CLIENT_SECRET
        }
    )
    
    token_data = token_response.json()
    
    # Decode ID token
    user_info = jwt.decode(
        token_data["id_token"],
        settings.OIDC_PUBLIC_KEY,
        algorithms=["RS256"],
        audience=settings.OIDC_CLIENT_ID
    )
    
    # Find or create user
    user = await get_or_create_sso_user(
        email=user_info["email"],
        sso_provider="oidc",
        sso_user_id=user_info["sub"]
    )
    
    # Generate JWT
    token = jwt.encode(
        {
            "sub": str(user.id),
            "tenant_id": str(user.tenant_id),
            "role": user.role,
            "exp": datetime.utcnow() + timedelta(hours=24)
        },
        settings.JWT_SECRET,
        algorithm="HS256"
    )
    
    return {"access_token": token, "token_type": "bearer"}
```

---

## 12. MVP Scope

### 12.1 In-Scope Features

**Core Platform**:
- ✅ Multi-tenant architecture (PostgreSQL schema)
- ✅ RBAC (VIEWER/OPERATOR/ADMIN roles)
- ✅ Quota enforcement (requests/day, tokens/day)
- ✅ Container isolation (Docker-based)
- ✅ Economic tracking (cost + income + ROI)

**Channels** (MVP: 3 channels):
- ✅ Slack
- ✅ Microsoft Teams
- ✅ Email (SMTP/IMAP)

**Agent Capabilities**:
- ✅ Computer use (keyboard, mouse, screenshot)
- ✅ Browser automation (Playwright)
- ✅ Code execution (Python, Bash)
- ✅ File operations

**Self-Learning**:
- ✅ Execution trace collection
- ✅ Quality evaluation (LLM-as-judge)
- ✅ DSPy prompt optimization (triggered manually for MVP)

**Dashboard**:
- ✅ Daily ROI metrics
- ✅ Cost/income breakdown
- ✅ Agent usage analytics
- ✅ GDPVal benchmark comparison

**Auth & Billing**:
- ✅ JWT authentication
- ✅ Stripe integration (checkout, portal, webhooks)
- ✅ Tier management (Starter, Professional, Enterprise)

### 12.2 Out-of-Scope (Post-MVP)

**Deferred to v1.1+**:
- ❌ SSO integration (Okta, Azure AD)
- ❌ PEFT fine-tuning (Enterprise tier only)
- ❌ Automatic DSPy optimization (manual trigger for MVP)
- ❌ Advanced analytics (cohort analysis, churn prediction)
- ❌ Custom channel plugins (API available, no UI)
- ❌ Multi-region deployment
- ❌ SOC 2 Type II certification (start audit post-MVP)
- ❌ Mobile app (web dashboard only)
- ❌ Webhooks for external integrations
- ❌ GraphQL API (REST only for MVP)

### 12.3 MVP Success Criteria

**Technical**:
- [ ] 10 concurrent tenants supported
- [ ] < 5s p95 response time for agent execution
- [ ] 99% uptime over 2-week period
- [ ] Zero cross-tenant data leakage (verified by isolation tests)
- [ ] All 6 QA gates passing (harness-engineering)

**Business**:
- [ ] 3 paying customers (Starter or Professional tier)
- [ ] $1,500+ MRR (Monthly Recurring Revenue)
- [ ] 80%+ customer satisfaction (CSAT survey)
- [ ] 50%+ of executions with quality score >= 0.6
- [ ] Positive ROI for at least 2 customers

---

## 13. Two-Week Sprint Plan

### Week 1: Foundation & Core Platform

#### Day 1-2: Infrastructure Setup
**Owner**: DevOps + Backend

**Tasks**:
- [ ] Set up PostgreSQL database (tenants, users, quotas, usage, audit_logs)
- [ ] Deploy Redis for rate limiting and message queue
- [ ] Create enterprise-gateway FastAPI project structure
- [ ] Set up Docker Compose for local development
- [ ] Configure CI/CD pipeline (GitHub Actions)

**Deliverables**:
- Database schema deployed
- enterprise-gateway skeleton running locally
- CI/CD pipeline green

#### Day 3-4: Multi-Tenancy & RBAC
**Owner**: Backend

**Tasks**:
- [ ] Implement TenantManager (CRUD, tier management)
- [ ] Implement RBAC (Permission, Role, check_permission)
- [ ] Implement QuotaEnforcer (rate limiting, daily limits)
- [ ] Implement JWT authentication
- [ ] Write unit tests (pytest)

**Deliverables**:
- `POST /tenants/`, `GET /tenants/{id}`, `PATCH /tenants/{id}`
- `POST /auth/login`, `POST /auth/register`
- RBAC middleware working
- 80%+ test coverage

#### Day 5-6: Container Isolation
**Owner**: Backend + Infrastructure

**Tasks**:
- [ ] Extend nanoclaw with NanoclawOrchestrator
- [ ] Implement spawn_container, execute_in_container, destroy_container
- [ ] Create tenant-specific CLAUDE.md template
- [ ] Implement container security profile (seccomp, AppArmor)
- [ ] Write isolation tests (verify no cross-tenant access)

**Deliverables**:
- `POST /containers/spawn` API working
- Container isolation verified (tests passing)
- Docker images built and tagged

#### Day 7: Integration: Gateway ↔ Nanoclaw
**Owner**: Backend

**Tasks**:
- [ ] Implement NanoclawClient in enterprise-gateway
- [ ] Implement EnterpriseGateway.handle_request()
- [ ] Integrate container lifecycle (spawn → execute → destroy)
- [ ] Write integration tests

**Deliverables**:
- End-to-end flow working (API → container → response)
- Integration tests passing

---

### Week 2: Agent Execution & Economics

#### Day 8-9: OpenManus Integration
**Owner**: Backend + AI

**Tasks**:
- [ ] Create OpenManusA2AClient
- [ ] Implement invoke_agent (A2A protocol)
- [ ] Implement quality evaluation (LLM-as-judge)
- [ ] Enable computer use + browser use tools
- [ ] Write tool safety validators

**Deliverables**:
- OpenManus executing tasks in containers
- Quality scores computed
- Safety validators blocking dangerous commands

#### Day 10: Economic Tracking
**Owner**: Backend

**Tasks**:
- [ ] Integrate ai-native-agentic-economic-sdk
- [ ] Implement CostTracker (LLM + infra costs)
- [ ] Implement IncomeTracker (GDPVal integration)
- [ ] Implement TaskValueEstimator
- [ ] Create economics JSONL ledger per tenant

**Deliverables**:
- Cost/income tracked for every execution
- ROI computed correctly
- JSONL ledger persisted

#### Day 11: OpenClaw Integration
**Owner**: Frontend + Backend

**Tasks**:
- [ ] Create EnterpriseGatewayClient (TypeScript)
- [ ] Extend Slack plugin with enterprise integration
- [ ] Extend Teams plugin with enterprise integration
- [ ] Extend Email plugin with enterprise integration
- [ ] Test end-to-end: Slack → Gateway → Agent → Response

**Deliverables**:
- Slack, Teams, Email channels working
- Messages routed to enterprise gateway
- Responses delivered to channels

#### Day 12: ROI Dashboard
**Owner**: Frontend + Backend

**Tasks**:
- [ ] Implement `GET /tenants/{id}/roi/daily`
- [ ] Implement `GET /tenants/{id}/roi/agents`
- [ ] Implement `GET /tenants/{id}/roi/benchmark`
- [ ] Create simple web dashboard (React or Vue)
- [ ] Display cost/income/ROI charts

**Deliverables**:
- ROI API endpoints working
- Dashboard showing metrics
- GDPVal benchmark comparison visible

#### Day 13: Billing Integration
**Owner**: Backend

**Tasks**:
- [ ] Integrate Stripe (checkout, portal, webhooks)
- [ ] Implement `POST /billing/checkout` (create checkout session)
- [ ] Implement `POST /billing/portal` (customer portal)
- [ ] Implement `POST /webhooks/stripe` (subscription events)
- [ ] Test subscription lifecycle (create → active → cancel)

**Deliverables**:
- Stripe integration working
- Subscription tiers enforced
- Webhooks handling subscription changes

#### Day 14: Testing & Launch Prep
**Owner**: Full Team

**Tasks**:
- [ ] Run full QA gate suite (harness-engineering)
- [ ] Fix any failing tests
- [ ] Load testing (10 concurrent tenants, 100 req/min)
- [ ] Security review (OWASP Top 10 checklist)
- [ ] Write deployment runbook
- [ ] Deploy to staging environment
- [ ] Smoke tests on staging
- [ ] Deploy to production

**Deliverables**:
- All 6 QA gates passing
- Load tests passing (< 5s p95)
- Production deployment successful
- MVP live and ready for customers

---

## 14. Business Model & ACV Projections

### 14.1 Pricing Tiers (Detailed)

| Feature | Starter | Professional | Enterprise |
|---------|---------|--------------|------------|
| **Price** | $500/mo | $2,500/mo | $10,000+/mo |
| **Agents** | 5 | 25 | Unlimited |
| **Requests/day** | 10,000 | 100,000 | Unlimited |
| **Tokens/day** | 10M | 100M | Unlimited |
| **Storage** | 10GB | 100GB | 1TB+ |
| **Channels** | 3 (Slack, Teams, Email) | All 22+ | All + Custom |
| **Support** | Email (48hr SLA) | Slack (4hr SLA) | Dedicated Slack (1hr SLA) |
| **SLA** | 95% uptime | 99% uptime | 99.9% uptime |
| **SSO** | ❌ | ✅ | ✅ |
| **RBAC** | Basic (2 roles) | Advanced (3 roles) | Custom roles |
| **Self-Learning** | Manual trigger | Auto (weekly) | Auto (daily) + PEFT |
| **ROI Dashboard** | Basic | Advanced | Custom reports |
| **Dedicated Infra** | ❌ | ❌ | ✅ |
| **Professional Services** | ❌ | Add-on ($5K/week) | Included (40hrs/mo) |

### 14.2 Revenue Model

**Subscription Revenue**:
- Starter: $500/mo × 12 = $6,000 ACV
- Professional: $2,500/mo × 12 = $30,000 ACV
- Enterprise: $10,000/mo × 12 = $120,000 ACV

**Usage Overage**:
- $0.01 per request over quota
- Example: Professional tier customer using 150K requests/day
  - Overage: 50K requests/day × 30 days = 1.5M requests/mo
  - Overage revenue: 1.5M × $0.01 = $15,000/mo
  - Total: $2,500 + $15,000 = $17,500/mo

**Professional Services**:
- Custom agent development: $15,000/week
- Integration consulting: $10,000/week
- Training & onboarding: $5,000/session

**Target Customer Mix (Year 1)**:
- 20 Starter customers: 20 × $6K = $120K ARR
- 10 Professional customers: 10 × $30K = $300K ARR
- 3 Enterprise customers: 3 × $120K = $360K ARR
- **Total ARR**: $780K

**Target Customer Mix (Year 2)**:
- 50 Starter: $300K ARR
- 30 Professional: $900K ARR
- 10 Enterprise: $1.2M ARR
- **Total ARR**: $2.4M ARR

### 14.3 Unit Economics

**Customer Acquisition Cost (CAC)**:
- Marketing spend: $50K/year
- Sales team (2 AEs): $200K/year
- Total: $250K/year
- Customers acquired (Year 1): 33
- **CAC**: $250K / 33 = $7,576 per customer

**Lifetime Value (LTV)**:
- Average ACV: $780K / 33 = $23,636
- Average customer lifetime: 3 years (assumed)
- Gross margin: 80% (SaaS standard)
- **LTV**: $23,636 × 3 × 0.8 = $56,726

**LTV:CAC Ratio**: $56,726 / $7,576 = **7.5x** (healthy, target > 3x)

**Payback Period**: $7,576 / ($23,636 × 0.8 / 12) = **4.8 months** (healthy, target < 12 months)

### 14.4 Key Metrics

**North Star Metric**: **Net Dollar Retention (NDR)**

**Target NDR**: 130%+ (indicates strong expansion revenue)

**NDR Drivers**:
- Upsell from Starter → Professional: 30% of Starter customers/year
- Upsell from Professional → Enterprise: 20% of Professional customers/year
- Usage overage: 40% of customers exceed quota
- Professional services attach rate: 50% of Enterprise customers

**Other Key Metrics**:
- **Monthly Recurring Revenue (MRR)**: Track monthly
- **Churn Rate**: Target < 5% monthly (< 60% annual)
- **Customer Satisfaction (CSAT)**: Target > 80%
- **Net Promoter Score (NPS)**: Target > 50
- **Agent Utilization**: Avg requests per agent per day
- **ROI per Customer**: Avg ROI across all customers (target > 200%)

---

## 15. Risk & Mitigation

### 15.1 Technical Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Container escape vulnerability** | Low | Critical | Regular security audits, seccomp profiles, AppArmor, read-only root FS |
| **Cross-tenant data leakage** | Low | Critical | Isolation tests in CI/CD, separate volumes, no shared memory |
| **LLM API rate limits** | Medium | High | Multi-provider fallback (Claude, GPT-4, Gemini), request queuing |
| **Performance degradation at scale** | Medium | High | Load testing, auto-scaling, container pool optimization |
| **Economic tracking inaccuracy** | Low | Medium | Validate against ClawWork benchmark, manual audits |
| **Self-learning regression** | Medium | Medium | A/B testing, rollback mechanism, quality thresholds |

**Mitigation Details**:

**Container Escape**:
- Weekly vulnerability scans (Trivy, Snyk)
- Monthly penetration testing
- Bug bounty program ($500-$5,000 rewards)
- Incident response plan (< 4hr detection, < 24hr remediation)

**Cross-Tenant Leakage**:
- Automated isolation tests in CI/CD (fail build if leakage detected)
- Quarterly third-party security audits
- Encryption at rest for all tenant data
- Separate database schemas per tenant (future enhancement)

**LLM API Rate Limits**:
- Multi-provider support (Claude primary, GPT-4 fallback, Gemini tertiary)
- Request queuing with exponential backoff
- Customer communication: "High demand, request queued, ETA 2 min"

**Performance at Scale**:
- Load testing: 100 concurrent tenants, 1,000 req/min
- Auto-scaling: Kubernetes HPA (Horizontal Pod Autoscaler)
- Container pool: Pre-warm 10 containers per tenant tier
- Database optimization: Connection pooling, read replicas

### 15.2 Business Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Low customer adoption** | Medium | Critical | Freemium tier, free trial (14 days), aggressive marketing |
| **High churn rate** | Medium | High | Customer success team, proactive support, ROI dashboards |
| **Competitive pressure** | High | High | Differentiation (economic ROI focus), fast iteration, customer lock-in |
| **Regulatory changes (AI)** | Medium | Medium | Legal counsel, compliance monitoring, flexible architecture |
| **Pricing too low** | Low | Medium | Annual pricing review, value-based pricing, usage-based upsell |

**Mitigation Details**:

**Low Adoption**:
- Freemium tier: 1 agent, 1,000 requests/day, free forever
- 14-day free trial for Starter/Professional (no credit card required)
- Content marketing: Case studies, ROI calculators, webinars
- Partnerships: Slack App Directory, Microsoft Teams Store

**High Churn**:
- Customer success team (1 CSM per 20 Enterprise customers)
- Proactive outreach: Weekly check-ins, monthly business reviews
- ROI dashboards: Show value delivered (income vs cost)
- Feature requests: Prioritize top customer requests

**Competitive Pressure**:
- Differentiation: Only platform with built-in economic ROI tracking
- Fast iteration: 2-week sprint cycles, monthly releases
- Customer lock-in: Self-learning improves over time (switching cost)
- Open-source components: Build community, reduce vendor lock-in fear

### 15.3 Operational Risks

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **Key person dependency** | Medium | High | Documentation, cross-training, backup on-call rotation |
| **Infrastructure outage** | Low | Critical | Multi-region deployment (future), 99.9% SLA, incident response |
| **Data loss** | Low | Critical | Daily backups, 30-day retention, disaster recovery plan |
| **Security breach** | Low | Critical | SOC 2 compliance, penetration testing, incident response plan |

**Mitigation Details**:

**Key Person Dependency**:
- Comprehensive documentation (AGENTS.md in every project)
- Cross-training: Every engineer knows 2+ components
- On-call rotation: 3+ engineers per rotation
- Runbooks: Step-by-step guides for common incidents

**Infrastructure Outage**:
- Multi-region deployment (post-MVP): US-East, EU-West
- 99.9% SLA for Enterprise tier (43 min downtime/month)
- Incident response: < 15 min detection, < 1 hr mitigation
- Status page: Real-time uptime monitoring (status.example.com)

**Data Loss**:
- Daily PostgreSQL backups (automated)
- 30-day retention (compliance requirement)
- Disaster recovery plan: < 4 hr RTO (Recovery Time Objective)
- Quarterly DR drills (test restore from backup)

---

## 16. Success Metrics

### 16.1 Technical Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **API Response Time (p95)** | < 5s | Prometheus + Grafana |
| **Container Spawn Time (p95)** | < 2s | Custom instrumentation |
| **Uptime** | 99.9% (Enterprise), 99% (Pro), 95% (Starter) | Pingdom, UptimeRobot |
| **Error Rate** | < 1% | Sentry, CloudWatch |
| **Test Coverage** | > 80% | pytest-cov, vitest |
| **Security Vulnerabilities** | 0 critical, < 5 high | Snyk, Trivy |

### 16.2 Product Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Agent Utilization** | > 50 requests/agent/day | PostgreSQL analytics |
| **Quality Score (avg)** | > 0.7 | Economic tracker |
| **ROI per Customer (avg)** | > 200% | Economic tracker |
| **Self-Learning Improvement** | +10% quality after 1,000 executions | A/B testing |
| **Channel Coverage** | 22+ channels supported | OpenClaw plugin count |

### 16.3 Business Metrics

| Metric | Target (Year 1) | Measurement |
|--------|-----------------|-------------|
| **MRR** | $65K (Month 12) | Stripe dashboard |
| **ARR** | $780K | Stripe + manual calc |
| **Customer Count** | 33 | PostgreSQL |
| **Churn Rate** | < 5% monthly | Cohort analysis |
| **NDR** | > 130% | Manual calculation |
| **CSAT** | > 80% | Quarterly survey |
| **NPS** | > 50 | Quarterly survey |
| **CAC** | < $8K | Marketing spend / customers |
| **LTV:CAC** | > 7x | LTV / CAC |
| **Payback Period** | < 5 months | CAC / (ACV × margin / 12) |

### 16.4 Customer Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Time to First Value** | < 1 hour (from signup to first agent execution) | Product analytics |
| **Activation Rate** | > 80% (% of signups that execute 10+ agents) | Product analytics |
| **Weekly Active Users** | > 60% of total users | Product analytics |
| **Support Ticket Volume** | < 5 tickets/customer/month | Zendesk |
| **Support Resolution Time** | < 4 hours (Professional), < 1 hour (Enterprise) | Zendesk |

---

## 17. Appendix

### 17.1 File Structure (New Components)

```
ai-native-agentic-org/
├── enterprise-gateway/                    # NEW: Core orchestration service
│   ├── src/
│   │   ├── main.py                       # FastAPI app
│   │   ├── orchestrator.py               # EnterpriseGateway class
│   │   ├── clients/
│   │   │   ├── nanoclaw.py               # NanoclawClient
│   │   │   └── openmanus.py              # OpenManusA2AClient
│   │   ├── economics/
│   │   │   ├── tracker.py                # EnterpriseEconomicTracker
│   │   │   └── gdpval.py                 # TaskValueEstimator
│   │   ├── quotas.py                     # QuotaEnforcer
│   │   ├── dashboard.py                  # ROI API endpoints
│   │   └── models.py                     # Pydantic models
│   ├── tests/
│   │   ├── test_orchestrator.py
│   │   ├── test_quotas.py
│   │   └── test_economics.py
│   ├── Dockerfile
│   ├── requirements.txt
│   └── README.md
│
├── nanoclaw/
│   └── src/
│       └── enterprise/                   # NEW: Enterprise extensions
│           ├── orchestrator.ts           # NanoclawOrchestrator
│           └── api.ts                    # HTTP API server
│
├── openmanus/
│   └── src/openmanus/
│       └── enterprise/                   # NEW: Enterprise extensions
│           ├── a2a_client.py             # OpenManusA2AClient
│           └── safety.py                 # Safety validators
│
├── openclaw/
│   └── packages/
│       └── enterprise/                   # NEW: Enterprise extensions
│           ├── src/
│           │   ├── gateway-client.ts     # EnterpriseGatewayClient
│           │   └── plugins/
│           │       ├── slack-enterprise.ts
│           │       ├── teams-enterprise.ts
│           │       └── email-enterprise.ts
│           └── package.json
│
├── ai-native-self-learning-agents/
│   └── src/
│       ├── agents/
│       │   └── enterprise_learning_agent.py  # NEW: Enterprise learning
│       └── workers/
│           └── learning_worker.py        # NEW: Background worker
│
└── documentation/
    └── UC5_enterprise_ai_ops.md          # THIS DOCUMENT
```

### 17.2 Environment Variables

```bash
# enterprise-gateway/.env
DATABASE_URL=postgresql://user:pass@localhost:5432/enterprise_ai_ops
REDIS_URL=redis://localhost:6379/0
JWT_SECRET=your-secret-key-here
STRIPE_API_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
NANOCLAW_API_URL=http://localhost:8080
OPENMANUS_API_URL=http://localhost:8000
OPENCLAW_API_URL=http://localhost:3000

# Tier quotas (defaults)
STARTER_REQUESTS_PER_DAY=10000
STARTER_MAX_AGENTS=5
PROFESSIONAL_REQUESTS_PER_DAY=100000
PROFESSIONAL_MAX_AGENTS=25

# LLM API keys
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GOOGLE_API_KEY=...

# Monitoring
SENTRY_DSN=https://...@sentry.io/...
PROMETHEUS_PORT=9090
```

### 17.3 Deployment Architecture (Kubernetes)

```yaml
# k8s/enterprise-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: enterprise-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: enterprise-gateway
  template:
    metadata:
      labels:
        app: enterprise-gateway
    spec:
      containers:
      - name: enterprise-gateway
        image: enterprise-gateway:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: enterprise-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: enterprise-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 10
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: enterprise-gateway
spec:
  selector:
    app: enterprise-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: enterprise-gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: enterprise-gateway
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### 17.4 API Reference (Quick Reference)

**Authentication**:
```bash
# Login
curl -X POST https://api.example.com/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "..."}'

# Response: {"access_token": "eyJ...", "token_type": "bearer"}
```

**Execute Agent**:
```bash
curl -X POST https://api.example.com/v1/execute \
  -H "Authorization: Bearer eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "channel_id": "slack-C12345",
    "tenant_id": "tenant-uuid",
    "user_id": "user-uuid",
    "message": "Generate Q4 sales report"
  }'
```

**Get ROI Metrics**:
```bash
curl -X GET https://api.example.com/tenants/tenant-uuid/roi/daily?days=30 \
  -H "Authorization: Bearer eyJ..."
```

**Create Tenant**:
```bash
curl -X POST https://api.example.com/tenants/ \
  -H "Authorization: Bearer eyJ..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Acme Corp",
    "tier": "PROFESSIONAL"
  }'
```

---

## Conclusion

This specification provides a complete, implementation-ready blueprint for the Enterprise AI Ops Platform (UC5). All components are defined with exact code snippets, integration points, and data flows. The 2-week sprint plan is actionable, and the business model is validated with proven economic benchmarks ($19K/8hr from ClawWork).

**Next Steps**:
1. Review and approve this specification
2. Assign engineering resources to sprint tasks
3. Kick off Day 1 (infrastructure setup)
4. Daily standups to track progress
5. Deploy MVP to production on Day 14

**Questions or Clarifications**: Contact @zzragida or file an issue in the documentation repository.

---

**Document Version**: 1.0  
**Last Updated**: 2026-03-10  
**Approved By**: [Pending]  
**Status**: Ready for Implementation
