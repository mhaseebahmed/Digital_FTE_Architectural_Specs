# Implementation Guide 01: Environment Setup

> *"Laying the Foundation: Enterprise-grade tooling for Autonomous Agents."*

## 1. The Stack
We are choosing a **Modern Stack** optimized for speed, security, and reproducibility.

| Tool | Purpose | Enterprise Justification |
| :--- | :--- | :--- |
| **Python 3.12+** | The Logic Layer. | Latest async/await features and security patches. |
| **UV (by Astral)** | Package Manager. | Replaces `pip`. Deterministic dependency resolution (Lockfiles). |
| **Node.js v20+** | The Runtime. | Required for the Anthropic CLI. LTS (Long Term Support) version recommended. |
| **Claude Code** | The Brain CLI. | Official Anthropic tool with filesystem agency capabilities. |

---

## 2. Step-by-Step Installation

### Step 1: Install `uv` (The Python Manager)
We use `uv` to manage Python versions strictly.

**For Windows (PowerShell Administrator):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**For Mac/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

> **Verification:** Close/Reopen terminal. Run `uv --version`.

### Step 2: Initialize the Project Workspace
We will create a contained workspace.

1.  Clone the Starter Kit:
    ```bash
    git clone https://github.com/mhaseebahmed/digital-fte-starter-kit.git
    cd digital-fte-starter-kit
    ```

2.  Sync Dependencies:
    ```bash
    uv sync
    ```
    *This creates a `.venv` folder and installs all libraries defined in `pyproject.toml`.*

### Step 3: Install Node.js & Claude Code
1.  **Install Node.js:** Download the **LTS** version from [nodejs.org](https://nodejs.org/).
2.  **Install Claude Code:**
    ```bash
    npm install -g @anthropic/claude-code
    ```

### Step 4: Authentication
1.  **Login:**
    ```bash
    claude login
    ```
    *(Follow the browser instructions).*

2.  **Verify Connection:**
    ```bash
    claude -p "Ping"
    ```
    *Expected Output: "Pong" or a conversational reply.*

---

## 3. Project Structure (Monorepo)
The project is organized into isolated tiers for modularity.

```text
digital-fte-starter-kit/
├── shared_foundation/      <-- Logging, Config, Exceptions
├── tier_1_bronze/          <-- File System Watcher & Brain
├── tier_2_silver/          <-- Gmail, WhatsApp, HITL
├── tier_3_gold/            <-- Finance, Audit, Social
├── src/                    <-- Entry Point (main.py)
├── scripts/                <-- Setup Tools
└── tests/                  <-- Automated Test Suite
```
