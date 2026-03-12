# UC1: 자율 소프트웨어 팩토리 (Autonomous Software Factory) 상세 설계서

## 1. 개요 및 비전 (Overview & Vision)

### 1.1 비전 (Vision)
"자율 소프트웨어 팩토리"는 인간 개발자의 개입 없이 이슈 탐지, 코드 구현, 품질 검증, 그리고 배포까지의 전 과정을 AI 에이전트 군단이 자율적으로 수행하는 차세대 소프트웨어 생산 체계입니다. 본 프로젝트는 단순한 코드 생성을 넘어, 경제적 인센티브와 엄격한 품질 게이트(Quality Gates)를 결합하여 신뢰할 수 있는 소프트웨어를 지속적으로 생산하는 것을 목표로 합니다.

### 1.2 핵심 가치 (Core Values)
- **자율성 (Autonomy)**: Linear 이슈 생성부터 PR 머지까지의 과정에서 인간 개입 80% 감소.
- **품질 보장 (Quality Assurance)**: 6-Gate QA 시스템을 통한 회귀 버그 제로화.
- **경제적 효율성 (Economic Efficiency)**: 에이전트의 작업 가치를 정량화하고 ROI 기반의 리소스 할당.
- **확장성 (Scalability)**: 새로운 에이전트와 도구를 플러그인 형태로 손쉽게 추가 가능한 아키텍처 구축.

---

## 2. 아키텍처 다이어그램 (Architecture Diagram)

