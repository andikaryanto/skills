# Laravel Architecture

See also: [Development Workflow](development-workflow.md)

## Layers

app
    Consoles
    Enums
    Http
        Controllers
        Middlewares
    Queries
    Repositories
    Services
    Models
    ViewModels

## Rules

- Controllers must stay thin
    - GET: Collection response uses LaravelCommon\Responses\PagedJsonResponse, and use getAll function
    - GET|PATCH|DELETE: single entity response uses LaravelCommon\Responses\SuccessResponse, and use get function
    - POST: single entity response uses LaravelCommon\Responses\ResourceCreatedResponse
- Business logic belongs in Services
- Simple Database access through Repositories
- Complex database access to like query through Queries
- Hydrate Post / Patch request to entity through Http/Middlewares/{Domain}Hydrators that extends HydratorMiddleware
- Hydrate path route entity through Http/Middlewares/{Domain}Hydrators that extends HydratorMiddleware
- Validation Http/Middlewares/{Domain}RequestValidatorMiddleware that extends ValidatorMiddleware
- Responses through vendor/andikaryanto/laravelcommon/src/Responses
