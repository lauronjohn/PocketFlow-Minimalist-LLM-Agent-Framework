# Formal Technical Report

## Evaluating Classical Software Process Models as Coordination Mechanisms for LLM-Based Software Generation

**Authors:** Duc Minh Ha, Phu Trac Kien, Tho Quan, Anh Nguyen-Duc
**Published:** September 2025 | arXiv:2509.13942 [cs.SE]

---

## Executive Summary

This study investigates whether **classical software process models** — Waterfall, V-Model, and Agile — can serve as **coordination scaffolds** for Large Language Model (LLM)-based multi-agent systems (MAS). The researchers used **MetaGPT** (a role-based multi-agent LLM platform) to execute **11 software projects** across **3 process models × 4 GPT variants**, totaling **132 experimental runs**.

**Key Finding:** Process model choice significantly impacts LLM agent performance. Waterfall is most efficient, V-Model produces the most verbose code, and Agile achieves the highest code quality — but at substantially higher computational cost. The study demonstrates that **human-centric software processes translate meaningfully into AI agent coordination**, and that **process structure matters more than model choice** for determining code quality.

---

## Methodology

### Study Design
- **Design Type:** Quasi-experiment with within-subject replication
- **Experimental Unit:** Project × Process Model × GPT Model
- **Intervention:** Three process models — Waterfall, Agile, V-Model
- **Control Measures:** Uniform prompt templates, Standard Operating Procedures (SOPs), consistent agent role definitions
- **Statistical Test:** One-way ANOVA

### Experiment Settings
- **Framework:** MetaGPT — role-based multi-agent LLM platform
- **Projects:** 11 diverse software projects (Snake Game, Expense Tracker, Tetris, Flappy Bird, QR Generator, etc.)
- **Languages:** JavaScript (6 projects), Python (5 projects)
- **LLMs Tested:** GPT-4o-mini, GPT-4.1-nano, DeepSeek-Chat, DeepSeek-Reasoning
- **Timeline:** January – May 2025

### Evaluation Metrics

| Dimension | Metrics |
|-----------|---------|
| **Size** | Number of files, Lines of Code (LOC), Tokens/LOC ratio |
| **Cost** | Total tokens used, LLM execution time |
| **Quality** | Code smells (SonarQube), AI-detected bugs, Human-detected bugs |

---

## Data Results

### RQ1: How can classical processes be instantiated in LLM-based MAS?

The researchers adapted each process model within MetaGPT:

**Waterfall-like Agent Structure:**
- Strictly sequential pipeline: Project Manager → Designer → Developer → Tester → Deployer
- The **Developer** acts as central orchestrator
- **No backward feedback** loops — stable but rigid
- Communication via structured natural language prompts

**Agile-like Agent Structure:**
- Iterative sprints covering requirements → design → implementation → testing → deployment
- **SprintManager** coordinates sprint context, caching, and versioning
- **Iterative feedback loops** allow adaptation
- Higher coordination overhead

**V-Model Agent Structure:**
- Development phases mirrored with validation phases
- Each stage (requirements, design, implementation) paired with a testing stage (acceptance, integration, unit)
- Enforces **strict traceability** between tests and requirements
- **Limited scope for iteration or negotiation**

### RQ2: Quantitative Comparisons

**Size (Output Volume):**

| Process | Median Files | Median LOC |
|---------|:-----------:|:----------:|
| Agile | 37 | 578 |
| Waterfall | 13 | 475 |
| V-Model | 19 | 934 |

> V-Model produces the **most verbose code** — nearly double Agile's LOC.

**Cost (Efficiency):**

| Process | Median Execution Time | Median Token Usage |
|---------|:-------------------:|:----------------:|
| Agile | 450s | 30,787 |
| Waterfall | **18s** | **13,720** |
| V-Model | 26s | 29,783 |

> Waterfall is **25x faster** than Agile in median execution time.

**Quality:**

| Process | AI Bug Detection | Human Failure Rate | Median Code Smells |
|---------|:---------------:|:-----------------:|:-----------------:|
| Agile | **35.6%** | **40%** | 4 |
| Waterfall | 18.5% | 100% | 2 |
| V-Model | 8.3% | 90% | 1 |

> Agile **significantly outperforms** in code quality metrics.

### LLM Model Comparison

| Model | Execution Time (avg) | Tokens (avg) | AI Bug Detection |
|-------|:------------------:|:------------:|:---------------:|
| GPT-4.1-nano | **90s** | 27,047 | 2.17% |
| GPT-4o-mini | 145s | **19,041** | 8.81% |
| DeepSeek-Chat | 60s+ | 30,912 | 30.39% |
| DeepSeek-Reasoner | 60s+ | 24,597 | **54.62%** |

> DeepSeek-Reasoner is best at bug detection (54.6%) but slow; GPT-4.1-nano is fastest but worst at bug detection.

---

## Challenges

1. **Stochastic LLM Outputs:** Non-deterministic behavior of LLMs introduces variability — mitigated through multiple runs and aggregated metrics
2. **Platform Bias:** MetaGPT's built-in mechanisms may favor certain coordination models (e.g., Waterfall's sequential handoffs match MetaGPT's CodeManager)
3. **Scale Limitations:** Small-to-medium projects — does not reflect industrial-scale systems with evolving requirements and multi-team coordination
4. **Quality Metric Validity:** LOC is a size-oriented proxy, not a direct indicator of productivity or maintainability

---

## Key Takeaways

| If you prioritize… | Choose… | Because… |
|------------------|---------|----------|
| **Speed & efficiency** | Waterfall | 25x faster execution, 2x fewer tokens |
| **Code quality & robustness** | Agile | 35.6% AI bug detection vs 18.5% (Waterfall) |
| **Traceability & completeness** | V-Model | Highest LOC, structured validation |
| **Balanced approach** | Agile + lightweight LLM | Best quality with moderate cost |

**Bottom Line:** Classical software processes **can** and **should** guide LLM-based agent coordination. The key insight from this study: **process structure impacts code quality more than the specific LLM model you choose.** Agile's iterative feedback loops produce the most reliable code, even if they cost more in compute.

---

*Report generated by Bob (OpenClaw AI) following RPW Protocol — April 28, 2026*
