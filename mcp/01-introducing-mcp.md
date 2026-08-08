# CCAF — MCP Notes

## 1. Introducing MCP

### What is MCP?

**Model Context Protocol (MCP)** is a communication layer that provides Claude with **context and tools** without requiring developers to implement large amounts of integration code.

The central idea is:

```text
Your Application
      │
  MCP Client
      │
      │ MCP
      ▼
  MCP Server
      │
      ▼
External Service
```

MCP shifts responsibility for **tool definitions and tool execution** away from your application and toward specialized MCP servers.

---

## 2. Problem MCP Solves

Suppose a user asks Claude:

```text
"What open pull requests are there
across all my GitHub repositories?"
```

Claude needs access to GitHub functionality.

### Without MCP

Your application would need to implement many GitHub tools:

```text
get_repositories()
get_pull_requests()
get_issues()
create_issue()
search_code()
...
```

For each capability, developers may need to maintain:

```text
Tool schema
   +
Tool function
   +
GitHub API integration
   +
Testing
   +
Maintenance
```

For a service as large as GitHub, this becomes expensive and difficult to maintain.

### With MCP

```text
Your Application
      │
  MCP Client
      │
      ▼
GitHub MCP Server
      │
      ▼
  GitHub API
```

The GitHub MCP server already exposes standardized GitHub capabilities.

Your application therefore doesn't need to implement every GitHub tool itself.

---

## 3. MCP Architecture

At this stage of the course, remember:

```text
┌──────────────────┐
│    MCP Client    │
│  Your server/app │
└────────┬─────────┘
         │
         │ MCP
         ▼
┌──────────────────┐
│    MCP Server    │
├──────────────────┤
│ Tools            │
│ Prompts          │
│ Resources        │
└────────┬─────────┘
         │
         ▼
 External Service
```

An **MCP Server acts as an interface to an outside service**.

Example:

```text
MCP Client
    │
    ▼
GitHub MCP Server
    │
    ├── get_repos()
    ├── get_pull_requests()
    └── ...
    │
    ▼
GitHub API
```

---

## 4. What MCP Servers Provide

The course introduces three important capabilities:

| Capability | Purpose |
|---|---|
| **Tools** | Functions/actions Claude can use |
| **Resources** | Data/context that can be accessed |
| **Prompts** | Reusable prompt definitions |

For now, remember:

```text
MCP Server
   ├── Tools
   ├── Prompts
   └── Resources
```

We'll expand these definitions as the course introduces them.

---

## 5. Who Authors MCP Servers?

**Anyone can create an MCP server implementation.**

They may be created by:

```text
Service providers
Community developers
Organizations
Individual developers
```

Service providers may publish official MCP servers for their own services.

Course example:

```text
AWS
 │
 ▼
Official AWS MCP Server
 │
 ▼
AWS Services
```

---

## 6. MCP vs Calling APIs Directly

This is an important certification distinction.

### Direct API approach

```text
Your application
      │
      │ Your tool definitions
      │ Your functions
      │ Your integration code
      ▼
External API
```

You are responsible for authoring the tool definitions and integration.

### MCP approach

```text
Your application
      │
      ▼
MCP Server
      │
      ▼
External API
```

The MCP server provides the tool schemas and functions.

### Key certification point

> **MCP does not mean external APIs disappear.**

The MCP server can call the external API internally.

The difference is **who implements the integration**.

---

## 7. MCP vs Tool Use

⚠️ **Important distinction**

```text
MCP ≠ Tool Use
```

They are **complementary concepts**.

### Tool Use

Tool use is about **how Claude calls tools**.

```text
Claude
   │
   │ decides to call
   ▼
get_repos()
```

### MCP

MCP provides the **tool schemas and functions** through an MCP server.

```text
MCP Server
    │
    │ exposes
    ▼
get_repos()
    │
    │ Claude uses tool use
    ▼
Claude calls it
```

Therefore:

> **MCP provides standardized access to tools; tool use describes Claude's ability to call those tools.**

---

## Certification Takeaways

For this lesson, memorize these five points:

1. **MCP is a communication layer** for providing Claude with context and tools.
2. **MCP servers expose tools, prompts, and resources.**
3. MCP moves **tool definitions and execution** away from your application into specialized MCP servers.
4. **Anyone can author an MCP server**, including service providers.
5. **MCP and tool use are not the same thing** — MCP provides implemented tools; tool use is how Claude invokes them.

### Typical CCAF-style trap

If asked:

> "Is MCP just another name for Claude tool use?"

**Answer: No.**

Tool use is Claude's mechanism for invoking tools. MCP is the standardized protocol/infrastructure through which external servers can expose already-implemented tools and other capabilities.
