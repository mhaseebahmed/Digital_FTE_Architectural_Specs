# Silver Tier 01: Gmail Watcher (API Implementation)

> *"Digital Eyes: Securely monitoring external communication channels via official APIs."*

## 1. The Requirement
The Silver Tier upgrades the Agent from "Local Folder" access to "External Service" access. We must implement a robust Gmail Poller that handles OAuth2 rotation automatically.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **OAuth2** | An open standard for access delegation. | How we give the Agent permission to read Gmail without giving it our Google Password. |
| **Refresh Token** | A long-lived credential used to obtain new access tokens. | The "Master Key" that allows the agent to stay logged in for months without asking you to re-login. |
| **MIME** | Multipurpose Internet Mail Extensions. | The complex format emails are sent in. We must parse this to get plain text. |

---

## 3. The Implementation Architecture

### A. The Dependency Stack
```bash
uv add google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

### B. The Authentication Helper (`gmail_auth.py`)
*This is the complex boilerplate code that solves the "Token Refresh" problem.*

```python
import os
import logging
from google.auth.transport.requests import Request
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']
TOKEN_PATH = 'Vault/System/token.json'
CREDS_PATH = 'Vault/System/credentials.json'

def get_gmail_service():
    creds = None
    
    # 1. Load existing token
    if os.path.exists(TOKEN_PATH):
        creds = Credentials.from_authorized_user_file(TOKEN_PATH, SCOPES)
        
    # 2. Refresh if expired
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            logging.info("🔄 Refreshing Expired Token...")
            creds.refresh(Request())
        else:
            # 3. New Login (First Run)
            logging.info("👤 Initiating New Login Flow...")
            flow = InstalledAppFlow.from_client_secrets_file(CREDS_PATH, SCOPES)
            creds = flow.run_local_server(port=0)
            
        # 4. Save new token
        with open(TOKEN_PATH, 'w') as token:
            token.write(creds.to_json())
            
    return build('gmail', 'v1', credentials=creds)
```

---

## 4. The Watcher Logic (`gmail_watcher.py`)

### The Algorithm (The Pull Cycle)
1.  **Auth:** Call `get_gmail_service()`.
2.  **Filter:** Query `is:unread label:INBOX`.
3.  **Deduplication:** Check against `processed_emails.json`.
4.  **Normalization:** Convert to Markdown.

### Code Snippet: Normalization
```python
def normalize_message(msg_id, service):
    msg = service.users().messages().get(userId='me', id=msg_id).execute()
    payload = msg['payload']
    headers = payload['headers']
    
    # Extract Headers
    subject = next(h['value'] for h in headers if h['name'] == 'Subject')
    sender = next(h['value'] for h in headers if h['name'] == 'From')
    
    # Extract Body (Simplified)
    snippet = msg.get('snippet', '')
    
    return f"""
# New Email Detected
- **From:** {sender}
- **Subject:** {subject}
- **Snippet:** {snippet}

@Claude: Read this email snippet. If it requires action, create a plan in /20_Plans.
"""
```

---

## 5. Why this is Enterprise-Grade
1.  **Automatic Rotation:** The `if creds.expired: creds.refresh()` logic ensures the agent can run 24/7 without human intervention.
2.  **Scopes:** We request `gmail.readonly`. Even if the Agent is hacked, it cannot delete your emails.
3.  **Local Storage:** The `token.json` is stored locally in the Vault (and gitignored), keeping your session secure.