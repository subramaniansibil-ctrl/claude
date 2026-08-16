# Effective Subagent Patterns in Claude Code

Creating a subagent is easy. Making it **focused, predictable, and useful** takes better configuration.

A poorly configured subagent may:

- wander through too many files
- run much longer than necessary
- return vague findings
- rediscover environment problems
- use tools it does not actually need
- produce output the main agent cannot easily act on

The most effective subagents usually follow four patterns:

1. **Specific descriptions**
2. **Structured output formats**
3. **Obstacle reporting**
4. **Limited tool access**

Together, these patterns turn a subagent from a vague helper into a focused worker with a clear job and a clear stopping point.

---

# 1. How Subagent Config Data Gets Used

When you send a message to the main Claude Code conversation, Claude has access to metadata about the available subagents.

The most important routing fields are:

```text
name
description
```

The main agent uses these to decide:

```text
Should I launch a subagent?
        |
        v
Which subagent fits this task?
```

The description has another important role.

When the main agent delegates work, it writes a task prompt for the subagent.

The subagent description helps shape that delegated prompt.

Conceptually:

```text
User request
    |
    v
Main Claude
    |
    +-- Reads available agent names/descriptions
    |
    +-- Chooses matching subagent
    |
    +-- Uses description to help frame delegated task
    |
    v
Subagent receives focused task prompt
```

So the description affects both:

```text
WHEN the subagent is used
        +
WHAT the subagent is told to do
```

This is why the description deserves careful attention.

---

# 2. Writing Descriptions That Shape Better Input Prompts

Consider a code-review agent with this description:

```yaml
description: Reviews code.
```

This tells Claude very little.

The main agent might delegate something vague such as:

```text
Review the current changes.
```

Now the subagent has to figure out:

- which files matter
- what scope to review
- which changes are relevant
- when enough exploration has been done

That encourages wandering.

A stronger description would be:

```yaml
description: Use this agent to review modified code for correctness, security, maintainability, and performance. The parent agent must provide the exact files, diff, or change scope to review. Do not modify files.
```

This description tells the parent agent that it should provide a bounded input.

Instead of:

```text
Review the changes.
```

it can produce something more like:

```text
Review these files:

- src/components/SearchFilter.tsx
- src/sagas/weight.ts
- src/types/weight.ts

Focus on correctness, rendering behavior, TypeScript safety,
and Redux-Saga race conditions.

Do not modify files.
```

This is much easier for the subagent to execute efficiently.

---

## Description Pattern

A strong description should answer:

```text
WHEN should this agent run?
WHAT input should the parent provide?
WHAT should the agent focus on?
WHAT should the agent avoid doing?
```

A useful template is:

```yaml
description: Use this agent when [trigger condition]. The parent agent should provide [required input/scope]. Focus on [responsibilities]. Do not [prohibited actions].
```

Example:

```yaml
description: Use this agent after significant React or TypeScript changes. The parent agent must provide the exact changed files or diff to review. Focus on hooks correctness, rendering performance, type safety, Redux behavior, and accessibility. Do not modify files.
```

---

## Research Agent Example

Weak:

```yaml
description: Searches the web.
```

Better:

```yaml
description: Use this agent for focused external research. The parent agent must provide a precise research question and scope. Return concise findings with sources that can be cited. Do not broaden the research topic unless necessary to answer the question.
```

The phrase:

```text
Return concise findings with sources that can be cited.
```

helps shape the delegated task and the resulting output.

---

# 3. Define a Structured Output Format

One of the strongest improvements you can make to a subagent is to define exactly how it should report its findings.

Without an output format, a subagent may keep researching because it does not know when it has gathered enough information.

A structured format gives it a natural stopping condition.

Conceptually:

```text
No output contract
      |
      v
"Should I search another file?"
"Maybe I need more context."
"Maybe one more command..."
      |
      v
Long-running investigation
```

With a defined output format:

