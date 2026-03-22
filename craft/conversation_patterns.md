# Effective Conversation Patterns

> How to structure your interactions with Claude Code for maximum productivity.

---

## 1. Session Planning

The most impactful thing you can do is spend 2-3 minutes planning before you start typing.

### Before You Open Claude Code

Ask yourself:

- **What is the goal?** Write it in one sentence. If you cannot, you are not ready.
- **What does "done" look like?** Define acceptance criteria (tests pass, endpoint returns X, UI renders Y).
- **What context will Claude need?** Files, architecture decisions, constraints, existing patterns.
- **What is the scope?** Draw a line around what you will and will not touch this session.

### Session Kickoff Template

```
I'm working on [goal]. The relevant files are [list]. The constraint is [constraint].

Here's what "done" looks like:
- [ ] [criterion 1]
- [ ] [criterion 2]
- [ ] [criterion 3]

Let's start in plan mode. Analyze the codebase and propose an approach.
```

### Using /rename for Session Organization

Name your sessions immediately so you can find and resume them later:

```
/rename payment-webhook-handler
/rename fix-auth-race-condition
/rename refactor-database-layer
```

Without names, your session history becomes an indecipherable list of timestamps. Use descriptive names that tell future-you what the session was about.

---

## 2. Plan Mode

Plan mode (`Shift+Tab` twice, or `/plan`) is Claude Code's most underused feature. It forces Claude into read-only mode: it can analyze your codebase and propose a plan, but cannot modify files or run commands.

### When to Use Plan Mode

| Situation | Use Plan Mode? |
|-----------|---------------|
| Single-line change you can describe as an exact diff | No |
| Multi-file change | Yes |
| Unfamiliar code | Yes |
| Architectural decision | Yes |
| You are unsure about the approach | Yes |
| Bug fix where the cause is known | No |
| Bug fix where the cause is unknown | Yes |

### The Plan-Then-Execute Workflow

This is the workflow Boris Cherny (creator of Claude Code) uses himself:

1. **Start in Plan Mode.** Describe what you want.
2. **Iterate on the plan.** Go back and forth until the plan is right. Use `Ctrl+G` to open the plan in your editor for direct editing.
3. **Switch to Auto-Accept Mode.** Let Claude execute the plan.
4. **Review the output.** Verify each change matches the plan.

```
# Step 1: Plan
> (Plan Mode) I need to add rate limiting to our API endpoints.
> The existing middleware is in src/middleware/. We use Express.
> What approach would you recommend?

# Step 2: Iterate
> Good plan, but we need to support per-user limits, not just
> global limits. Also, we need Redis backing for multi-instance
> deployment. Revise the plan.

# Step 3: Execute
> (Auto-Accept Mode) Looks good. Execute the plan.

# Step 4: Review
> Run the test suite and show me the diff summary.
```

### Plan Mode Anti-Patterns

- **Skipping plan mode for multi-file changes.** Claude will confidently solve the wrong problem.
- **Accepting the first plan without iteration.** Plans improve with feedback.
- **Writing the plan yourself.** Let Claude propose; you refine. Your job is judgment, not drafting.
- **Plans that are too large.** If the plan has more than 8-10 steps, break it into multiple sessions.

---

## 3. Course Correction

The ability to course-correct mid-session is what separates effective AI pair programming from frustrating prompt-and-pray.

### Recognizing When to Correct

Watch for these signals:

- **Claude is solving a different problem** than what you intended
- **The approach is getting increasingly complex** when a simpler path exists
- **Claude is modifying files you did not expect** it to touch
- **Test failures are accumulating** rather than decreasing

### Correction Techniques

**Direct redirect:**
```
Stop. That's not what I need. The goal is [restate goal].
The approach should be [simpler approach]. Let's start over
from [specific file].
```

**Scope narrowing:**
```
This is getting too broad. Let's focus only on [specific piece].
We'll handle [other pieces] in a separate session.
```

**Undo and retry:**
```
Undo those last changes. The issue was [diagnosis]. Instead,
let's try [alternative approach].
```

**Context injection:**
```
Wait -- there's context you're missing. [Explain constraint,
existing pattern, or business rule]. Given that, revise
your approach.
```

### The "Why Is This Wrong?" Discipline

When Claude produces incorrect output, resist the urge to just re-prompt with more detail. Instead:

1. **Diagnose why** the output was wrong
2. **Classify the cause:** missing context, wrong approach, ambiguous instruction, or genuine AI limitation
3. **Address the root cause** in your correction

This is faster than trial-and-error and builds your skill at communicating with the AI.

---

## 4. Iterative Refinement

Complex tasks are best accomplished through progressive refinement rather than single-shot attempts.

### The Scaffold Pattern

Build up complexity layer by layer:

```
# Layer 1: Basic structure
Create the endpoint handler with hardcoded responses. Just
the routing and response shape.

# Layer 2: Core logic
Now add the actual business logic. Use the existing
UserService for data access.

# Layer 3: Validation
Add input validation using our existing zod schemas in
src/schemas/.

# Layer 4: Error handling
Add error handling following the pattern in
src/middleware/error-handler.ts.

# Layer 5: Tests
Write tests covering the happy path and the three main
error cases.
```

