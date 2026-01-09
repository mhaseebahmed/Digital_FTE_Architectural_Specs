# System Architecture: The Anatomy of a Bionic Employee

This document details the technical architecture of the Digital FTE. The system is designed as a **Bionic Organism**, mimicking biological functions to achieve autonomy.

## 1. High-Level Architecture Diagram

```ascii
+---------------------------------------------------------------+
|                      THE HUMAN MANAGER                        |
|   (Interacts via Obsidian Dashboard & File System)            |
+------------------------------+--------------------------------+
                               |
                               v
+------------------------------+--------------------------------+
|                   THE MEMORY (Obsidian Vault)                 |
|  [Inbox]  [Needs_Action]  [Plans]  [Approved]  [Logs]         |
|           (The Shared State & Long-Term Storage)              |
+--------------+-------------------------------+----------------+
               ^                               |
               | (Read/Write)                  | (Triggers)
               v                               v
+--------------+-------------+   +-------------+----------------+
|  THE BRAIN (Reasoning)     |   |  THE SENSES (Perception)     |
|      Claude Code CLI       |<--|      Watcher Scripts         |
|   (Thinks & Plans Tasks)   |   |   (Monitor External World)   |
+--------------+-------------+   +-------------+----------------+
               |                               ^
               | (Calls Tools)                 | (Polls)
               v                               |
+--------------+-------------+   +-------------+----------------+
|    THE HANDS (Action)      |   |    EXTERNAL SOURCES          |
|       MCP Servers          |-->|   (Gmail, WhatsApp, Banks)   |
|   (Execute API Calls)      |   |                              |
+----------------------------+   +------------------------------+
```

---

## 2. Component Detail: The Senses (Watchers)
**Role:** The Nervous System.
**Function:** To bridge the gap between the "Passive" AI and the "Active" World.

### How it Works
Large Language Models cannot "listen." They need an external trigger.
*   **The Daemon:** A lightweight Python script runs continuously in the background.
*   **The Trigger:** It polls specific APIs or file paths.
*   **The Handoff:** When an event is detected (e.g., New Email), it does **not** process it. It simply downloads the data, converts it to a standardized Markdown file, and drops it into the `Inbox` folder.
*   **Why Separation?** This decouples "Seeing" from "Thinking." The Watcher is cheap and fast. The Brain (Claude) is expensive and slow. We only call the Brain when necessary.

---

## 3. Component Detail: The Brain (Claude Code)
**Role:** The Cognitive Engine.
**Function:** To process unstructured data and generate structured plans.

### The "Claude Code" Difference
We use the **Claude Code CLI**, not the web interface.
*   **System Access:** It runs in the terminal, giving it direct access to the computer's Operating System.
*   **Context Awareness:** It can recursively read the file structure to understand the "Project State" before making a decision.
*   **Tool Use:** It natively understands how to use the "Hands" (MCP) to execute complex chains of logic.

---

## 4. Component Detail: The Memory (Obsidian)
**Role:** State Management & User Interface.
**Function:** To provide a persistent record of reality.

### The "Dynamic Dashboard"
The system maintains a `Dashboard.md` file that is updated in real-time.
*   **Live Status:** Shows current bank balance, pending emails, and active projects.
*   **Audit Trail:** Every action taken by the Agent is logged in the `Logs/` folder. This allows the Human to "rewind" and understand *why* the Agent made a specific decision.

---

## 5. Component Detail: The Hands (MCP)
**Role:** The Action Interface.
**Function:** To safely interact with external APIs.

### Model Context Protocol (MCP)
The architecture uses MCP as a standardized "Driver Layer."
*   **The Server:** A small program that knows how to talk to a specific tool (e.g., "Stripe MCP").
*   **The Client:** Claude connects to the server to discover what tools are available (e.g., `stripe.charge_customer()`).
*   **The Abstraction:** If the Stripe API changes, we update the MCP Server. Claude (The Brain) does not need to be retrained. This makes the system modular and maintainable.

---

## 6. The Lifecycle of a Task

1.  **Perception:** Watcher sees an email: *"Please send invoice for $500."*
2.  **Ingestion:** Watcher creates file: `Inbox/Email_Request.md`.
3.  **Orchestration:** Orchestrator detects new file and launches Claude.
4.  **Reasoning:** Claude reads email, checks `Company_Handbook.md`, and drafts an invoice.
5.  **Proposal:** Claude creates `Pending_Approval/Invoice_Draft.md`.
6.  **Review:** Human moves file to `Approved/`.
7.  **Action:** Orchestrator sees approval, launches Claude to call `Email_MCP.send()`.
8.  **Completion:** File moved to `Done/`, Log updated.