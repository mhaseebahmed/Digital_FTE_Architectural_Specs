# Chapter 5: Reasoning Loops (The Cognitive Engine)

> *"How the AI moves from reading a file to formulating a plan."*

## 1. The Concept: The OODA Loop
In military strategy, the OODA Loop stands for **Observe, Orient, Decide, Act**. The Digital FTE uses this exact framework to process information. Unlike a chatbot that simply answers a question, the Agent must navigate a complex environment.

---

## 2. Key Terms & Concepts

| Term | Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Context Window** | The "Working Memory" of the AI. | The amount of text (files, handbook, logs) the AI can read at one time. |
| **System Prompt** | The "Personality" definition. | The hidden instruction that tells Claude: "You are a Digital Employee. Be professional." |
| **Chain of Thought** | Showing your work. | The process where the AI writes down its thinking steps *before* giving the final answer. |
| **Hallucination** | Confident but wrong answers. | The risk we mitigate by forcing the AI to check the `Company_Handbook.md`. |
| **Recursive Reading** | Reading a folder, then reading the files inside. | The ability of the Agent to "explore" the file system to find context. |

---

## 3. The 4-Step Reasoning Process

### Step 1: Ingestion (Observe)
The process begins when the Watcher drops a file into the `/Inbox`. The Agent "wakes up" and performs an ingestion scan.
*   **Target:** It reads the new file (e.g., `EMAIL_Invoice.md`).
*   **Context:** It *also* reads the `Company_Handbook.md` to understand the Rules of Engagement.
*   **History:** It may check the `Logs/` folder to see if it has dealt with this client before.

### Step 2: Orientation (Think)
The Agent must now understand **Intent**. It asks internal questions:
*   "Who is this email from?"
*   "Is this a known client?"
*   "What are they asking for?"
*   "Do I have a rule for this in the Handbook?"

> *Example:* "This is from Client A. They want an invoice. The Handbook says I must create a PDF and ask for approval."

### Step 3: Planning (Decide)
Before taking action, the Agent creates a **Plan**.
*   **The Plan File:** It creates a new file in the `/Plans` folder (e.g., `PLAN_Invoice_ClientA.md`).
*   **The Checklist:** It writes a step-by-step checklist of what it intends to do.
    1.  [ ] Calculate amount due.
    2.  [ ] Generate PDF using tool.
    3.  [ ] Draft email reply.
    4.  [ ] Request human approval.

### Step 4: Execution Strategy (Act)
The Agent looks at its "Hands" (Available Tools) and matches them to the plan.
*   "To generate a PDF, I need to use the `PDF_Creator` tool."
*   "To draft the email, I need to use the `File_Writer` tool."
It does **not** execute the final action yet; it simply prepares the artifacts.

---

## 4. The Decision Flowchart

```ascii
[ Start: File in /Inbox ]
          |
          v
[ Read: Company_Handbook.md ]
          |
          v
[ Question: Is this a known task? ]
     |                  |
     | (Yes)            | (No)
     v                  v
[ Create Plan ]    [ Flag for Human Review ]
     |
     v
[ Execute Draft ]
     |
     v
[ Create Approval Request ]
          |
          v
[ Stop & Wait ]
```

---

## 5. Failure Modes & Mitigations

### A. Context Overflow
*   **Problem:** The inbox has 50 files. The AI tries to read them all and runs out of "Memory" (Context Window).
*   **Mitigation:** The Orchestrator limits the AI to processing **one file at a time**.

### B. Ambiguity
*   **Problem:** The email says "Send it." The AI doesn't know what "it" is.
*   **Mitigation:** The System Prompt instructs the AI: *"If ambiguous, ask the human for clarification instead of guessing."*

### C. Logic Loops
*   **Problem:** The AI gets stuck trying to fix a file over and over.
*   **Mitigation:** A "Max Retry" counter. If it fails 3 times, it stops and alerts the human.
