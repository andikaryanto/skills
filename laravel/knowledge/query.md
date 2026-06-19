# Query Layer

Query classes are used for complex database reads and must extend `LaravelCommon\App\Queries\Query`.

## Rules

Query classes must:

- Define `identityClass()` and return the model class.
- Expose chainable domain filter methods.
- Return data ready for Controllers or Services to use.
- Never return HTTP responses.

## Example

```php
final class ProductQuery extends Query
{
    public function identityClass(): string
    {
        return Product::class;
    }

    public function whereStatus(string $status): ProductQuery
    {
        $this->where('products.status', '=', $status);

        return $this;
    }

    public function whereCategory(Category $category): ProductQuery
    {
        $this->where('products.category_id', '=', $category->getId());

        return $this;
    }
}
```
