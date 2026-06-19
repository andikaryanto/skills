# Middleware Layer

Middleware is used for validation and hydration before the request reaches the Controller.

## Validator Middleware

Use `Http/Middleware/RequestValidator/{Domain}RequestValidatorMiddleware` and extend `RequestValidatorMiddleware`.

The validator is responsible for:

- Validating the request body.
- Validating query parameters.
- Rejecting invalid requests before they reach the controller.
- Defining validation rules in methods named after actions, such as `post()` and `patch()`.
- Being called from routes with the method name as the middleware parameter.

Route usage:

```php
Route::post('/products', [ProductController::class, 'store'])
    ->middleware([
        ProductRequestValidatorMiddleware::class . ':post',
        ProductHydratorMiddleware::class,
    ]);
```

Validator example:

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

Use `Http/Middlewares/{Domain}HydratorMiddleware` and extend `HydratorMiddleware`.

The hydrator is responsible for:

- Filling the model from the request body.
- Resolving the entity from route parameters when the route has `{domain}`.
- Storing the hydrated result in request attributes.

Hydrator example:

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
