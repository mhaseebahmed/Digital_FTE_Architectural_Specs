# Implementation Guide 05: Gold Tier (The Autonomous Employee)

> *"The Executive Function: Auditing, Reporting, and Long-Term Memory."*

## 1. Objective
To implement the **Monday Morning CEO Briefing**. The Agent will proactively audit its own work and report back to you.

**Deliverables:**
1.  `scheduler.py` (The Calendar).
2.  `auditor.py` (The Analyst).

---

## 2. Installing Dependencies
We need a library to handle scheduling easily.

```bash
uv add schedule
```

---

## 3. The Audit Logic
The Agent needs to "read its own diary" (the logs).

**Logic Flow:**
1.  **Scan:** Read all `.json` files in `Vault/99_Logs`.
2.  **Filter:** Select logs from `last_7_days`.
3.  **Aggregate:**
    *   Count `tasks_completed`.
    *   Sum `money_spent`.
    *   List `pending_approvals` (Bottlenecks).
4.  **Synthesize:** Use Claude to write a summary.

**The Prompt:**
> "Here are the logs from the last week. Write a 'Monday Morning Briefing' in Markdown. Highlight any bottlenecks and summarize financial spend."

---

## 4. The Scheduler (`scheduler.py`)
This script keeps the "Calendar" events running.

**The Code Blueprint:**
```python
import schedule
import time
import subprocess

def run_monday_briefing():
    print("📅 Starting Weekly Audit...")
    # Trigger Claude with the Audit Prompt
    subprocess.run(["claude", "-p", "Vault/99_Logs", "Generate Monday Morning Briefing"])

# Schedule it
schedule.every().monday.at("09:00").do(run_monday_briefing)

print("Gold Tier Scheduler is ticking...")
while True:
    schedule.run_pending()
    time.sleep(60)
```

---

## 5. The Complete System (Systemd / PM2)
To reach Gold Status, you don't run these scripts manually. You install them as a **Service**.

**Using PM2 (Recommended):**
1.  Create an `ecosystem.config.js` file.
2.  Define your processes:
    *   `app: orchestrator` (Runs Watchers)
    *   `app: scheduler` (Runs Cron)
3.  Start: `pm2 start ecosystem.config.js`
4.  Save: `pm2 save`

**Result:**
Your AI Employee now runs on boot, restarts on crash, and reports to you every Monday. You have achieved full autonomy.
