# Laravel Workflow

This documentation defines reusable Laravel architecture and development workflow standards.

## Documents

- [Architecture](architecture.md): layer structure, dependency direction, class responsibilities, and anti-patterns.
- [Development Workflow](development-workflow.md): feature implementation flow from route to tests.
- [Naming](naming.md): naming conventions for classes, methods, routes, and request attributes.
- [API Response](api-response.md): API response standards and `LaravelCommon\Responses` usage.
- [Testing](testing.md): testing standards for endpoints, services, repositories, and queries.
- [CRUD Example](examples/crud.md): example implementation for a simple CRUD domain.

## Layer Guides

- [Controller](layers/controller.md)
- [Middleware](layers/middleware.md)
- [Model](layers/model.md)
- [Repository](layers/repository.md)
- [Query](layers/query.md)
- [Service](layers/service.md)
- [UnitOfWork Persistence](layers/unit-of-work.md)
- [ViewModel](layers/view-model.md)

## Recommended Reading Order

1. `architecture.md`
2. `development-workflow.md`
3. `layers/*.md`
4. `naming.md`
5. `api-response.md`
6. `testing.md`
7. `examples/crud.md`

## Core Principles

- Controllers must stay thin.
- Complex business logic belongs in Services.
- Repositories are only for simple database access.
- Query classes are for complex database reads.
- UnitOfWorkService handles simple CRUD persistence.
- Request validation and hydration are handled by Middleware.
- API responses use standard response classes.
- Tests must cover the main happy paths and failure paths.
