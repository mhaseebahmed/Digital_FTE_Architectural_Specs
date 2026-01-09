# System Architecture: The Anatomy of a Bionic Employee

To build a Digital FTE, we don't just write one big script. We build a system that mimics a living organism. It has four distinct body parts.

## 1. The Senses: "The Watchers" (Python)
*   **The Problem:** Large Language Models (the Brain) are deaf and blind. They only answer when you talk to them. They cannot "wait" for an email.
*   **The Solution:** We write tiny, efficient Python scripts that never sleep.
    *   **The Gmail Watcher:** Stares at your inbox.
    *   **The File Watcher:** Stares at a folder.
*   **Analogy:** The Security Guard. He is not the CEO. He doesn't make decisions. He just watches the door and radios the Boss when something arrives.

## 2. The Brain: "Claude Code" (Reasoning)
*   **The Component:** A specialized Command Line version of Claude.
*   **Superpower:** Unlike the web chat, this version breaks out of the browser. It has **System Access**. It can read your files, write code, and execute commands.
*   **The Flow:** The Watcher (Guard) wakes up Claude (CEO). Claude reads the data, thinks about it using the `Company_Handbook.md`, and decides what to do.

## 3. The Memory: "Obsidian" (State)
*   **The Component:** A folder of Markdown files.
*   **Function:** This is the "Office Record."
    *   **Dynamic State:** If the computer crashes, we don't lose the work. We just look at the `Dashboard.md` file to see "Invoice #4 is Pending."
    *   **Shared Brain:** It formats the raw text data into a beautiful dashboard so the Human Manager can see what's happening at a glance.

## 4. The Hands: "MCP" (Action)
*   **The Concept:** Model Context Protocol.
*   **The Problem:** A Brain in a jar can think "Pay the bill," but it has no hands to type the credit card number.
*   **The Solution:** MCP Servers are the "Robotic Arms."
    *   We plug in a "Gmail Arm" (MCP Server) -> Now Claude can click 'Send'.
    *   We plug in a "Bank Arm" (MCP Server) -> Now Claude can transfer money.
*   **Flexibility:** If you change banks, you just swap the robotic arm (the MCP Server). You don't have to perform brain surgery (retrain the AI).
