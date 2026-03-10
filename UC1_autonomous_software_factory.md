# UC1: 자율 소프트웨어 팩토리 (Autonomous Software Factory)

**Version:** 1.0.0  
**Date:** 2026-03-10  
**Status:** Implementation-Ready Specification  
**Owner:** @zzragida  

---

## 1. Overview & Vision

### 1.1 Executive Summary

The Autonomous Software Factory transforms Linear issue tickets into production-ready code without human intervention. Symphony orchestrates agent workflows, Codex executes implementation tasks, Agenthub provides distributed version control and collaboration, and Harness Engineering enforces quality gates. The system operates as a continuous delivery pipeline where agents are first-class contributors.

**Core Value Proposition:**
- Zero-touch issue resolution for well-specified tickets
- Mechanical quality enforcement (6-Gate QA)
- Economic accountability (pay-per-merged-PR model)
- Multi-agent collaboration with conflict resolution

### 1.2 Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| PR merge rate | >60% auto-merged | GitHub API |
| Time-to-merge | <4 hours (median) | Linear → GitHub timestamp delta |
| Defect rate | <5% post-merge bugs | Issue reopens within 7 days |
| Gate pass rate | >95% on first attempt | Harness logs |
| Economic ROI | >300% (revenue vs. LLM cost) | Economic SDK ledger |

### 1.3 Out of Scope (MVP)

- Multi-repository coordination (single repo only)
- Human code review integration (auto-merge only)
- Custom gate definitions (6-Gate fixed)
- Real-time collaboration UI (async only)
- Cross-project dependency resolution

---

## 2. Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         LINEAR (Issue Tracker)                   │
│  Issues: {id, title, description, state, priority, labels}      │
└────────────────┬────────────────────────────────────────────────┘
                 │ Poll every 30s
                 │ (GET /api/issues?filter=state:todo)
                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SYMPHONY (Orchestrator)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Linear       │  │ Agent        │  │ Agenthub     │          │
│  │ Poller       │─▶│ Runner       │─▶│ Client       │          │
│  │ (GenServer)  │  │ (Task.async) │  │ (HTTP)       │          │
│  └──────────────┘  └──────────────┘  └──────┬───────┘          │
│         │                  │                 │                   │
│         │                  ▼                 │                   │
│         │          ┌──────────────┐          │                   │
│         │          │ Harness      │          │                   │
│         │          │ Runner       │          │                   │
│         │          │ (Port)       │          │                   │
│         │          └──────────────┘          │                   │
│         │                  │                 │                   │
└─────────┼──────────────────┼─────────────────┼───────────────────┘
          │                  │                 │
          │                  │                 ▼
          │                  │    ┌─────────────────────────────┐
          │                  │    │   AGENTHUB (Git DAG + MQ)   │
          │                  │    │  ┌────────┐  ┌────────┐     │
          │                  │    │  │ Git    │  │ Message│     │
          │                  │    │  │ Bundle │  │ Board  │     │
          │                  │    │  │ Store  │  │ (Posts)│     │
          │                  │    │  └────────┘  └────────┘     │
          │                  │    │  ┌────────────────────┐     │
          │                  │    │  │ SQLite DB          │     │
          │                  │    │  │ (agents, commits,  │     │
          │                  │    │  │  channels, posts)  │     │
          │                  │    │  └────────────────────┘     │
          │                  │    └─────────────────────────────┘
          │                  │                 │
          │                  │                 │ POST /api/git/push
          │                  │                 │ GET /api/git/fetch/{hash}
          │                  │                 │ POST /api/channels/{name}/posts
          │                  │                 │
          ▼                  ▼                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    CODEX APP-SERVER (Agent)                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Conversation Loop:                                       │   │
│  │  1. Read issue description                              │   │
│  │  2. Plan implementation                                 │   │
│  │  3. Execute (Read/Edit/Bash tools)                      │   │
│  │  4. Self-verify (LSP diagnostics, tests)               │   │
│  │  5. Push bundle to Agenthub                             │   │
│  │  6. Trigger Harness gates                               │   │
│  │  7. Report results to Symphony                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
          │
          │ Shell subprocess
          ▼
┌─────────────────────────────────────────────────────────────────┐
│              HARNESS ENGINEERING (QA Gates)                      │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐   │
│  │Gate 1│─▶│Gate 2│─▶│Gate 3│─▶│Gate 4│─▶│Gate 5│─▶│Gate 6│   │
│  │Format│  │Import│  │AST   │  │Snap  │  │Tests │  │Equiv │   │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘   │
│     ▲                                                            │
│     │ .harness/run-gates.sh                                     │
│     │                                                            │
└─────┼──────────────────────────────────────────────────────────┘
      │
      │ Exit code + JSON output
      ▼
┌─────────────────────────────────────────────────────────────────┐
│          AI-NATIVE-AGENTIC-ECONOMIC-SDK (Ledger)                │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ JSONL Ledger:                                            │   │
│  │  - track_llm_call(model, tokens, cost)                  │   │
│  │  - add_work_income(amount, task_id, eval_score)         │   │
│  │  - Quality gate: payment only if score >= 0.6           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Component Responsibilities

### 3.1 Symphony (Orchestrator)

**Location:** `/home/lunark/projects/ai-native-agentic-org/symphony/`

**Responsibilities:**
- Poll Linear API every 30 seconds for `state:todo` issues
- Launch one Codex app-server per issue (isolated process)
- Monitor agent progress via HTTP API (`GET /api/v1/state`)
- Retry failed runs with exponential backoff (10s, 20s, 40s, 80s, 160s max)
- Update Linear issue state on completion/failure
- Coordinate with Agenthub for bundle push/fetch
- Trigger Harness gates post-implementation

**Key Modules:**
- `lib/symphony_elixir/linear/poller.ex` (GenServer, 30s interval)
- `lib/symphony_elixir/agent_runner.ex` (Task.Supervisor for Codex processes)
- `lib/symphony_elixir/agenthub/client.ex` (NEW: HTTP client for Agenthub)
- `lib/symphony_elixir/harness/runner.ex` (NEW: Port wrapper for run-gates.sh)

**Configuration:** `WORKFLOW.md` (YAML schema)

### 3.2 Agenthub (Distributed VCS + Message Board)

**Location:** `/home/lunark/projects/ai-native-agentic-org/agenthub/`

**Responsibilities:**
- Store git bundles as content-addressed blobs (SHA-256)
- Provide DAG traversal API (leaves, ancestors, diff)
- Message board for agent-to-agent communication
- Rate limiting (60 pushes/hr, 60 posts/hr, 60 diffs/hr per agent)
- Authentication via Bearer tokens
- Post-push hook integration for Harness gates

**Key Modules:**
- `internal/git/bundle.go` (bundle storage, fetch, push)
- `internal/api/git_handler.go` (HTTP endpoints)
- `internal/api/channel_handler.go` (message board)
- `internal/db/sqlite.go` (agents, commits, channels, posts, rate_limits)
- `internal/harness/runner.go` (NEW: post-push gate execution)

**Database Schema:**
```sql
CREATE TABLE agents (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    api_key TEXT NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE commits (
    hash TEXT PRIMARY KEY,
    agent_id TEXT NOT NULL,
    parent_hash TEXT,
    bundle_path TEXT NOT NULL,
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES agents(id)
);

CREATE TABLE gate_results (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    commit_hash TEXT NOT NULL,
    gate_name TEXT NOT NULL,
    status TEXT NOT NULL, -- pass/fail/skip
    output TEXT,
    duration_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (commit_hash) REFERENCES commits(hash)
);

CREATE TABLE channels (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    channel_id INTEGER NOT NULL,
    agent_id TEXT NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (channel_id) REFERENCES channels(id),
    FOREIGN KEY (agent_id) REFERENCES agents(id)
);

CREATE TABLE rate_limits (
    agent_id TEXT NOT NULL,
    action TEXT NOT NULL, -- push/post/diff
    count INTEGER DEFAULT 0,
    window_start TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (agent_id, action)
);
```

### 3.3 Harness Engineering (QA Gates)

**Location:** `/home/lunark/projects/ai-native-agentic-org/harness-engineering/`

**Responsibilities:**
- Execute 6-Gate QA pipeline in sequence
- Fail fast on first gate failure (unless `--continue-on-error`)
- Generate JSON report with per-gate status
- Validate AGENTS.md compliance
- Enforce RPEQ workflow (Research → Plan → Execute → QA)

**6-Gate Pipeline:**
1. **Format Gate:** `ruff format --check`, `oxfmt --check`, `mix format --check-formatted`
2. **Import Boundaries:** AST analysis for cross-experiment imports (Python), circular deps (TS)
3. **AST Ratchets:** Complexity metrics (cyclomatic, cognitive), forbidden patterns (`eval`, `exec`)
4. **Snapshots:** Vitest snapshots (TS), pytest snapshot tests (Python)
5. **Tests:** `pytest -m "not slow"`, `vitest run`, `mix test`
6. **Equivalence:** Differential testing (compare outputs before/after refactor)

