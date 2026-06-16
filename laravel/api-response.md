# Laravel API Response Standard

Dokumen ini mendefinisikan standar response API.

## Response Package

Semua response API standar harus menggunakan class dari:

```text
vendor/andikaryanto/laravelcommon/src/Responses
```

Jangan membuat response manual dengan `response()->json()` untuk endpoint API standar kecuali ada kebutuhan khusus yang disetujui.

## Response Mapping

| Method | Use Case | Response Class |
| --- | --- | --- |
| `GET` | Collection atau paginated list | `LaravelCommon\Responses\PagedJsonResponse` |
| `GET` | Single entity | `LaravelCommon\Responses\SuccessResponse` |
| `POST` | Created resource | `LaravelCommon\Responses\ResourceCreatedResponse` |
| `PATCH` | Updated resource | `LaravelCommon\Responses\SuccessResponse` |
| `DELETE` | Deleted resource | `LaravelCommon\Responses\SuccessResponse` |

## Controller Rules

Controller hanya memilih response class dan mengirim data dari Service.

```php
public function show(Request $request): SuccessResponse
{
    return new SuccessResponse(
        $this->service->get($request->attributes->get('product'))
    );
}
```

Controller tidak boleh:

- Membentuk struktur JSON manual.
- Mengubah data kompleks untuk response.
- Menentukan business status berdasarkan query langsung.

Gunakan ViewModel jika response shape tidak sama dengan Model.

## Pagination

Collection endpoint harus menggunakan response pagination standar.

```php
public function index(Request $request): PagedJsonResponse
{
    return new PagedJsonResponse(
        $this->service->getAll($request->query())
    );
}
```

Filtering, sorting, dan pagination diproses oleh Service dan Query class.

## Error Response

Validation error harus ditangani oleh `ValidatorMiddleware`.

Not found dan business error harus eksplisit di layer yang paling tepat:

- Route entity tidak ditemukan: Hydrator atau Repository.
- Business rule gagal: Service.
- Query result kosong untuk resource wajib: Query atau Service.

Endpoint tidak boleh menyembunyikan error penting dengan response success kosong.
