# Prompt Guidance

> Evolving guide to writing effective prompts for AI-assisted coding. Updated each session with lessons learned.

Last updated: 2026-03-23

---

## What Makes a Good Prompt for AI Coding

A good prompt gives the AI everything it needs to produce the right output on the first try. The strongest prompts share these qualities:

### 1. Specificity Over Vagueness

Tell the AI exactly what you want, not roughly what you want. Include the language, framework, file paths, function names, and constraints.

### 2. Context Before Instruction

Provide the relevant background (what exists, what the project does, what constraints apply) before giving the instruction. The AI cannot read your mind about your codebase.

### 3. Explicit Output Format

State what the output should look like: a function, a file, a diff, a table, a list. If you do not specify, the AI will guess -- and often guess wrong.

### 4. Constraints and Boundaries

Tell the AI what NOT to do. "Do not modify files outside of `src/auth/`." "Do not add new dependencies." "Keep the function under 50 lines." Constraints prevent scope creep.

### 5. One Task Per Prompt

Each prompt should request one clear thing. Combining "refactor auth, add tests, and update the docs" in a single prompt leads to partial completions and confusion.

### 6. Success Criteria

Define what "done" looks like. "The test suite passes." "The endpoint returns 200 with this payload." "The function handles null input without throwing."

---

## Common Prompt Mistakes

### Mistake 1: The Vague Request

**Problem:** "Fix the bug" or "Make it better" gives the AI no direction.

**Why it fails:** The AI does not know which bug, what "better" means, or what success looks like.

**Fix:** Be specific about the symptom, location, and expected behavior.

### Mistake 2: Missing Context

**Problem:** "Add authentication to the API" without specifying the framework, existing auth system, or requirements.

**Why it fails:** The AI will make assumptions about your stack, your auth provider, and your requirements. Those assumptions are usually wrong.

**Fix:** Include framework, existing code references, and specific requirements.

### Mistake 3: Multiple Tasks in One Prompt

**Problem:** "Refactor the database layer, add connection pooling, write tests, and update the migration scripts."

**Why it fails:** The AI tries to do everything at once, loses track of one or more tasks, and produces incomplete work.

**Fix:** Break into sequential prompts. Do one thing at a time.

### Mistake 4: No Output Specification

**Problem:** "Create a deployment script" without saying what language, what platform, what format, or where to put it.

**Why it fails:** The AI might produce a bash script when you wanted a GitHub Action, or a Terraform file when you wanted CDK.

**Fix:** Specify the output type, language, and location explicitly.

### Mistake 5: Forgetting Constraints

**Problem:** "Add a caching layer" without specifying memory limits, TTL, invalidation strategy, or what cache backend to use.

**Why it fails:** The AI will pick defaults that may not match your infrastructure or requirements.

**Fix:** State constraints upfront. If it matters, say it.

### Mistake 6: Not Using Plan Mode

**Problem:** Jumping straight into implementation on complex tasks.

**Why it fails:** The AI commits to an approach before exploring alternatives. Bad architectural decisions compound.

**Fix:** Start complex tasks with "Use plan mode. Before implementing, outline your approach and let me review it."

---

## Examples: Bad Prompts Rewritten as Good Ones

### Example 1: Bug Fix

**Bad:**
```
Fix the login bug
```

**Good:**
```
The login endpoint POST /api/auth/login returns 500 when the user's email
contains a plus sign (e.g., user+test@example.com). The error is in
src/auth/login.ts line 42 where the email is used in a regex without escaping.

Fix the regex to handle plus signs and other special characters. Add a test
case for emails with plus signs to tests/auth/login.test.ts.
```

**Why it is better:** Specific symptom, exact file and line, root cause hypothesis, clear deliverable, and test requirement.

---

### Example 2: New Feature

**Bad:**
```
Add rate limiting
```

**Good:**
```
Add rate limiting middleware to the Express.js API in src/middleware/.

Requirements:
- Per-IP rate limiting: 100 requests per 15-minute window
- Use Redis (already configured in src/config/redis.ts) for distributed counting
- Return 429 with Retry-After header when limit is exceeded
- Exempt health check endpoints (/api/health, /api/ready)
- Graceful degradation: if Redis is unavailable, allow all requests (fail open)

Write the middleware in TypeScript. Add unit tests. Do not modify existing middleware files.
```

**Why it is better:** Specifies the tech stack, rate limits, storage, error behavior, exemptions, failure mode, language, test requirement, and scope boundary.

---

### Example 3: Code Review

**Bad:**
```
Review this code
```

**Good:**
```
Review the changes in src/payments/ from the last commit (git diff HEAD~1 -- src/payments/).

Focus on:
1. Security: Are there SQL injection or XSS risks?
2. Error handling: Are all payment API calls wrapped in try/catch with proper logging?
3. Idempotency: Can the charge endpoint be safely retried?

Output a numbered list of issues sorted by severity (critical first).
```

**Why it is better:** Scopes the review, specifies focus areas relevant to the domain (payments), and defines output format.

---

### Example 4: Refactoring

**Bad:**
```
Clean up the code
```

**Good:**
```
Refactor src/services/orderService.ts. It is currently 450 lines with 8 methods
that each do multiple things.

Goals:
- Extract payment logic into src/services/paymentService.ts
- Extract inventory logic into src/services/inventoryService.ts
- Keep orderService.ts as the orchestrator that calls the extracted services
- Maintain all existing tests passing (tests/services/orderService.test.ts)
- Do not change any public method signatures

Use plan mode first to show me the extraction plan before making changes.
```

**Why it is better:** States the current problem, defines the target architecture, preserves backward compatibility, protects tests, and uses plan mode.

---

## Prompt Patterns Reference

For a comprehensive library of 24 proven prompt patterns with templates and examples, see:

**[patterns/prompt_patterns.md](../../patterns/prompt_patterns.md)**

Key patterns include:
- **SPARC Framework** -- Specification, Pseudocode, Architecture, Refinement, Completion
- **CIF Pattern** -- Context, Instruction, Format (minimum viable structure for any prompt)
- **RICE Pattern** -- Role, Instruction, Context, Expectation
- **Chain of Thought** -- Step-by-step reasoning for complex logic
- **Test-First Prompting** -- Write tests before implementation
- **Scope Fence** -- Explicitly bound what the AI should and should not touch
- **Plan Mode / Execute Mode Split** -- Separate planning from implementation

See also:
- [patterns/anti_patterns.md](../../patterns/anti_patterns.md) for what to avoid
- [craft/conversation_patterns.md](../../craft/conversation_patterns.md) for session-level strategies
- [craft/context_management.md](../../craft/context_management.md) for managing long conversations

---

## Session Log

*Prompt quality observations from each session are appended below.*

### 2026-03-23

- Initial guidance document created with foundational patterns extracted from the Claude Framework research.

---

*Part of the [Claude Framework](../../README.md). Evolving file -- updated each session per the [Prime Directive](../../new_prime_directive.md).*
