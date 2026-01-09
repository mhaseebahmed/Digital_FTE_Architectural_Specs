# Implementation Guide 02: Vault Architecture

> *"Designing the Office: A strict State-Machine Directory Structure."*

## 1. The Architecture
We treat the file system as a **Database**. Each folder represents a specific "State" in a task's lifecycle.

| Folder | State | Permissions |
| :--- | :--- | :--- |
| `00_Inbox` | **Input** | Agent: Read-Only (Move to Processing) |
| `10_Processing` | **Working** | Agent: Read/Write |
| `20_Done` | **Archived** | Agent: Write-Only |
| `30_Pending_Approval` | **Locked** | Agent: Create-Only |
| `40_Approved` | **Unlocked** | Agent: Read-Only (Trigger Execution) |
| `99_Logs` | **Audit** | Agent: Append-Only |
| `System` | **Config** | Human: Read/Write |

---

## 2. The Setup Script (`setup_vault.py`)
Do not create folders manually. Use this script to ensure permission compliance and structure.

**File:** `setup_vault.py`
```python
import os
import logging
from pathlib import Path

# Configure Logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def create_digital_office():
    root = Path("Vault")
    
    structure = {
        "00_Inbox": "Drop incoming files here",
        "10_Processing": "Temporary workspace for the Agent",
        "20_Done": "Completed tasks archive",
        "30_Pending_Approval": "High-risk actions waiting for signature",
        "40_Approved": "Signed actions ready for execution",
        "99_Logs": "System audit trails",
        "System": "Configuration and Handbooks"
    }

    for folder, desc in structure.items():
        path = root / folder
        path.mkdir(parents=True, exist_ok=True)
        # Create a README in each folder to explain its purpose
        (path / "README.txt").write_text(desc)
        logging.info(f"✅ Created Directory: {path}")

    # Create Core Config Files
    _create_handbook(root / "System")

def _create_handbook(system_path):
    handbook_content = """# Digital FTE Handbook
## 1. Core Directives
*   **Safety:** You are a Level 1 Agent. You have DRAFT authority only.
*   **Privacy:** Do not upload sensitive PII to external cloud tools.
*   **Protocol:** Always check the /30_Pending_Approval folder before executing financial transactions.
    """
    (system_path / "Company_Handbook.md").write_text(handbook_content)
    logging.info("📘 Created Company Handbook")

if __name__ == "__main__":
    create_digital_office()
```

---

## 3. Execution
Run the setup script:
```bash
uv run setup_vault.py
```
**Verification:** Check that the `Vault` folder exists and contains all subfolders.
