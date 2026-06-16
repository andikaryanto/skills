# Laravel Architecture

See also:

- [Development Workflow](development-workflow.md)
- [Naming Conventions](naming.md)
- [API Response Standard](api-response.md)
- [Testing Standard](testing.md)
- [CRUD Example](examples/crud.md)

## Purpose

This document defines the structure and responsibility boundaries for Laravel application layers.
For the feature implementation flow, use `development-workflow.md`.

## Application Structure

```text
app
    Console
    Enums
    Http
        Controllers
        Middlewares
    Models
    Queries
    Repositories
    Services
    ViewModels
```

## Dependency Direction

Dependencies must flow from the HTTP layer toward the domain and persistence layers.

```text
Controller -> Service -> Repository
Controller -> Service -> Query
Controller -> ViewModel
Middleware -> Model / Entity
```

Dependency rules:

- Controllers may call Services and ViewModels.
- Controllers must not access the database directly.
- Services may call Repositories and Queries.
- Services must not depend on `Request` or HTTP responses.
- Repositories and Queries must not return HTTP responses.
- Models must not depend on Controllers, Middleware, Services, Repositories, or Queries.

## Layer Responsibilities

### Controllers

Controllers are responsible for HTTP orchestration.

Controllers may:

- Read validated or hydrated data from the request.
- Call Services.
- Build responses using standard response classes.

Controllers must not:

- Contain business logic.
- Run database queries directly.
- Perform manual validation that belongs in Middleware.
- Manually transform request bodies into entities.

### Middlewares

Middleware is used for validation and hydration before the request reaches the Controller.

Validator:

- Located at `Http/Middleware/RequestValidator/{Domain}RequestValidatorMiddleware`.
- Extend `RequestValidatorMiddleware`.
- Responsible for validating body payloads, query parameters, and other request input.
- Exposes validation methods by action name, such as `post()` and `patch()`.
- Used in routes with middleware parameters, such as `{Domain}RequestValidatorMiddleware::class . ':post'`.

Hydrator:

- Located at `Http/Middlewares/{Domain}HydratorMiddleware`.
- Extend `HydratorMiddleware`.
- The constructor calls `parent::__construct('{domain}', $repository)`.
- Responsible for filling the model from the request body.
- Responsible for resolving the entity from route parameters when needed.

### Services

Services contain business logic and domain orchestration.

Services may:

- Execute business rules.
- Call Repositories for simple data operations.
- Call Queries for complex data reads.
- Manage database transactions.
- Return Models, entities, collections, paginators, or ViewModels.

Services must not:

- Read `Request` directly.
- Return HTTP responses.
- Contain request validation.

### Repositories

Repositories are used for simple database operations against one model or aggregate and must extend `LaravelCommon\App\Repositories\Repository`.

Each Repository passes its model class to the parent constructor.

Example:

```php
final class ProductRepository extends Repository
{
    public function __construct()
    {
        parent::__construct(Product::class);
    }
}
```

Repositories are not used for complex listings, dynamic filters, joins, aggregations, or report queries.

### Queries

Query classes are used for complex data reads and must extend `LaravelCommon\App\Queries\Query`.

Every Query class must define `identityClass()` and return the model class it represents.

Example Query responsibilities:

- Listing with filters.
- Pagination.
- Sorting.
- Search.
- Join.
- Aggregation.
- Report query.

Query classes expose chainable domain filter methods, such as `whereClinicCategory(...)`.
Query classes must return data that is ready for Services to process, not HTTP responses.

### Models

Models represent database data and relationships.

Models may contain:

- Relationships.
- Casts.
- Simple scopes.
- Accessors or mutators that are truly part of the data model.

Models must not contain complex business process orchestration.

### ViewModels

ViewModels are used when the response shape differs from the Model structure.

ViewModels are suitable for:

- Hiding internal fields.
- Combining multiple fields for API responses.
- Normalizing output formats.
- Preparing stable response shapes for clients.

## Response Rules

All API responses must use classes from:

```text
vendor/andikaryanto/laravelcommon/src/Responses
```

Response rules:

- `GET` collection uses `LaravelCommon\Responses\PagedJsonResponse` and the Service function `getAll`.
- `GET` single entity uses `LaravelCommon\Responses\SuccessResponse` and the Service function `get`.
- `POST` single entity uses `LaravelCommon\Responses\ResourceCreatedResponse`.
- `PATCH` single entity uses `LaravelCommon\Responses\SuccessResponse`.
- `DELETE` single entity uses `LaravelCommon\Responses\SuccessResponse`.

## Naming Conventions

Use the domain name as the class prefix.

Example for the `Product` domain:

```text
ProductController
ProductService
ProductRepository
ProductQuery
ProductHydratorMiddleware
ProductRequestValidatorMiddleware
ProductViewModel
```

## Anti-Patterns

Avoid these patterns:

- Controller contains business logic.
- Controller calls Model queries directly.
- Service accepts a `Request` object.
- Service returns `JsonResponse` or another HTTP response.
- Repository contains complex listing queries.
- Query class changes data.
- Request validation is spread across Controllers or Services.
- Responses are manually created with `response()->json()` for standard API endpoints.
