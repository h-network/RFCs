# Hybrid GPU Security Pipeline

A production-grade network security analysis pipeline combining GPU-accelerated ML inference with LLM-powered threat intelligence, orchestrated via Redis-backed agent lifecycle management.

## Architecture

```
                                    NeMo Agent Toolkit (NAT)
                              Redis Orchestration Layer (#1793)
           ┌──────────────────────────────────────────────────────────┐
           │  Task Lifecycle | Abort Control | Metrics | Recovery     │
           └──────────────────────────────────────────────────────────┘
                    │              │              │              │
    ┌───────────────┼──────────────┼──────────────┼──────────────┼───────┐
    │               ▼              ▼              ▼              ▼       │
    │   ┌──────────────┐  ┌──────────────┐  ┌───────────┐  ┌────────┐  │
    │   │  1. CAPTURE   │  │ 2. TRANSFORM │  │ 3. TRIAGE │  │4. INTEL│  │
    │   │              │  │              │  │           │  │        │  │
    │   │  tcpdump     │→ │  mistral:7b  │→ │  Morpheus │→ │qwen3:  │  │
    │   │  raw packets │  │  (4070 Ti)   │  │  XGBoost  │  │  32b   │  │
    │   │              │  │  pcap→JSON   │  │  500K/sec │  │(2x5090)│  │
    │   └──────────────┘  └──────────────┘  └───────────┘  └────────┘  │
    │                                              │              │     │
    │                                              │    ┌─────────┘     │
    │                                              ▼    ▼               │
    │                                      ┌──────────────────┐        │
    │                                      │   Redis State    │        │
    │                                      │                  │        │
    │                                      │  - Flow history  │        │
    │                                      │  - Correlations  │        │
    │                                      │  - Session mem   │        │
    │                                      │  - Metrics       │        │
    │                                      └──────────────────┘        │
    └──────────────────────────────────────────────────────────────────┘
```

## Pipeline Stages

### Stage 1 — Capture
Raw packet capture via `tcpdump`. Continuous or batched.
```bash
tcpdump -n -i any -s 64 -c 200
```

### Stage 2 — Transform
A small LLM (mistral:7b) converts raw packet data into the JSON schema Morpheus expects:
```json
{
  "timestamp": "1617810893485061",
  "src_ip": "10.100.8.98",
  "dest_ip": "10.100.1.237",
  "src_port": "28782",
  "dest_port": "6443",
  "flags": "16",
  "data_len": "54"
}
```
Runs on the lighter GPU (4070 Ti) to keep the 5090s free for heavy inference.

### Stage 3 — Triage
NVIDIA Morpheus + Triton Inference Server runs GPU-accelerated XGBoost classification. Binary first-pass filter at ~500K packets/sec. Flags anomalous traffic (~2-3% of total volume).

Only flagged packets advance to Stage 4.

### Stage 4 — Intelligence
The main LLM (qwen3:32b on dual 5090s) receives:
- The Morpheus classification result
- The original packet data for full context
- Redis session history for cross-flow correlation

Produces actionable threat intelligence:
- What type of anomaly (scan, C2, exfiltration, lateral movement)
- Why it was flagged with supporting evidence
- Correlation with previously seen patterns
- Recommended response

## Orchestration

Built on NeMo Agent Toolkit with Redis-backed middleware ([#1793](https://github.com/NVIDIA/NeMo-Agent-Toolkit/issues/1793)):

- **Task Lifecycle State Machine** — Redis-based state transitions (running/completed/failed/timed_out/aborted) with TTL and crash recovery
- **External Abort** — Redis pub/sub control channels for UI kill buttons and inter-agent coordination
- **Session Continuity** — Conversation history with TTL expiry and size-based rotation
- **Execution Metrics** — Daily counters for tasks, tokens, cost, errors
- **RL Training Loop** — NAT collects execution trajectories to continuously fine-tune models

## Hardware

| Box | GPUs | Role |
|-----|------|------|
| h-oracle | 2x RTX 5090 (64GB total) | Main LLM inference (qwen3:32b), Morpheus/Triton |
| h-titan | RTX 4070 Ti + RTX 3090 | Transform LLM (mistral:7b), overflow inference |

Driver: 570.211.01 | CUDA: 12.8

## Benchmarks (Morpheus Baseline)

Tested on dual RTX 5090 with Morpheus v25.06 + Triton Inference Server:

| Test | Records | Throughput | Detections |
|------|---------|------------|------------|
| NVSMI crypto mining | 1,242 | 33K inf/sec | 188 flagged (15.1%) |
| PCAP anomaly detection | 537,241 | 509K inf/sec | 13,921 flagged (2.6%) |

The LLM stage only processes flagged packets — at 2.6% flag rate, that's ~13K packets instead of 537K going through the expensive LLM path.

## Why This Architecture

| Approach | Speed | Intelligence | Adaptability |
|----------|-------|-------------|-------------|
| Morpheus only | 500K/sec | Binary true/false | Retrain model |
| LLM only | ~50/sec | Full analysis | Change prompt |
| **This pipeline** | **500K/sec triage + deep analysis on flags** | **Full context + correlation** | **Prompt + RL training** |

- **Morpheus** does what it's good at: raw GPU-accelerated throughput on a narrow classification task
- **LLMs** do what they're good at: reasoning, context, explanation, correlation
- **NAT** handles orchestration, training, and continuous improvement
- **Redis** provides state, memory, and auditability across the entire pipeline

## Dependencies

- [NVIDIA Morpheus](https://github.com/nv-morpheus/Morpheus) — GPU-accelerated ML pipeline
- [NVIDIA NeMo Agent Toolkit](https://github.com/NVIDIA/NeMo-Agent-Toolkit) — Agent orchestration + training
- [Triton Inference Server](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/tritonserver) — Model serving
- [Ollama](https://ollama.ai) — Local LLM inference (qwen3:32b, mistral:7b)
- Redis — State, orchestration, session memory

## Status

Early exploration. Morpheus baseline benchmarks completed. NAT Redis orchestration features proposed in [#1793](https://github.com/NVIDIA/NeMo-Agent-Toolkit/issues/1793).

