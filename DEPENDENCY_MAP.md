# AI-NATIVE AGENTIC ORG — DEPENDENCY MAP

**Generated:** 2026-03-10  
**Scope:** Cross-project import analysis across 14 projects

## OVERVIEW

**Key Insight:** Only 2 hard dependencies exist across 14 projects. 12 projects are fully independent.

- **nanobot** and **economic-sdk** are the only dependency providers
- **ClawWork** has hard imports from both
- **ai-native-self-learning-agents** has soft (lazy) imports from both
- All other 10 projects: zero cross-project imports

This architecture enables parallel development, independent deployment, and minimal coupling risk.

## DEPENDENCY GRAPH

```
TIER 1: Personal Assistants
┌─────────────────────────────────────────────────────────────┐
│ openclaw    nanoclaw    seedbot                             │
│ (isolated)  (isolated)  (isolated)                          │
└─────────────────────────────────────────────────────────────┘

                         nanobot (LEAF)
                            │
                            ├──[HARD]──> ClawWork
                            │
                            └──[TEST]──> ai-native-self-learning-agents

TIER 2: Research Labs
┌─────────────────────────────────────────────────────────────┐
│ lunark-ondevice-ai-lab    lunark-chatbot-lab                │
│ (isolated)                (isolated)                        │
└─────────────────────────────────────────────────────────────┘

TIER 3: Frameworks
┌─────────────────────────────────────────────────────────────┐
│ openmanus    symphony                                       │
│ (isolated)   (isolated)                                     │
└─────────────────────────────────────────────────────────────┘

TIER 4: Economic/Benchmarks
                    economic-sdk (LEAF)
                            │
                            ├──[HARD]──> ClawWork
                            │
                            └──[SOFT]──> ai-native-self-learning-agents

TIER 5: Infrastructure
┌─────────────────────────────────────────────────────────────┐
│ harness-engineering    trpg-chatbot-lab    documentation    │
│ (isolated)             (isolated)          (isolated)       │
└─────────────────────────────────────────────────────────────┘
```

## DEPENDENCY DETAILS

| Consumer | Provider | Type | Import Pattern | Coupling Risk |
|----------|----------|------|----------------|---------------|
| ClawWork | nanobot | HARD | `from nanobot import Agent, MessageBus, SessionManager, ChannelManager, load_config` | **HIGH** — production runtime dependency |
| ClawWork | economic-sdk | SOFT | `try: from economic_sdk import EconomicTracker, TrackerConfig` | LOW — graceful degradation |
| ai-native-self-learning-agents | nanobot | TEST | `from nanobot import ...` (test files only) | NONE — dev-time only |
| ai-native-self-learning-agents | economic-sdk | SOFT | `try: from economic_sdk import ...` in EconomicFeedbackAgent | LOW — optional feature |

### Import Strength Legend
- **HARD**: Direct import, fails if missing
- **SOFT**: Lazy import with try/except, degrades gracefully
- **TEST**: Dev-time only, not in production code

## DEPLOYMENT INDEPENDENCE MATRIX

| Project | Can Deploy Alone? | Dependencies | Notes |
|---------|-------------------|--------------|-------|
| openclaw | ✅ YES | None | Multi-channel gateway, fully isolated |
| nanoclaw | ✅ YES | None | Container-isolated agent |
| nanobot | ✅ YES | None | PyPI package, no cross-project deps |
| seedbot | ✅ YES | None | Bash + Codex CLI, standalone |
| lunark-ondevice-ai-lab | ✅ YES | None | Research experiments, isolated |
| ai-native-self-learning-agents | ✅ YES | nanobot (test), economic-sdk (soft) | Soft deps degrade gracefully |
| lunark-chatbot-lab | ✅ YES | None | Evaluation scaffolding, isolated |
| openmanus | ✅ YES | None | Computer use agent, standalone |
| symphony | ✅ YES | None | Elixir orchestration, isolated |
| ClawWork | ❌ NO | nanobot (hard), economic-sdk (soft) | Requires nanobot at runtime |
| economic-sdk | ✅ YES | None | JSONL ledger, no cross-project deps |
| harness-engineering | ✅ YES | None | QA infrastructure, project-agnostic |
| trpg-chatbot-lab | ✅ YES | None | Event-sourced engine, isolated |
| documentation | ✅ YES | None | Markdown reports, no runtime deps |