```text
Summary                 ✓
Critical Issues         ✓
Major Issues            ✓
Minor Issues            ✓
Recommendations         ✓
Approval Status         ✓
Obstacles Encountered   ✓

All sections complete
        |
        v
      STOP
```

This makes the agent more predictable and token-efficient.

---

## Example Code Review Output Format

Add something like this to the subagent's system prompt:

```text
Provide your review in this structure:

1. Summary
   - Briefly state what was reviewed.
   - Give the overall assessment.

2. Critical Issues
   - Security vulnerabilities
   - Data integrity risks
   - Logic errors that must be fixed immediately

3. Major Issues
   - Architecture problems
   - Significant correctness concerns
   - Major performance problems
   - Maintainability risks

4. Minor Issues
   - Small quality concerns
   - Documentation gaps
   - Minor optimizations

5. Recommendations
   - Suggested improvements
   - Refactoring opportunities
   - Relevant best practices

6. Approval Status
   - READY TO MERGE
   - READY WITH MINOR CHANGES
   - CHANGES REQUIRED

7. Obstacles Encountered
   - Setup issues
   - Environment quirks
   - Workarounds
   - Special command flags
   - Dependency/import problems
```

Now the reviewer knows exactly what a complete answer looks like.

---

## Output Formats Should Match the Agent's Job

Different agents should have different output contracts.

### Explore Agent

```text
1. Answer
2. Relevant files
3. Execution/data flow
4. Important implementation details
5. Unknowns / unresolved questions
6. Obstacles encountered
```

### Test Agent

```text
1. Tests inspected
2. Tests added/changed
3. Coverage gaps
4. Failing tests
5. Commands run
6. Result
7. Obstacles encountered
```

### Security Reviewer

```text
1. Scope reviewed
2. Critical vulnerabilities
3. High-risk findings
4. Medium/low-risk findings
5. Recommended fixes
6. Security verdict
7. Obstacles encountered
```

The output contract should make it obvious when the task is finished.

---

# 4. Reporting Obstacles

Subagents often discover useful implementation details while working.

Examples:

- a test command needs a special flag
- a dependency is missing
- a generated file must exist first
- a VPN or environment variable affects execution
- an import path behaves differently than expected
- a script only works from a particular directory
- a workaround was necessary

If the subagent solves these problems internally but does not report them, the main agent may have to rediscover the same solution.

That wastes time and context.

---

## Bad Outcome

```text
Subagent
   |
   +-- npm test failed
   +-- discovers --runInBand fixes it
   +-- completes review
   +-- reports only code findings

Main Agent later runs npm test
   |
   +-- encounters same problem again
```

---

## Better Outcome

```text
Subagent
   |
   +-- npm test failed
   +-- discovers --runInBand fixes it
   +-- reports obstacle

Main Agent receives:

Obstacles Encountered:
- Jest hung with the default invocation.
- `npm test -- --runInBand` completed successfully.
```

The main thread can immediately reuse that knowledge.

---

## Recommended Obstacles Section

Include this in subagent output formats:

```text
Obstacles Encountered

Report any obstacle discovered while performing the task, including:

- setup issues
- environment quirks
- workarounds
- commands requiring special flags
- configuration changes required to execute commands
- dependency problems
- import/module-resolution problems
- missing files or generated artifacts

If no obstacles were encountered, explicitly state:

None.
```

This prevents useful operational knowledge from being lost when the subagent context is discarded.

---

# 5. Limit Tool Access

Not every subagent needs every Claude Code tool.

The tool list should match the agent's responsibility.

This provides two important benefits:

1. Prevents unintended side effects
2. Makes the agent's role clearer and more focused

Use the principle of least privilege:

> Give the subagent only the tools required to perform its job.

---

## Research / Read-only Agent

A codebase exploration agent may only need:

```text
Glob
Grep
Read
```

Conceptually:

```text
Research Agent
    |
    +-- Find files
    +-- Search text
    +-- Read files
    |
    x-- Modify files
```

