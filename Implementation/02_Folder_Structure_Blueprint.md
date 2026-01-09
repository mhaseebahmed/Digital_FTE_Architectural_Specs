# Implementation Guide 02: Folder Structure Blueprint

> *"Designing the Office: Where the Digital Employee lives and works."*

## 1. The Vault Architecture
We do not scatter files randomly. We build a strict **Hierarchy of State**. Every folder represents a stage in the lifecycle of a task.

### The Standard Hierarchy
You must create a folder named `Vault` inside your `AI_Employee` project. Inside `Vault`, create exactly these subfolders:

```text
AI_Employee/
└── Vault/
    ├── 00_Inbox/           <-- (The Entry Point. Raw data lands here.)
    ├── 10_Needs_Action/    <-- (The Queue. Orchestrator moves valid tasks here.)
    ├── 20_Plans/           <-- (The Thinking. Claude writes step-by-step plans here.)
    ├── 30_Pending_Approval/<-- (The Pause. Dangerous tasks wait here for signature.)
    ├── 40_Approved/        <-- (The Trigger. Moving a file here launches the action.)
    ├── 50_Done/            <-- (The Archive. Completed tasks.)
    ├── 99_Logs/            <-- (The Black Box. JSON records of every action.)
    └── System/             <-- (The Config. Handbooks and Prompts.)
```

---

## 2. The Core Configuration Files
Inside the `Vault/System/` folder, you must create two critical Markdown files.

### A. `Company_Handbook.md`
This is the "Constitution" for your AI. It defines the rules it must never break.

**Template:**
```markdown
# Company Handbook for Digital FTE

## 1. Core Directives
*   **Privacy:** Never upload client PII (Personally Identifiable Information) to public servers.
*   **Tone:** Professional, concise, and polite.
*   **Safety:** You have DRAFT authority only. You must request APPROVAL for any financial transaction over $0.01.

## 2. Tools & Protocols
*   **Invoicing:** Use the PDF generation tool. Save drafts to /Needs_Action.
*   **Email:** Draft replies in Markdown. Do not send without approval.
```

### B. `SKILL.md` (Optional but Recommended)
This acts as a "Cheat Sheet" for specific tasks.

**Template:**
```markdown
# Skill: Monthly Audit
1.  Read all files in /99_Logs from the last 30 days.
2.  Sum the 'amount' field where type='expense'.
3.  Compare against Budget ($500).
4.  Write report to /00_Inbox.
```

---

## 3. Automation Script (setup.py)
To create this structure instantly, create a file named `setup_vault.py` in your root folder and run it with `uv run setup_vault.py`.

```python
import os
from pathlib import Path

def create_office():
    root = Path("Vault")
    folders = [
        "00_Inbox",
        "10_Needs_Action",
        "20_Plans",
        "30_Pending_Approval",
        "40_Approved",
        "50_Done",
        "99_Logs",
        "System"
    ]

    for folder in folders:
        (root / folder).mkdir(parents=True, exist_ok=True)
        print(f"✅ Created: {root / folder}")

    # Create Handbook
    handbook_path = root / "System" / "Company_Handbook.md"
    if not handbook_path.exists():
        handbook_path.write_text("# Company Handbook\n\n1. Safety First.")
        print(f"✅ Created: {handbook_path}")

if __name__ == "__main__":
    create_office()
```
