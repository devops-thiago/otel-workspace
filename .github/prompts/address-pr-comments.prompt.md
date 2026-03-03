---
mode: 'agent'
description: 'Fetch PR review comments, apply valid fixes, run quality checks, commit, push, and resolve addressed threads'
---

Address review comments on a GitHub PR in one of the OTel repos.

PR URL: **${input:pr_url}**

---

## Step 1 — Fetch comments and thread IDs

Run all three in parallel:

**a) Fetch the PR page** (human-readable comment bodies):
```
fetch_webpage <pr_url>
```

**b) List all PR check statuses** — identify what is actually failing:
```bash
gh pr checks <pr_url> --json name,state,description
```
Note every check with `state: FAILURE`. These are actionable regardless of whether there is a review comment.
Key checks to watch:
- `SonarCloud Code Analysis` — quality gate failure
- `codecov/patch` or `codecov/project` — coverage below threshold
- `coverage/coveralls` — same

**c) Fetch review thread IDs via GitHub GraphQL** (needed to resolve threads in Step 8):

Extract `owner`, `repo`, `pr_number` from the URL, then:
```bash
gh api graphql -f query='
{
  repository(owner: "<owner>", name: "<repo>") {
    pullRequest(number: <pr_number>) {
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          comments(first: 1) {
            nodes { body }
          }
        }
      }
    }
  }
}'
```

Build a map: `thread_id → first 120 chars of comment body`.
You will use this in Step 8 to resolve threads that were addressed.

---

## Step 2 — Checkout the PR branch

```bash
cd <repo_path>
git fetch origin <branch> && git checkout <branch>
```

---

## Step 3 — Evaluate each comment

For every comment decide: **apply** or **skip**.

**Apply** if it identifies a real bug, security issue, or clear improvement.

**Apply** failing CI checks as well as review comments:
- `SonarCloud Code Analysis` failed → fix every new issue flagged (see Step 4b)
- `codecov/patch` or `codecov/project` failed → add tests until threshold is met (see Step 4b)

**Skip** (note reason, do not change code) if it is:
- Editorial / PR description mismatch
- Stylistic preference with no correctness impact
- Already addressed in a prior commit on this branch
- Codecov bot comment reporting a minor drop when the check itself is still **passing**
- Informational bot messages (github-advanced-security setup notice, etc.)

---

## Step 4 — Apply all fixes

Use `multi_replace_string_in_file` to batch all changes in a single call.

### Common patterns across these repos

#### CI / GitHub Actions
| Problem | Fix |
|---------|-----|
| `trivy-action@master` unpinned | `aquasecurity/trivy-action@0.28.0` |
| Duplicate `format` + `lint` jobs both running identical command | Remove the redundant job; update `needs: [format, lint]` → `needs: [format]` |
| `build` job no longer waits for tests (`needs: [format, lint]`) | `needs: [format, test]` |
| SARIF upload fails on fork PRs | `if: ${{ always() && github.event.pull_request.head.repo.full_name == github.repository }}` |

#### Makefile (Java / Quarkus)
| Problem | Fix |
|---------|-----|
| `lint` missing from `.PHONY` | Add `lint` to `.PHONY` list |
| `verify` duplicates `fmt-check` + `test` commands inline | `verify: fmt-check test` |

#### Node.js — `userRepository.js`
| Problem | Fix |
|---------|-----|
| Optional field passed directly to `mysql2` — `undefined` errors | `const bio = userData.bio ?? null;` |

#### Node.js — `database.js`
| Problem | Fix |
|---------|-----|
| `CREATE TABLE IF NOT EXISTS` won't add new columns to existing tables | Add `information_schema` check + `ALTER TABLE ADD COLUMN` migration block |

#### Node.js — integration tests
| Problem | Fix |
|---------|-----|
| `if (skipIfNoDb()) return;` silently passes with 0 assertions | Replace with `RUN_INTEGRATION_TESTS` env-flag + `describe.skip` |

#### .NET — `UserDtoValidationTests.cs`
| Problem | Fix |
|---------|-----|
| `[InlineData("   ")]` on a `[Required]`-only field | Remove the whitespace-only case (keep `""`) |

#### .NET — `UserControllerIntegrationTests.cs`
| Problem | Fix |
|---------|-----|
| `_fixture.Create<string>().Substring(0, 5)` can throw | `Guid.NewGuid().ToString("N")[..5]` |

#### pom.xml
| Problem | Fix |
|---------|-----|
| Trailing space after `</project>` | Remove it |

