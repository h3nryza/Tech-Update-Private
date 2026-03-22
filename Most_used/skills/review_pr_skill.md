# PR Review Skill -- Pull Request Review

> Automates structured code review of pull requests with multi-dimensional analysis covering correctness, security, performance, and maintainability.

---

## Why This Skill Exists

Code reviews are time-consuming and inconsistent. This skill provides a systematic review framework that catches common issues, enforces standards, and produces actionable feedback -- whether reviewing your own PR before submission or reviewing others' PRs.

---

## SKILL.md Configuration

```yaml
---
name: review-pr
description: >
  Reviews pull requests with structured, multi-dimensional feedback.
  Use when the user asks to review a PR, check code quality, or
  mentions "review", "PR", or "pull request".
---

# PR Review Skill

## Review Dimensions

Analyze every PR across these dimensions, scoring each 1-5:

### 1. Correctness
- Does the code do what it claims?
- Are edge cases handled?
- Are error paths covered?
- Do types/interfaces match usage?

### 2. Security
- Input validation present?
- SQL injection / XSS / CSRF risks?
- Secrets or credentials exposed?
- Authentication/authorization gaps?
- Dependency vulnerabilities?

### 3. Performance
- N+1 queries or unnecessary loops?
- Missing indexes for new DB queries?
- Large allocations in hot paths?
- Unnecessary re-renders (frontend)?
- Missing pagination for list endpoints?

### 4. Maintainability
- Clear naming and structure?
- Appropriate abstraction level?
- Dead code or commented-out code?
- Consistent with project conventions?
- Adequate comments for complex logic?

### 5. Testing
- Are new features tested?
- Are edge cases covered?
- Do tests actually assert meaningful behavior?
- Are tests independent and deterministic?
- Is test coverage adequate for the change?

## Output Format

Structure your review as follows:

```markdown
## PR Review: [PR Title]

### Summary
[1-2 sentence overview of what the PR does and overall assessment]

### Scores
| Dimension | Score | Notes |
|-----------|-------|-------|
| Correctness | X/5 | Brief note |
| Security | X/5 | Brief note |
| Performance | X/5 | Brief note |
| Maintainability | X/5 | Brief note |
| Testing | X/5 | Brief note |

### Critical Issues (must fix)
- [ ] Issue 1: description + suggested fix
- [ ] Issue 2: description + suggested fix

### Suggestions (nice to have)
- [ ] Suggestion 1
- [ ] Suggestion 2

### Positive Highlights
- Good pattern: description

### Verdict: APPROVE / REQUEST CHANGES / NEEDS DISCUSSION
```

## Workflow

1. Get the PR diff: `gh pr diff <number>` or `git diff main...HEAD`
2. Read the PR description for context: `gh pr view <number>`
3. Identify all changed files and understand the scope
4. For each changed file:
   a. Read the full file for context (not just the diff)
   b. Check each dimension above
   c. Note specific line numbers for issues
5. Check for cross-file issues (broken imports, missing migrations, etc.)
6. Produce the structured review output
7. If requested, post the review as a GitHub comment

## Review Modes

### Quick Review (default)
Focus on critical issues and high-impact suggestions. Suitable for small PRs (<200 lines).

### Deep Review
Full analysis of every dimension. Add architecture impact assessment. Suitable for large PRs or critical paths.

### Security Review
Focus exclusively on security dimensions. Run through OWASP top 10 checklist. Suitable for auth/payment/data-handling code.

## Posting Reviews via GitHub CLI

```bash
# View PR details
gh pr view 123

# Get the diff
gh pr diff 123

# Post a review comment
gh pr review 123 --comment --body "Review content here"

# Approve
gh pr review 123 --approve --body "LGTM - review summary"

# Request changes
gh pr review 123 --request-changes --body "Changes needed - details"
```
```

---

## CLAUDE.md Integration

```markdown
## Code Review Standards

When reviewing PRs:
- Score across 5 dimensions: correctness, security, performance, maintainability, testing
- Always read full files for context, not just diffs
- Flag critical issues (must-fix) separately from suggestions (nice-to-have)
- Include specific line numbers and suggested fixes
- Use `gh pr diff` and `gh pr view` for PR context
- Post structured reviews with a clear verdict
```

---

## Subagent Pattern

For comprehensive reviews, use a multi-agent approach where separate subagents handle each dimension:

```markdown
<!-- In CLAUDE.md or as a skill instruction -->
When doing a deep PR review, spawn subagents for parallel analysis:

1. **Security Reviewer**: Focus only on security concerns. Tools: Read, Grep, Glob.
2. **Performance Reviewer**: Focus only on performance. Tools: Read, Grep, Bash (for profiling).
3. **Test Reviewer**: Focus on test coverage and quality. Tools: Read, Bash (for running tests).

Merge all subagent reports into the final structured review.
```

---

## Sources

- [Claude Code Sub-agents Documentation](https://code.claude.com/docs/en/sub-agents)
- [Code Review Specialist Subagent](https://subagents.app/agents/reviewer)
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
