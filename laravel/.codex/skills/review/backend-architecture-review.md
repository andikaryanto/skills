## Architecture Checklist

Verify that:

- Controllers only orchestrate HTTP flow.
- Business logic is placed in Services.
- Query classes handle complex reads.
- Repositories only contain simple database access.
- UnitOfWorkService is used for simple persistence.
- Middleware handles validation and hydration.
- Response classes follow project standards.