**Key Scripts:**
- `.harness/run-gates.sh` (entry point, calls gate-*.sh)
- `scripts/gate-1-format.sh`
- `scripts/gate-2-import-boundaries.sh`
- `scripts/gate-3-ast-ratchets.sh`
- `scripts/gate-4-snapshots.sh`
- `scripts/gate-5-tests.sh`
- `scripts/gate-6-equivalence.sh`
- `scripts/validate-agents-md.sh`

**Output Format (JSON):**
```json
{
  "gates": [
    {"name": "format", "status": "pass", "duration_ms": 1234, "output": ""},
    {"name": "import_boundaries", "status": "pass", "duration_ms": 567, "output": ""},
    {"name": "ast_ratchets", "status": "fail", "duration_ms": 890, "output": "Cyclomatic complexity 15 > 10 in foo.py:42"},
    {"name": "snapshots", "status": "skip", "duration_ms": 0, "output": "No snapshot tests found"},
    {"name": "tests", "status": "skip", "duration_ms": 0, "output": "Skipped due to previous failure"},
    {"name": "equivalence", "status": "skip", "duration_ms": 0, "output": "Skipped due to previous failure"}
  ],
  "overall": "fail",
  "total_duration_ms": 2691
}
```

### 3.4 AI-Native-Agentic-Economic-SDK (Ledger)

**Location:** `/home/lunark/projects/ai-native-agentic-org/ai-native-agentic-economic-sdk/`

**Responsibilities:**
- Track LLM API costs (model, tokens, USD)
- Record work income (task completion, evaluation score)
- Enforce quality gate: payment only if `evaluation_score >= 0.6`
- Generate daily/weekly economic reports
- Compute sustainability metrics (ROI, pay rate, burn rate)

**Key Classes:**
- `EconomicTracker` (main API)
- `LedgerEntry` (dataclass: timestamp, type, amount, metadata)
- `QualityGate` (evaluation score threshold)
- `MetricsCalculator` (ROI, pay rate, sustainability score)

**Ledger Format (JSONL):**
```jsonl
{"timestamp": "2026-03-10T10:30:00Z", "type": "llm_call", "amount": -0.05, "metadata": {"model": "claude-sonnet-4-5", "tokens": 2500}}
{"timestamp": "2026-03-10T11:45:00Z", "type": "work_income", "amount": 2.00, "metadata": {"task_id": "LIN-123", "evaluation_score": 0.85}}
{"timestamp": "2026-03-10T12:00:00Z", "type": "work_income", "amount": 0.00, "metadata": {"task_id": "LIN-124", "evaluation_score": 0.45, "rejected": true}}
```

**Integration Points:**
- Symphony calls `tracker.track_llm_call()` after each Codex run
- Symphony calls `tracker.add_work_income()` after PR merge (evaluation score from Harness gates)

---

## 4. Integration Points (Exact APIs)

### 4.1 Symphony ↔ Linear

**Endpoint:** `https://api.linear.app/graphql`

**Authentication:** Bearer token in `Authorization` header

**Query (Poll Issues):**
```graphql
query GetTodoIssues {
  issues(filter: {state: {name: {eq: "Todo"}}}) {
    nodes {
      id
      identifier
      title
      description
      state {
        name
      }
      priority
      labels {
        nodes {
          name
        }
      }
      createdAt
      updatedAt
    }
  }
}
```

**Mutation (Update Issue State):**
```graphql
mutation UpdateIssueState($issueId: String!, $stateId: String!) {
  issueUpdate(id: $issueId, input: {stateId: $stateId}) {
    success
    issue {
      id
      state {
        name
      }
    }
  }
}
```

**Configuration (WORKFLOW.md):**
```yaml
tracker:
  type: linear
  api_key: $LINEAR_API_KEY
  team_id: $LINEAR_TEAM_ID
  polling_interval_ms: 30000
  states:
    todo: "Todo"
    in_progress: "In Progress"
    done: "Done"
    failed: "Failed"
```

### 4.2 Symphony ↔ Codex App-Server

**Launch Command:**
```bash
codex app-server \
  --port 8081 \
  --cwd /path/to/workspace \
  --base-instructions "You are an autonomous agent. Implement the issue described below. Push your changes to Agenthub when done." \
  --developer-instructions "Issue: ${issue_title}\n\n${issue_description}" \
  --approval-policy never \
  --sandbox workspace-write
```

**State Monitoring (HTTP):**
```bash
GET http://localhost:8081/api/v1/state
```

**Response:**
```json
{
  "status": "running",
  "conversation_id": "conv_abc123",
  "message_count": 15,
  "last_activity": "2026-03-10T10:45:00Z",
  "workspace": "/path/to/workspace"
}
```

**Completion Detection:**
- Poll `/api/v1/state` every 5 seconds
- Status transitions: `running` → `completed` or `failed`
- Timeout: 30 minutes (configurable)

### 4.3 Symphony ↔ Agenthub

**Base URL:** `http://localhost:8080` (configurable)

**Authentication:** `Authorization: Bearer ${AGENTHUB_API_KEY}`

**Push Bundle:**
```bash
POST /api/git/push
Content-Type: application/octet-stream
Body: <git bundle binary data>

Response:
{
  "hash": "a1b2c3d4e5f6...",
  "parent_hash": "f6e5d4c3b2a1...",
  "agent_id": "symphony-agent-1",
  "created_at": "2026-03-10T10:50:00Z"
}
```

**Fetch Bundle:**
```bash
GET /api/git/fetch/a1b2c3d4e5f6

Response: <git bundle binary data>
```

**Get DAG Leaves:**
```bash
GET /api/git/leaves

Response:
{
  "leaves": [
    {"hash": "a1b2c3d4e5f6", "agent_id": "symphony-agent-1", "created_at": "2026-03-10T10:50:00Z"},
    {"hash": "b2c3d4e5f6a1", "agent_id": "symphony-agent-2", "created_at": "2026-03-10T10:55:00Z"}
  ]
}
```

**Get Diff:**
```bash
GET /api/git/diff/f6e5d4c3b2a1/a1b2c3d4e5f6

Response:
{
  "from": "f6e5d4c3b2a1",
  "to": "a1b2c3d4e5f6",
  "diff": "diff --git a/foo.py b/foo.py\nindex 1234567..abcdefg 100644\n--- a/foo.py\n+++ b/foo.py\n@@ -10,3 +10,4 @@\n def bar():\n-    pass\n+    return 42\n"
}
```

**Post Message:**
```bash
POST /api/channels/symphony-logs/posts
Content-Type: application/json

{
  "content": "Agent symphony-agent-1 completed issue LIN-123 in 15 minutes"
}

Response:
{
  "id": 42,
  "channel_id": 1,
  "agent_id": "symphony-agent-1",
  "content": "Agent symphony-agent-1 completed issue LIN-123 in 15 minutes",
  "created_at": "2026-03-10T11:00:00Z"
}
```

**Rate Limits (HTTP Headers):**
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1678451200
```

### 4.4 Symphony ↔ Harness Engineering

**Execution (Shell Subprocess):**
```bash
cd /path/to/workspace
/home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh \
  --project-root . \
  --output-format json \
  --fail-fast
```

**Exit Codes:**
- `0`: All gates passed
- `1`: One or more gates failed
- `2`: Script error (missing dependencies, invalid config)

**Output (stdout, JSON):**
```json
{
  "gates": [
    {"name": "format", "status": "pass", "duration_ms": 1234, "output": ""},
    {"name": "import_boundaries", "status": "pass", "duration_ms": 567, "output": ""},
    {"name": "ast_ratchets", "status": "pass", "duration_ms": 890, "output": ""},
    {"name": "snapshots", "status": "pass", "duration_ms": 456, "output": ""},
    {"name": "tests", "status": "pass", "duration_ms": 12345, "output": "42 passed in 12.34s"},
    {"name": "equivalence", "status": "pass", "duration_ms": 3456, "output": ""}
  ],
  "overall": "pass",
  "total_duration_ms": 18948
}
```

### 4.5 Agenthub ↔ Harness Engineering

**Post-Push Hook (Internal):**

When Agenthub receives a bundle push, it triggers Harness gates:

```go
// internal/harness/runner.go
func RunGatesForCommit(commitHash string, workspaceDir string) (*GateResults, error) {
    cmd := exec.Command(
        "/home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh",
        "--project-root", workspaceDir,
        "--output-format", "json",
        "--fail-fast",
    )
    
    output, err := cmd.CombinedOutput()
    if err != nil {
        return nil, fmt.Errorf("gates failed: %w", err)
    }
    
    var results GateResults
    if err := json.Unmarshal(output, &results); err != nil {
        return nil, fmt.Errorf("failed to parse gate results: %w", err)
    }
    
    // Store results in DB
    for _, gate := range results.Gates {
        db.Exec(`
            INSERT INTO gate_results (commit_hash, gate_name, status, output, duration_ms)
            VALUES (?, ?, ?, ?, ?)
        `, commitHash, gate.Name, gate.Status, gate.Output, gate.DurationMs)
    }
    
    return &results, nil
}
```

**Query Gate Results:**
```bash
GET /api/git/commits/a1b2c3d4e5f6/gates

