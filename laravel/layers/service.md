# Service Layer

Services are used only for complex business logic and domain orchestration.

Do not create a domain Service only to wrap simple CRUD persistence.

## Responsibilities

Services may:

- Execute business rules.
- Call Repositories.
- Call Query classes.
- Call UnitOfWorkService.
- Run database transactions.
- Transform entities into ViewModels when needed.

Services must not:

- Read raw HTTP requests directly.
- Return HTTP responses.
- Contain request validation that belongs in Middleware.

## Example

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
