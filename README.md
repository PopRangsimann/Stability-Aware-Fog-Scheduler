# Colony: Stability-Aware Load Balancing with Autonomous Leadership Recovery for Resilient Fog-Assisted IIoT Systems

A research simulation framework implementing a six-phase, security-hardened fog computing architecture for Industrial Internet of Things (IIoT) environments. Colony provides stability-aware task scheduling, autonomous coordinator failover, and post-quantum cryptographic protection across the full data pipeline, from device-level packet encryption through fog-layer processing to secure result retrieval.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
  - [Full Pipeline Demo](#full-pipeline-demo)
  - [Running Individual Phases](#running-individual-phases)
  - [Running the Evaluation](#running-the-evaluation)
- [Project Structure](#project-structure)
- [Configuration](#configuration)
- [The Six Phases](#the-six-phases)
- [Evaluation Framework](#evaluation-framework)
- [Cryptographic Primitives](#cryptographic-primitives)
- [Citation](#citation)
- [License](#license)

---

## Overview

Colony addresses three core challenges in fog-assisted IIoT systems:

1. **Stability-Aware Scheduling** — Workloads are assigned to fog nodes using a composite scoring function that accounts for queue depth, computational capability, communication latency, trust, and a Failure Resilience Index (FRI), smoothed with exponential moving averages to prevent oscillation.

2. **Autonomous Leadership Recovery** — A Master Fog Node (MFN) and Secondary Master Fog Node (SMFN) are elected via stability-penalized coordination scores. State is continuously replicated to cache nodes so that leadership can be restored (Level 1: SMFN promotion, Level 2: cluster re-election) without manual intervention.

3. **End-to-End Security** — Devices are authenticated via PUF challenge-response pairs, sessions are established with Kyber (post-quantum KEM), packets are protected with ChaCha20-Poly1305, and final results are encrypted under CP-ABE access policies for fine-grained authorization.

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Colony Framework                              │
│                                                                      │
│  Phase 1          Phase 2          Phase 3          Phase 4          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────────┐    │
│  │ System   │───▶│ IIoT     │───▶│ Workload │───▶│ Stability-   │    │
│  │ Init     │    │ Protect  │    │ Profiling│    │ Aware LB     │    │
│  │ (PUF,    │    │ (ChaCha, │    │ (Gateway,│    │ (Election,   │    │
│  │  Kyber,  │    │  Kyber   │    │  Batch,  │    │  FRI, Sched, │    │
│  │  CP-ABE) │    │  KDF)    │    │  ω/δ/ρ)  │    │  Replication)│    │
│  └──────────┘    └──────────┘    └──────────┘    └──────┬───────┘    │
│                                                         │            │
│                                                         ▼            │
│                  Phase 6          Phase 5                            │
│                 ┌──────────┐    ┌──────────────┐                     │
│                 │ Recovery │◀───│ Resource     │                     │
│                 │ & Results│    │ Assistance   │                     │
│                 │ (Failover│    │ (Helper Sel, │                     │
│                 │  CP-ABE  │    │  Collab,     │                     │
│                 │  AES-GCM)│    │  Offloading) │                     │
│                 └──────────┘    └──────────────┘                     │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Getting Started

### Prerequisites

- **Python** 3.9 or later
- **pip** package manager

### Installation

```bash
# Clone the repository
git clone https://github.com/PopRangsimann/stability-aware-fog-scheduler.git
cd stability-aware-fog-scheduler

# Install dependencies
pip install -r requirements.txt
```

**Dependencies:**
| Package        | Purpose                                          |
|----------------|--------------------------------------------------|
| `numpy`        | Numerical computation and array operations       |
| `cryptography` | AES-GCM, ChaCha20-Poly1305, and key derivation  |
| `bchlib`       | BCH error correction for PUF response recovery   |

---

## Usage

### Full Pipeline Demo

Run all six phases sequentially to see the complete framework in action:

```bash
python run_all_phases.py
```

This executes Phase I through Phase VI, demonstrating:
- CP-ABE key generation, PUF enrollment, and Kyber key establishment
- ChaCha20-Poly1305 packet protection and validation
- Micro-batch formation and workload profiling (ω, δ, ρ)
- MFN/SMFN coordinator election and FRI-based scheduling
- State replication, overload detection, and helper selection
- Heartbeat-based failure detection, leadership recovery, and secure result retrieval

### Running Individual Phases

Each phase has its own `demo.py` that can be executed independently:

```bash
python -m phase1_system_init.demo
python -m phase2_iiot_protection.demo
python -m phase3_workload_profiling.demo
python -m phase4_load_balancing.demo
python -m phase5_resource_assistance.demo
python -m phase6_recovery_and_results.demo
```

### Running the Evaluation

The evaluation framework runs four experiments comparing Colony against three published baselines:

```bash
cd evaluation
python run_all.py
```

Results (CSV data and plots) are saved to `evaluation/results/`.

---

## Project Structure

```
stability-aware-fog-scheduler/
│
├── config.py                        # Central configuration (all weights, thresholds, ranges)
├── run_all_phases.py                # Full pipeline demo script
├── requirements.txt                 # Python dependencies
│
├── phase1_system_init/              # Phase I: System Initialization
│   ├── attribute_authority.py       #   CP-ABE setup and key generation
│   ├── comm_init.py                 #   Kyber-based secure channel establishment
│   ├── crp_database.py              #   PUF challenge-response pair database
│   ├── fog_node.py                  #   FogNode class definition
│   └── demo.py                      #   Phase I standalone demo
│
├── phase2_iiot_protection/          # Phase II: IIoT Data Protection
│   ├── iiot_device.py               #   IIoTDevice class and data generation
│   ├── key_derivation.py            #   Symmetric key derivation from Kyber secrets
│   ├── packet_protection.py         #   ChaCha20-Poly1305 encryption and MAC
│   └── demo.py                      #   Phase II standalone demo
│
├── phase3_workload_profiling/       # Phase III: Workload Profiling
│   ├── gateway.py                   #   Edge gateway (receive, validate, batch)
│   ├── packet_validation.py         #   Cryptographic packet integrity checks
│   ├── batch_formation.py           #   Micro-batch grouping
│   ├── workload_profiler.py         #   Workload intensity (ω), urgency (δ), priority (ρ)
│   └── demo.py                      #   Phase III standalone demo
│
├── phase4_load_balancing/           # Phase IV: Stability-Aware Load Balancing
│   ├── coordinator_election.py      #   MFN/SMFN election with stability penalty
│   ├── fri.py                       #   Failure Resilience Index computation
│   ├── scheduler.py                 #   EMA-smoothed, FRI-aware task scheduling
│   ├── state_replication.py         #   State replication to SMFN and cache nodes
│   └── demo.py                      #   Phase IV standalone demo
│
├── phase5_resource_assistance/      # Phase V: Collaborative Resource Assistance
│   ├── assistance_request.py        #   Overload detection and assistance requests
│   ├── helper_selection.py          #   Recovery-preserving helper node selection
│   ├── collaborative.py             #   Workload partitioning and migration
│   └── demo.py                      #   Phase V standalone demo
│
├── phase6_recovery_and_results/     # Phase VI: Recovery and Secure Results
│   ├── failure_detection.py         #   Heartbeat monitoring and failure detection
│   ├── leadership_recovery.py       #   Level 1 and Level 2 recovery orchestration
│   ├── result_manager.py            #   AES-GCM result encryption + CP-ABE key wrapping
│   ├── result_retrieval.py          #   Authorized decryption and verification
│   └── demo.py                      #   Phase VI standalone demo
│
├── crypto_primitives/               # Cryptographic Building Blocks
│   ├── aes_gcm.py                   #   AES-256-GCM symmetric encryption
│   ├── chacha20.py                  #   ChaCha20-Poly1305 AEAD encryption
│   ├── cp_abe.py                    #   Ciphertext-Policy Attribute-Based Encryption
│   ├── dilithium.py                 #   Dilithium post-quantum digital signatures
│   ├── kyber.py                     #   Kyber post-quantum key encapsulation
│   └── puf.py                       #   Physical Unclonable Function simulation
│
├── evaluation/                      # Performance Evaluation Suite
│   ├── environment.py               #   Seeded environment generation (nodes, workload, failures)
│   ├── simulator.py                 #   Time-stepped simulation engine
│   ├── metrics.py                   #   Shared metric definitions (latency, deadline ratio, etc.)
│   ├── plotting.py                  #   Config-driven figure generation
│   ├── sim_config.py                #   Evaluation-specific parameters and baselines
│   ├── run_all.py                   #   Entry point for all experiments
│   ├── benchmark_recovery.py        #   Recovery latency benchmarking
│   ├── schemes/                     #   Scheme implementations (proposed + baselines)
│   ├── experiments/                 #   Experiment definitions
│   └── results/                     #   Output CSVs and plots
│
└── Papers/                          # Reference materials
    ├── Colony.pdf                   #   Main paper
    └── Ref[*].pdf                   #   Baseline reference papers
```

---

## Configuration

All tunable parameters live in [`config.py`](config.py). No magic numbers appear inline in algorithm code. Key parameter groups include:

| Parameter Group              | Equations      | Description                                         |
|------------------------------|----------------|-----------------------------------------------------|
| Workload intensity weights   | Eq. 21         | α₁, α₂, α₃ for batch size, volume, complexity      |
| Recovery priority weights    | Eq. 22         | β₁, β₂, β₃ for deadline urgency, intensity, app priority |
| Coordination score weights   | Eq. 24         | α₁–α₅ for capability, memory, latency, trust, readiness |
| Stability penalty            | Eq. 26         | γ for penalizing score variance                     |
| Failure Resilience Index     | Eq. 27         | θ₁, θ₂, θ₃ for trust, readiness, failure rate      |
| Scheduling weights           | Eq. 28         | w₁–w₇ for queue, latency, capability, memory, trust, FRI, priority |
| EMA smoothing factor         | Eq. 29         | η for exponential moving average responsiveness     |
| Recovery state weights       | Eq. 32         | μ₁, μ₂, μ₃ for coordination, FRI, cache freshness  |
| Helper selection weights     | Eq. 36–37      | ψ₁, ψ₂ and λ₁, λ₂, λ₃ for helper scoring          |
| Failure detection threshold  | Eq. 42         | τ_H heartbeat timeout                               |

Weights default to neutral equal values that sum to 1 unless cited otherwise from the paper.

---

## The Six Phases

| Phase | Name | Key Mechanism |
|-------|------|---------------|
| **I**   | System Initialization | CP-ABE attribute authority setup, PUF enrollment, Kyber session establishment |
| **II**  | Hardware-Bound IIoT Protection | Symmetric key derivation, ChaCha20-Poly1305 packet encryption |
| **III** | Scheduling-Aware Workload Profiling | Packet validation, micro-batch formation, workload scoring (ω, δ, ρ) |
| **IV**  | Stability-Aware Load Balancing | MFN/SMFN election, FRI computation, EMA-smoothed scheduling, state replication |
| **V**   | Recovery-Preserving Resource Assistance | Overload detection, helper selection (RC threshold), collaborative workload migration |
| **VI**  | Autonomous Recovery & Secure Results | Heartbeat failure detection, L1/L2 leadership recovery, AES-GCM + CP-ABE result protection |

---

## Evaluation Framework

The evaluation compares Colony against three published baselines across four experiments:

| Experiment | Independent Variable | Primary Metric | Focus |
|------------|---------------------|----------------|-------|
| 1 | Workload arrival rate (10²–10⁴ batches) | Avg completion latency | Stability-aware scheduling + FRI |
| 2 | Number of fog nodes | Leadership recovery latency | Replicated state + precomputed RS |
| 3 | Workload arrival rate to saturation | Deadline-satisfaction ratio | Recovery-preserving helper selection |
| 4 | Joint overload + coordinator failures | Successful completion ratio | Full framework resilience |

**Baselines:**
- Ala'anzy et al.
- Jasim and Al-Raweshidy
- Chen et al.

All schemes run on identical seeded environments to ensure fair comparison. Metrics are computed by shared functions applied uniformly across all schemes.

---

## Cryptographic Primitives

| Primitive | Standard | Usage |
|-----------|----------|-------|
| **Kyber** | NIST PQC Round 3 | Post-quantum key encapsulation for session establishment |
| **Dilithium** | NIST PQC Round 3 | Post-quantum digital signatures |
| **CP-ABE** | Ciphertext-Policy ABE | Fine-grained access control on computation results |
| **ChaCha20-Poly1305** | RFC 8439 | AEAD encryption for IIoT data packets |
| **AES-256-GCM** | NIST SP 800-38D | Symmetric encryption for final result protection |
| **PUF** | Physical Unclonable Function | Hardware-based device identity and authentication |

---

## Citation

If you use this framework in your research, please cite:

```bibtex
@article{colony2026,
  title   = {Stability-Aware Load Balancing with Autonomous Leadership Recovery
             for Resilient Fog-Assisted IIoT Systems},
  author  = {},
  year    = {2026},
  journal = {},
}
```

---

## License

This project is part of an academic research effort. Please see the repository for licensing details.
