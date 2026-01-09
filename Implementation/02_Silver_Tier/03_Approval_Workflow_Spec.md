# Silver Tier 03: Human-in-the-Loop (HITL) Workflow

> *"The Final Signature: Implementing a foolproof state-machine for high-risk actions."*

## 1. The Requirement
The Agent is now capable of sending emails and making payments. We must implement a "Physical Gateway" that requires a human action to complete these tasks.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **State Machine** | A mathematical model of computation that is in exactly one of a finite number of states at any given time. | The Task's status (Pending vs. Approved). |
| **Atomic Operation** | An operation that is completed in its entirety or not at all. | Moving a file. It is the "Trigger" for execution. |
| **Metadata Verification** | Checking the internal headers of a file to ensure it is valid. | Ensuring the "Approved" file contains the correct `trace_id`. |
| **HITL** | Human-in-the-Loop. | The design pattern where an AI provides suggestions and a Human provides the final decision. |

---

## 3. The Implementation Architecture

### Step 1: The Request (Created by Claude)
When Claude identifies a task requiring approval, it generates a "Proposal."
*   **Action:** Write to `Vault/30_Pending_Approval/REQ_123.md`.
*   **Schema (YAML):**
    ```yaml
    id: REQ_123
    action: SEND_EMAIL
    to: "boss@client.com"
    content: "Invoice is attached."
    status: PENDING
    ```

### Step 2: The Approval Watcher
A separate Python Watcher (The "Executor") monitors the `Vault/40_Approved` folder.
1.  **Event:** File `REQ_123.md` is moved into `/40_Approved`.
2.  **Validation:**
    *   Does the file exist?
    *   Is the YAML valid?
    *   Does the `id` match an existing log?
3.  **Execution:**
    *   Call the **MCP Server** to send the email.
    *   Log the outcome.
4.  **Archive:** Move to `Vault/20_Done`.

---

## 4. Why this is Enterprise-Grade
1.  **No Single Point of Failure:** The script that *proposes* the action is different from the script that *executes* the action. Even if Claude is "hacked" or hallucinates, it cannot bypass the physical folder restriction.
2.  **Audit Trail:** The file itself, including any edits you made during approval, is archived in `/20_Done`. We know exactly what was approved and when.
3.  **Non-Technical Interface:** The human doesn't need to know Python. They just need to know how to "Drag and Drop" a file.
4.  **Resilience:** If the power goes out while a task is "Pending," it stays in the folder. When the system reboots, it is still there waiting for you.

---

## 5. Security Note: Folder Permissions
In a corporate environment, the Agent's OS user should have:
*   `Write` access to `/30_Pending_Approval`.
*   `Read-Only` access to `/40_Approved`.
This ensures the Agent can **ask** for permission but cannot **grant** itself permission.
