# Chapter 6: Model Context Protocol (MCP)

> *"The Hands of the Agent: A universal standard for connecting AI to systems."*

## 1. The Core Problem: The "Brain in a Jar"
A powerful AI model (like Claude) is isolated. It runs on a server in a data center. It cannot access your local file system, your USB ports, or your private database.
*   **Old Solution:** Custom integration code. (Writing a specific Python script for every single tool you want the AI to use). This is unscalable.
*   **New Solution (MCP):** A standardized protocol (like USB) for AI tools.

---

## 2. Key Terms & Concepts

| Term | Definition | Context in Digital FTE |
| :--- | :--- | :--- |
| **MCP Server** | The "Driver." | A lightweight program that exposes specific capabilities (e.g., "Read File", "Send Email") to the AI. |
| **MCP Client** | The "User." | The AI application (Claude Code) that connects to the servers to use their tools. |
| **Tool** | A function the AI can call. | Example: `send_email(to="boss@corp.com", body="Hi")`. |
| **Resource** | Data the AI can read. | Example: A database row, a file content, or a live API feed. |
| **Transport** | The wire. | How the Client talks to the Server (usually `stdio` or `HTTP`). |

---

## 3. The Architecture: The "Universal Adapter"
Think of MCP like a **USB Port**.
*   **Before USB:** You needed a specific port for a mouse, a printer, and a keyboard.
*   **With USB:** You plug anything in, and the computer knows how to talk to it.

**MCP works the same way:**
1.  You run a "Gmail MCP Server" on your computer.
2.  You tell Claude: "Hey, there is a tool server running here."
3.  Claude "handshakes" with the server: "What can you do?"
4.  The Server replies: "I can `list_emails` and `send_reply`."
5.  Claude now "knows" how to use Gmail.

### The Ecosystem
The PDF recommends several standard MCP servers:
*   **filesystem:** Allows reading/writing local files (Essential for the Vault).
*   **email-mcp:** Allows drafting and sending emails via Gmail APIs.
*   **browser-mcp:** Allows navigating websites and filling forms.
*   **slack-mcp:** Allows reading channels and posting messages.

---

## 4. The Interaction Flow

```ascii
[ Claude Code (The Brain/Client) ]
          |
          | (1. Request: "I need to send an email")
          v
[ MCP Protocol (The Language) ]
          |
          | (2. JSON-RPC Command: call_tool("send_email"))
          v
[ Email MCP Server (The Hand) ]
          |
          | (3. API Call to Google)
          v
[ Gmail API (The External System) ]
```

---

## 5. Why this matters for the Digital FTE
The Digital FTE is designed to be **Modular**.
*   **Scenario:** You switch from Gmail to Outlook.
*   **Without MCP:** You have to rewrite the entire "Brain" logic and prompts.
*   **With MCP:** You just unplug the "Gmail Server" and plug in the "Outlook Server."
    *   The prompt remains: *"Send an email."*
    *   The Agent doesn't care *how* it happens; it just asks the tool to do it.

## 6. Security Implications
MCP is designed with **Sandboxing** in mind.
*   You don't give Claude "Root Access" to your computer.
*   You give it access *only* to the specific MCP Server.
*   The MCP Server can be configured to "Read Only" mode, ensuring the AI can never delete your files even if it tries.
