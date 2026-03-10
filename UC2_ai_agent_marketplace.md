# UC2: AI 에이전트 마켓플레이스 (AI Agent Marketplace)

**Version:** 1.0.0  
**Date:** 2026-03-10  
**Status:** Implementation-Ready Specification  
**Author:** AI-Native Agentic Organization  

---

## 1. 개요 및 비전 (Overview & Vision)

### 1.1 Executive Summary

The AI Agent Marketplace is a production-grade platform enabling developers to publish, monetize, and distribute AI agents while allowing organizations to discover, subscribe to, and deploy specialized agents across their communication channels. This marketplace bridges four existing ecosystem projects:

- **agency-agents**: 75 pre-built agents across 9 business divisions
- **openclaw**: Multi-channel gateway (22+ channels, 42 extensions)
- **ai-native-self-learning-agents**: Multi-tenant backend with RBAC and payments
- **nanobot**: Lightweight agent runtime (<500 LOC core)
- **ai-native-agentic-economic-sdk**: Revenue tracking and ROI analytics

### 1.2 Core Value Propositions

**For Agent Creators:**
- Publish agents to 10,000+ potential enterprise customers
- Earn 70% revenue share on subscriptions
- Zero infrastructure costs (platform handles hosting, billing, delivery)
- Built-in analytics dashboard (usage, revenue, ratings)

**For Organizations:**
- Access 75+ production-ready agents (day 1 catalog)
- Deploy agents to Slack, Discord, Telegram, email, web chat in <5 minutes
- Pay-per-agent pricing (no vendor lock-in)
- Enterprise-grade security (RBAC, audit logs, data isolation)

**For End Users:**
- Unified interface across all channels (openclaw)
- Consistent agent experience (nanobot runtime)
- Transparent pricing and usage tracking

### 1.3 Success Criteria (6 Months)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Active Agents | 150+ | Agent registry count |
| Paying Tenants | 200+ | Stripe active subscriptions |
| Monthly Transactions | $50K+ | economic-sdk revenue ledger |
| Agent Utilization | 70%+ | Agents with >10 invocations/week |
| Creator Earnings | $35K+ | 70% of platform revenue |
| Channel Coverage | 15+ | openclaw channel integrations |

---

## 2. 아키텍처 다이어그램 (Architecture Diagram)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          END USER INTERFACES                             │
│  Slack │ Discord │ Telegram │ Email │ Web Chat │ MS Teams │ WhatsApp   │
└────────┬────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    OPENCLAW GATEWAY (TypeScript)                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │  AgentMarketplaceExtension (NEW)                                 │   │
│  │  - Route messages to purchased agents                            │   │
│  │  - Check subscription via self-learning-agents API               │   │
│  │  - Handle agent responses (text, media, reactions)               │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└────────┬────────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              AGENT REGISTRY SERVICE (NEW - Python FastAPI)               │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │  POST   /agents              - Publish agent                     │   │
│  │  GET    /agents              - List/search agents                │   │
│  │  GET    /agents/{id}         - Agent details                     │   │
│  │  POST   /agents/{id}/install - Subscribe tenant to agent         │   │
│  │  DELETE /agents/{id}/install - Unsubscribe tenant                │   │
│  │  GET    /agents/{id}/metrics - Usage/revenue analytics           │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│  Database: PostgreSQL (agents, versions, ratings, reviews)              │
└────────┬────────────────────────────────────────────────────────────────┘
         │
         ├──────────────────┬──────────────────┬──────────────────┐
         ▼                  ▼                  ▼                  ▼
┌─────────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────────┐
│ SELF-LEARNING   │ │   NANOBOT    │ │ AGENCY-AGENTS│ │  ECONOMIC-SDK    │
│    AGENTS       │ │   RUNTIME    │ │   CATALOG    │ │   TRACKING       │
│ (Multi-tenant)  │ │ (Execution)  │ │ (75 agents)  │ │ (Revenue/ROI)    │
├─────────────────┤ ├──────────────┤ ├──────────────┤ ├──────────────────┤
│ • TenantManager │ │ • AgentLoop  │ │ • Engineering│ │ • track_llm_call │
│ • RBAC          │ │ • ToolRegistry│ │ • Design     │ │ • add_work_income│
│ • Stripe        │ │ • ExecTool   │ │ • Marketing  │ │ • compute_roi    │
│ • Quotas        │ │ • WebFetch   │ │ • PM/Testing │ │ • JSONL ledger   │
└─────────────────┘ └──────────────┘ └──────────────┘ └──────────────────┘

DATA FLOW:
1. User sends message via Slack → openclaw gateway
2. AgentMarketplaceExtension checks tenant subscription (self-learning-agents)
3. If authorized, routes to Agent Registry Service
4. Registry spawns nanobot AgentLoop with agent config
5. Agent executes, economic-sdk tracks costs/revenue
6. Response flows back through openclaw → Slack
```

---

## 3. 컴포넌트 책임 (Component Responsibilities)

### 3.1 Agent Registry Service (NEW)

**Location:** `/home/lunark/projects/ai-native-agentic-org/agent-registry-service/`

**Responsibilities:**
- Store agent metadata (name, description, category, pricing, creator)
- Version management (semantic versioning, changelog)
- Search and discovery (full-text search, category filters, ratings)
- Installation tracking (which tenants have which agents)
- Analytics aggregation (usage counts, revenue per agent)
- Creator payouts (calculate 70/30 split, generate invoices)

**Tech Stack:**
- FastAPI 0.115+
- PostgreSQL 16+ (agents, versions, installations, reviews)
- Redis 7+ (caching, rate limiting)
- Stripe API (payment processing)
- Docker + Kubernetes deployment

**Database Schema:**

```sql
CREATE TABLE agents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    creator_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(100) NOT NULL,
    icon_url TEXT,
    price_tier VARCHAR(50) NOT NULL, -- FREE, STARTER, PRO, ENTERPRISE
    monthly_price_cents INTEGER NOT NULL,
    is_published BOOLEAN DEFAULT FALSE,
    total_installs INTEGER DEFAULT 0,
    avg_rating DECIMAL(3,2) DEFAULT 0.0,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE agent_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
    version VARCHAR(50) NOT NULL, -- semver: 1.2.3
    config_json JSONB NOT NULL, -- nanobot AgentLoop config
    changelog TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(agent_id, version)
);

CREATE TABLE agent_installations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL, -- from self-learning-agents
    version_id UUID REFERENCES agent_versions(id),
    installed_at TIMESTAMPTZ DEFAULT NOW(),
    uninstalled_at TIMESTAMPTZ,
    UNIQUE(agent_id, tenant_id)
);

CREATE TABLE agent_reviews (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL,
    user_id UUID NOT NULL,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    review_text TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE agent_usage_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID REFERENCES agents(id) ON DELETE CASCADE,
    tenant_id UUID NOT NULL,
    invocation_count INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    total_cost_cents INTEGER DEFAULT 0,
    date DATE NOT NULL,
    UNIQUE(agent_id, tenant_id, date)
);

CREATE INDEX idx_agents_category ON agents(category);
CREATE INDEX idx_agents_published ON agents(is_published);
CREATE INDEX idx_installations_tenant ON agent_installations(tenant_id);
CREATE INDEX idx_usage_logs_date ON agent_usage_logs(date);
```

### 3.2 openclaw AgentMarketplaceExtension (NEW)

**Location:** `/home/lunark/projects/ai-native-agentic-org/openclaw/extensions/agent-marketplace/`

**Responsibilities:**
- Intercept incoming messages from all channels
- Check if message targets a marketplace agent (e.g., `@marketing-analyst`)
- Verify tenant subscription via Agent Registry Service API
- Route message to nanobot runtime
- Stream agent response back to channel
- Handle errors (quota exceeded, agent not found, subscription expired)

**Integration Points:**
- Implements `ChannelPlugin` interface from `@openclaw/types`
- Registers via `package.json` openclaw field
- Uses `@openclaw/gateway` for message routing
- Calls Agent Registry Service REST API

### 3.3 agency-agents Deployment Bridge (NEW)

**Location:** `/home/lunark/projects/ai-native-agentic-org/agency-agents/marketplace-bridge/`

**Responsibilities:**
- Convert agency-agents `.md` format to nanobot `AgentLoop` config
- Map agent tools to nanobot tool permissions
- Generate agent metadata for registry (category, description, pricing)
- Bulk import 75 agents to marketplace on initial deployment

**Conversion Logic:**
```python
# Input: agency-agents/agents/engineering/backend-engineer.md
# Output: nanobot AgentLoop config JSON

