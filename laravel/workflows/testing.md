# Laravel Testing Standard

This document defines testing standards for Laravel features.

## Minimum Coverage

New features must have at least:

- Feature test for the happy path.
- Feature test for validation errors.
- Feature test for not found, unauthorized, or forbidden cases when relevant.
- Unit test for Services when business rules are complex enough.
- Query test when filters, sorting, pagination, joins, or aggregations are complex enough.

## Feature Tests

Feature tests focus on endpoint behavior from the API consumer's perspective.

Tests to cover per method:

- `GET collection`: successful response, correct pagination, filters work.
- `GET single`: entity is found and response shape is correct.
- `POST`: validation runs, data is stored, response is created.
- `PATCH`: data is changed, response is successful.
- `DELETE`: data is deleted or status is changed, response is successful.

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

Query tests are used to ensure complex data reads are correct.

Minimum tests:

- Main chainable filters.
- Sorting.
- Pagination.
- Join or relation loading.
- Aggregation when present.

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
