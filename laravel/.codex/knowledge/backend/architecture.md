# Laravel Architecture

## Architecture Rules

Always follow the Laravel workflow documentation and layer guides.

Important rules:

- Simple CRUD uses Query + UnitOfWorkService.
- Services are only for complex business logic.
- Validators use RequestValidatorMiddleware with action methods such as `:post` and `:patch`.
- Hydrators extend HydratorMiddleware.
- Models use explicit getters and setters.
- Query classes handle complex reads, filtering, sorting, and pagination.
- Repositories are only for simple database access.
- API responses use LaravelCommon\Responses classes.
- Service tests use Specify and Prophecy.

## Multi-Tenancy

- The project uses `spatie/laravel-multitenancy`.
- All tenants share a single database.
- Tenant-owned records must include and be scoped by `tenant_id`.
- Query classes extend `App\Queries\BaseQuery`; tenant-owned Queries also use
  `App\Queries\Concerns\TenantScoped`.
- Queries, Services, background jobs, and console commands must preserve tenant isolation.
- Code must not access data across tenants unless the requirement explicitly defines an authorized cross-tenant operation.

## Purpose

This document defines the structure and responsibility boundaries for Laravel application layers.

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
Controller -> Query
Controller -> UnitOfWorkService
Controller -> ViewModel
Middleware -> Model / Entity
```

Dependency rules:

- Controllers may call Services, Queries, UnitOfWorkService, and ViewModels.
- Controllers must not access the database directly.
- Controllers may use Queries and UnitOfWorkService directly for simple CRUD.
- Services are used only for complex business logic.
- Services may call Repositories, Queries, and UnitOfWorkService.
- Services must not depend on `Request` or HTTP responses.
- Repositories and Queries must not return HTTP responses.
- Models must not depend on Controllers, Middleware, Services, Repositories, or Queries.

## Layer Responsibilities

### Controllers

Controllers are responsible for HTTP orchestration.

Controllers may:

- Read validated or hydrated data from the request.
- Call Queries and UnitOfWorkService for simple CRUD.
- Call Services for complex business logic.
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

Services contain complex business logic and domain orchestration.

Services are not used for simple CRUD. Simple CRUD reads are handled by Query classes, and simple CRUD persistence is handled by UnitOfWorkService.

Services may:

- Execute business rules.
- Call Repositories for simple data operations.
- Call Queries for complex data reads.
- Call UnitOfWorkService for persistence.
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

Query classes used for complex data reads extend `App\Queries\BaseQuery`.
Tenant-owned Queries must also use `App\Queries\Concerns\TenantScoped`; global
Queries omit the trait and are not filtered by `tenant_id`.

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

Tenant-owned models represent database data and relationships and must extend
`App\Models\BaseModel`. Global models may use their appropriate framework or
package base model.

Models may contain:

- Relationships.
- Casts.
- Simple scopes.
- Getter and setter methods for model fields.
- Relation wrapper objects such as `BelongsToRelation` when the model needs testable relation access.

Models must not contain complex business process orchestration.

Getter and setter methods are preferred over direct attribute access in domain code because they make models easier to mock in tests.

Setter methods should return `$this` for chaining:

```php
public function setName(string $name): Product
{
    $this->name = $name;

    return $this;
}
```

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

- `GET` collection uses `LaravelCommon\Responses\PagedJsonResponse`.
- `GET` single entity uses `LaravelCommon\Responses\SuccessResponse`.
- `POST` single entity uses `LaravelCommon\Responses\ResourceCreatedResponse`.
- `PATCH` single entity uses `LaravelCommon\Responses\SuccessResponse`.
- `DELETE` single entity uses `LaravelCommon\Responses\SuccessResponse`.

## Naming Conventions

Use the domain name as the class prefix.

Example for the `Product` domain:

```text
ProductController
ProductApprovalService
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
- Service is created only to wrap simple CRUD persistence.
- Repository contains complex listing queries.
- Query class changes data.
- Request validation is spread across Controllers or Services.
- Responses are manually created with `response()->json()` for standard API endpoints.
