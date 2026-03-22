# Module 06: Interactive Exercises

---

## How to Use These Exercises

Each exercise is a hands-on project using Claude Code. Work through them in order -- each builds on skills from previous modules.

**Difficulty levels:**
- **Starter** -- Focus on a single Claude Code feature
- **Intermediate** -- Combine multiple features
- **Advanced** -- Full project workflows with MCP, hooks, and agents

---

## Exercise 1: The Perfect CLAUDE.md (Starter)

### Goal

Create the most effective CLAUDE.md for an existing project.

### Steps

1. Open Claude Code in a project (your own, or clone a popular open-source project)
2. **Without** a CLAUDE.md, ask Claude: "What is this project and how is it structured?"
3. Note the quality and accuracy of the response
4. Now create a comprehensive CLAUDE.md with all essential sections
5. Clear the session (`/clear`) and ask the same question
6. Compare the two responses

### Success Criteria

- Claude's second response is noticeably more accurate and detailed
- Claude correctly identifies your tech stack, conventions, and key commands
- Claude follows your stated preferences when you ask it to write code

<details>
<summary>Template to start with</summary>

```markdown
# Project: [Name]

## Description
[What this project does, who uses it, what problem it solves]

## Tech Stack
[List every language, framework, library, tool]

## Project Structure
[Key directories with one-line descriptions]

## Coding Standards
[Your style rules, naming conventions, patterns]

## Commands
[Every useful command: build, test, lint, run, deploy]

## Rules
[ALWAYS do X, NEVER do Y]

## Architecture Decisions
[Why you chose X over Y for key decisions]
```

</details>

---

## Exercise 2: Custom Command Suite (Starter)

### Goal

Create a suite of 5 slash commands for your daily workflow.

### Steps

1. Create `.claude/commands/` in your project
2. Create these commands:
   - `review.md` -- Code review the current diff
   - `test-for.md` -- Write tests for a specified file
   - `explain.md` -- Explain a file or function in detail
   - `refactor.md` -- Refactor a specified file with improvements
   - `commit.md` -- Create a well-formatted git commit

3. Test each command with a real task
4. Refine based on results

### Success Criteria

- Each command produces useful output on first try
- Commands follow your project's conventions
- Commands are specific enough to be better than typing the prompt manually

<details>
<summary>Sample review.md</summary>

```markdown
Review the current git diff (staged and unstaged changes).

For each changed file, check:
1. Logic errors or bugs
2. Security vulnerabilities (XSS, injection, auth bypass)
3. Performance issues (N+1 queries, memory leaks)
4. Style violations per our CLAUDE.md standards
5. Missing error handling
6. Missing or insufficient tests

Format:
## [filename]
- **[CRITICAL/WARNING/INFO]**: [issue] (line ~[number])
  Suggestion: [how to fix]

End with a summary: X files reviewed, Y issues found (Z critical).
```

</details>

---

## Exercise 3: MCP-Powered Workflow (Intermediate)

### Goal

Set up GitHub MCP and use it for a complete PR workflow.

### Steps

1. Create a GitHub personal access token with `repo` scope
2. Configure the GitHub MCP server in `.claude/settings.json`
3. Create a new branch for a feature
4. Make changes to the code
5. Ask Claude to:
   - Create a PR with a proper title and description
   - Check if CI passes
   - List any review comments
   - Address feedback and push fixes

### Success Criteria

- Claude successfully creates the PR without you touching GitHub
- PR description is comprehensive and well-formatted
- Claude can read and respond to review feedback

<details>
<summary>Sample workflow prompts</summary>

```
1. "Create a new branch called feature/improve-error-handling"

2. "Improve error handling in src/api/routes.ts -- add try/catch
    blocks and proper error responses"

3. "Create a PR to main with a detailed description of these changes"

4. "Check the CI status on our PR"

5. "Are there any review comments? If so, address them."
```

</details>

---

## Exercise 4: Safety Hooks (Intermediate)

### Goal

Build a set of hooks that protect your project from common mistakes.

### Steps

1. Create a PreToolUse hook that blocks:
   - Writing to `.env`, `.env.local`, or any file matching `*.secret`
   - Deleting files in the `prisma/migrations/` directory
   - Running `rm -rf` commands

2. Create a PostToolUse hook that:
   - Auto-runs ESLint/Prettier after every file write
   - Logs all file operations to `.claude/activity.log`

3. Test each hook by asking Claude to do the blocked operations
4. Verify the hooks work correctly

### Success Criteria

- Blocked operations are prevented with a clear message
- Auto-formatting runs on every file write
- Activity log captures all operations

