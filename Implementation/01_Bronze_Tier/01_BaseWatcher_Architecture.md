# Bronze Tier 01: The Base Watcher Architecture

> *"Perception is the first step of reasoning. A standard interface for the 'Senses' ensures a stable 'Brain'."*

## 1. The Architectural Requirement
As defined in the PDF (Page 5), all "Watchers" must follow a common structure.
*   **The Problem:** If every sensor (Email, WhatsApp, File) sends data in a different way, the Orchestrator becomes a "Spaghetti Code" mess.
*   **The Solution:** The `BaseWatcher` Abstract Class. It defines the "Rules" that every future watcher must follow.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Abstract Base Class (ABC)** | A blueprint for other classes. You cannot create an "ABC" object; you can only create "Children" that inherit from it. | The `BaseWatcher`. It defines *what* a watcher does, but not *how*. |
| **Method Overriding** | When a child class provides a specific implementation for a method defined in its parent. | The `FileSystemWatcher` providing the specific code for "Checking a Folder." |
| **Polling Interval** | The frequency of the check loop. | How often the watcher "wakes up" to look for work. |
| **Normalization** | The process of transforming unstructured data into a standardized format. | Turning a PDF or an Email into a standard Markdown file in the `/Inbox`. |

---

## 3. The BaseWatcher Blueprint (Architecture)

Every sense in our system must perform three fundamental tasks:
1.  **Check:** Look at the source (API, Folder, Browser).
2.  **Verify:** Ensure the data is ready and not corrupted/incomplete.
3.  **Handoff:** Convert the data to a file and drop it in the Vault.

### Logic Flow Diagram
```ascii
[ START ]
    |
    v
[ run() Loop ] <------------------------+
    |                                   |
    v                                   |
[ check_for_updates() ] --(No Updates)--> [ Sleep(Interval) ]
    |                                   |
    +--(New Data Found!)                |
    |                                   |
    v                                   |
[ create_action_file() ]                |
    |                                   |
    v                                   |
[ Normalization & Vault Drop ] ---------+
```

---

## 4. The Core Implementation Logic
*This logic must be followed by every "Sense" script in the Bronze, Silver, and Gold tiers.*

**The Shared Interface:**
1.  **`__init__`**: Initializes the `Vault_Path` and `Logger`.
2.  **`check_for_updates()`**: **(MUST BE OVERRIDDEN)**. This is the code that actually "sees" (e.g., searches a folder or an API).
3.  **`create_action_file()`**: **(MUST BE OVERRIDDEN)**. This is the code that writes the data to the `/Inbox`.
4.  **`run()`**: The engine that keeps the script alive forever. It handles the `while True` loop and error recovery.

### Why this is Enterprise-Grade
1.  **Strictness:** By using Python's `abc` module, the program will **refuse to run** if a developer forgets to implement `check_for_updates`. This prevents "silent failures."
2.  **Isolation:** The `BaseWatcher` handles the complex stuff (error logging, retry logic, loops). The "Child" watcher only handles the simple stuff (looking at a folder).
3.  **Scalability:** When moving from Bronze (File) to Silver (Gmail), the developer only needs to write the "Check" logic. The rest of the system stays exactly the same.
