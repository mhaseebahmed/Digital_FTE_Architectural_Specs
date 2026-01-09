# Digital FTE: The Architectural Blueprint
> **Build Autonomous AI Employees, Not Just Chatbots.**

![License](https://img.shields.io/badge/license-MIT-blue.svg) ![Status](https://img.shields.io/badge/status-Enterprise%20Ready-green.svg) ![Stack](https://img.shields.io/badge/stack-Python%20%7C%20Claude%20Code%20%7C%20MCP-purple.svg)

## 📖 Introduction
This repository is not a collection of scripts; it is a **Master Class** in Agentic Architecture. It provides the definitive specification for building a "Digital FTE" (Full-Time Equivalent)—an autonomous software agent that operates 24/7, manages local files, negotiates with APIs, and respects human authority.

This blueprint transforms the vague concept of "AI Agents" into a concrete engineering discipline, covering **Perception, Reasoning, Action, and Resilience**.

---

## 🗺️ How to Use This Repository
The repository is divided into two distinct learning paths. We recommend following them sequentially.

### 📚 Part 1: The Theory (Architectural Specs)
*Located in `/Theory`*

Before writing code, you must understand the philosophy. These documents explain **WHY** we build this way.

*   **`01_Concept_Philosophy.md`**: Why "Agents" are different from "Automation." The shift from Deterministic to Probabilistic logic.
*   **`02_System_Architecture.md`**: The Anatomy of an Agent (Brain, Senses, Hands, Memory).
*   **`03_Economic_Model.md`**: The ROI calculation. Why an agent is "Headcount," not "Software."
*   **`04_Watcher_Patterns.md`**: Deep dive into Event Loops vs. Polling. How to wake up a passive LLM.
*   **`05_Reasoning_Loops.md`**: The OODA Loop (Observe, Orient, Decide, Act) applied to Claude Code.
*   **`06_MCP_Protocol.md`**: Understanding the "Universal Driver" standard for AI tools.
*   **`07_HITL_Workflow.md`**: Designing safety valves using File-Based State Machines.
*   **`08_Orchestration_Logic.md`**: Managing persistence, subprocesses, and Cron schedules.
*   **`09_Business_Handover.md`**: How to generate the "Monday Morning CEO Briefing."
*   **`10_Security_and_Resilience.md`**: Implementing Sandboxing, Redaction, and Self-Healing.

### 🛠️ Part 2: The Implementation (Engineering Blueprints)
*Located in `/Implementation`*

These are the "Lab Manuals." They contain production-ready Python code, security protocols, and deployment strategies.

#### **Foundation Layer**
*   **`00_Core_Foundation/`**: The shared libraries.
    *   **Enterprise Logging:** JSON-structured logs with PII redaction and rotation.
    *   **Configuration:** Strict Pydantic validation for `.env` variables.
    *   **Error Protocol:** A standardized `@retry` decorator with exponential backoff.

#### **Tiered Development Path**
*   **`01_Setup_Environment.md`**: Using `uv` for ultra-fast Python management. Secure Vault creation.
*   **`03_Impl_Bronze_The_Watcher.md`**: **The Minimum Viable Agent.** Building a robust File System watcher that handles large file uploads (Stabilization Loops).
*   **`04_Impl_Silver_The_Orchestrator.md`**: **The Assistant.** Adding Gmail Polling, Headless Browser Automation (WhatsApp), and the Approval State Machine.
*   **`05_Impl_Gold_The_Executive.md`**: **The Partner.** Implementing Autonomous Auditing, Scheduling, and Self-Healing Watchdogs using PM2.

---

## ⚡ Quick Start (The "Zero to One")

**Prerequisites:** Python 3.12+, Node.js v20+, Claude Code CLI.

```bash
# 1. Clone the Blueprint
git clone https://github.com/mhaseebahmed/Digital_FTE_Architectural_Specs.git
cd Digital_FTE_Architectural_Specs

# 2. Study the Architecture
# We strongly recommend reading Theory/02_System_Architecture.md first.

# 3. Begin Implementation
# Open Implementation/01_Setup_Environment.md and follow the guide step-by-step.
```

---

## 🏗️ Repository Structure (The Monorepo)

```text
Digital_FTE_Architectural_Specs/
├── README.md                           <-- (You are here)
├── Theory/                             <-- The Textbook
│   ├── 01_Concept_Philosophy.md
│   ├── 02_System_Architecture.md
│   ├── ... (Chapters 3-10)
│   └── 10_Security_and_Resilience.md
│
└── Implementation/                     <-- The Blueprint
    ├── 00_Core_Foundation/             <-- Shared Libraries
    │   ├── 01_Enterprise_Logging.md
    │   ├── 02_Configuration_Manager.md
    │   └── 03_Error_Handling_Protocol.md
    │
    ├── 01_Bronze_Tier/                 <-- Level 1: File Agent
    │   ├── 01_BaseWatcher_Architecture.md
    │   ├── 02_FileSystem_Watcher_Spec.md
    │   └── 03_Brain_Interface_Spec.md
    │
    ├── 02_Silver_Tier/                 <-- Level 2: Communication Agent
    │   ├── 01_Gmail_Watcher_Spec.md
    │   ├── 02_WhatsApp_Watcher_Spec.md
    │   └── 03_Approval_Workflow_Spec.md
    │
    ├── 03_Gold_Tier/                   <-- Level 3: Autonomous Executive
    │   ├── 01_Financial_Auditor_Spec.md
    │   ├── 02_Executive_Reporting_Spec.md
    │   └── 03_Resilience_Watcher_Spec.md
    │
    ├── 01_Setup_Environment.md         <-- Guide: Start Here
    ├── 02_Setup_Vault_Architecture.md  <-- Guide: Build the Office
    ├── 03_Impl_Bronze_The_Watcher.md   <-- Guide: Build Bronze
    ├── 04_Impl_Silver_The_Orchestrator.md <-- Guide: Build Silver
    └── 05_Impl_Gold_The_Executive.md   <-- Guide: Build Gold
```

---

## 🛡️ Enterprise Grade Guarantee
This repository adheres to strict engineering standards:
*   **Security:** No hardcoded secrets. strict `.gitignore` policies. PII scrubbing.
*   **Resilience:** No infinite loops. Race-condition protection. Self-healing process management.
*   **Scalability:** Modular architecture using Abstract Base Classes and standardized Interfaces.

> *Welcome to the future of work.*