<details>
<summary>Hook configuration skeleton</summary>

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "node .claude/hooks/protect-files.js"
      },
      {
        "matcher": "Bash",
        "command": "node .claude/hooks/protect-commands.js"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": ".claude/hooks/auto-format.sh"
      }
    ]
  }
}
```

</details>

---

## Exercise 5: Full Feature with Plan Mode (Intermediate)

### Goal

Use Plan Mode to build a complete feature from requirements to tests.

### Steps

1. Enter Plan Mode: `/plan`
2. Describe a feature:
   > Add a notification system to the app. Users should be able to:
   > - Receive notifications for task assignments
   > - Mark notifications as read
   > - See a notification count badge
   > - View notification history
3. Review Claude's plan -- ask questions and request modifications
4. Approve the plan
5. Let Claude execute each step
6. At each step, review the output before continuing
7. Run the test suite at the end

### Success Criteria

- Plan covers all requirements with a logical implementation order
- Each step produces working code
- Tests pass at the end
- The feature works as described

---

## Exercise 6: Multi-Agent Project (Advanced)

### Goal

Use multiple Claude Code instances to build a feature in parallel.

### Steps

1. Open three terminal windows in the same project

2. **Terminal 1 (Orchestrator):**
   ```
   You are the lead developer. Plan a user dashboard feature with:
   - Profile section (frontend)
   - Activity feed (backend API + frontend)
   - Settings page (full-stack)

   Create the plan and assign each section to a different developer.
   Create the branch and initial file structure.
   ```

3. **Terminal 2 (Backend Developer):**
   ```
   You are working on the backend for the user dashboard.
   Focus only on files in src/api/ and src/services/.
   Check git log periodically for changes from other developers.
   ```

4. **Terminal 3 (Frontend Developer):**
   ```
   You are working on the frontend for the user dashboard.
   Focus only on files in src/components/ and src/pages/.
   Check git log periodically for changes from other developers.
   ```

5. Coordinate through git commits
6. Merge all work together
7. Test the integrated feature

### Success Criteria

- All three terminals produce working code independently
- The code integrates without major conflicts
- The final feature works end-to-end

---

## Exercise 7: CI/CD Integration (Advanced)

### Goal

Set up a Claude Code-powered CI/CD review pipeline.

### Steps

1. Create a GitHub Actions workflow that:
   - Triggers on pull requests
   - Runs Claude Code in headless mode
   - Posts a review comment with findings

2. Create a second workflow that:
   - Triggers on push to main
   - Uses Claude Code to generate a changelog entry
   - Commits the changelog update

3. Create a third workflow that:
   - Runs weekly
   - Uses Claude Code to scan for outdated dependencies
   - Creates a PR with updates if any are found

### Success Criteria

- PR reviews happen automatically with useful feedback
- Changelog stays up to date without manual effort
- Dependencies are regularly checked and updated

<details>
<summary>Starter workflow</summary>

```yaml
name: Claude Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install -g @anthropic-ai/claude-code

      - name: Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          DIFF=$(git diff origin/main...HEAD)
          claude --print "Review these code changes. Be concise.
          Focus on bugs and security issues.

          $DIFF" > review.md

      - name: Comment
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const body = fs.readFileSync('review.md', 'utf8');
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: body,
            });
```

</details>

---

## Final Project: Build and Ship

### Goal

Use everything you've learned to build, test, review, and ship a complete feature.

### Requirements

1. Write a CLAUDE.md (Module 02)
2. Use custom slash commands (Module 03)
3. Use at least one MCP server (Module 04)
4. Have at least one safety hook active (Module 05)
5. Use Plan Mode for the initial design (Module 01)
6. Create a PR through Claude (Module 04)
7. Run automated review (Module 05)

### Suggested Projects

- A personal finance tracker with bank API integration
- A team standup bot for Slack
- A documentation generator that reads your codebase
- A deployment dashboard with real-time status

### Grading Rubric

| Criteria | Points |
|----------|--------|
| CLAUDE.md quality | /15 |
| Custom commands used effectively | /15 |
| MCP integration working | /15 |
| Safety hooks in place | /15 |
| Plan Mode used for design | /10 |
| Code quality (lint, test, review) | /15 |
| Feature completeness | /15 |

---

## Congratulations

You've completed the Claude Code course. You now have the skills to:

- Use Claude Code as a powerful daily driver for development
- Configure it deeply for your specific projects and preferences
- Extend it with MCP servers, hooks, and custom commands
- Run it in CI/CD for automated quality gates
- Coordinate multi-agent workflows for complex projects

### Continue Learning

- **Official docs**: [code.claude.com/docs](https://code.claude.com/docs/en/overview)
- **Claude Code GitHub**: [github.com/anthropics/claude-code](https://github.com/anthropics/claude-code)
- **MCP specification**: [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **Community**: Join the Claude Code channels on Discord and X

### Related Courses

- [Vibe Coding Course](../vibe_coding/README.md) -- The philosophy and broader tools
- [Vibe Ops Course](../vibe_ops/README.md) -- Applying AI to operations
