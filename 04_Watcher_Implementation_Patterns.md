# Chapter 4: Watcher Implementation Patterns

> *"The Senses of the Agent: Engineering the 'Wake Word' for passive AI systems."*

## 1. The Engineering Challenge
Large Language Models (LLMs) are **Stateless** and **Passive**.
*   **Stateless:** They possess no memory of past interactions unless context is re-injected.
*   **Passive:** They execute only upon request (Request/Response Cycle).
*   **The Problem:** Real-world data streams (Email, Webhooks, Logs) are **Asynchronous**. They occur at unpredictable intervals.
*   **The Solution:** We must implement **Daemons**—persistent background processes that bridge the gap between the active world and the passive brain.

---

## 2. Technical Lexicon

| Term | Technical Definition | Conceptual Analogy |
| :--- | :--- | :--- |
| **Daemon** | A computer program that runs as a background process, rather than being under the direct control of an interactive user. | A Security Guard who stays at their post 24/7. |
| **Event Loop** | A programming construct that waits for and dispatches events or messages in a program. | The Guard scanning the room repeatedly: "Anything new? No. Anything new? No." |
| **Polling** | Synchronous sampling of an external status. | Calling the Post Office every 5 minutes to ask "Do I have mail?" |
| **Interrupt (Signal)** | An asynchronous notification sent to a process. | The Doorbell. It forces the Guard to stop scanning and open the door immediately. |
| **DOM (Document Object Model)** | The data representation of the objects that comprise the structure and content of a document on the web. | The skeleton of a website that a script can read. |

---

## 3. Pattern I: The File System Watcher (Event-Driven)
*Implementation: `watchdog` library.*

This is the most efficient pattern because it is **Interrupt-Driven**, not Polling-Driven.

### The Algorithm
1.  **Subscription:** The script registers a "Callback Function" with the Operating System Kernel (e.g., `inotify` on Linux or `ReadDirectoryChangesW` on Windows).
2.  **Idle State:** The script suspends execution (blocking I/O), consuming near-zero CPU.
3.  **Kernel Event:** When a file is modified, the OS Kernel wakes the script.
4.  **Payload:** The Event object contains the `file_path` and `event_type` (Created, Modified, Deleted).
5.  **Trigger:** The script accepts this payload and invokes the Orchestrator.

### Use Case
*   **Manual Drops:** Dragging a PDF into the folder.
*   **System Integration:** Another program saving a log file.

---

## 4. Pattern II: The API Watcher (Polling-Based)
*Implementation: `google-api-python-client`.*

Used for external services (Gmail, Slack) where we do not own the server and cannot install an event hook.

### The Algorithm (The Polling Loop)
We must implement a specific cycle to avoid being banned.

1.  **Auth Handshake:** Exchange API Keys for a temporary Access Token (OAuth2).
2.  **Fetch State:** Query the endpoint: `GET /messages?q=is:unread`.
3.  **Diff Engine:** Compare the returned list against a local database of "Processed IDs."
    *   *New ID found:* Download content.
    *   *Old ID found:* Ignore.
4.  **Normalization:** Convert the JSON payload (nested dictionaries) into a flat Markdown file.
5.  **Backoff Strategy (Sleep):**
    *   **Crucial Step:** The script *must* sleep for N seconds (e.g., 60s).
    *   **Rate Limiting:** APIs enforce limits (e.g., "100 requests per minute"). Without sleep, the script will crash with `HTTP 429: Too Many Requests`.

---

## 5. Pattern III: The Browser Watcher (Headless Automation)
*Implementation: Playwright / Selenium.*

Used for "Human-First" interfaces (WhatsApp, Legacy Banking) that lack a public API.

### The Architecture
*   **Headless Browser:** A full instance of Chromium running without the graphical user interface (GUI) to save resources.
*   **Selector Engine:** The script uses CSS Selectors (e.g., `.unread-bubble`) to locate elements in the DOM.

### The Algorithm
1.  **Session Injection:** Load saved Cookies to bypass 2FA/QR Codes.
2.  **DOM Observation:** Instead of polling the server, we poll the *local HTML page*.
    *   `page.wait_for_selector(".message-in")`
3.  **Scraping:**
    *   Extract `innerText` from the element.
    *   Sanitize (remove emojis/HTML tags).
4.  **Persistence:** Write to `/Inbox`.

### Engineering Risk: Fragility
This pattern is "Brittle." If the website developer changes the CSS class name from `.unread-bubble` to `.new-msg`, the selector fails, and the script crashes. This requires exception handling and frequent maintenance.

---

## 6. Summary: Architectural Decision Matrix

| Pattern | Trigger Mechanism | CPU Cost | Reliability | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **File Watcher** | OS Interrupt (Push) | Low | High | Local Files |
| **API Poller** | HTTP Request (Pull) | Medium | High | Gmail, Stripe |
| **Browser Scraper**| DOM Mutation | High | Low | WhatsApp |