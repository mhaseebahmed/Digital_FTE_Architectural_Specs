# Chapter 4: Watcher Implementation Patterns

> *"The Senses of the Agent: How to solve the 'Wake Word' problem in passive AI systems."*

## 1. The Core Problem: Passive vs. Active
Large Language Models (LLMs) are **Stateless and Passive**. They are like a brain in a jar that only thinks when you tap on the glass.
*   **The Problem:** An AI cannot "wait" for an email. It has no internal clock or sensory nerves.
*   **The Solution:** **Watchers.** These are lightweight "Sentinels" (Python scripts) that monitor the outside world and "tap on the glass" for the AI.

---

## 2. Pattern I: The File System Watcher (Local Ingestion)
This is the simplest and most robust pattern. It treats a local folder as the "Entry Point" for all work.

### The "Why"
To allow humans to manually assign tasks to the AI by simply dragging and dropping files (PDFs, Images, Text).

### The "How" (Technical Logic)
It uses a library called `watchdog` to hook into the Operating System's file events.
1.  **Idle:** The script sits in a low-power state.
2.  **Interrupt:** You drop `report.pdf` into `/Inbox`. The OS sends a signal to the script.
3.  **Action:** The script captures the file name and path.
4.  **Handoff:** It triggers the Orchestrator: "Boss, someone just dropped a file. Wake up Claude."

### Workflow Diagram
```ascii
[ User ] --(Drag & Drop)--> [ /Inbox Folder ]
                                  |
                                  v
                        [ watchdog.Observer ] --(Signal)--> [ Orchestrator ]
```

---

## 3. Pattern II: The API Watcher (The Poller)
This pattern is used for services that have official "doors" (APIs), like Gmail, Outlook, or Slack.

### The "Why"
To allow the AI to monitor communication channels without a human needing to copy-paste messages.

### The "How" (Technical Logic)
Since we can't "hook" into Google's servers, we use **Polling**.
1.  **Interval:** Every X minutes, the script wakes up.
2.  **Query:** It asks: "Give me all `is:unread` messages since the last check."
3.  **Normalization:** This is the critical step. The script takes the complex JSON data from the API and converts it into a simple **Markdown File**.
    *   *Input:* Complex code.
    *   *Output:* `EMAIL_Subject.md` containing "From, Date, Body."
4.  **Drop:** It saves this file into the `/Inbox`. This triggers the **File System Watcher** (Pattern I).

### The Value
This "Normalizes" the world. Whether it's an email or a Slack message, it becomes a **Text File**. This makes the Brain's job much easier because it only has to learn one thing: how to read text files.

---

## 4. Pattern III: The Browser Watcher (Headless Automation)
This is the most complex pattern, used for services without APIs (e.g., WhatsApp Web, certain Banking Portals).

### The "Why"
To automate platforms that were designed *only* for humans to look at.

### The "How" (Technical Logic)
It uses a tool called **Playwright** to run a "Headless Browser" (Chrome without a window).
1.  **Simulation:** The script logs into `web.whatsapp.com`.
2.  **DOM Monitoring:** It scans the "Document Object Model" (the code behind the page) for specific markers, like the green "Unread" notification bubble.
3.  **Scraping:** If it finds a bubble, it "clicks" it virtually and reads the text on the screen.
4.  **Persistence:** It saves the chat as a file in `/Inbox`.

### Comparison Table
| **Pattern** | **Stability** | **Complexity** | **Typical Use Case** |
| :--- | :--- | :--- | :--- |
| **File System** | 10/10 (High) | Low | Local file processing, Manual tasks. |
| **API Poller** | 9/10 (High) | Medium | Gmail, Slack, Stripe, Calendar. |
| **Browser Scraper**| 4/10 (Low) | High | WhatsApp, Legacy Bank Portals. |

---

## 5. Summary: The "Sense" Philosophy
In this architecture, Watchers do **zero thinking**.
*   They don't decide if an email is important.
*   They don't decide how to reply.
*   **They only fetch and format.**

By keeping the "Senses" dumb and the "Brain" smart, we make the system much easier to debug and maintain.