def convert_agent_to_nanobot_config(agent_md_path: str) -> dict:
    """
    Parse YAML frontmatter + Markdown body.
    Return nanobot-compatible config.
    """
    with open(agent_md_path) as f:
        content = f.read()
    
    # Split frontmatter and body
    parts = content.split('---', 2)
    frontmatter = yaml.safe_load(parts[1])
    instructions = parts[2].strip()
    
    return {
        "name": frontmatter["name"],
        "model": "claude-sonnet-4-5",
        "temperature": 0.7,
        "max_iterations": 10,
        "system_prompt": instructions,
        "tools": infer_tools_from_instructions(instructions),
        "metadata": {
            "category": infer_category(agent_md_path),
            "description": frontmatter.get("description", ""),
            "color": frontmatter.get("color", "#000000")
        }
    }
```

### 3.4 ai-native-self-learning-agents (EXISTING)

**Location:** `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/`

**Responsibilities (Marketplace-Specific):**
- Tenant subscription management (FREE/STARTER/PRO/ENTERPRISE)
- RBAC enforcement (who can install/uninstall agents)
- Quota tracking (max_agents per tier)
- Stripe webhook handling (subscription.created, subscription.deleted)

**New API Endpoints:**

```python
# src/api/routes/marketplace.py

@router.get("/tenants/{tenant_id}/agents")
async def list_tenant_agents(tenant_id: UUID, db: Session = Depends(get_db)):
    """List all agents installed for this tenant."""
    installations = db.query(AgentInstallation).filter(
        AgentInstallation.tenant_id == tenant_id,
        AgentInstallation.uninstalled_at.is_(None)
    ).all()
    return [{"agent_id": i.agent_id, "version": i.version_id} for i in installations]

@router.post("/tenants/{tenant_id}/agents/{agent_id}/install")
async def install_agent(
    tenant_id: UUID,
    agent_id: UUID,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_user)
):
    """Install agent for tenant (requires ADMIN role)."""
    if not has_permission(current_user, Permission.MANAGE_AGENTS):
        raise HTTPException(403, "Insufficient permissions")
    
    tenant = db.query(Tenant).filter(Tenant.id == tenant_id).first()
    if not tenant:
        raise HTTPException(404, "Tenant not found")
    
    # Check quota
    current_count = db.query(AgentInstallation).filter(
        AgentInstallation.tenant_id == tenant_id,
        AgentInstallation.uninstalled_at.is_(None)
    ).count()
    
    if current_count >= tenant.tier.max_agents:
        raise HTTPException(429, f"Agent quota exceeded ({tenant.tier.max_agents})")
    
    # Create installation
    installation = AgentInstallation(
        agent_id=agent_id,
        tenant_id=tenant_id,
        version_id=get_latest_version(agent_id, db).id
    )
    db.add(installation)
    db.commit()
    
    return {"status": "installed", "agent_id": agent_id}
```

### 3.5 nanobot Runtime (EXISTING)

**Location:** `/home/lunark/projects/ai-native-agentic-org/nanobot/`

**Responsibilities:**
- Execute agent logic (AgentLoop)
- Manage tool permissions (ExecTool, ReadFile, WebFetch, etc.)
- Stream responses to openclaw
- Track token usage for economic-sdk

**Marketplace Integration:**

```python
# New module: nanobot/marketplace_runtime.py

from nanobot import AgentLoop, MessageBus
from economic_sdk import EconomicTracker

class MarketplaceAgentRunner:
    def __init__(self, agent_config: dict, tenant_id: str):
        self.agent_config = agent_config
        self.tenant_id = tenant_id
        self.tracker = EconomicTracker(ledger_path=f"/var/lib/marketplace/{tenant_id}.jsonl")
        
    async def run(self, user_message: str) -> str:
        """Execute agent and track economics."""
        bus = MessageBus()
        loop = AgentLoop(
            bus=bus,
            provider="anthropic",
            model=self.agent_config["model"],
            workspace="/tmp/agent-workspace",
            max_iterations=self.agent_config["max_iterations"],
            temperature=self.agent_config["temperature"]
        )
        
        # Track LLM call
        self.tracker.track_llm_call(
            model=self.agent_config["model"],
            input_tokens=len(user_message.split()),
            output_tokens=0,  # Updated after response
            cost=0.0  # Calculated based on Anthropic pricing
        )
        
        response = await loop.run(user_message)
        
        # Track revenue (if agent produces value)
        self.tracker.add_work_income(
            task_description=f"Agent {self.agent_config['name']} execution",
            evaluation_score=0.8,  # Placeholder: integrate with quality scoring
            amount=self.agent_config["monthly_price_cents"] / 30 / 100  # Daily proration
        )
        
        return response
```

### 3.6 ai-native-agentic-economic-sdk (EXISTING)

**Location:** `/home/lunark/projects/ai-native-agentic-org/ai-native-agentic-economic-sdk/`

**Responsibilities:**
- Track per-agent LLM costs (input/output tokens, model pricing)
- Track per-agent revenue (subscription fees, usage-based charges)
- Calculate ROI per agent (revenue / costs)
- Generate creator payout reports (70% revenue share)

**New Aggregation Functions:**

```python
# economic_sdk/marketplace_analytics.py

from economic_sdk import EconomicTracker
from datetime import datetime, timedelta
from pathlib import Path

class MarketplaceAnalytics:
    def __init__(self, ledger_dir: Path):
        self.ledger_dir = ledger_dir
        
    def compute_agent_revenue(self, agent_id: str, days: int = 30) -> dict:
        """Aggregate revenue for specific agent over time period."""
        total_revenue = 0.0
        total_costs = 0.0
        invocation_count = 0
        
        # Scan all tenant ledgers
        for ledger_path in self.ledger_dir.glob("*.jsonl"):
            tracker = EconomicTracker(ledger_path=str(ledger_path))
            
            # Filter entries by agent_id and date range
            cutoff_date = datetime.now() - timedelta(days=days)
            for entry in tracker.ledger:
                if entry.get("agent_id") == agent_id and entry["timestamp"] > cutoff_date:
                    if entry["type"] == "llm_call":
                        total_costs += entry["cost"]
                    elif entry["type"] == "work_income":
                        total_revenue += entry["amount"]
                        invocation_count += 1
        
        return {
            "agent_id": agent_id,
            "period_days": days,
            "total_revenue": total_revenue,
            "total_costs": total_costs,
            "net_profit": total_revenue - total_costs,
            "roi": (total_revenue / total_costs - 1) * 100 if total_costs > 0 else 0,
            "invocation_count": invocation_count,
            "avg_revenue_per_invocation": total_revenue / invocation_count if invocation_count > 0 else 0
        }
    
    def compute_creator_payout(self, creator_id: str, month: str) -> dict:
        """Calculate 70% revenue share for agent creator."""
        # Get all agents by this creator
        agent_ids = get_agent_ids_by_creator(creator_id)
        
        total_platform_revenue = 0.0
        for agent_id in agent_ids:
            stats = self.compute_agent_revenue(agent_id, days=30)
            total_platform_revenue += stats["total_revenue"]
        
        creator_share = total_platform_revenue * 0.70
        platform_share = total_platform_revenue * 0.30
        
        return {
            "creator_id": creator_id,
            "month": month,
            "total_platform_revenue": total_platform_revenue,
            "creator_payout": creator_share,
            "platform_revenue": platform_share,
            "agent_count": len(agent_ids)
        }
```

---

## 4. 마켓플레이스 데이터 모델 (Marketplace Data Model)

### 4.1 Agent Schema

```typescript
// openclaw/extensions/agent-marketplace/types.ts

export interface Agent {
  id: string; // UUID
  creatorId: string; // UUID
  name: string; // "Marketing Analyst"
  slug: string; // "marketing-analyst"
  description: string; // "Analyzes marketing campaigns and provides insights"
  category: AgentCategory;
  iconUrl?: string;
  priceTier: PriceTier;
  monthlyPriceCents: number;
  isPublished: boolean;
  totalInstalls: number;
  avgRating: number; // 0.0 - 5.0
  createdAt: Date;
  updatedAt: Date;
}

export enum AgentCategory {
  ENGINEERING = "engineering",
  DESIGN = "design",
  MARKETING = "marketing",
  PRODUCT = "product",
  TESTING = "testing",
  SUPPORT = "support",
  SALES = "sales",
  OPERATIONS = "operations",
  FINANCE = "finance"
}

export enum PriceTier {
  FREE = "FREE",
  STARTER = "STARTER",
  PRO = "PRO",
  ENTERPRISE = "ENTERPRISE"
}

export interface AgentVersion {
  id: string;
  agentId: string;
  version: string; // semver: "1.2.3"
  configJson: NanobotConfig;
  changelog?: string;
  isActive: boolean;
  createdAt: Date;
}

