---
mode: 'agent'
description: 'Run lint, format, audit/security-scan, and tests across all OTel repos; fix any issues found'
---

Run the full quality pipeline for every repo in this workspace.
For each repo: **format → lint → security audit → tests**.
Fix any issues found before reporting results.

---

## Node.js — `/Users/thiago/repos/otel-example-nodejs`

```bash
cd /Users/thiago/repos/otel-example-nodejs

# 1. Format
npm run format

# 2. Lint + auto-fix
npm run lint:fix

# 3. Security audit (fail on high+)
npm audit --audit-level=high
# If vulnerabilities found, attempt:
npm audit fix

# 4. Unit tests
npm run test:unit
```

Expected: 0 lint errors, 0 high vulnerabilities, all tests green.

---

## Python — `/Users/thiago/repos/otel-example-python`

```bash
cd /Users/thiago/repos/otel-example-python

# Ensure poetry is available (homebrew: /opt/homebrew/bin/poetry)
# Ensure deps installed:
poetry install --no-root

# 1. Import sorting (isort via ruff)
poetry run ruff check --select I --fix app/ tests/

# 2. Format with black
poetry run black app/ tests/

# 3. Lint with ruff (includes pyflakes, bugbear, pyupgrade, etc.)
poetry run ruff check app/ tests/
# Auto-fix if needed:
poetry run ruff check --fix app/ tests/

# 4. Unit tests (no database required)
poetry run pytest tests/unit/ -v
```

Expected: 0 lint errors, 0 warnings, all tests green.

### Python watchlist — fix on sight
- `from pythonjsonlogger.jsonlogger import JsonFormatter` →
  `from pythonjsonlogger.json import JsonFormatter`
- `status.HTTP_422_UNPROCESSABLE_ENTITY` →
  `status.HTTP_422_UNPROCESSABLE_CONTENT`
- If `tests/unit/conftest.py` is missing, create it with an `autouse` fixture
  that overrides `get_db` with a `MagicMock` session (prevents real DB calls):

```python
from collections.abc import AsyncGenerator
from unittest.mock import MagicMock
import pytest
from app.database import get_db
from app.main import app

@pytest.fixture(autouse=True)
def override_get_db():
    mock_session = MagicMock()
    async def mock_get_db():
        yield mock_session
    app.dependency_overrides[get_db] = mock_get_db
    yield
    app.dependency_overrides.clear()
```

---

## Go — `/Users/thiago/repos/otel-example-go`

```bash
cd /Users/thiago/repos/otel-example-go

# 1. Format
make fmt          # runs: gofmt -s -w .

# 2. Vet
make vet          # runs: go vet ./...

# 3. Lint (install if missing: brew install golangci-lint)
make lint         # runs: golangci-lint run ./...

# 4. Tests
make test         # runs: go test ./... -count=1
```

Expected: 0 vet issues, 0 lint issues, all packages pass.

---

## .NET — `/Users/thiago/repos/otel-core-example`

```bash
cd /Users/thiago/repos/otel-core-example

# 1. Format (specify solution to avoid ambiguity)
dotnet format UserApi.sln

# 2. Tests
dotnet test UserApi.sln
```

Expected: `dotnet format` produces no diff, all tests pass.

---

## Java / Spring Boot — `/Users/thiago/repos/otel-example-java`

```bash
cd /Users/thiago/repos/otel-example-java

# 1. Format with google-java-format (via Spotless)
make fmt          # runs: mvn spotless:apply

# 2. Check format is clean (CI gate)
make fmt-check    # runs: mvn spotless:check

# 3. Build + test (uses H2 in-memory DB)
make test         # runs: mvn test -q
```

Expected: `spotless:check` reports no violations, BUILD SUCCESS, 0 failures.

---

## Quarkus — `/Users/thiago/repos/otel-example-quarkus`

```bash
cd /Users/thiago/repos/otel-example-quarkus

# 1. Format with google-java-format (via Spotless)
make fmt          # runs: mvn spotless:apply

# 2. Check format is clean (CI gate)
make fmt-check    # runs: mvn spotless:check

# 3. Build + test (uses Quarkus test profile with H2)
make test         # runs: mvn test
```

Expected: `spotless:check` reports no violations, BUILD SUCCESS, 0 failures.

---

## Summary

After running all pipelines, report a table:

| Repo | Format | Lint | Audit/Security | Tests |
|------|--------|------|----------------|-------|
| Node.js | ✅/❌ | ✅/❌ | ✅/❌ | N passed |
| Python | ✅/❌ | ✅/❌ | N/A | N passed |
| Go | ✅/❌ | ✅/❌ | N/A | N passed |
| .NET | ✅/❌ | N/A | N/A | N passed |
| Java | ✅/❌ | N/A | N/A | N passed |
| Quarkus | ✅/❌ | N/A | N/A | N passed |
