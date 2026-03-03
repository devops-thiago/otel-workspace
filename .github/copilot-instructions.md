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

## Workspace Bootstrap

All repos must live under `/Users/thiago/repos/`. If any are missing, clone them:

```bash
cd /Users/thiago/repos
git clone https://github.com/devops-thiago/otel-core-example.git
git clone https://github.com/devops-thiago/otel-example-go.git
git clone https://github.com/devops-thiago/otel-example-nodejs.git
git clone https://github.com/devops-thiago/otel-example-python.git
git clone https://github.com/devops-thiago/otel-example-java.git
git clone https://github.com/devops-thiago/otel-example-quarkus.git
```

To pull all repos to latest `main` (or clone missing ones), use the
`.github/prompts/pull-all-repos.prompt.md` agent prompt.

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

---

## Git Workflow Standards

### Branch naming
`<type>/<short-slug>` — e.g. `feat/user-bio-field`, `ci/add-format-lint-pr-checks`, `fix/mysql2-undefined-bind`

### Commit messages — Conventional Commits
```
<type>[(<scope>)]: <imperative description>   ← max 72 chars, no period

[body: what + why, 72-char wrap, bullets for multi-change commits]
```
Types: `feat` `fix` `ci` `docs` `chore` `refactor` `test` `perf` `deps`

Rules:
- Imperative mood ("add", not "adds" / "added")
- Body explains **what** and **why**, not how
- No `* Initial plan` artifacts from Copilot SWE agent
- No YAML dependabot metadata in manually authored commits

### PR descriptions
Every repo has `.github/PULL_REQUEST_TEMPLATE.md` with four sections:
1. **What** — one imperative sentence
2. **Why** — motivation / linked issues
3. **Changes** — one bullet per file/component changed, specific values
4. **Test Plan + Checklist** — check every applicable box before requesting review

Full details: see `.github/prompts/pr-and-commit-standards.prompt.md`