export interface NanobotConfig {
  name: string;
  model: string;
  temperature: number;
  maxIterations: number;
  systemPrompt: string;
  tools: string[]; // ["exec", "read_file", "web_fetch"]
  metadata: {
    category: string;
    description: string;
    color: string;
  };
}
```

### 4.2 Subscription Schema

```typescript
// ai-native-self-learning-agents/src/models/subscription.ts

export interface AgentSubscription {
  id: string;
  agentId: string;
  tenantId: string;
  versionId: string;
  installedAt: Date;
  uninstalledAt?: Date;
  status: SubscriptionStatus;
}

export enum SubscriptionStatus {
  ACTIVE = "active",
  SUSPENDED = "suspended",
  CANCELLED = "cancelled"
}

export interface TenantQuota {
  tenantId: string;
  tier: TenantTier;
  maxAgents: number;
  currentAgents: number;
  requestsPerMinute: number;
  requestsPerDay: number;
  maxTokensPerDay: number;
}

export enum TenantTier {
  FREE = "FREE", // 3 agents
  STARTER = "STARTER", // 25 agents, $29/mo
  PRO = "PRO", // 100 agents, $99/mo
  ENTERPRISE = "ENTERPRISE" // unlimited, $499/mo+
}
```

### 4.3 Analytics Schema

```typescript
// agent-registry-service/models/analytics.ts

export interface AgentUsageMetrics {
  agentId: string;
  tenantId: string;
  date: Date;
  invocationCount: number;
  totalTokens: number;
  totalCostCents: number;
  avgResponseTimeMs: number;
  errorCount: number;
}

export interface AgentRevenueMetrics {
  agentId: string;
  period: "day" | "week" | "month";
  totalRevenue: number;
  totalCosts: number;
  netProfit: number;
  roi: number; // percentage
  activeSubscriptions: number;
  newSubscriptions: number;
  churnedSubscriptions: number;
}

export interface CreatorPayoutReport {
  creatorId: string;
  month: string; // "2026-03"
  totalPlatformRevenue: number;
  creatorPayout: number; // 70%
  platformRevenue: number; // 30%
  agentCount: number;
  topAgent: {
    agentId: string;
    name: string;
    revenue: number;
  };
}
```

---

## 5. 통합 지점 (Integration Points)

### 5.1 openclaw ↔ Agent Registry Service

**Protocol:** REST API over HTTPS  
**Authentication:** JWT tokens (issued by self-learning-agents)

```typescript
// openclaw/extensions/agent-marketplace/client.ts

import axios from 'axios';

export class AgentRegistryClient {
  private baseUrl: string;
  private apiKey: string;

  constructor(baseUrl: string, apiKey: string) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  async checkSubscription(tenantId: string, agentSlug: string): Promise<boolean> {
    const response = await axios.get(
      `${this.baseUrl}/tenants/${tenantId}/agents`,
      {
        headers: { Authorization: `Bearer ${this.apiKey}` }
      }
    );
    return response.data.some((a: any) => a.slug === agentSlug);
  }

  async getAgentConfig(agentId: string): Promise<NanobotConfig> {
    const response = await axios.get(
      `${this.baseUrl}/agents/${agentId}/config`,
      {
        headers: { Authorization: `Bearer ${this.apiKey}` }
      }
    );
    return response.data;
  }

  async logUsage(agentId: string, tenantId: string, metrics: UsageMetrics): Promise<void> {
    await axios.post(
      `${this.baseUrl}/agents/${agentId}/usage`,
      { tenantId, ...metrics },
      {
        headers: { Authorization: `Bearer ${this.apiKey}` }
      }
    );
  }
}
```

### 5.2 Agent Registry Service ↔ self-learning-agents

**Protocol:** REST API over HTTPS  
**Authentication:** Service-to-service JWT

```python
# agent-registry-service/integrations/self_learning_client.py

import httpx
from typing import Optional

class SelfLearningAgentsClient:
    def __init__(self, base_url: str, api_key: str):
        self.base_url = base_url
        self.api_key = api_key
        self.client = httpx.AsyncClient(
            headers={"Authorization": f"Bearer {api_key}"}
        )
    
    async def get_tenant(self, tenant_id: str) -> Optional[dict]:
        """Fetch tenant details including tier and quotas."""
        response = await self.client.get(f"{self.base_url}/tenants/{tenant_id}")
        if response.status_code == 200:
            return response.json()
        return None
    
    async def check_quota(self, tenant_id: str, resource: str) -> bool:
        """Check if tenant has quota for resource (e.g., 'agents')."""
        tenant = await self.get_tenant(tenant_id)
        if not tenant:
            return False
        
        if resource == "agents":
            current = tenant["current_agents"]
            max_allowed = tenant["tier"]["max_agents"]
            return current < max_allowed
        
        return False
    
    async def increment_agent_count(self, tenant_id: str) -> None:
        """Increment tenant's agent count after installation."""
        await self.client.post(
            f"{self.base_url}/tenants/{tenant_id}/increment-agents"
        )
```

### 5.3 Agent Registry Service ↔ nanobot

**Protocol:** Python subprocess (local) or gRPC (distributed)  
**Execution Model:** Spawn nanobot process per agent invocation

```python
# agent-registry-service/runtime/nanobot_executor.py

import asyncio
import json
from pathlib import Path
from nanobot import AgentLoop, MessageBus

class NanobotExecutor:
    def __init__(self, workspace_root: Path):
        self.workspace_root = workspace_root
    
    async def execute_agent(
        self,
        agent_config: dict,
        user_message: str,
        tenant_id: str
    ) -> str:
        """Spawn nanobot AgentLoop and execute user message."""
        workspace = self.workspace_root / tenant_id / agent_config["name"]
        workspace.mkdir(parents=True, exist_ok=True)
        
        bus = MessageBus()
        loop = AgentLoop(
            bus=bus,
            provider="anthropic",
            model=agent_config["model"],
            workspace=str(workspace),
            max_iterations=agent_config["max_iterations"],
            temperature=agent_config["temperature"]
        )
        
        # Set system prompt
        loop.system_prompt = agent_config["system_prompt"]
        
        # Execute
        response = await loop.run(user_message)
        
        return response
```

### 5.4 Agent Registry Service ↔ economic-sdk

**Protocol:** Python library import  
**Integration:** Direct function calls

```python
# agent-registry-service/analytics/economic_integration.py

from economic_sdk import EconomicTracker
from pathlib import Path

class MarketplaceEconomicTracker:
    def __init__(self, ledger_dir: Path):
        self.ledger_dir = ledger_dir
    
    def track_agent_invocation(
        self,
        agent_id: str,
        tenant_id: str,
        model: str,
        input_tokens: int,
        output_tokens: int,
        cost: float
    ):
        """Track LLM call for agent invocation."""
        ledger_path = self.ledger_dir / f"{tenant_id}.jsonl"
        tracker = EconomicTracker(ledger_path=str(ledger_path))
        
        tracker.track_llm_call(
            model=model,
            input_tokens=input_tokens,
            output_tokens=output_tokens,
            cost=cost,
            metadata={"agent_id": agent_id}
        )
    
    def track_agent_revenue(
        self,
        agent_id: str,
        tenant_id: str,
        amount: float,
        evaluation_score: float = 0.8
    ):
        """Track revenue from agent subscription."""
        ledger_path = self.ledger_dir / f"{tenant_id}.jsonl"
        tracker = EconomicTracker(ledger_path=str(ledger_path))
        
        tracker.add_work_income(
            task_description=f"Agent {agent_id} subscription",
            evaluation_score=evaluation_score,
            amount=amount,
            metadata={"agent_id": agent_id}
        )
```

### 5.5 agency-agents ↔ Agent Registry Service

**Protocol:** CLI tool + REST API  
**Workflow:** Bulk import on initial deployment

```bash
#!/bin/bash
# agency-agents/marketplace-bridge/import-agents.sh

set -e

REGISTRY_URL="https://marketplace.ai-native.org/api"
API_KEY="${MARKETPLACE_API_KEY}"
AGENTS_DIR="/home/lunark/projects/ai-native-agentic-org/agency-agents/agents"

echo "Importing agents from agency-agents to marketplace..."

