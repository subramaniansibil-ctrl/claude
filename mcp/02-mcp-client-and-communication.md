# CCAF — MCP Notes

## 2. MCP Client and Communication

### What is the MCP Client?

The **MCP client** is the communication bridge between your application/server and MCP servers.

It provides access to the tools exposed by an MCP server and handles the protocol-level message exchange so your application does not need to manage those details directly.

```text
Your Application / Server
          │
          ▼
      MCP Client
          │
          │ MCP messages
          ▼
      MCP Server
          │
          ▼
   External Service
```

A useful certification summary:

> **The MCP client is the application's access point to capabilities exposed by an MCP server.**

---

## 1. Transport-Agnostic Communication

One of MCP's key characteristics is that it is **transport agnostic**.

This means MCP does not depend on only one communication mechanism. The client and server can communicate using different transports depending on how the system is deployed.

### Common transport setup

A very common arrangement runs both the MCP client and MCP server on the **same machine** and communicates using standard input/output.

```text
Application
    │
MCP Client
    │
    │ stdin / stdout
    ▼
MCP Server
```

Other possible transports include:

- HTTP
- WebSockets
- Other network protocols

Conceptually:

```text
                 ┌─ stdin/stdout
MCP Client ──────┼─ HTTP
                 ├─ WebSockets
                 └─ Other transports
                         │
                         ▼
                     MCP Server
```

### Certification point

> **Transport agnostic means the MCP protocol is not tied to a single underlying communication transport.**

Do not confuse the **protocol** with the **transport** carrying the protocol messages.

---

## 2. Important MCP Message Types

After a client connects to an MCP server, they exchange message types defined by the MCP specification.

The main message pairs introduced in this lesson are:

```text
ListToolsRequest  → ListToolsResult
CallToolRequest   → CallToolResult
```

---

### ListToolsRequest / ListToolsResult

The MCP client asks the server which tools it provides.

```text
MCP Client
    │
    │ ListToolsRequest
    ▼
MCP Server
    │
    │ ListToolsResult
    ▼
Available tools
```

Conceptually:

```text
Client: "What tools do you provide?"

Server:
- get_repositories
- get_pull_requests
- get_issues
- ...
```

This process is **tool discovery**.

### Key point

> **ListToolsRequest discovers available tools; ListToolsResult returns their definitions.**

---

### CallToolRequest / CallToolResult

After Claude chooses a tool, the application asks the MCP client to execute it.

The client sends a **CallToolRequest** to the MCP server containing the selected tool and its arguments.

```text
MCP Client
    │
    │ CallToolRequest
    │ tool = get_repositories
    ▼
MCP Server
    │
    │ executes tool
    ▼
External Service
```

The result comes back as a **CallToolResult**.

```text
External Service
      │
      ▼
MCP Server
      │
      │ CallToolResult
      ▼
MCP Client
```

### Key point

> **CallToolRequest asks the MCP server to execute a tool; CallToolResult contains the result of that execution.**

---

## 3. Complete MCP Tool-Use Flow

Example user question:

> **"What repositories do I have?"**

The complete flow is:

### Step 1 — User Query

The user sends a question to your application/server.

```text
User
 │
 │ "What repositories do I have?"
 ▼
Your Server
```

### Step 2 — Tool Discovery Needed

Your server needs to determine which tools are available before sending the request to Claude.

### Step 3 — Server Asks MCP Client

```text
Your Server
    │
    │ Get available tools
    ▼
MCP Client
```

### Step 4 — List Tools Exchange

```text
MCP Client
    │
    │ ListToolsRequest
    ▼
MCP Server
    │
    │ ListToolsResult
    ▼
MCP Client
```

The server now knows which tools can be offered to Claude.

### Step 5 — Request Sent to Claude

Your application sends Claude:

```text
User query
    +
Available tool definitions
```

```text
Your Server
     │
     ▼
   Claude
```

### Step 6 — Claude Makes Tool-Use Decision

Claude determines that repository information is required and selects the appropriate tool.

```text
Claude
   │
   │ chooses
   ▼
get_repositories(...)
```

