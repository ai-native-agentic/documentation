# UC3: 자율 ML 연구소 (Autonomous ML Research Lab) Specification Document

**Version:** 1.0.0  
**Status:** Implementation-Ready  
**Date:** 2026-03-11  
**Owner:** Antigravity (Google DeepMind Agentic Coding Team)

---

## 1. Overview & Vision

### 1.1 Executive Summary
The **Autonomous ML Research Lab (UC3)** is a self-evolving, multi-agent ecosystem designed to automate the entire lifecycle of machine learning research—from hyperparameter exploration and architectural optimization to on-device deployment and economic validation. By leveraging a decentralized Git-based Directed Acyclic Graph (DAG) for experiment tracking and a swarm of autonomous agents, UC3 eliminates the manual bottleneck in ML development.

### 1.2 Problem Statement
Traditional ML research is slow, expensive, and often disconnected from real-world economic utility. Researchers spend 80% of their time on mechanical tasks like hyperparameter tuning and experiment tracking. Furthermore, models optimized for accuracy often fail on-device due to latency or memory constraints, and their economic ROI is rarely measured before deployment.

### 1.3 The UC3 Solution
UC3 creates a "Perpetual Research Machine" that:
1.  **Automates Exploration:** Swarms of agents explore the hyperparameter space 24/7.
2.  **Self-Optimizes:** Promising models are automatically refined using DSPy and PEFT.
3.  **Validates Everywhere:** Models are benchmarked on 10+ on-device backends.
4.  **Grounds in Economics:** Every model is audited for economic ROI via GDPVal.

---

## 2. Architecture Diagram

```text
+---------------------------------------------------------------------------------------+
|                                     UC3 ORCHESTRATION LAYER                           |
|  +-----------------------+       +------------------------+      +-----------------+  |
|  |   ResearchAgent Swarm | <---> | agenthub (Git DAG/Msg) | <--> | Experiment      |  |
|  |   (5-10 Parallel)     |       | #experiments channel   |      | Orchestrator    |  |
+--+-----------+-----------+-------+-----------+------------+------+--------+--------+--+
               |                               |                            |
               v                               v                            v
+--------------+-----------+       +-----------+------------+      +--------+--------+
|      autoresearch        |       | self-learning-agents   |      | ModelExporter   |
| (5-min Training Runs)    | ----> | (DSPy + PEFT Opt)      | ---->| (ONNX/TFLite)   |
+--------------+-----------+       +-----------+------------+------+--------+--------+
               |                               |                            |
               |                               v                            v
               |                   +-----------+------------+      +--------+--------+
               |                   | ClawWork (GDPVal)      | <--- | ondevice-ai-lab |
               +-----------------> | (Economic Validation)  |      | (10 Backends)   |
                                   +------------------------+      +-----------------+
```

---

## 3. Component Responsibilities

### 3.1 autoresearch
- **Location:** `/home/lunark/projects/ai-native-agentic-org/autoresearch/`
- **Role:** The core training engine for Small Language Models (SLMs).
- **Constraints:** Modifiable `train.py` only.
- **Inputs:** Hyperparameters (DEPTH, ASPECT_RATIO, WINDOW_PATTERN, BATCH_SIZE, LR, WEIGHT_DECAY, ADAM_BETAS).
- **Outputs:** `results.tsv` containing `val_bpb` (Bits Per Byte), `memory_gb`, `status`, and `description`.
- **Fixed Params:** MAX_SEQ_LEN=2048, TIME_BUDGET=300s, VAL_SHARD=6542, VOCAB_SIZE=8192.
- **Architecture:** GPT with ResFormer (value embeddings), Flash Attention 3, ReLU² MLP.

### 3.2 agenthub
- **Location:** `/home/lunark/projects/ai-native-agentic-org/agenthub/`
- **Role:** The decentralized coordination and versioning backbone.
- **Git DAG:** Stores every experiment as a unique commit. No merges allowed to maintain a pure history of exploration.
- **Message Board:** `#experiments` channel for agents to announce starts, failures, and breakthroughs.
- **Discovery:** `ah leaves` provides the current frontier of research, allowing agents to branch from the best known states.

