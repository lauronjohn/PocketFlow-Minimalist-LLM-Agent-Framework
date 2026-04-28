# PocketFlow — Software Architecture Recovery

> **Course:** Modern Software Architecture — Wuhan University, 2026  
> **Instructor:** Prof. Peng Liang  
> **Target System:** [PocketFlow](https://github.com/The-Pocket/PocketFlow)  
> **Reference:** [DESOSA 2019](https://se.ewi.tudelft.nl/desosa2019/)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Stakeholder Analysis](#2-stakeholder-analysis)
3. [Context View](#3-context-view)
4. [Development View](#4-development-view)
5. [Process View](#5-process-view)
6. [Design Patterns](#6-design-patterns)
7. [Quality Attribute Scenarios](#7-quality-attribute-scenarios)
8. [Technical Debt Analysis](#8-technical-debt-analysis)
9. [Conclusion](#9-conclusion)
10. [Weekly Progress Log](#10-weekly-progress-log)
11. [References](#11-references)

---

## 1. Introduction

### 1.1 What is PocketFlow?

<img width="2558" height="859" alt="image" src="https://github.com/user-attachments/assets/acdf2eda-cb69-4206-9d31-c3efe2a94da9" />


PocketFlow is a **100-line minimalist LLM (Large Language Model) orchestration framework** written in Python. It was created as a direct counter-argument to frameworks like LangChain and CrewAI, which the author argues are over-engineered for the problem they solve. Its central thesis is that the entire core abstraction needed for LLM application development — Nodes, Flows, and a Shared Store — can be expressed in exactly 100 lines of code, with zero external dependencies and zero vendor lock-in.

Despite its minimal size, PocketFlow is expressive enough to implement the full spectrum of modern LLM application patterns, including Agents, Workflows, Retrieval-Augmented Generation (RAG), MapReduce, Multi-Agent systems, and Structured Output. It models LLM workflows as a **Graph + Shared Store**, where:

- **Node** handles a single (LLM) task via a `prep → exec → post` lifecycle
- **Flow** connects Nodes through labeled edges called Actions
- **Shared Store** is a shared dictionary enabling communication between Nodes within a Flow

The framework is available as a Python package (`pip install pocketflow`) or by directly copying the 100-line source. It has since been ported to TypeScript, Java, C++, Go, Rust, and PHP by the community. As of 2026, the repository has over 10,000 GitHub stars, 1,100+ forks, and 200+ dependent projects.

The table below compares PocketFlow to other popular LLM frameworks:

| Framework | Abstraction | Lines | Size |
|---|---|---|---|
| LangChain | Agent, Chain | 405K | +166 MB |
| CrewAI | Agent, Chain | 18K | +173 MB |
| LangGraph | Agent, Graph | 37K | +51 MB |
| AutoGen | Agent | 7K | +26 MB |
| **PocketFlow** | **Graph** | **100** | **+56 KB** |

### 1.2 Why PocketFlow is Architecturally Interesting

PocketFlow is architecturally interesting not despite its minimalism, but because of it. It forces a fundamental question: **what is the irreducible core abstraction for LLM orchestration?** The project's answer — a nested directed graph with a shared store — is a deliberate and defensible architectural decision that warrants careful study.

Several properties make PocketFlow a rich subject for architecture recovery:

- **Radical minimalism as a design principle.** The 100-line constraint is not a limitation but an explicit architectural goal, making every line of code an intentional decision.
- **Intentional exclusion as architecture.** PocketFlow explicitly does not provide LLM vendor wrappers or app-specific utilities. This boundary decision — what the framework refuses to do — is as architecturally significant as what it provides.
- **Cross-language portability.** The same abstraction has been faithfully reproduced in six programming languages, which serves as evidence of the abstraction's structural soundness and language-independence.
- **Agentic coding philosophy.** PocketFlow is designed to be intuitive enough for AI agents themselves to build LLM applications on top of it, representing a forward-looking architectural stance on human-AI collaborative development.

### 1.3 Scope of This Document

This document recovers and describes the software architecture of PocketFlow as of 2026. The analysis covers the Python core package (`pocketflow/__init__.py`), the cookbook of 26+ example applications, the multi-language port ecosystem (TypeScript, Java, C++, Go, Rust, and PHP), and the project's community and documentation infrastructure.

The analysis does **not** cover LLM provider SDKs (OpenAI, Anthropic, etc.), vector databases, or third-party applications built on top of PocketFlow — these are external to the system boundary.

The document is organized around the 4+1 view model: a stakeholder and context view, a development view, a process view, a design pattern analysis, and a quality attributes and technical debt assessment. Together, these views argue that PocketFlow's architecture is defined as much by what it **deliberately excludes** as by what it includes — and that this boundary decision is its central architectural contribution.

---

## 2. Stakeholder Analysis

### 2.1 Stakeholder Identification

> _To be completed — Week 2_

| Stakeholder | Type | Role | Concerns |
|---|---|---|---|
| | | | |

### 2.2 Power / Interest Grid

> _To be completed — Week 2_

### 2.3 Key Architectural Decisions Driven by Stakeholders

> _To be completed — Week 2_

---

## 3. Context View

### 3.1 System Scope

> _To be completed — Week 2_

### 3.2 External Dependencies

> _To be completed — Week 2_

| Dependency Type | Where It Lives | Examples |
|---|---|---|
| | | |

---

## 4. Development View

### 4.1 Repository Structure

> _To be completed — Week 3_

### 4.2 Module Structure

> _To be completed — Week 3_

### 4.3 Core Abstraction

> _To be completed — Week 3_

### 4.4 Node Lifecycle

> _To be completed — Week 3_

---

## 5. Process View

### 5.1 Synchronous Execution

> _To be completed — Week 4_

### 5.2 Asynchronous Execution

> _To be completed — Week 4_

### 5.3 Batch Execution

> _To be completed — Week 4_

---

## 6. Design Patterns

### 6.1 Agent Pattern

> _To be completed — Week 5_

### 6.2 Workflow Pattern

> _To be completed — Week 5_

### 6.3 RAG Pattern

> _To be completed — Week 5_

### 6.4 MapReduce Pattern

> _To be completed — Week 5_

---

## 7. Quality Attribute Scenarios

### 7.1 Modifiability

> _To be completed — Week 6_

| Field | Description |
|---|---|
| **Source** | |
| **Stimulus** | |
| **Artifact** | |
| **Environment** | |
| **Response** | |
| **Measure** | |

### 7.2 Performance

> _To be completed — Week 6_

| Field | Description |
|---|---|
| **Source** | |
| **Stimulus** | |
| **Artifact** | |
| **Environment** | |
| **Response** | |
| **Measure** | |

### 7.3 Portability

> _To be completed — Week 6_

| Field | Description |
|---|---|
| **Source** | |
| **Stimulus** | |
| **Artifact** | |
| **Environment** | |
| **Response** | |
| **Measure** | |

### 7.4 Testability

> _To be completed — Week 6_

| Field | Description |
|---|---|
| **Source** | |
| **Stimulus** | |
| **Artifact** | |
| **Environment** | |
| **Response** | |
| **Measure** | |

### 7.5 Extensibility

> _To be completed — Week 6_

| Field | Description |
|---|---|
| **Source** | |
| **Stimulus** | |
| **Artifact** | |
| **Environment** | |
| **Response** | |
| **Measure** | |

---

## 8. Technical Debt Analysis

### 8.1 Intentional Trade-offs

> _To be completed — Week 7_

| Omitted Capability | Why Omitted | Consequence |
|---|---|---|
| | | |

### 8.2 Identified Structural Risks

> _To be completed — Week 7_

### 8.3 Comparison to Competitors

> _To be completed — Week 7_

| Quality Attribute | PocketFlow | LangChain |
|---|---|---|
| | | |

---

## 9. Conclusion

> _To be completed — Week 8_

---

## 10. Weekly Progress Log

### Week 1
- [ ] 
- [ ] 

### Week 2
- [ ] 
- [ ] 

### Week 3
- [ ] 
- [ ] 

### Week 4
- [ ] 
- [ ] 

### Week 5
- [ ] 
- [ ] 

### Week 6 — Midterm
- [ ] 
- [ ] 

### Week 7
- [ ] 
- [ ] 

### Week 8 — Final
- [ ] 
- [ ] 

---

## 11. References

- PocketFlow GitHub: https://github.com/The-Pocket/PocketFlow
- PocketFlow Docs: https://the-pocket.github.io/PocketFlow/
- DESOSA 2019: https://se.ewi.tudelft.nl/desosa2019/
- Bass, L., Clements, P., & Kazman, R. *Software Architecture in Practice*, 4th Ed.
- Kruchten, P. (1995). The 4+1 View Model of Architecture. *IEEE Software*.
- ISO/IEC/IEEE 42010:2011 — Architecture Description
