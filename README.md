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
6. [Architecture Design Decisions](#6-architecture-design-decisions)
7. [Design Patterns](#7-design-patterns)
8. [Quality Attribute Scenarios](#8-quality-attribute-scenarios)
9. [Technical Debt Analysis](#9-technical-debt-analysis)
10. [Conclusion](#10-conclusion)
11. [Weekly Progress Log](#11-weekly-progress-log)
12. [References](#12-references)

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

The document is organized around the 4+1 view model: a stakeholder and context view, a development view, a process view, an architecture decision analysis, a design pattern analysis, and a quality attributes and technical debt assessment. Together, these views argue that PocketFlow's architecture is defined as much by what it **deliberately excludes** as by what it includes — and that this boundary decision is its central architectural contribution.

---

## 2. Stakeholder Analysis

### 2.1 Stakeholder Identification

Stakeholders were identified through analysis of the GitHub repository (contributors list, issues, pull requests) and the official documentation of PocketFlow.

| Stakeholder | Type | Role | Key Concerns |
|---|---|---|---|
| **Zachary Huang (@zachary62)** | Creator & Lead Maintainer | Defines architectural vision, authors core code and tutorials, drives the agentic coding philosophy | Minimalism, correctness of core abstraction, community growth, long-term positioning against LangChain |
| **22 contributors** | Developers | Submit bug fixes, cookbook examples, documentation improvements, and new features | Ease of contribution, stability of core API, clear contribution guidelines |
| **LLM Application Developers** | Primary Users | Build agents, workflows, RAG systems, and multi-agent applications on top of PocketFlow | Expressiveness, ease of use, quality of cookbook examples, LLM vendor flexibility |
| **AI Researchers & Educators** | Secondary Users | Use PocketFlow as a teaching tool or research substrate due to its readable, minimal codebase | Transparency, simplicity, ability to fork and audit all 100 lines |
| **Agentic Coding Practitioners** | Emerging Users | Use PocketFlow as the target framework for AI-assisted (e.g., Cursor AI) LLM app development | Intuitive structure that AI agents can reason about and generate code for |
| **Multi-language Port Maintainers** | Downstream Developers | Maintain TypeScript, Java, C++, Go, Rust, and PHP ports of the core abstraction | Stability of the Python core abstraction, clear semantics, documentation accuracy |
| **214+ Dependent Projects** | Downstream Integrators | Build production systems that depend on PocketFlow as an upstream library | API stability, backward compatibility, release management |
| **LangChain / LangGraph / CrewAI** | Competitor Frameworks | Define the design space that PocketFlow explicitly reacts against | N/A — indirect influence on PocketFlow's architectural positioning |
| **Discord Community** | Community | Provide user support, share use cases, give feedback on pain points | Responsiveness of maintainers, growing library of tutorials and examples |
| **LLM Providers (OpenAI, Anthropic, etc.)** | External Systems | Supply the LLM APIs that PocketFlow applications call | N/A — PocketFlow deliberately excludes all vendor-specific wrappers |

### 2.2 Power / Interest Grid

Stakeholders are mapped by their ability to influence PocketFlow's architecture (power) and their degree of ongoing engagement with the project (interest).

<img width="1448" height="1086" alt="image" src="https://github.com/user-attachments/assets/f6a31d03-b003-4cf2-8b5e-0b86ec84ab50" />

**Key observations:**
- The lead maintainer holds almost all architectural power, consistent with a solo-founded OSS project.
- LLM App Developers are the highest-interest group but have low direct power — their influence operates through GitHub issues, Discord feedback, and community cookbook contributions.
- Competitor frameworks have high indirect power: PocketFlow's entire architecture is a deliberate reaction to their complexity, meaning LangChain's design decisions effectively shaped PocketFlow's by opposition.

### 2.3 Key Architectural Decisions Driven by Stakeholders

Each of PocketFlow's most significant architectural decisions can be traced back to a specific stakeholder concern:

| Architectural Decision | Driven By | Rationale |
|---|---|---|
| **Zero external dependencies** | LLM App Developers frustrated with dependency conflicts in LangChain | Eliminates version hell and supply chain risk; users control their own dependency graph |
| **No vendor-specific wrappers** | Lead Maintainer's philosophy; LLM Providers' API volatility | Frequent LLM API changes make hardcoded wrappers a maintenance burden; users implement their own `call_llm()` |
| **100-line constraint** | Lead Maintainer; AI Researchers needing an auditable codebase | Forces ruthless prioritization; any developer (or AI agent) can read and understand the entire framework in minutes |
| **Cookbook-based documentation** | LLM App Developers requesting concrete examples | Formal API docs alone are insufficient for LLM orchestration; runnable examples are more instructive |
| **Multi-language ports** | Community requests from non-Python developers | The Graph + Shared Store abstraction is language-agnostic; ports validate this claim |
| **Agentic coding support (.cursorrules)** | Agentic Coding Practitioners using Cursor AI | PocketFlow's simplicity makes it uniquely suited for AI agents to generate application code on top of it |

---

## 3. Context View

### PocketFlow System Scope and Responsibilities

#### 3.1 System Scope

**PocketFlow** is a lightweight workflow and orchestration core for building LLM applications.

Its scope is to provide the minimal abstractions needed to structure application logic as **nodes** and **flows**, while leaving integrations with LLMs, tools, storage, and external services to user-defined code.

PocketFlow sits between the developer’s application code and external systems such as **LLM providers**, **tool APIs**, and **vector databases**.

#### Responsibilities

PocketFlow is responsible for:

- Providing core workflow abstractions such as `Node`, `Flow`, `BatchNode`, and async flow variants.
- Defining how steps in an LLM application are connected and executed.
- Supporting reusable graph-like workflows for:
  - agents
  - RAG pipelines
  - batch jobs
  - workflows
  - MapReduce-style applications
- Managing shared state between workflow steps through a simple shared store.
- Allowing developers to subclass and compose nodes and flows to build custom application behavior.
- Keeping the core lightweight and dependency-minimal, relying only on the Python standard library.
- Providing a stable abstraction that can be reused by cookbook examples, language ports, and ecosystem tooling.

#### Outside the System Scope

PocketFlow is not responsible for:

- Hosting or providing LLM models.
- Implementing provider-specific APIs for OpenAI, Claude, Gemini, Ollama, or DeepSeek.
- Owning external tools such as web search, file systems, REST APIs, MCP servers, or text-to-speech services.
- Managing vector databases such as ChromaDB, FAISS, or Pinecone.
- Providing authentication, billing, monitoring, deployment, or production infrastructure.
- Storing long-term application data beyond the workflow’s shared runtime state.
- Deciding application-specific prompts, business rules, retrieval logic, or tool behavior.

The diagram below illustrates the full context of PocketFlow, showing the system at the centre surrounded by the external actors and systems it interacts with.

<img width="1586" height="992" alt="image" src="https://github.com/user-attachments/assets/2fcc8d19-69a1-4853-8c1a-f8740c963a22" />

### 3.2 Context Diagram Description

This PocketFlow Context View shows PocketFlow as the central system and the external people, tools, platforms, and dependencies around it. PocketFlow Core is the main system in the center. It provides a minimal workflow/orchestration layer for building LLM applications using nodes and flows.

- **LLM App Developers** are the primary users. They build applications by subclassing PocketFlow’s Node and Flow abstractions.

- **User Space** represents the developer’s own application code. This is where custom utility functions such as call_llm() and tool wrappers are implemented.

- **LLM Providers** are external model services such as OpenAI, Claude, Gemini, Ollama, and DeepSeek. PocketFlow reaches them through user-defined LLM call functions.

- **External Tool APIs** are services or systems used by PocketFlow apps, such as web search, file systems, REST APIs, MCP servers, and text-to-speech tools.

- **Vector Databases** support embedding search and retrieval workflows, commonly used in RAG applications. Examples include ChromaDB, FAISS, and Pinecone.

- **Python Standard Library** is shown as the only true dependency, meaning PocketFlow is designed to stay lightweight and avoid requiring heavy external packages.

- **Cookbook Examples**, **Language Ports**, and **Dev Infrastructure** represent the surrounding ecosystem: sample apps, implementations in other languages, and project support tools such as GitHub, PyPI, MkDocs, and Discord.

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

## 6. Architecture Design Decisions

This section documents PocketFlow's key architecture design decisions using the course template from the lecture slides on architectural decisions: **Issue, Importance, Decision, Status, Group, Assumptions, Alternatives, Arguments, Implications, and Possible negative impact on quality**. 

### 6.1 Decision Overview

| ID | Decision | Primary quality drivers |
|---|---|---|
| ADD-01 | Keep core implementation to exactly 100 lines with zero dependencies | Maintainability, learning curve, flexibility |
| ADD-02 | Model workflows as Graph + Shared Store | Modifiability, simplicity, expressiveness |
| ADD-03 | Do not provide built-in utilities; instead provide examples | Portability, maintainability, vendor independence |
| ADD-04 | Three-phase lifecycle: Prep, Exec, Post | Separation of concerns, testability, reliability |
| ADD-05 | Nodes return string "actions" as routing keys | Extensibility, composability, understandability |
| ADD-06 | Humans design high-level flows; AI agents implement nodes | Learnability, ecosystem growth, controlled core scope |
| ADD-07 | Use a shared dictionary as a data contract | Modifiability, loose coupling, state management |

### 6.2 ADD-01: Minimalist Core Framework

| Template Item | Description |
|---|---|
| Issue | What should be the size and scope of the core framework? |
| Importance | High - affects maintainability, learning curve, and flexibility |
| Decision | Keep core implementation to exactly 100 lines with zero dependencies |
| Status | Accepted |
| Group | Core Architecture |
| Assumptions | Complex patterns can be built from simple primitives; developers prefer flexibility over convenience |
| Alternatives | Full-featured frameworks like LangChain (405K lines), CrewAI (18K lines) |
| Arguments | Reduces bloat, easier to understand, eliminates vendor lock-in, forces developers to understand fundamentals |
| Implications | Users must implement their own utility functions; steeper initial learning curve |
| Possible negative impact on quality | More initial setup work compared to batteries-included frameworks |

### 6.3 ADD-02: Graph + Shared Store Abstraction

| Template Item | Description |
|---|---|
| Issue | What should be the fundamental abstraction for LLM workflows? |
| Importance | High - this is the core conceptual model that everything else builds upon |
| Decision | Model workflows as Graph (nodes connected by labeled edges called Actions) + Shared Store (central state management) |
| Status | Accepted |
| Group | Core Architecture |
| Assumptions | Most LLM workflows can be modeled as directed graphs with shared state; state management is a cross-cutting concern |
| Alternatives | Chain-based (linear sequences only), Agent-based only, Pipeline-based |
| Arguments | More flexible than chains, simpler than full agent systems, enables complex patterns like loops and branching |
| Implications | All workflows must be expressible as graphs; state management is explicit rather than implicit |
| Possible negative impact on quality | May be limiting for workflows that don't fit graph model well |

### 6.4 ADD-03: No Built-in Utility Functions

| Template Item | Description |
|---|---|
| Issue | Should the framework include built-in utility functions for common tasks (LLM calls, embeddings, etc.)? |
| Importance | High - affects developer experience and framework flexibility |
| Decision | Do not provide built-in utilities; instead provide examples and let users implement their own |
| Status | Accepted |
| Group | Framework Philosophy |
| Assumptions | Vendor APIs change frequently; developers need flexibility to switch providers; optimizations are easier without lock-in |
| Alternatives | Include comprehensive utility library like LangChain, include basic utilities with extension points |
| Arguments | Avoids vendor lock-in, reduces maintenance burden, allows optimizations like prompt caching and streaming, enables switching between providers |
| Implications | Developers must implement their own utilities; more initial setup but long-term flexibility |
| Possible negative impact on quality | Higher barrier to entry; more boilerplate code for simple applications |

### 6.5 ADD-04: Node Lifecycle Pattern (Prep-Exec-Post)

| Template Item | Description |
|---|---|
| Issue | How should individual nodes structure their execution logic? |
| Importance | High - defines the fundamental unit of work pattern |
| Decision | Three-phase lifecycle: Prep (read/serialize from shared), Exec (perform computation isolated from shared), Post (write results to shared and return action) |
| Status | Accepted |
| Group | Node Architecture |
| Assumptions | Separation of concerns improves testability and retry logic; isolation enables portability |
| Alternatives | Single execute method with direct shared access, event-driven pattern, functional composition |
| Arguments | Enables safe retries (exec is idempotent), clear separation of data flow, easier testing, supports async operations |
| Implications | All nodes must follow this pattern; some boilerplate required for simple operations |
| Possible negative impact on quality | May feel verbose for simple operations; learning curve for the pattern |

### 6.6 ADD-05: Action-Based Routing

| Template Item | Description |
|---|---|
| Issue | How should nodes connect and determine execution flow? |
| Importance | High - enables dynamic workflows and conditional branching |
| Decision | Nodes return string "actions" that serve as routing keys to determine successor nodes |
| Status | Accepted |
| Group | Flow Control |
| Assumptions | String-based routing is sufficient for most workflows; conditional logic belongs in nodes not in flow definition |
| Alternatives | Hard-coded successor chains, boolean conditions, complex routing DSL |
| Arguments | Enables dynamic decision-making, supports loops and branching, simple to implement and understand |
| Implications | Flow control is decentralized to individual nodes; action naming becomes important |
| Possible negative impact on quality | Action naming conventions must be consistent; debugging flow paths may be complex |

### 6.7 ADD-06: Agentic Coding Methodology

| Template Item | Description |
|---|---|
| Issue | How should humans and AI agents collaborate in building LLM applications? |
| Importance | Medium - affects development process and team workflow |
| Decision | Humans design high-level flows and data schemas; AI agents implement specific nodes and utility functions |
| Status | Accepted |
| Group | Development Process |
| Assumptions | Humans are better at system design; AI is better at implementation details; iterative refinement is necessary |
| Alternatives | Fully human-driven development, fully AI-driven development, pair programming approach |
| Arguments | Leverages strengths of both humans and AI, enables rapid iteration, reduces human implementation burden |
| Implications | Requires clear design documentation; humans must understand framework well enough to design effectively |
| Possible negative impact on quality | Dependency on AI quality; may not work for all team structures |

### 6.8 ADD-07: Shared Store as Data Contract

| Template Item | Description |
|---|---|
| Issue | How should nodes communicate and share state? |
| Importance | High - fundamental to the architecture's data flow |
| Decision | Use a shared dictionary (shared store) as a data contract that all nodes agree upon for retrieving and storing data |
| Status | Accepted |
| Group | State Management |
| Assumptions | Dictionary-based state is sufficient for most use cases; explicit data contracts improve maintainability |
| Alternatives | Message passing, function parameters, event bus, database-backed state |
| Arguments | Simple and flexible; enables loose coupling between nodes; supports both in-memory and persistent implementations |
| Implications | Data schema design becomes critical; potential for data inconsistency if contract not followed |
| Possible negative impact on quality | No compile-time checking of data contracts; runtime errors possible |

<img width="2504" height="1430" alt="achitecture-decisions" src="https://github.com/user-attachments/assets/a9226abe-e24f-4d22-8a7f-8dd1f1fb47d5" />

### 6.10 Key Relationships

| Relationship | Explanation |
|---|---|
| Minimalist Core → Graph + Shared Store | The 100-line constraint forced the framework to adopt the simplest possible abstraction that could still support complex patterns |
| Graph + Shared Store → Node Lifecycle Pattern | The graph abstraction requires a consistent node interface, leading to the Prep-Exec-Post pattern |
| Graph + Shared Store → Action-Based Routing | The graph model needs a mechanism for edges, implemented as string actions returned by nodes |
| Graph + Shared Store → Shared Store as Data Contract | The shared store is the communication mechanism between nodes in the graph |
| No Built-in Utilities → Agentic Coding Methodology | Without built-in utilities, the framework relies on AI agents to implement project-specific utilities, making agentic coding essential |
| Node Lifecycle Pattern → Agentic Coding | The consistent node pattern makes it easier for AI agents to implement nodes correctly |

---

## 7. Design Patterns

### 7.1 Agent Pattern

> _To be completed — Week 5_

### 7.2 Workflow Pattern

> _To be completed — Week 5_

### 7.3 RAG Pattern

> _To be completed — Week 5_

### 7.4 MapReduce Pattern

> _To be completed — Week 5_

---

## 8. Quality Attribute Scenarios

### 8.1 Modifiability

> _To be completed — Week 6_

### 8.2 Performance

> _To be completed — Week 6_

### 8.3 Portability

> _To be completed — Week 6_

### 8.4 Testability

> _To be completed — Week 6_

### 8.5 Extensibility

> _To be completed — Week 6_

---

## 9. Technical Debt Analysis

### 9.1 Intentional Trade-offs

> _To be completed — Week 7_

### 9.2 Identified Structural Risks

> _To be completed — Week 7_

### 9.3 Comparison to Competitors

> _To be completed — Week 7_

---

## 10. Conclusion

> _To be completed — Week 8_

---

## 11. Weekly Progress Log

### Week 1
- [ Table of Contents ] 
- [ Introduction ] 

### Week 2
- [ Stakeholder Analysis ] 
- [ Context View ] 

### Week 3
- [ Architecture Design Decisions ] 
- [ Design Decision Relationship Graph ] 

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

## 12. References

- PocketFlow GitHub: https://github.com/The-Pocket/PocketFlow
- PocketFlow 100-line Python core: https://raw.githubusercontent.com/The-Pocket/PocketFlow/main/pocketflow/__init__.py
- PocketFlow Docs: https://the-pocket.github.io/PocketFlow/
- PocketFlow Core Abstraction: https://the-pocket.github.io/PocketFlow/core_abstraction/node.html
- PocketFlow Flow Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/flow.html
- PocketFlow Communication Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/communication.html
- PocketFlow Batch Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/batch.html
- PocketFlow Async Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/async.html
- PocketFlow Parallel Documentation: https://the-pocket.github.io/PocketFlow/core_abstraction/parallel.html
- Course slides: `pdfs/03-软件体系结构设计-01.pdf`
- Course slides: `pdfs/03-软件体系结构设计-02.pdf`
- DESOSA 2019: https://se.ewi.tudelft.nl/desosa2019/
- Bass, L., Clements, P., & Kazman, R. *Software Architecture in Practice*, 4th Ed.
- Kruchten, P. (1995). The 4+1 View Model of Architecture. *IEEE Software*.
- ISO/IEC/IEEE 42010:2011 — Architecture Description
