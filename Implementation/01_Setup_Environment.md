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

1.  Create directory:
    ```bash
    mkdir AI_Employee
    cd AI_Employee
    ```

2.  Initialize Project:
    ```bash
    uv init
    ```
    *This creates a `pyproject.toml` and `.python-version` file.*

3.  **Critical Security Step:** Create a `.gitignore` file immediately.
    *   Create a file named `.gitignore`.
    *   Add the following content:
        ```text
        .venv/
        .env
        Vault/99_Logs/
        Vault/System/credentials.json
        Vault/System/token.json
        __pycache__/
        .vscode/
        .idea/
        ```

4.  **Cleanup:** Remove the default `hello.py` file created by `uv init`.
    ```bash
    rm hello.py
    ```

### Step 3: Install Node.js & Claude Code
1.  **Install Node.js:** Download the **LTS** version from [nodejs.org](https://nodejs.org/).
2.  **Install Claude Code:**
    ```bash
    npm install -g @anthropic/claude-code
    ```

### Step 4: Authentication & Configuration
For enterprise stability, we use Environment Variables instead of browser sessions where possible.

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

## 3. Verification Checklist
Do not proceed until all 3 pass.

*   [ ] `uv --version` returns 0.4.x or higher.
*   [ ] `node --version` returns v20.x or higher.
*   [ ] `claude --version` returns the CLI version.