This makes accidental code changes impossible.

Example:

```yaml
tools: Glob, Grep, Read
```

---

## Code Reviewer

A code reviewer may also need `Bash`.

Why?

Because it may need commands such as:

```bash
git diff
git status
git log
npm test
npm run lint
```

A reviewer still normally does not need editing tools.

Example:

```yaml
tools: Bash, Glob, Grep, Read
```

Conceptually:

```text
Code Reviewer
    |
    +-- Read code       ✓
    +-- Search code     ✓
    +-- Inspect diff    ✓
    +-- Run tests       ✓
    +-- Edit files      ✗
    +-- Write files     ✗
```

---

## Styling / Implementation Agent

An agent whose responsibility is to actually modify code needs write capabilities.

Example tools:

```text
Read
Glob
Grep
Edit
Write
Bash
```

Conceptually:

```text
Implementation Agent
    |
    +-- Understand code
    +-- Modify code
    +-- Create files
    +-- Run tests
```

Only grant `Edit` and `Write` when changing source code is part of the agent's explicit responsibility.

---

# 6. Tool Access by Agent Type

A useful reference:

| Agent type | Read | Glob/Grep | Bash | Edit/Write |
|---|---:|---:|---:|---:|
| Explorer / Researcher | ✓ | ✓ | Usually no | ✗ |
| Code Reviewer | ✓ | ✓ | ✓ | ✗ |
| Security Reviewer | ✓ | ✓ | Often ✓ | Usually ✗ |
| Test Reviewer | ✓ | ✓ | ✓ | ✗ |
| Test Writer | ✓ | ✓ | ✓ | ✓ |
| Implementation Agent | ✓ | ✓ | ✓ | ✓ |
| Documentation Reviewer | ✓ | ✓ | Maybe | ✗ |
| Documentation Writer | ✓ | ✓ | Maybe | ✓ |

The exact tools depend on your project, but the key question is always:

```text
Does this agent need this capability to complete its assigned job?
```

If the answer is no, remove it.

---

# 7. Putting Everything Together

An effective code-review subagent might look like this:

```markdown
---
name: code-quality-reviewer
description: Proactively use this agent after significant code changes. The parent agent must provide the exact changed files or diff to review. Focus on correctness, security, maintainability, architecture alignment, and performance. Do not modify files.
tools: Bash, Glob, Grep, Read
model: sonnet
color: purple
---

You are a senior code-quality reviewer.

Review only the scope provided by the parent agent.

Do not modify files.

Use repository context only when needed to understand the supplied changes.
Avoid unrelated exploration.

Provide your review in this exact structure:

1. Summary
   - What was reviewed
   - Overall assessment

2. Critical Issues
   - Security vulnerabilities
   - Data integrity risks
   - Severe logic errors

3. Major Issues
   - Correctness problems
   - Architecture misalignment
   - Significant performance issues
   - Maintainability concerns

4. Minor Issues
   - Small quality concerns
   - Documentation gaps
   - Minor optimizations

5. Recommendations
   - Suggested improvements
   - Refactoring opportunities

6. Approval Status
   Choose one:
   - READY TO MERGE
   - READY WITH MINOR CHANGES
   - CHANGES REQUIRED

7. Obstacles Encountered
   Report:
   - setup issues
   - workarounds
   - environment quirks
   - special command flags
   - dependency/import problems

   If there were none, write: None.
```

Notice how the configuration defines:

```text
WHEN to use it
        |
WHAT input it should receive
        |
WHAT it should inspect
        |
WHAT it must not do
        |
WHAT tools it may use
        |
HOW it must report results
        |
WHEN it should stop
```

That is what makes the agent predictable.

---

# 8. Example: Effective React Reviewer

