# PocketFlow — Software Architecture Recovery

> **Course:** Modern Software Architecture — Wuhan University, 2026
> **Instructor:** Prof. Peng Liang
> **Target System:** [PocketFlow](https://github.com/The-Pocket/PocketFlow)
> **Frameworks:** [DESOSA 2019](https://se.ewi.tudelft.nl/desosa2019/) · Rozanski & Woods (2012) · Kruchten 4+1 (1995) · ISO/IEC/IEEE 42010:2011

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Stakeholder Analysis](#2-stakeholder-analysis)
3. [Context View](#3-context-view)
4. [Logical View](#4-logical-view)
5. [Development View](#5-development-view)
6. [Process View](#6-process-view)
7. [Deployment View](#7-deployment-view)
8. [Architecture Design Decisions](#8-architecture-design-decisions)
9. [Design Patterns](#9-design-patterns)
10. [Quality Attribute Scenarios & Tactics](#10-quality-attribute-scenarios--tactics)
11. [Technical Debt Analysis](#11-technical-debt-analysis)
12. [Conclusion](#12-conclusion)
13. [Weekly Progress Log](#13-weekly-progress-log)
14. [References](#14-references)

---

## 1. Introduction

### 1.1 What is PocketFlow?

<img width="2558" height="859" alt="image" src="https://github.com/user-attachments/assets/acdf2eda-cb69-4206-9d31-c3efe2a94da9" />

PocketFlow is a **100-line minimalist LLM (Large Language Model) orchestration framework** written in Python. It was created as a direct counter-argument to frameworks like LangChain and CrewAI, which the author argues are over-engineered for the problem they solve. Its central thesis is that the entire core abstraction needed for LLM application development — Nodes, Flows, and a Shared Store — can be expressed in exactly 100 lines of code, with zero external dependencies and zero vendor lock-in.

Despite its minimal size, PocketFlow is expressive enough to implement the full spectrum of modern LLM application patterns, including Agents, Workflows, Retrieval-Augmented Generation (RAG), MapReduce, Multi-Agent systems, and Structured Output. It models LLM workflows as a **Graph + Shared Store**, where:

- **Node** handles a single (LLM) task via a `prep → exec → post` lifecycle
- **Flow** connects Nodes through labeled edges called Actions
- **Shared Store** is a shared dictionary enabling communication between Nodes within a Flow

The framework is available as a Python package (`pip install pocketflow`) or by directly copying the 100-line source. It has since been ported to TypeScript, Java, C++, Go, Rust, and PHP by the community. As of 2026, the repository has over 10,000 GitHub stars, 1,200+ forks, and 200+ dependent projects.

| Framework | Abstraction | Lines of Code | Package Size |
|---|---|---|---|
| LangChain | Agent, Chain | 405K | +166 MB |
| CrewAI | Agent, Chain | 18K | +173 MB |
| LangGraph | Agent, Graph | 37K | +51 MB |
| AutoGen | Agent | 7K | +26 MB |
| **PocketFlow** | **Graph** | **100** | **+56 KB** |

### 1.2 Why PocketFlow is Architecturally Interesting

PocketFlow is architecturally interesting not despite its minimalism, but **because** of it. It forces a fundamental question: *what is the irreducible core abstraction for LLM orchestration?*

- **Radical minimalism as a design principle.** The 100-line constraint is not a limitation but an explicit architectural goal, making every line of code an intentional decision.
- **Intentional exclusion as architecture.** PocketFlow explicitly does not provide LLM vendor wrappers or app-specific utilities. This boundary decision — what the framework *refuses* to do — is as architecturally significant as what it provides.
- **Cross-language portability.** The same abstraction has been faithfully reproduced in six programming languages, evidencing the abstraction's structural soundness and language-independence.
- **Agentic coding philosophy.** PocketFlow is designed to be intuitive enough for AI agents themselves to build LLM applications on top of it.

### 1.3 Scope and Methodology

This document recovers and describes the software architecture of PocketFlow as of 2026. The analysis is grounded in three complementary theoretical frameworks:

| Framework | Source | Application in This Document |
|---|---|---|
| **4+1 View Model** | Kruchten (1995) | Organises §3–§7 into Logical, Process, Development, Deployment, and Context views |
| **Viewpoint Catalog** | Rozanski & Woods (2012) | Provides Stakeholder, Context, Functional, Information, Concurrency, and Deployment viewpoints |
| **Architecture Description Standard** | ISO/IEC/IEEE 42010:2011 | Maps stakeholder concerns → viewpoints → views → models |

#### Viewpoint Specification (ISO 42010)

| Viewpoint | Stakeholders Addressed | Concerns Addressed | Section | Notation |
|---|---|---|---|---|
| **Context** | All stakeholders | System boundaries, external interfaces, responsibilities | §3 | Box-and-line diagram |
| **Logical / Functional** | LLM Developers, AI Researchers | Key abstractions, class hierarchy, data flow | §4 | UML Class Diagram |
| **Development** | Contributors, Port Maintainers | Module structure, build conventions, code organisation | §5 | Package structure, tables |
| **Process / Concurrency** | LLM Developers, Integrators | Runtime execution model, concurrency, fault tolerance | §6 | Sequence diagram, flowchart |
| **Deployment** | LLM Developers, PyPI users | Installation, distribution, cross-language portability | §7 | Deployment diagram |

The analysis covers the Python core package (`pocketflow/__init__.py`), the cookbook of 40+ example applications, the multi-language port ecosystem, and the project's community infrastructure. LLM provider SDKs, vector databases, and third-party applications built on top of PocketFlow are outside the system boundary.

---

## 2. Stakeholder Analysis

### 2.1 Stakeholder Identification

Stakeholders were identified through analysis of the GitHub repository (contributors, issues, pull requests) and the official documentation, following the stakeholder taxonomy of ISO/IEC/IEEE 42010 and Rozanski & Woods.

| Stakeholder | Type | Role | Key Concerns |
|---|---|---|---|
| **Zachary Huang (@zachary62)** | Creator & Lead Maintainer | Defines architectural vision, authors core code and tutorials | Minimalism, correctness of core abstraction, community growth |
| **22 contributors** | Developers | Submit bug fixes, cookbook examples, documentation | Ease of contribution, API stability, clear guidelines |
| **LLM Application Developers** | Primary Users | Build agents, workflows, RAG systems on PocketFlow | Expressiveness, ease of use, quality of cookbook examples |
| **AI Researchers & Educators** | Secondary Users | Use as a teaching tool or research substrate | Transparency, simplicity, auditability of 100-line core |
| **Agentic Coding Practitioners** | Emerging Users | Use PocketFlow as a target for AI-assisted development | Intuitive structure that AI agents can reason about |
| **Multi-language Port Maintainers** | Downstream Developers | Maintain TypeScript, Java, C++, Go, Rust, PHP ports | Stability of core abstraction, clear semantics |
| **214+ Dependent Projects** | Downstream Integrators | Build production systems depending on PocketFlow | API stability, backward compatibility, release management |
| **Competitor Frameworks** | Indirect Influence | LangChain, CrewAI, LangGraph define the design space | N/A — indirect architectural influence by opposition |
| **Discord Community** | Community | Provide support, feedback, share use cases | Responsiveness, tutorials, examples |
| **LLM Providers (OpenAI, Anthropic…)** | External Systems | Supply the LLM APIs that PocketFlow apps call | N/A — PocketFlow excludes all vendor-specific wrappers |

### 2.2 Power / Interest Grid

<img width="1448" height="1086" alt="image" src="https://github.com/user-attachments/assets/f6a31d03-b003-4cf2-8b5e-0b86ec84ab50" />

**Key observations:**
- The lead maintainer holds almost all architectural power, consistent with a solo-founded OSS project.
- LLM App Developers are the highest-interest group but have low direct power — their influence operates through GitHub issues, Discord feedback, and community cookbook contributions.
- Competitor frameworks have high *indirect* power: PocketFlow's entire architecture is a deliberate reaction to their complexity.

### 2.3 Stakeholder Concerns Mapping

| Stakeholder | Primary Concern | Quality Attribute | Driven ADD |
|---|---|---|---|
| Lead Maintainer | Core must be auditable and minimal | Maintainability | ADD-01 |
| LLM App Developers | No vendor lock-in; easy vendor switching | Modifiability | ADD-03 |
| LLM App Developers | Complex patterns from simple primitives | Expressiveness | ADD-02 |
| AI Researchers | Entire framework readable in minutes | Understandability | ADD-01 |
| Port Maintainers | Abstraction must be language-agnostic | Portability | ADD-02 |
| Agentic Coding Practitioners | AI agents must understand the framework | Learnability | ADD-06 |
| Dependent Projects | API must not break unexpectedly | Stability | ADD-04 |

### 2.4 Key Architectural Decisions Driven by Stakeholders

| Architectural Decision | Driven By | Rationale |
|---|---|---|
| **Zero external dependencies** | LLM App Developers frustrated by LangChain dependency conflicts | Eliminates version hell and supply chain risk |
| **No vendor-specific wrappers** | Lead Maintainer; LLM Providers' API volatility | Frequent API changes make hardcoded wrappers a maintenance burden |
| **100-line constraint** | Lead Maintainer; AI Researchers | Forces ruthless prioritisation; any developer or AI agent can read the entire framework in minutes |
| **Cookbook-based documentation** | LLM App Developers requesting concrete examples | Formal API docs alone are insufficient for LLM orchestration |
| **Multi-language ports** | Community requests from non-Python developers | The Graph + Shared Store abstraction is language-agnostic |
| **Agentic coding support (.cursorrules)** | Agentic Coding Practitioners | PocketFlow's simplicity makes it uniquely suited for AI-generated application code |

---

## 3. Context View

### 3.1 System Scope

**PocketFlow** is a lightweight workflow and orchestration core for building LLM applications. Its scope is to provide the minimal abstractions needed to structure application logic as **nodes** and **flows**, while leaving integrations with LLMs, tools, storage, and external services to user-defined code.

#### Responsibilities

PocketFlow **is responsible for:**
- Providing core workflow abstractions: `Node`, `Flow`, `BatchNode`, async variants
- Defining how steps in an LLM application are connected and executed
- Supporting reusable graph-like workflows for agents, RAG pipelines, batch jobs, and MapReduce
- Managing shared state between workflow steps through a simple shared store

PocketFlow **is NOT responsible for:**
- Hosting or providing LLM models
- Implementing provider-specific APIs (OpenAI, Claude, Gemini, Ollama, DeepSeek)
- Managing external tools (web search, file systems, REST APIs, MCP servers)
- Managing vector databases (ChromaDB, FAISS, Pinecone)
- Authentication, billing, monitoring, deployment, or production infrastructure

### 3.2 Context Diagram

<img width="1586" height="992" alt="image" src="https://github.com/user-attachments/assets/2fcc8d19-69a1-4853-8c1a-f8740c963a22" />

### 3.3 External Interface Summary

| External System | Relationship | Interface Type | Owned By |
|---|---|---|---|
| **LLM Providers** (OpenAI, Anthropic…) | Apps call via user-defined `call_llm()` | HTTP/SDK (user-implemented) | User |
| **Vector Databases** (ChromaDB, FAISS…) | Apps call for RAG retrieval | Python SDK (user-implemented) | User |
| **Tool APIs** (Search, Files, MCP…) | Apps call as action nodes | HTTP/SDK (user-implemented) | User |
| **Python Standard Library** | Only true framework dependency | `typing`, `abc`, `asyncio` | Python |
| **PyPI** | Package distribution | `pip install pocketflow` | Maintainer |
| **GitHub** | Source, issues, CI, cookbook | Git/GitHub Actions | Maintainer |

---

## 4. Logical View

The **Logical View** (Kruchten, 1995; Rozanski & Woods' *Functional Viewpoint*) reveals the system's key abstractions, their responsibilities, and their structural relationships. This view is addressed primarily at **LLM Application Developers** and **AI Researchers**.

### 4.1 Key Abstractions: Class Hierarchy

PocketFlow's core abstraction hierarchy is defined entirely within `pocketflow/__init__.py`. The complete type hierarchy is shown below.

```mermaid
classDiagram
    class BaseNode {
        +params : dict
        +successors : dict
        +prep(shared) prep_res
        +exec(prep_res) exec_res
        +post(shared, prep_res, exec_res) action
        +_run(shared) action
        +set_params(params)
        +__rshift__(other) BaseNode
    }

    class Node {
        +max_retries : int
        +wait : float
        +exec_fallback(prep_res, exc) exec_res
    }

    class BatchNode {
        +_exec(items)
    }

    class AsyncNode {
        +prep_async(shared) prep_res
        +exec_async(prep_res) exec_res
        +exec_fallback_async(prep_res, exc) exec_res
        +post_async(shared, prep_res, exec_res) action
        +_run_async(shared) action
    }

    class AsyncBatchNode {
    }

    class AsyncParallelBatchNode {
    }

    class Flow {
        +start_node : BaseNode
        +_orch(shared, params) action
        +_run(shared) action
    }

    class BatchFlow {
        +_run(shared) action
    }

    class AsyncFlow {
        +_orch_async(shared, params) action
        +_run_async(shared) action
    }

    class AsyncBatchFlow {
    }

    class AsyncParallelBatchFlow {
    }

    BaseNode <|-- Node
    BaseNode <|-- Flow
    Node <|-- BatchNode
    Node <|-- AsyncNode
    AsyncNode <|-- AsyncBatchNode
    BatchNode <|-- AsyncBatchNode
    AsyncNode <|-- AsyncParallelBatchNode
    BatchNode <|-- AsyncParallelBatchNode
    Flow <|-- BatchFlow
    Flow <|-- AsyncFlow
    AsyncNode <|-- AsyncFlow
    AsyncFlow <|-- AsyncBatchFlow
    BatchFlow <|-- AsyncBatchFlow
    AsyncFlow <|-- AsyncParallelBatchFlow
    BatchFlow <|-- AsyncParallelBatchFlow

    BaseNode "1" --> "0..*" BaseNode : successors
    Flow --> BaseNode : start_node
```

| Class | Role | Key Responsibility |
|---|---|---|
| `BaseNode` | Abstract base | Defines `prep → exec → post` template; manages successor routing |
| `Node` | Standard node | Adds retry logic (`max_retries`, `wait`, `exec_fallback`) |
| `BatchNode` | Batch processor | Iterates `_exec()` over each item returned by `prep()` |
| `AsyncNode` | Async node | Provides `*_async` variants of all lifecycle methods |
| `AsyncBatchNode` | Sequential async batch | Multiple inheritance from `AsyncNode` + `BatchNode`; async sequential iteration |
| `AsyncParallelBatchNode` | Parallel async batch | Multiple inheritance from `AsyncNode` + `BatchNode`; concurrent execution via `asyncio.gather` |
| `Flow` | Orchestrator | Runs the `_orch` loop; is itself a `BaseNode`, enabling nesting |
| `BatchFlow` | Batch flow orchestrator | Runs the entire flow once per item in a batch |
| `AsyncFlow` | Async orchestrator | Multiple inheritance from `Flow` + `AsyncNode`; async `_orch_async` loop |
| `AsyncBatchFlow` | Async batch flow | Multiple inheritance from `AsyncFlow` + `BatchFlow`; async batch orchestration |
| `AsyncParallelBatchFlow` | Parallel async batch flow | Multiple inheritance from `AsyncFlow` + `BatchFlow`; parallel async batch orchestration |

### 4.2 Core Abstraction: Nested Directed Graph

The fundamental architectural abstraction is a **nested directed graph with a shared store**:

```mermaid
graph TD
    subgraph OuterFlow["Outer Flow"]
        N1["NodeA"] -->|default| N2["NodeB"]
        N2 -->|approve| SF["Sub-Flow\nnested Flow node"]
        N2 -->|reject| N3["NodeC"]
        SF -->|done| N4["NodeD"]

        subgraph SF_inner["Sub-Flow internal"]
            S1["SubNode1"] -->|default| S2["SubNode2"]
        end
    end

    SS[("Shared Store\n{dict}")] <-->|read/write| N1
    SS <-->|read/write| N2
    SS <-->|read/write| SF
    SS <-->|read/write| N3
    SS <-->|read/write| N4

    style SS fill:#f9a825,stroke:#e65100,color:#000
    style OuterFlow fill:#e3f2fd,stroke:#1565c0
    style SF_inner fill:#fce4ec,stroke:#c62828
```

### 4.3 Information View: Shared Store Data Contract

The **Shared Store** is PocketFlow's central data-exchange mechanism — a plain Python dictionary designed upfront as an explicit **data contract**. This corresponds to the *Information Viewpoint* in Rozanski & Woods.

```mermaid
graph LR
    subgraph Contract["Shared Store Contract (designed first)"]
        K1["input_text : str"]
        K2["retrieved_docs : list"]
        K3["llm_response : str"]
        K4["final_output : str"]
    end

    N1["RetrieveNode\nwrites retrieved_docs"] -->|writes| K2
    N2["GenerateNode\nreads retrieved_docs\nwrites llm_response"] -->|reads| K2
    N2 -->|writes| K3
    N3["FormatNode\nreads llm_response\nwrites final_output"] -->|reads| K3
    N3 -->|writes| K4

    style Contract fill:#fff9c4,stroke:#f57f17
```

The shared store schema is designed *before* implementing any node. This "schema-first" approach prevents runtime key errors and is a direct consequence of ADD-07 (Shared Store as Data Contract).

---

## 5. Development View

The **Development View** (Kruchten, 1995; Rozanski & Woods' *Development Viewpoint*) describes the organisation of the software for development. Relevant to **contributors** and **port maintainers**.

### 5.1 Repository Structure

Applications built with PocketFlow follow a strict project convention:

```
my_project/
├── main.py              # Entry point; initialises shared store and runs flow
├── nodes.py             # All Node class definitions
├── flow.py              # Flow creation and node wiring
├── utils/               # User-defined utility functions (LLM calls, APIs)
│   └── call_llm.py      # Example: wraps vendor LLM SDK
└── docs/
    └── design.md        # Design document — source of truth for data schema and flow
```

### 5.2 Module Structure

| Class | Responsibility |
|---|---|
| `BaseNode` | Template lifecycle; edge routing via successor dict |
| `Node` | Standard node with retry logic and fallback mechanisms |
| `BatchNode` | Sequential batch processing over collections |
| `AsyncNode` | Non-blocking async operations via `*_async` lifecycle methods |
| `AsyncBatchNode` / `AsyncParallelBatchNode` | Sequential and parallel async batch variants |
| `Flow` / `AsyncFlow` | Graph orchestration loop (`_orch`) |

### 5.3 Node Lifecycle

Every node follows a strict three-phase lifecycle defined by `BaseNode`:

1. **`prep(shared)`** — Read and serialize data from the shared store. Returns `prep_res`.
2. **`exec(prep_res)`** — Perform the core computation in **isolation** from the shared store. **Idempotent** by design, enabling safe retries. Returns `exec_res`.
3. **`post(shared, prep_res, exec_res)`** — Write results back to the shared store and return an **action string** routing to the next node.

### 5.4 Dependencies and Technology Stack

**Core Framework Dependencies:** Zero. Imports only Python standard library:

```python
import copy, asyncio
from abc import ABC, abstractmethod
from typing import Any, Optional
```

**Application Dependencies:** Fully user-controlled. Typical `requirements.txt`:

```
pocketflow
openai          # or anthropic, google-generativeai — user's choice
PyYAML
```

### 5.5 Development Process: Agentic Coding Methodology

| Step | Activity | Human | AI Agent | Primary Artifact |
|------|----------|:------:|:--------:|-----------------|
| 1 | Requirements | ★★★ | ★☆☆ | `docs/design.md` |
| 2 | Flow Design | ★★☆ | ★★☆ | `docs/design.md` |
| 3 | Utility Functions | ★★☆ | ★★☆ | `utils/*.py` |
| 4 | Data Contract Design | ★☆☆ | ★★★ | `docs/design.md` |
| 5 | Node Design | ★☆☆ | ★★★ | `docs/design.md` |
| 6 | Implementation | ★☆☆ | ★★★ | `flow.py`, `nodes.py`, `main.py` |
| 7 | Optimisation | ★★☆ | ★★☆ | Prompt refinement |
| 8 | Reliability | ★☆☆ | ★★★ | Test cases, retry config |


### 5.6 Key Development Constraints: The Three Zeros

```mermaid
mindmap
  root((Three Zeros\nPhilosophy))
    Zero Bloat
      Core limited to 100 lines
      No feature creep
      Every line intentional
    Zero Dependencies
      Python stdlib only
      No version conflicts
      No supply chain risk
    Zero Vendor Lock-in
      No bundled LLM wrappers
      Users own their utils/
      Switch providers freely
```

---

## 6. Process View

The **Process View** (Kruchten, 1995; Rozanski & Woods' *Concurrency Viewpoint*) describes the system's runtime execution model, concurrency structure, and fault-tolerance mechanisms.

### 6.1 Runtime Architecture Overview

```mermaid
graph TD
    subgraph Storage["State Management"]
        SharedStore[("Shared Store\nGlobal Dictionary / Data Contract")]
    end

    subgraph FlowOrchestrator["Flow Orchestrator: _orch loop"]
        NodeA["Node A\nprep / exec / post"]
        NodeB["Node B\nprep / exec / post"]
        SubFlow["Sub-Flow\nNested Node"]

        NodeA -->|default| NodeB
        NodeB -->|retry| NodeA
        NodeA -->|approve| SubFlow
    end

    NodeA <-->|Reads / Writes| SharedStore
    NodeB <-->|Reads / Writes| SharedStore
    SubFlow <-->|Reads / Writes| SharedStore

    style SharedStore fill:#f9a825,stroke:#e65100,color:#000
    style FlowOrchestrator fill:#e8f5e9,stroke:#2e7d32,stroke-dasharray:5 5
```

### 6.2 Node Execution Lifecycle: Phase Isolation and Fault Tolerance

```mermaid
sequenceDiagram
    autonumber
    participant Store as Shared Store
    participant Node as PocketFlow Node Engine
    participant Ext as External Service / LLM

    Note over Node: Phase 1 — prep()
    Node->>Store: prep(shared) — read state
    Store-->>Node: return prep_res

    Note over Node: Phase 2 — exec() isolated from Shared Store
    loop Retry Loop up to max_retries
        Node->>Ext: exec(prep_res) — LLM / API call
        alt Success
            Ext-->>Node: return exec_res
        else Exception
            Ext-->>Node: raise Exception — increment cur_retry
            Note over Node: If retries exhausted: exec_fallback()
        end
    end

    Note over Node: Phase 3 — post()
    Node->>Store: post(shared, prep_res, exec_res) — write results
    Node->>Node: return Action string
```

### 6.3 Flow Orchestration and Action-Based Routing

The `Flow` class manages runtime execution through an internal orchestrator (`_orch`):

```python
def _orch(self, shared, params=None):
    curr, p, last_action = copy.copy(self.start_node), (params or {**self.params}), None
    while curr:
        curr.set_params(p)
        last_action = curr._run(shared)
        curr = copy.copy(self.get_next_node(curr, last_action))
    return last_action
```

| Routing Type | Syntax | Trigger |
|---|---|---|
| **Default transition** | `node_a >> node_b` | `post()` returns `"default"` |
| **Named action** | `node_a - "approve" >> node_b` | `post()` returns `"approve"` |
| **Loop** | `node_a - "retry" >> node_a` | `post()` returns `"retry"` |
| **Terminal** | No successor defined | Any unmatched action string |

### 6.4 Communication Patterns

| Mechanism | Scope | Use Case |
|---|---|---|
| **Shared Store** | Global, persistent across all nodes | LLM results, large content, intermediate state |
| **Params** | Local, scoped per node execution | Task identifiers in batch mode; flow-level metadata |

### 6.5 Concurrency Patterns

```mermaid
graph TD
    subgraph BatchNode["Standard Batch — BatchNode"]
        B_prep["prep() returns List"] --> B_loop{for each item}
        B_loop -->|sequential| B_exec1["exec(item 1)"]
        B_exec1 --> B_exec2["exec(item 2)"]
        B_exec2 --> B_post["post() receives List of results"]
    end

    subgraph AsyncParallel["Parallel Async — AsyncParallelBatchNode"]
        P_prep["prep_async() returns List"] --> P_gather["asyncio.gather()"]
        P_gather --> P_exec1["exec_async(item 1)"]
        P_gather --> P_exec2["exec_async(item 2)"]
        P_gather --> P_exec3["exec_async(item 3)"]
        P_exec1 & P_exec2 & P_exec3 --> P_post["post_async() receives List of results"]
    end
```

### 6.6 Error Handling and Fault Tolerance

| Parameter | Description | Default |
|---|---|---|
| `max_retries` | Maximum execution attempts | `1` |
| `wait` | Seconds between retry attempts | `0` |
| `exec_fallback(prep_res, exc)` | Called when all retries are exhausted | Raises exception |
| `cur_retry` | Current retry count, accessible inside `exec` | `0` |

### 6.7 Nested Flow Composition

Flows can act as Nodes, enabling hierarchical composition:
- Sub-flows can be embedded as nodes within parent flows
- Node params merge from all parent flows in the hierarchy
- Flows execute `prep()` and `post()` but not `exec()` — their internal logic *is* the orchestration

---

## 7. Deployment View

The **Deployment View** (Kruchten, 1995; Rozanski & Woods' *Deployment Viewpoint*) describes how the software is distributed, installed, and deployed. Because PocketFlow is a library rather than a deployed service, this view focuses on **distribution mechanisms** and **cross-language portability**.

### 7.1 Distribution and Installation Model

```mermaid
graph LR
    subgraph Maintainer["Maintainer"]
        SRC["pocketflow/__init__.py\n100 lines"]
        PKG["PyPI Package\npocketflow"]
        SRC -->|pip publish| PKG
    end

    subgraph Developer["LLM Developer / Project"]
        INST["pip install pocketflow"]
        APP["Application\nnodes.py / flow.py / main.py"]
        INST --> APP
    end

    subgraph UserDeps["User-Managed Integrations"]
        LLM["LLM Provider SDK\nopenai, anthropic..."]
        VDB["Vector Database\nchromadb, faiss..."]
        TOOLS["Tool APIs\nsearch, files, MCP..."]
    end

    PKG -->|PyPI distribution| INST
    APP -->|utils/call_llm.py| LLM
    APP -->|utils/retrieval.py| VDB
    APP -->|utils/tools.py| TOOLS

    style Maintainer fill:#e8f5e9,stroke:#2e7d32
    style Developer fill:#e3f2fd,stroke:#1565c0
    style UserDeps fill:#fff3e0,stroke:#e65100
```

The deployment boundary enforced by ADD-03 (No Built-in Utilities) means PocketFlow's installation footprint is 56 KB. All heavyweight dependencies live in the user's zone.

### 7.2 Cross-Language Portability

| Language Port | Paradigm | Evidence of Abstraction Fidelity |
|---|---|---|
| **TypeScript** | Object-oriented, typed | Same `prep/exec/post` lifecycle |
| **Java** | Object-oriented, typed | Same `prep/exec/post` lifecycle |
| **C++** | Systems, performance-critical | Same graph traversal model |
| **Go** | Concurrent, compiled | Same action-based routing |
| **Rust** | Memory-safe, systems | Same ownership model applied to shared store |
| **PHP** | Web-focused | Same data contract pattern |

Consistent re-implementation across six language paradigms is strong evidence that the core abstraction is **architecturally sound and genuinely language-independent**.

---

## 8. Architecture Design Decisions

This section documents PocketFlow's key architecture design decisions using the **Tyree & Ackerman (2005)** template as taught in the course: Issue, Importance, Decision, Status, Group, Assumptions, Alternatives, Arguments, Implications, and Possible negative impact on quality.

### 8.1 Decision Overview

| ID | Decision | Primary Quality Drivers |
|---|---|---|
| ADD-01 | Keep core to exactly 100 lines with zero dependencies | Maintainability, understandability, portability |
| ADD-02 | Model workflows as Graph + Shared Store | Modifiability, expressiveness, simplicity |
| ADD-03 | Provide no built-in utilities; use examples instead | Portability, maintainability, vendor independence |
| ADD-04 | Three-phase lifecycle: Prep → Exec → Post | Testability, reliability, separation of concerns |
| ADD-05 | Nodes return string "actions" as routing keys | Extensibility, composability, understandability |
| ADD-06 | Humans design flows; AI agents implement nodes | Usability, learnability, productivity |
| ADD-07 | Shared dictionary as explicit data contract | Modifiability, loose coupling, state management |

### 8.2 ADD-01: Minimalist Core Framework

| Template Item | Description |
|---|---|
| **Issue** | What should be the size and scope of the core framework? |
| **Importance** | High — affects maintainability, learning curve, and portability |
| **Decision** | Keep core to exactly 100 lines with zero external dependencies |
| **Status** | Accepted |
| **Group** | Core Architecture |
| **Assumptions** | Complex patterns can be composed from simple primitives; developers prefer flexibility over convenience |
| **Alternatives** | Full-featured frameworks: LangChain (405K lines), CrewAI (18K lines) |
| **Arguments** | **Pros:** Eliminates vendor lock-in; makes the framework fully auditable; forces ruthless prioritisation; uniquely suitable for agentic coding.<br>**Cons:** Fewer ready-made features; shifts implementation burden to users; steeper initial learning curve.<br>**Summary:** Minimalism wins on auditability and portability, but shifts implementation work onto users. |
| **Implications** | Users must implement their own utility functions; cookbook examples become essential documentation |
| **Possible negative quality impact** | Higher barrier to entry; more boilerplate for simple applications |

### 8.3 ADD-02: Graph + Shared Store Abstraction

| Template Item | Description |
|---|---|
| **Issue** | What should be the fundamental abstraction for LLM workflows? |
| **Importance** | High — this is the core conceptual model everything else builds upon |
| **Decision** | Model workflows as a directed graph (nodes + labeled edges) + Shared Store (central state) |
| **Status** | Accepted |
| **Group** | Core Architecture |
| **Assumptions** | Most LLM workflows can be modelled as directed graphs; state management is a cross-cutting concern |
| **Alternatives** | Chain-based (linear only), pure agent-based, pipeline-based |
| **Arguments** | **Pros:** More flexible than chains; simpler than full agent systems; supports loops, branching, and conditionals.<br>**Cons:** Requires explicit graph modelling; large workflows need supporting diagrams.<br>**Summary:** The graph model offers the right balance between expressiveness and simplicity, at the cost of requiring upfront design work. |
| **Implications** | All workflows must be expressible as graphs; shared store schema becomes a critical design artifact |
| **Possible negative quality impact** | May be limiting for workflows that resist graph decomposition |

### 8.4 ADD-03: No Built-in Utility Functions

| Template Item | Description |
|---|---|
| **Issue** | Should the framework include built-in utilities for LLM calls, embeddings, tool use? |
| **Importance** | High — directly determines the developer experience |
| **Decision** | No built-in utilities; provide cookbook examples and let users implement their own |
| **Status** | Accepted |
| **Group** | Framework Philosophy |
| **Assumptions** | Vendor APIs change frequently; optimisations (prompt caching, streaming) are project-specific |
| **Alternatives** | Comprehensive utility library (LangChain approach); basic utilities with extension points |
| **Arguments** | **Pros:** Avoids vendor lock-in; eliminates maintenance burden from vendor API changes; enables per-project optimisations.<br>**Cons:** Increases boilerplate; less help for beginners; can lead to inconsistent patterns across projects.<br>**Summary:** Trading built-in convenience for long-term flexibility and freedom from vendor churn. |
| **Implications** | Developers must implement their own `utils/` directory; initial setup takes longer |
| **Possible negative quality impact** | Higher barrier to entry; beginners may implement utilities poorly |

### 8.5 ADD-04: Node Lifecycle Pattern (Prep–Exec–Post)

| Template Item | Description |
|---|---|
| **Issue** | How should individual nodes structure their execution logic? |
| **Importance** | High — defines the fundamental unit of work |
| **Decision** | Three-phase lifecycle: `prep()` (read from shared), `exec()` (compute, isolated), `post()` (write + route) |
| **Status** | Accepted |
| **Group** | Node Architecture |
| **Assumptions** | Separation of concerns improves testability; `exec()` isolation enables safe retries |
| **Alternatives** | Single `execute()` method with direct shared access; event-driven pattern; functional composition |
| **Arguments** | **Pros:** Safe retries when `exec` is idempotent; clean separation of data access from computation; improves testability.<br>**Cons:** Adds structural boilerplate to simple nodes; developers must learn phase boundaries.<br>**Summary:** Phase separation is the key enabler of both reliable retries and clean unit testing, justified by a small boilerplate cost. |
| **Implications** | All nodes must follow this pattern; some boilerplate required even for trivial operations |
| **Possible negative quality impact** | Can feel verbose for simple transformations |

### 8.6 ADD-05: Action-Based Routing

| Template Item | Description |
|---|---|
| **Issue** | How should nodes connect and determine execution flow? |
| **Importance** | High — enables dynamic workflows and conditional branching |
| **Decision** | Nodes return string "actions" as routing keys to determine successor nodes |
| **Status** | Accepted |
| **Group** | Flow Control |
| **Assumptions** | String-based routing is sufficient; conditional logic belongs in nodes, not in flow definition |
| **Alternatives** | Hard-coded successor chains; boolean conditions; complex routing DSL |
| **Arguments** | **Pros:** Enables dynamic decision-making; supports loops and branching; simple to implement and understand.<br>**Cons:** No compile-time validation of routes; consistent action naming requires discipline.<br>**Summary:** String actions give maximum routing flexibility at the cost of moving validation from compile-time to runtime. |
| **Implications** | Flow control is decentralised to individual nodes; action naming becomes a design concern |
| **Possible negative quality impact** | Typos in action strings cause silent routing failures |

### 8.7 ADD-06: Agentic Coding Methodology

| Template Item | Description |
|---|---|
| **Issue** | How should humans and AI agents collaborate in building LLM applications? |
| **Importance** | Medium — affects development process and team workflow |
| **Decision** | Humans design high-level flows and data schemas; AI agents implement specific nodes and utilities |
| **Status** | Accepted |
| **Group** | Development Process |
| **Assumptions** | Humans are better at system design; AI is better at implementation details; the framework must be simple enough for AI to fully understand |
| **Alternatives** | Fully human-driven; fully AI-driven; traditional pair programming |
| **Arguments** | **Pros:** Leverages complementary human/AI strengths; enables rapid iteration; reduces repetitive coding.<br>**Cons:** Depends on AI code quality; requires careful human review; may not suit compliance-heavy environments.<br>**Summary:** The methodology works well when the framework is simple enough for AI to reason about, which PocketFlow's 100-line core uniquely enables. |
| **Implications** | Requires clear design documentation upfront; humans must understand the framework well |
| **Possible negative quality impact** | Over-reliance on AI can introduce subtle bugs if design documents are ambiguous |

### 8.8 ADD-07: Shared Store as Data Contract

| Template Item | Description |
|---|---|
| **Issue** | How should nodes communicate and share state? |
| **Importance** | High — fundamental to the architecture's data flow |
| **Decision** | Use a shared dictionary designed upfront as an explicit data contract between all nodes |
| **Status** | Accepted |
| **Group** | State Management |
| **Assumptions** | Dictionary-based state is sufficient; explicit schemas improve maintainability |
| **Alternatives** | Message passing; function parameters; event bus; database-backed state |
| **Arguments** | **Pros:** Simple and flexible; enables loose coupling; supports both in-memory and persistent implementations.<br>**Cons:** No compile-time schema enforcement; runtime errors if contract is violated; requires disciplined documentation.<br>**Summary:** A plain dictionary is the simplest possible shared state mechanism; its lack of enforcement is acceptable when the schema is designed explicitly upfront. |
| **Implications** | Data schema design is a critical first step; `docs/design.md` is the schema source of truth |
| **Possible negative quality impact** | Runtime `KeyError` or type errors possible when contract is not followed |

### 8.9 Decision Relationships

Following Kruchten, Lago & van Vliet (2006), architectural decisions are not independent — they form a network of relationships. The diagram below shows how PocketFlow's seven ADDs interact, using relationship types taught in the course (enables, requires, constrains):

```mermaid
graph TD
    ADD01["ADD-01\nMinimalist Core\n100 lines, zero deps"]
    ADD02["ADD-02\nGraph and Shared Store\nCore abstraction"]
    ADD03["ADD-03\nNo Built-in Utilities\nUser-owned integrations"]
    ADD04["ADD-04\nPrep-Exec-Post\nNode lifecycle"]
    ADD05["ADD-05\nAction Routing\nLabeled edges"]
    ADD06["ADD-06\nAgentic Coding\nHuman and AI workflow"]
    ADD07["ADD-07\nShared Store Contract\nData contract"]

    ADD01 -->|enables| ADD02
    ADD01 -->|requires| ADD03
    ADD02 -->|enables| ADD04
    ADD02 -->|enables| ADD05
    ADD02 -->|requires| ADD07
    ADD03 -->|enables| ADD06
    ADD04 -->|enables| ADD06
    ADD07 -->|constrains| ADD04

    style ADD01 fill:#4CAF50,color:#fff,stroke:#2e7d32
    style ADD02 fill:#2196F3,color:#fff,stroke:#1565c0
    style ADD03 fill:#FF9800,color:#fff,stroke:#e65100
    style ADD04 fill:#9C27B0,color:#fff,stroke:#6a1b9a
    style ADD05 fill:#9C27B0,color:#fff,stroke:#6a1b9a
    style ADD06 fill:#F44336,color:#fff,stroke:#b71c1c
    style ADD07 fill:#00BCD4,color:#fff,stroke:#006064
```

| Relationship | Type | Explanation |
|---|---|---|
| ADD-01 → ADD-02 | *enables* | The 100-line constraint forced adoption of the simplest abstraction that could still support complex patterns |
| ADD-01 → ADD-03 | *requires* | Staying within 100 lines necessitates excluding all utility code |
| ADD-02 → ADD-04 | *enables* | The graph model requires a consistent node interface; Prep–Exec–Post provides it |
| ADD-02 → ADD-05 | *enables* | The graph model needs labeled edges; action strings are that mechanism |
| ADD-02 → ADD-07 | *requires* | Nodes in a graph must communicate; the shared store is the only mechanism |
| ADD-03 → ADD-06 | *enables* | Without built-in utilities, developers rely on AI agents to implement project-specific code |
| ADD-04 → ADD-06 | *enables* | The consistent lifecycle pattern makes it tractable for AI to implement nodes correctly |
| ADD-07 → ADD-04 | *constrains* | The data contract defines what `prep()` reads and `post()` writes, constraining lifecycle behaviour |

<img width="2504" height="1430" alt="architecture-decisions" src="https://github.com/user-attachments/assets/a9226abe-e24f-4d22-8a7f-8dd1f1fb47d5" />

---

## 9. Design Patterns

PocketFlow's architecture exhibits patterns at three levels: architectural style, GoF design patterns, and application-level patterns. Each is described using the formal description format from the course slides on *Software Architecture Patterns* (Buschmann et al., 1996; Bass et al., 2022).

### 9.1 Architectural Style: Blackboard Pattern

PocketFlow's Shared Store is an instance of the **Blackboard** architectural pattern — a data-centred style where independent processing units communicate exclusively through a shared data space, with no direct coupling between transformers.

| Property | Blackboard Pattern | PocketFlow Implementation |
|---|---|---|
| **Concepts** | Data-centred; concurrent transformations on shared data | Nodes transform data via a central dictionary |
| **Components** | Blackboard (data space) + Transformers (processing units) | `Shared Store` (dict) + `Node` instances |
| **Connectors** | Shared data space | `prep()` (read) and `post()` (write) |
| **Interfaces** | Asynchronous put/get | `shared["key"]` dictionary access |
| **Topology** | Star (all transformers connect to blackboard) | All nodes access one shared store per flow |
| **Constraint** | No direct communication between transformers | Nodes never call each other; only via shared store |

**Quality attribute impact:**

| Quality Attribute | Impact | Rationale |
|---|---|---|
| **Modularity** | ✅ High | Nodes are fully independent; no inter-node coupling |
| **Scalability** | ✅ High | New nodes added without modifying existing nodes |
| **Adaptability** | ✅ High | Node behaviour can be changed in isolation |
| **Testability** | ✅ High | `exec()` isolated from blackboard; unit-testable independently |
| **Consistency** | ⚠️ Risk | No built-in transactions; data contract must be maintained manually |
| **Performance** | ⚠️ Risk | Shared mutable state can bottleneck concurrent scenarios |

### 9.2 GoF Pattern: Template Method (Node Lifecycle)

The Prep–Exec–Post lifecycle is an instance of the **Template Method** pattern (GoF). `BaseNode._run()` defines the fixed algorithm skeleton; concrete subclasses override the individual steps.

```mermaid
classDiagram
    class BaseNode {
        <<Abstract Template>>
        +_run(shared) action
        +prep(shared) prep_res
        +exec(prep_res) exec_res
        +post(shared, prep_res, exec_res) action
    }
    note for BaseNode "_run() invariant skeleton:\n1. prep_res = prep(shared)\n2. exec_res = exec(prep_res)\n3. return post(shared, prep_res, exec_res)"

    class ConcreteNode {
        <<Subclass overrides hooks>>
        +prep(shared) prep_res
        +exec(prep_res) exec_res
        +post(shared, prep_res, exec_res) action
    }
    BaseNode <|-- ConcreteNode
```

### 9.3 GoF Pattern: Mediator (Flow Orchestration)

The `Flow` class implements the **Mediator** pattern — it centralises coordination so nodes never communicate directly. The `_orch` loop is the sole routing authority.

```mermaid
sequenceDiagram
    participant Flow as Flow (Mediator)
    participant NodeA
    participant NodeB
    participant NodeC

    Flow->>NodeA: _run(shared)
    NodeA-->>Flow: "approve"
    Note over Flow: Lookup successor for "approve" — NodeB
    Flow->>NodeB: _run(shared)
    NodeB-->>Flow: "default"
    Note over Flow: Lookup successor for "default" — NodeC
    Flow->>NodeC: _run(shared)
    NodeC-->>Flow: "done"
    Note over Flow: No successor for "done" — halt
```

### 9.4 Application-Level Patterns (Cookbook)

| Pattern | Core Mechanism | Key Node Types | Primary Use Case |
|---|---|---|---|
| **Agent** | Central "brain" node routes to action nodes based on LLM output | DecisionNode → ActionNode(s) → loop | Q&A with tool use |
| **Workflow** | Linear `>>` chain of task nodes | Sequential `Node` chain | Multi-step document processing |
| **RAG** | Offline index + online query as separate flows | IndexNode, RetrieveNode, GenerateNode | Document Q&A with vector search |
| **Map-Reduce** | `BatchNode` maps; aggregation node reduces | `BatchNode`, ReduceNode | Summarising many documents in parallel |
| **Structured Output** | Generate → validate → retry loop | GenerateNode, ValidateNode | Extracting typed data from text |
| **Multi-Agent** | Sub-flows as specialist agents sharing a parent store | Nested `Flow` instances | Complex tasks requiring specialised roles |

### 9.5 Pattern Summary

```mermaid
mindmap
  root((PocketFlow\nPatterns))
    Architectural Style
      Blackboard
        Shared Store as blackboard
        Nodes as transformers
        No inter-node coupling
    GoF Design Patterns
      Template Method
        prep-exec-post skeleton
        BaseNode._run as template
      Mediator
        Flow._orch as coordinator
        No direct node-to-node calls
    Application-Level Patterns
      Agent Loop
      Sequential Workflow
      RAG Pipeline
      Map-Reduce
      Structured Output
      Multi-Agent
```

---

## 10. Quality Attribute Scenarios & Tactics

Quality attributes in PocketFlow are analysed at two levels: **Quality Attribute Scenarios (QAS)** define concrete measurable requirements; **Architecture Tactics** identify the specific design decisions that control the quality attribute response (Bass et al., 2022).

### 10.1 Quality Attribute Scenarios

Following the standard QAS format from the course (Source → Stimulus → Environment → Artifact → Response → Measure):

| ID | Quality | Source | Stimulus | Environment | Artifact | Response | Measure |
|---|---|---|---|---|---|---|---|
| **QAS-01** | Performance | LLM Developer | 10 documents need simultaneous summarisation | Normal operation | `AsyncParallelBatchNode` | Executes all 10 `exec_async()` calls concurrently via `asyncio.gather` | ≥3× speedup vs. sequential; latency = slowest single call |
| **QAS-02** | Reliability | LLM Provider API | Rate limit exception raised during `exec()` | Normal operation | `Node(max_retries=3, wait=10)` | Automatically retries with configured wait; calls `exec_fallback` if exhausted | Succeeds within 3 attempts or degrades gracefully without crash |
| **QAS-03** | Modifiability | LLM Developer | Switch from OpenAI to Anthropic | Development phase | `utils/call_llm.py` | Only `call_llm.py` is replaced; no node or flow code changes | Change confined to `utils/` directory; zero core modifications required |
| **QAS-04** | Usability | AI Coding Agent (Cursor) | Developer provides natural-language specification | Development phase | 100-line core + `.cursorrules` | AI agent generates complete working flow, nodes, and utilities | Full application produced in one session; human reviews only high-level design |
| **QAS-05** | Portability | Enterprise Team | Need same workflow in TypeScript stack | Deployment phase | Core Graph abstraction | TypeScript port replicates identical semantics | Identical behaviour confirmed across 6 language ports |
| **QAS-06** | Maintainability | Developer | Diagnose incorrect output from `exec()` | Maintenance phase | Prep–Exec–Post lifecycle | `exec()` is isolated from shared state; unit-testable with mock inputs | Debug scope isolated to one phase; no side effects on store or other nodes |
| **QAS-07** | Availability | Critical LLM service | Service becomes unavailable mid-workflow | Production | `exec_fallback` mechanism | Returns fallback result instead of propagating exception | Workflow continues with degraded output; no crash, no data loss |

### 10.2 Architecture Tactics

**Tactics** are design decisions that influence the control of a quality attribute response (Bass et al., 2022). The subsections below map each key quality attribute to its relevant tactics and concrete realisation in PocketFlow.

#### Performance Tactics

```mermaid
graph TD
    PERF["Performance"]
    PERF --> RD["Reduce Resource Demand"]
    PERF --> RM["Manage Resources"]

    RD --> T1["Increase algorithm efficiency\nMinimal overhead in _orch loop"]
    RD --> T2["Reduce computing overhead\nZero-dependency core; no startup cost"]
    RM --> T3["Increase concurrency\nAsyncParallelBatchNode\nuses asyncio.gather"]
    RM --> T4["Maintain resource clones\nNode instances copied per\nexecution via copy.copy"]

    style PERF fill:#e3f2fd,stroke:#1565c0,color:#000
```

#### Reliability Tactics

```mermaid
graph TD
    REL["Reliability"]
    REL --> FD["Fault Detection"]
    REL --> FR["Fault Recovery"]
    REL --> FP["Fault Prevention"]

    FD --> R1["Exceptions\nexec() exceptions caught\nand counted vs. max_retries"]
    FR --> R2["Active redundancy\nRetry loop up to\nmax_retries attempts"]
    FR --> R3["exec_fallback()\nGraceful degradation\nwhen retries exhausted"]
    FP --> R4["Exec isolation\nexec() has no shared store access\nSafe to retry without side effects"]

    style REL fill:#e8f5e9,stroke:#2e7d32,color:#000
```

#### Modifiability Tactics

| Tactic Category | Sub-tactic | PocketFlow Implementation |
|---|---|---|
| **Localise changes** | Semantic coherence | All vendor-specific code confined to `utils/` directory |
| **Localise changes** | Anticipate expected changes | Shared Store schema designed upfront as explicit data contract |
| **Prevention of ripple effects** | Information hiding | Nodes expose only `prep/exec/post` interface; internals hidden |
| **Prevention of ripple effects** | Maintain existing interfaces | Flow's `_orch` API stable across all node changes |

#### Tactic-to-Pattern Mapping

| Architectural Pattern | Quality Driver | Tactics Implemented |
|---|---|---|
| **Blackboard (Shared Store)** | Modifiability | Localise changes; information hiding; prevent ripple effects |
| **Template Method (Prep–Exec–Post)** | Reliability, Testability | Fault prevention via exec isolation; modular components |
| **Mediator (Flow._orch)** | Modifiability | Localise routing changes; prevent ripple effects |
| **AsyncParallelBatchNode** | Performance | Increase concurrency; manage event rate |

#### Summary: Quality Attribute → Tactic → Framework Mechanism

| Quality Attribute | Tactic | PocketFlow Mechanism | QAS |
|---|---|---|---|
| Performance | Increase concurrency | `AsyncParallelBatchNode` + `asyncio.gather` | QAS-01 |
| Reliability | Fault recovery | Node retry + `exec_fallback` | QAS-02 |
| Reliability | Fault prevention | `exec()` isolated from Shared Store | QAS-06 |
| Modifiability | Localise changes | `utils/` directory isolation | QAS-03 |
| Modifiability | Anticipate change | Shared Store data contract design | QAS-07 |
| Portability | Portability layer | Language-agnostic Graph abstraction | QAS-05 |
| Usability | Minimise expertise required | 100-line core + agentic coding methodology | QAS-04 |

---

## 11. Technical Debt Analysis

PocketFlow's technical debt is primarily **intentional** — deliberate trade-offs aligned with its minimalist philosophy. The framework systematically transfers implementation burden to users in exchange for simplicity, flexibility, and zero vendor lock-in.

### 11.1 Design Debt

| Debt Item | Description | Root Cause | Trade-off Accepted |
|---|---|---|---|
| **100-line constraint** | Limits type annotations, advanced error handling, built-in validation | Intentional architectural constraint (ADD-01) | Portability and auditability gained |
| **No built-in utilities** | Users implement all vendor integrations | ADD-03 — vendor API volatility avoidance | Flexibility and zero lock-in gained |
| **Zero external dependencies** | Cannot use Pydantic, tenacity, etc. | ADD-01 — eliminates dependency conflicts | Supply chain safety gained |
| **Minimal abstraction layer** | No Agent, Chain, or QA helpers | Keeps core minimal and flexible | Expressiveness transferred to users |
| **Shared Store scalability** | In-memory dict; no transactions, no async race protection | ADD-07 design simplicity | Simplicity gained; scalability risk accepted |
| **Type safety** | Core uses `Any` types; no runtime schema validation | 100-line constraint limits annotation | Flexibility gained; IDE support limited |

### 11.2 Documentation Debt

| Debt Item | Description | Root Cause | Impact |
|---|---|---|---|
| **Documentation-First Policy** | AI agents must request MDC files before coding | Enables Agentic Coding (ADD-06) | Documentation overhead; drift risk |
| **Cookbook as primary docs** | Examples serve as documentation; no formal API reference | Examples more instructive for LLM apps | Risk of stale examples; edge-case gaps |

### 11.3 Test Debt

| Debt Item | Description | Root Cause | Impact |
|---|---|---|---|
| **No built-in testing framework** | No mock utilities, no flow test harness | ADD-01 — no external deps allowed | Inconsistent testing patterns across projects |
| **No async test patterns** | `AsyncParallelBatchNode` rate-limit testing is non-trivial | Async complexity beyond 100-line scope | Risk of untested concurrency edge cases |

### 11.4 Project Debt Overview

```mermaid
graph LR
    subgraph Framework["PocketFlow Provides"]
        F1["Graph abstraction"]
        F2["Node lifecycle"]
        F3["Action routing"]
        F4["Basic retry"]
    end

    subgraph User["User Must Implement"]
        U1["LLM / VDB integrations"]
        U2["Design patterns: Agent, RAG..."]
        U3["State persistence"]
        U4["Complex error handling"]
        U5["Testing infrastructure"]
    end

    Framework -->|intentional transfer| User

    style Framework fill:#e8f5e9,stroke:#2e7d32
    style User fill:#fff3e0,stroke:#e65100
```

**Summary:** Each debt item is a direct consequence of one or more ADDs, not of neglect. The primary runtime risk is that users implement utilities poorly — a risk the cookbook examples partially mitigate.

---

## 12. Conclusion

This document has recovered PocketFlow's software architecture using three complementary frameworks: Kruchten's 4+1 View Model, Rozanski & Woods' Viewpoint Catalog, and the ISO/IEC/IEEE 42010 standard. Five architectural views were produced — Context, Logical, Development, Process, and Deployment — each addressed to specific stakeholders and concerns, as required by the viewpoint specification in §1.3.

**The central architectural finding is that PocketFlow's architecture is defined as much by what it deliberately excludes as by what it includes.** The seven ADDs form a coherent, mutually-enabling decision network with no significant internal contradictions: the 100-line constraint (ADD-01) forces the Graph + Shared Store abstraction (ADD-02), which necessitates action routing (ADD-05) and the lifecycle pattern (ADD-04), which in turn enables the Agentic Coding Methodology (ADD-06).

Applying the course's tactics framework reveals that PocketFlow addresses all major quality attributes through architectural mechanisms: reliability via `exec()` isolation and retry (Reliability tactics); modifiability via `utils/` isolation and data contracts (Modifiability tactics); performance via `asyncio.gather` (Performance tactics); portability via a language-agnostic core (Portability layer). These mechanisms are realised through three patterns: Blackboard, Template Method, and Mediator.

**Primary quality risk and recommendation:** The absence of Shared Store schema validation is the most significant structural weakness. Runtime `KeyError` or type mismatches from undisciplined data contract usage are the most likely source of production failures. An optional Pydantic-based validator — delivered as a separate package, not in the 100-line core — would address this risk without violating any existing ADD.

In conclusion, PocketFlow is an architecturally coherent and well-motivated framework for LLM orchestration. Its minimalism is a deliberate strength: every architectural decision is traceable to a specific stakeholder concern or quality attribute, and the decision network is internally consistent.

---

## 13. Weekly Progress Log

### Week 1
- [x] Table of Contents
- [x] Introduction

### Week 2
- [x] Stakeholder Analysis
- [x] Context View

### Week 3
- [x] Architecture Design Decisions
- [x] Decision Relationship Graph
- [x] Development View

### Week 4
- [x] Process View

### Week 5
- [x] Design Patterns

### Week 6 — Midterm
- [x] Quality Attribute Scenarios

### Week 7
- [x] Technical Debt Analysis

### Week 8 — Final
- [x] Logical View (4+1 model — new)
- [x] Deployment View (4+1 model — new)
- [x] Architecture Tactics
- [x] Viewpoint Specification (ISO 42010)
- [x] Decision Relationships Network
- [x] Conclusion

---

## 14. References

### Primary Sources
- PocketFlow GitHub: https://github.com/The-Pocket/PocketFlow
- PocketFlow Python Core (100 lines): https://raw.githubusercontent.com/The-Pocket/PocketFlow/main/pocketflow/__init__.py
- PocketFlow Documentation: https://the-pocket.github.io/PocketFlow/
- PocketFlow Core Abstraction: https://the-pocket.github.io/PocketFlow/core_abstraction/node.html
- PocketFlow Flow Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/flow.html

### Course Materials
- Prof. Peng Liang — *软件体系结构描述* (Architecture Description), Wuhan University, 2026
- Prof. Peng Liang — *软件体系结构设计* (Architecture Design), Wuhan University, 2026
- Prof. Peng Liang — *软件体系结构模式* (Architecture Patterns), Wuhan University, 2026

### Reference Architecture Projects
- DESOSA 2019 — Delft Students on Software Architecture: https://se.ewi.tudelft.nl/desosa2019/

### Books and Standards
- Rozanski, N. & Woods, E. (2012). *Software Systems Architecture: Working with Stakeholders Using Viewpoints and Perspectives*, 2nd Ed. Addison-Wesley.
- Bass, L., Clements, P. & Kazman, R. (2022). *Software Architecture in Practice*, 4th Ed. Addison-Wesley.
- Kruchten, P. (1995). The 4+1 View Model of Architecture. *IEEE Software*, 12(6), 42–50.
- Tyree, J. & Ackerman, A. (2005). Architecture Decisions: Demystifying Architecture. *IEEE Software*, 22(2), 19–27.
- Kruchten, P., Lago, P. & van Vliet, H. (2006). Building Up and Reasoning about Architectural Knowledge. *QoSA 2006*.
- Buschmann, F. et al. (1996). *Pattern-Oriented Software Architecture: A System of Patterns*. Wiley.
- ISO/IEC/IEEE 42010:2011 — *Systems and Software Engineering — Architecture Description*.
