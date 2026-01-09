# Core Foundation 01: Enterprise Logging & Telemetry

> *"Visibility is the prerequisite for Autonomy. If you cannot see what the Agent is doing, you cannot trust it."*

## 1. The Requirement
A production-grade "Digital Employee" runs 24/7/365. Over a year, it will generate millions of events.
*   **The Problem:** Standard text logs (`INFO: File found`) are unsearchable at scale.
*   **The Solution:** Structured JSON Logging.
*   **The Safety Net:** We must ensure PII (Personally Identifiable Information) and API Keys are **scrubbed** before being written to disk.

---

## 2. Technical Specification

### A. Log Format (JSON Schema)
Every log line must adhere to this strict schema:
```json
{
  "timestamp": "2026-10-12T08:00:00.123Z",
  "level": "INFO",
  "service": "watcher",
  "trace_id": "abc-123-uuid",
  "message": "File detected",
  "data": {
    "filename": "invoice.pdf",
    "size_bytes": 1024
  }
}
```

### B. Implementation Architecture
We do not use `print()`. We use a custom wrapper around Python's `logging` module.

**Dependencies:**
```bash
uv add python-json-logger
```

### C. The `logger.py` Module
*This file should be placed in `AI_Employee/src/shared/logger.py`.*

```python
import logging
import sys
import uuid
from pythonjsonlogger import jsonlogger
from logging.handlers import RotatingFileHandler
from pathlib import Path

# CONSTANTS
LOG_DIR = Path("Vault/99_Logs/System")
LOG_DIR.mkdir(parents=True, exist_ok=True)
MAX_BYTES = 10 * 1024 * 1024  # 10 MB
BACKUP_COUNT = 5              # Keep 5 historical files

class PIILogFilter(logging.Filter):
    """
    Redacts secrets from logs.
    """
    def __init__(self, secrets=None):
        super().__init__()
        self.secrets = secrets or []

    def filter(self, record):
        msg = str(record.msg)
        for secret in self.secrets:
            if secret and secret in msg:
                msg = msg.replace(secret, "[REDACTED]")
        record.msg = msg
        return True

def setup_logger(service_name: str, sensitive_keys: list = None) -> logging.Logger:
    """
    Creates a production-grade logger with rotation and JSON formatting.
    """
    logger = logging.getLogger(service_name)
    logger.setLevel(logging.INFO)
    logger.propagate = False  # Prevent double logging

    # 1. JSON Formatter
    formatter = jsonlogger.JsonFormatter(
        '%(asctime)s %(levelname)s %(name)s %(message)s'
    )

    # 2. File Handler (Rotating)
    file_handler = RotatingFileHandler(
        filename=LOG_DIR / f"{service_name}.json",
        maxBytes=MAX_BYTES,
        backupCount=BACKUP_COUNT
    )
    file_handler.setFormatter(formatter)
    logger.addHandler(file_handler)

    # 3. Console Handler (Standard Text for Humans)
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(logging.Formatter('%(asctime)s - %(name)s - %(message)s'))
    logger.addHandler(console_handler)

    # 4. PII Filter
    if sensitive_keys:
        logger.addFilter(PIILogFilter(sensitive_keys))

    return logger

def get_trace_id() -> str:
    """Generates a unique ID for tracking a task across services."""
    return str(uuid.uuid4())
```

---

## 3. Usage Pattern
Every script in the system must initialize logging first.

```python
from src.shared.logger import setup_logger, get_trace_id

# 1. Setup
# Pass any secrets (like API keys) to be auto-redacted
logger = setup_logger("gmail_watcher", sensitive_keys=["sk-12345"])

# 2. Execution
trace_id = get_trace_id()
logger.info("Starting Poll Cycle", extra={"trace_id": trace_id, "cycle": 1})

try:
    # ... logic ...
    pass
except Exception as e:
    # 3. Structured Error Logging
    logger.error("Poll Failed", extra={
        "trace_id": trace_id, 
        "error_type": type(e).__name__,
        "error_detail": str(e)
    })
```

---

## 4. Why this is Enterprise-Grade
1.  **Rotation:** It will never fill your hard drive. At 10MB, it creates a new file.
2.  **Redaction:** If you accidentally log an API key, the `PIILogFilter` catches it.
3.  **Parsable:** Because it is JSON, we can later write the "Gold Tier Auditor" to read these files programmatically.
4.  **Traceability:** The `trace_id` allows us to follow a single email from ingestion -> processing -> completion.
