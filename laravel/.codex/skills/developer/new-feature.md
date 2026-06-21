# New Feature Definition

This document describes the workflow for defining a new feature before implementation.

Implementation details belong in `.codex/skills/developer/development.md`.

## Goal

Produce a complete feature contract before writing code.

Do not start implementation if important requirements remain missing after
inspecting the request, migrations, existing code, and tests.

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

Before implementation, establish a feature contract containing:

- Entity definition.
- Fields.
- Relationships.
- Endpoints.
- Business rules.
- Permissions.
- Acceptance criteria.

If the contract is fully supported by the request and repository context,
continue with `.codex/skills/developer/development.md`.

Ask for approval or clarification only when a missing business decision would
materially change the implementation.
