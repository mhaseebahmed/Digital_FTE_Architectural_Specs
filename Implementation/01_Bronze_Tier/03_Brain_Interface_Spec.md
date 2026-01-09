# Bronze Tier 03: The Brain Interface (Claude CLI)

> *"The Cognitive Handoff: Connecting the Sensory Nerves (Python) to the Executive Center (Claude Code)."*

## 1. The Requirement
Once a Watcher has "seen" a file, it must invoke the **Brain**. This is done by calling the `claude-code` CLI tool from within Python.

---

## 2. Technical Lexicon

| Term | Technical Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **Subprocess** | A process launched by another process. | Our Python Watcher "spawns" a Claude process to think. |
| **Stdout / Stderr** | Standard Output and Standard Error. | The "speech" of the AI. We must capture this to see what Claude is thinking/doing. |
| **Timeout** | A pre-defined limit on how long a process can run. | We limit Claude to 5 minutes. If it gets stuck in a loop, the system kills it to save resources. |
| **Exit Code** | A number returned by a process to indicate success (0) or failure (>0). | We check the exit code to know if Claude succeeded. |

---

## 3. The Implementation Strategy: The `BrainClient`

We do not call `subprocess.run()` casually. We wrap it in a class that manages the "Session."

### The Protocol (Logic)
1.  **Preparation:** Assemble the prompt. (e.g., "Analyze this file in /10_Processing").
2.  **Launch:** Spawn the `claude` process.
3.  **Monitor:** Wait for the process to finish, but watch the clock (Timeout).
4.  **Capture:** Read the output logs.
5.  **Log:** Save the results to `Vault/99_Logs/System/brain_activity.json`.

---

## 4. Interaction Diagram

```ascii
[ WATCHER ] --(Detected File)--> [ ORCHESTRATOR ]
                                       |
                                       v
                                [ BrainClient ]
                                       |
          +----------------------------+----------------------------+
          |                            |                            |
          v                            v                            v
[ SPAWN: 'claude -p ...' ]    [ START TIMEOUT TIMER ]      [ ATTACH LOG CAPTURE ]
          |                            |                            |
          +----------------------------+----------------------------+
                                       |
                                       v
                             [ SUCCESS OR FAILURE? ]
                                       |
          +----------------------------+----------------------------+
          | (Success)                                               | (Failure)
          v                                                         v
[ Move File to /Done ]                                    [ Log Error & Alert Human ]
```

---

## 5. Why this is Enterprise-Grade
1.  **Timeout Protection:** Prevents the Agent from running indefinitely and burning your API credits or CPU.
2.  **Error Isolation:** If Claude crashes, the Python script stays alive. The "Watcher" continues to watch.
3.  **Non-Blocking Logic:** (Silver/Gold extension) We can launch multiple Claude instances in parallel to handle multiple files.
4.  **Logging:** Every "Thought" is captured. If the AI makes a mistake, we have the "Record of Reasoning" to audit.

---

## 6. Closing the Bronze Loop
At the end of this module, the "Minimum Viable Deliverable" is complete:
1.  **Sense:** File Watcher detects a drop.
2.  **Think:** Claude is invoked via the `BrainClient`.
3.  **Act:** Claude moves the file to `/Done` (or as per Handbook instructions).