Response:
{
  "commit_hash": "a1b2c3d4e5f6",
  "gates": [
    {"name": "format", "status": "pass", "duration_ms": 1234, "output": ""},
    {"name": "import_boundaries", "status": "pass", "duration_ms": 567, "output": ""},
    {"name": "ast_ratchets", "status": "pass", "duration_ms": 890, "output": ""},
    {"name": "snapshots", "status": "pass", "duration_ms": 456, "output": ""},
    {"name": "tests", "status": "pass", "duration_ms": 12345, "output": "42 passed in 12.34s"},
    {"name": "equivalence", "status": "pass", "duration_ms": 3456, "output": ""}
  ],
  "overall": "pass",
  "created_at": "2026-03-10T11:05:00Z"
}
```

---

## 5. Glue Code Specification

### 5.1 Symphony: Agenthub Client (Elixir)

**File:** `/home/lunark/projects/ai-native-agentic-org/symphony/lib/symphony_elixir/agenthub/client.ex`

```elixir
defmodule SymphonyElixir.Agenthub.Client do
  @moduledoc """
  HTTP client for Agenthub API.
  Handles bundle push/fetch, message board, and rate limiting.
  """

  require Logger

  @type bundle :: binary()
  @type commit_hash :: String.t()
  @type error :: {:error, String.t()}

  @doc """
  Push a git bundle to Agenthub.
  
  ## Parameters
  - bundle: Binary git bundle data
  - parent_hash: Optional parent commit hash
  
  ## Returns
  - {:ok, commit_hash} on success
  - {:error, reason} on failure
  """
  @spec push_bundle(bundle(), String.t() | nil) :: {:ok, commit_hash()} | error()
  def push_bundle(bundle, parent_hash \\ nil) do
    url = "#{base_url()}/api/git/push"
    headers = [
      {"Authorization", "Bearer #{api_key()}"},
      {"Content-Type", "application/octet-stream"},
      {"X-Parent-Hash", parent_hash || ""}
    ]

    case HTTPoison.post(url, bundle, headers, timeout: 60_000, recv_timeout: 60_000) do
      {:ok, %{status_code: 200, body: body}} ->
        case Jason.decode(body) do
          {:ok, %{"hash" => hash}} ->
            Logger.info("Pushed bundle to Agenthub: #{hash}")
            {:ok, hash}

          {:error, reason} ->
            {:error, "Failed to parse response: #{inspect(reason)}"}
        end

      {:ok, %{status_code: 429}} ->
        {:error, "Rate limit exceeded"}

      {:ok, %{status_code: status, body: body}} ->
        {:error, "HTTP #{status}: #{body}"}

      {:error, %HTTPoison.Error{reason: reason}} ->
        {:error, "HTTP request failed: #{inspect(reason)}"}
    end
  end

  @doc """
  Fetch a git bundle from Agenthub by commit hash.
  
  ## Parameters
  - hash: Commit hash to fetch
  
  ## Returns
  - {:ok, bundle} on success
  - {:error, reason} on failure
  """
  @spec fetch_bundle(commit_hash()) :: {:ok, bundle()} | error()
  def fetch_bundle(hash) do
    url = "#{base_url()}/api/git/fetch/#{hash}"
    headers = [{"Authorization", "Bearer #{api_key()}"}]

    case HTTPoison.get(url, headers, timeout: 60_000, recv_timeout: 60_000) do
      {:ok, %{status_code: 200, body: body}} ->
        Logger.info("Fetched bundle from Agenthub: #{hash}")
        {:ok, body}

      {:ok, %{status_code: 404}} ->
        {:error, "Bundle not found: #{hash}"}

      {:ok, %{status_code: status, body: body}} ->
        {:error, "HTTP #{status}: #{body}"}

      {:error, %HTTPoison.Error{reason: reason}} ->
        {:error, "HTTP request failed: #{inspect(reason)}"}
    end
  end

  @doc """
  Get DAG leaves (commits with no children).
  
  ## Returns
  - {:ok, leaves} on success (list of %{hash, agent_id, created_at})
  - {:error, reason} on failure
  """
  @spec get_leaves() :: {:ok, list(map())} | error()
  def get_leaves do
    url = "#{base_url()}/api/git/leaves"
    headers = [{"Authorization", "Bearer #{api_key()}"}]

    case HTTPoison.get(url, headers) do
      {:ok, %{status_code: 200, body: body}} ->
        case Jason.decode(body) do
          {:ok, %{"leaves" => leaves}} -> {:ok, leaves}
          {:error, reason} -> {:error, "Failed to parse response: #{inspect(reason)}"}
        end

      {:ok, %{status_code: status, body: body}} ->
        {:error, "HTTP #{status}: #{body}"}

      {:error, %HTTPoison.Error{reason: reason}} ->
        {:error, "HTTP request failed: #{inspect(reason)}"}
    end
  end

  @doc """
  Get diff between two commits.
  
  ## Parameters
  - from_hash: Starting commit hash
  - to_hash: Ending commit hash
  
  ## Returns
  - {:ok, diff_text} on success
  - {:error, reason} on failure
  """
  @spec get_diff(commit_hash(), commit_hash()) :: {:ok, String.t()} | error()
  def get_diff(from_hash, to_hash) do
    url = "#{base_url()}/api/git/diff/#{from_hash}/#{to_hash}"
    headers = [{"Authorization", "Bearer #{api_key()}"}]

    case HTTPoison.get(url, headers) do
      {:ok, %{status_code: 200, body: body}} ->
        case Jason.decode(body) do
          {:ok, %{"diff" => diff}} -> {:ok, diff}
          {:error, reason} -> {:error, "Failed to parse response: #{inspect(reason)}"}
        end

      {:ok, %{status_code: 429}} ->
        {:error, "Rate limit exceeded (60 diffs/hour)"}

      {:ok, %{status_code: status, body: body}} ->
        {:error, "HTTP #{status}: #{body}"}

      {:error, %HTTPoison.Error{reason: reason}} ->
        {:error, "HTTP request failed: #{inspect(reason)}"}
    end
  end

  @doc """
  Post a message to a channel.
  
  ## Parameters
  - channel_name: Channel name (created if doesn't exist)
  - content: Message content
  
  ## Returns
  - {:ok, post_id} on success
  - {:error, reason} on failure
  """
  @spec post_message(String.t(), String.t()) :: {:ok, integer()} | error()
  def post_message(channel_name, content) do
    url = "#{base_url()}/api/channels/#{channel_name}/posts"
    headers = [
      {"Authorization", "Bearer #{api_key()}"},
      {"Content-Type", "application/json"}
    ]
    body = Jason.encode!(%{content: content})

    case HTTPoison.post(url, body, headers) do
      {:ok, %{status_code: 200, body: response_body}} ->
        case Jason.decode(response_body) do
          {:ok, %{"id" => id}} ->
            Logger.info("Posted message to channel #{channel_name}: #{id}")
            {:ok, id}

          {:error, reason} ->
            {:error, "Failed to parse response: #{inspect(reason)}"}
        end

      {:ok, %{status_code: 429}} ->
        {:error, "Rate limit exceeded (60 posts/hour)"}

      {:ok, %{status_code: status, body: response_body}} ->
        {:error, "HTTP #{status}: #{response_body}"}

      {:error, %HTTPoison.Error{reason: reason}} ->
        {:error, "HTTP request failed: #{inspect(reason)}"}
    end
  end

  # Private helpers

  defp base_url do
    Application.get_env(:symphony_elixir, :agenthub_url, "http://localhost:8080")
  end

  defp api_key do
    System.get_env("AGENTHUB_API_KEY") ||
      raise "AGENTHUB_API_KEY environment variable not set"
  end
