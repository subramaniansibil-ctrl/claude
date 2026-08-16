# When to Use Subagents in Claude Code

Subagents are useful when they isolate work that your main thread does not need to see in detail.

The central decision rule is simple:

> **If the intermediate work matters, keep the task in the main thread. If only the final result matters, delegate it to a subagent.**

---

## The Decision Rule

Ask:

```text
Does the intermediate work matter?
              |
       +------+------+
       |             |
      YES            NO
       |             |
       v             v
 Main Thread      Subagent
```

Use the main thread when you need to observe and react to each step.

Use a subagent when you mainly care about the final answer and the exploration would otherwise clutter the main context.

---

# When Subagents Shine

Subagents work especially well when:

- You need a result, not a play-by-play of how it was found
- Exploratory work would consume a lot of context
- The task benefits from a fresh perspective
- The task needs a specialized system prompt
- The work is independent enough to summarize cleanly

A useful mental model is:

```text
Exploration separate from execution
             |
             v
          Subagent
```

But:

```text
Step 2 depends heavily on Step 1
Step 3 depends heavily on Step 2
             |
             v
        Main Thread
```

---

# 1. Research and Exploration

Research is one of the best subagent use cases.

Suppose you want to know:

```text
Where is JWT validation implemented?
```

A research subagent may:

```text
Search authentication files
        |
        v
Read middleware
        |
        v
Trace router usage
        |
        v
Inspect helper functions
        |
        v
Follow request flow
        |
        v
Return concise answer
```

Your main thread does not need every search result.

It mainly needs something like:

```text
JWT validation happens in middleware/auth.js.
It is invoked by the API router before protected routes execute.
```

This is a strong subagent task because:

```text
Large exploration cost
        +
Small useful result
        =
Good subagent candidate
```

---

## Example

```text
Use the Explore subagent to determine how authentication works.

Return only:
1. Where JWT validation occurs
2. Where authorization/role checks occur
3. Relevant file paths
4. Request flow
5. Any important caveats

Do not modify files.
```

The exploration stays isolated while the main thread receives the useful findings.

---

# 2. Code Reviews

Code review is another strong subagent use case.

Suppose your main Claude conversation spent many turns implementing a feature:

```text
Main Claude
    |
    +-- designed feature
    +-- edited components
    +-- changed saga
    +-- fixed tests
    +-- discussed tradeoffs
```

If you ask the same context to review the implementation, it already knows all the reasoning behind the choices.

An independent reviewer subagent gets a cleaner perspective:

```text
Main Agent
   |
   +-- implements feature
   |
   v
Reviewer Subagent
   |
   +-- reads git diff
   +-- reads changed files
   +-- applies review criteria
   +-- has no implementation conversation history
   |
   v
Independent findings
```

This separation is useful because the reviewer sees the code more like code written by someone else.

It can focus on the evidence in the diff rather than defending earlier decisions.

---

## Project-specific Review Standards

A custom reviewer can also encode standards that you want applied consistently.

For example:

```text
React Reviewer
    |
    +-- Hooks correctness
    +-- Memoization
    +-- TypeScript safety
    +-- Redux-Saga races
    +-- Accessibility
    +-- Project conventions
```

This makes reviews repeatable across different features and developers.

---

# 3. Tasks That Need a Custom System Prompt

Subagents become particularly useful when a task needs a very different style of behavior from normal Claude Code work.

The value is not simply calling the agent an "expert."

The value comes from giving it genuinely different instructions, context, constraints, or output requirements.

---

## Copywriting Subagent

Claude Code's normal behavior is optimized for concise technical work.

A copywriting agent might instead have instructions such as:

```text
Audience: software engineering managers
Tone: confident but conversational
Avoid technical jargon unless necessary
Use short paragraphs
Lead with user benefit
End with a clear call to action
```

This system prompt meaningfully changes how the work should be performed.

Good use case:

```text
Main Claude       -> engineering work
Copywriting agent -> landing-page or campaign copy
```

---

## Styling / Design-system Subagent

A styling agent may be configured to focus on your project's design-system conventions.

For example:

```text
Styling Agent
    |
    +-- design tokens
    +-- spacing conventions
    +-- typography rules
    +-- reusable components
    +-- responsive patterns
```

Its system prompt can require it to inspect the relevant design-system files before making CSS or UI changes.

The advantage comes from a focused context and specialized instructions, not merely from the title "CSS expert."

---

# When Subagents Hurt

Subagents add overhead:

- a separate context is created
- the parent loses visibility into intermediate work
- discoveries are compressed into a summary
- information can be lost during handoffs

That overhead is worth paying only when isolation provides a real benefit.

Three common anti-patterns are:

1. Expert-only personas
2. Sequential pipelines
3. Test-runner agents

---

# Anti-pattern 1: Expert Claims

A subagent is not automatically useful just because its prompt says:

```text
You are a Python expert.
```

or:

```text
You are a Kubernetes specialist.
```

If it has:

- the same model
- the same tools
- the same context requirements
- no specialized instructions
- no specialized data

then it may provide little benefit over the main thread.

Bad reason to create a subagent:

```text
"I want an expert."
```

Better reasons:

```text
"I want an independent reviewer."
```

```text
"I want a separate research context."
```

```text
"I need a different system prompt and output format."
```

```text
"I want restricted tool permissions."
```

The real value should come from **separation, specialization, permissions, context, or workflow**, not from the label alone.

---

# Anti-pattern 2: Sequential Pipelines

Sequential pipelines can lose important information between agents.

Consider:

```text
Agent 1
Reproduce bug
    |
    v
Agent 2
Debug bug
    |
    v
Agent 3
Fix bug
```

