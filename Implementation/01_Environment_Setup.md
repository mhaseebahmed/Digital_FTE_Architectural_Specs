# Implementation Guide 01: Environment Setup

> *"Laying the Foundation: Modern, high-performance tooling for Autonomous Agents."*

## 1. The Stack
We are choosing a **Modern Stack** optimized for speed and reproducibility.

| Tool | Purpose | Why not standard tools? |
| :--- | :--- | :--- |
| **Python 3.12+** | The Logic Layer (Watchers). | We need the latest `asyncio` features. |
| **UV (by Astral)** | Package Manager. | Replaces `pip` and `venv`. It is 100x faster and handles Python versioning automatically. |
| **Node.js v20+** | The Runtime for Claude. | Claude Code is built in TypeScript/Node. |
| **Claude Code** | The Brain CLI. | The official Anthropic tool for file system agency. |

---

## 2. Step-by-Step Installation

### Step 1: Install `uv` (The Python Manager)
We do not install Python manually from python.org. We let `uv` handle it.

**For Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**For Mac/Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

> **Action:** Close and reopen your terminal after installation.
> **Verify:** Run `uv --version`.

### Step 2: Initialize the Project
We will create a structured Python project.

1.  Create the directory:
    ```bash
    mkdir AI_Employee
    cd AI_Employee
    ```

2.  Initialize with `uv`:
    ```bash
    uv init
    ```
    *This creates a `.venv` (Virtual Environment) and a `pyproject.toml` file automatically.*

3.  Pin the Python version (Ensures consistency):
    ```bash
    uv python install 3.12
    ```

### Step 3: Install Node.js & Claude Code
The "Brain" runs on Node.js.

1.  **Install Node.js:** Download the LTS version from [nodejs.org](https://nodejs.org/) (or use `nvm`).
    *   **Verify:** Run `node --version` (Must be v18+).

2.  **Install Claude Code:**
    ```bash
    npm install -g @anthropic/claude-code
    ```
    *The `-g` flag installs it globally so you can run it from any folder.*

### Step 4: Authentication
You must link your machine to your Anthropic account.

1.  Run the login command:
    ```bash
    claude login
    ```
2.  A browser window will open.
3.  Authorize the request.
4.  Copy the code provided and paste it back into your terminal.

---

## 3. The "Hello World" Test
Before proceeding, we must verify that all "Organs" are functioning.

**Test 1: Check the Brain**
Run this command in your terminal:
```bash
claude "What is 2 + 2?"
```
*   **Success:** It replies "4".
*   **Failure:** "Command not found" (Check Node install) or "Auth Error" (Check Login).

**Test 2: Check the Python Environment**
Run this command:
```bash
uv run python -c "print('The Senses are Online')"
```
*   **Success:** It prints "The Senses are Online".
*   **Failure:** Error message about python path (Check `uv` install).

---

## 4. Summary of Artifacts
At the end of this step, your folder `AI_Employee` should look like this:

```text
AI_Employee/
├── .venv/               <-- (Created by uv, holds Python libs)
├── .python-version      <-- (Pinned version, e.g., 3.12)
├── pyproject.toml       <-- (Configuration file, replaces requirements.txt)
├── hello.py             <-- (Default file created by init)
└── README.md
```
