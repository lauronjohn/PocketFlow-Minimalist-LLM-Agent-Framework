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

### 3.1 System Scope

The context view defines the boundaries of PocketFlow as a system — what it is responsible for, what surrounds it, and how it interacts with the external world. PocketFlow's boundary is unusually narrow by design: the system is responsible only for **graph-based orchestration of LLM tasks**. Everything else — calling LLMs, accessing databases, searching the web, managing storage — is deliberately placed outside the boundary in user space.

The diagram below illustrates the full context of PocketFlow, showing the system at the centre surrounded by the external actors and systems it interacts with.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/94cba42d-d4ba-4dd7-a311-e1ff1d00bc8d" />


**Figure 1 — PocketFlow Context View.** The teal box is the system boundary (100-line core). All LLM provider integrations, tool APIs, and vector databases live outside the boundary in user-implemented utility functions. Downstream consumers (LLM app developers, cookbook examples, and language port maintainers) interact with PocketFlow from below, while external services are accessed by user code from above.

### 3.2 Context Diagram Description

The context diagram reveals four distinct categories of external interaction:

**① Upstream: LLM Providers and Tool APIs**

PocketFlow interacts with LLM providers and external tools only indirectly — through user-implemented utility functions such as `call_llm()` and `search_web()`. The framework itself makes no network calls and imports no vendor SDKs. Supported LLM providers demonstrated in the cookbook include OpenAI (GPT-4o), Anthropic Claude, Google Gemini, Ollama for local models, and DeepSeek. External tools include web search APIs, file systems, REST services, Model Context Protocol (MCP) servers, and text-to-speech services. This indirect relationship is a defining architectural choice: it is the mechanism by which PocketFlow achieves zero vendor lock-in.

**② Downstream: LLM Application Developers**

LLM application developers consume PocketFlow either via `pip install pocketflow` from PyPI or by directly copying the 100-line source file. They subclass `Node` and `Flow` to build their own agents, workflows, and RAG pipelines. The framework imposes no constraints on what tools or LLMs they use — these are all resolved at the user level.

**③ Downstream: Cookbook Examples**

The 26+ cookbook applications in the repository serve a dual role: they are both documentation and proof-of-concept. Each cookbook example is a self-contained Python project that demonstrates one design pattern (Agent, Workflow, RAG, MapReduce, etc.) by combining PocketFlow's core classes with user-defined utility functions. They sit outside the system boundary because they are not part of the framework itself — they are consumers of it.

**④ Downstream: Language Port Ecosystem**

Six community-maintained ports (TypeScript, Java, C++, Go, Rust, PHP) reproduce the identical Graph + Shared Store abstraction in other language ecosystems. These ports are separate repositories under the `The-Pocket` GitHub organisation and are architecturally significant because their existence validates the language-independence of PocketFlow's core design.

### 3.3 External Dependencies

The table below summarises all external systems that interact with PocketFlow, their location relative to the system boundary, and their relationship type.

| External System | Location | Relationship | Examples |
|---|---|---|---|
| **LLM Provider APIs** | User utility functions (`utils/call_llm.py`) | Called by user code inside `Node.exec()`; no direct framework dependency | OpenAI, Anthropic, Gemini, Ollama, DeepSeek |
| **Vector Databases** | Cookbook examples | Used by RAG pattern implementations; not referenced by core | ChromaDB, FAISS, Pinecone |
| **External Tool APIs** | Cookbook examples and user utility functions | Called by Agent nodes as tool actions | DuckDuckGo Search, file systems, REST APIs |
| **MCP Servers** | Cookbook examples (`pocketflow-mcp`) | Accessed via the Model Context Protocol for standardised tool use | Numerical operations, custom MCP-compliant services |
| **PyPI** | Distribution | PocketFlow is published as a pip-installable package | `pip install pocketflow` |
| **GitHub** | Development infrastructure | Hosts source code, issues, pull requests, contributor discussions | github.com/The-Pocket/PocketFlow |
| **MkDocs / GitHub Pages** | Documentation infrastructure | Renders the official documentation site | the-pocket.github.io/PocketFlow |
| **Discord** | Community infrastructure | Hosts developer community for support and feedback | discord.gg/hUHHE9Sa6T |
| **Language Port Repositories** | Sibling repositories | Independently maintained ports of the core abstraction | PocketFlow-Typescript, PocketFlow-Java, PocketFlow-Go, etc. |
| **Python Standard Library** | Core package imports | The only imports used by `pocketflow/__init__.py` itself | `typing`, `abc`, `asyncio`, `copy` |

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

### 7.2 Performance

> _To be completed — Week 6_

### 7.3 Portability

> _To be completed — Week 6_

### 7.4 Testability

> _To be completed — Week 6_

### 7.5 Extensibility

> _To be completed — Week 6_

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
