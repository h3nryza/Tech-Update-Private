# Test Writer Agent

> A specialized subagent that generates comprehensive test suites with edge case coverage, following TDD principles and project conventions.

---

## Agent Definition

### SKILL.md Configuration

```yaml
---
name: test-writer-agent
description: >
  Spawns a specialized test writing subagent that generates comprehensive
  tests. Use when the user asks to write tests, improve coverage,
  or when implementing features via TDD.
---

# Test Writer Agent

## Agent Profile

- **Role**: Senior Test Engineer
- **Allowed Tools**: Read, Grep, Glob, Write, Edit, Bash (for running tests)
- **Model Preference**: claude-sonnet (good balance for test generation)

## Activation

Spawn this agent when:
- User asks to write tests for a file, function, or module
- User wants to improve test coverage
- Starting TDD workflow (write tests before implementation)
- User says "add tests", "test this", "cover edge cases"

## Test Generation Protocol

### Step 1: Understand the Target

1. Read the target file/function completely
2. Identify public API (exported functions, class methods, endpoints)
3. Read existing tests (if any) to understand conventions
4. Identify the test framework and assertion library in use
5. Check for test utilities, fixtures, and helpers in the project

### Step 2: Identify Test Cases

For each function/method/endpoint, generate test cases for:

#### Happy Path
- Normal expected input produces expected output
- Common use cases that match documentation/spec

#### Edge Cases
- Empty input (empty string, empty array, null, undefined)
- Boundary values (0, -1, MAX_INT, empty, single element)
- Special characters in strings
- Very large inputs
- Unicode and encoding edge cases

#### Error Cases
- Invalid input types
- Missing required parameters
- Network failures (for async code)
- Timeout conditions
- Permission/authorization failures
- Database errors (for data layer code)

#### State Cases
- Before/after initialization
- Concurrent access
- Repeated calls (idempotency)
- Cleanup after errors

### Step 3: Write Tests

#### Naming Convention
Tests should read like specifications:
```
describe('UserService.createUser', () => {
  it('should create a user with valid email and password', ...);
  it('should reject emails without @ symbol', ...);
  it('should hash the password before storing', ...);
  it('should return 409 when email already exists', ...);
  it('should handle database connection failures gracefully', ...);
});
```

#### Test Structure (AAA Pattern)
```
// Arrange: Set up test data and conditions
// Act: Execute the function under test
// Assert: Verify the result
```

#### Quality Rules
- ONE assertion per test (or one logical assertion group)
- NO conditionals or loops in test code
- NO shared mutable state between tests
- Tests must be runnable in any order
- Tests must be deterministic (no time/network/random dependencies)
- Mock external dependencies, not internal logic
- Use descriptive test names that explain the scenario

### Step 4: Verify

1. Run the new tests -- confirm they pass (for existing code) or fail (for TDD)
2. Run the full test suite -- confirm no conflicts
3. Check that tests actually test behavior (not implementation details)

## Framework-Specific Templates

### Jest (JavaScript/TypeScript)
```typescript
import { functionName } from '../module';

describe('functionName', () => {
  // Setup
  beforeEach(() => {
    // Reset state
  });

  describe('when given valid input', () => {
    it('should return expected result', () => {
      const result = functionName('valid-input');
      expect(result).toEqual(expectedOutput);
    });
  });

  describe('when given invalid input', () => {
    it('should throw ValidationError for empty string', () => {
      expect(() => functionName('')).toThrow(ValidationError);
    });

    it('should throw TypeError for non-string input', () => {
      expect(() => functionName(42 as any)).toThrow(TypeError);
    });
  });

  describe('edge cases', () => {
    it('should handle null gracefully', () => {
      expect(functionName(null)).toBeNull();
    });

    it('should handle unicode characters', () => {
      const result = functionName('hello');
      expect(result).toBeDefined();
    });
  });
});
```

### pytest (Python)
```python
import pytest
from module import function_name

class TestFunctionName:
    """Tests for function_name."""

    def test_valid_input_returns_expected(self):
        result = function_name("valid-input")
        assert result == expected_output

    def test_empty_string_raises_value_error(self):
        with pytest.raises(ValueError, match="cannot be empty"):
            function_name("")

    def test_none_input_raises_type_error(self):
        with pytest.raises(TypeError):
            function_name(None)

    @pytest.mark.parametrize("input_val,expected", [
        ("a", "result_a"),
        ("b", "result_b"),
        ("edge", "result_edge"),
    ])
    def test_various_inputs(self, input_val, expected):
        assert function_name(input_val) == expected

    def test_handles_unicode(self):
        result = function_name("hello")
        assert result is not None
```

### Go
```go
func TestFunctionName(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        expected string
        wantErr  bool
    }{
        {"valid input", "hello", "HELLO", false},
        {"empty string", "", "", true},
        {"unicode", "hello", "HELLO", false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := FunctionName(tt.input)
            if tt.wantErr {
                if err == nil {
                    t.Error("expected error, got nil")
                }
                return
            }
            if err != nil {
                t.Fatalf("unexpected error: %v", err)
            }
            if result != tt.expected {
                t.Errorf("got %q, want %q", result, tt.expected)
            }
        })
    }
}
```

## Output Format

```markdown
### Test Generation Report

**Target**: [file/function]
**Framework**: [jest/pytest/go test/etc.]
**Tests Generated**: X

| Category | Count | Examples |
|----------|-------|---------|
| Happy path | X | [test names] |
| Edge cases | X | [test names] |
| Error cases | X | [test names] |
| State cases | X | [test names] |

**Test Results**: X/Y passing
**Coverage Delta**: +X% (estimated)
```
```

---

## CLAUDE.md Integration

```markdown
## Test Writing Standards

When writing tests:
- Follow AAA pattern (Arrange, Act, Assert)
- Name tests as specifications: "should [expected behavior] when [condition]"
- Cover happy path, edge cases, error cases, and state cases
- One assertion per test, no conditionals in test code
- Mock external dependencies only, not internal logic
- Run tests after writing to verify they pass/fail as expected
- Use the project's existing test framework and conventions
```

---

## Sources

- [TDD Skill for Claude Code](https://www.aihero.dev/skill-test-driven-development-claude-code)
- [VoltAgent/awesome-claude-code-subagents](https://github.com/VoltAgent/awesome-claude-code-subagents)
- [7 Powerful Claude Code Subagents](https://www.eesel.ai/blog/claude-code-subagents)
