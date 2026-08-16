# CCAF — MCP Notes

## 3. MCP Server Primitives — Tools, Resources, and Prompts

MCP servers expose three core primitives:

```text
Tools
Resources
Prompts
```

The easiest way to distinguish them is by asking:

> **Who controls when this primitive is used?**

```text
Tools      → Model-controlled
Resources  → Application-controlled
Prompts    → User-controlled
```

This distinction is one of the most important MCP concepts to remember for CCAF.

---

## 1. Tools — Model-Controlled

A **tool gives Claude a capability it can choose to use autonomously**.

Claude decides whether a tool is needed based on the user's request.

Example:

```text
User
  │
  │ "What open issues are in my GitHub repo?"
  ▼
Claude
  │
  │ decides GitHub data is required
  ▼
MCP Tool
  │
  │ get_issues()
  ▼
GitHub API
  │
  ▼
Tool Result
  │
  ▼
Claude
  │
  ▼
Final Answer
```

The application exposes the available tools to Claude, but **Claude chooses whether and when to call them**.

### Example Tool

```python
@mcp.tool()
def get_github_issues(repo: str):
    ...
```

Possible tool use cases:

- search GitHub repositories
- fetch issues or pull requests
- execute JavaScript
- query a database
- create or update an external record
- perform calculations

### Mental Model

```text
Does Claude need a new capability?
          │
          └── Yes → TOOL
```

> **CCAF key point:** Tools are **model-controlled**. Claude decides when to invoke them.

---

## 2. Resources — Application-Controlled

A **resource exposes data that the host application can retrieve from an MCP server**.

The application decides when to read the resource and what to do with the returned data.

Typical uses include:

- populating UI elements
- loading autocomplete options
- loading configuration
- fetching documents
- adding external content to Claude's context

Example:

```python
@mcp.resource("docs://documents/{document_id}")
def get_document(document_id: str):
    ...
```

The application may retrieve the resource after a user selects a document:

```text
User
  │
  │ selects report.pdf
  ▼
Host Application
  │
  │ ReadResourceRequest
  ▼
MCP Server
  │
  ▼
docs://documents/report.pdf
  │
  ▼
Document Content
  │
  ▼
Host Application
  │
  │ adds content to conversation context
  ▼
Claude
```

The important point is that Claude did not autonomously choose to fetch the resource.

The **host application controls the resource access**.

### Mental Model

```text
Does my application need data
for UI or conversation context?
          │
          └── Yes → RESOURCE
```

> **CCAF key point:** Resources are **application-controlled**.

---

## 3. Prompts — User-Controlled

A **prompt is a predefined reusable workflow or prompt template exposed by an MCP server**.

Prompts are typically started through an explicit user action such as:

- clicking a button
- selecting a menu item
- choosing a workflow
- running a slash command

Example UI:

```text
┌──────────────────────────────┐
│ Ask Claude...                │
├──────────────────────────────┤
│ [ Review Pull Request ]      │
│ [ Summarize Document ]       │
│ [ Explain Error ]            │
└──────────────────────────────┘
```

When the user clicks **Review Pull Request**, the application can retrieve and execute a predefined MCP prompt.

Example:

```python
@mcp.prompt()
def review_pull_request():
    return """
    Review this pull request for:
    - correctness
    - security
    - performance
    - maintainability
    """
```

Flow:

```text
User
  │
  │ clicks "Review Pull Request"
  ▼
MCP Prompt
  │
  │ predefined instructions
  ▼
Claude
  │
  │ may call tools if required
  ▼
Final Result
```

### Mental Model

```text
Should the user explicitly start
this predefined workflow?
          │
          └── Yes → PROMPT
```

> **CCAF key point:** Prompts are **user-controlled**.

---

## 4. The Core Control Model

This is the most important summary:

```text
┌─────────────────────────────────────────────┐
│                 MCP SERVER                  │
│                                             │
│   TOOLS          RESOURCES       PROMPTS    │
│     ▲                ▲              ▲       │
│     │                │              │       │
│   Model          Application       User     │
│  controls          controls       controls  │
│                                             │
└─────────────────────────────────────────────┘
```

Memorize:

```text
Tool      → Model
Resource  → App
Prompt    → User
```

A useful shortcut is:

```text
T R P
│ │ │
│ │ └── Prompt   → User
│ └──── Resource → App
└────── Tool     → Model
```

