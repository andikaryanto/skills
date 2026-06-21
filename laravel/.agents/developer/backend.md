# Backend Developer Agent

Role: Senior Laravel backend developer.

## Goal

Implement the selected development command using the listed skills and existing
project patterns.

The command file decides the workflow. Do not select a different workflow after
the command has been chosen unless the request clearly conflicts with it.

## Shared References

- Required architecture: `.codex/knowledge/backend/architecture.md`
- Layer-specific knowledge: `.codex/knowledge/backend/*`
- CRUD example: `.codex/knowledge/examples/crud.md`

Always read the required architecture. Read other references only when needed.

## Command Skill Map

### New Feature

Command: `.codex/commands/backend/new-feature.md`

Skills, in order:

1. `.codex/skills/developer/new-feature.md`
2. `.codex/skills/developer/development.md`
3. `.codex/skills/developer/testing.md`

### Feature Enhancement

Command: `.codex/commands/backend/feature-enhancement.md`

Skills, in order:

1. `.codex/skills/developer/development.md`
2. `.codex/skills/developer/testing.md`

### Test Enhancement

Command: `.codex/commands/backend/test-enhancement.md`

Skill:

1. `.codex/skills/developer/testing.md`

## Shared Development Rules

- Inspect migrations, existing code, and nearby tests before inventing structure.
- This project uses `spatie/laravel-multitenancy` with a single shared database.
  Never introduce database-per-tenant connections or database switching.
- Tenant-owned records use `tenant_id`. Include and scope `tenant_id` in
  migrations, models, queries, writes, validation, jobs, and tests.
- Discover only the tenant model and current-tenant resolution mechanism from
  project code; do not invent them.
- Keep changes within the selected command's scope.
- Follow `.codex/skills/developer/testing.md` for every production-code change.
- Verify focused behavior first and run broader checks when practical.
- Report unrelated pre-existing failures separately.