### Step 7 — Server Requests Tool Execution

Claude does not directly call GitHub.

Your server receives Claude's tool-use request and asks the MCP client to execute the selected tool.

```text
Claude
   │
   ▼
Your Server
   │
   ▼
MCP Client
```

### Step 8 — MCP Client Sends CallToolRequest

```text
MCP Client
    │
    │ CallToolRequest
    ▼
MCP Server
    │
    ▼
GitHub API
```

The MCP server performs the actual service-specific integration.

### Step 9 — Result Returns

```text
GitHub API
    │
    ▼
MCP Server
    │
    │ CallToolResult
    ▼
MCP Client
```

The repository data flows back through the MCP client to your server.

### Step 10 — Tool Result Sent to Claude

```text
MCP Client
    │
    ▼
Your Server
    │
    │ tool result
    ▼
Claude
```

### Step 11 — Claude Generates Final Response

Claude uses the returned repository data to construct a natural-language answer.

### Step 12 — User Receives Answer

```text
Claude
   │
   ▼
Your Server
   │
   ▼
User
```

---

## 4. Full Flow in One Diagram

```text
User
 │
 │ 1. Question
 ▼
Application / Server
 │
 │ 2. Need available tools
 ▼
MCP Client
 │
 │ 3. ListToolsRequest
 ▼
MCP Server
 │
 │ 4. ListToolsResult
 ▼
MCP Client
 │
 ▼
Application / Server
 │
 │ 5. Query + tools
 ▼
Claude
 │
 │ 6. Tool-use decision
 ▼
Application / Server
 │
 │ 7. Execute selected tool
 ▼
MCP Client
 │
 │ 8. CallToolRequest
 ▼
MCP Server
 │
 │ External API request
 ▼
GitHub API
 │
 │ Repository data
 ▼
MCP Server
 │
 │ 9. CallToolResult
 ▼
MCP Client
 │
 ▼
Application / Server
 │
 │ 10. Tool result
 ▼
Claude
 │
 │ 11. Final response
 ▼
Application / Server
 │
 │ 12. Answer
 ▼
User
```

---

## 5. Responsibility of Each Component

| Component | Main responsibility |
|---|---|
| **User** | Provides the request/question |
| **Application / Server** | Coordinates Claude and the MCP client |
| **MCP Client** | Handles communication with MCP servers |
| **MCP Server** | Exposes and executes tools/capabilities |
| **External Service** | Provides the underlying data/functionality |
| **Claude** | Decides when a tool is needed and uses results to answer |

This separation of responsibility is important.

The MCP client abstracts server communication, while the MCP server handles service-specific functionality.

---

## Certification Takeaways

Remember these points:

1. **The MCP client is the communication bridge** between your application and MCP servers.
2. MCP is **transport agnostic** — it is not tied to only one communication mechanism.
3. A common local transport is **standard input/output (stdin/stdout)**.
4. Other transports can include **HTTP and WebSockets**.
5. **ListToolsRequest** asks which tools are available.
6. **ListToolsResult** returns the available tools.
7. **CallToolRequest** asks the server to execute a particular tool with arguments.
8. **CallToolResult** contains the tool execution result.
9. Claude typically **decides which tool to use**, while the MCP server performs the actual external-service interaction.
10. Tool results are sent back to Claude so it can generate the final answer.

---

## Assessment Traps

### Is the MCP client the component that directly implements GitHub API logic?

**No.**

The MCP client handles MCP communication. The GitHub-specific implementation belongs to the GitHub MCP server.

### Does MCP require HTTP?

**No.**

MCP is transport agnostic. Communication can use stdin/stdout, HTTP, WebSockets, or other supported transports.

### Does Claude directly send CallToolRequest to GitHub?

**No.**

Claude selects the tool. Your application coordinates execution through the MCP client, which sends the MCP request to the MCP server. The MCP server then interacts with GitHub.

### What is the difference between ListToolsRequest and CallToolRequest?

```text
ListToolsRequest → Discover which tools exist
CallToolRequest  → Execute one of those tools
```
