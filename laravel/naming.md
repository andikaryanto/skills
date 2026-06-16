# Laravel Naming Conventions

Dokumen ini mendefinisikan standar penamaan class, method, route, dan request attribute.

## Domain Prefix

Gunakan nama domain sebagai prefix class.

Contoh domain `Product`:

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

Controller menggunakan nama singular domain.

```text
ProductController
CustomerController
OrderController
```

Method controller mengikuti aksi HTTP:

```text
index   -> GET collection
show    -> GET single entity
store   -> POST
update  -> PATCH
destroy -> DELETE
```

## Services

Service menggunakan nama singular domain.

```text
ProductService
```

Method service standar:

```text
getAll(array $filters)
get(int|string $id)
create(Entity|Model $entity)
update(Entity|Model $entity)
delete(Entity|Model $entity)
```

Gunakan nama method berbasis business action ketika logic tidak cocok dengan CRUD.

```text
approve(Order $order)
cancel(Order $order)
publish(Article $article)
```

## Repositories

Repository digunakan untuk operasi data sederhana.

```text
ProductRepository
```

Method repository standar:

```text
find(int|string $id)
findOrFail(int|string $id)
save(Model $model)
delete(Model $model)
findBySku(string $sku)
```

## Queries

Query class digunakan untuk pembacaan data kompleks.

```text
ProductQuery
```

Method query yang umum:

```text
paginate(array $filters)
search(array $filters)
findWithRelations(int|string $id)
summary(array $filters)
```

## Middlewares

Validator middleware:

```text
ProductRequestValidatorMiddleware
```

Hydrator middleware:

```text
ProductHydratorMiddleware
```

Hydrator middleware menggunakan constructor repository dan nama domain.

## Request Attributes

Entity hasil hydrasi disimpan dengan nama singular domain dalam format camel case.

```text
product
customer
order
```

Jika ada lebih dari satu entity dalam request yang sama, gunakan nama yang eksplisit.

```text
sourceAccount
targetAccount
```

## Routes

Gunakan plural kebab-case untuk resource route.

```text
/products
/customers
/purchase-orders
```

Gunakan nama action hanya untuk operasi non-CRUD.

```text
POST /orders/{order}/approve
POST /articles/{article}/publish
```