### 3.3 ai-native-self-learning-agents
- **Location:** `/home/lunark/projects/ai-native-agentic-org/ai-native-self-learning-agents/`
- **Role:** Post-training optimization and refinement.
- **DSPy Integration:** Uses `auto_compiler.py` for BootstrapFewShot recompilation to improve prompt-level performance.
- **PEFT:** Uses `auto_finetuner.py` for LoRA/QLoRA fine-tuning on failure cases identified during inference.
- **Promotion:** `ModelVersionManager` auto-promotes models that show >2% improvement in `val_bpb`.

### 3.4 lunark-ondevice-ai-lab
- **Location:** `/home/lunark/projects/ai-native-agentic-org/lunark-ondevice-ai-lab/`
- **Role:** Cross-platform performance validation.
- **Backends:** ONNX, TFLite, CoreML, MLC-LLM, llama.cpp, MediaPipe, OpenVINO, ExecuTorch, NCNN, MNN.
- **Metrics:** Latency (ms), Memory (MB), Throughput (tokens/sec), and Device Compatibility.
- **Scenarios:** 36 real-world on-device scenarios (e.g., offline translation, local RAG, real-time STT).

### 3.5 ClawWork
- **Location:** `/home/lunark/projects/ai-native-agentic-org/ClawWork/`
- **Role:** Economic utility verification.
- **GDPVal:** Runs 220 tasks across 44 professions to measure real-world productivity.
- **ROI Calculation:** `task_value - (compute_cost + API_cost)`.
- **Final Gate:** Only models with positive economic ROI are marked as "Production Ready".

---

## 4. Experiment Lifecycle (Step-by-Step)

1.  **Initialization:** `ExperimentOrchestrator` queries `agenthub` for the current best `val_bpb` and the latest "leaf" commits.
2.  **Proposal:** A `ResearchAgent` analyzes previous results and proposes a new hyperparameter set. It uses a mix of Bayesian optimization (for continuous params like LR) and Evolutionary strategies (for discrete params like DEPTH).
3.  **Execution:** The `ResearchAgent` modifies `autoresearch/train.py` (or passes arguments) and executes it. The run is capped at 300 seconds.
4.  **Commit:** Upon completion, the agent parses `results.tsv`, creates a Git bundle, and pushes it to `agenthub` via `ah push`.
5.  **Coordination:** The agent posts a summary (BPB, params, hash) to the `#experiments` channel.
6.  **Optimization:** If the new `val_bpb` is a global best, the `SelfLearningEngine` is triggered. It applies DSPy optimizations to the model's prompt/logic layer.
7.  **Benchmarking:** The optimized model is exported to ONNX and sent to `ondevice-ai-lab`. It is tested against all 10 backends to find the best deployment target.
8.  **Economic Audit:** The model is loaded into `ClawWork` to perform GDPVal tasks. This step verifies if the architectural improvements translate to actual task success.
9.  **Reporting:** A final report is generated, linking the Git hash to its performance across all tiers (BPB, Latency, ROI).

---

## 5. Multi-Agent Research Coordination

### 5.1 Swarm Intelligence via agenthub
UC3 employs a "Swarm Intelligence" approach where agents coordinate without a central master:
-   **Parallel Exploration:** 5-10 agents run concurrently. Each agent specializes in a specific region (e.g., "Depth Specialist", "LR Explorer").
-   **Conflict Avoidance:** Before starting a run, an agent posts "STARTING: {params_hash}" to `#experiments`. Other agents check this to avoid redundant work.
-   **DAG Frontier:** Agents always branch from the "leaf" commit with the lowest `val_bpb`. This ensures the swarm always moves towards the global optimum.
-   **Failure Recovery:** If an agent crashes, the `ExperimentOrchestrator` detects the lack of a "COMPLETED" message and re-assigns the hyperparameter region.

### 5.2 Agent Specialization Table
| Agent ID | Specialization | Strategy |
| :--- | :--- | :--- |
| `agent-depth` | Model depth (layers) | Grid search DEPTH 4-12 |
| `agent-width` | Model width (aspect ratio) | Random search ASPECT_RATIO 0.5-2.0 |
| `agent-lr` | Learning rate | Log-uniform sampling LR 1e-4 to 1e-2 |
| `agent-optim` | Optimizer config | Bayesian optimization of ADAM_BETAS, WEIGHT_DECAY |
| `agent-attn` | Attention patterns | Evolutionary search of WINDOW_PATTERN |
| `agent-hybrid` | Multi-param exploration | Evolutionary from top-5 experiments |

