# Week 11 Day 2: Testing in CI

![CI](https://github.com/tsushumi/week-11-day-2-assignment/actions/workflows/ci.yml/badge.svg)
![PR Check](https://github.com/tsushumi/week-11-day-2-assignment/actions/workflows/pr-check.yml/badge.svg)
![Scheduled Health Check](https://github.com/tsushumi/week-11-day-2-assignment/actions/workflows/scheduled.yml/badge.svg)


A small Express API with three GitHub Actions workflows.

## Workflows

| File | Trigger | What it does |
| --- | --- | --- |
| `.github/workflows/ci.yml` | every push, any branch | Lint, then tests with coverage on Node 18/20 x ubuntu/macOS (4 matrix jobs); uploads `coverage/` as an artifact |
| `.github/workflows/pr-check.yml` | pull requests into `main` | `lint` job, then `test` job (`needs: lint`, `NODE_ENV=test`) running coverage |
| `.github/workflows/scheduled.yml` | Mondays 06:00 UTC (9:00 AM EAT) and manual | Curls the health endpoint, logs the result, posts to Discord if a webhook secret is set |

## Scripts

```bash
npm install
npm start      # run the API on PORT (default 3000)
npm run lint   # ESLint
npm test       # Vitest (single run)
npm run test:coverage   # Vitest with coverage; fails below 70% lines/functions/statements, 60% branches
```

`GET /api/health` returns `{ "status": "ok", ... }` and is what the scheduled workflow checks once deployed.

## Tests

- `tests/unit/helpers.test.js`: 4 unit tests for `utils/helpers.js`
- `tests/api/endpoints.test.js`: 4 supertest integration tests against `app.js`
- `*.extra.test.js`: additional tests that push coverage well above the thresholds

Tests use an in-memory store and need no database, network, or env vars.
`app.js` exports the Express app and `server.js` calls `listen()`, so supertest never binds a port.

## Demo pull requests (CI as a quality gate)

`demo-prs/` holds a passing and a failing change. After merging this work into `main`, run:

```bash
bash demo-prs/create-demo-prs.sh
```

then open a PR from each branch into `main`:

- `feature/add-helper` adds `capitalize()` with tests: **green check**
- `feature/broken-test` changes `formatAmount` output without updating tests: **red X**


## Configuring the scheduled workflow

In the repo go to **Settings > Secrets and variables > Actions**:

- **Variable** `HEALTH_CHECK_URL`: your deployed health URL, e.g. `https://your-api.railway.app/api/health`. If unset, the workflow uses a placeholder URL, reports "unhealthy", and still passes.


Test it any time from **Actions > Scheduled Health Check > Run workflow**.

## Testing the PR workflow

```bash
git checkout -b test-pr-workflow
# make a small change, then
git commit -am "Test PR workflow"
git push -u origin test-pr-workflow
# open a pull request into main on GitHub
```
