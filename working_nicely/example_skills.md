# Example Skills: Bridging AI Coding Tools

Claude Code skills live in `.claude/skills/*/SKILL.md` or `.claude/commands/*.md`. These examples create skills that invoke other AI tools.

## Skill 1: Gemini Second Opinion

`.claude/skills/gemini-review/SKILL.md`:

```yaml
---
description: Get a free second opinion from Gemini on the current changes
disable-model-invocation: true
---
```

```markdown
# Gemini Second Opinion

When invoked, do the following:

1. Run `git diff` to capture current changes
2. Write the diff to a temporary file `/tmp/current_diff.txt`
3. Run the following command and capture its output:
   ```
   echo "Review this git diff for bugs, security issues, and improvements:\n$(cat /tmp/current_diff.txt)" | gemini
   ```
4. Present Gemini's feedback alongside your own analysis
5. Highlight any disagreements between the two reviews
```

Invoke with: `/gemini-review`

## Skill 2: Multi-Model Analysis

`.claude/skills/multi-model-analyze/SKILL.md`:

```yaml
---
description: Analyze code using multiple AI models for higher confidence
disable-model-invocation: true
---
```

```markdown
# Multi-Model Analysis

When invoked with a file path or question:

1. Run your own analysis of the target
2. Run `echo "<the question or analysis request>" | gemini` to get Gemini's perspective
3. If Codex CLI is available, run `codex "<the question or analysis request>"` for a third opinion
4. Synthesize all perspectives into a unified report:
   - Points of agreement (high confidence)
   - Points of disagreement (needs human decision)
   - Unique insights from each model
```

Invoke with: `/multi-model-analyze src/auth.ts "Is this authentication implementation secure?"`

## Skill 3: Gemini Research

`.claude/skills/gemini-research/SKILL.md`:

```yaml
---
description: Use Gemini CLI (free, with Google Search) to research a topic
disable-model-invocation: true
---
```

```markdown
# Gemini Research

Gemini CLI has built-in Google Search grounding. Use it for research tasks.

When invoked:

1. Take the user's research question
2. Run: `echo "<research question>. Search the web for current information." | gemini`
3. Capture the output
4. Summarize the findings and apply them to our codebase context
5. If the research leads to actionable changes, propose them
```

Invoke with: `/gemini-research "What are the latest security advisories for Express.js 5?"`

## Skill 4: Codex Auto-Implement

`.claude/skills/codex-implement/SKILL.md`:

```yaml
---
description: Hand off an implementation task to Codex CLI in full-auto mode
disable-model-invocation: true
---
```

```markdown
# Codex Auto-Implement

When invoked:

1. Create a detailed implementation plan based on the user's request
2. Write the plan to `.codex-task.md` in the project root
3. Run: `codex --full-auto "Read .codex-task.md and implement all changes described. Commit each logical change separately."`
4. After Codex finishes, review the git log for new commits
5. Run `git diff HEAD~N..HEAD` where N is the number of new commits
6. Review the changes and flag any issues
7. Clean up: `rm .codex-task.md`
```

Invoke with: `/codex-implement "Add comprehensive input validation to all REST endpoints"`

## Skill 5: Cross-Tool Test Suite

`.claude/skills/cross-tool-test/SKILL.md`:

```yaml
---
description: Generate and verify tests using multiple AI tools
disable-model-invocation: true
---
```

```markdown
# Cross-Tool Test Generation

When invoked with a file or module path:

1. Analyze the target code to understand what needs testing
2. Generate a test plan
3. Run: `codex --auto-edit "Write unit tests for <target> following this test plan: <plan>"`
4. After Codex writes tests, run the test suite
5. If tests fail, fix them yourself
6. Run: `echo "Review these tests for completeness and edge case coverage: $(cat <test-file>)" | gemini`
7. Incorporate Gemini's suggestions for missing test cases
8. Run the final test suite to confirm everything passes
```

Invoke with: `/cross-tool-test src/services/auth-service.ts`

## Skill 6: Memorix Context Loader

`.claude/skills/memorix-context/SKILL.md`:

```yaml
---
description: Load project decisions and context from Memorix shared memory
user-invocable: false
---
```

```markdown
# Memorix Context Loader

This skill runs automatically when Claude Code starts a session.

1. Check if Memorix MCP server is configured
2. If available, retrieve all recent memories for this workspace
3. Summarize key decisions, gotchas, and architectural choices
4. Present this context before proceeding with the user's request
5. Any decisions made during this session should be stored back to Memorix

This ensures continuity across sessions and across tools (Gemini CLI, Codex CLI, etc. all share the same Memorix memory store).
```

## Skill 7: Local Model Fallback

`.claude/skills/local-fallback/SKILL.md`:

```yaml
---
description: Fall back to a local model via Ollama for private/sensitive code
disable-model-invocation: true
---
```

```markdown
# Local Model Fallback

For code that should not leave the local machine:

1. Check if Ollama is running: `ollama list`
2. If not running, start it: `ollama serve &`
3. Use Aider with a local model: `aider --model ollama/deepseek-coder-v2 --message "<task>"`
4. Or use OpenCode: `opencode --provider ollama --model deepseek-coder-v2`
5. Review the changes made by the local model
6. Flag if quality seems insufficient -- suggest switching to a cloud model with user approval
```

Invoke with: `/local-fallback "Fix the database connection handling in this proprietary module"`

## How to Install These Skills

```bash
# Create the skills directories
mkdir -p .claude/skills/gemini-review
mkdir -p .claude/skills/multi-model-analyze
mkdir -p .claude/skills/gemini-research
mkdir -p .claude/skills/codex-implement
mkdir -p .claude/skills/cross-tool-test
mkdir -p .claude/skills/memorix-context
mkdir -p .claude/skills/local-fallback

# Copy each SKILL.md into its directory
# Then they're available as /gemini-review, /multi-model-analyze, etc.
```

## Notes

- Skills that call external tools (gemini, codex) require those tools to be installed and authenticated
- The Gemini-based skills are effectively free (1,000 req/day)
- Codex-based skills require a ChatGPT subscription or OpenAI API key
- Local model skills require Ollama and sufficient hardware (8GB+ RAM recommended)
- Memorix skills require Memorix to be configured as an MCP server in both Claude Code and the other tools

## Sources

- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills)
- [Claude Code Customization Guide](https://alexop.dev/posts/claude-code-customization-guide-claudemd-skills-subagents/)
- [Essential Claude Code Skills and Commands](https://batsov.com/articles/2026/03/11/essential-claude-code-skills-and-commands/)
