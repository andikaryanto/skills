# Laravel Testing Standard

This document defines testing standards for Laravel features.

## Core Principle

Feature tests and focused class tests serve different purposes:

- Feature tests verify that routes, middleware, controllers, persistence, and
  responses work together from the API consumer's perspective.
- Focused tests verify the smallest useful behavior of one application class,
  with collaborators replaced or controlled where practical.

Endpoint coverage does not replace focused coverage for a changed class.

Every changed concrete application class with executable behavior must have a
dedicated test class unless it qualifies for an exclusion below.

## Changed-Class Coverage Audit

Before writing tests:

1. List production PHP files changed against `development`.
2. Ignore tests, generated files, migrations, configuration arrays, and route
   declaration files when building the class list.
3. For each changed concrete application class, identify:
   - dedicated test class;
   - public methods;
   - conditional branches and failure paths;
   - existing feature or integration coverage;
   - missing focused coverage.
4. Do not finish until every class is marked `covered` or `excluded` with a
   reason allowed by this document.

Example:

| Changed class | Dedicated test | Feature coverage | Status |
| --- | --- | --- | --- |
| `ProductController` | `ProductControllerTest` | `ProductApiTest` | Covered |
| `ProductHydratorMiddleware` | `ProductHydratorMiddlewareTest` | `ProductApiTest` | Covered |

## Test Location and Naming

Mirror the application namespace under `tests/Unit` for focused tests:

- `app/Http/Controllers/ProductController.php`
  → `tests/Unit/Http/Controllers/ProductControllerTest.php`
- `app/Http/Middleware/ProductHydratorMiddleware.php`
  → `tests/Unit/Http/Middleware/ProductHydratorMiddlewareTest.php`

Feature tests belong under `tests/Feature`.

The dedicated test name must match the production class:

- `ProductController` → `ProductControllerTest`
- `ProductHydratorMiddleware` → `ProductHydratorMiddlewareTest`

## Allowed Exclusions

A dedicated focused test is not required by default for:

- migrations;
- route declaration files;
- configuration files that only return static arrays;
- framework bootstrap wiring with no project-specific branching;
- empty marker classes or classes containing no executable project behavior.

Factories should be exercised through model or feature tests. A separate factory
test is required only when the factory contains states or custom behavior.

Document every exclusion in the final test report. Do not exclude controllers,
middleware, validators, services, queries, repositories, models, or view models
only because a feature test executes them indirectly.

## Minimum Coverage

New features must have at least:

- Feature test for the happy path.
- Feature test for validation errors.
- Feature test for not found, unauthorized, or forbidden cases when relevant.
- For tenant-owned data, feature tests proving cross-tenant reads and writes are
  rejected and tenant-scoped uniqueness behaves correctly.
- Focused test for each controller method and its collaborator interactions.
- Focused test for each middleware's hydration, validation, or failure behavior.
- Unit test for Services when business rules are complex enough.
- Query test when filters, sorting, pagination, joins, or aggregations are complex enough.

## Feature Tests

Feature tests focus on endpoint behavior from the API consumer's perspective.
They provide integration confidence but do not count as the dedicated test for
controllers or middleware.

Tests to cover per method:

- `GET collection`: successful response, correct pagination, filters work.
- `GET single`: entity is found and response shape is correct.
- `POST`: validation runs, data is stored, response is created.
- `PATCH`: data is changed, response is successful.
- `DELETE`: data is deleted or status is changed, response is successful.

## Controller Tests

Test controller methods directly with controlled Requests and mocked or
test-double collaborators.

Cover:

- the response type and resource passed to it;
- query filters selected from request input;
- `persist()`, `remove()`, and `flush()` interactions;
- delegation to Services or Queries;
- controller-specific branches.

Do not repeat routing or middleware assertions here; those belong in feature
tests.

## Hydrator Middleware Tests

Test hydration separately from the endpoint.

Cover:

- POST creates and hydrates a new model;
- PATCH changes only supplied fields;
- GET and DELETE resolve the requested model;
- each mapped input invokes the correct model setter;
- missing resources and hydration failures produce the expected response.

## Validation Tests

For each request validator, test at least:

- Required field.
- Invalid type.
- Invalid enum value.
- Invalid relation id when present.
- Boundary values such as minimum, maximum, length, or date range.

## Service Tests

Service tests are used when business logic is not safe enough to test only through endpoints.

Simple CRUD persistence through UnitOfWorkService does not require a domain Service unit test.

Examples of logic that needs unit tests:

- State transition.
- Domain permissions.
- Calculation.
- Transaction behavior.
- Rules involving more than one entity.

### Service Test Style

Service tests use Codeception Specify and Prophecy.

Use:

- `Codeception\Specify`
- `Prophecy\PhpUnit\ProphecyTrait`
- `beforeSpecify()` for shared setup
- `describe()` for the method under test
- `it()` for the behavior being verified
- `prophesize()` for service, repository, and UnitOfWork dependencies
- `shouldBeCalled()` for expected dependency calls
- `Argument::that()` for complex object assertions
- `verify()` for final state assertions

Example:

```php
final class ProductApprovalServiceTest extends TestCase
{
    use Specify;
    use ProphecyTrait;

    protected ProductApprovalService $service;

    public function test()
    {
        $this->beforeSpecify(function () {
            $this->unitOfWork = $this->prophesize(UnitOfWork::class);

            $this->service = new ProductApprovalService(
                $this->unitOfWork->reveal()
            );

            $this->product = (new Product())
                ->setId(10)
                ->setStatus(Product::STATUS_DRAFT);
        });

        $this->describe('->approve()', function () {
            $this->it('approves the product and persists it', function () {
                $this->unitOfWork
                    ->persist(Argument::that(function ($product) {
                        return $product instanceof Product &&
                            $product->getId() === 10 &&
                            $product->getStatus() === Product::STATUS_APPROVED;
                    }))
                    ->shouldBeCalled();

                $product = $this->service->approve($this->product);

                verify($product->getStatus())->equals(Product::STATUS_APPROVED);
            });
        });
    }
}
```

## Model Tests

Model getter and setter methods should stay simple, but relation wrappers and non-trivial casts should be covered when they affect behavior.

Getter and setter methods make models easier to use in Service tests because test setup can create real model instances without relying on raw attributes:

```php
$registration = (new Registration())
    ->setId(10);

$registrationStatus = (new RegistrationStatus())
    ->setId(Registration::STATUS_PEMERIKSAAN_AWAL)
    ->setName('Pemeriksaan Awal');
```

Mocking model getters and setters is allowed in Service tests when the Service only needs the model contract.

## Query Tests

Every changed custom Query class requires a dedicated query test. Test custom
filters even when they appear simple; add sorting, pagination, joins, and
aggregation tests when implemented.

Minimum tests:

- Main chainable filters.
- Sorting.
- Pagination.
- Join or relation loading.
- Aggregation when present.

## Repository Tests

Every changed custom Repository class requires a focused test for its
project-specific model binding and any custom lookup or persistence behavior.
Inherited vendor behavior does not need to be exhaustively retested.

## ViewModel Tests

Test:

- response shape;
- links;
- type conversion;
- conditional or embedded resources;
- collection behavior, including rejected model types when applicable.

## Test Naming

Feature and query tests should use method names that describe behavior.

```php
public function test_it_creates_product(): void
public function test_it_rejects_invalid_price(): void
public function test_it_filters_products_by_status(): void
```

Specify-based Service tests may use a single `test()` method and put behavior names inside `describe()` and `it()`.

## Factories

Use factories for test data setup.

Avoid large fixtures that are hard to read. Test setup should be small enough to explain the scenario.
