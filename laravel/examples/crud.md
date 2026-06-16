# CRUD Example

This example uses the `Product` domain to demonstrate the CRUD implementation pattern.

## Route

```php
Route::get('/products', [ProductController::class, 'index']);

Route::post('/products', [ProductController::class, 'store'])
    ->middleware([
        ProductRequestValidatorMiddleware::class . ':post',
        ProductHydratorMiddleware::class,
    ]);

Route::get('/products/{product}', [ProductController::class, 'show'])
    ->middleware([
        ProductHydratorMiddleware::class,
    ]);

Route::patch('/products/{product}', [ProductController::class, 'update'])
    ->middleware([
        ProductRequestValidatorMiddleware::class . ':patch',
        ProductHydratorMiddleware::class,
    ]);

Route::delete('/products/{product}', [ProductController::class, 'destroy'])
    ->middleware([
        ProductHydratorMiddleware::class,
    ]);
```

## Controller

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

    public function show(Request $request): SuccessResponse
    {
        return new SuccessResponse(
            $this->service->get($request->attributes->get('product'))
        );
    }

    public function store(Request $request): ResourceCreatedResponse
    {
        return new ResourceCreatedResponse(
            $this->service->create($request->attributes->get('product'))
        );
    }

    public function update(Request $request): SuccessResponse
    {
        return new SuccessResponse(
            $this->service->update($request->attributes->get('product'))
        );
    }

    public function destroy(Request $request): SuccessResponse
    {
        return new SuccessResponse(
            $this->service->delete($request->attributes->get('product'))
        );
    }
}
```

## Service

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

    public function get(Product $product): Product
    {
        return $product;
    }

    public function create(Product $product): Product
    {
        return $this->repository->save($product);
    }

    public function update(Product $product): Product
    {
        return $this->repository->save($product);
    }

    public function delete(Product $product): Product
    {
        $this->repository->delete($product);

        return $product;
    }
}
```

## Repository

```php
final class ProductRepository
{
    public function findOrFail(int|string $id): Product
    {
        return Product::query()->findOrFail($id);
    }

    public function save(Product $product): Product
    {
        $product->save();

        return $product;
    }

    public function delete(Product $product): void
    {
        $product->delete();
    }
}
```

## Query

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

## Validator Middleware

```php
final class ProductRequestValidatorMiddleware extends RequestValidatorMiddleware
{
    public function post()
    {
        return [
            'name' => 'required|string|max:255',
            'sku' => 'required|string|max:100',
            'price' => 'required|numeric|min:0',
            'status' => 'required|string',
        ];
    }

    public function patch()
    {
        return [
            'name' => 'required|string|max:255',
            'sku' => 'required|string|max:100',
            'price' => 'required|numeric|min:0',
            'status' => 'required|string',
        ];
    }
}
```

## Hydrator Middleware

```php
final class ProductHydratorMiddleware extends HydratorMiddleware
{
    public function __construct(ProductRepository $repository)
    {
        parent::__construct('product', $repository);
    }

    public function hydrate()
    {
        $this->when(
            'name',
            [$this->model, 'setName']
        )->when(
            'sku',
            [$this->model, 'setSku']
        )->when(
            'price',
            [$this->model, 'setPrice']
        )->when(
            'status',
            [$this->model, 'setStatus']
        );
    }
}
```

## Tests

Minimum tests:

- `test_it_lists_products`
- `test_it_creates_product`
- `test_it_rejects_invalid_product_payload`
- `test_it_shows_product`
- `test_it_updates_product`
- `test_it_deletes_product`
