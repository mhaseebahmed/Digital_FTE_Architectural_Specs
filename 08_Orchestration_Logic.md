# Chapter 8: Orchestration & Scheduling

> *"The Heartbeat: Keeping the Agent alive, awake, and on time."*

## 1. The Core Problem: Fragility
A Python script running in a terminal is fragile.
*   If your computer restarts, it stops.
*   If the internet blips, it crashes.
*   If you close the window, it dies.
*   **The Goal:** We need "Industrial Grade" persistence. The Agent must act like a System Service, not a toy script.

---

## 2. Key Terms & Concepts

| Term | Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Orchestrator** | The Master Process. | A script that manages *other* scripts. It decides *when* to run Claude. |
| **Cron** | Time-based Job Scheduler. | Standard Linux tool to run tasks at specific times (e.g., "Every Monday at 9 AM"). |
| **PM2** | Process Manager 2. | A production tool (originally for Node.js) that keeps scripts running forever. It auto-restarts them if they crash. |
| **Subprocess** | A process launched by another process. | The Orchestrator launches Claude Code as a subprocess to keep the memory clean. |

---

## 3. The Three Time-Domains
The Digital FTE operates in three distinct time dimensions.

### A. Continuous (The Event Loop)
*   **Used For:** Watchers (File System).
*   **Mechanism:** `while True:` loop.
*   **Logic:** Block and wait. React instantly.
*   **Example:** A file drop triggers an immediate response.

### B. Interval (The Poller)
*   **Used For:** API Watchers (Gmail, Stripe).
*   **Mechanism:** `time.sleep(300)`.
*   **Logic:** Check, Sleep, Repeat.
*   **Why:** To respect API Rate Limits and save CPU. We don't need to check email every millisecond.

### C. Scheduled (The Calendar)
*   **Used For:** Reporting and Maintenance.
*   **Mechanism:** System Cron / Windows Task Scheduler.
*   **Logic:** `0 9 * * 1` (At 09:00 on Monday).
*   **Example:** "Generate the Weekly CEO Briefing." The Agent wakes up specifically to perform this audit, even if there are no new emails.

---

## 4. The Tooling: PM2
The PDF recommends using **PM2** to manage the "Watcher" scripts.

### Why PM2?
1.  **Auto-Restart:** If `gmail_watcher.py` crashes because Google API failed, PM2 restarts it instantly.
2.  **Startup:** It hooks into the OS startup. If you reboot your laptop, the Agent starts automatically.
3.  **Logs:** It manages the `stdout` (print statements) into log files so you can debug what happened while you were asleep.

### The Orchestrator Script Logic
The Orchestrator is the "Brain Manager." It doesn't do the work; it assigns it.

```ascii
[ Orchestrator.py ]
       |
       +--- (Listen) ---> [ /Inbox Folder ]
       |
       +--- (Detects File)
       |
       +--- (Spawn Subprocess) ---> [ Claude Code CLI ]
                                          |
                                          +--- (Read File)
                                          +--- (Do Work)
                                          +--- (Exit)
       |
       +--- (Cleanup) ---> [ Move File to /Done ]
```

> **Critical Note:** We launch Claude as a *subprocess* and then let it *exit*. We do not keep Claude running forever. This prevents "Context Bloat" (running out of memory) and ensures every task starts with a fresh brain.