---

## 6. Integration Points (API Specifications)

### 6.1 agenthub API
- `POST /api/git/push`: 
    - Payload: `{ "bundle": binary, "agent_id": string, "message": string }`
    - Response: `{ "hash": string, "status": "success" }`
- `GET /api/git/leaves`: 
    - Response: `[ { "hash": string, "val_bpb": float, "agent_id": string }, ... ]`
- `POST /api/channels/#experiments/posts`: 
    - Payload: `{ "content": string, "metadata": dict }`

### 6.2 self-learning-agents API
- `InferenceCollector.log_inference(result)`: 
    - Logs input, output, and ground truth (if available).
- `ModelVersionManager.register_model(version_id, path, metric)`: 
    - Tracks model lineage and performance.
- `SelfLearningEngine.run_learning_cycle(strategy)`:
    - Triggers DSPy + PEFT pipeline.

### 6.3 ondevice-ai-lab Integration
- **Path:** `/home/lunark/projects/ai-native-agentic-org/lunark-ondevice-ai-lab/core/benchmark_runner.py`
- **Input:** ONNX model path, target backends list.
- **Output:** JSON report with latency/memory per backend.

---

## 7. Glue Code Specification

### 7.1 ExperimentOrchestrator
Location: `/home/lunark/projects/ai-native-agentic-org/uc3_glue/orchestrator.py`

```python
import subprocess
import json
import requests
import time
import os
from typing import Dict, List

class ExperimentOrchestrator:
    def __init__(self):
        self.agenthub_url = "http://localhost:8080"
        self.channel = "#experiments"
        self.best_bpb = float('inf')

    def run_experiment(self, hyperparams: Dict) -> Dict:
        """Executes autoresearch/train.py with injected hyperparams."""
        # Logic to modify train.py or pass env vars
        cmd = ["python3", "/home/lunark/projects/ai-native-agentic-org/autoresearch/train.py"]
        env = {**os.environ, **{k: str(v) for k, v in hyperparams.items()}}
        try:
            result = subprocess.run(cmd, env=env, capture_output=True, text=True, timeout=360)
            if result.returncode != 0:
                print(f"Training failed: {result.stderr}")
                return {"status": "failed"}
            return self.parse_results_tsv()
        except subprocess.TimeoutExpired:
            print("Training timed out.")
            return {"status": "timeout"}

    def parse_results_tsv(self) -> Dict:
        """Parses the results.tsv file from autoresearch."""
        path = "/home/lunark/projects/ai-native-agentic-org/autoresearch/results.tsv"
        if not os.path.exists(path): return {"status": "failed"}
        with open(path, 'r') as f:
            lines = f.readlines()
            if len(lines) < 2: return {"status": "failed"}
            last_line = lines[-1].strip().split('\t')
            return {
                "commit": last_line[0],
                "val_bpb": float(last_line[1]),
                "memory_gb": float(last_line[2]),
                "status": last_line[3],
                "description": last_line[4]
            }

    def push_to_agenthub(self, result: Dict, params: Dict) -> str:
        """Commits and pushes the experiment to the agenthub DAG."""
        msg = f"Exp: bpb={result['val_bpb']} params={json.dumps(params)}"
        subprocess.run(["ah", "push", "-m", msg])
        res = requests.get(f"{self.agenthub_url}/api/git/leaves")
        if res.status_code == 200:
            return res.json()[-1]['hash']
        return "unknown_hash"

    def monitor_loop(self):
        """Main loop to monitor agenthub and trigger downstream pipelines."""
        while True:
            try:
                response = requests.get(f"{self.agenthub_url}/api/git/leaves")
                if response.status_code == 200:
                    leaves = response.json()
                    if leaves:
                        best_leaf = min(leaves, key=lambda x: x.get('val_bpb', float('inf')))
                        if best_leaf['val_bpb'] < self.best_bpb:
                            self.best_bpb = best_leaf['val_bpb']
                            self.trigger_optimization_pipeline(best_leaf)
            except Exception as e:
                print(f"Error in monitor loop: {e}")
            time.sleep(60)

    def trigger_optimization_pipeline(self, leaf: Dict):
        """Triggers self-learning, export, and benchmarking."""
        print(f"New Best Found! BPB: {leaf['val_bpb']}. Triggering optimization...")
        # 1. Self-Learning (DSPy + PEFT)
        # 2. Export to ONNX
        # 3. Benchmark on-device
        # 4. ClawWork Validation
```