Or simply:

> **Tools serve the model, resources serve the application, prompts serve the user.**

---

## 5. Decision Guide

Use this flow when deciding which primitive to implement:

```text
What am I trying to provide?
          │
          ├── A capability Claude should decide to use
          │        └── TOOL
          │
          ├── Data the application should retrieve
          │        └── RESOURCE
          │
          └── A predefined workflow the user starts
                   └── PROMPT
```

### Quick Comparison

| Primitive | Controlled By | Main Purpose | Example |
|---|---|---|---|
| **Tool** | Claude / model | Give Claude capabilities | `get_github_issues()` |
| **Resource** | Host application | Provide data/context | `docs://documents/123` |
| **Prompt** | User | Start predefined workflows | `Review Pull Request` |

---

## 6. Example — GitHub Assistant Using All Three

Imagine we build a GitHub assistant.

The user clicks a **Review Pull Request** workflow:

```text
User
  │
  │ clicks "Review PR"
  ▼
PROMPT
  │
  │ predefined review instructions
  ▼
Claude
  │
  │ decides PR data is required
  ▼
TOOL
  │
  │ get_pull_request()
  ▼
GitHub
  │
  ▼
Claude
  │
  ▼
Review Result
```

Meanwhile, the application might load coding standards using a resource:

```text
Host Application
      │
      ▼
RESOURCE
      │
      │ coding-standards.md
      ▼
Host Application
      │
      │ adds content to context
      ▼
Claude
```

So all three primitives can participate in the same application while still having different controllers.

---

## 7. Common CCAF Question Patterns

### Question 1

> Claude should decide when it needs to query a database.

Answer:

```text
Tool
```

Why?

Claude is choosing when to use the capability.

---

### Question 2

> Your UI needs autocomplete values from an MCP server.

Answer:

```text
Resource
```

Why?

The application is retrieving data for UI behavior.

---

### Question 3

> Users should click a button to start a "summarize document" workflow.

Answer:

```text
Prompt
```

Why?

The user explicitly starts a predefined workflow.

---

### Question 4

> Claude needs the ability to create GitHub issues when appropriate.

Answer:

```text
Tool
```

Why?

This is a capability Claude may autonomously decide to invoke.

---

### Question 5

> The application needs to retrieve the contents of a selected document and add them to Claude's context.

Answer:

```text
Resource
```

Why?

The host application controls when the data is retrieved.

---

## 8. Common Confusions

### Tool vs Resource

Both can involve fetching data.

The difference is **who controls the action**.

```text
Claude decides to fetch data
        → TOOL

Application decides to fetch data
        → RESOURCE
```

Do not classify something as a resource merely because it returns data.

---

### Tool vs Prompt

A prompt can eventually cause Claude to call tools.

Example:

```text
User clicks "Review PR"
        │
        ▼
      Prompt
        │
        ▼
      Claude
        │
        └── decides to call get_pr() tool
```

The prompt starts the workflow.

The tool provides the capability used inside the workflow.

---

### Resource vs Prompt

A resource provides **data**.

A prompt provides **instructions/workflow structure**.

```text
Resource → "Here is the document content."

Prompt   → "Summarize this document using these steps."
```

---

## 9. CCAF Exam Shortcut

When reading a question, identify the controller first.

```text
Who decides?

Claude      → Tool
Application → Resource
User        → Prompt
```

Then identify the purpose:

```text
Capability → Tool
Data       → Resource
Workflow   → Prompt
```

Combining both gives a strong decision rule:

```text
Claude + Capability   = Tool
App + Data            = Resource
User + Workflow       = Prompt
```

---

## 10. Final Revision Summary

```text
MCP SERVER PRIMITIVES

1. TOOLS
   Controller: Claude / Model
   Purpose: Give Claude capabilities
   Example: get_github_issues()

2. RESOURCES
   Controller: Host Application
   Purpose: Provide data or context
   Example: docs://documents/report.pdf

3. PROMPTS
   Controller: User
   Purpose: Trigger predefined workflows
   Example: "Review Pull Request"
```

### One-Line Memory Rule

> **Tool = Claude decides, Resource = App decides, Prompt = User decides.**

### CCAF Memory Formula

```text
T → M  (Tool → Model)
R → A  (Resource → App)
P → U  (Prompt → User)
```

If you remember this control model, most MCP primitive questions become straightforward.
