# Feature Enhancement

## Select This Command When

- Extending an existing feature.
- Adding business rules to an existing module.
- Adding new endpoints to an existing resource.
- Fixing or changing existing application behavior.

Do not use this command when the requested capability is a new module or resource;
use `new-feature`. If the primary deliverable is tests only, use
`test-enhancement`.

## Execution Chain

1. Agent: `.agents/developer/backend.md`
2. Skills, in order:
   1. `.codex/skills/developer/development.md`
   2. `.codex/skills/developer/testing.md`
3. Knowledge and examples: read only files referenced by those skills.

## Required Outcome

1. Compare the request with current implementation and tests.
2. Identify the smallest existing feature boundary that must change.
3. Implement only the requested behavior.
4. Update feature tests and dedicated focused tests for changed classes.
5. Run focused verification, then the complete relevant suite when practical.
6. Report implementation, tests, and unrelated pre-existing failures separately.
