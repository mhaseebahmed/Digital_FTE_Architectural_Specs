# Implementation Guide 05: Gold Tier (The Executive)

> *"The Boss Mode: Automated Auditing and Reporting."*

## 1. Objective
To implement the **Monday Morning CEO Briefing**. The Agent will proactively audit its own work and report back to you.

**Dependencies:**
```bash
uv add schedule
```

---

## 2. The Auditor Script (`auditor.py`)
This script does NOT run continuously. It is a "One-Shot" task triggered by the scheduler.

**Logic:**
1.  It reads the JSON logs from `Vault/99_Logs`.
2.  It summarizes the data.
3.  It generates a Markdown report.

```python
import json
import logging
from pathlib import Path
from datetime import datetime, timedelta

def run_audit():
    log_dir = Path("Vault/99_Logs")
    logs = []
    
    # 1. Read Logs (Last 7 Days)
    cutoff = datetime.now() - timedelta(days=7)
    for log_file in log_dir.glob("*.json"):
        # (Simplified) Check file date
        data = json.loads(log_file.read_text())
        logs.append(data)
    
    # 2. Synthesize with Claude
    # We dump the summary to a temp file so Claude can read it
    summary_path = Path("Vault/10_Processing/AUDIT_DATA.txt")
    summary_path.write_text(str(logs))
    
    prompt = "Read 'Vault/10_Processing/AUDIT_DATA.txt'. Write a 'Monday Morning Briefing' markdown report. Save it to 'Vault/00_Inbox'."
    
    logging.info("📊 Generating Report...")
    subprocess.run(["claude", prompt])

if __name__ == "__main__":
    run_audit()
```

---

## 3. The Scheduler (`scheduler.py`)
This runs in the background.

```python
import schedule
import time
import subprocess
import logging

logging.basicConfig(level=logging.INFO)

def job():
    logging.info("⏰ Triggering Weekly Audit")
    subprocess.run(["uv", "run", "auditor.py"])

# Schedule: Every Monday at 9:00 AM
schedule.every().monday.at("09:00").do(job)

if __name__ == "__main__":
    logging.info("⏳ Scheduler Active...")
    while True:
        schedule.run_pending()
        time.sleep(60)
```

---

## 4. Deployment (PM2)
To run this 24/7 without a terminal window, use PM2.

1.  **Install PM2:** `npm install -g pm2`
2.  **Start Orchestrator:** `pm2 start orchestrator.py --interpreter python`
3.  **Start Scheduler:** `pm2 start scheduler.py --interpreter python`
4.  **Save List:** `pm2 save`
5.  **Startup Hook:** `pm2 startup`

**Status:**
You now have a fully autonomous Digital Employee that:
1.  Watches files and emails.
2.  Asks for permission on dangerous tasks.
3.  Audits itself every week.
4.  Restarts automatically if it crashes.
