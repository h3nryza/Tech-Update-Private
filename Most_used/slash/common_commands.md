# Common Slash Commands -- Detailed Examples

> Working configurations and usage examples for the most popular slash commands in Claude Code.

---

## Built-in Commands: Deep Dive

### /compact -- Context Window Management

The single most important command for long sessions. When your context window fills up, Claude's quality degrades. `/compact` summarizes the conversation while preserving key context.

**Usage**:
```
/compact
```

**With custom instructions**:
```
/compact focus on the database migration work and ignore the earlier debugging
```

**Best practices**:
- Use proactively before context fills up (watch for quality degradation)
- Use between major task switches within the same session
- Provide focus instructions to keep relevant context

---

### /clear -- Fresh Start

Resets the conversation completely. Unlike `/compact`, this preserves nothing.

**Usage**:
```
/clear
```

**When to use**:
- Between completely unrelated tasks
- After two failed correction attempts (start over with a better prompt)
- When Claude is confused or going in circles

---

### /init -- Project Setup

Analyzes your codebase and generates a `CLAUDE.md` file with project context, conventions, and commands.

**Usage**:
```
/init
```

**What it generates**:
- Project description and purpose
- Language and framework detection
- Build and test commands
- Code style conventions
- Directory structure overview

**Best practice**: Run this once per repository, then hand-edit the generated `CLAUDE.md` to add project-specific knowledge Claude cannot infer from code alone.

---

### /diff -- Review Changes

Opens an interactive diff viewer showing all changes Claude has made during the session.

**Usage**:
```
/diff
```

**Best practice**: Use after any significant code generation or refactoring to verify changes before committing.

---

### /model -- Switch Models

Change the Claude model mid-session without losing context.

**Usage**:
```
/model sonnet
/model opus
/model haiku
```

**When to use**:
- Switch to Haiku for quick, simple tasks (cheaper and faster)
- Switch to Opus for complex reasoning or architecture decisions
- Switch to Sonnet for balanced general work

---

### /cost -- Monitor Spend

Shows token usage and estimated cost for the current session.

**Usage**:
```
/cost
```

---

### /loop -- Iterative Execution

Runs a task repeatedly with conditions, useful for iterative refinement.

**Usage**:
```
/loop run tests and fix failures until all tests pass (max 5 iterations)
```

---

### /batch -- Bulk Operations

Process multiple items in a single operation.

**Usage**:
```
/batch add JSDoc comments to all exported functions in src/utils/
```

---

## Custom Command Examples

### /commit -- Conventional Commits

**File**: `.claude/commands/commit.md`

```markdown
Create a git commit for the staged changes following these rules:

1. Run `git status` to see all changes
2. Run `git diff --staged` to understand what will be committed
3. If nothing is staged, suggest which files to stage based on logical grouping
4. Format the commit message as: type(scope): description
   - Types: feat, fix, docs, style, refactor, test, chore, perf, ci, build
   - Scope: the component or area affected
   - Description: imperative mood, lowercase, no period, max 72 chars
5. For multi-line messages, add a blank line then body text wrapped at 80 chars
6. Stage the files and create the commit
7. Show `git log --oneline -3` to confirm
```

**Invocation**:
```
/commit
```

---

### /review -- Code Review

**File**: `.claude/commands/review.md`

```markdown
Review the current changes for code quality. Analyze:

1. Run `git diff` to see all unstaged changes, or `git diff --staged` for staged
2. For each changed file, assess:
   - Correctness: Does the logic work? Edge cases handled?
   - Security: Input validation? Injection risks? Exposed secrets?
   - Performance: Unnecessary loops? Missing indexes? N+1 queries?
   - Style: Consistent naming? Clean structure? Dead code?
   - Tests: Are changes tested? Are tests meaningful?
3. Output a structured review with:
   - Summary (1-2 sentences)
   - Critical issues (must fix before merge)
   - Suggestions (nice to have)
   - Positive highlights
```

**Invocation**:
```
/review
/review src/auth/  # review specific directory
```

---

### /test -- Run and Fix Tests

**File**: `.claude/commands/test.md`

```markdown
Run the project's test suite and handle results:

1. Detect the test framework from package.json, pyproject.toml, Cargo.toml, etc.
2. Run the full test suite
3. If all tests pass, report success with count and duration
4. If tests fail:
   a. Analyze each failure
   b. Determine if it's a test bug or implementation bug
   c. Fix the issue
   d. Re-run tests
   e. Repeat until all pass (max 3 iterations)
5. Report final results

For specific test files, pass the path as an argument:
$ARGUMENTS
```