for category_dir in "$AGENTS_DIR"/*; do
  category=$(basename "$category_dir")
  
  for agent_file in "$category_dir"/*.md; do
    agent_name=$(basename "$agent_file" .md)
    
    echo "Processing: $category/$agent_name"
    
    # Convert to nanobot config
    config_json=$(python3 marketplace-bridge/convert.py "$agent_file")
    
    # Publish to registry
    curl -X POST "$REGISTRY_URL/agents" \
      -H "Authorization: Bearer $API_KEY" \
      -H "Content-Type: application/json" \
      -d "$config_json"
    
    echo "✓ Published $agent_name"
  done
done

echo "Import complete!"
```

---

## 6. 글루 코드 사양 (Glue Code Specification)

### 6.1 openclaw AgentMarketplaceExtension

**File:** `/home/lunark/projects/ai-native-agentic-org/openclaw/extensions/agent-marketplace/index.ts`

```typescript
import type { ChannelPlugin, Message, MessageContext } from '@openclaw/types';
import { AgentRegistryClient } from './client';
import { NanobotExecutor } from './executor';

export class AgentMarketplaceExtension implements ChannelPlugin {
  id = 'agent-marketplace';
  meta = {
    name: 'Agent Marketplace',
    version: '1.0.0',
    description: 'Route messages to marketplace agents',
    author: 'AI-Native Org'
  };

  private registryClient: AgentRegistryClient;
  private executor: NanobotExecutor;

  constructor(config: { registryUrl: string; apiKey: string }) {
    this.registryClient = new AgentRegistryClient(config.registryUrl, config.apiKey);
    this.executor = new NanobotExecutor();
  }

  async onMessage(message: Message, context: MessageContext): Promise<void> {
    // Check if message mentions an agent (e.g., "@marketing-analyst")
    const agentMention = this.extractAgentMention(message.text);
    if (!agentMention) {
      return; // Not targeting a marketplace agent
    }

    // Verify subscription
    const hasAccess = await this.registryClient.checkSubscription(
      context.tenantId,
      agentMention
    );

    if (!hasAccess) {
      await context.reply({
        text: `❌ Agent "${agentMention}" not installed. Visit marketplace to subscribe.`,
        threadId: message.threadId
      });
      return;
    }

    // Get agent config
    const agentConfig = await this.registryClient.getAgentConfig(agentMention);

    // Execute agent
    const startTime = Date.now();
    try {
      const response = await this.executor.execute(
        agentConfig,
        message.text,
        context.tenantId
      );

      await context.reply({
        text: response,
        threadId: message.threadId
      });

      // Log usage
      const duration = Date.now() - startTime;
      await this.registryClient.logUsage(agentConfig.id, context.tenantId, {
        invocationCount: 1,
        responseTimeMs: duration,
        success: true
      });
    } catch (error) {
      await context.reply({
        text: `⚠️ Agent error: ${error.message}`,
        threadId: message.threadId
      });

      await this.registryClient.logUsage(agentConfig.id, context.tenantId, {
        invocationCount: 1,
        responseTimeMs: Date.now() - startTime,
        success: false,
        errorMessage: error.message
      });
    }
  }

  private extractAgentMention(text: string): string | null {
    const match = text.match(/@([a-z0-9-]+)/);
    return match ? match[1] : null;
  }
}
```

**File:** `/home/lunark/projects/ai-native-agentic-org/openclaw/extensions/agent-marketplace/executor.ts`

```typescript
import { spawn } from 'child_process';
import type { NanobotConfig } from './types';

export class NanobotExecutor {
  async execute(
    config: NanobotConfig,
    userMessage: string,
    tenantId: string
  ): Promise<string> {
    return new Promise((resolve, reject) => {
      const nanobotProcess = spawn('python3', [
        '-m', 'nanobot.marketplace_runtime',
        '--config', JSON.stringify(config),
        '--tenant-id', tenantId,
        '--message', userMessage
      ]);

      let stdout = '';
      let stderr = '';

      nanobotProcess.stdout.on('data', (data) => {
        stdout += data.toString();
      });

      nanobotProcess.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      nanobotProcess.on('close', (code) => {
        if (code === 0) {
          resolve(stdout.trim());
        } else {
          reject(new Error(`Nanobot failed: ${stderr}`));
        }
      });

      // Timeout after 60 seconds
      setTimeout(() => {
        nanobotProcess.kill();
        reject(new Error('Agent execution timeout'));
      }, 60000);
    });
  }
}
```

**File:** `/home/lunark/projects/ai-native-agentic-org/openclaw/extensions/agent-marketplace/package.json`

```json
{
  "name": "@openclaw/agent-marketplace",
  "version": "1.0.0",
  "description": "Agent Marketplace extension for openclaw",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "openclaw": {
    "type": "extension",
    "entrypoint": "dist/index.js",
    "capabilities": ["messaging", "agent-routing"]
  },
  "dependencies": {
    "@openclaw/types": "workspace:*",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.3.0",
    "vitest": "^1.0.0"
  },
  "scripts": {
    "build": "tsdown src/index.ts --dts",
    "test": "vitest run",
    "lint": "oxlint src/"
  }
}
```

### 6.2 Agent Registry Service Core

**File:** `/home/lunark/projects/ai-native-agentic-org/agent-registry-service/main.py`

```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from sqlalchemy.orm import Session
from typing import List, Optional
import uvicorn

from .database import get_db, engine, Base
from .models import Agent, AgentVersion, AgentInstallation
from .schemas import AgentCreate, AgentResponse, AgentInstallRequest
from .integrations.self_learning_client import SelfLearningAgentsClient
from .analytics.economic_integration import MarketplaceEconomicTracker

# Create tables
Base.metadata.create_all(bind=engine)

app = FastAPI(title="Agent Registry Service", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Clients
self_learning_client = SelfLearningAgentsClient(
    base_url="https://api.self-learning.ai-native.org",
    api_key="SERVICE_API_KEY"
)

economic_tracker = MarketplaceEconomicTracker(
    ledger_dir="/var/lib/marketplace/ledgers"
)

@app.post("/agents", response_model=AgentResponse)
async def create_agent(agent: AgentCreate, db: Session = Depends(get_db)):
    """Publish new agent to marketplace."""
    db_agent = Agent(
        creator_id=agent.creator_id,
        name=agent.name,
        slug=agent.slug,
        description=agent.description,
        category=agent.category,
        price_tier=agent.price_tier,
        monthly_price_cents=agent.monthly_price_cents,
        is_published=False  # Requires approval
    )
    db.add(db_agent)
    db.commit()
    db.refresh(db_agent)
    
    # Create initial version
    version = AgentVersion(
        agent_id=db_agent.id,
        version="1.0.0",
        config_json=agent.config_json,
        is_active=True
    )
    db.add(version)
    db.commit()
    
    return db_agent

@app.get("/agents", response_model=List[AgentResponse])
async def list_agents(
    category: Optional[str] = None,
    search: Optional[str] = None,
    db: Session = Depends(get_db)
):
    """List published agents with optional filters."""
    query = db.query(Agent).filter(Agent.is_published == True)
    
    if category:
        query = query.filter(Agent.category == category)
    
    if search:
        query = query.filter(Agent.name.ilike(f"%{search}%"))
    
    return query.all()

@app.get("/agents/{agent_id}", response_model=AgentResponse)
async def get_agent(agent_id: str, db: Session = Depends(get_db)):
    """Get agent details."""
    agent = db.query(Agent).filter(Agent.id == agent_id).first()
    if not agent:
        raise HTTPException(404, "Agent not found")
    return agent

@app.post("/agents/{agent_id}/install")
async def install_agent(
    agent_id: str,
    request: AgentInstallRequest,
    db: Session = Depends(get_db)
):
    """Install agent for tenant."""
    # Check quota
    has_quota = await self_learning_client.check_quota(request.tenant_id, "agents")
    if not has_quota:
        raise HTTPException(429, "Agent quota exceeded")
    
    # Check if already installed
    existing = db.query(AgentInstallation).filter(
        AgentInstallation.agent_id == agent_id,
        AgentInstallation.tenant_id == request.tenant_id,
        AgentInstallation.uninstalled_at.is_(None)
    ).first()
    
    if existing:
        raise HTTPException(409, "Agent already installed")
    
    # Get latest version
    latest_version = db.query(AgentVersion).filter(
        AgentVersion.agent_id == agent_id,
        AgentVersion.is_active == True
    ).order_by(AgentVersion.created_at.desc()).first()
    
    if not latest_version:
        raise HTTPException(404, "No active version found")
    
    # Create installation
    installation = AgentInstallation(
        agent_id=agent_id,
        tenant_id=request.tenant_id,
        version_id=latest_version.id
    )
    db.add(installation)
    db.commit()
    
    # Increment tenant agent count
    await self_learning_client.increment_agent_count(request.tenant_id)
    
    return {"status": "installed", "agent_id": agent_id}

@app.delete("/agents/{agent_id}/install")
async def uninstall_agent(
    agent_id: str,
    tenant_id: str,
    db: Session = Depends(get_db)
):
    """Uninstall agent for tenant."""
    installation = db.query(AgentInstallation).filter(
        AgentInstallation.agent_id == agent_id,
        AgentInstallation.tenant_id == tenant_id,
        AgentInstallation.uninstalled_at.is_(None)
    ).first()
    
    if not installation:
        raise HTTPException(404, "Installation not found")
    
    from datetime import datetime
    installation.uninstalled_at = datetime.now()
    db.commit()
    
    return {"status": "uninstalled", "agent_id": agent_id}

@app.get("/agents/{agent_id}/metrics")
async def get_agent_metrics(agent_id: str, days: int = 30):
    """Get usage and revenue metrics for agent."""
    from .analytics.marketplace_analytics import MarketplaceAnalytics
    
    analytics = MarketplaceAnalytics(ledger_dir="/var/lib/marketplace/ledgers")
    metrics = analytics.compute_agent_revenue(agent_id, days=days)
    
    return metrics

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### 6.3 agency-agents Conversion Script

**File:** `/home/lunark/projects/ai-native-agentic-org/agency-agents/marketplace-bridge/convert.py`

```python
#!/usr/bin/env python3
"""Convert agency-agents .md format to nanobot config JSON."""

import sys
import yaml
import json
from pathlib import Path
from typing import Dict, List

def infer_category(agent_path: Path) -> str:
    """Infer category from directory structure."""
    parts = agent_path.parts
    if "engineering" in parts:
        return "engineering"
    elif "design" in parts:
        return "design"
    elif "marketing" in parts:
        return "marketing"
    elif "product" in parts:
        return "product"
    elif "testing" in parts:
        return "testing"
    elif "support" in parts:
        return "support"
    elif "sales" in parts:
        return "sales"
    elif "operations" in parts:
        return "operations"
    elif "finance" in parts:
        return "finance"
    return "general"

def infer_tools(instructions: str) -> List[str]:
    """Infer required tools from agent instructions."""
    tools = []
    
    keywords = {
        "exec": ["execute", "run command", "shell", "terminal"],
        "read_file": ["read file", "open file", "file content"],
        "write_file": ["write file", "save file", "create file"],
        "web_fetch": ["fetch", "http", "api call", "web request"],
        "web_search": ["search", "google", "find information"],
    }
    
    instructions_lower = instructions.lower()
    for tool, keywords_list in keywords.items():
        if any(kw in instructions_lower for kw in keywords_list):
            tools.append(tool)
    
    # Default tools
    if not tools:
        tools = ["read_file", "write_file"]
    
    return tools

def convert_agent(agent_path: Path) -> Dict:
    """Convert agency-agents .md to nanobot config."""
    with open(agent_path) as f:
        content = f.read()
    
    # Split frontmatter and body
    parts = content.split('---', 2)
    if len(parts) < 3:
        raise ValueError(f"Invalid agent format: {agent_path}")
    
    frontmatter = yaml.safe_load(parts[1])
    instructions = parts[2].strip()
    
    category = infer_category(agent_path)
    tools = infer_tools(instructions)
    
    # Determine pricing based on category
    price_tiers = {
        "engineering": ("PRO", 999),
        "design": ("STARTER", 499),
        "marketing": ("STARTER", 499),
        "product": ("PRO", 999),
        "testing": ("STARTER", 299),
        "support": ("FREE", 0),
        "sales": ("PRO", 799),
        "operations": ("STARTER", 399),
        "finance": ("ENTERPRISE", 1999)
    }
    
    price_tier, monthly_price_cents = price_tiers.get(category, ("STARTER", 499))
    
    return {
        "name": frontmatter.get("name", agent_path.stem),
        "slug": agent_path.stem.lower().replace(" ", "-"),
        "description": frontmatter.get("description", ""),
        "category": category,
        "price_tier": price_tier,
        "monthly_price_cents": monthly_price_cents,
        "config_json": {
            "name": frontmatter.get("name", agent_path.stem),
            "model": "claude-sonnet-4-5",
            "temperature": 0.7,
            "max_iterations": 10,
            "system_prompt": instructions,
            "tools": tools,
            "metadata": {
                "category": category,
                "description": frontmatter.get("description", ""),
                "color": frontmatter.get("color", "#000000")
            }
        }
    }

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: convert.py <agent.md>")
        sys.exit(1)
    
    agent_path = Path(sys.argv[1])
    if not agent_path.exists():
        print(f"Error: {agent_path} not found")
        sys.exit(1)
    
    config = convert_agent(agent_path)
    print(json.dumps(config, indent=2))
```

### 6.4 nanobot Marketplace Runtime

**File:** `/home/lunark/projects/ai-native-agentic-org/nanobot/marketplace_runtime.py`

```python
#!/usr/bin/env python3
"""Marketplace-specific nanobot runtime with economic tracking."""

import asyncio
import json
import sys
from pathlib import Path
from argparse import ArgumentParser

from nanobot import AgentLoop, MessageBus
from economic_sdk import EconomicTracker

async def run_marketplace_agent(config: dict, tenant_id: str, user_message: str):
    """Execute agent with economic tracking."""
    workspace = Path(f"/tmp/marketplace/{tenant_id}/{config['name']}")
    workspace.mkdir(parents=True, exist_ok=True)
    
    ledger_path = Path(f"/var/lib/marketplace/ledgers/{tenant_id}.jsonl")
    ledger_path.parent.mkdir(parents=True, exist_ok=True)
    
    tracker = EconomicTracker(ledger_path=str(ledger_path))
    
    # Create agent loop
    bus = MessageBus()
    loop = AgentLoop(
        bus=bus,
        provider="anthropic",
        model=config["model"],
        workspace=str(workspace),
        max_iterations=config["max_iterations"],
        temperature=config["temperature"]
    )
    
    # Track start
    input_tokens = len(user_message.split()) * 1.3  # Rough estimate
    
    # Execute
    response = await loop.run(user_message)
    
    # Track completion
    output_tokens = len(response.split()) * 1.3
    cost = (input_tokens * 0.003 + output_tokens * 0.015) / 1000  # Claude Sonnet pricing
    
    tracker.track_llm_call(
        model=config["model"],
        input_tokens=int(input_tokens),
        output_tokens=int(output_tokens),
        cost=cost,
        metadata={"agent_name": config["name"]}
    )
    
    # Track revenue (prorated daily subscription)
    daily_revenue = config.get("monthly_price_cents", 0) / 30 / 100
    tracker.add_work_income(
        task_description=f"Agent {config['name']} execution",
        evaluation_score=0.8,
        amount=daily_revenue,
        metadata={"agent_name": config["name"]}
    )
    
    return response

def main():
    parser = ArgumentParser(description="Marketplace agent runtime")
    parser.add_argument("--config", required=True, help="Agent config JSON")
    parser.add_argument("--tenant-id", required=True, help="Tenant ID")
    parser.add_argument("--message", required=True, help="User message")
    args = parser.parse_args()
    
    config = json.loads(args.config)
    
    response = asyncio.run(
        run_marketplace_agent(config, args.tenant_id, args.message)
    )
    
    print(response)

if __name__ == "__main__":
    main()
```

---

## 7. 데이터 흐름 워크스루 (Data Flow Walkthrough)

### 7.1 Agent Installation Flow

```
1. User visits marketplace web UI (React app)
   ↓
2. Browses agents, clicks "Install Marketing Analyst"
   ↓
3. Frontend calls: POST /agents/{id}/install
   {
     "tenant_id": "tenant-123",
     "user_id": "user-456"
   }
   ↓
4. Agent Registry Service:
   a. Calls self-learning-agents: GET /tenants/tenant-123
   b. Checks quota: current_agents < max_agents
   c. Creates AgentInstallation record
   d. Calls self-learning-agents: POST /tenants/tenant-123/increment-agents
   ↓
5. Returns success response
   ↓
6. Frontend shows "✓ Agent installed" notification
```

### 7.2 Agent Invocation Flow

```
1. User sends Slack message: "@marketing-analyst analyze Q1 campaign"
   ↓
2. Slack webhook → openclaw gateway
   ↓
3. openclaw routes to AgentMarketplaceExtension
   ↓
4. Extension extracts agent mention: "marketing-analyst"
   ↓
5. Extension calls Agent Registry: GET /tenants/tenant-123/agents
   ↓
6. Registry checks AgentInstallation table
   ↓
7. If subscribed:
   a. Registry returns agent config (nanobot JSON)
   b. Extension spawns nanobot process
   c. nanobot executes agent logic
   d. economic-sdk tracks LLM call + revenue
   e. Response streams back to extension
   f. Extension posts to Slack thread
   ↓
8. If not subscribed:
   a. Extension replies: "Agent not installed. Visit marketplace."
```

### 7.3 Revenue Tracking Flow

```
1. Agent invocation completes (see 7.2)
   ↓
2. nanobot marketplace_runtime.py:
   a. Calculates LLM cost (input/output tokens × pricing)
   b. Calls economic_sdk.track_llm_call()
   c. Calculates daily revenue (monthly_price / 30)
   d. Calls economic_sdk.add_work_income()
   ↓
3. economic-sdk writes to JSONL ledger:
   /var/lib/marketplace/ledgers/tenant-123.jsonl
   ↓
4. Nightly batch job (cron):
   a. Runs MarketplaceAnalytics.compute_agent_revenue()
   b. Aggregates all tenant ledgers
   c. Updates agent_usage_logs table
   ↓
5. Monthly payout job:
   a. Runs MarketplaceAnalytics.compute_creator_payout()
   b. Generates Stripe payouts (70% to creators)
   c. Sends payout notification emails
```

### 7.4 Agent Update Flow

```
1. Creator publishes new version via CLI:
   $ marketplace-cli publish marketing-analyst --version 1.1.0
   ↓
2. CLI calls: POST /agents/{id}/versions
   {
     "version": "1.1.0",
     "config_json": {...},
     "changelog": "Added sentiment analysis"
   }
   ↓
3. Agent Registry Service:
   a. Creates new AgentVersion record
   b. Sets is_active=true, deactivates old version
   c. Notifies all subscribed tenants (email/webhook)
   ↓
4. Next invocation:
   a. openclaw fetches latest version
   b. Uses new config automatically
   ↓
5. Users see changelog in marketplace UI
```

---

## 8. MVP 범위 (MVP Scope)

### 8.1 In-Scope Features

**Phase 1 (Weeks 1-2):**
- Agent Registry Service (CRUD APIs)
- PostgreSQL schema + migrations
- openclaw AgentMarketplaceExtension (basic routing)
- agency-agents conversion script (75 agents)
- nanobot marketplace runtime (economic tracking)
- Subscription check (self-learning-agents integration)
- Basic web UI (React, list/search/install agents)

**Phase 2 (Weeks 3-4):**
- Stripe payment integration (checkout, webhooks)
- Creator dashboard (revenue, usage analytics)
- Agent versioning (publish updates)
- Rating and review system
- Usage quotas (requests per day)
- Email notifications (installation, updates)

**Phase 3 (Weeks 5-6):**
- Advanced search (filters, sorting)
- Agent categories and tags
- Creator payout automation (70/30 split)
- Admin approval workflow (publish agents)
- Audit logs (who installed what, when)
- Performance monitoring (response times, error rates)

### 8.2 Out-of-Scope (Future Roadmap)

- Custom agent builder UI (drag-and-drop)
- Agent marketplace mobile app
- Multi-language support (i18n)
- Agent collaboration (chaining multiple agents)
- White-label marketplace (enterprise customers)
- Agent testing sandbox (try before buy)
- Advanced analytics (A/B testing, cohort analysis)
- Agent certification program (verified creators)

### 8.3 Technical Constraints

| Constraint | Limit | Rationale |
|------------|-------|-----------|
| Max agents per tenant (FREE) | 3 | Encourage paid upgrades |
| Max agents per tenant (STARTER) | 25 | Balance cost and value |
| Max agents per tenant (PRO) | 100 | Support large teams |
| Agent execution timeout | 60s | Prevent runaway processes |
| Max agent config size | 100KB | Prevent abuse |
| Max concurrent executions per tenant | 10 | Resource management |
| Rate limit (API calls) | 100/min | DDoS protection |

---

## 9. 2주 스프린트 계획 (2-Week Sprint Plan)

### Week 1: Foundation

**Day 1-2: Infrastructure Setup**
- [ ] Create agent-registry-service repo
- [ ] Set up FastAPI project structure
- [ ] Configure PostgreSQL database (Docker Compose)
- [ ] Create database schema (agents, versions, installations)
- [ ] Write Alembic migrations
- [ ] Set up Redis for caching
- [ ] Configure CI/CD (GitHub Actions)

**Day 3-4: Core APIs**
- [ ] Implement POST /agents (create agent)
- [ ] Implement GET /agents (list/search)
- [ ] Implement GET /agents/{id} (details)
- [ ] Implement POST /agents/{id}/install
- [ ] Implement DELETE /agents/{id}/install
- [ ] Write unit tests (pytest, 80%+ coverage)
- [ ] Set up API documentation (Swagger/OpenAPI)

**Day 5: Integration Layer**
- [ ] Implement SelfLearningAgentsClient
- [ ] Implement MarketplaceEconomicTracker
- [ ] Write integration tests (mock external APIs)
- [ ] Set up logging (structured JSON logs)
- [ ] Configure error tracking (Sentry)

### Week 2: Glue Code & Deployment

**Day 6-7: openclaw Extension**
- [ ] Create agent-marketplace extension directory
- [ ] Implement AgentMarketplaceExtension class
- [ ] Implement AgentRegistryClient (TypeScript)
- [ ] Implement NanobotExecutor (subprocess spawning)
- [ ] Write unit tests (Vitest)
- [ ] Test with Slack channel (end-to-end)

**Day 8-9: agency-agents Bridge**
- [ ] Write convert.py script (md → nanobot config)
- [ ] Write import-agents.sh bulk import script
- [ ] Test conversion on 10 sample agents
- [ ] Run full import (75 agents)
- [ ] Verify agents in registry database
- [ ] Create agent catalog documentation

**Day 10: nanobot Runtime**
- [ ] Implement marketplace_runtime.py
- [ ] Integrate economic-sdk tracking
- [ ] Test agent execution (5 sample agents)
- [ ] Verify JSONL ledger writes
- [ ] Test timeout handling (60s limit)
- [ ] Test error handling (invalid config)

**Day 11-12: Web UI (MVP)**
- [ ] Create React app (Vite + TypeScript)
- [ ] Implement agent list page (search, filters)
- [ ] Implement agent detail page (description, pricing)
- [ ] Implement install button (API call)
- [ ] Implement tenant dashboard (installed agents)
- [ ] Add authentication (JWT from self-learning-agents)
- [ ] Deploy to staging environment

**Day 13: Integration Testing**
- [ ] End-to-end test: Install agent via UI
- [ ] End-to-end test: Invoke agent via Slack
- [ ] End-to-end test: Verify economic tracking
- [ ] End-to-end test: Uninstall agent
- [ ] Load testing (100 concurrent requests)
- [ ] Security audit (OWASP Top 10)

**Day 14: Documentation & Launch**
- [ ] Write API documentation (README.md)
- [ ] Write deployment guide (Docker Compose)
- [ ] Write creator onboarding guide
- [ ] Record demo video (5 minutes)
- [ ] Announce to beta testers (10 organizations)
- [ ] Monitor metrics (installs, invocations, errors)

### Sprint Deliverables

| Deliverable | Status | Owner |
|-------------|--------|-------|
| Agent Registry Service (API) | ✅ Ready | Backend Team |
| openclaw Extension | ✅ Ready | Frontend Team |
| agency-agents Conversion | ✅ Ready | DevOps Team |
| nanobot Runtime | ✅ Ready | Backend Team |
| Web UI (MVP) | ✅ Ready | Frontend Team |
| Documentation | ✅ Ready | Tech Writer |
| Deployment Scripts | ✅ Ready | DevOps Team |

---

## 10. 비즈니스 모델 및 수익 (Business Model & Revenue)

### 10.1 Pricing Tiers

| Tier | Price | Max Agents | Requests/Day | Target Audience |
|------|-------|------------|--------------|-----------------|
| FREE | $0/mo | 3 | 100 | Individuals, hobbyists |
| STARTER | $29/mo | 25 | 1,000 | Small teams (5-10 people) |
| PRO | $99/mo | 100 | 10,000 | Growing companies (10-50) |
| ENTERPRISE | $499/mo+ | Unlimited | Unlimited | Large orgs (50-200+) |

### 10.2 Revenue Streams

**1. Subscription Fees (Primary)**
- Tenants pay monthly for agent access
- 70% to agent creators, 30% to platform
- Projected: $50K MRR by month 6

**2. Usage-Based Pricing (Future)**
- Charge per agent invocation (e.g., $0.10/call)
- Useful for high-volume enterprise customers
- Projected: $10K MRR by month 12

**3. Premium Features (Future)**
- Priority support ($99/mo add-on)
- Custom agent development ($5K one-time)
- White-label marketplace ($10K/mo)
- Projected: $15K MRR by month 12

### 10.3 Revenue Projections (6 Months)

| Month | Tenants | Avg. Tier | MRR | Creator Payout | Platform Revenue |
|-------|---------|-----------|-----|----------------|------------------|
| 1 | 20 | STARTER | $580 | $406 | $174 |
| 2 | 50 | STARTER | $1,450 | $1,015 | $435 |
| 3 | 100 | STARTER | $2,900 | $2,030 | $870 |
| 4 | 150 | PRO | $14,850 | $10,395 | $4,455 |
| 5 | 180 | PRO | $17,820 | $12,474 | $5,346 |
| 6 | 200 | PRO | $19,800 | $13,860 | $5,940 |

**Total 6-Month Revenue:** $57,400  
**Creator Payouts:** $40,180 (70%)  
**Platform Revenue:** $17,220 (30%)

### 10.4 Unit Economics

**Customer Acquisition Cost (CAC):**
- Marketing spend: $5K/mo
- Sales team: $10K/mo
- Total: $15K/mo
- Customers acquired: 30/mo (months 4-6)
- CAC: $500/customer

**Lifetime Value (LTV):**
- Avg. subscription: $99/mo (PRO tier)
- Avg. retention: 18 months
- LTV: $1,782

**LTV:CAC Ratio:** 3.56:1 (healthy, target >3:1)

**Payback Period:** 5 months

### 10.5 Creator Economics

**Top Creator Scenario:**
- 10 agents published
- Avg. 50 installs per agent
- Avg. price: $9.99/mo per agent
- Monthly revenue: 10 × 50 × $9.99 = $4,995
- Creator payout (70%): $3,497/mo
- Annual income: $41,964

**Average Creator Scenario:**
- 3 agents published
- Avg. 20 installs per agent
- Avg. price: $4.99/mo per agent
- Monthly revenue: 3 × 20 × $4.99 = $299
- Creator payout (70%): $209/mo
- Annual income: $2,508

### 10.6 Cost Structure

| Cost Category | Monthly | Annual | Notes |
|---------------|---------|--------|-------|
| Infrastructure (AWS) | $2,000 | $24,000 | EC2, RDS, S3, CloudFront |
| LLM API (Anthropic) | $5,000 | $60,000 | Claude Sonnet usage |
| Payment processing (Stripe) | $500 | $6,000 | 2.9% + $0.30 per transaction |
| Engineering team | $30,000 | $360,000 | 3 engineers × $10K/mo |
| Marketing | $5,000 | $60,000 | Content, ads, events |
| Support | $3,000 | $36,000 | 1 support engineer |
| **Total** | **$45,500** | **$546,000** | |

**Break-Even Point:** $45,500 MRR (month 9-10 projected)

---

## 11. 위험 및 완화 (Risk & Mitigation)

### 11.1 Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| nanobot execution timeout | High | Medium | Implement 60s timeout, retry logic, fallback responses |
| Database performance (high load) | High | Medium | PostgreSQL read replicas, Redis caching, query optimization |
| openclaw extension crashes | High | Low | Comprehensive error handling, circuit breakers, health checks |
| Agent config injection attacks | Critical | Low | JSON schema validation, sandboxed execution, tool whitelisting |
| Economic tracking data loss | Medium | Low | JSONL append-only logs, daily backups, S3 archival |
| Stripe webhook failures | Medium | Medium | Retry queue (SQS), idempotency keys, manual reconciliation |

**Mitigation Strategy:**
- Implement comprehensive monitoring (Prometheus, Grafana)
- Set up alerting (PagerDuty) for critical failures
- Run weekly disaster recovery drills
- Maintain 99.9% uptime SLA (43 minutes downtime/month)

### 11.2 Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Low creator adoption | High | Medium | Offer $500 signing bonus for first 50 creators |
| High churn rate (>10%/mo) | High | Medium | Implement usage analytics, proactive support, feature requests |
| Competitor launches similar marketplace | Medium | High | Focus on ecosystem integration (openclaw, nanobot), creator community |
| Regulatory compliance (GDPR, CCPA) | High | Low | Implement data deletion APIs, privacy policy, DPA templates |
| Agent quality issues (spam, malicious) | High | Medium | Manual approval workflow, rating system, abuse reporting |
| Payment fraud | Medium | Low | Stripe Radar (fraud detection), manual review for high-value transactions |

**Mitigation Strategy:**
- Build strong creator community (Discord, monthly meetups)
- Implement Net Promoter Score (NPS) surveys
- Hire compliance consultant (GDPR, SOC 2)
- Establish agent review board (3 reviewers per agent)

### 11.3 Operational Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Key engineer leaves | High | Low | Document all systems, pair programming, knowledge sharing sessions |
| Infrastructure outage (AWS) | High | Low | Multi-region deployment (us-east-1, eu-west-1), automated failover |
| Security breach (data leak) | Critical | Low | Penetration testing (quarterly), bug bounty program, encryption at rest |
| Support ticket backlog | Medium | Medium | Hire 2nd support engineer at 500 tenants, self-service docs |
| Creator payout errors | Medium | Low | Automated testing, manual review for payouts >$1K, audit trail |

**Mitigation Strategy:**
- Maintain runbooks for all critical systems
- Run quarterly tabletop exercises (security, outage)
- Implement automated backups (daily, 30-day retention)
- Set up status page (status.marketplace.ai-native.org)

---

## 12. 성공 지표 (Success Metrics)

### 12.1 Product Metrics (Week 1-2)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Agents published | 75+ | Agent registry count |
| API uptime | 99.5%+ | Prometheus uptime checks |
| Avg. API response time | <200ms | p95 latency |
| Test coverage | 80%+ | pytest/vitest coverage reports |
| openclaw integration success | 100% | Manual testing checklist |

### 12.2 Business Metrics (Month 1-6)

| Metric | Month 1 | Month 3 | Month 6 | Measurement |
|--------|---------|---------|---------|-------------|
| Active tenants | 20 | 100 | 200 | Stripe active subscriptions |
| MRR | $580 | $2,900 | $19,800 | Stripe revenue reports |
| Agent installs | 50 | 500 | 2,000 | AgentInstallation table count |
| Avg. agents per tenant | 2.5 | 5 | 10 | installs / tenants |
| Creator count | 10 | 30 | 75 | Unique creator_id count |
| Avg. creator earnings | $40 | $100 | $500 | Payout reports |

### 12.3 Engagement Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Daily active agents | 50%+ | Agents with invocations in last 24h |
| Avg. invocations per agent | 10+/day | agent_usage_logs aggregation |
| User satisfaction (NPS) | 40+ | Quarterly surveys |
| Agent rating | 4.0+/5.0 | agent_reviews avg_rating |
| Support ticket resolution time | <24h | Zendesk metrics |
| Creator retention (6mo) | 80%+ | Creators with active agents |

### 12.4 Technical Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Agent execution success rate | 95%+ | Successful invocations / total |
| Avg. agent response time | <5s | p95 execution time |
| Economic tracking accuracy | 99%+ | Manual audit vs. JSONL ledger |
| Database query performance | <50ms | p95 query time |
| Error rate | <1% | Error logs / total requests |
| Security incidents | 0 | Incident reports |

### 12.5 Dashboard (Real-Time Monitoring)

**Grafana Dashboard Panels:**
1. MRR trend (line chart)
2. Active tenants (gauge)
3. Agent installs (bar chart by category)
4. Top 10 agents by revenue (table)
5. API latency (heatmap)
6. Error rate (line chart)
7. Creator payouts (stacked area chart)
8. Agent execution success rate (gauge)

**Alerts:**
- MRR drops >10% week-over-week → Slack #revenue-alerts
- API error rate >5% → PagerDuty (critical)
- Database CPU >80% → PagerDuty (warning)
- Agent execution timeout >10% → Slack #eng-alerts
- Stripe webhook failures >5 → Slack #payments-alerts

---

## 13. 배포 및 운영 (Deployment & Operations)

### 13.1 Infrastructure

**Architecture:**
- Kubernetes cluster (EKS on AWS)
- PostgreSQL (RDS, Multi-AZ)
- Redis (ElastiCache)
- S3 (agent configs, ledgers)
- CloudFront (CDN for web UI)

**Environments:**
- Development (local Docker Compose)
- Staging (single-node k8s)
- Production (multi-region k8s)

### 13.2 Deployment Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy Agent Registry Service

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest --cov=src --cov-report=xml
      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: |
          docker build -t agent-registry:${{ github.sha }} .
      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
          docker tag agent-registry:${{ github.sha }} $ECR_REGISTRY/agent-registry:${{ github.sha }}
          docker push $ECR_REGISTRY/agent-registry:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to k8s
        run: |
          kubectl set image deployment/agent-registry agent-registry=$ECR_REGISTRY/agent-registry:${{ github.sha }}
          kubectl rollout status deployment/agent-registry
```

### 13.3 Monitoring & Alerting

**Prometheus Metrics:**
```python
# agent-registry-service/metrics.py

from prometheus_client import Counter, Histogram, Gauge

agent_installs_total = Counter(
    'agent_installs_total',
    'Total agent installations',
    ['agent_id', 'tenant_id']
)

agent_invocations_total = Counter(
    'agent_invocations_total',
    'Total agent invocations',
    ['agent_id', 'tenant_id', 'status']
)

agent_execution_duration = Histogram(
    'agent_execution_duration_seconds',
    'Agent execution duration',
    ['agent_id']
)

active_subscriptions = Gauge(
    'active_subscriptions',
    'Number of active subscriptions'
)
```

### 13.4 Backup & Recovery

**Daily Backups:**
- PostgreSQL: Automated RDS snapshots (7-day retention)
- JSONL ledgers: S3 sync (30-day retention)
- Redis: AOF persistence (append-only file)

**Recovery Procedures:**
1. Database corruption: Restore from latest RDS snapshot (<15 min)
2. Ledger data loss: Restore from S3 backup (<5 min)
3. Full region outage: Failover to secondary region (<30 min)

---

## 14. 다음 단계 (Next Steps)

### 14.1 Post-MVP Features (Months 2-3)

1. **Agent Testing Sandbox**
   - Try agents before subscribing
   - 10 free invocations per agent
   - Isolated test environment

2. **Advanced Analytics Dashboard**
   - Creator revenue trends
   - Agent performance benchmarks
   - User engagement heatmaps

3. **Agent Collaboration**
   - Chain multiple agents (workflows)
   - Agent-to-agent communication
   - Shared context/memory

4. **Mobile App**
   - iOS/Android native apps
   - Push notifications for agent responses
   - Offline mode (cached responses)

### 14.2 Enterprise Features (Months 4-6)

1. **White-Label Marketplace**
   - Custom branding
   - Private agent catalogs
   - SSO integration (SAML, OIDC)

2. **Advanced RBAC**
   - Custom roles (beyond VIEWER/OPERATOR/ADMIN)
   - Fine-grained permissions (per-agent access)
   - Audit logs (who did what, when)

3. **SLA Guarantees**
   - 99.95% uptime commitment
   - Priority support (1-hour response time)
   - Dedicated account manager

4. **Compliance Certifications**
   - SOC 2 Type II
   - ISO 27001
   - HIPAA (for healthcare agents)

### 14.3 Community Building

1. **Creator Program**
   - Monthly creator meetups (virtual)
   - Agent development workshops
   - Revenue milestone rewards ($1K, $10K, $100K)

2. **Agent Certification**
   - Verified creator badge
   - Quality standards (test coverage, docs)
   - Featured placement in marketplace

3. **Open Source Contributions**
   - Publish agent templates (GitHub)
   - Contribute to nanobot/openclaw
   - Sponsor ecosystem projects

---

## 15. 부록 (Appendix)

### 15.1 File Structure

```
agent-registry-service/
├── src/
│   ├── main.py                    # FastAPI app
│   ├── database.py                # SQLAlchemy setup
│   ├── models.py                  # ORM models
│   ├── schemas.py                 # Pydantic schemas
│   ├── api/
│   │   └── routes/
│   │       ├── agents.py          # Agent CRUD
│   │       ├── installations.py   # Install/uninstall
│   │       └── analytics.py       # Metrics
│   ├── integrations/
│   │   ├── self_learning_client.py
│   │   └── stripe_client.py
│   └── analytics/
│       ├── economic_integration.py
│       └── marketplace_analytics.py
├── tests/
│   ├── test_agents.py
│   ├── test_installations.py
│   └── test_analytics.py
├── alembic/                       # Database migrations
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md

openclaw/extensions/agent-marketplace/
├── src/
│   ├── index.ts                   # Main extension
│   ├── client.ts                  # Registry API client
│   ├── executor.ts                # Nanobot executor
│   └── types.ts                   # TypeScript types
├── tests/
│   └── index.test.ts
├── package.json
└── README.md

agency-agents/marketplace-bridge/
├── convert.py                     # MD → nanobot config
├── import-agents.sh               # Bulk import script
└── README.md

nanobot/
├── marketplace_runtime.py         # Marketplace-specific runtime
└── README.md
```

### 15.2 Environment Variables

```bash
# agent-registry-service/.env

DATABASE_URL=postgresql://user:pass@localhost:5432/marketplace
REDIS_URL=redis://localhost:6379/0
SELF_LEARNING_API_URL=https://api.self-learning.ai-native.org
SELF_LEARNING_API_KEY=sk_live_...
STRIPE_API_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
ANTHROPIC_API_KEY=sk-ant-...
SENTRY_DSN=https://...@sentry.io/...
LOG_LEVEL=INFO
```

### 15.3 API Examples

**Install Agent:**
```bash
curl -X POST https://marketplace.ai-native.org/api/agents/agent-123/install \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tenant_id": "tenant-456"
  }'
```

**List Installed Agents:**
```bash
curl https://marketplace.ai-native.org/api/tenants/tenant-456/agents \
  -H "Authorization: Bearer $TOKEN"
```

**Get Agent Metrics:**
```bash
curl https://marketplace.ai-native.org/api/agents/agent-123/metrics?days=30 \
  -H "Authorization: Bearer $TOKEN"
```

### 15.4 Database Queries

**Top 10 Agents by Revenue:**
```sql
SELECT 
  a.name,
  a.category,
  COUNT(ai.id) as install_count,
  SUM(aul.total_cost_cents) / 100.0 as total_revenue
FROM agents a
JOIN agent_installations ai ON a.id = ai.agent_id
JOIN agent_usage_logs aul ON a.id = aul.agent_id
WHERE ai.uninstalled_at IS NULL
GROUP BY a.id
ORDER BY total_revenue DESC
LIMIT 10;
```

**Creator Payout Report:**
```sql
SELECT 
  a.creator_id,
  COUNT(DISTINCT a.id) as agent_count,
  SUM(aul.total_cost_cents) / 100.0 * 0.70 as creator_payout
FROM agents a
JOIN agent_usage_logs aul ON a.id = aul.agent_id
WHERE aul.date >= DATE_TRUNC('month', CURRENT_DATE)
GROUP BY a.creator_id
ORDER BY creator_payout DESC;
```

### 15.5 References

**Internal Documentation:**
- `/home/lunark/projects/ai-native-agentic-org/openclaw/README.md`
- `/home/lunark/projects/ai-native-agentic-org/agency-agents/README.md`
- `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/README.md`
- `/home/lunark/projects/ai-native-agentic-org/nanobot/README.md`
- `/home/lunark/projects/ai-native-agentic-org/ai-native-agentic-economic-sdk/README.md`

**External Resources:**
- FastAPI Documentation: https://fastapi.tiangolo.com
- Stripe API Reference: https://stripe.com/docs/api
- Anthropic Claude API: https://docs.anthropic.com
- PostgreSQL Documentation: https://www.postgresql.org/docs
- Kubernetes Documentation: https://kubernetes.io/docs

---

## 16. 결론 (Conclusion)

The AI Agent Marketplace represents a strategic integration of four mature ecosystem projects (openclaw, agency-agents, ai-native-self-learning-agents, nanobot) into a unified platform that creates value for three stakeholder groups:

1. **Agent Creators**: Monetize expertise, reach 10K+ customers, earn 70% revenue share
2. **Organizations**: Access 75+ production-ready agents, deploy in <5 minutes, pay-per-agent pricing
3. **End Users**: Unified experience across 22+ channels, consistent quality, transparent pricing

**Key Success Factors:**
- Leverage existing ecosystem (no greenfield development)
- Focus on creator economics (70/30 split attracts talent)
- Tight integration with openclaw (seamless user experience)
- Economic tracking from day 1 (data-driven optimization)
- 2-week MVP timeline (rapid validation)

**Expected Outcomes (6 Months):**
- 200+ paying tenants
- $19.8K MRR
- 150+ agents published
- $35K+ creator earnings
- 99.9% uptime

This specification provides a complete, implementation-ready blueprint for building the marketplace. All glue code, integration points, and data flows are fully specified with exact file paths and code snippets. The 2-week sprint plan ensures rapid delivery while maintaining quality standards.

**Next Action:** Kick off Week 1 infrastructure setup (Day 1-2 tasks).

---

**Document Version:** 1.0.0  
**Last Updated:** 2026-03-10  
**Status:** Ready for Implementation  
**Approvers:** Engineering Lead, Product Manager, CTO