### The Spike-Then-Refine Pattern

For exploratory work:

```
# Spike: Get something working, quality doesn't matter
Build a quick prototype of [feature]. Don't worry about
error handling, tests, or code quality. I just want to
validate the approach.

# Refine: Now make it production-quality
Good, the approach works. Now let's clean this up:
1. Extract the [X] logic into its own module
2. Add proper error handling
3. Write tests
4. Match the code style of the rest of the project
```

### The Review-Then-Iterate Pattern

Use Claude to review its own output:

```
Before we move on, review the code you just wrote.
Check for:
- Edge cases we missed
- Error handling gaps
- Performance concerns
- Consistency with existing patterns in the codebase
```

---

## 5. Context Management in Conversations

### Front-Loading Context

The single most effective technique: provide all relevant context in your first message.

**Poor:**
```
Fix the login bug.
```

**Better:**
```
There's a bug in the login flow. When a user with an expired
session token tries to log in, they get a 500 error instead
of being redirected to the login page.

The relevant files are:
- src/auth/session.ts (session validation)
- src/middleware/auth.ts (auth middleware)
- src/routes/login.ts (login route handler)

The expected behavior: expired tokens should be cleared and
the user redirected to /login with a flash message.

The actual behavior: the session.validate() call throws an
unhandled exception on expired tokens.
```

### Managing Long Sessions

For sessions that run long (approaching context limits):

1. **Use /compact proactively** before you hit the limit, not after.
2. **Specify what to preserve:** `/compact retain the list of modified files, the test failures, and the current approach`
3. **Summarize periodically:** "Before we continue, summarize where we are and what's left to do."
4. **Break into fresh sessions** for distinct subtasks rather than cramming everything into one session.

### The Checkpoint Pattern

At natural breakpoints, create explicit checkpoints:

```
Let's pause and checkpoint. Summarize:
1. What we've accomplished so far
2. What files were changed
3. What's left to do
4. Any open questions or concerns

I'll use this to resume if we need a new session.
```

### Session Handoff

When continuing work across sessions:

```
I'm resuming work from a previous session. Here's the context:

Goal: [original goal]
Progress: [what was completed]
Remaining: [what's left]
Key decisions: [decisions made in previous session]
Open questions: [unresolved issues]

The relevant files are still [list]. Let's continue with
[next step].
```

---

## 6. Working Configurations

### CLAUDE.md Session Instructions

Add session-relevant instructions to your CLAUDE.md:

```markdown
# Session Conventions

- Always run `npm test` after making changes
- Use the existing error handling pattern in src/middleware/error-handler.ts
- Follow the naming convention: camelCase for functions, PascalCase for types
- When modifying API endpoints, update the OpenAPI spec in docs/api.yaml
```

### Custom Slash Commands

Create reusable session workflows in `.claude/commands/`:

**`.claude/commands/review.md`:**
```markdown
Review the changes made in this session. Check for:
1. Missing error handling
2. Missing tests
3. Inconsistencies with existing code patterns
4. Security concerns
5. Performance issues

Provide a summary with specific file:line references.
```

**`.claude/commands/checkpoint.md`:**
```markdown
Create a checkpoint summary of the current session:
1. Goal and current status
2. Files modified (with brief description of each change)
3. Tests added or modified
4. Remaining work
5. Open questions

Format as markdown suitable for pasting into a session log.
```

**`.claude/commands/plan-review.md`:**
```markdown
Review the current plan before execution:
1. Are all steps necessary?
2. Are any steps missing?
3. Is the order correct?
4. What could go wrong at each step?
5. What's the rollback plan if something fails?
```

---

## Quick Reference

| Pattern | When to Use |
|---------|------------|
| Session Planning | Every non-trivial session |
| Plan Mode | Multi-file changes, unfamiliar code, architectural decisions |
| Direct Redirect | Claude is solving the wrong problem |
| Scope Narrowing | Complexity is growing beyond the session's goal |
| Scaffold Pattern | Building up complex features layer by layer |
| Spike-Then-Refine | Exploratory work where the approach is uncertain |
| Checkpoint Pattern | Long sessions, before /compact, before ending a session |
| Session Handoff | Resuming work across sessions |

## Sources

- [Best Practices for Claude Code - Official Docs](https://code.claude.com/docs/en/best-practices)
- [Plan Mode in Claude Code - codewithmukesh](https://codewithmukesh.com/blog/plan-mode-claude-code/)
- [Claude Code Plan Mode - Steve Kinney](https://stevekinney.com/courses/ai-development/claude-code-plan-mode)
- [Claude Code Planning Mode: Shift+Tab Twice](https://claudefa.st/blog/guide/mechanics/planning-modes)
- [What Actually Is Claude Code's Plan Mode? - Armin Ronacher](https://lucumr.pocoo.org/2025/12/17/what-is-plan-mode/)
- [50 Claude Code Tips and Best Practices](https://www.builder.io/blog/claude-code-tips-best-practices)
- [Claude Code Session Management - Steve Kinney](https://stevekinney.com/courses/ai-development/claude-code-session-management)
