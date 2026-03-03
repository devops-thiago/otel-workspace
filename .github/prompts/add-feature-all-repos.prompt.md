---
mode: 'agent'
description: 'Add a new field or endpoint to all 6 OTel repos simultaneously, keeping them equivalent'
---

Add a new feature (field, endpoint, or validation rule) to all repos simultaneously.
The feature to add is: **${input:feature}**

Use the canonical patterns below to implement it consistently in every repo.

---

## Checklist per repo

For each change, update ALL of:

### Node.js (`/Users/thiago/repos/otel-example-nodejs`)
- [ ] `init.sql` — add column to `CREATE TABLE` and UPDATE sample INSERTs
- [ ] `src/database.js` — update the `CREATE TABLE IF NOT EXISTS` schema
- [ ] `src/repositories/userRepository.js` — update SELECT / INSERT / UPDATE SQL strings
- [ ] `src/validators/userValidator.js` — add Joi rule to `create` and `update` schemas
- [ ] `src/routes/userRoutes.js` — update PUT merge logic
- [ ] `tests/unit/userRepository.test.js` — update mock objects and query param assertions
- [ ] `tests/unit/userValidator.test.js` — add/update validation test cases
- [ ] `tests/unit/userRoutes.test.js` — update mock user objects and test payloads
- [ ] `tests/integration/users.test.js` — update INSERT statements and POST/PUT payloads

### Python (`/Users/thiago/repos/otel-example-python`)
- [ ] `app/models.py` — add SQLAlchemy column
- [ ] `app/schemas/user_schema.py` — add field to `UserCreate`, `UserUpdate`, `UserResponse`
- [ ] `app/repository/user_repository.py` — update create/update logic
- [ ] `tests/unit/test_user_routes.py` — update fixtures and assertions
- [ ] `tests/unit/test_user_repository.py` — update mock data

### Go (`/Users/thiago/repos/otel-example-go`)
- [ ] `internal/models/user.go` — add struct field + db tag
- [ ] `internal/handlers/user_handler.go` — update create/update request structs
- [ ] `internal/repository/` — update INSERT/UPDATE queries
- [ ] `internal/handlers/user_handler_test.go` — update test payloads

### .NET (`/Users/thiago/repos/otel-core-example`)
- [ ] `Models/User.cs` — add property with data annotations
- [ ] `DTOs/UserDto.cs` — update `CreateUserDto`, `UpdateUserDto`, `UserResponseDto`
- [ ] `Services/UserService.cs` — update `CreateUserAsync`, `UpdateUserAsync`, `MapToResponseDto`
- [ ] `Data/UserDbContext.cs` — update model builder + seed data
- [ ] `UserApi.Tests/Models/UserModelTests.cs`
- [ ] `UserApi.Tests/DTOs/UserDtoValidationTests.cs`
- [ ] `UserApi.Tests/Services/UserServiceTests.cs`
- [ ] `UserApi.Tests/Controllers/UserControllerIntegrationTests.cs`
- [ ] `UserApi.Tests/TestUtilities.cs` — update factory methods

### Java / Spring Boot (`/Users/thiago/repos/otel-example-java`)
- [ ] `src/main/java/.../model/User.java` — add JPA field + column annotation
- [ ] `src/main/java/.../dto/` — add to request/response DTOs
- [ ] `src/main/java/.../service/UserService.java` — update create/update logic
- [ ] `src/test/java/.../` — update test data

### Quarkus (`/Users/thiago/repos/otel-example-quarkus`)
- [ ] `src/main/java/.../model/User.java` — add Panache field + column annotation
- [ ] Resource / service layer — update create/update logic
- [ ] `src/test/java/.../` — update test data

---

## After implementing

1. Run `quality-checks` to verify lint, format, and tests still pass in all repos.
2. Run `check-equivalence` to confirm all repos are in sync.
