# Silver Tier 01: Gmail Watcher (API Implementation)

> *"Digital Eyes: Securely monitoring external communication channels via official APIs."*

## 1. The Requirement
The Silver Tier upgrades the Agent from "Local Folder" access to "External Service" access. The first and most critical channel is **Email**.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **OAuth2** | An open standard for access delegation. | How we give the Agent permission to read Gmail without giving it our Google Password. |
| **Scopes** | Specific permissions granted to an application. | We give the Agent `gmail.readonly` (it can see) but not `gmail.metadata` (it cannot delete). |
| **Access Token** | A temporary key used to talk to the API. | A key that expires every 60 minutes. The system must "Refresh" it automatically. |
| **Message Threading** | Grouping messages by a common subject/ID. | The Agent must see the *Conversation*, not just a single email, to understand context. |

---

## 3. The Implementation Architecture
This watcher inherits from the `BaseWatcher` we defined in the Bronze Tier.

### The Algorithm (The Pull Cycle)
1.  **Auth Check:** Before every check, verify if the token is valid. If expired, use the `refresh_token` to get a new one.
2.  **The Filter:** Query the Gmail API for: `is:unread label:INBOX`.
3.  **Deduplication:** Check the `Vault/99_Logs/System/processed_emails.json` to see if we have already handled this Message ID.
4.  **Content Extraction:** 
    *   Download the email Body (HTML or Plain Text).
    *   Download Attachments (PDFs, Images) to a temporary folder.
5.  **Normalization:** Generate the Markdown file in `Vault/00_Inbox`.

---

## 4. Normalization Schema (Markdown Jacket)
Every email must be converted into this standard format for the Brain:

```markdown
---
type: email
sender: "john@client.com"
subject: "Project Alpha Invoice"
received: "2026-10-12T09:00:00Z"
id: "msg_12345"
attachments: ["invoice.pdf"]
---

# Email Body
Hi, please see the attached invoice for last month's work.

# Suggested Actions
- [ ] Read invoice.pdf
- [ ] Draft payment request
```

---

## 5. Why this is Enterprise-Grade
1.  **Rate Limit Awareness:** The script includes a `time.sleep(settings.POLL_INTERVAL)` to ensure we do not hit Google's API limits.
2.  **Security:** Credentials are NEVER saved in the vault. They are stored in `src/shared/.env` and gitignored.
3.  **Resilience:** If the internet is down, the `with_retry` decorator (from Foundation) will wait and try again without crashing the entire agent.
4.  **Traceability:** Every email is assigned a `trace_id` that follows it through the Brain's planning phase.

---

## 6. The "Search vs. Listen" Decision
The PDF suggests **Polling** (checking every 60 seconds) rather than **Webhooks** (Google pushing to you). 
*   **Reason:** Webhooks require you to have a public IP address and a web server running. 
*   **Result:** Polling is more robust for a "Local-First" agent running on a laptop or home computer.
