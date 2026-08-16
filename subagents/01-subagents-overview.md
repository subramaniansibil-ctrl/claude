# Claude Code Subagents

## What are Subagents?

Subagents are specialized assistants that Claude Code can delegate tasks to.

Think of them like focused helpers:

```text
Main Claude = Team Lead
Subagent    = Specialist
```

Each subagent runs in its own conversation context window, performs its work, and returns a concise summary to the main Claude conversation.

The intermediate work — file reads, searches, tool calls, and reasoning — stays isolated from the main context.

---

## Why Subagents Matter

Every time you use Claude Code, the main conversation accumulates context:

- Your messages
- Claude's responses
- File reads
- Search results
- Tool calls
- Code exploration

The context window is finite. In a long session, too much exploration can make the main conversation noisy and reduce how much earlier information Claude can keep available.

Subagents solve this by doing focused work in a separate context window.

### Without a Subagent

Suppose you ask:

> Find how authentication works in this project.

Claude may do all of this in the main context:

```text
Main Context
   |
   +-- Search "auth"
   +-- Read middleware/auth.js
   +-- Read routes/index.js
   +-- Read config/index.js
   +-- Search JWT
   +-- Read many more files
   +-- Trace request flow
   +-- Answer
```

All of those intermediate steps consume the same main context window.

### With a Subagent

```text
                    MAIN CLAUDE
                         |
                         | "Investigate authentication"
                         v
                +-----------------+
                | Explore Agent   |
                |                 |
                | Search files    |
                | Read middleware |
                | Trace JWT       |
                | Inspect routes  |
                | Analyze flow    |
                +--------+--------+
                         |
                         | Summary
                         v
                    MAIN CLAUDE
```

The main conversation receives mainly the useful result instead of every exploration step.

---

## What a Subagent Receives

A subagent receives two important things:

1. **System prompt** — defines what kind of specialist the subagent is and how it should behave.
2. **Task description** — tells the subagent what it needs to do for the current request.

Think of it like this:

```text
System prompt = Job description
Task prompt   = Today's ticket
```

Example:

```text
System prompt:
You are a senior React reviewer.
Do not edit files.
Focus on correctness and performance.

Task:
Review the /new-weight implementation and identify
React rendering or memoization problems.
```

---

## Built-in Subagents

Claude Code provides built-in subagents for common workflows.

### 1. Explore

Use **Explore** when you want to find or understand something in a codebase.

Typical jobs:

- Find files
- Search implementations
- Trace function calls
- Understand architecture
- Follow imports
- Investigate how an existing feature works

Example:

```text
Use the Explore subagent to investigate how sorting works
in the legacy Weight module.

Find:
- React component
- Redux action
- Saga
- API helper
- BFF endpoint

Do not change any files.
Return the relevant files and flow.
```

Conceptually:

```text
Need to FIND / UNDERSTAND something
              |
              v
           Explore
```

---

### 2. Plan

Use **Plan** when Claude needs to research the codebase before proposing how a change should be implemented.

Example:

```text
Plan how multiple Product IDs should be supported
in the /new-weight add screen.
```

The agent may investigate:

```text
Existing implementation
        |
        v
API contract
        |
        v
Redux / Saga flow
        |
        v
UI requirements
        |
        v
Testing conventions
        |
        v
Implementation plan
```

Conceptually:

```text
Need to DESIGN a change
          |
          v
         Plan
```

---

### 3. General-purpose

Use a general-purpose subagent for multi-step tasks that involve both investigation and action.

For example:

```text
Implement multiple Product ID support and add tests.
```

A general-purpose agent may:

```text
Understand existing implementation
          |
          v
Modify React form
          |
          v
Modify Saga
          |
          v
Update TypeScript types
          |
          v
Add tests
          |
          v
Run tests
          |
          v
Return summary
```

Conceptually:

```text
Need to IMPLEMENT / perform multi-step work
                  |
                  v
          General-purpose
```

---

## How to Use the Explore Subagent

There usually is not a special `/explore` command that you need to run.

The simplest approach is to tell Claude Code directly in natural language:

```text
Use the Explore subagent to find how authentication is implemented
in this project.

Do not modify any files.
```

Another example:

```text
Use an Explore subagent.

Investigate the /new-weight implementation and determine
how Product ID search works.

Do not edit files.
Do not implement anything.

Return:
1. Entry component
2. Redux flow
3. Saga
4. API call
5. BFF endpoint
6. Important file paths
```

Claude Code can then delegate that task to the Explore agent internally.

### `/agents`

Claude Code also provides agent management functionality for working with subagents, especially custom subagents.

This is different from using `/explore` as though Explore were a normal slash command.

For a one-off exploration task, simply ask Claude:

```text
Use the Explore subagent to trace how /new-weight search works.
```

---

## Custom Subagents

You can create your own specialized subagents with custom system prompts and tool access.

Examples:

```text
Main Claude
    |
    +-- Explore Agent
    |      +-- Understand codebase
    |
    +-- React Reviewer
    |      +-- Check React + TypeScript patterns
    |
    +-- Test Agent
    |      +-- Write/run tests
    |
    +-- Security Reviewer
           +-- Check auth/RBAC/API issues
```

Example custom role:

```text
You are a senior React + TypeScript reviewer.

Focus on:
- React patterns
- Hook correctness
- Unnecessary renders
- Memoization
- TypeScript safety
- Redux-Saga race conditions
- Accessibility

Do not modify code.
Return findings with severity and file locations.
```

---

## Subagents vs Skills vs MCP vs Hooks

These concepts solve different problems.

```text
Claude Code
|
+-- Main Agent
|
+-- Subagents
|    +-- Explore
|    +-- Plan
|    +-- General-purpose
|    +-- Custom Agents
|
+-- Skills
|
+-- MCP
|    +-- Tools
|    +-- Resources
|    +-- Prompts
|
+-- Hooks
```

### Subagent

**Who performs the work.**

Example: React Reviewer agent.

### Skill

**Instructions or knowledge about how to perform a type of work.**

Example: React best-practices skill.

### MCP Tool

**A capability the agent can invoke.**

Example: GitHub tool that reads a pull request.

### Hook

**Automation triggered by Claude Code activity.**

Example: automatically run verification after Claude modifies code.

They can work together:

```text
React Reviewer Subagent
        |
        | uses
        v
React Best Practices Skill
        |
        | may call
        v
GitHub MCP Tool
        |
        v
Reads Pull Request
```

---

## Recommended Workflow

For a large codebase, a useful workflow is:

```text
Explore -> Plan -> Implement -> Verify
```

### Step 1 — Explore

```text
Use the Explore subagent to understand how the legacy Weight
add screen submits Product IDs.

Do not modify anything.
```

### Step 2 — Plan

```text
Based on that investigation, create an implementation plan
for supporting multiple Product IDs in /new-weight.
```

### Step 3 — Implement

```text
Implement the plan and run the relevant tests.
```

### Step 4 — Verify

Use tests, code review, or a verification skill/subagent to validate the result.

---

## Key Takeaways

Subagents provide three main benefits:

1. **Focused work** — each agent concentrates on a specific task.
2. **Cleaner main context** — intermediate searches and file reads stay isolated.
3. **Concise results** — only the useful summary is returned to the main conversation.

The most important sentence to remember is:

> **Subagents allow Claude Code to delegate a focused task to another isolated Claude context and receive the useful result back.**

And for choosing an agent:

```text
Find / Understand -> Explore
Design            -> Plan
Implement         -> General-purpose / Main Agent
```
