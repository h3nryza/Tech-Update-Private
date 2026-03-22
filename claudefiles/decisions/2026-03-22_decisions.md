# Decisions — 2026-03-22

## Decision 1: Parallel Agent Architecture
**Decision**: Launch 6 specialized research agents simultaneously instead of sequential research
**Alternatives Considered**:
1. Sequential research — slower, but more control over quality
2. Single agent doing everything — would hit context limits
3. 6 parallel agents — fastest, covers most ground
**Chosen**: Option 3
**Why**: User explicitly requested autonomous operation ("run 90 loops", "don't bug the human till morning"). Parallel agents maximize coverage within time constraints. Each agent has a focused scope to avoid overlap.

## Decision 2: Directory Structure
**Decision**: Flat top-level directories for platforms instead of nested
**Alternatives**:
1. `/platforms/aws/`, `/platforms/github/` — more organized but deeper paths
2. `/aws/`, `/github/` — flatter, easier to navigate
**Chosen**: Option 2
**Why**: User's request explicitly listed these as top-level dirs. Flatter structure is more browseable.

## Decision 3: Course Format
**Decision**: Markdown-based interactive courses with collapsible answers
**Alternatives**:
1. Jupyter notebooks — more interactive but requires setup
2. Markdown with `<details>` tags — works everywhere, no setup
3. HTML-based course — richer but harder to maintain
**Chosen**: Option 2
**Why**: Zero dependencies, works in any GitHub renderer, easy to maintain and extend.

## Decision 4: No Commit Prefix
**Decision**: Proceed without commit prefix
**Why**: User said "don't bug me till morning." Saved question in ask_the_human.md. Will use descriptive commit messages without prefix.

## Decision 5: Research-First Approach
**Decision**: Let agents complete research before writing index.md
**Why**: Index needs to link to actual files. Writing it last ensures all links are valid.
