# shared-workflows

Reusable GitHub Actions workflows (via `workflow_call`) used across my repos. Change logic here once; every caller picks it up on next CI run.

## Workflows

### `ci-python.yml`

Python backend CI. Three jobs run in parallel:

- **backend** — `poetry install` → optional Alembic migration → `pytest`
- **lint** — `ruff check` + `ruff format --check`
- **frontend** (optional) — `npm ci` → `npm run lint` (non-blocking) → `npm run build`

#### Inputs

| Input | Type | Default | Purpose |
|-------|------|---------|---------|
| `python-version` | string | `"3.12"` | Python version for backend + lint |
| `node-version` | string | `"20"` | Node version for frontend build |
| `needs-postgres` | bool | `false` | Spin up `postgres:16-alpine` service container |
| `needs-redis` | bool | `false` | Spin up `redis:7-alpine` service container |
| `has-migrations` | bool | `false` | Run `alembic upgrade head` before tests |
| `test-command` | string | `pytest tests/unit --tb=short -q` | Pytest invocation |
| `ruff-paths` | string | `"src tests"` | Paths passed to ruff |
| `has-frontend` | bool | `false` | Run the Next.js build job |
| `frontend-dir` | string | `"frontend"` | Directory containing `package.json` |
| `frontend-public-api-url` | string | `"http://localhost:8000"` | Baked into `NEXT_PUBLIC_API_URL` at build |

#### Minimal caller

```yaml
# .github/workflows/ci.yml in YOUR repo
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    uses: akejonescode/shared-workflows/.github/workflows/ci-python.yml@main
```

#### Full caller (backend + db + frontend)

```yaml
jobs:
  ci:
    uses: akejonescode/shared-workflows/.github/workflows/ci-python.yml@main
    with:
      needs-postgres: true
      needs-redis: true
      has-migrations: true
      has-frontend: true
```

## Versioning

Callers pin `@main` for rolling updates. If you need stability, tag a release (`v1`) and pin to that tag instead.

## Related

- [`akejonescode/.github`](https://github.com/akejonescode/.github) — workflow templates shown in GitHub's "New workflow" picker that wire into this repo.
