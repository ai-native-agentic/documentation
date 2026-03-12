# UC2: AI 에이전트 마켓플레이스 (AI Agent Marketplace) Specification

**Version:** 1.0.0
**Status:** Implementation-Ready
**Date:** 2026-03-11
**Owner:** Antigravity (Google DeepMind)

---

## 1. 개요 및 비전 (Overview & Vision)

### 1.1 개요
AI 에이전트 마켓플레이스(UC2)는 개발자가 고성능 AI 에이전트를 게시하고, 기업(테넌트)이 이를 구독하여 자신의 비즈니스 워크플로우에 즉시 통합할 수 있는 개방형 플랫폼입니다. `agency-agents`의 75개 에이전트 카탈로그를 시작으로, `openclaw`의 멀티채널 게이트웨이와 `nanobot`의 경량 런타임을 결합하여 에이전트의 유통과 실행을 자동화합니다.

### 1.2 비전
- **에이전트 경제 활성화**: 에이전트 제작자에게 수익 창출 기회를 제공하고, 사용자에게는 검증된 에이전트를 제공합니다.
- **심리스한 통합**: `openclaw`를 통해 Slack, Telegram, Discord 등 다양한 채널에서 에이전트와 즉시 대화할 수 있습니다.
- **투명한 경제 지표**: `economic-sdk`를 통해 에이전트의 ROI, 비용, 수익을 실시간으로 추적합니다.

---

## 2. MVP 아키텍처 (MVP Architecture)

```text
+-----------------------------------------------------------------------+
|                          User / Client (Slack, Telegram, Web)         |
+-----------------------------------------------------------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
| [openclaw] Multi-channel Gateway                                      |
| - Marketplace Extension (Subscription Check, Routing)                 |
+-----------------------------------------------------------------------+
                                   |
                                   v
+-----------------------------------------------------------------------+
| [Agent Registry Service] (FastAPI)                                    |
| - Agent Catalog Management                                            |
| - Subscription & RBAC (via self-learning-agents)                      |
| - Deployment Bridge (Agency-Agents -> Nanobot)                        |
+-----------------------------------------------------------------------+
          |                        |                        |
          v                        v                        v
+-----------------------+  +-----------------------+  +-----------------------+
| [nanobot] Runtime     |  | [self-learning-agents]|  | [economic-sdk]        |
| - AgentLoop Execution |  | - Tenant/Billing      |  | - Revenue Tracking    |
| - Tool Execution      |  | - RBAC/Permissions    |  | - ROI Analytics       |
+-----------------------+  +-----------------------+  +-----------------------+
```

---

## 3. 컴포넌트 역할 (Component Responsibilities)

| 컴포넌트 | 역할 | 주요 파일 경로 |
|:---|:---|:---|
| **Agent Registry** | 에이전트 등록, 검색, 구독 관리 | `/api/v2/routes/agents_marketplace.py` |
| **openclaw Ext** | 채널 메시지 라우팅 및 구독 확인 | `/extensions/marketplace/src/channel.ts` |
| **Deployment Bridge** | MD 포맷 에이전트를 Nanobot 설정으로 변환 | `/agents/bridge/nanobot_bridge.py` |
| **nanobot** | 구매한 에이전트의 실제 실행 엔진 | `/nanobot/nanobot/agent/loop.py` |
| **Economic SDK** | 에이전트별 수익 및 비용 추적 | `/src/economic_sdk/tracker.py` |
| **Self-Learning** | 테넌트 관리 및 결제 시스템 제공 | `/orchestration/tenancy/tenant_manager.py` |

---

## 4. 마켓플레이스 데이터 모델 (Marketplace Data Model)

### 4.1 Agent Schema
```python
class Agent(BaseModel):
    id: str                  # Unique ID (e.g., "eng-senior-dev")
    name: str                # 에이전트 이름
    description: str         # 상세 설명
    category: str            # engineering, design, marketing 등
    price_tier: str          # FREE, STARTER, PRO
    version: str             # "1.0.0"
    author_id: str           # 제작자 테넌트 ID
    nanobot_config: dict     # Nanobot 실행 설정
    rating: float = 0.0      # 사용자 평점
```

### 4.2 Subscription Schema
```python
class Subscription(BaseModel):
    tenant_id: str           # 구독자 테넌트 ID
    agent_id: str            # 구독한 에이전트 ID
    status: str              # ACTIVE, EXPIRED, SUSPENDED
    started_at: datetime
    expires_at: datetime
```

