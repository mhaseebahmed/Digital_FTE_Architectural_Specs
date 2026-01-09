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

## 3. The Script (`watcher.py`)

```python
import time
import shutil
import logging
import subprocess
import os
from pathlib import Path
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

# Logging Setup
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')

class SafeHandler(FileSystemEventHandler):
    def on_created(self, event):
        if event.is_directory:
            return

        src_path = Path(event.src_path)
        
        # Filter: Ignore temporary system files
        if src_path.name.startswith("."): 
            return

        logging.info(f"👀 Detected: {src_path.name}")
        self.process_file(src_path)

    def process_file(self, file_path):
        # Step 0: Stabilization Loop
        if not self.wait_for_file_ready(file_path):
            return

        # Step 1: Move to Processing (The Lock)
        processing_dir = Path("Vault/10_Processing")
        target_path = processing_dir / file_path.name
        
        # Retry Loop for Windows File Locking
        success = False
        for _ in range(3):
            try:
                shutil.move(str(file_path), str(target_path))
                logging.info(f"🔒 Locked file into: {target_path}")
                success = True
                break
            except PermissionError:
                logging.warning("File locked by another process. Retrying in 1s...")
                time.sleep(1)
            except Exception as e:
                logging.error(f"Failed to move file: {e}")
                return

        if not success:
            logging.error("❌ Could not lock file. Aborting.")
            return

        # Step 2: Wake up Claude
        self.trigger_brain(target_path)

        # Step 3: Archive (The Cleanup)
        done_dir = Path("Vault/20_Done")
        final_path = done_dir / file_path.name
        shutil.move(str(target_path), str(final_path))
        logging.info(f"✅ Task Complete. Archived to: {final_path}")

    def wait_for_file_ready(self, file_path, timeout=10):
        """Waits until file size stops changing (Upload Complete)."""
        start_time = time.time()
        last_size = -1
        
        while time.time() - start_time < timeout:
            try:
                current_size = os.path.getsize(file_path)
                if current_size == last_size and current_size > 0:
                    return True
                last_size = current_size
                time.sleep(1)
            except FileNotFoundError:
                return False
        
        logging.error(f"❌ File Timeout: {file_path}")
        return False

    def trigger_brain(self, file_path):
        prompt = f"Analyze the file '{file_path}'. Follow instructions in Vault/System/Company_Handbook.md."
        
        logging.info("🧠 Invoking Claude Code...")
        # CRITICAL: Use -p flag for non-interactive prompt
        result = subprocess.run(
            ["claude", "-p", prompt], 
            capture_output=True, 
            text=True
        )
        
        if result.returncode == 0:
            logging.info("🤖 Claude finished successfully.")
        else:
            logging.error(f"❌ Claude failed: {result.stderr}")

def start_daemon():
    observer = Observer()
    handler = SafeHandler()
    observer.schedule(handler, path='Vault/00_Inbox', recursive=False)
    observer.start()
    logging.info("🚀 Bronze Tier Agent Active. Drop files in /00_Inbox.")
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

if __name__ == "__main__":
    start_daemon()
```

---

## 4. Verification
1.  Run `uv run watcher.py`.
2.  Drop a **Large** PDF file in `00_Inbox`.
3.  **Expectation:**
    *   Console says "Detected".
    *   **Pause (1-2s):** Script waits for upload.
    *   File moves to Processing.
    *   Claude executes.
    *   File moves to Done.