end
```

### 5.2 Symphony: Harness Runner (Elixir)

**File:** `/home/lunark/projects/ai-native-agentic-org/symphony/lib/symphony_elixir/harness/runner.ex`

```elixir
defmodule SymphonyElixir.Harness.Runner do
  @moduledoc """
  Subprocess wrapper for Harness Engineering 6-Gate QA.
  Executes run-gates.sh and parses JSON results.
  """

  require Logger

  @type gate_result :: %{
          name: String.t(),
          status: String.t(),
          duration_ms: integer(),
          output: String.t()
        }

  @type gate_results :: %{
          gates: list(gate_result()),
          overall: String.t(),
          total_duration_ms: integer()
        }

  @type error :: {:error, String.t()}

  @doc """
  Run Harness 6-Gate QA on a workspace.
  
  ## Parameters
  - workspace_dir: Absolute path to project workspace
  - opts: Options map
    - fail_fast: Stop on first gate failure (default: true)
    - timeout_ms: Maximum execution time (default: 600_000 = 10 minutes)
  
  ## Returns
  - {:ok, gate_results} on success (even if gates fail)
  - {:error, reason} on script error
  """
  @spec run_gates(String.t(), map()) :: {:ok, gate_results()} | error()
  def run_gates(workspace_dir, opts \\ %{}) do
    fail_fast = Map.get(opts, :fail_fast, true)
    timeout_ms = Map.get(opts, :timeout_ms, 600_000)

    script_path = harness_script_path()
    args = [
      "--project-root",
      workspace_dir,
      "--output-format",
      "json"
    ]

    args = if fail_fast, do: args ++ ["--fail-fast"], else: args

    Logger.info("Running Harness gates: #{script_path} #{Enum.join(args, " ")}")

    case System.cmd(script_path, args,
           stderr_to_stdout: true,
           cd: workspace_dir,
           env: [{"PATH", System.get_env("PATH")}],
           timeout: timeout_ms
         ) do
      {output, 0} ->
        parse_results(output)

      {output, 1} ->
        # Gates failed, but script succeeded
        parse_results(output)

      {output, exit_code} ->
        Logger.error("Harness script error (exit #{exit_code}): #{output}")
        {:error, "Script exited with code #{exit_code}: #{output}"}
    end
  rescue
    e in ErlangError ->
      if e.original == :timeout do
        {:error, "Harness gates timed out"}
      else
        {:error, "Unexpected error: #{inspect(e)}"}
      end
  end

  @doc """
  Parse JSON output from run-gates.sh.
  
  ## Parameters
  - output: Raw stdout from script
  
  ## Returns
  - {:ok, gate_results} on success
  - {:error, reason} on parse failure
  """
  @spec parse_results(String.t()) :: {:ok, gate_results()} | error()
  def parse_results(output) do
    case Jason.decode(output) do
      {:ok, %{"gates" => gates, "overall" => overall, "total_duration_ms" => duration}} ->
        results = %{
          gates:
            Enum.map(gates, fn gate ->
              %{
                name: gate["name"],
                status: gate["status"],
                duration_ms: gate["duration_ms"],
                output: gate["output"]
              }
            end),
          overall: overall,
          total_duration_ms: duration
        }

        Logger.info("Harness gates completed: #{overall} (#{duration}ms)")
        {:ok, results}

      {:ok, _} ->
        {:error, "Invalid JSON structure: missing required fields"}

      {:error, reason} ->
        {:error, "Failed to parse JSON: #{inspect(reason)}"}
    end
  end

  # Private helpers

  defp harness_script_path do
    Application.get_env(
      :symphony_elixir,
      :harness_script_path,
      "/home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh"
    )
  end
end
```

### 5.3 Symphony: Agent Runner Integration (Elixir)

**File:** `/home/lunark/projects/ai-native-agentic-org/symphony/lib/symphony_elixir/agent_runner.ex` (MODIFIED)

Add Agenthub push and Harness gate execution after Codex completion:

```elixir
defmodule SymphonyElixir.AgentRunner do
  # ... existing code ...

  alias SymphonyElixir.Agenthub.Client, as: AgenthubClient
  alias SymphonyElixir.Harness.Runner, as: HarnessRunner

  @doc """
  Run agent for a Linear issue.
  
  ## Workflow
  1. Launch Codex app-server
  2. Monitor until completion
  3. Create git bundle
  4. Push to Agenthub
  5. Run Harness gates
  6. Update Linear issue state
  7. Track economics
  """
  def run_for_issue(issue) do
    Logger.info("Starting agent for issue #{issue.identifier}")

    workspace_dir = prepare_workspace(issue)
    codex_port = launch_codex(issue, workspace_dir)

    # Monitor Codex until completion
    case monitor_codex(codex_port, timeout_ms: 1_800_000) do
      {:ok, :completed} ->
        Logger.info("Codex completed for issue #{issue.identifier}")

        # Create git bundle
        case create_bundle(workspace_dir) do
          {:ok, bundle} ->
            # Push to Agenthub
            case AgenthubClient.push_bundle(bundle) do
              {:ok, commit_hash} ->
                Logger.info("Pushed bundle: #{commit_hash}")

                # Run Harness gates
                case HarnessRunner.run_gates(workspace_dir) do
                  {:ok, %{overall: "pass"} = results} ->
                    Logger.info("All gates passed")

                    # Update Linear issue to Done
                    update_linear_issue(issue.id, "Done")

                    # Track economics (payment only if gates pass)
                    track_work_income(issue, evaluation_score: 1.0)

                    # Post success message
                    AgenthubClient.post_message(
                      "symphony-logs",
                      "✓ Issue #{issue.identifier} completed: #{commit_hash}"
                    )

                    {:ok, :completed, commit_hash, results}

                  {:ok, %{overall: "fail"} = results} ->
                    Logger.warn("Gates failed for issue #{issue.identifier}")

                    # Update Linear issue to Failed
                    update_linear_issue(issue.id, "Failed")

                    # No payment for failed gates
                    track_work_income(issue, evaluation_score: 0.0)

                    # Post failure message
                    AgenthubClient.post_message(
                      "symphony-logs",
                      "✗ Issue #{issue.identifier} failed gates: #{commit_hash}"
                    )

                    {:error, :gates_failed, results}

                  {:error, reason} ->
                    Logger.error("Harness error: #{reason}")
                    update_linear_issue(issue.id, "Failed")
                    {:error, :harness_error, reason}
                end

              {:error, reason} ->
                Logger.error("Agenthub push failed: #{reason}")
                update_linear_issue(issue.id, "Failed")
                {:error, :push_failed, reason}
            end

          {:error, reason} ->
            Logger.error("Bundle creation failed: #{reason}")
            update_linear_issue(issue.id, "Failed")
            {:error, :bundle_failed, reason}
        end

      {:error, :timeout} ->
        Logger.error("Codex timed out for issue #{issue.identifier}")
        update_linear_issue(issue.id, "Failed")
        {:error, :timeout}

      {:error, reason} ->
        Logger.error("Codex failed: #{reason}")
        update_linear_issue(issue.id, "Failed")
        {:error, :codex_failed, reason}
    end
  end

  defp create_bundle(workspace_dir) do
    bundle_path = Path.join(System.tmp_dir!(), "bundle-#{:rand.uniform(999999)}.bundle")

    case System.cmd("git", ["bundle", "create", bundle_path, "HEAD"],
           cd: workspace_dir,
           stderr_to_stdout: true
         ) do
      {_output, 0} ->
        bundle = File.read!(bundle_path)
        File.rm!(bundle_path)
        {:ok, bundle}

      {output, exit_code} ->
        {:error, "git bundle failed (exit #{exit_code}): #{output}"}
    end
  end

  defp track_work_income(issue, opts) do
    evaluation_score = Keyword.get(opts, :evaluation_score, 0.0)

    # Calculate payment based on issue priority
    amount =
      case issue.priority do
        1 -> 5.0  # Urgent
        2 -> 3.0  # High
        3 -> 2.0  # Medium
        4 -> 1.0  # Low
        _ -> 0.5  # No priority
      end

    # Economic SDK integration (lazy import)
    try do
      tracker = EconomicTracker.new()

      EconomicTracker.add_work_income(tracker, amount,
        task_id: issue.identifier,
        evaluation_score: evaluation_score
      )

      Logger.info("Tracked work income: $#{amount} (score: #{evaluation_score})")
    rescue
      _ -> Logger.warn("Economic SDK not available, skipping income tracking")
    end
  end

  # ... rest of existing code ...
end
```

### 5.4 Agenthub: Harness Integration (Go)

**File:** `/home/lunark/projects/ai-native-agentic-org/agenthub/internal/harness/runner.go` (NEW)

```go
package harness

import (
	"encoding/json"
	"fmt"
	"os/exec"
	"time"
)

// GateResult represents a single gate execution result
type GateResult struct {
	Name       string `json:"name"`
	Status     string `json:"status"` // pass/fail/skip
	DurationMs int64  `json:"duration_ms"`
	Output     string `json:"output"`
}

// GateResults represents the full gate execution results
type GateResults struct {
	Gates           []GateResult `json:"gates"`
	Overall         string       `json:"overall"` // pass/fail
	TotalDurationMs int64        `json:"total_duration_ms"`
}

// Runner executes Harness gates for a commit
type Runner struct {
	scriptPath string
	db         *sql.DB
}

// NewRunner creates a new Harness runner
func NewRunner(scriptPath string, db *sql.DB) *Runner {
	return &Runner{
		scriptPath: scriptPath,
		db:         db,
	}
}

