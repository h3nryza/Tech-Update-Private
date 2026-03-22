# Framework for Responsible AI Coding

> Guidelines for ethical, accountable, and transparent use of AI in software development.

## When to Use AI Coding Tools

### Good Use Cases

- **Boilerplate and scaffolding**: Generating repetitive code structures (CRUD operations, configuration files, test scaffolds) where patterns are well-established.
- **Exploration and prototyping**: Rapidly testing ideas, exploring unfamiliar APIs, or generating proof-of-concept code.
- **Documentation generation**: Writing docstrings, comments, and API documentation from existing code.
- **Test generation**: Creating unit tests for existing functions, especially edge case generation.
- **Code translation**: Converting between languages or frameworks when the logic is well-understood.
- **Accessibility improvements**: Generating accessible markup, ARIA attributes, and compliance-related code (see [Accessibility](accessibility.md)).
- **Learning and understanding**: Using AI to explain unfamiliar codebases or concepts.

### When NOT to Use AI (or Use with Extreme Caution)

- **Security-critical code**: Authentication, authorization, encryption, input sanitization. AI-generated security code has a high rate of subtle vulnerabilities. A 2025 Stanford study found that developers using AI assistants produced significantly less secure code while believing it was more secure.
- **Novel algorithms or business logic**: AI excels at pattern-matching, not invention. Core business logic requires human domain expertise.
- **Regulatory compliance code**: Healthcare (HIPAA), financial (SOX), privacy (GDPR) -- where incorrect implementation has legal consequences.
- **Performance-critical paths**: AI-generated code frequently skips energy-efficient design patterns (87% of samples in a 2025 arXiv study) and may introduce unnecessary computational overhead.
- **Code you do not understand**: If you cannot explain what the AI-generated code does line by line, do not ship it.

## Review Obligations

### The Accountability Principle

**The person who accepts AI-generated code owns it.** This is the foundational ethical principle. You cannot blame the AI for bugs, security vulnerabilities, or biased behavior in code you chose to deploy.

### Mandatory Review Checklist

Before merging any AI-generated or AI-assisted code:

- [ ] **Correctness**: Does it do what it claims? Have you traced the logic?
- [ ] **Security**: Are there injection vectors, hardcoded secrets, insecure defaults, or missing input validation?
- [ ] **Bias**: Does the code make cultural, linguistic, or demographic assumptions? (See [Bias and Fairness](bias_fairness.md))
- [ ] **Licensing**: Could this code be derived from copyleft-licensed training data? Is attribution required?
- [ ] **Performance**: Is the code unnecessarily wasteful in compute, memory, or network usage?
- [ ] **Accessibility**: Does it meet WCAG standards? Does it include proper ARIA attributes?
- [ ] **Tests**: Are there adequate tests? Did you verify the tests actually test the right behavior (not just pass)?
- [ ] **Readability**: Will future developers (including you in 6 months) understand this code?

### Team-Level Obligations

- Establish clear policies on AI tool usage and disclosure.
- Require AI-assisted code to go through the same (or stricter) review process as human-written code.
- Track AI tool usage metrics to understand dependency patterns.
- Conduct periodic audits of AI-generated code quality and security.

## Quality Gates

### Gate 1: Generation

Before generating code with AI:
- Define the problem clearly in natural language first.
- Specify constraints (language version, framework conventions, security requirements).
- Consider whether this is an appropriate use case for AI assistance.

### Gate 2: Review

After receiving AI output:
- Read every line. Do not skim.
- Run the code locally. Do not assume it works.
- Check for hallucinated APIs, deprecated methods, or nonexistent libraries.
- Verify edge cases the AI may not have considered.

### Gate 3: Integration

Before committing:
- Ensure full test coverage, including tests you wrote (not just AI-generated tests).
- Run static analysis, linting, and security scanning.
- Verify the code follows project conventions and architectural patterns.
- Add comments explaining non-obvious logic, even if the AI did not.

### Gate 4: Monitoring

After deployment:
- Monitor AI-generated code paths for unexpected behavior.
- Track error rates compared to human-written code in the same codebase.
- Schedule periodic reviews of older AI-generated code as tools and best practices evolve.

## Attribution and Transparency

### The Attribution Problem

The GitHub Copilot class action lawsuit (filed November 2022, still progressing through courts as of 2026) alleges that AI coding tools reproduce open-source code without proper license attribution. The court has allowed breach-of-contract and license-breach claims to proceed, treating open-source licenses as binding agreements.

### Attribution Guidelines

1. **Disclose AI usage in commit messages or PR descriptions.** A simple tag like `[AI-assisted]` makes the provenance clear.
2. **Document significant AI-generated sections in code comments.** Not every auto-completed line, but substantial blocks.
3. **Maintain a project-level AI usage policy** that documents which tools are used and how.
4. **Respect open-source licenses.** If an AI tool suggests code that closely matches a known open-source implementation, apply the appropriate license.
5. **Do not claim AI-generated code as solely your own work** in contexts where attribution matters (academic work, patent filings, contract deliverables).

### Intellectual Property Considerations

- **Who owns AI-generated code?** This remains legally unsettled. Most AI tool terms of service assign ownership to the user, but this has not been fully tested in court.
- **Copyleft contamination**: If AI training data included GPL-licensed code and the output reproduces it, your project may have license obligations.
- **Patent implications**: AI-generated code may not be patentable in some jurisdictions, as patent law typically requires a human inventor.

## Organizational Policy Template

Teams adopting AI coding tools should document:

```
1. APPROVED TOOLS: Which AI coding tools are authorized for use?
2. USE CASES: What types of tasks may use AI assistance?
3. PROHIBITED USES: What is explicitly off-limits (e.g., security code, PII handling)?
4. DISCLOSURE: How must AI usage be disclosed in code reviews and documentation?
5. REVIEW STANDARDS: What additional review is required for AI-generated code?
6. TRAINING: What training must developers complete before using AI tools?
7. MONITORING: How is AI-generated code tracked and audited over time?
8. INCIDENT RESPONSE: What happens when AI-generated code causes a production issue?
```

## Actionable Guidelines Summary

| Principle | Action |
|-----------|--------|
| Own what you ship | Review all AI output as if you wrote it yourself |
| Disclose AI usage | Tag commits, PRs, and significant code blocks |
| Verify before trusting | Run, test, and trace AI-generated code before merging |
| Respect licenses | Check AI output against known open-source implementations |
| Protect users | Never ship AI-generated security or compliance code without expert review |
| Maintain skills | Use AI to augment, not replace, your understanding |
| Monitor continuously | Track AI-generated code quality over time |

## Sources

- [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/)
- [GitHub Copilot Intellectual Property Litigation](https://www.saverilawfirm.com/our-cases/github-copilot-intellectual-property-litigation)
- [The Ethics of AI Code Review - JetBrains Qodana Blog](https://blog.jetbrains.com/qodana/2026/03/ethics-of-ai-code-review/)
- [Microsoft Responsible AI Principles](https://www.microsoft.com/en-us/ai/principles-and-approach)
- [Ethics of Using AI - AI for Efficient Programming](https://hutchdatascience.org/AI_for_Efficient_Programming/ethics-of-using-ai.html)
- [ACM Code of Ethics](https://www.acm.org/code-of-ethics)
- [Insights from the Pending Copilot Class Action Lawsuit - Finnegan](https://www.finnegan.com/en/insights/articles/insights-from-the-pending-copilot-class-action-lawsuit.html)

---

*Last updated: 2026-03-22*
