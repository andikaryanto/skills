# Laravel API Response Standard

This document defines API response standards.

## Response Package

All standard API responses must use classes from:

```text
vendor/andikaryanto/laravelcommon/src/Responses
```

Do not create manual responses with `response()->json()` for standard API endpoints unless there is an approved special case.

## Response Mapping

| Method | Use Case | Response Class |
| --- | --- | --- |
| `GET` | Collection or paginated list | `LaravelCommon\Responses\PagedJsonResponse` |
| `GET` | Single entity | `LaravelCommon\Responses\SuccessResponse` |
| `POST` | Created resource | `LaravelCommon\Responses\ResourceCreatedResponse` |
| `PATCH` | Updated resource | `LaravelCommon\Responses\SuccessResponse` |
| `DELETE` | Deleted resource | `LaravelCommon\Responses\SuccessResponse` |

## Controller Rules

Controllers only select the response class and pass data from Services.

```php
public function show(Request $request): SuccessResponse
{
    return new SuccessResponse(
        $this->service->get($request->attributes->get('product'))
    );
}
```

Controllers must not:

- Manually build JSON structures.
- Perform complex data transformation for responses.
- Determine business status from direct queries.

Use ViewModels when the response shape differs from the Model.

## Pagination

Collection endpoints must use the standard pagination response.

```php
public function index(Request $request): PagedJsonResponse
{
    return new PagedJsonResponse(
        $this->service->getAll($request->query())
    );
}
```

Filtering, sorting, and pagination are processed by Services and Query classes.

## Error Response

Validation errors must be handled by `RequestValidatorMiddleware`.

Not found and business errors must be explicit in the most appropriate layer:

- Route entity not found: Hydrator or Repository.
- Business rule failure: Service.
- Empty query result for a required resource: Query or Service.

Endpoints must not hide important errors behind empty success responses.