### 7.2 ResearchAgent
Location: `/home/lunark/projects/ai-native-agentic-org/uc3_glue/research_agent.py`

```python
import random
import time
from typing import Dict

class ResearchAgent:
    def __init__(self, agent_id: str, strategy: str):
        self.id = agent_id
        self.strategy = strategy
        self.orchestrator = ExperimentOrchestrator()

    def propose_hyperparams(self) -> Dict:
        """Proposes next hyperparams based on the assigned strategy."""
        if self.strategy == "lr_explorer":
            return {"LR": 10 ** random.uniform(-4, -2), "DEPTH": 8}
        elif self.strategy == "depth_specialist":
            return {"DEPTH": random.randint(4, 12), "LR": 0.001}
        elif self.strategy == "width_specialist":
            return {"ASPECT_RATIO": random.uniform(0.5, 2.0), "DEPTH": 8}
        elif self.strategy == "batch_explorer":
            return {"BATCH_SIZE": random.choice([8, 16, 32, 64]), "DEPTH": 8}
        return {"LR": 0.001, "DEPTH": 8}

    def run_cycle(self):
        """Single iteration of the research loop."""
        params = self.propose_hyperparams()
        print(f"[{self.id}] Starting experiment: {params}")
        result = self.orchestrator.run_experiment(params)
        if result.get("status") == "success":
            h = self.orchestrator.push_to_agenthub(result, params)
            print(f"[{self.id}] Committed experiment: {h}")
            self.post_to_message_board(result, h)
        else:
            print(f"[{self.id}] Experiment failed.")

    def post_to_message_board(self, result: Dict, h: str):
        """Posts result to agenthub message board."""
        url = f"{self.orchestrator.agenthub_url}/api/channels/%23experiments/posts"
        payload = {
            "content": f"Agent {self.id} found BPB {result['val_bpb']} at hash {h}",
            "metadata": result
        }
        requests.post(url, json=payload)
```

### 7.3 ModelExporter & Benchmarker
Location: `/home/lunark/projects/ai-native-agentic-org/uc3_glue/exporter.py`

```python
import torch
import subprocess
import json

class ModelExporter:
    @staticmethod
    def export_to_onnx(model_path: str, output_path: str):
        """Converts PyTorch checkpoint to ONNX format."""
        # Load model from autoresearch
        # model = GPT(...)
        # model.load_state_dict(torch.load(model_path))
        # torch.onnx.export(...)
        print(f"Exporting {model_path} to {output_path}...")

    @staticmethod
    def run_on_device_benchmarks(onnx_path: str):
        """Runs the benchmark suite in lunark-ondevice-ai-lab."""
        cmd = [
            "python3", 
            "/home/lunark/projects/ai-native-agentic-org/lunark-ondevice-ai-lab/core/benchmark.py", 
            "--model", onnx_path
        ]
        result = subprocess.run(cmd, capture_output=True, text=True)
        if result.returncode == 0:
            return json.loads(result.stdout)
        return {"error": result.stderr}
```

---

## 8. Data Flows & State Management

### 8.1 The "Single Source of Truth"
The `agenthub` Git DAG is the only source of truth for experiment state. Every `ResearchAgent` must fetch the latest leaves before proposing a new experiment. This prevents "state drift" where agents work on outdated architectures.

### 8.2 Data Flow Diagram
1.  **Agent** → `ah fetch` (Get latest state)
2.  **Agent** → `train.py` (Execute training)
3.  **Agent** → `ah push` (Save result to DAG)
4.  **Orchestrator** → `ah leaves` (Monitor for breakthroughs)
5.  **Orchestrator** → `self-learning-agents` (Optimize best model)
6.  **Orchestrator** → `ondevice-ai-lab` (Benchmark optimized model)
7.  **Orchestrator** → `ClawWork` (Validate economic ROI)

---

## 9. Economic Tracking Integration

UC3 uses the `ai-native-agentic-economic-sdk` to maintain a real-time ledger of research costs and potential revenue.

