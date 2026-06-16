# Laravel Development Workflow

This document describes the end-to-end Laravel feature development workflow.
Layer-specific implementation details live in `layers/`.

See also:

- [Architecture](architecture.md)
- [Controller Layer](layers/controller.md)
- [Middleware Layer](layers/middleware.md)
- [Model Layer](layers/model.md)
- [Repository Layer](layers/repository.md)
- [Query Layer](layers/query.md)
- [Service Layer](layers/service.md)
- [UnitOfWork Persistence](layers/unit-of-work.md)
- [ViewModel Layer](layers/view-model.md)
- [Naming Conventions](naming.md)
- [API Response Standard](api-response.md)
- [Testing Standard](testing.md)
- [CRUD Example](examples/crud.md)

## Goal

- Controllers stay thin and only manage HTTP flow.
- Complex business logic belongs in Services.
- Simple database access belongs in Repositories.
- Complex database reads belong in Query classes.
- Simple CRUD persistence belongs in UnitOfWorkService.
- Request validation and hydration are handled by Middleware.
- Models expose explicit getters and setters for testability.
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

Short contract example:

```text
POST /api/products
Body: name, sku, price
Response: ResourceCreatedResponse
```

### 2. Add or Update Route

Add the route in the appropriate route file.

Use validator and hydrator middleware as needed.
See [Middleware Layer](layers/middleware.md).

### 3. Add Validator Middleware

Use action-based request validation methods such as `post()` and `patch()`.

Route usage example:

```php
ProductRequestValidatorMiddleware::class . ':post'
```

See [Middleware Layer](layers/middleware.md).

### 4. Add Hydrator Middleware

Hydrate request body and route parameters into the request attributes.

See [Middleware Layer](layers/middleware.md).

### 5. Add Repository or Query

Use a Repository for simple model persistence infrastructure.
Use a Query class for complex reads, filters, and pagination.

See:

- [Repository Layer](layers/repository.md)
- [Query Layer](layers/query.md)

### 6. Add Model Getters and Setters

Expose explicit getter and setter methods for fields used by hydrators, queries, services, or tests.

See [Model Layer](layers/model.md).

### 7. Implement Controller

Controllers orchestrate HTTP flow and select the response class.

For simple CRUD:

- Read with Query classes.
- Persist with UnitOfWorkService.

For complex business logic:

- Call a domain Service.

See:

- [Controller Layer](layers/controller.md)
- [UnitOfWork Persistence](layers/unit-of-work.md)
- [Service Layer](layers/service.md)

### 8. Add Service Only for Complex Logic

Do not create a domain Service only to wrap simple CRUD.

Create a Service when the feature has business rules, orchestration, transactions, calculations, or state transitions.

See [Service Layer](layers/service.md).

### 9. Add ViewModel When Needed

Use a ViewModel only when the response shape differs from the Model.

See [ViewModel Layer](layers/view-model.md).

### 10. Add Tests

Minimum tests for a new feature:

- Feature test for the endpoint happy path.
- Feature test for validation errors.
- Feature test for not found or unauthorized cases when relevant.
- Service test for complex business rules.
- Query test for complex filters, sorting, or pagination.

See [Testing Standard](testing.md).

### 11. Review Checklist

Before merging, ensure:

- Controllers do not contain business logic.
- Services are not created only to wrap simple CRUD.
- Services do not read `Request` directly.
- Repositories only contain simple database access.
- Complex reads are placed in Query classes.
- Validator and hydrator middleware are used for request/route entities.
- Models expose getter and setter methods for fields used by hydrators or domain logic.
- Response classes follow method rules.
- Class naming is consistent with the domain.
- Error handling is explicit for important cases.
- Tests cover the main happy paths and failure paths.

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