```text
+-----------------------------------------------------------------------+
|                         Autonomous Software Factory                   |
+-----------------------------------------------------------------------+
|                                                                       |
|  +-------------------+       +-------------------+      +----------+  |
|  |     Symphony      | <---> |      Linear       |      |  User    |  |
|  | (Orchestrator)    |       | (Issue Tracker)   |      | Dashboard|  |
|  +---------+---------+       +-------------------+      +----^-----+  |
|            |                                                 |        |
|            | 1. Poll Issues                                  |        |
|            v                                                 |        |
|  +---------+---------+       +-------------------+           |        |
|  |   AgentRunner     | ----> |   Codex Server    |           |        |
|  | (Lifecycle Mgmt)  |       | (LLM Execution)   |           |        |
|  +---------+---------+       +---------+---------+           |        |
|            |                           |                     |        |
|            | 2. Execute RPEQ           | 3. Track Costs      |        |
|            v                           v                     |        |
|  +---------+---------+       +-------------------+           |        |
|  | Harness-Eng       |       | Economic SDK      | <---------+        |
|  | (6-Gate QA)       |       | (Ledger/Metrics)  |                    |
|  +---------+---------+       +---------+---------+                    |
|            |                           |                              |
|            | 4. Push Bundle            | 5. Record Income             |
|            v                           v                              |
|  +-------------------------------------------------------+            |
|  |                      Agenthub                         |            |
|  | (Git DAG API / Message Board / Post-Push Gates)       |            |
|  +-------------------------------------------------------+            |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 3. 구성 요소별 역할 (Component Responsibilities)

### 3.1 Symphony (`/home/lunark/projects/ai-native-agentic-org/symphony/`)
- **역할**: 전체 워크플로우의 지휘자(Orchestrator).
- **핵심 기능**:
    - Linear API를 30초 주기로 폴링하여 신규 이슈 탐지.
    - 이슈별로 독립된 `AgentRunner` 프로세스 할당.
    - `WORKFLOW.md` 설정을 기반으로 에이전트 환경 구성.
    - 에이전트 실행 전후의 Hook(Pre-run, Post-run) 실행.
    - 에이전트 상태 모니터링 및 재시도 로직 관리.

### 3.2 Agenthub (`/home/lunark/projects/ai-native-agentic-org/agenthub/`)
- **역할**: 에이전트 전용 버전 관리 및 통신 허브.
- **핵심 기능**:
    - Git DAG 기반의 바이너리 번들 푸시/페치 API 제공.
    - 에이전트 간 협업을 위한 메시지 보드(Channel/Post) 운영.
    - 푸시된 코드에 대한 서버 측 Post-push QA 게이트 실행.
    - 에이전트별 Bearer 토큰 인증 및 속도 제한(Rate Limiting).
    - 커밋 이력 및 게이트 결과 영구 저장 (SQLite).

### 3.3 Harness-Engineering (`/home/lunark/projects/ai-native-agentic-org/harness-engineering/`)
- **역할**: 엄격한 품질 검증 엔진.
- **핵심 기능**:
    - **6-Gate QA 상세**:
        1. **Gate 1: Format Gate**: `ruff format`, `oxfmt`, `mix format` 등을 사용하여 코드 스타일 통일성 검증.
        2. **Gate 2: Import Boundaries**: 프로젝트 간 부적절한 참조 및 순환 의존성 탐지 (AST 분석).
        3. **Gate 3: AST Ratchets**: 코드 복잡도(Cyclomatic Complexity) 및 금지된 패턴(`eval`, `exec`) 사용 제한.
        4. **Gate 4: Snapshots**: UI 및 데이터 구조의 의도치 않은 변경을 방지하기 위한 스냅샷 비교.
        5. **Gate 5: Tests**: 단위 테스트 및 통합 테스트 실행 (Pytest, Vitest, Mix Test).
        6. **Gate 6: Equivalence**: 리팩토링 전후의 출력 결과 동일성 검증 (Differential Testing).
    - **RPEQ 워크플로우**: Research → Plan → Execute → QA 단계 강제.
    - `.harness/run-gates.sh`를 통한 자동화된 검증 스크립트 제공.
    - AGENTS.md 준수 여부 확인 및 에이전트 메타데이터 검증.

### 3.4 Economic SDK (`/home/lunark/projects/ai-native-agentic-org/ai-native-agentic-economic-sdk/`)
- **역할**: 에이전트 경제 시스템 관리.
- **핵심 기능**:
    - LLM API 호출 비용 실시간 트래킹 (`track_llm_call`).
    - 작업 완료 시 품질 점수에 따른 수익 기록 (`add_work_income`).
    - 일일 요약 및 ROI 분석 보고서 생성.
    - JSONL 기반의 불변 원장(Ledger) 관리.
    - 품질 점수 0.6 미만 시 지급 거절 로직 포함.

---

## 4. 연동 포인트 (Integration Points)

### 4.1 Symphony ↔ Linear
- **Protocol**: HTTPS / GraphQL
- **Endpoint**: `https://api.linear.app/graphql`
- **Action**: `GET` issues with state "Todo" or "Backlog".
- **Auth**: Bearer Token (Linear API Key).

### 4.2 Symphony ↔ Agenthub
- **Protocol**: HTTP / Binary (Bundle) & JSON (Messages)
- **Endpoints**:
    - `POST /api/git/push`: 코드 번들 업로드.
    - `GET /api/git/fetch/{hash}`: 특정 버전 코드 다운로드.
    - `GET /api/git/leaves`: 최신 커밋 목록 조회.
    - `POST /api/channels/{name}/posts`: 작업 로그 기록.

### 4.3 Symphony ↔ Harness
- **Protocol**: Local Subprocess (Port)
- **Action**: `bash .harness/run-gates.sh` 실행 및 JSON 결과 파싱.

### 4.4 Symphony ↔ Economic SDK
- **Protocol**: Python Library / CLI
- **Action**: `EconomicTracker` 인스턴스를 통한 비용 및 수익 기록.
- **Storage**: `/var/lib/symphony/economic-ledger.jsonl`.

---

## 5. 연동 코드 명세 (Glue Code Specification)

### 5.1 `SymphonyElixir.Agenthub.Client` (Elixir)
Symphony에서 Agenthub API를 호출하기 위한 클라이언트 모듈입니다.

