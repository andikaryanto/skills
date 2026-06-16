# Laravel Development Workflow

This document describes the Laravel feature development workflow based on the layer rules in `architecture.md`.

See also:

- [Architecture](architecture.md)
- [Naming Conventions](naming.md)
- [API Response Standard](api-response.md)
- [Testing Standard](testing.md)
- [CRUD Example](examples/crud.md)

## Goal

- Controllers stay thin and only manage HTTP flow.
- Complex business logic belongs in Services.
- Simple database access belongs in Repositories.
- Complex queries belong in Query classes.
- Simple CRUD persistence belongs in UnitOfWorkService.
- Request/entity validation and hydration are handled by Middleware.
- All responses use response classes from `LaravelCommon\Responses`.

## Feature Workflow

### 1. Define the Feature Contract

Before writing the implementation, define the feature contract:

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

Use validator and hydrator middleware as needed:

```php
Route::post('/products', [ProductController::class, 'store'])
    ->middleware([
        ProductRequestValidatorMiddleware::class . ':post',
        ProductHydratorMiddleware::class,
    ]);
```

Route parameters that need to be resolved into entities must also go through the hydrator:

```php
Route::patch('/products/{product}', [ProductController::class, 'update'])
    ->middleware([
        ProductRequestValidatorMiddleware::class . ':patch',
        ProductHydratorMiddleware::class,
    ]);
```

### 3. Create or Update Middleware

#### Validator Middleware

Use `Http/Middleware/RequestValidator/{Domain}RequestValidatorMiddleware` and extend `RequestValidatorMiddleware`.

The validator is responsible for:

- Validating the request body.
- Validating query parameters.
- Rejecting invalid requests before they reach the controller.
- Defining validation rules in methods named after actions, such as `post()` and `patch()`.
- Being called from routes with the method name as the middleware parameter.

#### Hydrator Middleware

Use `Http/Middlewares/{Domain}HydratorMiddleware` and extend `HydratorMiddleware`.

The hydrator is responsible for:

- Filling the model from the request body.
- Resolving the entity from route parameters when the route has `{domain}`.
- Storing the hydrated result in request attributes.

### 4. Implement Controller

Controllers may only:

- Read validated or hydrated input from the request.
- Call Queries and UnitOfWorkService for simple CRUD.
- Call Services for complex business logic.
- Return the appropriate response class.

Response rule:

- `GET` collection: use `LaravelCommon\Responses\PagedJsonResponse`.
- `GET`, `PATCH`, `DELETE` single entity: use `LaravelCommon\Responses\SuccessResponse`.
- `POST` single entity: use `LaravelCommon\Responses\ResourceCreatedResponse`.

Controller structure example:

```php
final class ProductController
{
    public function __construct(
        private readonly ProductQuery $query,
        private readonly UnitOfWorkService $unitOfWork,
    ) {
    }

    public function index(Request $request): PagedJsonResponse
    {
        $filters = $request->query();

        if (isset($filters['status'])) {
            $this->query->whereStatus($filters['status']);
        }

        return new PagedJsonResponse(
            $this->query->paginate()
        );
    }

    public function store(Request $request): ResourceCreatedResponse
    {
        return new ResourceCreatedResponse(
            $this->unitOfWork->save($request->attributes->get('product'))
        );
    }
}
```

### 5. Use UnitOfWorkService for Simple CRUD Persistence

Simple CRUD create, update, and delete operations are persisted through UnitOfWorkService.

Do not create a domain Service only to wrap simple CRUD persistence.

### 6. Implement Service Only for Complex Logic

Services contain complex business logic and orchestration.

Services may:

- Call Repositories.
- Call Query classes.
- Call UnitOfWorkService.
- Run database transactions.
- Enforce business rules.
- Transform entities into ViewModels when needed.

Services must not:

- Read raw HTTP requests directly.
- Return HTTP responses.
- Contain request validation that belongs in Middleware.

Example:

```php
final class ProductApprovalService
{
    public function __construct(
        private readonly UnitOfWorkService $unitOfWork,
    ) {
    }

    public function approve(Product $product): Product
    {
        $product->approve();

        return $this->unitOfWork->save($product);
    }
}
```

### 7. Implement Repository

Repositories are thin wrappers around `LaravelCommon\App\Repositories\Repository`.

Each Repository passes its model class to the parent constructor:

```php
final class ProductRepository extends Repository
{
    public function __construct()
    {
        parent::__construct(Product::class);
    }
}
```

Repositories are not used for complex queries with many filters, joins, aggregations, or custom pagination. Use Query classes for those cases.

### 8. Implement Query Class

Query classes are used for complex database access:

- Listing with filters and pagination.
- Joins across tables.
- Search.
- Sorting.
- Aggregation.
- Report query.

Query classes must:

- Extend `LaravelCommon\App\Queries\Query`.
- Define `identityClass()` and return the model class.
- Expose chainable domain filter methods.
- Return data ready for Controllers or Services to use, not HTTP responses.

### 9. Implement ViewModel When Needed

Use `ViewModels` when the response needs a shape that differs from the Model.

ViewModels are suitable for:

- Hiding internal fields.
- Combining multiple fields.
- Normalizing response formats.
- Preparing data for API consumers.

### 10. Implement Model Getters and Setters

Models should expose explicit getter and setter methods for fields that are used by hydrators, services, queries, or tests.

Use getters and setters instead of direct attribute access in domain code when possible. This keeps models easier to mock in tests.

Setter methods should return the model instance.

### 11. Testing Workflow

Minimum tests for a new feature:

- Feature test for the endpoint happy path.
- Feature test for validation errors.
- Feature test for not found or unauthorized cases when relevant.
- Unit test for Services when business rules are complex enough.
- Query class test when filters, sorting, or pagination are complex.

Checklist test per method:

- `GET collection`: data is paginated, filters work, response shape is correct.
- `GET single`: entity is found, not found is handled.
- `POST`: validation runs, entity is created, response uses created response.
- `PATCH`: validation runs, entity is updated, response is successful.
- `DELETE`: entity is deleted or marked as deleted, response is successful.

### 12. Review Checklist

Before merging, ensure:

- Controllers do not contain business logic.
- Services do not read `Request` directly.
- Services are not created only to wrap simple CRUD.
- Repositories only contain simple database access.
- Complex queries are not placed in Repositories.
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
5. UnitOfWork persistence
6. Controller
7. Model getters and setters
8. ViewModel
9. Tests

This order keeps the HTTP contract clear from the start, then moves implementation from input, to persistence or domain logic, to response.