---

## 5. 통합 포인트 (Integration Points)

### 5.1 openclaw -> Agent Registry
- **Endpoint**: `GET /marketplace/agents/{id}/validate`
- **Purpose**: 메시지 수신 시 해당 테넌트가 에이전트를 구독 중인지 확인.

### 5.2 Agent Registry -> self-learning-agents
- **Endpoint**: `POST /tenants/{id}/billing/subscribe`
- **Purpose**: 에이전트 구매 시 테넌트의 구독 정보 업데이트 및 결제 처리.

### 5.3 Agent Registry -> nanobot
- **Function**: `deploy_to_nanobot(agent_id, tenant_id)`
- **Purpose**: 에이전트 설정을 Nanobot `AgentLoop` 인스턴스로 로드.

---

## 6. 글루 코드 사양 (Glue Code Specification)

### A. Agent Registry Service (Python FastAPI)
파일 위치: `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/api/v2/routes/agents_marketplace.py`

```python
from fastapi import APIRouter, Depends, HTTPException
from orchestration.tenancy.tenant_manager import get_tenant_manager
from economic_sdk.tracker import EconomicTracker

router = APIRouter(prefix="/marketplace")

@router.post("/agents/{agent_id}/install")
async def install_agent(agent_id: str, tenant_id: str):
    tm = get_tenant_manager()
    tenant = tm.get_tenant(tenant_id)
    if not tenant:
        raise HTTPException(status_code=404, detail="Tenant not found")
    
    # 구독 로직 (self-learning-agents billing 연동)
    # ... 
    
    # Economic SDK 초기화 (수익 추적 시작)
    tracker = EconomicTracker(signature=f"agent-{agent_id}-tenant-{tenant_id}")
    tracker.initialize()
    
    return {"status": "installed", "agent_id": agent_id}

@router.get("/agents/{agent_id}/validate")
async def validate_subscription(agent_id: str, tenant_id: str):
    # 테넌트의 구독 상태 확인
    # ...
    return {"valid": True}
```

### B. openclaw AgentMarketplace Extension (TypeScript)
파일 위치: `/home/lunark/projects/ai-native-agentic-org/openclaw/extensions/marketplace/src/channel.ts`

```typescript
import { ChannelPlugin } from "../../../src/channels/plugins/types.plugin.js";

export const MarketplaceExtension: ChannelPlugin = {
  id: "marketplace",
  meta: {
    id: "marketplace",
    label: "Agent Marketplace",
    selectionLabel: "Marketplace",
    docsPath: "extensions/marketplace",
    blurb: "Route messages to purchased AI agents"
  },
  capabilities: {
    chatTypes: ["direct", "group"],
    reply: true
  },
  messaging: {
    async onMessage(msg, ctx) {
      const tenantId = ctx.cfg.get("tenant.id");
      const agentId = extractAgentId(msg.text); // 메시지에서 에이전트 ID 추출
      
      // 구독 확인 API 호출
      const isValid = await fetch(`http://localhost:8000/api/v2/marketplace/agents/${agentId}/validate?tenant_id=${tenantId}`);
      
      if (isValid) {
        // Nanobot 런타임으로 메시지 전달
        return routeToNanobot(agentId, msg.text);
      }
    }
  }
};
```

### C. agency-agents -> nanobot Deployment Bridge (Python)
파일 위치: `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/agents/bridge/nanobot_bridge.py`

```python
import yaml
from pathlib import Path
from nanobot.nanobot.agent.loop import AgentLoop

def convert_agency_to_nanobot(md_path: str) -> dict:
    """agency-agents MD 파일을 nanobot 설정으로 변환"""
    content = Path(md_path).read_text()
    # YAML Frontmatter 파싱
    _, frontmatter, instructions = content.split("---", 2)
    meta = yaml.safe_load(frontmatter)
    
    return {
        "name": meta["name"],
        "description": meta["description"],
        "system_prompt": instructions.strip(),
        "tools": map_tools(meta.get("tools", [])),
        "model": "gpt-4o" # 기본 모델
    }

def map_tools(agency_tools: list) -> list:
    # agency-agents 도구명을 nanobot 도구명으로 매핑
    mapping = {"shell": "ExecTool", "files": "ReadFile", "web": "WebSearch"}
    return [mapping[t] for t in agency_tools if t in mapping]
