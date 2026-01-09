# Silver Tier 02: WhatsApp Watcher (Browser Automation)

> *"The Digital Ghost: Automating platforms that were designed only for humans."*

## 1. The Requirement
WhatsApp is a critical communication channel for modern business, yet it lacks an open API for personal accounts. We must use **Headless Browser Automation** to bridge this gap.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Playwright** | A framework for Web Testing and Automation. | The library we use to "control" a Chrome browser via Python. |
| **Headless Mode** | Running a browser without a graphical user interface. | Allows the Agent to see the web without popping up a window on your screen. |
| **Session Storage** | Local data saved by the browser (Cookies, LocalStorage). | How we save your WhatsApp login so you only scan the QR code once. |
| **Selector (CSS/XPath)** | A string used to identify a specific element on a webpage. | How we tell the script: "Look at the little green circle with a number in it." |

---

## 3. The Implementation Architecture

### Step 1: Session Management
We do not login on every run.
1.  **First Run:** Script opens a Visible Browser. You scan the QR code.
2.  **Save:** Script saves the `session.json` (Auth tokens) to a secure folder.
3.  **Subsequent Runs:** Script loads `session.json` and launches in **Headless** mode.

### Step 2: The Observation Loop
1.  **Navigation:** Go to `web.whatsapp.com`.
2.  **Wait for Load:** Wait for the chat list to appear.
3.  **Scan:** Look for the HTML class `.unread-count` (or similar).
4.  **Action:** 
    *   If found, click the chat.
    *   Read the last 5 messages.
    *   Normalize to Markdown.

---

## 4. Normalization Schema (Chat Transcript)
Conversations are different from Emails. They are snippets of text.

```markdown
---
type: whatsapp
contact: "+1 555-0123"
time: "2026-10-12T09:15:00Z"
---

# Chat Transcript
- [09:10] Client: "Hey, did you get my invoice?"
- [09:12] Client: "I sent it to your email."

# Contextual Note
The Agent should check the /00_Inbox for a corresponding Gmail file.
```

---

## 5. Why this is Enterprise-Grade
1.  **Stealth & Politeness:** We implement random delays (`0.5s` to `2.0s`) between actions to simulate human movement, preventing WhatsApp from flagging the account as a bot.
2.  **Robust Selectors:** We use "Text-based" selectors (e.g., `contains("Unread")`) rather than fragile CSS paths that change when WhatsApp updates its UI.
3.  **Health Checks:** If the script finds itself at the "Login Page" instead of the "Chat Page," it alerts the Human that the session has expired.
4.  **Efficiency:** The script only "scrapes" the message if it hasn't been processed before (checking against a local DB).
