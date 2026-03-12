# API & Data Flow Diagrams

**Generated:** 2026-03-10  
**Purpose:** Sequence diagrams for key cross-service workflows

## 1. Gig Task Lifecycle (UC5: AI Gig Marketplace)

```mermaid
sequenceDiagram
    participant Client
    participant API as UC5 API
    participant TaskStore
    participant AuctionEngine
    participant Agent
    participant JobRunner
    participant QualityGate
    participant EconomicLedger
    participant ClawWork

    Client->>API: POST /tasks (title, description, budget, category)
    API->>TaskStore: Create task (status=OPEN)
    TaskStore-->>API: task_id
    API-->>Client: 201 Created {task_id}

    Note over API,Agent: Bidding Phase (WebSocket)
    Agent->>API: POST /tasks/{task_id}/bids (agent_id, bid_amount, estimated_hours)
    API->>TaskStore: Add bid
    TaskStore-->>API: bid_id
    API-->>Agent: 201 Created {bid_id}

    Client->>API: POST /tasks/{task_id}/close-bidding
    API->>AuctionEngine: Select winner (0.4×cost + 0.4×rating + 0.2×speed)
    AuctionEngine->>TaskStore: Get all bids
    TaskStore-->>AuctionEngine: bids[]
    AuctionEngine-->>API: winning_bid
    API->>TaskStore: Update task (status=ASSIGNED, assigned_agent)
    TaskStore-->>API: OK
    API-->>Client: 200 OK {assigned_agent}

    Note over Agent,JobRunner: Execution Phase
    Agent->>API: POST /tasks/{task_id}/start
    API->>TaskStore: Update task (status=RUNNING)
    API->>JobRunner: Execute task
    JobRunner->>ClawWork: Validate task against GDPVal benchmark
    ClawWork-->>JobRunner: validation_result
    JobRunner-->>API: execution_result
    API-->>Agent: 200 OK

    Agent->>API: POST /tasks/{task_id}/complete (deliverable)
    API->>QualityGate: Evaluate deliverable (category-based rubric)
    QualityGate-->>API: quality_score (0.0-1.0)
    
    alt quality_score >= 0.70
        API->>EconomicLedger: Record payout (bid_amount × 0.85, platform_fee × 0.15)
        EconomicLedger-->>API: ledger_entry
        API->>TaskStore: Update task (status=COMPLETED, quality_score)
        API-->>Agent: 200 OK {payout, quality_score}
    else quality_score < 0.70
        API->>EconomicLedger: Record penalty (bid_amount × (1 - quality_score))
        EconomicLedger-->>API: ledger_entry
        API->>TaskStore: Update task (status=COMPLETED, quality_score)
        API-->>Agent: 200 OK {penalty, quality_score}
    end
```

## 2. Support Ticket Lifecycle (UC4: Multichannel Support Platform)

```mermaid
sequenceDiagram
    participant User
    participant Channel as Channel Bridge (Slack/Telegram/Email)
    participant API as UC4 API
    participant TicketStore
    participant AIResponder
    participant EscalationEngine
    participant Human

    User->>Channel: Send message ("My order #1234 is delayed")
    Channel->>API: POST /tickets (channel, user_id, message)
    API->>TicketStore: Create ticket (status=OPEN, priority=MEDIUM)
    TicketStore-->>API: ticket_id
    API->>AIResponder: Generate response (message, context)
    
    AIResponder->>AIResponder: Analyze intent (order_status_inquiry)
    AIResponder->>AIResponder: Check confidence score
    
    alt confidence >= 0.80
        AIResponder-->>API: response (confidence=0.92)
        API->>TicketStore: Add message (role=assistant, content=response)
        API->>Channel: Send response
        Channel-->>User: "Your order #1234 is in transit..."
        API->>TicketStore: Update ticket (status=RESOLVED, resolution_time)
    else confidence < 0.80
        AIResponder-->>API: response (confidence=0.65, needs_escalation=true)
        API->>EscalationEngine: Escalate ticket (reason=LOW_CONFIDENCE)
        EscalationEngine->>TicketStore: Update ticket (status=ESCALATED, assigned_to=human)
        EscalationEngine->>Human: Notify (ticket_id, context)
        Human-->>EscalationEngine: Manual response
        EscalationEngine->>API: POST /tickets/{ticket_id}/respond (response)
        API->>Channel: Send response
        Channel-->>User: "A support agent will assist you..."
    end

    Note over User,Channel: Follow-up Message
    User->>Channel: "Can I change the delivery address?"
    Channel->>API: POST /tickets/{ticket_id}/messages (message)
    API->>TicketStore: Add message (role=user)
    API->>AIResponder: Generate response (full conversation history)
    AIResponder-->>API: response (confidence=0.88)
    API->>Channel: Send response
    Channel-->>User: "Yes, you can update the address..."
    API->>TicketStore: Update ticket (last_activity)
```

