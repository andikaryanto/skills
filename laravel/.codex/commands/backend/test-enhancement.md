# Test Enhancement

## Select This Command When

- Adding or updating tests only for code changed against the `development` branch.
- A changed class, public method, branch, or failure path is missing coverage.
- A changed class has endpoint coverage but no focused class test.

Do not modify production behavior. If production changes are the primary
deliverable, use `feature-enhancement`.

## Execution Chain

1. Agent: `.agents/developer/backend.md`
2. Skill: `.codex/skills/developer/testing.md`
3. Knowledge and examples: read only files referenced by the skill.

## Required Workflow

1. Run `git diff --name-status development...HEAD` and inspect changed production PHP files.
2. Build a changed-class coverage checklist before editing tests.
3. Map every changed concrete application class to:
   - its dedicated test class;
   - public methods and meaningful branches;
   - existing feature or integration coverage;
   - missing focused unit coverage.
4. Add or update only tests related to the branch diff.
5. Run the focused tests first, then the complete test suite when practical.
6. Report:
   - tests added or updated;
   - changed classes intentionally excluded and the reason;
   - focused and full-suite results;
   - unrelated pre-existing failures separately.

Do not ask the user which tests to add when the missing coverage can be determined
from the diff and testing standard. Ask only when expected behavior is genuinely
ambiguous.

Feature or endpoint coverage does not replace a dedicated class test. It may
satisfy integration coverage, but changed class behavior must also be exercised
in isolation as required by the testing standard.
