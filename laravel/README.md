# Laravel Workflow

Dokumentasi ini berisi standar arsitektur dan workflow pengembangan Laravel yang bisa dipakai ulang di project Laravel lain.

## Documents

- [Architecture](architecture.md): struktur layer, dependency direction, tanggung jawab class, dan anti-pattern.
- [Development Workflow](development-workflow.md): urutan kerja implementasi fitur dari route sampai test.
- [Naming](naming.md): konvensi nama class, method, route, dan request attribute.
- [API Response](api-response.md): standar response API dan penggunaan `LaravelCommon\Responses`.
- [Testing](testing.md): standar test untuk endpoint, service, repository, dan query.
- [CRUD Example](examples/crud.md): contoh implementasi CRUD domain sederhana.

## Recommended Reading Order

1. `architecture.md`
2. `development-workflow.md`
3. `naming.md`
4. `api-response.md`
5. `testing.md`
6. `examples/crud.md`

## Core Principles

- Controller harus tipis.
- Business logic berada di Service.
- Repository hanya untuk akses database sederhana.
- Query class untuk pembacaan database kompleks.
- Request validation dan hydration dilakukan lewat Middleware.
- API response menggunakan response class standar.
- Test harus mencakup happy path dan failure path utama.