```elixir
defmodule SymphonyElixir.Agenthub.Client do
  @moduledoc """
  Agenthub API와 상호작용하기 위한 HTTP 클라이언트입니다.
  경로: /home/lunark/projects/ai-native-agentic-org/symphony/lib/symphony_elixir/agenthub/client.ex
  """
  use HTTPoison.Base
  require Logger

  defp endpoint, do: Application.get_env(:symphony, :agenthub_url, "http://localhost:8080")
  defp api_key, do: System.get_env("AGENTHUB_API_KEY")

  def push_bundle(agent_id, bundle_path, parent_hash \\ nil) do
    url = "#{endpoint()}/api/git/push"
    body = File.read!(bundle_path)
    headers = [
      {"Authorization", "Bearer #{api_key()}"},
      {"X-Agent-ID", agent_id},
      {"X-Parent-Hash", parent_hash || ""},
      {"Content-Type", "application/octet-stream"}
    ]
    case post(url, body, headers, [timeout: 60_000, recv_timeout: 60_000]) do
      {:ok, %{status_code: 200, body: body}} ->
        {:ok, Jason.decode!(body)}
      {:ok, %{status_code: code, body: body}} ->
        {:error, "HTTP #{code}: #{body}"}
      {:error, reason} ->
        {:error, reason}
    end
  end

  def post_message(channel, author, content) do
    url = "#{endpoint()}/api/channels/#{channel}/posts"
    body = Jason.encode!(%{author: author, content: content})
    headers = [
      {"Authorization", "Bearer #{api_key()}"},
      {"Content-Type", "application/json"}
    ]
    post(url, body, headers)
  end
end
```

### 5.2 `SymphonyElixir.Harness.Runner` (Elixir)
Symphony에서 Harness QA 게이트를 실행하기 위한 래퍼 모듈입니다.

```elixir
defmodule SymphonyElixir.Harness.Runner do
  @moduledoc """
  Harness QA 게이트를 실행하고 결과를 파싱합니다.
  경로: /home/lunark/projects/ai-native-agentic-org/symphony/lib/symphony_elixir/harness/runner.ex
  """
  require Logger

  def run_gates(project_path, harness_path) do
    script_path = Path.join(harness_path, ".harness/run-gates.sh")
    args = ["--project-root", project_path, "--output-format", "json"]
    
    case System.cmd("bash", [script_path | args], cd: project_path, stderr_to_stdout: true) do
      {output, 0} ->
        {:ok, parse_json_output(output)}
      {output, _exit_code} ->
        {:error, parse_json_output(output)}
    end
  end

  defp parse_json_output(output) do
    case Jason.decode(output) do
      {:ok, json} -> json
      {:error, _} -> 
        # JSON 파싱 실패 시 텍스트 기반 파싱 시도
        %{
          "overall" => "fail",
          "output" => output,
          "gates" => []
        }
    end
  end
end
```

### 5.3 `agenthub/internal/harness/runner.go` (Go)
Agenthub 서버 측에서 푸시된 코드에 대해 QA 게이트를 실행하는 로직입니다.

```go
package harness

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"os/exec"
	"time"
)

// Runner는 서버 측 QA 게이트 실행을 담당합니다.
// 경로: /home/lunark/projects/ai-native-agentic-org/agenthub/internal/harness/runner.go
type Runner struct {
	db *sql.DB
}

func NewRunner(db *sql.DB) *Runner {
	return &Runner{db: db}
}

func (r *Runner) RunPostPushGates(commitHash string, projectPath string, harnessScript string) error {
	start := time.Now()
	cmd := exec.Command("bash", harnessScript, "--project-root", projectPath, "--output-format", "json")
	cmd.Dir = projectPath
	output, err := cmd.CombinedOutput()
	
	status := "PASSED"
	if err != nil {
		status = "FAILED"
	}

	// DB에 결과 기록
	query := `
		INSERT INTO gate_results (commit_hash, gate_name, status, output, duration_ms, created_at)
		VALUES (?, ?, ?, ?, ?, ?)`
	
	_, err = r.db.Exec(query, 
		commitHash, 
		"post-push-all", 
		status, 
		string(output), 
		time.Since(start).Milliseconds(), 
		time.Now(),
	)
	
	if status == "FAILED" {
		return fmt.Errorf("post-push gates failed for commit %s", commitHash)
	}
	return nil
}
```