**Summary:** 13/14 projects (93%) can deploy independently. Only ClawWork requires nanobot.

## DEVELOPMENT ORDER

If building the ecosystem from scratch, follow this sequence:

### Phase 1: Foundation (Parallel)
```
nanobot              # Lightweight agent core (<500 LOC)
economic-sdk         # Economic tracking primitives
harness-engineering  # QA infrastructure (6-Gate system)
```

### Phase 2: Dependents (After Phase 1)
```
ClawWork             # Requires nanobot + economic-sdk
```

### Phase 3: Independent Projects (Parallel, Any Time)
```
# Tier 1: Personal Assistants
openclaw             # Multi-channel gateway
nanoclaw             # Container-isolated agent
seedbot              # Self-evolving agent

# Tier 2: Research Labs
lunark-ondevice-ai-lab          # On-device ML experiments
ai-native-self-learning-agents  # Production agent framework
lunark-chatbot-lab              # Chatbot evaluation

# Tier 3: Frameworks
openmanus            # Computer use agent
symphony             # Agent orchestration

# Tier 5: Infrastructure
trpg-chatbot-lab     # TRPG/divination engine
documentation        # Ecosystem validation reports
```

**Critical Path:** nanobot → ClawWork (only hard dependency chain)

## INTEGRATION NOTES

### ClawWork ↔ nanobot Coupling

**Why ClawWork depends on nanobot:**
- ClawWork is an economic benchmark that measures agent productivity
- Uses nanobot's `Agent`, `MessageBus`, `SessionManager`, `ChannelManager` as the agent runtime
- Imports `load_config` for configuration management
- **Design decision:** ClawWork validates nanobot's economic viability ($19K/8hrs proven)

**Coupling risk mitigation:**
- nanobot is stable (<500 LOC, PyPI published)
- ClawWork pins nanobot version in requirements.txt
- Breaking changes in nanobot trigger ClawWork test failures immediately

### ClawWork ↔ economic-sdk Coupling

**Why soft import:**
- Economic tracking is optional for ClawWork's core benchmark functionality
- Lazy import pattern: `try: from economic_sdk import ...` with fallback
- Enables ClawWork to run without economic-sdk (degraded mode)

### ai-native-self-learning-agents ↔ nanobot

**Test-only dependency:**
- nanobot imported in test files for integration testing
- Not used in production code paths
- Safe to break without runtime impact

### ai-native-self-learning-agents ↔ economic-sdk

**Soft dependency in EconomicFeedbackAgent:**
- Optional feature for economic feedback loops
- Lazy import with try/except
- Agent framework functions fully without economic-sdk

## ANTI-PATTERNS AVOIDED

✅ **No circular dependencies** — all deps are unidirectional  
✅ **No deep dependency chains** — max depth is 1 (nanobot → ClawWork)  
✅ **No implicit coupling** — all imports are explicit and documented  
✅ **No version conflicts** — each project manages its own dependencies  
✅ **No monorepo lock-in** — independent git repos, independent releases  

## FUTURE CONSIDERATIONS

**If adding new cross-project dependencies:**
1. Prefer soft imports (try/except) over hard imports
2. Document coupling risk in both provider and consumer AGENTS.md
3. Add integration tests in harness-engineering
4. Update this dependency map
5. Consider extracting shared code to a new leaf project instead

**Refactoring opportunities:**
- Extract shared config patterns from nanobot/ClawWork into a config-sdk (if 3+ projects need it)
- Consider making ClawWork's nanobot dependency soft (plugin architecture)
- Evaluate economic-sdk promotion to Tier 3 (Framework) if adoption grows

## METRICS

- **Total projects:** 14
- **Leaf projects (no deps):** 12 (86%)
- **Projects with hard deps:** 1 (ClawWork)
- **Projects with soft deps:** 2 (ClawWork, ai-native-self-learning-agents)
- **Max dependency depth:** 1
- **Deployment independence:** 93% (13/14)
- **Parallel development capacity:** 12 projects (86%)

---

**Maintained by:** @zzragida  
**Last validated:** 2026-03-10  
**Related docs:** `/AGENTS.md` (root), `INTEGRATION_ROADMAP.md`, `HARNESS_INTEGRATION.md`
