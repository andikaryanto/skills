# Feature Definition Workflow

This document describes the workflow for defining a new feature before implementation.

Implementation details belong in `development-workflow.md`.

## Goal

Produce a complete feature contract before writing code.

Do not start implementation if important requirements are missing.

## When to Use

Use this workflow when:

- Creating a new module.
- Creating a new API resource.
- Creating a new business process.
- Requirements are not fully defined.

Examples:

- Create Product Module.
- Create Inventory Management.
- Create Payment Feature.

## Define the Main Entity

Identify:

- Entity name.
- Entity purpose.
- Main responsibilities.

Example:

```text
Entity: Product

Purpose:
Represents a sellable product in the catalog.
```

## Define Fields

For each field, define:

- Name.
- Type.
- Required or optional.
- Validation constraints.

Example:

```text
name: string, required
sku: string, required, unique
price: decimal, required, greater than zero
status: enum, required
```

## Define Relationships

Identify entity relationships.

Examples:

```text
Product belongs to Category.

Product has many ProductImages.
```

## Define Endpoints

For each endpoint, define:

- Route.
- HTTP method.
- Request payload.
- Success response.
- Possible error responses.

Example:

```text
GET /api/products

POST /api/products

PATCH /api/products/{id}

DELETE /api/products/{id}
```

## Define Business Rules

Identify important domain rules.

Examples:

- SKU must be unique.
- Price cannot be negative.
- Archived products cannot be updated.

## Define Permissions

Identify authentication and authorization requirements.

Examples:

- Only authenticated users may create products.
- Only administrators may delete products.

## Define Acceptance Criteria

Describe expected behavior.

Examples:

- Product can be created.
- Product can be updated.
- Product can be archived.
- Validation errors return appropriate responses.

## Output

Produce a feature contract document containing:

- Entity definition.
- Fields.
- Relationships.
- Endpoints.
- Business rules.
- Permissions.
- Acceptance criteria.

Wait for approval before implementation.
