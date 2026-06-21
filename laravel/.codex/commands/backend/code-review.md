# Code Review

## Select This Command When

- Reviewing branch changes against `development`.
- Checking correctness, architecture, test coverage, performance, or maintainability.
- The user requests findings rather than implementation.

Do not modify production code or tests unless the user explicitly asks for fixes.

## Execution Chain

1. Agent: `.agents/reviewer/backend.md`
2. Skills, in priority order:
   1. `.codex/skills/review/backend-architecture-review.md`
   2. `.codex/skills/review/backend-testcovered-review.md`
   3. `.codex/skills/review/backend-performance-review.md`
3. Knowledge and examples: inspect them only when needed to validate a finding.

## Required Outcome

1. Compare the branch against `development`.
2. Inspect changed production code and related tests.
3. Prioritize correctness, architecture, test coverage, performance, then maintainability.
4. Report actionable findings ordered by severity with file and line references.
5. State explicitly when no findings are discovered.
