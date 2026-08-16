# Creating Custom Subagents in Claude Code

Claude Code comes with built-in subagents such as **Explore**, **Plan**, and **General-purpose**, but you can also create your own custom subagents.

Custom subagents are useful when you repeatedly need a specialist for a specific job, such as:

- reviewing code
- writing tests
- checking documentation
- performing security reviews
- validating React/TypeScript patterns
- checking API contracts
- reviewing pull requests

A custom subagent is defined using a Markdown file with YAML frontmatter.

The YAML defines things such as:

- the agent name
- when Claude should use it
- which tools it can access
- which model it should use
- its UI color

The Markdown body defines the subagent's **system prompt**.

---

## Creating a Subagent

The easiest way to create a custom subagent is with:

```text
/agents
```

This opens the Claude Code agent-management interface.

From there, choose:

```text
Create new agent
```

The general flow is:

```text
/agents
   |
   v
Create new agent
   |
   v
Choose scope
   |
   v
Describe the agent
   |
   v
Choose tools
   |
   v
Choose model
   |
   v
Choose color
   |
   v
Agent config created
```

---

## Choosing the Scope

Claude Code asks where the agent should be available.

### Project-level

Use this when the agent is specific to one repository or project.

The config is typically stored inside:

```text
.claude/agents/
```

Example:

```text
my-project/
├── .claude/
│   └── agents/
│       └── react-reviewer.md
├── src/
├── package.json
└── ...
```

This agent is available only in that project.

Good examples:

```text
search-admin-reviewer
weight-module-reviewer
api-contract-checker
```

---

### User-level

Use this when you want the same agent available across multiple projects on your machine.

Typical examples:

```text
typescript-reviewer
security-reviewer
test-reviewer
documentation-reviewer
```

A simple rule:

```text
Specific to one codebase
        |
        v
Project-level

Useful everywhere
        |
        v
User-level
```

---

## Letting Claude Generate the Agent

You can manually write the configuration file, but an easier approach is to let Claude generate it.

For example, when creating the agent, you might describe it like this:

```text
Create a React and TypeScript code reviewer.

It should inspect recently modified frontend code for:

- React hook problems
- unnecessary renders
- memoization issues
- TypeScript safety
- Redux-Saga race conditions
- accessibility
- project convention violations

It should not modify code.
```

Claude can then generate:

```text
name
description
tools
model
color
system prompt
```

You can edit the generated configuration afterward.

---

## Customizing Tool Access

One of the most important parts of creating a subagent is deciding which tools it can use.

Claude Code lets you control tool access.

Typical categories include:

```text
Read-only tools
Edit tools
Execution tools
MCP tools
Other tools
```

Give the agent only the tools it actually needs.

---

## Example: Code Reviewer

A code reviewer generally needs to:

```text
Search code
Read files
Inspect git changes
Possibly run tests
```

It normally does **not** need to modify code.

So you may allow:

```text
Read
Glob
Grep
Bash
```

while leaving editing tools disabled.

The idea is:

```text
Reviewer
   |
   +-- Read       YES
   +-- Search     YES
   +-- Execute    MAYBE
   +-- Edit       NO
```

This helps keep the agent focused on reviewing rather than fixing.

---

## Example: Test Writer

A test-writing subagent may need more capabilities:

```text
Read
Glob
Grep
Edit
Write
Bash
```

because it needs to:

```text
understand existing code
        |
        v
inspect existing tests
        |
        v
write new tests
        |
        v
run test suite
```

---

## Choosing a Model

During agent creation, you can choose which Claude model should power the subagent.

Typical options are:

```text
Haiku
Sonnet
Opus
Inherit
```

### Haiku

Good for:

```text
fast searches
simple checks
lightweight analysis
repetitive tasks
```

Use when speed and cost matter more than deep reasoning.

---

### Sonnet

Good default for most development tasks.

Useful for:

```text
code review
test writing
architecture investigation
debugging
documentation
```

It provides a good balance between speed and reasoning.

---

### Opus

Use for difficult analysis.

Examples:

```text
complex architecture review
security analysis
large refactors
subtle concurrency bugs
cross-service investigation
```

---

### Inherit

The subagent uses the same model as the main Claude Code conversation.

Useful when you do not want to manage model selection separately.

A simple rule:

```text
Simple / fast task      -> Haiku
Normal development task -> Sonnet
Deep analysis           -> Opus
Follow main session     -> Inherit
```

---

## Choosing a Color

Claude Code also lets you assign a color to the subagent.

