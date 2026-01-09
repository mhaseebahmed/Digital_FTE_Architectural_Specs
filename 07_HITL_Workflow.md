# Chapter 7: Human-in-the-Loop (HITL) Workflow

> *"The Safety Valve: How to grant autonomy without losing control."

## 1. The Core Problem: The "Runaway Robot"
A fully autonomous agent is dangerous.
*   **Risk:** Hallucination. The AI might misread "$5.00" as "$5000" and send the payment.
*   **Risk:** Social Faux Pas. The AI might send a rude email to a client.
*   **The Goal:** We want the AI to do 99% of the work (Preparation) but 0% of the risk (Execution).

---

## 2. Key Terms & Concepts

| Term | Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Draft Authority** | The right to *prepare* an action. | The Agent can write the email or fill out the check. |
| **Execution Authority** | The right to *finalize* an action. | Only the Human can click "Send" or sign the check. |
| **Asynchronous Approval** | Decoupling the "Ask" from the "Answer." | The Agent doesn't wait for you to stare at the screen. It leaves a note and moves on to other work. |
| **Atomic Transaction** | A single, indivisible operation. | Moving a file is atomic. It's either in Folder A or Folder B. There is no confusing "halfway" state. |

---

## 3. The File-Based Approval Pattern
Instead of building a complex web app with "Approve" buttons, we use the file system.

### Step 1: The Request (Created by Agent)
When the Agent needs permission, it creates a Markdown file in `/Vault/Pending_Approval`.
*   **Filename:** `PAYMENT_ClientA_Invoice123.md`
*   **Content (Structured Metadata):**
    ```yaml
    Action: Send Payment
    Amount: $500.00
    Recipient: Client A (client@example.com)
    Reason: Monthly Service Fee
    Status: Pending
    Expire: 24h
    ```
*   **Attachment:** It may include a link to the generated PDF.

### Step 2: The Pause (System State)
The Agent **stops working on this specific task**.
*   It logs: "Waiting for approval on Payment #123."
*   It goes back to sleep or picks up a *different* task from the Inbox.
*   The system is now in a "Holding Pattern."

### Step 3: The Review (Human Action)
*   You open your Obsidian Dashboard.
*   You see a list of files in `Pending_Approval`.
*   You open one, read the YAML summary.
*   **Decision:**
    *   **Approve:** You drag the file to the `/Vault/Approved` folder.
    *   **Reject:** You drag the file to the `/Vault/Rejected` folder.
    *   **Modify:** You edit the file (e.g., change $500 to $450) and *then* move it to Approved.

### Step 4: The Execution (Orchestrator Trigger)
The Orchestrator watches the `/Approved` folder.
*   **Trigger:** It sees a new file land in `/Approved`.
*   **Action:** It wakes up Claude.
*   **Instruction:** "Execute the action described in this file."
*   **Result:** Claude reads the *approved* file and finally calls the `payment_mcp` tool.

---

## 4. The Safety Diagram

```ascii
[ Agent Reasoner ]
       |
       | (Wants to Pay)
       v
[ Create File: /Pending/Pay.md ] ----> [ STOP: Wait Mode ]
                                              |
                                              | (Time Passes...)
                                              v
                                    [ Human Reviewer ]
                                    (Reads Pay.md)
                                         /     \
                                     (No)       (Yes)
                                     /             \
                        [ Move to /Rejected ]   [ Move to /Approved ]
                                                        |
                                                        v
                                                [ Orchestrator ]
                                                (Detects File)
                                                        |
                                                        v
                                                [ Agent Executor ]
                                                (Calls Bank API)
```

---

## 5. Why File Movement?
Why not just type "Yes" in a chat?
1.  **Intent:** Dragging a file is a deliberate physical action. It's harder to do by accident than typing "ok."
2.  **Audit Trail:** The file itself becomes the permanent record. We keep the "Approved" file in the `Logs` folder forever. It proves *you* signed off on it.
3.  **Simplicity:** No database, no server, no login page. Just folders.