### 9.1 Cost/Revenue Model
-   **Training Cost:** $0.50 per 5-minute run (GPU + Power).
-   **Optimization Cost:** $1.00 per DSPy/PEFT cycle.
-   **Benchmarking Cost:** $0.10 per backend test.
-   **Potential Revenue:** Calculated by `ClawWork` based on task success (e.g., $5.00 per successful complex task).

### 9.2 Ledger Example
```json
{
  "timestamp": "2026-03-11T10:00:00Z",
  "agent_id": "agent-depth-1",
  "action": "training_run",
  "debit": 0.50,
  "credit": 0.00,
  "balance": -150.50,
  "metadata": { "commit": "abc123hash", "val_bpb": 1.45 }
}
```

---

## 10. MVP Scope (Weeks 1-2)

### 10.1 Week 1: The Foundation
-   **Goal:** Establish the basic research loop with 3 agents.
-   **Deliverables:**
    -   `ExperimentOrchestrator` core implementation.
    -   `ResearchAgent` with random and grid search strategies.
    -   Successful `ah push` of 50+ experiments to `agenthub`.
    -   Automated `results.tsv` parsing.

### 10.2 Week 2: Optimization & Validation
-   **Goal:** Connect the downstream optimization and validation tiers.
-   **Deliverables:**
    -   Trigger `self-learning-agents` on the top 5% of experiments.
    -   Automated ONNX export for best-in-class models.
    -   Latency benchmarking on 3 backends (ONNX, TFLite, llama.cpp).
    -   One full `ClawWork` GDPVal run for the best optimized model.
    -   Economic ledger integration for all MVP operations.

---

## 11. 2-Week Sprint Plan

### Sprint 1: Core Research Loop (Days 1-7)
- **Day 1:** Setup `uc3_glue` directory and environment variables.
- **Day 2:** Implement `ExperimentOrchestrator.run_experiment` and `parse_results_tsv`.
- **Day 3:** Integrate `agenthub` CLI (`ah`) for pushing and fetching.
- **Day 4:** Implement `ResearchAgent` with `lr_explorer` and `depth_specialist` strategies.
- **Day 5:** Run first multi-agent swarm (3 agents) for 8 hours.
- **Day 6:** Build a simple CLI dashboard to view the `agenthub` leaderboard.
- **Day 7:** QA Gate: Verify Git DAG integrity and ensure no duplicate experiments.

### Sprint 2: The Optimization Pipeline (Days 8-14)
- **Day 8:** Connect `self-learning-agents`. Implement the trigger mechanism in `Orchestrator`.
- **Day 9:** Implement `ModelExporter` (PyTorch -> ONNX).
- **Day 10:** Integrate `ondevice-ai-lab`. Run benchmarks on the top 5 models.
- **Day 11:** Connect `ClawWork`. Implement the `EconomicValidationBridge`.
- **Day 12:** Run full E2E pipeline: Research → Optimize → Benchmark → ROI Audit.
- **Day 13:** Implement real-time economic ledger tracking.
- **Day 14:** Final stress test (10 agents) and documentation finalization.

---

## 12. Business Model & Revenue Projections

### 12.1 Revenue Streams
1.  **Research-as-a-Service (RaaS):** Enterprise teams pay $2,000/month to run dedicated swarms on their proprietary datasets.
2.  **Model Marketplace:** High-performance SLMs (optimized for specific backends) sold on Hugging Face. Price: $500 - $5,000 per model.
3.  **Compute Credits:** Pay-per-experiment model for independent researchers ($0.50/run).
4.  **Consulting:** $10,000/week for custom research pipeline setup and integration.

### 12.2 Revenue Projections (Year 1)
- **Q1 (Beta):** $15,000 (5 early-adopter teams).
- **Q2 (Launch):** $60,000 (Marketplace launch + 10 RaaS teams).
- **Q3 (Scale):** $150,000 (25 RaaS teams + enterprise licensing).
- **Q4 (Expansion):** $300,000 (First major enterprise deal + 50 RaaS teams).
- **Total Year 1:** ~$525,000.

---

## 13. Risk & Mitigation

