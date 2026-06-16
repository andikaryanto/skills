# Laravel Naming Conventions

This document defines naming standards for classes, methods, routes, and request attributes.

## Domain Prefix

Use the domain name as the class prefix.

Example for the `Product` domain:

```text
ProductController
ProductService
ProductRepository
ProductQuery
ProductHydratorMiddleware
ProductRequestValidatorMiddleware
ProductViewModel
```

## Controllers

Controllers use the singular domain name.

```text
ProductController
CustomerController
OrderController
```

Controller methods follow HTTP actions:

```text
index   -> GET collection
show    -> GET single entity
store   -> POST
update  -> PATCH
destroy -> DELETE
```

## Services

Services use the singular domain name.

```text
ProductService
```

Standard Service methods:

```text
getAll(array $filters)
get(int|string $id)
create(Entity|Model $entity)
update(Entity|Model $entity)
delete(Entity|Model $entity)
```

Use business-action method names when the logic does not fit CRUD.

```text
approve(Order $order)
cancel(Order $order)
publish(Article $article)
```

## Repositories

Repositories are thin wrappers around `LaravelCommon\App\Repositories\Repository`.

```text
ProductRepository
```

Standard Repository constructor:

```php
public function __construct()
{
    parent::__construct(Product::class);
}
```

## Queries

Query classes are used for complex data reads.

```text
ProductQuery
```

Common Query methods:

```text
identityClass()
whereStatus(string $status)
whereCategory(Category $category)
whereCreatedBetween(DateTimeInterface $from, DateTimeInterface $to)
```

Query filter methods should be chainable and return the concrete Query class.

## Middlewares

Validator middleware:

```text
ProductRequestValidatorMiddleware
```

Hydrator middleware:

```text
ProductHydratorMiddleware
```

Hydrator middleware uses a repository constructor and the domain name.

Validator middleware exposes action methods used as middleware parameters:

```text
ProductRequestValidatorMiddleware::class . ':post'
ProductRequestValidatorMiddleware::class . ':patch'
```

## Request Attributes

Hydrated entities are stored using the singular domain name in camel case.

```text
product
customer
order
```

If more than one entity exists in the same request, use explicit names.

```text
sourceAccount
targetAccount
```

## Routes

Use plural kebab-case for resource routes.

```text
/products
/customers
/purchase-orders
```

Use action names only for non-CRUD operations.

```text
POST /orders/{order}/approve
POST /articles/{article}/publish
```