```

---

## 7. 데이터 흐름 (Data Flow Walkthrough)

1.  **에이전트 게시**: 개발자가 `agency-agents` 포맷의 MD 파일을 `Agent Registry`에 업로드. `Deployment Bridge`가 이를 `nanobot` 설정으로 변환하여 DB에 저장.
2.  **에이전트 구매**: 테넌트가 마켓플레이스 UI에서 에이전트 선택. `self-learning-agents` 결제 시스템을 통해 구독 생성.
3.  **메시지 수신**: 사용자가 Slack(`openclaw`)을 통해 에이전트에게 메시지 전송.
4.  **구독 검증**: `openclaw Marketplace Extension`이 `Agent Registry`에 구독 유효성 확인.
5.  **에이전트 실행**: 유효한 경우, `nanobot` 런타임이 해당 에이전트 설정을 로드하여 응답 생성.
6.  **경제 지표 기록**: 응답 완료 후 `economic-sdk`가 토큰 사용량(비용)과 작업 가치(수익)를 기록.

---

## 8. MVP 범위 (MVP Scope)

- **카탈로그**: `agency-agents`의 Engineering 및 Design 부문 에이전트 20종 우선 탑재.
- **채널**: Slack 및 Telegram 지원.
- **결제**: Stripe Test Mode를 통한 월간 구독 처리.
- **런타임**: `nanobot` 기본 도구(Exec, Read, Write, Search) 지원.

---

## 9. 2주 스프린트 계획 (2-Week Sprint Plan)

### Week 1: 인프라 및 브릿지 구축
- **Day 1-2**: `Agent Registry` 기본 API 구현 (등록, 검색).
- **Day 3-4**: `Agency-to-Nanobot Bridge` 개발 및 20종 에이전트 변환 테스트.
- **Day 5**: `self-learning-agents` 테넌트 시스템과 마켓플레이스 DB 연동.

### Week 2: 통합 및 QA
- **Day 6-7**: `openclaw Marketplace Extension` 개발 및 라우팅 로직 구현.
- **Day 8-9**: `economic-sdk` 연동을 통한 수익/비용 대시보드 데이터 생성.
- **Day 10-11**: End-to-End 테스트 (Slack -> openclaw -> nanobot -> economic-sdk).
- **Day 12**: 문서화 및 배포 준비.

---

## 10. 비즈니스 모델 및 수익 (Business Model & Revenue)

### 10.1 요금제 (Pricing Tiers)
- **Free**: 에이전트 3개 무료 사용, 월 10만 토큰 제한.
- **Starter ($29/mo)**: 에이전트 10개 사용, 월 100만 토큰, 이메일 지원.
- **Pro ($99/mo)**: 에이전트 25개 사용, 월 1,000만 토큰, 우선 지원.
- **Enterprise ($499/mo+)**: 무제한 에이전트, 전용 인프라, SLA 보장.

### 10.2 수익 배분 (Revenue Split)
- **에이전트 제작자**: 70%
- **플랫폼 (Antigravity)**: 30%

### 10.3 수익 추정 (Revenue Projection - Year 1)
- 목표 테넌트: 500개 (평균 ARPU $50)
- 월 매출: $25,000
- 연 매출: $300,000

---

## 11. 리스크 및 완화 방안 (Risk & Mitigation)

| 리스크 | 영향도 | 완화 방안 |
|:---|:---|:---|
| **에이전트 품질 저하** | 높음 | `economic-sdk`의 evaluation score가 0.6 미만인 에이전트는 노출 순위 강등. |
| **보안 사고 (RCE)** | 매우 높음 | `nanobot` 실행 시 Docker 컨테이너 격리 및 `ExecTool` 권한 제한. |
| **API 비용 폭증** | 중간 | 테넌트별/에이전트별 Quota 시스템(`self-learning-agents` quota manager) 적용. |

---

## 12. 성공 지표 (Success Metrics)

1.  **에이전트 활성 구독 수**: 월간 1,000건 이상.
2.  **평균 에이전트 ROI**: 1.5 이상 (수익이 비용의 1.5배).
3.  **사용자 만족도**: 에이전트 평균 평점 4.2/5.0 이상.
4.  **시스템 가동률**: 99.9% 이상.

---
**End of Document**