### 5.4 `WORKFLOW.md` 스키마 확장
Symphony의 워크플로우 설정 파일에 Agenthub 및 Harness 설정을 추가합니다.

```yaml
# /home/lunark/projects/ai-native-agentic-org/symphony/WORKFLOW.md

agenthub:
  enabled: true
  server_url: "http://localhost:8080"
  api_key: "${AGENTHUB_API_KEY}"
  push_after_run: true
  channel: "factory-log"

harness:
  enabled: true
  gates_path: "/home/lunark/projects/ai-native-agentic-org/harness-engineering"
  fail_on_gate_failure: true
  required_gates:
    - format
    - tests
    - ast-ratchets

economics:
  enabled: true
  track_llm: true
  min_eval_score: 0.6
  payment_per_priority:
    1: 5.0   # Urgent
    2: 3.0   # High
    3: 2.0   # Medium
    4: 1.0   # Low
```

---

## 6. 데이터 흐름 상세 (Data Flow Walkthrough)

1.  **이슈 탐지 (Issue Detection)**: Symphony의 `LinearPoller`가 Linear API를 호출하여 "Todo" 상태의 이슈를 가져옵니다.
2.  **에이전트 할당 (Agent Allocation)**: Symphony가 이슈 ID를 기반으로 전용 작업 디렉토리를 생성하고 `AgentRunner`를 시작합니다.
3.  **경제적 트래킹 시작 (Cost Tracking)**: `EconomicTracker`가 해당 세션의 비용 추적을 시작합니다.
4.  **RPEQ 실행 (Execution)**:
    - **Research**: 에이전트가 코드베이스를 분석합니다.
    - **Plan**: 구현 계획을 수립합니다.
    - **Execute**: 코드를 수정합니다. LLM 호출 시마다 `track_llm_call`이 호출됩니다.
5.  **로컬 QA 검증 (Local QA)**: Symphony가 `Harness.Runner`를 통해 `run-gates.sh`를 실행합니다.
    - 게이트 실패 시 에이전트에게 피드백을 주고 재시도(최대 3회)합니다.
6.  **코드 푸시 (Code Push)**: 모든 로컬 게이트 통과 시, Symphony가 수정된 코드를 바이너리 번들로 묶어 Agenthub의 `/api/git/push`로 전송합니다.
7.  **서버 QA 검증 (Server QA)**: Agenthub 서버가 푸시된 번들을 풀고 서버 측 QA 게이트를 실행합니다.
8.  **최종 승인 및 보상 (Approval & Reward)**: 서버 게이트 통과 시, `EconomicTracker.add_work_income`을 통해 에이전트에게 수익을 배정하고 Linear 이슈 상태를 "Done"으로 변경합니다.
9.  **로그 기록 (Logging)**: 모든 과정은 Agenthub의 메시지 보드에 실시간으로 기록되어 대시보드에서 확인 가능합니다.

---

## 7. MVP 범위 (MVP Scope)

### 7.1 포함 범위 (In Scope)
- Linear 이슈 폴링 및 상태 업데이트 연동.
- Symphony ↔ Agenthub 간의 코드 번들 전송 API.
- Harness 6-Gate 중 핵심 3종(Format, Tests, AST Ratchets) 연동.
- Economic SDK를 통한 LLM 비용 및 기본 수익 기록.
- 단일 리포지토리 내에서의 자율 수정 및 검증.
- 에이전트 실행 로그의 실시간 메시지 보드 기록.

### 7.2 제외 범위 (Out Scope)
- 다중 리포지토리 간의 의존성 해결.
- 복잡한 Git 컨플릭트 자동 해결 (인간 개입 필요).
- 에이전트 간의 고도화된 협상 및 리소스 경매 시스템.
- 실시간 화상 채팅 기반의 인간-에이전트 협업 UI.
- 커스텀 QA 게이트 정의 UI.

---

## 8. 2주 스프린트 계획 (2-Week Sprint Plan)

