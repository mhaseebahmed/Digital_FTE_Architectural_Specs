# Implementation Guide 03: Bronze Tier (The Watcher)

> *"The First Breath: Implementing a robust, non-looping File System Watcher."*

## 1. Objective
Build a daemon that watches `00_Inbox`. When a file arrives, it must **safely** hand it off to Claude without crashing or looping.

**Dependencies:**
```bash
uv add watchdog
```

---

## 2. The Logic Flow (Avoiding the Infinite Loop)
A naive watcher triggers on *any* file event. If Claude reads the file, the OS might trigger a "Modified" event, causing the watcher to wake up again. **Loop of Death.**

**The Safe Protocol:**
1.  **Detect:** File appears in `00_Inbox`.
2.  **Stabilize:** Wait for the file size to stop changing (Large PDF upload protection).
3.  **Lock:** Watcher immediately moves file to `10_Processing`.
4.  **Process:** Claude works on the file in `10_Processing`.
5.  **Archive:** Orchestrator moves file to `20_Done`.

---

## 3. The Script (`tier_1_bronze/filesystem.py`)

```python
import time
import shutil
import os
from pathlib import Path
from watchdog.events import FileSystemEventHandler
from tenacity import retry, stop_after_attempt, wait_fixed, retry_if_exception_type

# Monorepo Imports
from shared_foundation.logger import setup_logger
from shared_foundation.config import settings
from shared_foundation.exceptions import TransientError
from tier_1_bronze.claude_client import ClaudeClient
from tier_3_gold.finance import FinancialEngine

logger = setup_logger("filesystem_watcher")

class RobustHandler(FileSystemEventHandler):
    def __init__(self):
        self.brain = ClaudeClient()
        self.finance = FinancialEngine()

    def on_created(self, event):
        if event.is_directory: return
        file_path = Path(event.src_path)
        if file_path.name.startswith("."): return
        
        logger.info(f"👀 Detected: {file_path.name}")
        self.process_workflow(file_path)

    def process_workflow(self, inbox_path: Path):
        if not self._stabilize_file(inbox_path):
            return

        # 1. LOCK
        processing_path = settings.get_processing_path() / inbox_path.name
        if not self._safe_move(inbox_path, processing_path):
            return

        # 2. INTELLIGENT ROUTING
        if processing_path.suffix.lower() == '.csv':
            self._handle_financial(processing_path)
        else:
            self._handle_generic(processing_path)

        # 3. ARCHIVE
        final_path = settings.get_done_path() / inbox_path.name
        self._safe_move(processing_path, final_path)
        logger.info("✨ Task Cycle Complete")

    def _handle_financial(self, file_path: Path):
        logger.info("💰 Routing to Financial Engine...")
        transactions = self.finance.process_csv(file_path)
        report = self.finance.generate_report(transactions)
        
        report_name = f"REPORT_{file_path.stem}.md"
        report_path = settings.get_done_path() / report_name
        report_path.write_text(report, encoding='utf-8')
        logger.info(f"📊 Financial Report generated: {report_name}")

    def _handle_generic(self, file_path: Path):
        logger.info("🧠 Routing to Claude Brain...")
        prompt = f"Read the file '{file_path}'. Follow instructions in Vault/System/Company_Handbook.md."
        self.brain.think(prompt)

    # (Helper methods _stabilize_file and _safe_move omitted for brevity - see repo)
```

---

## 4. Verification
1.  Run `uv run src/main.py`.
2.  Drop a **Large** PDF file in `00_Inbox`.
3.  **Expectation:**
    *   Console says "Detected".
    *   **Pause (1-2s):** Script waits for upload.
    *   File moves to Processing.
    *   Claude executes.
    *   File moves to Done.