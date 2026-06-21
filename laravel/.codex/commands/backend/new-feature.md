# New Feature

## Select This Command When

- Creating a new module.
- Creating a new API resource.
- Creating a new business process.
- The requested capability does not already exist.

Do not use this command for changes to an existing feature; use
`feature-enhancement`.

## Execution Chain

1. Agent: `.agents/developer/backend.md`
2. Skills, in order:
   1. `.codex/skills/developer/new-feature.md`
   2. `.codex/skills/developer/development.md`
   3. `.codex/skills/developer/testing.md`
3. Knowledge and examples: read only files referenced by those skills.

## Required Outcome

1. Inspect migrations and nearby modules before defining structure.
2. Define or confirm the feature contract.
3. Implement the feature using project architecture.
4. Add feature tests and dedicated focused tests required by the testing standard.
5. Run focused verification, then the complete relevant suite when practical.
6. Report implementation, tests, exclusions, and unresolved ambiguity.

Ask before implementation only when essential business behavior cannot be
discovered from the request, migrations, existing code, or tests.