### Week 1: 기반 인프라 및 연동 모듈 구축
- **Day 1: Symphony ↔ Agenthub 연동**
    - `SymphonyElixir.Agenthub.Client` 모듈 구현.
    - Agenthub API Key 기반 인증 로직 및 에러 핸들링 추가.
    - 바이너리 번들 전송 성능 최적화 (Chunked Transfer).
- **Day 2: Symphony ↔ Harness 연동**
    - `SymphonyElixir.Harness.Runner` 모듈 구현.
    - `System.cmd`를 통한 서브프로세스 실행 및 실시간 스트리밍 로그 캡처.
    - JSON 결과 파싱 및 게이트별 상태 매핑 로직 완성.
- **Day 3: Agenthub 서버 기능 확장**
    - SQLite 스키마 업데이트 (`gate_results` 테이블 생성).
    - `internal/harness/runner.go` 구현 및 Post-push Hook 연동.
    - 커밋 해시 기반 게이트 결과 조회 API (`GET /api/git/commits/{hash}/gates`) 추가.
- **Day 4: Economic SDK 통합**
    - Symphony `AgentRunner` 내 `EconomicTracker` 데코레이터 적용.
    - LLM 토큰 사용량 계산 및 비용 기록 로직 검증.
    - 품질 점수 기반 수익 배분 알고리즘 구현.
- **Day 5: 워크플로우 설정 및 검증**
    - `WORKFLOW.md` 파서 업데이트 및 환경 변수 치환 기능 추가.
    - 에이전트 실행 전 필수 도구(Git, Python, Go) 설치 여부 확인 로직 추가.
- **Day 6-7: 1주차 통합 테스트 및 버그 수정**
    - 모의(Mock) 에이전트를 이용한 전체 파이프라인 연동 테스트.
    - 네트워크 지연 및 API 타임아웃 시나리오 대응.

### Week 2: 엔드투엔드 워크플로우 완성 및 최적화
- **Day 8: Linear API 심화 연동**
    - Linear GraphQL API를 통한 이슈 상세 정보 및 라벨 기반 필터링 구현.
    - 이슈 상태 전환 자동화 (Todo → In Progress → Done/Failed).
    - 작업 완료 시 Linear 댓글로 작업 요약 및 Agenthub 커밋 링크 자동 게시.
- **Day 9: 에이전트 자가 치유(Self-healing) 로직**
    - QA 게이트 실패 시 에러 로그를 에이전트에게 전달하여 자동 수정 유도.
    - 최대 재시도 횟수 및 지수 백오프(Exponential Backoff) 정책 적용.
- **Day 10: 실시간 모니터링 대시보드 연동**
    - Agenthub 메시지 보드를 활용한 작업 단계별 실시간 브로드캐스팅.
    - 에이전트별 현재 상태 및 작업 이력 조회 API 구현.
- **Day 11: 경제 시스템 고도화**
    - 에이전트별 지갑(Wallet) 시스템 및 일일 정산 보고서 자동 생성.
    - ROI가 낮은 에이전트 자동 비활성화 정책 구현.
- **Day 12: 엔드투엔드 시나리오 테스트**
    - 실제 오픈소스 프로젝트 버그 수정을 대상으로 한 전 과정 자동화 테스트.
    - 다중 에이전트 병렬 실행 시 리소스 경합 및 컨플릭트 테스트.
- **Day 13: 성능 최적화 및 보안 강화**
    - 코드 번들 압축 알고리즘 개선 및 병렬 게이트 실행 도입.
    - 에이전트 샌드박스 권한 최소화 및 민감 정보 필터링.
- **Day 14: 최종 문서화 및 MVP 시연**
    - 운영 매뉴얼 및 API 명세서 최종 업데이트.
    - 주요 이해관계자 대상 엔드투엔드 워크플로우 시연.

---

## 9. 비즈니스 모델 및 수익 구조 (Business Model & Revenue)

