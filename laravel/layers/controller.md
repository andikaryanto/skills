# Controller Layer

Controllers are responsible for HTTP orchestration.

## Responsibilities

Controllers may:

- Read validated or hydrated input from the request.
- Call Queries and UnitOfWorkService for simple CRUD.
- Call Services for complex business logic.
- Return the appropriate response class.

Controllers must not:

- Contain business logic.
- Run database queries directly through Models.
- Perform manual validation that belongs in Middleware.
- Manually transform request bodies into entities.
- Build custom JSON structures for standard API endpoints.

## Response Rules

- `GET` collection: use `LaravelCommon\Responses\PagedJsonResponse`.
- `GET`, `PATCH`, `DELETE` single entity: use `LaravelCommon\Responses\SuccessResponse`.
- `POST` single entity: use `LaravelCommon\Responses\ResourceCreatedResponse`.

## Simple CRUD Example

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

    public function show(Request $request): SuccessResponse
    {
        return new SuccessResponse(
            $request->attributes->get('product')
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
