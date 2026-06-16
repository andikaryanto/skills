# Repository Layer

Repositories are thin wrappers around `LaravelCommon\App\Repositories\Repository`.

Each Repository passes its model class to the parent constructor.

## Example

```php
final class ProductRepository extends Repository
{
    public function __construct()
    {
        parent::__construct(Product::class);
    }
}
```

Repositories are not used for complex queries with many filters, joins, aggregations, or custom pagination.

Use Query classes for complex database reads.
