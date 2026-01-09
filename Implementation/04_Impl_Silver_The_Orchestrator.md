# Implementation Guide 04: Silver Tier (The Orchestrator)

> *"The Manager: coordinating Multiple Watchers and Human Approvals."*

## 1. Objective
Merge the "File Watcher" with a "Gmail Poller" and implement the "Approval Workflow."

**Dependencies:**
```bash
uv add google-api-python-client google-auth-oauthlib python-dotenv
```

---

## 2. The Gmail Poller (`gmail_poller.py`)
This script must run independently. It fetches email and drops it into `00_Inbox`.

*Note: Requires `credentials.json` in `Vault/System/` (Gitignored).*

```python
import time
import logging
from pathlib import Path
# (Imports for Google API would go here - simplified for Architecture Guide)

def poll_gmail():
    logging.info("📧 Checking Gmail...")
    # Simulation of API Call
    new_emails = [] 
    
    for email in new_emails:
        # NORMALIZE: Convert JSON to Markdown
        content = f"# From: {email['sender']}\n{email['body']}"
        filename = f"EMAIL_{email['id']}.md"
        
        # SAVE TO INBOX (Triggers the File Watcher from Guide 03)
        Path(f"Vault/00_Inbox/{filename}").write_text(content)
        logging.info(f"kv Dropped email {filename} into Inbox")

if __name__ == "__main__":
    while True:
        poll_gmail()
        time.sleep(60) # Rate Limit Protection
```

---

## 3. The Approval Watcher (`approval_watcher.py`)
This watches the `/40_Approved` folder.

**Logic:**
1.  Detect file in `40_Approved`.
2.  Read the file (It contains the *Approved* Plan).
3.  Tell Claude: "EXECUTE this plan."

```python
import shutil
import subprocess
import logging
from watchdog.events import FileSystemEventHandler

class ApprovalHandler(FileSystemEventHandler):
    def on_created(self, event):
        logging.info(f"✅ APPROVAL DETECTED: {event.src_path}")
        # Trigger Claude with "EXECUTE" mode
        subprocess.run(["claude", "-p", event.src_path, "Execute this approved plan immediately."])
        # Archive to Done
        shutil.move(event.src_path, "Vault/20_Done")
```

---

## 4. The Orchestrator (`orchestrator.py`)
This script launches all subsystems as separate processes. This prevents one crash from killing the whole agent.

```python
import subprocess
import time
import sys
from pathlib import Path

def start_system():
    procs = []
    
    # Path Safety: Ensure we find the scripts relative to THIS file
    base_dir = Path(__file__).parent.resolve()
    watcher_script = base_dir / "watcher.py"
    gmail_script = base_dir / "gmail_poller.py"
    
    # Launch File Watcher (Guide 03)
    p1 = subprocess.Popen([sys.executable, str(watcher_script)])
    procs.append(p1)
    
    # Launch Gmail Poller
    p2 = subprocess.Popen([sys.executable, str(gmail_script)])
    procs.append(p2)
    
    print("🌟 Silver Tier Orchestrator Online. Press Ctrl+C to stop.")
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("🛑 Shutting down...")
        for p in procs:
            p.terminate()

if __name__ == "__main__":
    start_system()
```