### 13.1 Technical Risks
| Risk | Impact | Mitigation |
| :--- | :--- | :--- |
| **Hyperparameter Divergence** | High | Implement early-stopping if `val_bpb` doesn't improve in first 60s. |
| **agenthub DAG Bloat** | Medium | Periodic pruning of non-leaf commits that are >20% worse than SOTA. |
| **On-device Backend Failure** | Low | Fallback to "Mock Backend" in `ondevice-ai-lab` to ensure pipeline continuity. |
| **Compute Cost Overrun** | High | Hard budget caps in `economic-sdk` that kill all agents if limit reached. |
| **Model Overfitting** | Medium | Use `VAL_SHARD=6542` strictly and add a secondary "Hold-out" shard for final validation. |

### 13.2 Operational Risks
- **Resource Contention:** Multiple agents competing for the same GPU. *Mitigation:* Implement a simple semaphore-based lock in the `Orchestrator`.
- **Data Corruption:** Interrupted `ah push` operations. *Mitigation:* Use atomic file writes and checksum verification for Git bundles.

---

## 14. Success Metrics

-   **Primary Metric:** `val_bpb` reduction over time (Target: < 1.5 for 8K vocab).
-   **Efficiency:** Number of experiments per hour (Target: 12 runs/agent/hour).
-   **Optimization Gain:** % improvement from `self-learning-agents` (Target: > 5%).
-   **Economic ROI:** `ClawWork` GDPVal score vs. compute cost (Target: > 2.0x).
-   **Deployment Readiness:** % of models passing all 10 `ondevice-ai-lab` backends (Target: 90%).

---

## 15. Future Roadmap (V2 & V3)

### 15.1 V2: Advanced Architectural Search
-   **Neural Architecture Search (NAS):** Agents will be able to propose new layer types and connection patterns, not just hyperparameters.
-   **Multi-Objective Optimization:** Simultaneously optimizing for BPB, Latency, and Memory using NSGA-II.

### 15.2 V3: Fully Autonomous Lab
-   **Self-Provisioning:** Agents can spin up their own GPU instances based on budget availability.
-   **Automated Paper Writing:** Agents generate LaTeX reports of their findings for human review.

---

## 16. Glossary of Terms
- **BPB (Bits Per Byte):** A metric for language model performance. Lower is better.
- **DAG (Directed Acyclic Graph):** A data structure used by `agenthub` to track experiment history.
- **DSPy:** A framework for optimizing language model prompts and weights.
- **PEFT (Parameter-Efficient Fine-Tuning):** Techniques like LoRA that allow fine-tuning with minimal compute.
- **GDPVal:** A benchmark for measuring the economic value of AI agents.

---

## 17. Appendix: Hardware Requirements
- **Minimum:** 1x NVIDIA RTX 3090 (24GB VRAM), 32GB RAM, 500GB SSD.
- **Recommended:** 4x NVIDIA A100 (80GB VRAM), 256GB RAM, 2TB NVMe SSD.
- **On-Device Testing:** Raspberry Pi 5, Jetson Orin Nano, and various mobile SoC simulators.

---

## 18. Detailed API Response Examples

### 18.1 agenthub `GET /api/git/leaves`
```json
[
  {
    "hash": "a1b2c3d4e5f6",
    "val_bpb": 1.42,
    "agent_id": "agent-lr-1",
    "timestamp": "2026-03-11T10:05:00Z",
    "message": "Exp: bpb=1.42 params={\"LR\": 0.0005, \"DEPTH\": 8}"
  },
  {
    "hash": "f6e5d4c3b2a1",
    "val_bpb": 1.38,
    "agent_id": "agent-depth-2",
    "timestamp": "2026-03-11T10:12:00Z",
    "message": "Exp: bpb=1.38 params={\"LR\": 0.001, \"DEPTH\": 10}"
  }
]
```

### 18.2 ondevice-ai-lab Benchmark Report
```json
{
  "model_id": "autoresearch-f6e5d4c3",
  "timestamp": "2026-03-11T10:20:00Z",
  "results": [
    {
      "backend": "onnx",
      "latency_ms": 15.4,
      "memory_mb": 128,
      "throughput_tps": 65.2
    },
    {
      "backend": "tflite",
      "latency_ms": 18.2,
      "memory_mb": 120,
      "throughput_tps": 54.9
    }
  ]
}
```

---
**End of Specification Document**
