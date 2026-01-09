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

## 2. The Setup Script (`scripts/setup_vault.py`)
Run this script to initialize your office.

```python
from pathlib import Path
import sys

# Hack to import from src without installing package yet
sys.path.append(str(Path(__file__).parent.parent))

from shared_foundation.config import settings
from shared_foundation.logger import setup_logger

logger = setup_logger("setup")

def build_office():
    root = settings.VAULT_PATH
    
    folders = [
        "00_Inbox", "10_Processing", "20_Done",
        "30_Pending_Approval", "40_Approved", "99_Logs/System", "System"
    ]

    for f in folders:
        (root / f).mkdir(parents=True, exist_ok=True)
        logger.info(f"📂 Created: {f}")

    # 1. Handbook
    handbook = root / "System" / "Company_Handbook.md"
    if not handbook.exists():
        handbook.write_text("# 📘 Company Handbook\n\n1. Safety First.\n2. Be concise.\n3. Protect PII.", encoding='utf-8')
        logger.info("📘 Created Handbook")

    # 2. Dashboard (The GUI)
    dashboard = root / "Dashboard.md"
    if not dashboard.exists():
        dashboard_content = """# 🖥️ Digital FTE Dashboard
## 🚀 System Status
- **Status:** Operational
- **Level:** Bronze Tier

## 📥 Active Tasks
![[Vault/00_Inbox]]

## 📊 Performance
- Total Tasks Completed: 0
- Revenue Generated: $0.00
"""
        dashboard.write_text(dashboard_content, encoding='utf-8')
        logger.info("🖥️ Created Dashboard.md")

if __name__ == "__main__":
    build_office()
    print("\n✅ Bronze Tier Office is ready for move-in.")
```

---

## 3. Execution
Run the setup script:
```bash
uv run scripts/setup_vault.py
```
**Verification:** Check that the `Vault` folder exists and contains all subfolders.