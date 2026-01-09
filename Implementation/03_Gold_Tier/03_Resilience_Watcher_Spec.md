# Gold Tier 03: The Watchdog (Self-Healing Architecture)

> *"The Survival Instinct: Ensuring the Agent remains operational through network failures, system reboots, and code crashes."*

## 1. The Requirement
An autonomous system is useless if it stops working when you aren't looking. The Gold Tier Agent must have a "Health Monitor" that ensures 99.9% uptime.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Zombie Process** | A process that has completed execution but still has an entry in the process table. | A Watcher that has crashed but hasn't "closed," blocking the port. |
| **Heartbeat** | A periodic signal sent by a script to indicate it is still alive. | The Watchers writing their "Last Active" timestamp to a hidden file. |
| **Graceful Degradation** | The ability of a system to maintain limited functionality even when a portion of it has failed. | If Gmail is down, the File Watcher still works. |
| **Self-Healing** | The ability of a system to detect a failure and automatically restart the failed component. | The Watchdog restarting a crashed WhatsApp Sentinel. |

---

## 3. The Implementation Architecture

### Step 1: The Heartbeat Protocol
Every script (Gmail Watcher, File Watcher, etc.) must write its status to a shared health file: `Vault/System/Health.json`.
```json
{
  "gmail_watcher": "2026-10-12T09:30:00Z",
  "file_watcher": "2026-10-12T09:30:05Z"
}
```

### Step 2: The Watchdog Daemon
A master script (The "Monitor") runs every 60 seconds:
1.  **Read:** Open `Health.json`.
2.  **Check:** Is any timestamp older than 3 minutes?
3.  **Action:** 
    *   If yes, the process is "Dead."
    *   **Restart:** Issue a `pm2 restart [process_name]` command.
    *   **Log:** Record the crash in the Audit Log.

---

## 4. Resilience Patterns

### A. The "Offline Inbox"
If the internet is down, the Watchers cannot talk to APIs, but the **File Watcher** is still alive. 
*   **Logic:** The system continues to accept local "Drops" in the Inbox.
*   **Recovery:** When the internet returns, the Agent finds a "Backlog" in the processing folder and works through it chronologically.

### B. Graceful Shutdown
When the human turns off the computer, the Agent must not leave "Half-written" files.
*   **Logic:** The script listens for the `SIGTERM` signal. 
*   **Action:** It finishes the current task, saves its state, and then closes cleanly.

---

## 5. Why this is Enterprise-Grade
1.  **Persistence:** Using **PM2**, the Agent survives system reboots. It starts up as soon as the computer is turned on.
2.  **Telemetry:** You can look at the `Health.json` at any time to see if your employee is "At their desk."
3.  **No Data Loss:** By using the "Locked" file pattern (`/10_Processing`), even a hard power cut won't delete the file. The task stays in "Processing" until it is successfully moved to "Done."
4.  **Error Escalation:** If a script crashes 3 times in 10 minutes, the Watchdog stops trying to restart it and sends an **Emergency Alert** to the CEO's phone.
        