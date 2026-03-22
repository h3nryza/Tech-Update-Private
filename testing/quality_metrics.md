# AI Code Quality Metrics

> What to measure, how to measure, benchmarks, and automated quality gates. Includes working CI/CD configurations for GitHub Actions, GitLab CI, and integration with SonarQube, Codecov, and AI code review tools.

---

## Table of Contents

- [Why AI Code Needs Different Metrics](#why-ai-code-needs-different-metrics)
- [Metric Categories](#metric-categories)
- [Measurement Tools](#measurement-tools)
- [Benchmarks and Targets](#benchmarks-and-targets)
- [Automated Quality Gates](#automated-quality-gates)
- [CI/CD Configurations](#cicd-configurations)
- [AI Code Review Integration](#ai-code-review-integration)
- [Claude Code Skills](#claude-code-skills)

---

## Why AI Code Needs Different Metrics

AI-generated code introduces distinct quality patterns that traditional metrics alone cannot capture:

```mermaid
graph TD
    subgraph "Traditional Code Quality"
        T1[Coverage %]
        T2[Cyclomatic Complexity]
        T3[Duplication %]
        T4[Lint Errors]
    end

    subgraph "AI-Specific Quality Concerns"
        A1[Hallucinated APIs<br/>Functions that don't exist]
        A2[Subtle Logic Errors<br/>Plausible but wrong]
        A3[Security Anti-Patterns<br/>Known vulnerability patterns]
        A4[Stale Patterns<br/>Deprecated APIs/approaches]
        A5[Over-Engineering<br/>Unnecessary complexity]
        A6[Insufficient Error Handling<br/>Happy path bias]
    end

    subgraph "Required Metrics"
        R1[Mutation Score]
        R2[Static Analysis + AI Review]
        R3[Dependency Vulnerability Scan]
        R4[API Existence Verification]
        R5[Property-Based Test Results]
    end

    T1 --> R1
    A1 --> R4
    A2 --> R1
    A3 --> R2
    A4 --> R3
    A5 --> R2
    A6 --> R5

    style A1 fill:#ff6b6b,stroke:#333,color:#fff
    style A2 fill:#ff6b6b,stroke:#333,color:#fff
    style A3 fill:#ff6b6b,stroke:#333,color:#fff
```

**Key insight from research:** Although LLMs can generate functional code, they also introduce software defects including bugs, security vulnerabilities, and code smells. Static analysis is a valuable instrument for detecting these latent defects (arxiv 2508.14727).

---

## Metric Categories

### 1. Coverage Metrics

| Metric | What It Measures | Target | Tool |
|--------|-----------------|--------|------|
| **Statement Coverage** | Lines executed by tests | >= 80% | Istanbul/nyc, coverage.py, JaCoCo |
| **Branch Coverage** | Decision paths taken | >= 75% | Same as above |
| **Function Coverage** | Functions called by tests | >= 90% | Same as above |
| **Mutation Score** | Tests that catch injected bugs | >= 65% | Stryker, PIT, mutmut |
| **Condition Coverage** | Boolean sub-expressions | >= 70% | JaCoCo (Java), specialized tools |

**Why mutation score matters more than line coverage:**

```mermaid
graph LR
    A[100% Line Coverage<br/>0% Mutation Score] --> B[All code runs<br/>but no bugs caught]
    C[80% Line Coverage<br/>70% Mutation Score] --> D[Less code runs<br/>but tests actually work]

    style A fill:#ff6b6b,stroke:#333,color:#fff
    style B fill:#ff6b6b,stroke:#333,color:#fff
    style C fill:#4ecdc4,stroke:#333,color:#fff
    style D fill:#4ecdc4,stroke:#333,color:#fff
```

### 2. Complexity Metrics

| Metric | What It Measures | Target | Tool |
|--------|-----------------|--------|------|
| **Cyclomatic Complexity** | Number of independent paths | <= 10 per function | SonarQube, ESLint, Radon |
| **Cognitive Complexity** | How hard code is to understand | <= 15 per function | SonarQube |
| **Halstead Metrics** | Code vocabulary and volume | Context-dependent | Various |
| **Maintainability Index** | Composite maintainability score | >= 65 (0-100 scale) | Visual Studio, radon |
| **Lines per Function** | Function size | <= 50 | Linters |

### 3. Duplication and Debt Metrics

| Metric | What It Measures | Target | Tool |
|--------|-----------------|--------|------|
| **Code Duplication** | Copy-pasted or near-identical blocks | <= 3% | SonarQube, jscpd |
| **Technical Debt Ratio** | Time to fix all issues / development time | <= 5% | SonarQube |
| **Code Smells** | Maintainability issues | 0 critical, low total | SonarQube, CodeClimate |

### 4. Security Metrics

| Metric | What It Measures | Target | Tool |
|--------|-----------------|--------|------|
| **Vulnerabilities** | Known security issues | 0 critical/high | SonarQube, Snyk, CodeQL |
| **Dependency Vulnerabilities** | Vulnerable packages | 0 critical, fix within SLA | Snyk, Dependabot, npm audit |
| **OWASP Top 10 Coverage** | Common web vulnerabilities tested | All 10 categories | OWASP ZAP, Snyk Code |
| **Secret Detection** | Hardcoded secrets/credentials | 0 findings | GitLeaks, TruffleHog |

### 5. AI-Specific Metrics

| Metric | What It Measures | Target | How to Measure |
|--------|-----------------|--------|---------------|
| **API Existence Score** | Do referenced APIs actually exist? | 100% | TypeScript compiler, import checks |
| **Deprecation Score** | Are deprecated APIs used? | 0 usages | Linters with deprecation rules |
| **Error Handling Ratio** | % of external calls with error handling | >= 90% | Custom lint rules, AI review |
| **Test Semantic Diversity** | How varied are test scenarios? | Embedding similarity < 0.7 | Custom analysis with embeddings |
| **Assertion Specificity** | Are assertions meaningful vs trivial? | 0 trivial assertions | Custom lint rules |

---

## Measurement Tools

### Tool Comparison (2026)

```mermaid
graph TB
    subgraph "Static Analysis"
        SA1[SonarQube<br/>Enterprise standard<br/>AI CodeFix]
        SA2[CodeRabbit<br/>AI-first PR review]
        SA3[CodeAnt.ai<br/>Line-by-line AI review]
        SA4[ESLint/Pylint<br/>Language-specific linting]
    end

    subgraph "Coverage"
        COV1[Codecov<br/>PR coverage feedback<br/>Flaky test detection]
        COV2[Istanbul/nyc<br/>JS/TS coverage]
        COV3[JaCoCo<br/>Java coverage]
        COV4[coverage.py<br/>Python coverage]
    end

    subgraph "Mutation Testing"
        MT1[Stryker<br/>JS/TS/C#/Scala]
        MT2[PIT<br/>Java/Kotlin]
        MT3[mutmut<br/>Python]
    end

    subgraph "Security"
        SEC1[Snyk<br/>Dependencies + Code]
        SEC2[CodeQL<br/>Semantic analysis]
        SEC3[GitLeaks<br/>Secret detection]
        SEC4[OWASP ZAP<br/>DAST scanning]
    end

    subgraph "AI Code Review"
        AI1[CodeRabbit]
        AI2[Codacy]
        AI3[SonarQube AI CodeFix]
        AI4[Greptile]
    end
```

### SonarQube Configuration

SonarQube remains the enterprise standard, now enhanced with AI CodeFix for automated suggestions. Key 2025-2026 updates:
- **Swift support** added
- **40% faster** JavaScript/TypeScript analysis
- **AI CodeFix** generates context-aware fix suggestions
- Supply chain security improvements

```properties
# sonar-project.properties
sonar.projectKey=my-project
sonar.sources=src
sonar.tests=test
sonar.exclusions=**/node_modules/**,**/dist/**,**/*.test.ts
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.typescript.lcov.reportPaths=coverage/lcov.info

# Quality Gate thresholds
sonar.qualitygate.wait=true

# AI-specific rules
sonar.issue.ignore.multicriteria=e1
sonar.issue.ignore.multicriteria.e1.ruleKey=typescript:S1854
sonar.issue.ignore.multicriteria.e1.resourceKey=**/*.generated.ts
```

---

## Benchmarks and Targets

### Recommended Quality Gate Thresholds

```mermaid
graph TD
    subgraph "Tier 1: Minimum (All Projects)"
        T1A[Statement Coverage >= 70%]
        T1B[0 Critical Vulnerabilities]
        T1C[0 Detected Secrets]
        T1D[All Tests Pass]
        T1E[Lint Errors = 0]
    end

    subgraph "Tier 2: Standard (Production)"
        T2A[Branch Coverage >= 75%]
        T2B[Mutation Score >= 50%]
        T2C[Cyclomatic Complexity <= 15]
        T2D[Duplication <= 5%]
        T2E[0 High Vulnerabilities]
        T2F[Technical Debt Ratio <= 10%]
    end

    subgraph "Tier 3: High Quality (Critical Systems)"
        T3A[Branch Coverage >= 85%]
        T3B[Mutation Score >= 70%]
        T3C[Cyclomatic Complexity <= 10]
        T3D[Duplication <= 3%]
        T3E[Full OWASP Top 10 Coverage]
        T3F[Property-Based Tests for Core Logic]
        T3G[Technical Debt Ratio <= 5%]
    end

    T1A --> T2A --> T3A
    T1B --> T2E --> T3E
```

### Industry Benchmarks (2025-2026 Data)

| Metric | Median (Industry) | Top Quartile | AI-Generated Code (Observed) |
|--------|-------------------|-------------|------------------------------|
| Statement Coverage | 65% | 85% | 70.2% (LLM median) |
| Branch Coverage | 50% | 75% | 52.8% (LLM median) |
| Mutation Score | 35% | 65% | 45-62% (varies by model) |
| Cyclomatic Complexity | 8 | 5 | 6-12 (AI tends toward moderate) |
| Code Duplication | 8% | 3% | 5-15% (AI duplicates patterns) |
| Technical Debt Ratio | 12% | 5% | 8-20% (depends on review) |
| Vulnerability Density | 2.5/KLOC | 0.5/KLOC | 1.5-4/KLOC (AI introduces known patterns) |

---

## Automated Quality Gates

### Quality Gate Architecture

```mermaid
flowchart TD
    A[Code Push / PR] --> B[Pre-commit Hooks]
    B --> C[CI Pipeline Triggered]

    C --> D[Stage 1: Fast Checks]
    D --> D1[Lint]
    D --> D2[Type Check]
    D --> D3[Secret Scan]

    D1 --> E{Gate 1 Pass?}
    D2 --> E
    D3 --> E
    E -->|No| F[Block PR, Report]
    E -->|Yes| G[Stage 2: Tests]

    G --> G1[Unit Tests + Coverage]
    G --> G2[Integration Tests]
    G --> G3[E2E Tests - Smoke]

    G1 --> H{Gate 2 Pass?}
    G2 --> H
    G3 --> H
    H -->|No| F
    H -->|Yes| I[Stage 3: Quality Analysis]

    I --> I1[SonarQube Analysis]
    I --> I2[Dependency Scan]
    I --> I3[AI Code Review]

    I1 --> J{Gate 3 Pass?}
    I2 --> J
    I3 --> J
    J -->|No| F
    J -->|Yes| K[Stage 4: Deep Quality]

    K --> K1[Mutation Testing - Changed Files]
    K --> K2[Performance Regression Check]

    K1 --> L{Gate 4 Pass?}
    K2 --> L
    L -->|No| F
    L -->|Yes| M[Ready for Human Review]

    style F fill:#ff6b6b,stroke:#333,color:#fff
    style M fill:#4ecdc4,stroke:#333,color:#fff
```

### Pre-commit Hook Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
        args: ['--maxkb=500']

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks

  - repo: local
    hooks:
      - id: lint
        name: lint
        entry: npm run lint
        language: system
        types: [typescript]
        pass_filenames: false

      - id: typecheck
        name: typecheck
        entry: npx tsc --noEmit
        language: system
        types: [typescript]
        pass_filenames: false

      - id: test-related
        name: test affected files
        entry: npx jest --bail --findRelatedTests
        language: system
        types: [typescript]
```

---

## CI/CD Configurations

### GitHub Actions: Complete Quality Pipeline

```yaml
# .github/workflows/quality-gate.yml
name: Quality Gate

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  # Stage 1: Fast Checks (< 2 min)
  fast-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type Check
        run: npx tsc --noEmit

      - name: Secret Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  # Stage 2: Tests (< 10 min)
  unit-tests:
    needs: fast-checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci

      - name: Unit Tests with Coverage
        run: npx jest --coverage --coverageReporters=lcov --coverageReporters=text-summary

      - name: Upload Coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          file: ./coverage/lcov.info
          fail_ci_if_error: true

      - name: Coverage Gate
        run: |
          COVERAGE=$(npx jest --coverage --coverageReporters=json-summary 2>/dev/null | tail -1)
          STMT=$(node -e "const c=require('./coverage/coverage-summary.json'); console.log(c.total.statements.pct)")
          BRANCH=$(node -e "const c=require('./coverage/coverage-summary.json'); console.log(c.total.branches.pct)")
          echo "Statement: ${STMT}%, Branch: ${BRANCH}%"
          node -e "
            const c = require('./coverage/coverage-summary.json');
            if (c.total.statements.pct < 80) { console.error('Statement coverage below 80%'); process.exit(1); }
            if (c.total.branches.pct < 75) { console.error('Branch coverage below 75%'); process.exit(1); }
          "

  integration-tests:
    needs: fast-checks
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        ports:
          - 6379:6379
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'
      - run: npm ci
      - name: Integration Tests
        run: npx jest --config jest.integration.config.ts
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379

  # Stage 3: Quality Analysis (< 15 min)
  sonarqube:
    needs: unit-tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci
      - run: npx jest --coverage --coverageReporters=lcov

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v3
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}

      - name: SonarQube Quality Gate
        uses: SonarSource/sonarqube-quality-gate-action@v1
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  dependency-scan:
    needs: fast-checks
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  # Stage 4: Deep Quality (runs on main only, < 30 min)
  mutation-testing:
    if: github.ref == 'refs/heads/main'
    needs: [unit-tests, sonarqube]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm'

      - run: npm ci

      - name: Get Changed Files
        id: changed
        run: |
          FILES=$(git diff --name-only HEAD~1 -- 'src/**/*.ts' | grep -v '.test.ts' | tr '\n' ',')
          echo "files=${FILES}" >> $GITHUB_OUTPUT

      - name: Mutation Testing (Changed Files)
        if: steps.changed.outputs.files != ''
        run: |
          npx stryker run --mutate "${{ steps.changed.outputs.files }}"

      - name: Upload Mutation Report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: mutation-report
          path: reports/mutation/
```

### GitLab CI: Quality Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - fast-checks
  - test
  - quality
  - deep-quality

variables:
  NODE_IMAGE: node:22-alpine
  POSTGRES_DB: testdb
  POSTGRES_USER: test
  POSTGRES_PASSWORD: test

# Stage 1: Fast Checks
lint:
  stage: fast-checks
  image: $NODE_IMAGE
  cache:
    key: $CI_COMMIT_REF_SLUG
    paths:
      - node_modules/
  script:
    - npm ci
    - npm run lint
    - npx tsc --noEmit

secret-scan:
  stage: fast-checks
  image:
    name: zricethezav/gitleaks:latest
    entrypoint: [""]
  script:
    - gitleaks detect --source . --verbose

# Stage 2: Tests
unit-tests:
  stage: test
  image: $NODE_IMAGE
  script:
    - npm ci
    - npx jest --coverage --coverageReporters=lcov --coverageReporters=cobertura
  coverage: '/Statements\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/

integration-tests:
  stage: test
  image: $NODE_IMAGE
  services:
    - postgres:16
    - redis:7
  script:
    - npm ci
    - npx jest --config jest.integration.config.ts
  variables:
    DATABASE_URL: postgresql://test:test@postgres:5432/testdb
    REDIS_URL: redis://redis:6379

# Stage 3: Quality Analysis
sonarqube:
  stage: quality
  image:
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  dependencies:
    - unit-tests
  script:
    - sonar-scanner
      -Dsonar.projectKey=$CI_PROJECT_PATH_SLUG
      -Dsonar.sources=src
      -Dsonar.tests=test
      -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
      -Dsonar.qualitygate.wait=true

dependency-scan:
  stage: quality
  image: $NODE_IMAGE
  script:
    - npm ci
    - npm audit --audit-level=high
    - npx snyk test --severity-threshold=high
  allow_failure: false

# Stage 4: Deep Quality (main only)
mutation-testing:
  stage: deep-quality
  image: $NODE_IMAGE
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script:
    - npm ci
    - npx stryker run
  artifacts:
    paths:
      - reports/mutation/
```

### Codecov Configuration

```yaml
# codecov.yml
coverage:
  status:
    project:
      default:
        target: 80%
        threshold: 2%
        flags:
          - unit
    patch:
      default:
        target: 85%
        threshold: 5%

  # Require coverage on new code to be higher than existing
  round: down
  precision: 2

flags:
  unit:
    paths:
      - src/
    carryforward: true
  integration:
    paths:
      - src/
    carryforward: false

# Flaky test detection
test_analytics:
  enabled: true

# Comment configuration
comment:
  layout: "reach,diff,flags,files"
  behavior: default
  require_changes: true
  require_base: true
  require_head: true

# Ignore generated files
ignore:
  - "**/*.generated.ts"
  - "**/dist/**"
  - "**/node_modules/**"
```

---

## AI Code Review Integration

### CodeRabbit Configuration

```yaml
# .coderabbit.yaml
language: en
reviews:
  auto_review:
    enabled: true
    drafts: false
    base_branches:
      - main
      - develop
  path_instructions:
    - path: "src/**/*.ts"
      instructions: |
        Review for:
        - TypeScript best practices
        - Error handling completeness
        - Input validation
        - SQL injection risks
        - Proper async/await usage
    - path: "test/**/*.test.ts"
      instructions: |
        Review for:
        - Test isolation (no shared state)
        - Assertion specificity (no toBeTruthy/toBeDefined)
        - Edge case coverage
        - Mock correctness
        - Flakiness risks (timing, network, randomness)
  tools:
    eslint:
      enabled: true
    biome:
      enabled: true
```

### Custom Quality Gate Script

```bash
#!/bin/bash
# scripts/quality-gate.sh
# Run this locally or in CI to check all quality metrics

set -e

echo "=== Quality Gate Check ==="
echo ""

# 1. Lint
echo "[1/6] Running linter..."
npm run lint || { echo "FAIL: Lint errors found"; exit 1; }
echo "PASS: Lint clean"

# 2. Type check
echo "[2/6] Running type check..."
npx tsc --noEmit || { echo "FAIL: Type errors found"; exit 1; }
echo "PASS: Types clean"

# 3. Tests with coverage
echo "[3/6] Running tests with coverage..."
npx jest --coverage --coverageReporters=json-summary || { echo "FAIL: Tests failed"; exit 1; }

# Parse coverage
STMT=$(node -e "const c=require('./coverage/coverage-summary.json'); console.log(c.total.statements.pct)")
BRANCH=$(node -e "const c=require('./coverage/coverage-summary.json'); console.log(c.total.branches.pct)")
FN=$(node -e "const c=require('./coverage/coverage-summary.json'); console.log(c.total.functions.pct)")

echo "  Statement: ${STMT}% (target: 80%)"
echo "  Branch:    ${BRANCH}% (target: 75%)"
echo "  Function:  ${FN}% (target: 90%)"

node -e "
const c = require('./coverage/coverage-summary.json');
let failed = false;
if (c.total.statements.pct < 80) { console.error('FAIL: Statement coverage below 80%'); failed = true; }
if (c.total.branches.pct < 75) { console.error('FAIL: Branch coverage below 75%'); failed = true; }
if (c.total.functions.pct < 90) { console.error('FAIL: Function coverage below 90%'); failed = true; }
if (failed) process.exit(1);
console.log('PASS: Coverage meets targets');
"

# 4. Security audit
echo "[4/6] Running security audit..."
npm audit --audit-level=high || { echo "FAIL: High severity vulnerabilities found"; exit 1; }
echo "PASS: No high/critical vulnerabilities"

# 5. Secret scan
echo "[5/6] Scanning for secrets..."
if command -v gitleaks &> /dev/null; then
  gitleaks detect --source . --no-git || { echo "FAIL: Secrets detected"; exit 1; }
  echo "PASS: No secrets found"
else
  echo "SKIP: gitleaks not installed"
fi

# 6. Mutation testing (optional, slow)
if [ "$RUN_MUTATION" = "true" ]; then
  echo "[6/6] Running mutation testing..."
  npx stryker run
  # Parse mutation score from Stryker output
  echo "Check reports/mutation/ for detailed results"
else
  echo "[6/6] Mutation testing skipped (set RUN_MUTATION=true to enable)"
fi

echo ""
echo "=== Quality Gate PASSED ==="
```

---

## Claude Code Skills

### Skill: Quality Audit

```markdown
## /quality-audit Skill

Perform a comprehensive quality audit of the codebase or a specific file.

**Usage:**
```
/quality-audit [file-or-directory] [--fix] [--report]
```

**Checks Performed:**
1. **Code Smells**: Long functions, deep nesting, large classes, feature envy
2. **Complexity Hotspots**: Functions with cyclomatic complexity > 10
3. **Duplication**: Copy-pasted code blocks
4. **Error Handling Gaps**: Unhandled promise rejections, missing try/catch on I/O
5. **Type Safety**: Any/unknown usage, type assertions, missing null checks
6. **Security Patterns**: SQL injection, XSS, path traversal, hardcoded secrets
7. **Test Quality**: Coverage gaps, weak assertions, missing edge cases
8. **Dependency Health**: Outdated packages, known vulnerabilities, unused deps
9. **AI-Specific Issues**: Hallucinated APIs, deprecated patterns, over-engineering

**Output:**
- Severity-rated findings (critical / high / medium / low)
- Specific file and line references
- Fix suggestions (applied automatically if --fix flag)
- Quality score summary (A-F rating)
```

### Skill: Quality Gate Setup

```markdown
## /setup-quality-gates Skill

Set up automated quality gates for a project.

**Input:**
- Project language/framework (detected automatically)
- CI/CD platform (GitHub Actions / GitLab CI / Jenkins)
- Quality tier (minimum / standard / high)
- Existing tools (detected from package.json / pyproject.toml)

**Output:**
1. CI/CD pipeline configuration file
2. Pre-commit hook configuration
3. SonarQube/quality tool configuration
4. Codecov/coverage configuration
5. Custom quality gate script
6. Documentation of thresholds and how to adjust them

**Will NOT overwrite existing configurations — generates additions/patches.**
```

### Skill: Metrics Dashboard Generator

```markdown
## /metrics-dashboard Skill

Generate a quality metrics dashboard for the project.

**Output:** Markdown dashboard with:
- Current coverage percentages (statement, branch, function)
- Trend arrows (up/down vs last measurement)
- Complexity hotspot list (top 10 most complex functions)
- Security finding summary
- Test health (pass rate, flaky test count, mutation score)
- Dependency health (outdated count, vulnerability count)
- Mermaid charts for visual representation

**Data Sources:** Coverage reports, lint output, npm audit, git history
```

---

## Metric Anti-Patterns to Avoid

### Goodhart's Law in Testing

"When a measure becomes a target, it ceases to be a good measure."

| Anti-Pattern | Why It's Harmful | Better Approach |
|-------------|-----------------|-----------------|
| Targeting 100% line coverage | Leads to trivial tests and testing implementation details | Target mutation score >= 65% |
| Counting test quantity | Encourages many shallow tests | Measure mutation kill ratio |
| Zero technical debt | Leads to over-engineering and gold-plating | Track debt ratio trend, not absolute |
| Complexity = 0 | Not achievable for real business logic | Focus on cognitive complexity <= 15 |
| Merge only with green AI review | AI reviews have false positives | Use AI review as advisory, human decides |

### The Quality Metrics Pyramid

```mermaid
graph TD
    A[Mutation Score] --> B[Branch Coverage]
    B --> C[Statement Coverage]
    C --> D[Tests Pass]

    E[Security Scan Clean] --> F[Dependency Audit Clean]
    F --> G[Secret Scan Clean]

    H[AI Review Advisory] --> I[Human Review Approval]

    A --> J[Quality Gate]
    E --> J
    H --> J
    I --> J

    style A fill:#4ecdc4,stroke:#333,color:#fff
    style J fill:#4ecdc4,stroke:#333,color:#fff
```

---

## Sources

- [Assessing Quality and Security of AI-Generated Code](https://arxiv.org/abs/2508.14727)
- [Sonar: LLMs for Code Generation Quality Research](https://www.sonarsource.com/resources/library/llm-code-generation/)
- [SonarQube Code Quality Tool](https://www.sonarsource.com/products/sonarqube/)
- [Codecov Coverage Solution](https://about.codecov.io/)
- [Datadog Bits AI Test Optimization](https://www.datadoghq.com/blog/bits-ai-test-optimization/)
- [Best Code Analysis Tools 2026](https://www.qodo.ai/blog/code-analysis-tools/)
- [Best AI Code Review Tools Comparison](https://www.codewalnut.com/learn/best-ai-code-review-tools-comparison)
- [18 Best Code & Test Coverage Tools 2026](https://www.codeant.ai/blogs/best-code-test-coverage-tools-2025)
- [8 Best Unit Test Code Coverage Tools 2026](https://zencoder.ai/blog/unit-test-code-coverage-tools)
