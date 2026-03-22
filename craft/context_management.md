# Managing Context Windows

> How to optimize what Claude Code knows, preserves, and forgets -- and why it matters more than your prompts.

---

## Why Context Management Matters

The context window is Claude Code's working memory. Every token spent on irrelevant information is a token unavailable for your actual work. A typical Claude Code session consumes context across four categories:

| Category | Typical Token Cost |
|----------|-------------------|
| System prompt + CLAUDE.md | 2,000 - 5,000 tokens |
| File reads | 500 - 5,000 tokens per file |
| Tool use and outputs | 200 - 2,000 tokens per call |
| Conversation history | Grows continuously |

When the window fills, information gets lost through compaction. The question is not whether you will lose context -- it is whether you will lose the *right* context.

---

## 1. The /compact Command

### What /compact Does

When your conversation approaches the context window limit, Claude Code compacts the conversation:

1. Analyzes the conversation to identify key information
2. Creates a concise summary of previous interactions and code changes
3. Replaces old messages with the summary
4. Preserves CLAUDE.md content (always survives compaction)

### Auto-Compact vs. Manual /compact

**Auto-compact** triggers when context usage reaches approximately 95%. By this point, you may have already lost coherence. **Manual /compact** gives you control over timing and what to preserve.

**Rule of thumb: Compact proactively at 70-80% context usage, not reactively at 95%.**

### Custom Compaction Instructions

You can tell /compact what to preserve:

```
/compact retain the list of all modified files, the current
refactoring plan steps, and the test failures we've been debugging
```

Add standing compaction instructions to your CLAUDE.md:

```markdown
# Compaction Behavior

When compacting, always preserve:
- The full list of files modified in this session
- The current plan or task list with completion status
- Any test commands and their most recent output
- Active debugging hypotheses
- Key decisions made and their rationale
```

### When to /compact vs. /clear vs. New Session

| Situation | Action |
|-----------|--------|
| Same task, long conversation | `/compact` with specific retention instructions |
| Switching to a completely different task | `/clear` or start a new session |
| Context is polluted with wrong approaches | New session with a fresh start |
| Resuming work after a break | `/resume` the named session |
| Complex task with multiple phases | Separate sessions per phase, named with `/rename` |

---

## 2. CLAUDE.md Optimization

CLAUDE.md is the only content guaranteed to survive every compaction. It is loaded at session start and preserved through the entire session. This makes it the most valuable real estate in your context window.

### What Belongs in CLAUDE.md

**Include:**
- Commands to build, test, lint, and deploy
- Code style and naming conventions
- Architecture overview (brief -- link to detailed docs)
- Common patterns and where to find examples
- Project-specific warnings and gotchas
- Testing instructions and conventions
- Compaction preservation instructions

**Exclude:**
- API keys, credentials, secrets
- Detailed implementation documentation (link to it instead)
- Task-specific instructions (put these in conversation or commands)
- Information that changes frequently (use session context instead)
- Redundant information already in config files the AI can read

### The Token Budget Principle

> Every token in CLAUDE.md is a token you cannot use for conversation.

A bloated CLAUDE.md (10,000+ tokens) significantly reduces your effective context window. Aim for 1,000-3,000 tokens: enough to orient Claude, not enough to overwhelm it.

### CLAUDE.md Structure Template

```markdown
# Project Name

Brief one-line description.

## Stack
- Language: TypeScript 5.x, Node 22
- Framework: Express
- Database: PostgreSQL 16
- Testing: Vitest

## Commands
- `npm run dev` -- start dev server
- `npm test` -- run all tests
- `npm run test:watch` -- watch mode
- `npm run lint` -- lint and format check
- `npm run build` -- production build

## Architecture
- `src/routes/` -- API route handlers
- `src/services/` -- business logic
- `src/models/` -- database models and types
- `src/middleware/` -- Express middleware
- See `docs/architecture.md` for detailed overview

## Conventions
- Use `Result<T, E>` pattern for error handling (see src/lib/result.ts)
- All API responses use the shape in `src/types/api-response.ts`
- Database queries go through the repository pattern in `src/repositories/`
- Tests live next to source files: `foo.ts` -> `foo.test.ts`

## Warnings
- Do NOT modify `src/legacy/` -- it's being strangled out
- The `payment` module has external side effects -- always mock in tests
- Run `npm run db:migrate` after changing any model

## Compaction
When compacting, preserve:
- List of files modified this session
- Current task status and remaining steps
- Active test failures and debugging context
```

### Hierarchical CLAUDE.md Files

Claude Code supports CLAUDE.md files at multiple levels:

```
~/.claude/CLAUDE.md          # Global (all projects)
project/CLAUDE.md            # Project root
project/.claude/CLAUDE.md    # Project config directory
project/src/CLAUDE.md        # Subdirectory-specific
```

**Strategy:**
- **Global:** Personal preferences (editor, OS, communication style)
- **Project root:** Build commands, architecture, conventions
- **Subdirectory:** Module-specific patterns and warnings

### The Reference Pattern

Instead of putting everything in CLAUDE.md, reference external files:

```markdown
## Detailed Documentation
- Architecture: Read `docs/architecture.md` when making structural changes
- API Design: Read `docs/api-conventions.md` when modifying endpoints
- Database: Read `docs/database-guide.md` when changing models

Only read these files when working in the relevant area.
```

This keeps CLAUDE.md lean while making detailed context available on demand.

---

## 3. Multi-File Awareness