This looks organized, but bug fixing is usually highly dependent on intermediate observations.

For example:

```text
Reproduce
   |
   +-- exact error output
   +-- timing behavior
   +-- unexpected logs
   +-- environment details
   |
   v
Debug
   |
   +-- forms hypothesis
   +-- runs experiment
   +-- changes hypothesis
   |
   v
Fix
```

Compressing each step into a summary can lose subtle details.

A better approach is often:

```text
Main Thread
    |
    +-- reproduce
    +-- inspect output
    +-- form hypothesis
    +-- modify code
    +-- rerun
    +-- react to result
```

because every next action depends on the exact result of the previous action.

---

## When Pipelines Can Work

Multiple subagents are more useful when the tasks are genuinely independent.

For example:

```text
                    Main Agent
                        |
         +--------------+--------------+
         |              |              |
         v              v              v
  Security Review   React Review   API Review
         |              |              |
         +--------------+--------------+
                        |
                        v
                Combined findings
```

Each reviewer can inspect the same diff from a different perspective without depending on another reviewer's discoveries.

That parallelism makes more sense than a dependent chain.

---

# Anti-pattern 3: Test Runner Subagents

Testing often produces intermediate output that directly affects debugging.

Suppose a test fails with:

```text
Expected: 200
Received: 403

at auth.test.ts:87
```

That exact output may immediately tell you what to inspect next.

If a subagent hides it and returns only:

```text
Tests failed.
```

then the main thread has lost critical information.

The parent may have to rerun the tests anyway.

---

## Better Testing Workflow

Keep testing in the main thread when you are actively debugging:

```text
Change code
    |
    v
Run tests
    |
    v
Read exact failure
    |
    v
Modify hypothesis
    |
    v
Fix code
    |
    v
Run again
```

Here, the intermediate test output absolutely matters.

---

## When a Test-related Subagent Can Still Help

A subagent can still be useful for **test review**, where you care about an independent result rather than interactive debugging.

Example:

```text
Review these modified tests.

Check:
- whether assertions are meaningful
- whether tests were weakened
- missing edge cases
- excessive mocking
- missing failure scenarios
```

That is very different from:

```text
Run the tests and debug whatever fails.
```

The first is independent review.
The second is an interactive feedback loop.

---

# Comparing Good and Bad Uses

| Task | Subagent? | Why |
|---|---|---|
| Find where JWT is validated | ✓ Yes | Exploration is noisy; final answer matters |
| Understand an unfamiliar module | ✓ Yes | Many file reads can stay isolated |
| Review a completed feature | ✓ Yes | Fresh context improves independence |
| Security review of a diff | ✓ Yes | Independent specialized criteria |
| Write marketing copy with a special voice | ✓ Yes | Different system prompt adds value |
| Reproduce → debug → fix one bug | Usually no | Each step depends on previous observations |
| Run tests while debugging | Usually no | Exact output matters immediately |
| Create "Python expert" with no other differences | Usually no | Main agent already has the same underlying knowledge |
| Three independent reviews of one diff | ✓ Often | Tasks can run independently |

---

# Practical Decision Questions

Before creating or invoking a subagent, ask:

```text
1. Do I need the intermediate output?

2. Will I need to react to discoveries as they happen?

3. Can the task be summarized without losing important information?

4. Would isolated exploration save significant main-context space?

5. Does the task benefit from an independent perspective?

6. Does it need a meaningfully different system prompt?

7. Is the task independent from the next step?
```

Then decide:

```text
Mostly final-result oriented
        |
        v
     Subagent
```

or:

```text
Highly interactive / sequential
        |
        v
    Main Thread
```

---

# Examples

## Good: Explore Existing Weight Flow

```text
Use the Explore subagent to determine how the legacy Weight module
submits a new Product ID.

Return:
1. Component
2. Action
3. Saga
4. API helper
5. Backend endpoint
6. Request flow

Do not modify files.
```

Why this works:

```text
You need the architecture result.
You do not need every grep and file read.
```

---

## Good: Independent React Review

```text
Use the react-typescript-reviewer subagent to review these changed files:

- NewWeightForm.tsx
- weightSaga.ts
- weightTypes.ts

Return only actionable findings.
Do not modify files.
```

Why this works:

```text
Implementation context stays in main thread.
Review happens from fresh context.
```

---

## Bad: Three-agent Debug Pipeline

```text
Agent 1 -> reproduce issue
Agent 2 -> debug issue
Agent 3 -> implement fix
```

Why this can fail:

```text
Agent 2 needs details Agent 1 observed.
Agent 3 needs reasoning Agent 2 developed.
Each summary can lose information.
```

Keep the debugging loop together unless the steps are genuinely independent.

---

# The Core Mental Model

Think about whether you care about the **journey** or the **destination**.

```text
Need the journey
     |
     v
Main Thread
```

Examples:

```text
debugging
iterative test failures
interactive refactoring
step-by-step architecture decisions
```

Versus:

```text
Need the destination
     |
     v
Subagent
```

Examples:

```text
research result
code review
security assessment
architecture discovery
focused documentation research
```

---

# Key Takeaways

Use subagents for:

```text
Research / exploration
Independent code reviews
Fresh perspectives
Tasks with specialized system prompts
Independent parallel analysis
```

Avoid or be cautious with subagents for:

```text
Generic "expert" personas
Sequential debugging pipelines
Interactive test execution
Tasks where every next step depends on the previous output
```

The most important rule is:

> **If the intermediate work matters to what you will do next, keep it in the main thread. If you only need a clean final result, delegate it.**
