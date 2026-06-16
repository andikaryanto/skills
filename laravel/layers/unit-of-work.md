# UnitOfWork Persistence

UnitOfWorkService handles simple CRUD persistence.

Use UnitOfWorkService for:

- Creating an entity.
- Updating an entity.
- Deleting an entity.

Do not create a domain Service only to wrap simple CRUD persistence.

## Example

```php
public function store(Request $request): ResourceCreatedResponse
{
    return new ResourceCreatedResponse(
        $this->unitOfWork->save($request->attributes->get('product'))
    );
}

public function update(Request $request): SuccessResponse
{
    return new SuccessResponse(
        $this->unitOfWork->save($request->attributes->get('product'))
    );
}

public function destroy(Request $request): SuccessResponse
{
    $product = $request->attributes->get('product');
    $this->unitOfWork->delete($product);

    return new SuccessResponse($product);
}
```