**Invocation**:
```
/test
/test src/auth/login.test.ts
```

---

### /explain -- Code Explanation

**File**: `.claude/commands/explain.md`

```markdown
Explain the code specified by the user with clarity and depth.

Target: $ARGUMENTS (file path, function name, or concept)

1. Read the target code and its surrounding context
2. Explain at three levels:
   - **High level**: What does this code accomplish? (1-2 sentences)
   - **Step by step**: Walk through the logic with inline commentary
   - **Why**: Explain design decisions, trade-offs, and alternatives
3. Use a mermaid diagram if it helps illustrate flow or architecture
4. Note any potential issues, edge cases, or improvements
5. If the code references other files, briefly explain those connections
```

**Invocation**:
```
/explain src/auth/middleware.ts
/explain "the caching strategy in this project"
```

---

### /fix -- Bug Investigation

**File**: `.claude/commands/fix.md`

```markdown
Investigate and fix the described bug:

Bug description: $ARGUMENTS

1. Understand the bug from the description
2. Search the codebase for relevant code (use Grep, Glob, Read)
3. Form a hypothesis about the root cause
4. Write a failing test that reproduces the bug
5. Implement the fix (minimum change needed)
6. Run the test -- confirm it passes
7. Run the full test suite -- confirm no regressions
8. Summarize: what was wrong, why, and how the fix works
```

**Invocation**:
```
/fix "login fails when email has a plus sign"
/fix "API returns 500 on empty POST body"
```

---

### /doc -- Generate Documentation

**File**: `.claude/commands/doc.md`

```markdown
Generate or update documentation for the target:

Target: $ARGUMENTS

1. If target is a file: generate inline documentation (JSDoc, docstrings, etc.)
2. If target is a directory: generate a README.md for that module
3. If target is "api": generate API documentation from route handlers
4. If no target: generate/update the project root README.md

Documentation must include:
- Purpose and responsibility
- Parameters and return types
- Usage examples
- Error conditions
- Related files/modules
```

**Invocation**:
```
/doc src/utils/validators.ts
/doc src/api/
/doc api
/doc
```

---

### /brainstorm -- Collaborative Planning

**File**: `.claude/commands/brainstorm.md`

```markdown
Brainstorm solutions for the given problem:

Problem: $ARGUMENTS

1. Restate the problem to confirm understanding
2. Generate 3-5 distinct approaches, each with:
   - Brief description
   - Pros and cons
   - Estimated effort (small/medium/large)
   - Risk level (low/medium/high)
3. Present as a comparison table
4. Recommend one approach with reasoning
5. Ask the user which approach to pursue before implementing anything

Do NOT implement anything -- this is planning only.
```

**Invocation**:
```
/brainstorm "how to add real-time notifications to the app"
/brainstorm "migrate from REST to GraphQL"
```

---

### /security -- Security Audit

**File**: `.claude/commands/security.md`

```markdown
Run a security audit on the codebase or specified target:

Target: $ARGUMENTS (or entire project if empty)

Check for:
1. Hardcoded secrets, API keys, tokens, passwords
2. SQL injection vulnerabilities
3. XSS (cross-site scripting) risks
4. CSRF (cross-site request forgery) gaps
5. Insecure dependencies (check package-lock.json / requirements.txt)
6. Missing input validation
7. Improper error handling (leaking stack traces)
8. Insecure file operations
9. Authentication/authorization bypasses
10. Missing rate limiting

Output a structured report with severity levels (CRITICAL / HIGH / MEDIUM / LOW).
```

**Invocation**:
```
/security
/security src/api/auth/
```

---

## Tips for Writing Custom Commands

1. **Use `$ARGUMENTS`** to accept parameters from the user
2. **Be specific** about output format -- Claude follows structured instructions better
3. **Include error handling** -- what should happen if a step fails?
4. **Set boundaries** -- explicitly state what NOT to do (e.g., "do NOT implement anything")
5. **Test your commands** -- iterate on the instructions until the output is consistently good

---

## Sources

- [Claude Code Slash Commands](https://prg.sh/notes/Claude-Code-Slash-Commands)
- [wshobson/commands](https://github.com/wshobson/commands) -- 57 production-ready commands
- [obra/superpowers](https://github.com/obra/superpowers) -- Community skill library
- [50 Claude Code Tips and Best Practices](https://www.builder.io/blog/claude-code-tips-best-practices)
- [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)