// RunGatesForCommit executes Harness gates for a commit and stores results
func (r *Runner) RunGatesForCommit(commitHash string, workspaceDir string) (*GateResults, error) {
	startTime := time.Now()

	cmd := exec.Command(
		r.scriptPath,
		"--project-root", workspaceDir,
		"--output-format", "json",
		"--fail-fast",
	)
	cmd.Dir = workspaceDir

	output, err := cmd.CombinedOutput()
	duration := time.Since(startTime)

	// Parse results even if command failed (gates may have failed)
	var results GateResults
	if parseErr := json.Unmarshal(output, &results); parseErr != nil {
		return nil, fmt.Errorf("failed to parse gate results: %w (output: %s)", parseErr, string(output))
	}

	// Store results in database
	for _, gate := range results.Gates {
		_, dbErr := r.db.Exec(`
			INSERT INTO gate_results (commit_hash, gate_name, status, output, duration_ms, created_at)
			VALUES (?, ?, ?, ?, ?, ?)
		`, commitHash, gate.Name, gate.Status, gate.Output, gate.DurationMs, time.Now())

		if dbErr != nil {
			return nil, fmt.Errorf("failed to store gate result: %w", dbErr)
		}
	}

	// Log overall result
	if results.Overall == "pass" {
		log.Printf("Harness gates passed for commit %s (duration: %v)", commitHash, duration)
	} else {
		log.Printf("Harness gates failed for commit %s (duration: %v)", commitHash, duration)
	}

	// Return results even if gates failed (not an error condition)
	return &results, nil
}

