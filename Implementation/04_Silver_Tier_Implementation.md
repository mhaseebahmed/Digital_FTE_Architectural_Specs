# Implementation Guide 04: Silver Tier (The Assistant)

> *"The Safety Upgrade: Adding eyes (Gmail) and brakes (Approval)."*

## 1. Objective
To expand the agent's capabilities to include **Email Monitoring** and **Human Authorization**.

**Deliverables:**
1.  `gmail_watcher.py` (The API Poller).
2.  `orchestrator.py` (The Manager that runs both watchers).
3.  Integration of the `Pending_Approval` workflow.

---

## 2. Installing Dependencies
We need the Google Client Library.

```bash
uv add google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

---

## 3. The Gmail Watcher (`gmail_watcher.py`)
*Note: You need a `credentials.json` from Google Cloud Console for this.*

**Logic Flow:**
1.  Authenticate with Google.
2.  **Poll:** Check for messages with `q="is:unread"`.
3.  **Normalize:** For each message, create a Markdown file in `Vault/00_Inbox`.
    *   Filename: `EMAIL_{message_id}.md`
    *   Content: `From: ... 
 Subject: ... 
 Body: ...`
4.  **Sleep:** Wait 60 seconds before checking again.

---

## 4. The Human-in-the-Loop Logic
We must update our `Company_Handbook.md` to enforce safety.

**Update Handbook:**
```markdown
## Safety Protocol
If the task involves:
1.  Sending an email to an external address.
2.  Making a financial transaction.

You MUST:
1.  Draft the file.
2.  Save it to `/Vault/30_Pending_Approval`.
3.  STOP. Do not execute.
```

## 5. The Approval Watcher
We need a *second* file watcher that listens to the `/Approved` folder.

**Logic Flow:**
1.  Watch `/Vault/40_Approved`.
2.  **Event:** A file is moved here (by you).
3.  **Action:** Wake up Claude.
4.  **Prompt:** "This file has been APPROVED. Execute the action described inside."

---

## 6. The Orchestrator
We combine the pieces.

```python
# orchestrator.py
import threading
from watcher import start_file_watcher
from gmail_watcher import start_gmail_poller

if __name__ == "__main__":
    # Run File Watcher in Thread 1
    t1 = threading.Thread(target=start_file_watcher)
    
    # Run Gmail Poller in Thread 2
    t2 = threading.Thread(target=start_gmail_poller)
    
    t1.start()
    t2.start()
    
    print("Silver Tier Agent is Online (Dual-Core Mode)")
```
