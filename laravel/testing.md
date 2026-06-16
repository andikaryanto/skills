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

Examples of logic that needs unit tests:

- State transition.
- Domain permissions.
- Calculation.
- Transaction behavior.
- Rules involving more than one entity.

## Query Tests

Query tests are used to ensure complex data reads are correct.

Minimum tests:

- Main chainable filters.
- Sorting.
- Pagination.
- Join or relation loading.
- Aggregation when present.

## Test Naming

Use test names that describe behavior.

```php
public function test_it_creates_product(): void
public function test_it_rejects_invalid_price(): void
public function test_it_filters_products_by_status(): void
```

## Factories

Use factories for test data setup.

Avoid large fixtures that are hard to read. Test setup should be small enough to explain the scenario.
