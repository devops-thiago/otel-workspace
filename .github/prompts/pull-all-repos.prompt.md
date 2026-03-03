---
mode: 'agent'
description: 'Clone any missing OTel repos and pull latest main for all of them'
---

Ensure all OTel workspace repositories are present and up to date.

## Repo registry

| Local path | GitHub URL |
|------------|------------|
| `/Users/thiago/repos/otel-core-example` | `https://github.com/devops-thiago/otel-core-example.git` |
| `/Users/thiago/repos/otel-example-go` | `https://github.com/devops-thiago/otel-example-go.git` |
| `/Users/thiago/repos/otel-example-nodejs` | `https://github.com/devops-thiago/otel-example-nodejs.git` |
| `/Users/thiago/repos/otel-example-python` | `https://github.com/devops-thiago/otel-example-python.git` |
| `/Users/thiago/repos/otel-example-java` | `https://github.com/devops-thiago/otel-example-java.git` |
| `/Users/thiago/repos/otel-example-quarkus` | `https://github.com/devops-thiago/otel-example-quarkus.git` |

## Step 1 — Clone any missing repos

For each repo whose local path does **not** exist:
```bash
git clone <github_url> <local_path>
```

## Step 2 — Pull latest main for all repos

For each repo (whether just cloned or already present):
```bash
cd <local_path>
git fetch origin
git checkout main
git pull origin main
```

If a repo has uncommitted changes that would block the pull, report them and **skip
that repo** without stashing or discarding changes.

## Step 3 — Report

```
| Repo                  | Result                          |
|-----------------------|---------------------------------|
| otel-core-example     | cloned / up to date / updated / skipped (dirty) |
| otel-example-go       | ...                             |
| otel-example-nodejs   | ...                             |
| otel-example-python   | ...                             |
| otel-example-java     | ...                             |
| otel-example-quarkus  | ...                             |
```