// GetGateResults retrieves stored gate results for a commit
func (r *Runner) GetGateResults(commitHash string) (*GateResults, error) {
	rows, err := r.db.Query(`
		SELECT gate_name, status, output, duration_ms
		FROM gate_results
		WHERE commit_hash = ?
		ORDER BY created_at ASC
	`, commitHash)
	if err != nil {
		return nil, fmt.Errorf("failed to query gate results: %w", err)
	}
	defer rows.Close()

	var gates []GateResult
	var totalDuration int64

	for rows.Next() {
		var gate GateResult
		if err := rows.Scan(&gate.Name, &gate.Status, &gate.Output, &gate.DurationMs); err != nil {
			return nil, fmt.Errorf("failed to scan gate result: %w", err)
		}
		gates = append(gates, gate)
		totalDuration += gate.DurationMs
	}

	if len(gates) == 0 {
		return nil, fmt.Errorf("no gate results found for commit %s", commitHash)
	}

	// Determine overall status
	overall := "pass"
	for _, gate := range gates {
		if gate.Status == "fail" {
			overall = "fail"
			break
		}
	}

	return &GateResults{
		Gates:           gates,
		Overall:         overall,
		TotalDurationMs: totalDuration,
	}, nil
}
```

**File:** `/home/lunark/projects/ai-native-agentic-org/agenthub/internal/api/git_handler.go` (MODIFIED)

Add gate results endpoint:

```go
// GetCommitGates returns gate results for a commit
func (h *GitHandler) GetCommitGates(w http.ResponseWriter, r *http.Request) {
	commitHash := chi.URLParam(r, "hash")

	results, err := h.harnessRunner.GetGateResults(commitHash)
	if err != nil {
		http.Error(w, err.Error(), http.StatusNotFound)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(results)
}
```

**File:** `/home/lunark/projects/ai-native-agentic-org/agenthub/cmd/server/main.go` (MODIFIED)

Register new route:

```go
r.Get("/api/git/commits/{hash}/gates", gitHandler.GetCommitGates)
```

### 5.5 WORKFLOW.md Schema Extension

**File:** `/home/lunark/projects/ai-native-agentic-org/symphony/WORKFLOW.md` (MODIFIED)

Add new sections:

```yaml
# ... existing tracker, polling, workspace, agent sections ...

# Agenthub integration
agenthub:
  enabled: true
  server_url: http://localhost:8080
  api_key: $AGENTHUB_API_KEY
  push_after_run: true
  post_to_channel: symphony-logs

# Harness Engineering integration
harness:
  enabled: true
  script_path: /home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh
  fail_on_gate_failure: true
  timeout_ms: 600000  # 10 minutes

# Economic tracking
economics:
  enabled: true
  ledger_path: /var/lib/symphony/economic-ledger.jsonl
  payment_per_priority:
    1: 5.0   # Urgent
    2: 3.0   # High
    3: 2.0   # Medium
    4: 1.0   # Low
    default: 0.5
  quality_gate_threshold: 0.6  # Minimum evaluation score for payment
```

---

## 6. Data Flow Walkthrough

### 6.1 Happy Path: Issue → Merged PR

**Step 1: Linear Issue Created**
```
User creates issue in Linear:
  Title: "Add user authentication endpoint"
  Description: "Implement POST /api/auth/login with JWT tokens"
  State: Todo
  Priority: High (2)
```

**Step 2: Symphony Polls Linear**
```
[10:00:00] Symphony.Linear.Poller: Polling Linear API
[10:00:01] Found 1 new issue: LIN-123
[10:00:01] Symphony.AgentRunner: Starting agent for LIN-123
[10:00:01] Workspace prepared: /tmp/symphony-workspace-LIN-123
```

**Step 3: Codex Launched**
```
[10:00:02] Launching Codex app-server on port 8081
[10:00:02] Command: codex app-server --port 8081 --cwd /tmp/symphony-workspace-LIN-123 \
           --base-instructions "You are an autonomous agent..." \
           --developer-instructions "Issue: Add user authentication endpoint\n\nImplement POST /api/auth/login..."
[10:00:03] Codex started, conversation_id: conv_abc123
```

**Step 4: Codex Implements Feature**
```
[10:00:05] Codex: Reading codebase structure
[10:00:10] Codex: Creating /tmp/symphony-workspace-LIN-123/api/auth.py
[10:00:15] Codex: Writing JWT token generation logic
[10:00:20] Codex: Adding tests in tests/test_auth.py
[10:00:25] Codex: Running pytest... 5 passed
[10:00:30] Codex: Running LSP diagnostics... no errors
[10:00:35] Codex: Implementation complete
```

**Step 5: Bundle Created**
```
[10:00:36] Symphony.AgentRunner: Creating git bundle
[10:00:37] Command: git bundle create /tmp/bundle-123456.bundle HEAD
[10:00:38] Bundle created: 2.3 MB
```

**Step 6: Push to Agenthub**
```
[10:00:39] Symphony.Agenthub.Client: Pushing bundle to http://localhost:8080/api/git/push
[10:00:42] Agenthub: Received bundle, hash: a1b2c3d4e5f6...
[10:00:42] Agenthub: Stored bundle at /var/lib/agenthub/bundles/a1b2c3d4e5f6.bundle
[10:00:42] Agenthub: Triggering post-push hooks
[10:00:42] Response: {"hash": "a1b2c3d4e5f6...", "agent_id": "symphony-agent-1"}
```

**Step 7: Harness Gates Executed**
```
[10:00:43] Symphony.Harness.Runner: Running gates for /tmp/symphony-workspace-LIN-123
[10:00:44] Gate 1 (format): PASS (1234ms)
[10:00:45] Gate 2 (import_boundaries): PASS (567ms)
[10:00:46] Gate 3 (ast_ratchets): PASS (890ms)
[10:00:47] Gate 4 (snapshots): PASS (456ms)
[10:00:59] Gate 5 (tests): PASS (12345ms) - 5 passed in 12.34s
[10:01:02] Gate 6 (equivalence): PASS (3456ms)
[10:01:02] Overall: PASS (18948ms)
```

**Step 8: Agenthub Stores Gate Results**
```
[10:01:03] Agenthub: Storing gate results for commit a1b2c3d4e5f6
[10:01:03] Inserted 6 gate results into database
```

**Step 9: Linear Issue Updated**
```
[10:01:04] Symphony.AgentRunner: Updating Linear issue LIN-123 to Done
[10:01:05] Linear API: Issue state updated successfully
```

**Step 10: Economic Tracking**
```
[10:01:06] Symphony.AgentRunner: Tracking work income
[10:01:06] Amount: $3.00 (priority: High)
[10:01:06] Evaluation score: 1.0 (all gates passed)
[10:01:06] Economic SDK: Payment approved (score >= 0.6)
[10:01:06] Ledger entry written: {"timestamp": "2026-03-10T10:01:06Z", "type": "work_income", "amount": 3.00, ...}
```

**Step 11: Message Posted**
```
[10:01:07] Symphony.Agenthub.Client: Posting to channel symphony-logs
[10:01:08] Message: "✓ Issue LIN-123 completed: a1b2c3d4e5f6"
[10:01:08] Post ID: 42
```

**Total Duration:** 67 seconds (1 minute 7 seconds)

### 6.2 Failure Path: Gates Fail

**Scenario:** AST ratchet gate fails due to high cyclomatic complexity

```
[10:00:46] Gate 3 (ast_ratchets): FAIL (890ms)
[10:00:46] Output: "Cyclomatic complexity 15 > 10 in api/auth.py:42"
[10:00:46] Gate 4 (snapshots): SKIP (0ms) - Previous gate failed
[10:00:46] Gate 5 (tests): SKIP (0ms) - Previous gate failed
[10:00:46] Gate 6 (equivalence): SKIP (0ms) - Previous gate failed
[10:00:46] Overall: FAIL (2691ms)

[10:00:47] Symphony.AgentRunner: Gates failed for issue LIN-123
[10:00:48] Updating Linear issue LIN-123 to Failed
[10:00:49] Economic SDK: Payment rejected (evaluation_score: 0.0)
[10:00:50] Posting failure message to symphony-logs
[10:00:51] Message: "✗ Issue LIN-123 failed gates: a1b2c3d4e5f6"
```

**Retry Logic:**
```
[10:00:52] Symphony.AgentRunner: Scheduling retry (attempt 1/5)
[10:00:52] Backoff: 10 seconds
[10:01:02] Symphony.AgentRunner: Retrying issue LIN-123
[10:01:03] Launching Codex app-server on port 8082
... (repeat workflow)
```

### 6.3 Multi-Agent Collaboration

**Scenario:** Two agents work on related issues, need to merge changes

**Agent 1 (LIN-123): Add authentication endpoint**
```
[10:00:00] Agent 1: Implements POST /api/auth/login
[10:01:00] Agent 1: Pushes bundle a1b2c3d4e5f6
[10:01:05] Agent 1: Gates pass, issue marked Done
```

**Agent 2 (LIN-124): Add authorization middleware**
```
[10:00:30] Agent 2: Implements authorization decorator
[10:01:30] Agent 2: Fetches latest leaves from Agenthub
[10:01:31] Agenthub: Returns leaf a1b2c3d4e5f6 (Agent 1's commit)
[10:01:32] Agent 2: Fetches bundle a1b2c3d4e5f6
[10:01:35] Agent 2: Merges changes locally
[10:01:40] Agent 2: Resolves conflicts (if any)
[10:01:45] Agent 2: Pushes bundle b2c3d4e5f6a1 (parent: a1b2c3d4e5f6)
[10:01:50] Agent 2: Gates pass, issue marked Done
```

**DAG Structure:**
```
main (f6e5d4c3b2a1)
  |
  ├─ Agent 1 (a1b2c3d4e5f6) - Add auth endpoint
  |
  └─ Agent 2 (b2c3d4e5f6a1) - Add auth middleware (parent: a1b2c3d4e5f6)
```

---

## 7. MVP Scope

### 7.1 In Scope

**Core Features:**
- Linear issue polling (30s interval)
- Single-agent workflow (one issue per agent)
- Codex app-server integration
- Git bundle push/fetch via Agenthub
- 6-Gate QA enforcement
- Economic tracking (LLM costs + work income)
- Message board logging
- Retry logic (exponential backoff, max 5 attempts)
- Linear issue state updates (Todo → In Progress → Done/Failed)

**Supported Project Types:**
- Python (pytest, ruff, mypy)
- TypeScript (vitest, oxlint, tsc)
- Single repository only

**Quality Gates:**
- Format (ruff, oxfmt)
- Import boundaries (AST analysis)
- AST ratchets (complexity, forbidden patterns)
- Snapshots (vitest, pytest)
- Tests (pytest, vitest)
- Equivalence (differential testing)

### 7.2 Out of Scope (Post-MVP)

**Deferred Features:**
- Multi-repository coordination
- Human code review integration (GitHub PR reviews)
- Custom gate definitions (6-Gate fixed for MVP)
- Real-time collaboration UI (async only)
- Cross-project dependency resolution
- Agent-to-agent negotiation (conflict resolution is manual)
- Advanced retry strategies (fixed exponential backoff only)
- Multi-language support beyond Python/TypeScript
- Performance optimization (caching, parallel gates)
- Monitoring dashboard (logs only)

### 7.3 Success Criteria (MVP)

| Criterion | Target | Measurement |
|-----------|--------|-------------|
| End-to-end workflow | 1 successful run | Manual test |
| Gate pass rate | >80% | Harness logs |
| No regressions | 0 broken tests | CI |
| Economic tracking | 1 ledger entry | JSONL file |
| Documentation | All APIs documented | This spec |

---

## 8. 2-Week Sprint Plan

### Week 1: Foundation & Integration

**Day 1 (Mon): Symphony Agenthub Client**
- [ ] Create `lib/symphony_elixir/agenthub/client.ex`
- [ ] Implement `push_bundle/2`, `fetch_bundle/1`, `get_leaves/0`, `get_diff/2`, `post_message/2`
- [ ] Add HTTPoison dependency to `mix.exs`
- [ ] Write unit tests with mocked HTTP responses
- [ ] Test against local Agenthub instance

**Day 2 (Tue): Symphony Harness Runner**
- [ ] Create `lib/symphony_elixir/harness/runner.ex`
- [ ] Implement `run_gates/2`, `parse_results/1`
- [ ] Add timeout handling (10 minutes default)
- [ ] Write unit tests with fixture JSON outputs
- [ ] Test against harness-engineering repo

**Day 3 (Wed): Symphony Agent Runner Integration**
- [ ] Modify `lib/symphony_elixir/agent_runner.ex`
- [ ] Add bundle creation logic (`git bundle create`)
- [ ] Integrate Agenthub client (push after Codex completion)
- [ ] Integrate Harness runner (run gates after push)
- [ ] Add economic tracking calls
- [ ] Write integration tests (end-to-end workflow)

**Day 4 (Thu): Agenthub Harness Integration**
- [ ] Create `internal/harness/runner.go`
- [ ] Implement `RunGatesForCommit`, `GetGateResults`
- [ ] Add `gate_results` table migration
- [ ] Modify `internal/api/git_handler.go` (add `/api/git/commits/{hash}/gates` endpoint)
- [ ] Write unit tests with mocked subprocess
- [ ] Test post-push hook execution

**Day 5 (Fri): WORKFLOW.md Schema & Config**
- [ ] Extend `WORKFLOW.md` with `agenthub`, `harness`, `economics` sections
- [ ] Update Symphony config parser to read new sections
- [ ] Add environment variable validation (`AGENTHUB_API_KEY`, `LINEAR_API_KEY`)
- [ ] Write config validation tests
- [ ] Document all config options in README

### Week 2: Testing & Deployment

**Day 6 (Mon): End-to-End Testing**
- [ ] Set up test Linear workspace (sandbox team)
- [ ] Create test issues (simple, medium, complex)
- [ ] Run full workflow: Linear → Symphony → Codex → Agenthub → Harness
- [ ] Verify gate results stored in Agenthub DB
- [ ] Verify Linear issue state updates
- [ ] Verify economic ledger entries

**Day 7 (Tue): Failure Scenarios**
- [ ] Test gate failures (inject complexity violations)
- [ ] Test Codex timeout (30-minute limit)
- [ ] Test Agenthub rate limiting (exceed 60 pushes/hour)
- [ ] Test retry logic (exponential backoff)
- [ ] Test bundle creation failures (corrupted git repo)
- [ ] Verify error messages posted to message board

**Day 8 (Wed): Multi-Agent Collaboration**
- [ ] Create two related Linear issues
- [ ] Launch two Symphony agents in parallel
- [ ] Verify DAG structure in Agenthub (parent-child commits)
- [ ] Test conflict resolution (manual merge)
- [ ] Verify both issues marked Done
- [ ] Verify economic tracking for both agents

**Day 9 (Thu): Performance & Optimization**
- [ ] Profile Symphony (identify bottlenecks)
- [ ] Optimize bundle push (compression, streaming)
- [ ] Optimize gate execution (parallel where possible)
- [ ] Add logging for all critical paths
- [ ] Add metrics collection (Prometheus format)
- [ ] Document performance characteristics

**Day 10 (Fri): Documentation & Deployment**
- [ ] Write deployment guide (Docker Compose setup)
- [ ] Write operator manual (monitoring, troubleshooting)
- [ ] Write developer guide (adding new gates, extending workflow)
- [ ] Create architecture diagrams (Mermaid/PlantUML)
- [ ] Record demo video (end-to-end workflow)
- [ ] Deploy to staging environment

### Contingency Buffer

**Days 11-14:** Reserved for:
- Bug fixes from testing
- Performance issues
- Integration edge cases
- Documentation gaps
- Stakeholder feedback

---

## 9. Business Model & Revenue

### 9.1 Pricing Tiers

**Tier 1: Developer (Self-Hosted)**
- **Price:** Free (open source)
- **Target:** Individual developers, hobbyists
- **Limits:** Single agent, 10 issues/day
- **Support:** Community (GitHub issues)

**Tier 2: Team (SaaS)**
- **Price:** $150/developer/month
- **Target:** Engineering teams (5-50 developers)
- **Limits:** Unlimited agents, 1000 issues/day
- **Support:** Email support (48-hour SLA)
- **Features:**
  - Hosted Symphony + Agenthub
  - Linear integration
  - GitHub integration (auto-PR creation)
  - Economic dashboard
  - 30-day audit logs

**Tier 3: Enterprise (SaaS + On-Prem)**
- **Price:** $5,000/month flat rate (unlimited developers)
- **Target:** Large engineering orgs (50-500 developers)
- **Limits:** Unlimited agents, unlimited issues
- **Support:** Dedicated Slack channel (4-hour SLA)
- **Features:**
  - All Team features
  - On-premises deployment option
  - Custom gate definitions
  - Multi-repository coordination
  - SSO/SAML integration
  - 1-year audit logs
  - Quarterly business reviews

**Tier 4: Pay-Per-PR**
- **Price:** $2/merged PR
- **Target:** Occasional users, contractors
- **Limits:** No monthly commitment
- **Support:** Email support (72-hour SLA)
- **Features:**
  - Hosted Symphony + Agenthub
  - Linear integration
  - Economic dashboard

### 9.2 Revenue Projections (Year 1)

**Assumptions:**
- 100 Team customers (avg 10 developers each) = $150K/month
- 10 Enterprise customers = $50K/month
- 500 Pay-Per-PR users (avg 5 PRs/month) = $5K/month

**Monthly Recurring Revenue (MRR):** $205K  
**Annual Recurring Revenue (ARR):** $2.46M

**Cost Structure:**
- Infrastructure (AWS/GCP): $20K/month
- LLM API costs (Claude): $30K/month (avg $0.30/PR × 100K PRs)
- Engineering team (5 FTE): $100K/month
- Sales & marketing: $30K/month
- **Total costs:** $180K/month

**Net profit:** $25K/month ($300K/year)  
**Profit margin:** 12%

### 9.3 Key Metrics

**North Star Metric:** PR merge rate (% of agent-generated PRs merged without human intervention)

**Supporting Metrics:**
- Time-to-merge (median, p95)
- Defect rate (% of merged PRs with bugs within 7 days)
- Gate pass rate (% of first-attempt gate passes)
- Economic ROI (revenue per dollar of LLM cost)
- Customer retention (monthly churn rate)
- Net Promoter Score (NPS)

**Target Metrics (Year 1):**
- PR merge rate: >60%
- Time-to-merge (median): <4 hours
- Defect rate: <5%
- Gate pass rate: >95%
- Economic ROI: >300%
- Monthly churn: <5%
- NPS: >50

### 9.4 Go-to-Market Strategy

**Phase 1 (Months 1-3): Early Adopters**
- Launch open-source version (GitHub)
- Target AI-native engineering teams (YC companies, AI startups)
- Offer free Team tier for first 10 customers (3-month pilot)
- Collect feedback, iterate on product

**Phase 2 (Months 4-6): Product-Market Fit**
- Launch paid Team tier
- Add GitHub integration (auto-PR creation)
- Publish case studies (PR merge rate, time savings)
- Speak at conferences (AI Engineer Summit, GitHub Universe)

**Phase 3 (Months 7-12): Scale**
- Launch Enterprise tier
- Hire sales team (2 AEs, 1 SDR)
- Launch Pay-Per-PR tier
- Expand to multi-repository support
- Partner with Linear, GitHub (co-marketing)

---

## 10. Risk & Mitigation

### 10.1 Technical Risks

**Risk 1: Codex Reliability**
- **Description:** Codex may produce incorrect code, fail to converge, or timeout
- **Impact:** High (blocks entire workflow)
- **Probability:** Medium (30% of issues)
- **Mitigation:**
  - Retry logic (exponential backoff, max 5 attempts)
  - Harness gates catch most errors before merge
  - Human fallback (escalate to Linear after 5 failures)
  - Improve prompts based on failure patterns

**Risk 2: Gate False Positives**
- **Description:** Gates may fail on valid code (e.g., complexity threshold too strict)
- **Impact:** Medium (blocks valid PRs)
- **Probability:** Low (5% of runs)
- **Mitigation:**
  - Tunable gate thresholds (per-project config)
  - Manual override option (with audit log)
  - Continuous gate calibration (analyze false positive rate)

**Risk 3: Agenthub Scalability**
- **Description:** Bundle storage may grow unbounded, DB may slow down
- **Impact:** Medium (degrades performance)
- **Probability:** Medium (after 10K commits)
- **Mitigation:**
  - Implement garbage collection (prune old bundles)
  - Add DB indexes (commit_hash, agent_id, created_at)
  - Shard by agent_id (horizontal scaling)
  - Monitor storage usage (alert at 80% capacity)

**Risk 4: Multi-Agent Conflicts**
- **Description:** Two agents may modify the same file, causing merge conflicts
- **Impact:** Medium (requires manual resolution)
- **Probability:** Low (10% of parallel runs)
- **Mitigation:**
  - Implement conflict detection (diff analysis)
  - Automatic retry with latest base (fetch leaves, rebase)
  - Human escalation after 3 failed merge attempts
  - Future: Agent-to-agent negotiation protocol

### 10.2 Business Risks

**Risk 5: LLM Cost Volatility**
- **Description:** Claude API pricing may increase, eroding margins
- **Impact:** High (affects profitability)
- **Probability:** Medium (annual price changes)
- **Mitigation:**
  - Pass-through pricing (adjust per-PR cost quarterly)
  - Multi-model support (fallback to cheaper models)
  - Negotiate volume discounts with Anthropic
  - Monitor cost per PR (alert if >$0.50)

**Risk 6: Customer Adoption**
- **Description:** Teams may resist autonomous agents (trust, job security concerns)
- **Impact:** High (affects revenue)
- **Probability:** Medium (cultural resistance)
- **Mitigation:**
  - Emphasize augmentation, not replacement
  - Publish case studies (time savings, quality improvements)
  - Offer free pilots (prove value before commitment)
  - Provide human-in-the-loop option (review before merge)

**Risk 7: Competitive Landscape**
- **Description:** GitHub Copilot Workspace, Cursor, Devin may offer similar features
- **Impact:** High (market share erosion)
- **Probability:** High (inevitable)
- **Mitigation:**
  - Focus on quality (6-Gate QA is differentiator)
  - Focus on economics (transparent cost tracking)
  - Open-source core (build community moat)
  - Integrate with competitors (Copilot as execution engine)

### 10.3 Operational Risks

**Risk 8: Linear API Rate Limits**
- **Description:** Polling every 30s may hit Linear rate limits (1000 req/hour)
- **Impact:** Medium (delays issue detection)
- **Probability:** Low (only at scale)
- **Mitigation:**
  - Implement webhook support (push instead of poll)
  - Increase polling interval (60s) if rate limited
  - Cache issue state (only fetch changed issues)
  - Negotiate higher limits with Linear

**Risk 9: Security Vulnerabilities**
- **Description:** Agents may introduce security bugs (SQL injection, XSS, etc.)
- **Impact:** High (customer data breach)
- **Probability:** Low (gates catch most issues)
- **Mitigation:**
  - Add security gate (Semgrep, Bandit, ESLint security rules)
  - Sandbox agent execution (read-only by default)
  - Audit all merged PRs (automated + manual review)
  - Bug bounty program (incentivize external security research)

**Risk 10: Data Privacy**
- **Description:** Customer code sent to Claude API (GDPR, SOC 2 concerns)
- **Impact:** High (regulatory compliance)
- **Probability:** Medium (enterprise requirement)
- **Mitigation:**
  - Offer on-premises deployment (Enterprise tier)
  - Use Anthropic's zero-retention API (no training on customer data)
  - Implement data residency options (EU, US regions)
  - SOC 2 Type II certification (Year 2 goal)

---

## 11. Success Metrics

### 11.1 Product Metrics

**Primary Metrics:**
- **PR Merge Rate:** % of agent-generated PRs merged without human intervention
  - Target: >60% (MVP), >80% (Year 1)
  - Measurement: GitHub API (merged_by = agent)
- **Time-to-Merge:** Median time from issue creation to PR merge
  - Target: <4 hours (MVP), <2 hours (Year 1)
  - Measurement: Linear created_at → GitHub merged_at
- **Defect Rate:** % of merged PRs with bugs within 7 days
  - Target: <5% (MVP), <2% (Year 1)
  - Measurement: Issue reopens, hotfix commits

**Secondary Metrics:**
- **Gate Pass Rate:** % of first-attempt gate passes
  - Target: >95%
  - Measurement: Harness logs (overall = pass)
- **Economic ROI:** Revenue per dollar of LLM cost
  - Target: >300%
  - Measurement: Economic SDK (work_income / llm_cost)
- **Agent Utilization:** % of time agents are actively working
  - Target: >70%
  - Measurement: Codex runtime / total uptime

### 11.2 Business Metrics

**Revenue Metrics:**
- **MRR (Monthly Recurring Revenue):** Total subscription revenue
  - Target: $50K (Month 6), $200K (Month 12)
- **ARR (Annual Recurring Revenue):** MRR × 12
  - Target: $2.4M (Year 1)
- **ARPU (Average Revenue Per User):** MRR / active customers
  - Target: $150/developer/month
- **Customer Acquisition Cost (CAC):** Sales & marketing spend / new customers
  - Target: <$5K/customer
- **Lifetime Value (LTV):** ARPU × avg customer lifetime (months)
  - Target: >$18K (12-month retention)

**Growth Metrics:**
- **Monthly Active Users (MAU):** Unique developers using the platform
  - Target: 100 (Month 6), 500 (Month 12)
- **Net Revenue Retention (NRR):** (MRR + expansion - churn) / MRR
  - Target: >100% (expansion offsets churn)
- **Churn Rate:** % of customers canceling per month
  - Target: <5%

### 11.3 Operational Metrics

**Reliability Metrics:**
- **Uptime:** % of time Symphony + Agenthub are available
  - Target: >99.5% (4 hours downtime/month)
- **Error Rate:** % of agent runs that fail (non-gate failures)
  - Target: <1%
- **P95 Latency:** 95th percentile response time for Agenthub API
  - Target: <500ms

**Cost Metrics:**
- **LLM Cost per PR:** Average Claude API cost per merged PR
  - Target: <$0.30
- **Infrastructure Cost per PR:** AWS/GCP cost per merged PR
  - Target: <$0.10
- **Gross Margin:** (Revenue - COGS) / Revenue
  - Target: >70%

---

## 12. Appendix: Config Schemas

### 12.1 WORKFLOW.md (Complete Schema)

```yaml
# Linear issue tracker configuration
tracker:
  type: linear
  api_key: $LINEAR_API_KEY
  team_id: $LINEAR_TEAM_ID
  polling_interval_ms: 30000
  states:
    todo: "Todo"
    in_progress: "In Progress"
    done: "Done"
    failed: "Failed"

# Polling behavior
polling:
  enabled: true
  interval_ms: 30000
  max_concurrent_agents: 5
  retry:
    max_attempts: 5
    initial_backoff_ms: 10000
    max_backoff_ms: 300000
    backoff_multiplier: 2

# Workspace configuration
workspace:
  base_dir: /tmp/symphony-workspaces
  cleanup_after_completion: false
  git_config:
    user_name: "Symphony Agent"
    user_email: "agent@symphony.local"

# Agent configuration
agent:
  type: codex
  executable: codex
  args:
    - app-server
    - --approval-policy
    - never
    - --sandbox
    - workspace-write
  timeout_ms: 1800000  # 30 minutes
  port_range:
    start: 8081
    end: 8181

# Agenthub integration
agenthub:
  enabled: true
  server_url: http://localhost:8080
  api_key: $AGENTHUB_API_KEY
  push_after_run: true
  post_to_channel: symphony-logs
  retry:
    max_attempts: 3
    initial_backoff_ms: 1000
    max_backoff_ms: 10000

# Harness Engineering integration
harness:
  enabled: true
  script_path: /home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh
  fail_on_gate_failure: true
  timeout_ms: 600000  # 10 minutes
  gates:
    - format
    - import_boundaries
    - ast_ratchets
    - snapshots
    - tests
    - equivalence

# Economic tracking
economics:
  enabled: true
  ledger_path: /var/lib/symphony/economic-ledger.jsonl
  payment_per_priority:
    1: 5.0   # Urgent
    2: 3.0   # High
    3: 2.0   # Medium
    4: 1.0   # Low
    default: 0.5
  quality_gate_threshold: 0.6  # Minimum evaluation score for payment
  llm_cost_tracking: true

# Logging
logging:
  level: info
  format: json
  output: /var/log/symphony/symphony.log
  rotation:
    max_size_mb: 100
    max_files: 10

# Monitoring
monitoring:
  prometheus:
    enabled: true
    port: 9090
  health_check:
    enabled: true
    port: 8080
    path: /health
```

### 12.2 Agenthub Config (config.yaml)

```yaml
# Server configuration
server:
  host: 0.0.0.0
  port: 8080
  read_timeout_seconds: 60
  write_timeout_seconds: 60

# Database configuration
database:
  type: sqlite
  path: /var/lib/agenthub/agenthub.db
  max_connections: 10
  connection_timeout_seconds: 5

# Storage configuration
storage:
  bundles_dir: /var/lib/agenthub/bundles
  max_bundle_size_mb: 100
  cleanup:
    enabled: true
    retention_days: 90
    schedule: "0 2 * * *"  # 2 AM daily

# Rate limiting
rate_limits:
  max_pushes_per_hour: 60
  max_posts_per_hour: 60
  max_diffs_per_hour: 60
  max_fetches_per_hour: 300

# Authentication
auth:
  token_header: Authorization
  token_prefix: Bearer

# Harness integration
harness:
  enabled: true
  script_path: /home/lunark/projects/ai-native-agentic-org/harness-engineering/.harness/run-gates.sh
  run_on_push: true
  timeout_seconds: 600

# Logging
logging:
  level: info
  format: json
  output: /var/log/agenthub/agenthub.log

# Monitoring
monitoring:
  prometheus:
    enabled: true
    port: 9091
```

### 12.3 Economic SDK Config (economic_config.yaml)

```yaml
# Ledger configuration
ledger:
  path: /var/lib/symphony/economic-ledger.jsonl
  rotation:
    enabled: true
    max_size_mb: 100
    max_files: 10

# Quality gate
quality_gate:
  enabled: true
  threshold: 0.6
  reject_below_threshold: true

# LLM cost tracking
llm_costs:
  claude-sonnet-4-5:
    input_cost_per_1k_tokens: 0.003
    output_cost_per_1k_tokens: 0.015
  claude-opus-4:
    input_cost_per_1k_tokens: 0.015
    output_cost_per_1k_tokens: 0.075

# Metrics
metrics:
  compute_daily_summary: true
  compute_weekly_summary: true
  sustainability_threshold: 1.0  # ROI must be >= 1.0

# Reporting
reporting:
  enabled: true
  output_dir: /var/lib/symphony/reports
  formats:
    - json
    - csv
  schedule: "0 0 * * *"  # Midnight daily
```

---

## 13. Next Steps

### 13.1 Immediate Actions (Week 1)

1. **Set up development environment:**
   - Clone all repos (symphony, agenthub, harness-engineering, economic-sdk)
   - Install dependencies (Elixir, Go, Python)
   - Configure environment variables (LINEAR_API_KEY, AGENTHUB_API_KEY)

2. **Create feature branches:**
   - `symphony/feature/agenthub-integration`
   - `agenthub/feature/harness-integration`

3. **Implement glue code:**
   - Start with Symphony Agenthub client (Day 1)
   - Follow 2-week sprint plan

4. **Set up test infrastructure:**
   - Create test Linear workspace
   - Deploy local Agenthub instance
   - Configure test harness-engineering

### 13.2 Validation Checkpoints

**Checkpoint 1 (Day 5):** Glue code complete
- All modules implemented
- Unit tests passing
- Integration tests written (not yet passing)

**Checkpoint 2 (Day 10):** End-to-end workflow
- One successful run (Linear → Symphony → Codex → Agenthub → Harness)
- Gate results stored in Agenthub
- Economic ledger entry created

**Checkpoint 3 (Day 14):** MVP complete
- All success criteria met
- Documentation complete
- Deployed to staging

### 13.3 Post-MVP Roadmap

**Q2 2026: GitHub Integration**
- Auto-create PRs from agent commits
- Link Linear issues to GitHub PRs
- Merge PRs automatically if gates pass

**Q3 2026: Multi-Repository Support**
- Coordinate changes across multiple repos
- Dependency graph analysis
- Atomic multi-repo commits

**Q4 2026: Advanced Features**
- Custom gate definitions (YAML config)
- Agent-to-agent negotiation (conflict resolution)
- Real-time collaboration UI (WebSocket dashboard)

---

## 14. Conclusion

The Autonomous Software Factory (UC1) represents a paradigm shift in software development: agents as first-class contributors, mechanical quality enforcement, and economic accountability. By integrating Symphony (orchestration), Agenthub (distributed VCS), Harness Engineering (QA gates), and Economic SDK (ledger), we create a closed-loop system where code quality and economic viability are inseparable.

**Key Differentiators:**
- **Quality-first:** 6-Gate QA catches errors before merge (not after)
- **Economic transparency:** Every PR has a cost and value
- **Agent collaboration:** DAG-based version control enables multi-agent workflows
- **Open core:** Community can self-host, enterprises get premium features

**Success Hinges On:**
- Codex reliability (retry logic, prompt engineering)
- Gate calibration (minimize false positives)
- Customer trust (prove value through pilots)
- Operational excellence (uptime, performance, security)

This specification provides a complete blueprint for implementation. All APIs are defined, all glue code is specified, and all risks are identified. The 2-week sprint plan is aggressive but achievable with focused execution.

**Let's build the future of software development.**

---

**Document Version:** 1.0.0  
**Last Updated:** 2026-03-10  
**Authors:** @zzragida, Symphony Team  
**Status:** Ready for Implementation  

**Approval Signatures:**
- [ ] Technical Lead: _______________
- [ ] Product Manager: _______________
- [ ] Engineering Manager: _______________

**Next Review Date:** 2026-03-24 (2 weeks post-kickoff)
