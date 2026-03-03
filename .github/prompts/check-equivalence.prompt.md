---
mode: 'agent'
description: 'Audit and fix cross-repo equivalence: same User model, same API surface, same test coverage'
---

Audit all OTel repos to ensure they are equivalent in terms of:
1. **User model fields** — must match the canonical schema
2. **API endpoint surface** — must match the canonical routes
3. **HTTP status codes** — especially DELETE must return 204
4. **Test coverage** — every endpoint must have unit tests

## Canonical User Model

| Field        | Type          | Constraints             |
|--------------|---------------|-------------------------|
| `id`         | INT / BIGINT  | PK, AUTO INCREMENT      |
| `name`       | VARCHAR       | NOT NULL                |
| `email`      | VARCHAR       | NOT NULL, UNIQUE        |
| `bio`        | TEXT / VARCHAR| NULLABLE (optional)     |
| `created_at` | TIMESTAMP     | DEFAULT CURRENT_TIMESTAMP |
| `updated_at` | TIMESTAMP     | AUTO-UPDATE             |

## Canonical API Endpoints

| Method   | Path              | Success code |
|----------|-------------------|--------------|
| `GET`    | `/api/users`      | 200          |
| `POST`   | `/api/users`      | 201          |
| `GET`    | `/api/users/:id`  | 200 or 404   |
| `PUT`    | `/api/users/:id`  | 200 or 404   |
| `DELETE` | `/api/users/:id`  | **204** or 404 |

Health: `GET /health`, `GET /health/live`, `GET /health/ready`, `GET /`

## Repos to audit

| Path | Language |
|------|----------|
| `/Users/thiago/repos/otel-core-example` | C# / ASP.NET Core |
| `/Users/thiago/repos/otel-example-go` | Go / Gin |
| `/Users/thiago/repos/otel-example-nodejs` | Node.js / Express |
| `/Users/thiago/repos/otel-example-python` | Python / FastAPI |
| `/Users/thiago/repos/otel-example-java` | Java / Spring Boot |
| `/Users/thiago/repos/otel-example-quarkus` | Java / Quarkus |

## Audit steps

1. **Model audit** — read the model/entity file in each repo and compare field
   names, types, and nullability against the canonical schema above. Flag any
   extra, missing, or misnamed fields.

2. **Route audit** — read the router/controller file in each repo and verify all
   5 CRUD endpoints exist and return the correct HTTP status codes. Flag the Go
   handler's DELETE if it still returns 200 instead of 204.

3. **Test audit** — check that each endpoint has at least one positive and one
   negative (404 / 409 / 422) test case in the unit test file.

4. **Fix all discrepancies** — apply changes across all affected files in each
   repo (model, schema/DTO, repository, validator, routes, tests, init.sql /
   migration files) to bring every repo into alignment.

5. **Report** — after all fixes, summarise what was changed per repo.

## Key file locations per language

### Node.js
- Model / schema: `src/repositories/userRepository.js`, `src/validators/userValidator.js`, `src/database.js`, `init.sql`
- Routes: `src/routes/userRoutes.js`
- Tests: `tests/unit/userRoutes.test.js`, `tests/unit/userRepository.test.js`, `tests/unit/userValidator.test.js`, `tests/integration/users.test.js`

### Python
- Model: `app/models.py`, `app/schemas/user_schema.py`
- Routes: `app/routes/user_routes.py`
- Repository: `app/repository/user_repository.py`
- Tests: `tests/unit/test_user_routes.py`, `tests/unit/test_user_repository.py`

### Go
- Model: `internal/models/user.go`
- Handler: `internal/handlers/user_handler.go`
- Tests: `internal/handlers/user_handler_test.go`

### .NET
- Model: `Models/User.cs`
- DTOs: `DTOs/UserDto.cs`
- Service: `Services/UserService.cs`
- DbContext: `Data/UserDbContext.cs`
- Tests: `UserApi.Tests/` (Controllers, Services, Models, DTOs subdirs)

### Java / Spring Boot
- Model: `src/main/java/.../model/User.java`
- Controller: `src/main/java/.../controller/UserController.java`
- Tests: `src/test/java/.../`

### Quarkus
- Model: `src/main/java/.../model/User.java`
- Resource: `src/main/java/.../resource/UserResource.java`
- Tests: `src/test/java/.../`