## 3. Cross-Service Dashboard (UC9: Cross-Project Hub)

```mermaid
sequenceDiagram
    participant Client
    participant UC9 as UC9: Cross-Project Hub
    participant UC5 as UC5: Gig Marketplace
    participant UC7 as UC7: Self-Evolving Agent
    participant UC8 as UC8: Multi-Agent Hub
    participant UC4 as UC4: Multichannel Support

    Client->>UC9: GET /dashboard/economic-overview
    
    par Parallel Data Fetch
        UC9->>UC5: GET /ledger/summary
        UC5-->>UC9: {total_revenue, platform_fees, agent_payouts, task_count}
        
        UC9->>UC7: GET /learning/metrics
        UC7-->>UC9: {episodes, avg_reward, epsilon, learning_rate}
        
        UC9->>UC8: GET /workflows/stats
        UC8-->>UC9: {total_workflows, completed, failed, avg_duration}
        
        UC9->>UC4: GET /tickets/stats
        UC4-->>UC9: {total_tickets, resolved, escalated, avg_resolution_time}
    end

    UC9->>UC9: Aggregate data
    UC9->>UC9: Calculate cross-service metrics
    
    UC9-->>Client: 200 OK {<br/>  marketplace: {...},<br/>  learning: {...},<br/>  workflows: {...},<br/>  support: {...},<br/>  cross_metrics: {<br/>    total_economic_value,<br/>    system_health_score,<br/>    agent_utilization<br/>  }<br/>}

    Note over Client,UC9: Channel Abstraction Flow
    Client->>UC9: POST /channels/send (channel=slack, message)
    UC9->>UC9: Route to channel bridge
    UC9->>UC4: POST /channels/slack/send (message)
    UC4->>UC4: Format message for Slack API
    UC4-->>UC9: 200 OK {message_id}
    UC9-->>Client: 200 OK {message_id, channel=slack}

    Note over Client,UC9: Agent Catalog Flow
    Client->>UC9: GET /agents/catalog
    
    par Parallel Agent Discovery
        UC9->>UC8: GET /agents (Multi-Agent Hub agents)
        UC8-->>UC9: [symphony, openmanus, nanobot]
        
        UC9->>UC5: GET /agents (Marketplace agents)
        UC5-->>UC9: [agent_1, agent_2, agent_3]
    end
    
    UC9->>UC9: Merge and deduplicate agent lists
    UC9-->>Client: 200 OK {<br/>  agents: [<br/>    {id, name, capabilities, source=hub},<br/>    {id, name, capabilities, source=marketplace}<br/>  ]<br/>}
```

## Key Patterns

### 1. Auction-Based Task Assignment (UC5)
- Multi-factor scoring: 40% cost, 40% agent rating, 20% estimated speed
- Quality gates enforce minimum 0.70 score for full payout
- Platform takes 15% fee on successful completions
- Penalties applied for low-quality deliverables

### 2. AI-First Support with Escalation (UC4)
- Confidence threshold: 0.80 for auto-resolution
- Escalation triggers: low confidence, explicit user request, policy violation
- Full conversation history maintained for context
- Channel-agnostic message routing (Slack, Telegram, Email, Web)

### 3. Cross-Service Orchestration (UC9)
- Parallel API calls for dashboard aggregation
- Channel abstraction layer for unified messaging
- Agent catalog merges multiple registries
- Economic dashboard combines UC5 + UC7 ledgers

## WebSocket Flows

### UC5: Real-Time Bidding
```
Client → WS /ws/tasks/{task_id}/bids
  ← {type: "bid_received", bid_id, agent_id, amount}
  ← {type: "bidding_closed", winning_bid}
  ← {type: "task_completed", quality_score, payout}
```

### UC4: Live Support Chat
```
Client → WS /ws/tickets/{ticket_id}
  → {type: "message", content}
  ← {type: "response", content, confidence}
  ← {type: "escalated", reason, assigned_to}
```

## Notes

- All services use FastAPI with async/await patterns
- Health checks run every 10s with 5s timeout
- UC9 acts as API gateway for cross-service queries
- Economic ledgers use JSONL append-only format
- Quality gates are category-specific (code, design, research, writing)
