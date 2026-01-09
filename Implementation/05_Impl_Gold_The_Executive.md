# Implementation Guide 05: Gold Tier (The Executive)

> *"The Boss Mode: Automated Auditing and Reporting."*

## 1. Objective
To implement the **Monday Morning CEO Briefing**. The Agent will proactively audit its own work and report back to you.

**Dependencies:**
```bash
uv add schedule
```

---

## 2. The Auditor Script (`tier_3_gold/auditor.py`)
This script does NOT run continuously. It is a "One-Shot" task triggered by the scheduler.

**Logic:**
1.  It reads the JSON logs from `Vault/99_Logs`.
2.  It summarizes the data.
3.  It generates a Markdown report.

---

## 3. The Scheduler Integration (`src/main.py`)
The scheduler is integrated into the main Orchestrator as a separate process.

```python
import schedule
from tier_3_gold.auditor import AuditEngine

def run_scheduler():
    auditor = AuditEngine()
    
    def job():
        report = auditor.run_weekly_audit()
        # Save Report logic...

    schedule.every().monday.at("09:00").do(job)
    
    while True:
        schedule.run_pending()
        time.sleep(60)
```

---

## 4. Deployment (PM2)
To run this 24/7 without a terminal window, use PM2.

1.  **Install PM2:** `npm install -g pm2`
2.  **Start:** `pm2 start deployment/ecosystem.config.js`
3.  **Save:** `pm2 save`
4.  **Startup:** `pm2 startup`

**Status:**
You now have a fully autonomous Digital Employee that:
1.  Watches files and emails.
2.  Asks for permission on dangerous tasks.
3.  Audits itself every week.
4.  Restarts automatically if it crashes.
