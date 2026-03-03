---
mode: 'agent'
description: 'Pull latest changes for all OTel repos to their main branch'
---

Pull the latest changes for all repositories in this workspace.

For each of the following repos, in order:
1. `/Users/thiago/repos/otel-core-example`
2. `/Users/thiago/repos/otel-example-go`
3. `/Users/thiago/repos/otel-example-nodejs`
4. `/Users/thiago/repos/otel-example-python`
5. `/Users/thiago/repos/otel-example-java`
6. `/Users/thiago/repos/otel-example-quarkus`

Run these steps for each repo:
```bash
cd <repo>
git fetch origin
git checkout main
git pull origin main
```

Report the result per repo (up to date / fast-forward / error). If any repo has
uncommitted changes that would block the pull, report them and skip that repo
without stashing or discarding changes.
