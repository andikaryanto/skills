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
- Business logic belongs in Services.
- Simple database access belongs in Repositories.
- Complex queries belong in Query classes.
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
- Call Services.
- Return the appropriate response class.

Response rule:

- `GET` collection: use `LaravelCommon\Responses\PagedJsonResponse` and the Service function `getAll`.
- `GET`, `PATCH`, `DELETE` single entity: use `LaravelCommon\Responses\SuccessResponse` and the Service function `get`.
- `POST` single entity: use `LaravelCommon\Responses\ResourceCreatedResponse`.

Controller structure example:

```php
final class ProductController
{
    public function __construct(
        private readonly ProductService $service,
    ) {
    }

    public function index(Request $request): PagedJsonResponse
    {
        return new PagedJsonResponse(
            $this->service->getAll($request->query())
        );
    }

    public function store(Request $request): ResourceCreatedResponse
    {
        return new ResourceCreatedResponse(
            $this->service->create($request->attributes->get('product'))
        );
    }
}
```

### 5. Implement Service

Services contain business logic and orchestration.

Services may:

- Call Repositories.
- Call Query classes.
- Run database transactions.
- Enforce business rules.
- Transform entities into ViewModels when needed.

Services must not:

- Read raw HTTP requests directly.
- Return HTTP responses.
- Contain request validation that belongs in Middleware.

Example:

```php
final class ProductService
{
    public function __construct(
        private readonly ProductRepository $repository,
        private readonly ProductQuery $query,
    ) {
    }

    public function getAll(array $filters): LengthAwarePaginator
    {
        if (isset($filters['status'])) {
            $this->query->whereStatus($filters['status']);
        }

        return $this->query->paginate();
    }

    public function create(Product $product): Product
    {
        return $this->repository->save($product);
    }
}
```

### 6. Implement Repository

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

### 7. Implement Query Class

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
- Return data ready for Services to use, not HTTP responses.

### 8. Implement ViewModel When Needed

Use `ViewModels` when the response needs a shape that differs from the Model.

ViewModels are suitable for:

- Hiding internal fields.
- Combining multiple fields.
- Normalizing response formats.
- Preparing data for API consumers.

### 9. Testing Workflow

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

### 10. Review Checklist

Before merging, ensure:

- Controllers do not contain business logic.
- Services do not read `Request` directly.
- Repositories only contain simple database access.
- Complex queries are not placed in Repositories.
- Validator and hydrator middleware are used for request/route entities.
- Response classes follow method rules.
- Class naming is consistent with the domain.
- Error handling is explicit for important cases.
- Tests cover the main happy paths and failure paths.

## Recommended Implementation Order

1. Route
2. Validator Middleware
3. Hydrator Middleware
4. Repository or Query
5. Service
6. Controller
7. ViewModel
8. Tests

This order keeps the HTTP contract clear from the start, then moves implementation from input, to domain logic, to response.