### 9.1 수익 모델 (Revenue Model)
1.  **SaaS 라이선스**: 개발자 인당 월 $150 (에이전트 무제한 사용).
2.  **PR 기반 과금**: 성공적으로 머지된 PR당 $2 (소규모 팀 타겟).
3.  **엔터프라이즈**: 온프레미스 설치 및 커스텀 에이전트 학습 지원 (월 $5,000+).
4.  **컴퓨팅 리소스 재판매**: 에이전트 실행에 필요한 GPU 자원 마진 판매.

### 9.2 시장 분석 및 경쟁 우위 (Market Analysis & Competitive Advantage)
- **시장 상황**: GitHub Copilot Workspace, Cursor 등 AI 코딩 도구의 급격한 성장.
- **차별화 포인트**:
    - **품질 우선주의**: 단순 생성이 아닌 6-Gate QA를 통한 "검증된 코드"만 머지.
    - **경제적 투명성**: 모든 작업의 비용과 가치를 실시간으로 추적하여 ROI 증명.
    - **자율적 협업**: Agenthub의 Git DAG를 통해 여러 에이전트가 동시에 안전하게 작업 가능.
    - **플랫폼 독립성**: 특정 IDE나 클라우드에 종속되지 않는 유연한 아키텍처.

### 9.3 수익 추정 (Revenue Projections - Year 1)
- **가정**: 100개 고객사, 평균 20명 개발자.
- **월 매출**: 100개사 * 20명 * $150 = $300,000 (약 4억원).
- **운영 비용**: 
    - LLM API 비용: $60,000 (매출의 20%).
    - 인프라 비용: $30,000 (매출의 10%).
    - 인건비 및 마케팅: $100,000.
- **예상 월 순이익**: $110,000 (약 1.5억원).
- **예상 영업이익률**: 36.7%.

---

## 10. 리스크 및 완화 전략 (Risk & Mitigation)

| 리스크 | 영향도 | 완화 전략 |
| :--- | :--- | :--- |
| **에이전트 할루시네이션** | 높음 | 6-Gate QA를 통해 코드 레벨에서 엄격히 차단. 특히 Equivalence Gate를 통해 의도치 않은 로직 변경 방지. |
| **보안 취약점 노출** | 높음 | Agenthub 내에서 정적 분석(SAST) 게이트 필수 실행. 민감한 API 키나 비밀번호가 코드에 포함되지 않도록 스캔. |
| **LLM 비용 폭주** | 중간 | Economic SDK에 세션별/일별 하드 캡(Hard Cap) 설정. 비정상적으로 많은 토큰을 사용하는 에이전트 자동 차단. |
| **Git 컨플릭트** | 중간 | 에이전트 전용 브랜치 전략 및 선점형 잠금(Locking) 도입. 충돌 발생 시 최신 base로 자동 rebase 시도. |
| **Linear API 속도 제한** | 낮음 | 웹훅(Webhook) 도입을 통한 폴링 부하 감소. 지수 백오프를 적용한 재시도 메커니즘 구현. |
| **인프라 장애** | 중간 | Symphony 및 Agenthub의 고가용성(HA) 구성. 데이터베이스 정기 백업 및 재해 복구(DR) 계획 수립. |

---

## 11. 성공 지표 (Success Metrics)

### 11.1 정량적 지표 (Quantitative Metrics)
- **PR 머지 성공률 (PR Merge Rate)**: 에이전트가 생성한 PR 중 70% 이상이 추가 수정 없이 머지됨.
- **평균 수정 시간 (MTTR)**: 이슈 할당 후 해결까지 평균 15분 이내 완료 (인간 개발자 대비 10배 이상 빠름).
- **결함 밀도 (Defect Density)**: 에이전트 수정 코드에서 발생하는 버그 수가 인간 대비 50% 이하로 유지.
- **경제적 ROI**: 투입된 LLM 비용 대비 절감된 개발 공수(Man-hour) 가치가 3배 이상 달성.
- **에이전트 가동률 (Utilization)**: 전체 업무 시간 중 에이전트가 활발히 작업을 수행하는 비율 80% 이상.

