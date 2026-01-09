# Chapter 4: Watcher Implementation Patterns

> *"The Senses of the Agent: Bridging the gap between a passive Brain and an active World."*

## 1. The Core Problem
Large Language Models (AI Brains) are **Passive Systems**.
*   They are **Stateless:** They do not "remember" the previous second unless you feed it back to them.
*   They are **Reactive:** They only speak when spoken to.
*   **The Challenge:** Real-world events (Emails, WhatsApps) are **Asynchronous**. They happen at random times. We need a bridge.

---

## 2. Key Terms & Concepts

| Term | Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Daemon** | A background process that runs continuously, waiting for requests. | Our Watcher scripts are "Daemons" that never sleep. |
| **Polling** | Repeatedly checking a resource ("Are we there yet?"). | Used for Gmail/APIs. We ask "Any new mail?" every 60 seconds. |
| **Webhook** | A "Push" notification. The server calls *you* when data arrives. | Ideal, but rarely available for local-first setups due to firewall issues. |
| **Headless** | Running a browser (Chrome) without the graphical window. | Used to automate WhatsApp Web invisibly in the background. |
| **DOM** | Document Object Model. The HTML structure of a webpage. | We inspect the DOM to find the "Unread Message" bubble on WhatsApp. |
| **Normalization** | Converting different data formats into one standard format. | We turn complex JSON (Gmail) and HTML (WhatsApp) into simple Markdown files. |

---

## 3. Pattern I: The File System Watcher (Local Ingestion)
*The Foundation of the entire architecture.*

### The Mechanism
It relies on the Operating System's kernel (ReadDirectoryChangesW on Windows, inotify on Linux). It does not "poll"; it waits for an **Interrupt Signal**.

### Detailed Workflow
1.  **State:** The script is asleep (blocked on I/O), consuming 0% CPU.
2.  **Trigger:** User drags `invoice.pdf` into `C:\Vault\Inbox`.
3.  **Signal:** The OS Kernel sends an event to the Python script.
4.  **Handler:** The `on_created(event)` function fires.
5.  **Logic:**
    *   Check if it's a file (not a folder).
    *   Wait 500ms (to ensure the copy operation finished).
    *   **Action:** Trigger the Orchestrator to launch Claude.

---

## 4. Pattern II: The API Watcher (The Poller)
*Used for: Gmail, Outlook, Slack, Stripe.*

### The Mechanism
Since we cannot receive "Push" notifications easily on a local laptop (without exposing ports to the internet), we use **Interval Polling**.

### Detailed Workflow (Pseudo-Code)
```python
while True:
    # 1. Connect to Google
    service = build('gmail', 'v1', credentials=creds)
    
    # 2. Ask for IDs of unread messages
    results = service.users().messages().list(q="is:unread label:INBOX").execute()
    
    # 3. If new messages exist:
    for message in results:
        # 4. Fetch full content
        msg_content = service.users().messages().get(id=message['id']).execute()
        
        # 5. NORMALIZE (Critical Step)
        # Convert JSON -> Markdown
        markdown_content = f"""
        # Email From: {msg_content['sender']}
        # Subject: {msg_content['subject']}
        {msg_content['body']}
        """
        
        # 6. Save to Inbox (Triggers Pattern I)
        save_file(f"/Inbox/EMAIL_{id}.md", markdown_content)
        
    # 7. Sleep (Rate Limit Protection)
    time.sleep(60) 
```

### Failure Modes
*   **Rate Limiting:** If you poll too fast (e.g., every 1 second), Google will ban your API key.
*   **Token Expiry:** OAuth tokens expire. The script must handle "Token Refresh" logic automatically.

---

## 5. Pattern III: The Browser Watcher (Headless Scraper)
*Used for: WhatsApp, Legacy Banks, Portals.*

### The Mechanism
We use **Playwright** or **Selenium** to drive a real Chrome instance. This is "fragile" automation because it depends on the visual layout of a website.

### Detailed Workflow
1.  **Initialization:** Launch Chrome (Headless). Load cookies (session) to avoid scanning QR code again.
2.  **Navigation:** Go to `web.whatsapp.com`.
3.  **Wait:** Wait for the specific CSS selector `.unread-count` to appear in the DOM.
4.  **Action:**
    *   Click the unread chat.
    *   Scrape all text `div` elements.
    *   Format as Markdown.
    *   Save to `/Inbox`.
5.  **Cleanup:** Close the browser context to free up RAM.

### Failure Modes
*   **DOM Change:** If WhatsApp renames the "unread" class to "new-message", the script breaks immediately.
*   **Session Timeout:** If the phone disconnects, the web session dies.

---

## 6. Summary Comparison

| Feature | File Watcher | API Poller | Browser Scraper |
| :--- | :--- | :--- | :--- |
| **Reliability** | **High** (Native OS) | **High** (Stable Contract) | **Low** (UI Dependent) |
| **Speed** | Instant (Interrupt) | Delayed (Interval) | Slow (Page Load) |
| **Maintenance** | Zero | Low (Auth tokens) | High (CSS fixes) |
| **Cost** | Free | Free (usually) | High (CPU/RAM heavy) |