---

## Step 4b — Fix coverage gaps and SonarCloud issues

Run **only if** Step 1b shows a failing coverage or SonarCloud check.

### Coverage gaps

Measure coverage and identify uncovered lines introduced by this PR:

| Repo | Command |
|------|---------|
| Node.js | `npm run test:unit -- --coverage` |
| Python | `poetry run pytest tests/unit/ --cov=app --cov-report=term-missing -v` |
| Go | `go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out` |
| .NET | `dotnet test UserApi.sln --collect:"XPlat Code Coverage"` |
| Java / Quarkus | `make test` (JaCoCo report: `target/site/jacoco/index.html`) |

Focus on **new public methods and handlers added in this PR**. Add unit tests until the
threshold is met. Do not pad with trivial tests — cover meaningful execution paths
(happy path + at least one error/edge case per handler).

### SonarCloud quality gate

1. Find the project key in `sonar-project.properties` at the repo root.
2. Read the `sonarcloud[bot]` comment on the PR — it lists every new issue with file and line.
3. Fix issues in priority order: **Bugs → Vulnerabilities → Security Hotspots → Code Smells**.
4. Common patterns:

| SonarCloud issue | Typical fix |
|------------------|-------------|
| Unhandled promise / missing `await` | Add `await` or `.catch()` |
| Resource not closed (Java/Go) | `defer` / try-with-resources |
| Hardcoded credential smell | Move to env var / config |
| Cognitive complexity too high | Extract helper function |
| Duplicate string literal | Extract constant |
| Missing null/error check | Add guard clause |

5. After fixing, re-run local tests to confirm nothing broke.

---

## Step 5 — Run quality checks for the affected repo

Run these in the repo directory **before committing**. Fix any failures before proceeding.

### Node.js
```bash
npm run format
npm run lint
npm audit --audit-level=high
npm run test:unit -- --coverage
```

### Python
```bash
poetry run ruff check --select I --fix app/ tests/
poetry run black app/ tests/
poetry run ruff check app/ tests/
poetry run pytest tests/unit/ --cov=app --cov-report=term-missing -v
```

### Go
```bash
make fmt && make vet && make lint
go test ./... -coverprofile=coverage.out && go tool cover -func=coverage.out
```

### .NET
```bash
dotnet format UserApi.sln --verify-no-changes
dotnet test UserApi.sln --collect:"XPlat Code Coverage"
```

### Java / Spring Boot
```bash
make fmt-check && make test
# JaCoCo report: target/site/jacoco/index.html
```

### Quarkus
```bash
make fmt-check && make test
# JaCoCo report: target/site/jacoco/index.html
```

**If any check fails:** fix the issue, re-run that check, then continue.
Do not commit with failing tests or lint errors.

---

## Step 6 — Verify fixes are in place

```bash
grep -n "<key_patterns>" <changed_files>
```

Spot-check each addressed comment against the file diff before committing.

---

## Step 7 — Commit and push

```bash
git add <changed_files>
git commit -m "<type>: address PR review comments

- <one bullet per change — file, old value → new value>"
git push origin <branch>
```

Subject line: follow Conventional Commits (see `pr-and-commit-standards.prompt.md`).
Body: one bullet per logical change, specific (file name + what changed).

---

## Step 8 — Resolve addressed threads

For each comment that was **applied**, resolve its review thread using the IDs
collected in Step 1:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {threadId: "<thread_id>"}) {
    thread { isResolved }
  }
}'
```

Run one call per addressed thread. Skip threads that were in the **skip** list.

After resolving, confirm all targeted threads show `"isResolved": true`:
```bash
gh api graphql -f query='
{
  repository(owner: "<owner>", name: "<repo>") {
    pullRequest(number: <pr_number>) {
      reviewThreads(first: 100) {
        nodes { id isResolved }
      }
    }
  }
}'
```

---

## Step 9 — Report

```
## Applied (N comments / failing checks)
- [file:line] <brief description of fix>
- ...

## Skipped (N items)
- <reason>
- ...

## Quality checks
- Format:   ✅ / ❌
- Lint:     ✅ / ❌
- Audit:    ✅ / ❌
- Tests:    ✅ N passed / ❌ N failed
- Coverage: ✅ N% (above threshold) / ❌ N% (below threshold)

## CI gate status (after push)
- SonarCloud:   ✅ passed / ❌ N issues remaining
- Codecov:      ✅ passed / ❌ N% (threshold: N%)

## Threads resolved: N / total
```
