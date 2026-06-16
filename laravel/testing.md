# Laravel Testing Standard

Dokumen ini mendefinisikan standar testing untuk fitur Laravel.

## Minimum Coverage

Fitur baru minimal memiliki:

- Feature test untuk happy path.
- Feature test untuk validation error.
- Feature test untuk not found, unauthorized, atau forbidden bila relevan.
- Unit test Service jika business rule cukup kompleks.
- Test Query jika filter, sorting, pagination, join, atau aggregation cukup kompleks.

## Feature Tests

Feature test berfokus pada behavior endpoint dari sudut pandang API consumer.

Test yang perlu ada per method:

- `GET collection`: response sukses, pagination benar, filter bekerja.
- `GET single`: entity ditemukan dan response shape benar.
- `POST`: validasi berjalan, data tersimpan, response created.
- `PATCH`: data berubah, response success.
- `DELETE`: data terhapus atau status berubah, response success.

## Validation Tests

Untuk setiap request validator, test minimal:

- Required field.
- Invalid type.
- Invalid enum value.
- Invalid relation id bila ada.
- Boundary value seperti minimum, maximum, length, atau date range.

## Service Tests

Service test digunakan ketika ada business logic yang tidak cukup aman hanya diuji lewat endpoint.

Contoh logic yang perlu unit test:

- State transition.
- Permission domain.
- Calculation.
- Transaction behavior.
- Rule yang melibatkan lebih dari satu entity.

## Query Tests

Query test digunakan untuk memastikan pembacaan data kompleks benar.

Test minimal:

- Filter utama.
- Sorting.
- Pagination.
- Join atau relation loading.
- Aggregation bila ada.

## Test Naming

Gunakan nama test yang menjelaskan behavior.

```php
public function test_it_creates_product(): void
public function test_it_rejects_invalid_price(): void
public function test_it_filters_products_by_status(): void
```

## Factories

Gunakan factory untuk setup data.

Hindari membuat fixture besar yang sulit dibaca. Setup data harus cukup kecil untuk menjelaskan skenario test.
