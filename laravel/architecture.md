# Laravel Architecture

See also:

- [Development Workflow](development-workflow.md)
- [Naming Conventions](naming.md)
- [API Response Standard](api-response.md)
- [Testing Standard](testing.md)
- [CRUD Example](examples/crud.md)

## Purpose

Dokumen ini mendefinisikan struktur dan batas tanggung jawab layer aplikasi Laravel.
Untuk urutan implementasi fitur, gunakan `development-workflow.md`.

## Application Structure

```text
app
    Console
    Enums
    Http
        Controllers
        Middlewares
    Models
    Queries
    Repositories
    Services
    ViewModels
```

## Dependency Direction

Dependency harus mengalir dari layer HTTP menuju domain dan persistence.

```text
Controller -> Service -> Repository
Controller -> Service -> Query
Controller -> ViewModel
Middleware -> Model / Entity
```

Aturan dependency:

- Controller boleh memanggil Service dan ViewModel.
- Controller tidak boleh mengakses database langsung.
- Service boleh memanggil Repository dan Query.
- Service tidak boleh bergantung pada `Request` atau response HTTP.
- Repository dan Query tidak boleh mengembalikan response HTTP.
- Model tidak boleh bergantung pada Controller, Middleware, Service, Repository, atau Query.

## Layer Responsibilities

### Controllers

Controller bertanggung jawab untuk HTTP orchestration.

Controller boleh:

- Mengambil data yang sudah divalidasi atau dihidrasi dari request.
- Memanggil Service.
- Membentuk response menggunakan response class standar.

Controller tidak boleh:

- Menyimpan business logic.
- Melakukan query database langsung.
- Melakukan validasi manual yang seharusnya ada di Middleware.
- Mengubah request body menjadi entity secara manual.

### Middlewares

Middleware digunakan untuk validasi dan hydrasi sebelum request masuk ke Controller.

Validator:

- Berada di `Http/Middlewares/{Domain}RequestValidatorMiddleware`.
- Extend `ValidatorMiddleware`.
- Bertanggung jawab untuk validasi body, query parameter, dan input request lain.

Hydrator:

- Berada di `Http/Middlewares/{Domain}HydratorMiddleware`.
- Extend `HydratorMiddleware`.
- Constructor memanggil `parent::__construct('{domain}', $repository)`.
- Bertanggung jawab untuk mengisi model dari body request.
- Bertanggung jawab untuk mengambil entity dari route parameter ketika diperlukan.

### Services

Service adalah tempat business logic dan orchestration domain.

Service boleh:

- Menjalankan business rule.
- Memanggil Repository untuk operasi data sederhana.
- Memanggil Query untuk pembacaan data kompleks.
- Mengatur transaksi database.
- Menghasilkan Model, entity, collection, paginator, atau ViewModel.

Service tidak boleh:

- Membaca `Request` secara langsung.
- Mengembalikan response HTTP.
- Menyimpan validasi request.

### Repositories

Repository digunakan untuk operasi database sederhana terhadap satu model atau aggregate.

Contoh tanggung jawab Repository:

- `find`
- `save`
- `update`
- `delete`
- lookup sederhana berdasarkan field unik

Repository tidak digunakan untuk listing kompleks, filter dinamis, join, aggregation, atau report query.

### Queries

Query class digunakan untuk pembacaan data kompleks.

Contoh tanggung jawab Query:

- Listing dengan filter.
- Pagination.
- Sorting.
- Search.
- Join.
- Aggregation.
- Report query.

Query class harus mengembalikan data yang siap diproses Service, bukan response HTTP.

### Models

Model merepresentasikan data dan relasi database.

Model boleh berisi:

- Relationship.
- Cast.
- Scope sederhana.
- Accessor atau mutator yang benar-benar melekat pada data.

Model tidak boleh berisi orchestration business process yang kompleks.

### ViewModels

ViewModel digunakan ketika bentuk response berbeda dari struktur Model.

ViewModel cocok untuk:

- Menyembunyikan field internal.
- Menggabungkan beberapa field untuk API response.
- Menormalisasi format output.
- Menyiapkan response shape yang stabil untuk client.

## Response Rules

Semua response API harus menggunakan class dari:

```text
vendor/andikaryanto/laravelcommon/src/Responses
```

Aturan response:

- `GET` collection menggunakan `LaravelCommon\Responses\PagedJsonResponse` dan Service function `getAll`.
- `GET` single entity menggunakan `LaravelCommon\Responses\SuccessResponse` dan Service function `get`.
- `POST` single entity menggunakan `LaravelCommon\Responses\ResourceCreatedResponse`.
- `PATCH` single entity menggunakan `LaravelCommon\Responses\SuccessResponse`.
- `DELETE` single entity menggunakan `LaravelCommon\Responses\SuccessResponse`.

## Naming Conventions

Gunakan nama domain sebagai prefix class.

Contoh untuk domain `Product`:

```text
ProductController
ProductService
ProductRepository
ProductQuery
ProductHydratorMiddleware
ProductRequestValidatorMiddleware
ProductViewModel
```

## Anti-Patterns

Hindari pola berikut:

- Controller berisi business logic.
- Controller memanggil Model query langsung.
- Service menerima object `Request`.
- Service mengembalikan `JsonResponse` atau response HTTP lain.
- Repository berisi query listing kompleks.
- Query class melakukan perubahan data.
- Validasi request tersebar di Controller atau Service.
- Response dibuat manual dengan `response()->json()` untuk endpoint API standar.