For example:

```text
purple
blue
green
red
```

The color is mainly a UI aid.

It helps you quickly recognize which agent is active when multiple agents are running.

For example:

```text
Explore Agent       -> blue
React Reviewer      -> purple
Test Agent          -> green
Security Reviewer   -> red
```

The color does not change the agent's behavior.

---

## Subagent Configuration File

A custom agent may look like this:

```yaml
---
name: code-quality-reviewer
description: Use this agent when you need to review recently written or modified code for quality, security, maintainability, and best-practice compliance.
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch
model: sonnet
color: purple
---

You are an expert code reviewer specializing in quality assurance,
security best practices, maintainability, reliability, and performance.

Review recently written or modified code carefully.

Focus on:

- correctness
- security
- maintainability
- performance
- project conventions
- unnecessary complexity

Do not modify files.

Return findings with:
1. Severity
2. File location
3. Problem
4. Why it matters
5. Recommended fix
```

The file would normally be saved as something like:

```text
.claude/agents/code-quality-reviewer.md
```

---

## Understanding the YAML Fields

### `name`

Example:

```yaml
name: code-quality-reviewer
```

This is the unique identifier for the agent.

A good name should clearly describe the agent's purpose.

Examples:

```text
react-reviewer
test-writer
security-reviewer
api-contract-reviewer
documentation-checker
```

You can also explicitly ask Claude to use the agent.

For example:

```text
Use the code-quality-reviewer agent to review my latest changes.
```

Depending on your Claude Code version/interface, custom agents may also be directly selectable or referenceable from the agent UI.

---

## `description`

Example:

```yaml
description: Use this agent when you need to review recently written or modified code for quality, security, maintainability, and best-practice compliance.
```

This is one of the most important fields.

The description helps Claude decide:

> "Should I delegate this task to this agent?"

A vague description:

```yaml
description: Reviews code.
```

is less useful.

A better description:

```yaml
description: Use this agent after frontend code changes to review React hooks, TypeScript safety, rendering performance, accessibility, and project conventions.
```

The second version gives Claude much clearer trigger conditions.

---

## `tools`

Example:

```yaml
tools: Bash, Glob, Grep, Read
```

This determines which tools the agent can use.

For a reviewer:

```yaml
tools: Bash, Glob, Grep, Read
```

may be enough.

For an implementation agent:

```yaml
tools: Bash, Glob, Grep, Read, Edit, Write
```

may be appropriate.

Use the principle:

> Give the subagent the minimum tool access required to perform its job.

---

## `model`

Example:

```yaml
model: sonnet
```

Supported choices commonly include:

```text
haiku
sonnet
opus
inherit
```

Example:

```yaml
model: opus
```

for a complex security reviewer.

---

## `color`

Example:

```yaml
color: purple
```

This controls how the agent is visually identified in Claude Code.

It has no effect on reasoning or permissions.

---

# The System Prompt

Everything after the YAML frontmatter is the subagent's system prompt.

Example:

```text
You are an expert React and TypeScript reviewer.

Focus on:

- hooks correctness
- unnecessary renders
- memoization
- TypeScript safety
- Redux-Saga behavior
- accessibility

Do not modify code.

Report findings using:

Critical
High
Medium
Low
```

This defines:

```text
What the agent is
What it should inspect
What it should avoid
How it should report results
```

You can think of it as:

```text
YAML
  =
When / how Claude should launch the agent

Markdown body
  =
How the agent should behave after it starts
```

---

# Making Claude Use the Agent Automatically

Claude can choose a custom agent automatically based largely on its description.

So the description should clearly state when the agent is useful.

For example:

```yaml
description: Proactively use this agent after significant React or TypeScript changes to review correctness, hooks usage, performance, accessibility, and project conventions.
```

The key idea is not merely the word `proactively`; the important part is giving Claude a **clear trigger condition**.

For example:

```text
after significant React changes
after authentication changes
after API contract changes
before a release
after new tests are written
```

Clear conditions improve delegation.

---

## Adding Trigger Examples

You can make the intended usage even clearer.

For example:

```yaml
description: Proactively review significant frontend changes. Use this agent when the user asks things like "review my React changes", "check this component", "verify the new-weight implementation", or after a substantial React/TypeScript implementation that would benefit from an independent review.
```

This gives Claude concrete examples of when delegation makes sense.

---

# Example: React + TypeScript Reviewer

For a React/Vite project, a custom reviewer could look like:

