# Laravel Development Workflow

Dokumen ini menjelaskan alur kerja pengembangan fitur Laravel berdasarkan aturan layer di `architecture.md`.

## Goal

- Controller tetap tipis dan hanya mengatur HTTP flow.
- Business logic ditempatkan di Service.
- Akses database sederhana ditempatkan di Repository.
- Query kompleks ditempatkan di Query class.
- Validasi dan hydrasi request/entity dilakukan melalui Middleware.
- Semua response menggunakan response class dari `LaravelCommon\Responses`.

## Feature Workflow

### 1. Define the Feature Contract

Sebelum menulis implementasi, tentukan kontrak fitur:

- Route dan HTTP method.
- Request payload, query parameter, dan route parameter.
- Response success.
- Response error yang mungkin terjadi.
- Permission atau authentication requirement.
- Entity utama yang diproses.

Contoh kontrak singkat:

```text
POST /api/products
Body: name, sku, price
Response: ResourceCreatedResponse
```

### 2. Add or Update Route

Tambahkan route di file route yang sesuai.

Gunakan middleware validator dan hydrator sesuai kebutuhan:

```php
Route::post('/products', [ProductController::class, 'store'])
    ->middleware([
        ProductRequestValidatorMiddleware::class,
        ProductHydrator::class,
    ]);
```

Route parameter yang perlu diubah menjadi entity juga harus melalui hydrator:

```php
Route::patch('/products/{product}', [ProductController::class, 'update'])
    ->middleware([
        ProductRouteHydrator::class,
        ProductRequestValidatorMiddleware::class,
        ProductHydrator::class,
    ]);
```

### 3. Create or Update Middleware

#### Validator Middleware

Gunakan `Http/Middlewares/{Domain}RequestValidatorMiddleware` dan extend `ValidatorMiddleware`.

Validator bertanggung jawab untuk:

- Validasi body request.
- Validasi query parameter.
- Menolak request sebelum masuk controller ketika data tidak valid.

#### Hydrator Middleware

Gunakan `Http/Middlewares/{Domain}Hydrator` dan extend `HydratorMiddleware`.

Hydrator bertanggung jawab untuk:

- Mengubah request body menjadi entity atau DTO yang siap dipakai Service.
- Mengubah route parameter menjadi entity.
- Menyimpan hasil hydrasi ke request attribute.

### 4. Implement Controller

Controller hanya boleh:

- Mengambil input yang sudah divalidasi atau dihidrasi dari request.
- Memanggil Service.
- Mengembalikan response class yang sesuai.

Response rule:

- `GET` collection: gunakan `LaravelCommon\Responses\PagedJsonResponse` dan Service function `getAll`.
- `GET`, `PATCH`, `DELETE` single entity: gunakan `LaravelCommon\Responses\SuccessResponse` dan Service function `get`.
- `POST` single entity: gunakan `LaravelCommon\Responses\ResourceCreatedResponse`.

Contoh struktur controller:

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

    public function store(Request $request): ResourceCreatedResponse
    {
        return new ResourceCreatedResponse(
            $this->service->create($request->attributes->get('product'))
        );
    }
}
```

### 5. Implement Service

Service berisi business logic dan orchestration.

Service boleh:

- Memanggil Repository.
- Memanggil Query class.
- Melakukan transaksi database.
- Mengatur business rule.
- Mengubah entity menjadi ViewModel bila dibutuhkan.

Service tidak boleh:

- Membaca raw request HTTP secara langsung.
- Mengembalikan response HTTP.
- Menyimpan validasi request yang seharusnya ada di Middleware.

Contoh:

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
        return $this->query->paginate($filters);
    }

    public function create(Product $product): Product
    {
        return $this->repository->save($product);
    }
}
```

### 6. Implement Repository

Repository digunakan untuk operasi database sederhana:

- `find`
- `save`
- `update`
- `delete`
- lookup sederhana berdasarkan field unik

Repository tidak digunakan untuk query kompleks dengan banyak filter, join, aggregation, atau pagination khusus. Gunakan Query class untuk kebutuhan tersebut.

### 7. Implement Query Class

Query class digunakan untuk akses database kompleks:

- Listing dengan filter dan pagination.
- Join antar tabel.
- Search.
- Sorting.
- Aggregation.
- Report query.

Query class harus mengembalikan data yang siap dipakai Service, bukan response HTTP.

### 8. Implement ViewModel When Needed

Gunakan `ViewModels` ketika response membutuhkan bentuk data yang berbeda dari Model.

ViewModel cocok untuk:

- Menyembunyikan field internal.
- Menggabungkan beberapa field.
- Menormalisasi format response.
- Menyiapkan data untuk API consumer.

### 9. Testing Workflow

Minimal test untuk fitur baru:

- Feature test untuk happy path endpoint.
- Feature test untuk validation error.
- Feature test untuk not found atau unauthorized bila relevan.
- Unit test Service bila business rule cukup kompleks.
- Test Query class bila filter, sorting, atau pagination kompleks.

Checklist test per method:

- `GET collection`: data ter-paginate, filter bekerja, response shape benar.
- `GET single`: entity ditemukan, not found ditangani.
- `POST`: validasi berjalan, entity dibuat, response menggunakan created response.
- `PATCH`: validasi berjalan, entity berubah, response success.
- `DELETE`: entity dihapus atau ditandai deleted, response success.

### 10. Review Checklist

Sebelum merge, pastikan:

- Controller tidak berisi business logic.
- Service tidak membaca `Request` langsung.
- Repository hanya berisi akses database sederhana.
- Query kompleks tidak ditempatkan di Repository.
- Middleware validator dan hydrator digunakan untuk request/route entity.
- Response class mengikuti aturan method.
- Naming class konsisten dengan domain.
- Error handling eksplisit untuk kondisi penting.
- Test mencakup happy path dan failure path utama.

## Recommended Implementation Order

1. Route
2. Validator Middleware
3. Hydrator Middleware
4. Repository atau Query
5. Service
6. Controller
7. ViewModel
8. Tests

Urutan ini membantu kontrak HTTP jelas dari awal, lalu implementasi bergerak dari input, domain logic, sampai response.
