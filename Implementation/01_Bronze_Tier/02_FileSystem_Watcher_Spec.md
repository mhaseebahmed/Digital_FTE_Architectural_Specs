# Bronze Tier 02: File System Watcher Implementation

> *"Precision Perception: Handling the real-world messiness of file system events."*

## 1. The Requirement
The File System Watcher is the "Primary Sense" for the Bronze Tier. It must monitor a local directory (`Vault/00_Inbox`) and ingest any file dropped there.

---

## 2. The "Stabilization" Challenge
In enterprise software, you never assume a file is ready just because it "exists."
*   **The Problem:** Large files (PDFs, Images) take time to write to the disk. If the Watcher acts the microsecond the file appears, it will read a "Partial File."
*   **The Solution:** **The Stabilization Loop.** We must monitor the file size and only proceed when the size remains constant for a set duration (e.g., 1.5 seconds).

---

## 3. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Race Condition** | When the output of a process depends on the timing of other uncontrollable events. | The Watcher trying to read a file before the OS finishes writing it. |
| **Polling (Fallback)** | Checking at fixed intervals if an event hasn't fired. | If the OS "Notify" event fails, we check the folder manually every 10 seconds. |
| **Atomic Move** | A file system operation that happens in a single step at the kernel level. | Moving a file from `Inbox` to `Processing`. This prevents other processes from seeing the file. |

---

## 4. The Implementation Algorithm

### Step 1: Detect
We use the `watchdog` library to subscribe to `on_created` events.

### Step 2: Stabilize
Before passing the file to the Brain, run this internal loop:
1.  Check `os.path.getsize(file)`.
2.  Wait 1 second.
3.  Check size again.
4.  If `size1 != size2`, repeat. If `size1 == size2`, proceed.

### Step 3: Ingest & Normalize
We don't just "read" the file. We wrap it in a **Metadata Jacket**.

**Example Transformation:**
*   **Input:** `invoice_october.pdf`
*   **Action:** Create a Markdown file `INBOX_invoice_october.md` in the `/10_Processing` folder.
*   **Content:**
    ```markdown
    # New File Detected
    *   **Source:** Local File System
    *   **Original Name:** invoice_october.pdf
    *   **Size:** 1.2MB
    *   **Path:** [Link to file]
    ---
    @Claude: Analyze this file and follow Handbook rules.
    ```

---

## 5. Decision Logic (The Flow)

```ascii
[ OS Event: New File ]
          |
          v
[ Is it a hidden/temp file? ] --(Yes)--> [ Ignore ]
          |
          v (No)
[ LOOP: Stabilization ]
    |  Check Size
    |  Wait 1s
    |  Check Size
    +-- Is Size Changing? --(Yes)--> [ Loop ]
          |
          v (No)
[ Move to /10_Processing ] <---( The Lock )
          |
          v
[ Generate Markdown Jacket ]
          |
          v
[ Trigger Brain (Claude) ]
```

---

## 6. Why this is Enterprise-Grade
1.  **Safety:** The stabilization loop prevents the system from crashing on large file uploads.
2.  **Auditability:** By creating a "Markdown Jacket," we provide the AI with **Context** (metadata) that isn't inside the PDF itself.
3.  **Cleanliness:** We use `shutil.move` for atomic operations, ensuring that the `/00_Inbox` is always a clean "Starting State."
4.  **Resilience:** If the script crashes during the "Brain" step, the file is safe in the `/10_Processing` folder, not lost in the trash.
