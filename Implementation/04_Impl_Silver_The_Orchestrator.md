# Implementation Guide 04: Silver Tier (The Orchestrator)

> *"The Manager: coordinating Multiple Watchers and Human Approvals."*

## 1. Objective
Merge the "File Watcher" with a "Gmail Poller" and implement the "Approval Workflow."

**Dependencies:**
```bash
uv add google-api-python-client google-auth-oauthlib python-dotenv playwright
```

---

## 2. The Gmail Poller (`tier_2_silver/gmail.py`)
This script must run independently. It fetches email and drops it into `00_Inbox`.

```python
import os
from tenacity import retry, wait_fixed, stop_after_attempt
# Google Imports...
from shared_foundation.logger import setup_logger
from shared_foundation.config import settings

class GmailSentinel:
    def check_for_mail(self):
        # ... (See repo for full OAuth logic)
        pass
```

---

## 3. The Approval Watcher (`tier_2_silver/approval.py`)
This watches the `/40_Approved` folder.

```python
from tier_1_bronze.claude_client import ClaudeClient

class ApprovalHandler(FileSystemEventHandler):
    def on_moved(self, event):
        # Trigger Claude with "EXECUTE" mode
        pass
```

---

## 4. The Orchestrator (`src/main.py`)
This script launches all subsystems as separate processes. This prevents one crash from killing the whole agent.

```python
import multiprocessing
from tier_1_bronze.filesystem import RobustHandler
from tier_2_silver.approval import ApprovalHandler
from tier_2_silver.gmail import run_gmail_loop
from tier_2_silver.whatsapp import WhatsAppSentinel

def main():
    processes = [
        multiprocessing.Process(target=run_filesystem_watcher, name="FS"),
        multiprocessing.Process(target=run_approval_watcher, name="Approval"),
        multiprocessing.Process(target=run_gmail_loop, name="Gmail"),
        multiprocessing.Process(target=run_whatsapp_sentinel, name="WhatsApp"),
    ]
    # ...
```