```markdown
---
name: react-typescript-reviewer
description: Proactively use this agent after significant React or TypeScript changes. The parent agent must provide the exact changed frontend files or diff. Review React correctness, hooks, rendering behavior, TypeScript safety, Redux/Redux-Saga usage, accessibility, and maintainability. Do not modify files.
tools: Bash, Glob, Grep, Read
model: sonnet
color: purple
---

You are a senior React and TypeScript reviewer.

Review only the files or diff supplied by the parent agent.
Expand into related code only when necessary to understand the change.

Do not modify files.

Check:

## React correctness
- Rules of Hooks
- Missing dependencies
- Stale closures
- Incorrect state updates
- Side effects during render

## Rendering and performance
- Unnecessary renders
- Broken memoization
- Unstable callbacks
- Recreated objects/functions passed as props
- Expensive calculations during render

## TypeScript
- Unsafe `any`
- Unsafe assertions
- Non-null assertions
- Missing type narrowing
- Weak or incorrect interfaces

## Redux / Redux-Saga
- Race conditions
- `takeEvery` vs `takeLatest`
- State mutation
- Inconsistent error handling
- Unnecessary dispatches

## Accessibility
- Missing labels
- Keyboard access
- Semantic HTML
- Incorrect ARIA

Return exactly:

1. Summary
2. Critical Issues
3. Major Issues
4. Minor Issues
5. Recommendations
6. Approval Status
7. Obstacles Encountered

For each issue include:

- severity
- file
- line/location when available
- problem
- why it matters
- recommended fix

If there are no issues in a section, write: None.
```

This agent has:

```text
Specific description       ✓
Bounded input               ✓
Structured output          ✓
Obstacle reporting         ✓
Read/review-only tools      ✓
Clear stopping condition   ✓
```

---

# 9. Common Mistakes

## Mistake 1: Generic description

```yaml
description: Helps with code.
```

Problem:

Claude cannot reliably determine when or how to use it.

Better:

```yaml
description: Use this agent after backend API changes. The parent must provide the changed routes, controllers, or diff. Review request validation, authorization, error handling, API compatibility, and security. Do not modify files.
```

---

## Mistake 2: No output contract

```text
Review the code and report anything useful.
```

Problem:

The agent has no natural stopping point.

Better:

```text
Return:
1. Summary
2. Critical issues
3. Major issues
4. Minor issues
5. Recommendations
6. Approval status
7. Obstacles encountered
```

---

## Mistake 3: Giving every tool

```text
Read
Write
Edit
Bash
Everything else
```

Problem:

A reviewer may accidentally act like an implementation agent.

Better:

```yaml
tools: Bash, Glob, Grep, Read
```

for review-only work.

---

## Mistake 4: Hiding workarounds

The subagent discovers a special test command but does not include it in the result.

Problem:

The main agent must rediscover it later.

Better:

Always include:

```text
Obstacles Encountered
```

---

# 10. A Simple Mental Model

Think of a custom subagent like a real team member receiving a ticket.

A bad ticket says:

```text
Review the project.
```

A good ticket says:

```text
Review these three changed files.

Focus on:
- React hooks
- memoization
- TypeScript safety
- Redux-Saga races

Do not edit code.

Return:
- critical findings
- major findings
- minor findings
- recommendation
- approval status
- obstacles encountered
```

The same principle applies to subagents.

---

# Key Takeaways

Effective subagents have four main characteristics:

```text
1. Specific descriptions
        |
        +-- Control when the agent is selected
        +-- Help shape the delegated task prompt

2. Structured output
        |
        +-- Makes results easy to consume
        +-- Gives the agent a stopping point

3. Obstacle reporting
        |
        +-- Preserves workarounds and environment knowledge
        +-- Prevents the main agent from rediscovering problems

4. Limited tool access
        |
        +-- Prevents unintended side effects
        +-- Reinforces the agent's responsibility
```

The most useful rule to remember is:

> **A good subagent has a bounded input, a narrow responsibility, the minimum required tools, and a clearly defined output contract.**

When those four pieces are in place, subagents become much more focused, predictable, and efficient.