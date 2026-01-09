# Implementation Guide 03: Bronze Tier (The Foundation)

> *"The First Breath: Building a system that can See, Think, and Write."*

## 1. Objective
To build a "Minimum Viable Employee" that monitors a local folder (`00_Inbox`), detects a new file, and uses Claude Code to analyze it.

**Deliverables:**
1.  A functioning `watcher.py` script.
2.  A connection to the `claude-code` CLI.

---

## 2. Installing Dependencies
We need the library that listens to the Operating System.

```bash
uv add watchdog
```

---

## 3. The Watcher Script (`watcher.py`)
Create this file in your project root. This is the "Nervous System."

**Logic Flow:**
1.  Import `Observer` from `watchdog`.
2.  Define a class `Handler` that looks for **File Created** events.
3.  When a file appears in `Vault/00_Inbox`, print "I see a file!"
4.  **Crucial Step:** Trigger Claude using `subprocess`.

**The Code Blueprint:**
```python
import time
import subprocess
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

class NewFileHandler(FileSystemEventHandler):
    def on_created(self, event):
        if event.is_directory:
            return
        
        print(f"👀 Detected new file: {event.src_path}")
        
        # THE HANDOFF: Wake up Claude
        # We tell Claude: "Look at this file and do what the Handbook says."
        command = [
            "claude", 
            "-p", "Vault/00_Inbox",  # Point to the inbox
            "Process this new file according to Vault/System/Company_Handbook.md"
        ]
        
        print("🧠 Waking up the Brain...")
        subprocess.run(command)

def start_watching():
    observer = Observer()
    event_handler = NewFileHandler()
    
    # Watch the Inbox folder recursively
    observer.schedule(event_handler, path='Vault/00_Inbox', recursive=False)
    observer.start()
    print("MVE is listening... (Press Ctrl+C to stop)")
    
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        observer.stop()
    observer.join()

if __name__ == "__main__":
    start_watching()
```

---

## 4. Running the Simulation
1.  Open your terminal.
2.  Run the watcher: `uv run watcher.py`
3.  **The Trigger:** Create a text file named `Task.txt` in `Vault/00_Inbox` with the text: "Write a poem about AI."
4.  **The Result:**
    *   The script detects `Task.txt`.
    *   It launches Claude.
    *   Claude reads the file.
    *   Claude writes a response (likely in the terminal or a new file, depending on your prompt).

## 5. Success Criteria
*   [ ] The script runs without crashing.
*   [ ] Dropping a file triggers a "Detected" message.
*   [ ] Claude executes and exits cleanly.