### The File Reading Strategy

Claude Code reads files on demand. How you manage file reads matters:

**Anti-pattern: Reading everything upfront**
```
Read all files in src/. Then...
```
This wastes context on files that may be irrelevant.

**Pattern: Read targeted files**
```
The bug is in the order processing flow. Read:
1. src/services/order-service.ts
2. src/routes/orders.ts
3. src/models/order.ts

We'll read more files if we need to trace further.
```

### The Breadcrumb Pattern

Start narrow, expand as needed:

```
# Start with the entry point
Read src/routes/orders.ts

# Follow the imports
What does OrderService.process() do? Read that file.

# Follow the dependency
What does the repository's save() method do? Read that.
```

This way, context is spent only on files that matter to the current investigation.

### Keeping Track of Modified Files

In long sessions, explicitly track what has been modified:

```
Before we continue, list all files we've modified in this
session, with a one-line summary of each change.
```

Add this to your compaction instructions so it survives /compact.

---

## 4. Session Segmentation

### When to Start a New Session

Start a new session when:
- You are switching to a fundamentally different task
- Context is polluted with failed approaches
- You have completed a logical phase and want a clean slate
- You are over 70% context usage with significant work remaining

### Passing Context Between Sessions

When ending a session you plan to resume:

```
Create a handoff summary I can use to resume this work:

1. Original goal
2. What was completed
3. What remains
4. Key decisions made (and why)
5. Files modified
6. Any gotchas or context that would be lost
```

Save this somewhere persistent (session log, a markdown file, or your task tracker) and paste it into the next session.

### The Named Session Pattern

```
# Session 1: Planning
/rename auth-redesign-plan
(Plan the refactoring approach)

# Session 2: Phase 1 execution
/rename auth-redesign-phase1
(Extract auth middleware)

# Session 3: Phase 2 execution
/rename auth-redesign-phase2
(Implement new token validation)
```

Resume any session with:
```
claude --resume auth-redesign-phase2
```

---

## 5. Context-Efficient Prompting

### Be Specific, Not Verbose

**Wasteful:**
```
I have this file called user-service.ts which contains a class
called UserService. The class has methods for creating users,
updating users, deleting users, and listing users. I want to
add a method that gets a user by their email address. The method
should query the database and return the user object. Can you
add this method for me?
```

**Efficient:**
```
Add a getUserByEmail(email: string) method to UserService in
src/services/user-service.ts. Follow the pattern of the
existing getUserById method.
```

### Reference Patterns, Don't Explain Them

**Wasteful:**
```
Handle errors by wrapping the return value in a Result type
where Ok contains the success value and Err contains the error.
Use the Result type defined in our codebase...
```

**Efficient:**
```
Handle errors using our Result<T, E> pattern (see src/lib/result.ts).
```

### Use Commands for Repeated Context

If you find yourself providing the same context repeatedly, create a slash command:

**`.claude/commands/add-endpoint.md`:**
```markdown
Add a new API endpoint.

Arguments: $ARGUMENTS

Follow these conventions:
- Route handler in src/routes/ following existing patterns
- Business logic in src/services/
- Input validation with zod schemas in src/schemas/
- Error handling with Result<T, E> pattern
- Tests in the corresponding .test.ts files
- Update the OpenAPI spec in docs/api.yaml
```

Now instead of explaining conventions every time:
```
/project:add-endpoint POST /api/orders/refund
```

---

## 6. Context Management Slash Commands

### `.claude/commands/context-check.md`

```markdown
Report on the current context state:

1. Estimate current context window usage (low/medium/high)
2. List the main topics and files in the current conversation
3. Identify any context that is no longer relevant to the
   current task
4. Recommend whether to /compact, /clear, or continue

If recommending /compact, suggest what to preserve.
```

### `.claude/commands/session-handoff.md`

```markdown
Create a session handoff document for resuming this work later.

Include:
1. **Goal**: What we set out to accomplish
2. **Status**: What was completed and what remains
3. **Decisions**: Key decisions made and their rationale
4. **Files**: All files modified with one-line summaries
5. **Context**: Any non-obvious information needed to resume
6. **Next Steps**: Concrete next actions, in order

Format as markdown suitable for pasting into a new session.
```

---

## Quick Reference

| Action | When | Command |
|--------|------|---------|
| Proactive compact | 70-80% context usage | `/compact retain [critical info]` |
| Switch tasks | Different problem domain | `/clear` or new session |
| Resume work | Continuing previous work | `claude --resume session-name` |
| Name session | Start of session | `/rename descriptive-name` |
| Track changes | Before compact or end of session | "List all modified files" |
| Reduce CLAUDE.md | Context feels cramped | Move details to referenced docs |

## Sources

- [Context Windows - Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Best Practices for Claude Code - Official Docs](https://code.claude.com/docs/en/best-practices)
- [Claude Code Context Window: Optimize Your Token Usage](https://claudefa.st/blog/guide/mechanics/context-management)
- [Claude Code /compact and Auto-Memory - Beam](https://getbeam.dev/blog/claude-code-compact-context-management.html)
- [The Ultimate Guide to Claude Code Context Management - Substratia](https://substratia.io/blog/context-management-guide/)
- [Using CLAUDE.MD Files - Claude Blog](https://claude.com/blog/using-claude-md-files)
- [Writing a Good CLAUDE.md - HumanLayer](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [How to Write a Good CLAUDE.md File - Builder.io](https://www.builder.io/blog/claude-md-guide)
