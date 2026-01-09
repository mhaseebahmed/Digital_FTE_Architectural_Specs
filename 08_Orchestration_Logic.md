# Chapter 8: Orchestration & Scheduling

> *"The Heartbeat: Implementing persistence and temporal logic."*

## 1. The Engineering Challenge: Persistence
Software running on a personal computer is inherently ephemeral.
*   **The Problem:** Processes die. Exceptions occur. Memory leaks happen.
*   **The Goal:** We need to elevate a Python script to the level of a **System Service** (like the Printer Spooler or Wi-Fi driver). It must be self-healing and persistent across reboots.

---

## 2. Technical Lexicon

| Term | Technical Definition | Conceptual Analogy |
| :--- | :--- | :--- |
| **Orchestrator** | The Master Process that controls the execution flow of worker processes. | The Conductor of an orchestra. They don't play an instrument; they signal when the violin should start. |
| **Cron** | A time-based job scheduler in Unix-like computer operating systems. | The alarm clock that wakes specific processes at specific times. |
| **Subprocess** | A child process created by a parent process. It runs independently but shares resources. | A contractor hired for a specific job. |
| **Process Manager (PM2)** | A production process manager for Node.js/Python with a built-in load balancer. | A Life Support System. If the heart stops, it shocks it back to life instantly. |

---

## 3. The Three Time-Domains
An autonomous system must handle time in three different ways.

### A. Continuous (Event Loop)
*   **Implementation:** `while True:` (Infinite Loop) or Event Listeners.
*   **Use Case:** Watchers.
*   **Behavior:** Blocking. It occupies the CPU (minutely) waiting for an interrupt.

### B. Interval (Poller)
*   **Implementation:** `time.sleep(300)` inside a loop.
*   **Use Case:** API Fetching.
*   **Behavior:** Intermittent. Active for 1 second, dormant for 299 seconds.

### C. Scheduled (Cron)
*   **Implementation:** System Cron or Task Scheduler.
*   **Use Case:** Reporting (Monday Briefing).
*   **Behavior:** Precise execution.

#### Understanding Cron Syntax
The standard syntax is 5 asterisks: `* * * * *`
1.  **Minute** (0-59)
2.  **Hour** (0-23)
3.  **Day of Month** (1-31)
4.  **Month** (1-12)
5.  **Day of Week** (0-6) (Sunday=0)

**Example: The Monday Morning Briefing**
`0 9 * * 1`
*   `0`: At the 0th minute.
*   `9`: Of the 9th hour (9 AM).
*   `*`: Every day of the month.
*   `*`: Every month.
*   `1`: Only if it is Monday.

---

## 4. The Process Architecture (PM2)
We use **PM2** to manage the lifecycle of our scripts.

### Why PM2?
1.  **Daemonization:** It detaches the script from your current terminal. You can close the window, and the script keeps running.
2.  **Auto-Restart:** It monitors the `PID` (Process ID). If the PID disappears (crash), PM2 spawns a new instance immediately.
3.  **Boot Persistence:** It generates a startup script so your agent launches when Windows boots.

### The Orchestration Logic
The Orchestrator acts as a **Dispatcher**.

1.  **Monitor:** It uses `watchdog` to listen to `/Inbox`.
2.  **Detect:** File event received.
3.  **Spawn Subprocess:** It calls Claude Code using the `subprocess.run()` command.
    *   *Why Subprocess?* Memory Management.
    *   If we ran Claude in the *same* process, variables from Task A might leak into Task B.
    *   By spawning a new process, we ensure a "Clean Slate" (Fresh RAM) for every task.
4.  **Reap:** When Claude finishes, the subprocess terminates, freeing all resources.

```ascii
[ SYSTEM KERNEL ]
       |
       v
[ PM2 SUPERVISOR ] --(Restarts if Crash)--> [ ORCHESTRATOR.PY ]
                                                   |
                                                   +--- (Spawn) ---> [ CLAUDE WORKER 1 ]
                                                   |
                                                   +--- (Spawn) ---> [ CLAUDE WORKER 2 ]
```
