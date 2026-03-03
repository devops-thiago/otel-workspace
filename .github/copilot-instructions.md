# OTel Workspace — Copilot Instructions

This workspace is a collection of equivalent OpenTelemetry CRUD API examples,
one per language/framework. All repos must stay equivalent in functionality,
data model, API surface, and test coverage.

## Repository Map

| Repo folder             | Language / Framework       | Base path                              |
|-------------------------|----------------------------|----------------------------------------|
| `otel-core-example`     | C# / ASP.NET Core          | `/Users/thiago/repos/otel-core-example`  |
| `otel-example-go`       | Go / Gin                   | `/Users/thiago/repos/otel-example-go`    |
| `otel-example-nodejs`   | Node.js / Express + Joi    | `/Users/thiago/repos/otel-example-nodejs`|
| `otel-example-python`   | Python / FastAPI           | `/Users/thiago/repos/otel-example-python`|
| `otel-example-java`     | Java / Spring Boot         | `/Users/thiago/repos/otel-example-java`  |
| `otel-example-quarkus`  | Java / Quarkus (Panache)   | `/Users/thiago/repos/otel-example-quarkus`|

All repos share the same OTel observability stack: **Alloy → Mimir, Loki, Tempo → Grafana**.

---

## Canonical User Model

```
id          INT / BIGINT  PRIMARY KEY AUTO_INCREMENT
name        VARCHAR       NOT NULL
email       VARCHAR       NOT NULL UNIQUE
bio         TEXT/VARCHAR  NULLABLE (optional)
created_at  TIMESTAMP     DEFAULT CURRENT_TIMESTAMP
updated_at  TIMESTAMP     ON UPDATE CURRENT_TIMESTAMP
```

## Canonical API Endpoints

| Method   | Path              | Success response |
|----------|-------------------|------------------|
| `GET`    | `/api/users`      | 200 + pagination |
| `POST`   | `/api/users`      | 201 + created user |
| `GET`    | `/api/users/:id`  | 200 + user or 404 |
| `PUT`    | `/api/users/:id`  | 200 + updated user or 404 |
| `DELETE` | `/api/users/:id`  | **204 No Content** or 404 |

Health endpoints: `GET /health`, `GET /health/live`, `GET /health/ready`, `GET /`

---

## Quality Baseline (per language)

### Node.js
- Formatter: `prettier`
- Linter: `eslint` (eslint-config-prettier)
- Audit: `npm audit --audit-level=high`
- Tests: `jest` (unit + integration, `--runInBand`)

### Python
- Import order: `ruff --select I` (isort rules inside ruff)
- Formatter: `black`
- Linter: `ruff check` (includes pyflakes, pycodestyle, bugbear, etc.)
- Tests: `pytest` via `poetry run`
- Note: `tests/unit/conftest.py` overrides `get_db` dependency to prevent real DB connections

### Go
- Formatter: `gofmt -s -w`
- Vet: `go vet ./...`
- Linter: `golangci-lint run ./...`
- Tests: `go test ./... -count=1`

### .NET
- Formatter: `dotnet format UserApi.sln`
- Tests: `dotnet test UserApi.sln`

### Java / Spring Boot & Quarkus
- Formatter: `make fmt` (`mvn spotless:apply`) — google-java-format 1.27.0, AOSP style
- Format check: `make fmt-check` (`mvn spotless:check`)
- Build & test: `make test` (`mvn test`)
- JDK note: requires JDK 25; `.mvn/jvm.config` provides `--add-exports` for google-java-format

---

## Key Implementation Notes

- **Python venv**: project uses `poetry`; poetry is at `/opt/homebrew/bin/poetry`
- **Go lint**: `golangci-lint` must be installed (`brew install golangci-lint`)
- **Python deprecations to watch**:
  - Use `pythonjsonlogger.json` (not `.jsonlogger`)
  - Use `status.HTTP_422_UNPROCESSABLE_CONTENT` (not `_ENTITY`)
- **Python unit tests**: `tests/unit/conftest.py` provides an `autouse` fixture that overrides `get_db` with a `MagicMock` session so tests never require a real database
