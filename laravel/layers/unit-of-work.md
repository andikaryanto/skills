# UnitOfWork Persistence

UnitOfWorkService handles simple CRUD persistence.

Use UnitOfWorkService for:

- Creating an entity with `persist($model)`.
- Updating an entity with `persist($model)`.
- Deleting an entity with `remove($model)`.
- Committing changes with `flush()`.

Do not create a domain Service only to wrap simple CRUD persistence.

## Rules

- `persist($model)` always receives a model instance.
- `persist($model)` starts a transaction when needed and schedules create or update persistence.
- `remove($model)` always receives a model instance and schedules removal.
- `flush()` commits the scheduled persistence operations.
- The order is always `persist()` or `remove()`, then `flush()`.

## Example

```php
public function store(Request $request): ResourceCreatedResponse
{
    $product = $request->attributes->get('product');
    $this->unitOfWork->persist($product);
    $this->unitOfWork->flush();

    return new ResourceCreatedResponse(
        $product
    );
}

public function update(Request $request): SuccessResponse
{
    $product = $request->attributes->get('product');
    $this->unitOfWork->persist($product);
    $this->unitOfWork->flush();

    return new SuccessResponse(
        $product
    );
}

public function destroy(Request $request): SuccessResponse
{
    $product = $request->attributes->get('product');
    $this->unitOfWork->remove($product);
    $this->unitOfWork->flush();

    return new SuccessResponse($product);
}
```
