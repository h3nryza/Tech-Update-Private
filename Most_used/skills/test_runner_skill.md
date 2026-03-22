# Test Runner Skill -- Test Execution and TDD

> Drives test-driven development cycles and automates test execution with feedback loops. One of the most impactful skills for code quality.

---

## Why This Skill Exists

Without explicit test-running instructions, Claude often writes code and assumes it works. This skill enforces a RED-GREEN-REFACTOR cycle and ensures tests are run after every change, creating the feedback loop that is critical for reliable AI-assisted development.

---

## SKILL.md Configuration

```yaml
---
name: test-runner
description: >
  Runs tests and drives TDD (Test-Driven Development) cycles.
  Use when the user asks to write tests, run tests, do TDD,
  or when implementing any new feature or fixing a bug.
---

# Test Runner Skill

## TDD Cycle: RED -> GREEN -> REFACTOR

### Phase 1: RED (Write a Failing Test)

1. Understand the requirement or bug to be fixed
2. Write ONE test that captures the expected behavior
3. Run the test suite -- confirm the new test FAILS
4. If the test passes immediately, the test is not capturing new behavior -- rewrite it

### Phase 2: GREEN (Make It Pass)

1. Write the MINIMUM code needed to make the failing test pass
2. Do not add extra features, optimizations, or abstractions
3. Run the full test suite -- ALL tests must pass
4. If any test fails, fix the implementation (not the test) until green

### Phase 3: REFACTOR (Clean Up)

1. Only enter this phase when ALL tests are green
2. Look for:
   - Duplicated code that can be extracted
   - Unclear naming that can be improved
   - Overly complex logic that can be simplified
   - Missing abstractions that would aid readability
3. After each refactoring step, run tests again to confirm still green
4. Never change behavior during refactoring -- tests must stay green

## Test Commands by Framework

Detect the project's test framework and use the appropriate command:

| Framework | Language | Command |
|-----------|----------|---------|
| Jest | JavaScript/TypeScript | `npx jest` or `npm test` |
| Vitest | JavaScript/TypeScript | `npx vitest run` |
| pytest | Python | `python -m pytest` |
| unittest | Python | `python -m pytest` (works with unittest too) |
| RSpec | Ruby | `bundle exec rspec` |
| Go test | Go | `go test ./...` |
| Cargo test | Rust | `cargo test` |
| JUnit | Java | `./gradlew test` or `mvn test` |
| PHPUnit | PHP | `./vendor/bin/phpunit` |
| ExUnit | Elixir | `mix test` |

### Running Specific Tests

Always prefer running specific tests during development for faster feedback:

```bash
# Jest -- single file
npx jest path/to/file.test.ts

# Jest -- single test by name
npx jest -t "should handle empty input"

# pytest -- single file
python -m pytest tests/test_auth.py

# pytest -- single test
python -m pytest tests/test_auth.py::test_login_success -v

# Go -- single package
go test ./pkg/auth/...

# Go -- single test
go test ./pkg/auth/ -run TestLoginSuccess
```

## Workflow for Bug Fixes

1. **Reproduce**: Write a test that demonstrates the bug (should fail)
2. **Fix**: Implement the fix (test should pass)
3. **Regression**: Run full suite to ensure no regressions
4. **Commit**: Commit test and fix together

## Workflow for New Features

1. **Spec**: List out expected behaviors as test names
2. **Iterate**: For each behavior, do one RED-GREEN cycle
3. **Refactor**: After all behaviors pass, clean up
4. **Integration**: Run full suite including integration tests
5. **Commit**: Commit in logical chunks

## Test Quality Checklist

- [ ] Tests have descriptive names that read like specifications
- [ ] Each test asserts ONE behavior
- [ ] Tests are independent (no shared mutable state)
- [ ] Tests are deterministic (no time-dependent, network-dependent, or random behavior)
- [ ] Edge cases covered: empty input, null/undefined, boundary values, error cases
- [ ] Assertions are specific (not just "truthy" -- check exact values)
- [ ] No test logic (conditionals, loops) in test code
- [ ] Mocks/stubs are minimal and targeted

## Output Format

After each test run, report:

```markdown
### Test Results
- **Status**: PASS / FAIL
- **Passed**: X / Y
- **Failed**: [list of failed test names]
- **Duration**: Xs
- **Next Step**: [what to do based on results]
```
```

---

## CLAUDE.md Integration

```markdown
## Testing Requirements

- Follow TDD: write failing test first, then implementation, then refactor
- Run tests after EVERY code change -- never assume code works
- Use `npm test` for JS/TS, `python -m pytest` for Python, `go test ./...` for Go
- Run specific tests during development, full suite before committing
- Every bug fix must include a regression test
- Test names should read like specifications
```

---

## Hook Integration

Auto-run tests after every file edit:

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "if echo '$ARGUMENTS' | jq -r '.file_path' | grep -qE '\\.(ts|js|py|go|rs)$'; then npm test 2>&1 | tail -20; fi"
      }
    ]
  }
}
```

---

## Sources

- [TDD Skill for Claude Code](https://www.aihero.dev/skill-test-driven-development-claude-code)
- [obra/superpowers](https://github.com/obra/superpowers) -- Battle-tested TDD skill
- [Claude Code Best Practices](https://code.claude.com/docs/en/best-practices) -- Feedback loops section