### 11.2 정성적 지표 (Qualitative Metrics)
- **개발자 만족도**: 반복적인 버그 수정 업무에서 해방된 개발자들의 창의적 업무 집중도 향상.
- **코드 일관성**: 모든 에이전트가 동일한 6-Gate 기준을 따름으로써 프로젝트 전체의 코드 품질 상향 평준화.
- **지식 자산화**: 에이전트의 작업 로그와 결정 과정을 통해 프로젝트 히스토리가 자동으로 문서화됨.

---

## 12. 에이전트 협업 모델 (Agent Collaboration Model)

### 12.1 Git DAG 기반 협업
Agenthub는 Git의 Directed Acyclic Graph(DAG) 구조를 활용하여 여러 에이전트가 동시에 작업할 수 있는 환경을 제공합니다.
- **분기(Branching)**: 각 에이전트는 이슈 할당 시 최신 커밋(Leaf)에서 자신만의 분기를 생성합니다.
- **병합(Merging)**: 작업 완료 후 Agenthub에 푸시할 때, 다른 에이전트의 최신 변경 사항을 자동으로 감지하고 병합을 시도합니다.
- **충돌 해결(Conflict Resolution)**: 자동 병합 실패 시, Symphony는 해당 이슈를 "Manual Review" 상태로 변경하고 인간 개발자에게 알림을 보냅니다.

### 12.2 메시지 보드를 통한 상태 공유
에이전트들은 Agenthub의 채널 시스템을 통해 서로의 상태를 공유합니다.
- **`factory-log` 채널**: 전체 워크플로우의 주요 이벤트(이슈 할당, 푸시 성공, 게이트 실패 등)가 기록됩니다.
- **`agent-sync` 채널**: 에이전트들이 현재 수정 중인 파일 목록을 공유하여 잠재적인 충돌을 사전에 방지합니다.

---

## 13. 부록: 설정 스키마 (Appendix: Config Schemas)

### 12.1 Agenthub DB Schema (SQLite)
```sql
CREATE TABLE gate_results (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    commit_hash TEXT NOT NULL,
    gate_name TEXT NOT NULL,
    status TEXT CHECK(status IN ('PASSED', 'FAILED')) NOT NULL,
    output TEXT,
    duration_ms INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE agent_wallets (
    agent_id TEXT PRIMARY KEY,
    balance REAL DEFAULT 0.0,
    total_earned REAL DEFAULT 0.0,
    total_spent REAL DEFAULT 0.0,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 12.2 Economic SDK Ledger Entry (JSONL)
```json
{"timestamp": "2026-03-11T10:00:00Z", "agent_id": "agent-001", "type": "expense", "amount": 0.05, "reason": "gpt-4o-call", "issue_id": "ENG-123"}
{"timestamp": "2026-03-11T10:15:00Z", "agent_id": "agent-001", "type": "income", "amount": 5.00, "reason": "pr-merge-bonus", "issue_id": "ENG-123", "quality_score": 0.95}
```

---

## 14. 결론 (Conclusion)

본 "자율 소프트웨어 팩토리" 프로젝트는 AI 에이전트가 단순한 보조 도구를 넘어 독립적인 개발 주체로 거듭나는 전환점이 될 것입니다. Symphony의 정교한 오케스트레이션, Agenthub의 안정적인 버전 관리, Harness의 엄격한 품질 검증, 그리고 Economic SDK의 투명한 경제 시스템이 결합되어, 지속 가능하고 신뢰할 수 있는 소프트웨어 생산 생태계를 구축할 것입니다.

우리는 이 시스템을 통해 개발자들이 반복적이고 소모적인 작업에서 벗어나 더 가치 있고 창의적인 문제 해결에 집중할 수 있는 미래를 꿈꿉니다. 2주간의 집중적인 스프린트를 통해 MVP를 성공적으로 구축하고, 이를 바탕으로 전 세계 소프트웨어 개발 프로세스의 혁신을 이끌어 나갈 것입니다.

---
**작성자**: Antigravity (Google Deepmind Advanced Agentic Coding Team)
**최종 수정일**: 2026-03-11
**문서 버전**: 1.1.0
