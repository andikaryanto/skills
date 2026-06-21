# Laravel Workflow

This documentation defines reusable Laravel architecture and development workflow standards.

## Documents

- [Architecture](.codex/knowledge/backend/architecture.md): layer structure, dependency direction, class responsibilities, and anti-patterns.
- [Development Workflow](.codex/skills/developer/development.md): feature implementation flow from route to tests.
- [Naming](codex/knowledge/backend/naming.md): naming conventions for classes, methods, routes, and request attributes.
- [API Response](.codex/knowledge/backend/api-response.md): API response standards and `LaravelCommon\Responses` usage.
- [Testing](codex/skills/developer/naming.mdtesting.md): testing standards for endpoints, services, repositories, and queries.
- [CRUD Example](.codex/knowledge/examples/crud.md): example implementation for a simple CRUD domain.

## Layer Guides

- [Controller](.codex/knowledge/backend/controller.md)
- [Middleware](.codex/knowledge/backend/middleware.md)
- [Model](.codex/knowledge/backend/model.md)
- [Repository](.codex/knowledge/backend/repository.md)
- [Query](.codex/knowledge/backend/query.md)
- [Service](.codex/knowledge/backend/service.md)
- [UnitOfWork Persistence](.codex/knowledge/backend/unit-of-work.md)
- [ViewModel](.codex/knowledge/backend/view-model.md)

## Recommended Reading Order

1. `.codex/knowledge/backend/architecture.md`
2. `.codex/skills/developer/development.md`
3. `.codex/knowledge/backend/*.md`
4. `.codex/knowledge/backend/naming.md`
5. `.codex/knowledge/backend/api-response.md`

## Core Principles

- Controllers must stay thin.
- Complex business logic belongs in Services.
- Repositories are only for simple database access.
- Query classes are for complex database reads.
- UnitOfWorkService handles simple CRUD persistence.
- Request validation and hydration are handled by Middleware.
- API responses use standard response classes.
- Tests must cover the main happy paths and failure paths.