```yaml
---
name: react-typescript-reviewer
description: Proactively use this agent after significant React or TypeScript changes to review hooks correctness, rendering behavior, TypeScript safety, Redux patterns, accessibility, testing, and project conventions.
tools: Bash, Glob, Grep, Read
model: sonnet
color: purple
---

You are a senior React and TypeScript code reviewer.

Review the requested code without modifying files.

Focus on:

## React

- Hook dependency correctness
- Invalid hook usage
- Unnecessary re-renders
- Memoization problems
- Incorrect React.memo usage
- Component responsibility
- State management
- Side effects
- Event handler stability

## TypeScript

- Unsafe any usage
- Incorrect type assertions
- Non-null assertions
- Missing narrowing
- Weak interfaces
- Incorrect generics
- Reusable type opportunities

## Redux / Redux-Saga

- takeEvery vs takeLatest
- race conditions
- mutation
- inconsistent error handling
- unnecessary dispatches
- selector usage

## Performance

- unnecessary calculations
- unstable props
- unnecessary object creation
- expensive rendering
- missing memoization

## Accessibility

- labels
- keyboard access
- semantic HTML
- ARIA misuse

Report findings as:

### Critical

### High

### Medium

### Low

For each issue include:

- file
- line/location
- issue
- reason
- recommended fix

Do not modify the code.
```

---

# Testing the Subagent

After creating the agent, test it with a real task.

For example, make some React changes and ask:

```text
Review my latest React changes.
```

If the description is good, Claude may decide to delegate to your custom reviewer.

You can also explicitly invoke it:

```text
Use the react-typescript-reviewer agent to review my latest changes.
```

Then check whether it:

```text
reads the correct files
finds relevant issues
avoids unrelated files
uses appropriate tools
returns useful findings
```

---

# If Claude Does Not Use the Agent

The first thing to check is the `description`.

For example, change:

```yaml
description: Reviews React code.
```

to:

```yaml
description: Proactively use this agent after significant React or TypeScript changes, when reviewing pull requests, or when the user asks to check hooks, rendering performance, TypeScript safety, Redux behavior, or accessibility.
```

Be specific about:

```text
WHEN to use it
WHAT it should inspect
WHAT kinds of requests should trigger it
```

---

# Recommended Subagents for a Development Project

A useful setup could look like:

```text
.claude/
└── agents/
    ├── react-typescript-reviewer.md
    ├── test-reviewer.md
    ├── security-reviewer.md
    └── api-reviewer.md
```

With responsibilities:

```text
React Reviewer
      |
      +-- hooks
      +-- rendering
      +-- TypeScript
      +-- Redux

Test Reviewer
      |
      +-- test coverage
      +-- weakened tests
      +-- missing edge cases

Security Reviewer
      |
      +-- authentication
      +-- authorization
      +-- validation
      +-- secrets

API Reviewer
      |
      +-- request contracts
      +-- response contracts
      +-- error handling
      +-- backward compatibility
```

---

# Good Design Principle

Avoid making one giant subagent that tries to do everything.

Less effective:

```text
super-developer-agent

- architecture
- React
- backend
- security
- tests
- documentation
- performance
- deployment
```

Better:

```text
Main Claude
    |
    +-- React Reviewer
    |
    +-- Security Reviewer
    |
    +-- Test Reviewer
    |
    +-- API Reviewer
```

Each one has:

```text
Focused system prompt
Focused tool access
Clear trigger conditions
Focused output
```

That is the real strength of custom subagents.

---

# Recommended Workflow

A strong workflow is:

```text
                 MAIN CLAUDE
                      |
           implements feature
                      |
        +-------------+-------------+
        |                           |
        v                           v
 React Reviewer                Test Reviewer
        |                           |
        +-------------+-------------+
                      |
                      v
                 MAIN CLAUDE
                      |
                 fixes issues
                      |
                      v
                   Verify
```

The main Claude remains responsible for the overall task while specialized subagents independently inspect specific areas.

---

# Key Takeaways

Remember these points:

```text
/agents
   |
   v
Create custom subagent
```

A subagent is typically defined by:

```text
YAML frontmatter
      +
System prompt
```

And:

```text
name        -> identity
description -> when Claude should use it
tools       -> what it is allowed to do
model       -> which Claude model runs it
color       -> UI identification
body        -> system prompt / behavior
```

Most importantly:

> **A good custom subagent should have one clear responsibility, only the tools it needs, clear trigger conditions, and a specific reporting format.**

For development work, think of custom subagents as adding specialist members to your Claude Code team.
