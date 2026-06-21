# Laravel Development Workflow

This document describes the end-to-end Laravel feature development workflow.

Before implementation, read
`.codex/knowledge/backend/architecture.md` and apply its multi-tenancy rules.

## Goal

- Controllers stay thin and only manage HTTP flow.
- Complex business logic belongs in Services.
- Simple database access belongs in Repositories.   
- Complex database reads belong in Query classes.
- Simple CRUD persistence belongs in UnitOfWorkService.
- Request validation and hydration are handled by Middleware.
- Models expose explicit getters and setters for testability.
- Tenant-owned models extend `App\Models\BaseModel`.
- API responses use response classes from `LaravelCommon\Responses`.

## Feature Workflow

### 1. Define the Feature Contract

Before writing implementation, define:

- Route and HTTP method.
- Request payload, query parameters, and route parameters.
- Success response.
- Possible error responses.
- Permission or authentication requirement.
- Main entity being processed.
- Whether the entity is tenant-owned.

For a tenant-owned entity:

- Add a required indexed `tenant_id` foreign key in its migration.
- Scope all reads and writes to the current tenant.
- Make uniqueness validation tenant-scoped unless the value is intentionally
  global.
- Never accept the effective `tenant_id` from untrusted request input.
- Add tests proving one tenant cannot read, update, delete, or conflict with
  another tenant's records.

Short contract example:

```text
POST /api/products
Body: name, sku, price
Response: ResourceCreatedResponse
```

### 2. Add or Update Route

Add the route in the appropriate route file.

Use validator and hydrator middleware as needed.
See [Middleware Layer](.codex/knowledge/middleware.md).

### 3. Add Validator Middleware

Use action-based request validation methods such as `post()` and `patch()`.

Route usage example:

```php
ProductRequestValidatorMiddleware::class . ':post'
```

See [Middleware Layer](.codex/knowledge/middleware.md).

### 4. Add Hydrator Middleware

Hydrate request body and route parameters into the request attributes.

See [Middleware Layer](.codex/knowledge/middleware.md).

### 5. Add Repository or Query

Use a Repository for simple model persistence infrastructure.
Use a Query class for complex reads, filters, and pagination.

Query classes extend `App\Queries\BaseQuery`. Add
`App\Queries\Concerns\TenantScoped` only when the query's table contains
`tenant_id`. Tenant-owned Repository operations remain protected by the model
tenant scope. Route-model hydration must not resolve another tenant's records.

See:

- [Repository Layer](.codex/knowledge/repository.md)
- [Query Layer](.codex/knowledge/query.md)

### 6. Add Model Getters and Setters

Expose explicit getter and setter methods for fields used by hydrators, queries, services, or tests.

See [Model Layer](.codex/knowledge/model.md).

### 7. Implement Controller

Controllers orchestrate HTTP flow and select the response class.

For simple CRUD:

- Read with Query classes.
- Persist with UnitOfWorkService using `persist()` or `remove()`, then `flush()`.

For complex business logic:

- Call a domain Service.

See:

- [Controller Layer](.codex/knowledge/controller.md)
- [UnitOfWork Persistence](.codex/knowledge/unit-of-work.md)
- [Service Layer](.codex/knowledge/service.md)

### 8. Add Service Only for Complex Logic

Do not create a domain Service only to wrap simple CRUD.

Create a Service when the feature has business rules, orchestration, transactions, calculations, or state transitions.

See [Service Layer](.codex/knowledge/service.md).

### 9. Add ViewModel When Needed

Use a ViewModel only when the response shape differs from the Model.

See [ViewModel Layer](.codex/knowledge/view-model.md).

### 10. Add Tests

Minimum tests for a new feature:

- Feature test for the endpoint happy path.
- Feature test for validation errors.
- Feature test for not found or unauthorized cases when relevant.
- Dedicated focused test for each changed controller and middleware.
- Dedicated focused test for each changed application class unless the testing
  standard explicitly permits an exclusion.
- Service test for complex business rules.
- Query test for every custom filter, plus sorting or pagination when present.

See [Testing Standard](.codex/skills/developer/testing.md).

## Recommended Implementation Order

1. Route
2. Validator Middleware
3. Hydrator Middleware
4. Repository or Query
5. Model getters and setters
6. Controller
7. UnitOfWork persistence
8. Service, only for complex logic
9. ViewModel, when needed
10. Tests
