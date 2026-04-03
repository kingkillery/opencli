# Testing Guide

> A testing reference guide for developers and AI Agents.

## Table of Contents

- [Test Architecture](#test-architecture)
- [Current Coverage](#current-coverage)
- [Running Tests Locally](#running-tests-locally)
- [How to Add New Tests](#how-to-add-new-tests)
- [CI/CD Pipeline](#cicd-pipeline)
- [Browser Mode](#browser-mode)
- [Site Compatibility](#site-compatibility)

---

## Test Architecture

Tests are organized into three layers, all run with **vitest**:

```text
tests/
├── e2e/                           # E2E integration tests (real CLI run in subprocess)
│   ├── helpers.ts                 # runCli() / parseJsonOutput() shared utilities
│   ├── public-commands.test.ts    # Public API commands
│   ├── browser-public.test.ts     # Browser commands (public data)
│   ├── browser-auth.test.ts       # Commands requiring login (graceful failure)
│   ├── management.test.ts         # Management commands (list / validate / verify / help)
│   └── output-formats.test.ts     # Output format validation
├── smoke/
│   └── api-health.test.ts         # External API, adapter definition, and command registry health checks
src/
└── **/*.test.ts                   # Unit tests (currently 32 files)
```

| Layer | Location | File Count | Run Command | Purpose |
|---|---|---:|---|---|
| Unit tests | `src/**/*.test.ts` | 32 | `npx vitest run src/` | Internal modules, pipeline, adapter utilities |
| E2E tests | `tests/e2e/*.test.ts` | 5 | `npx vitest run tests/e2e/` | Real CLI command execution |
| Smoke tests | `tests/smoke/*.test.ts` | 1 | `npx vitest run tests/smoke/` | External API and registry integrity |

---

## Current Coverage

### Unit Tests (32 files)

| Domain | Files |
|---|---|
| Core runtime & output | `src/browser.test.ts`, `src/browser/dom-snapshot.test.ts`, `src/build-manifest.test.ts`, `src/capabilityRouting.test.ts`, `src/doctor.test.ts`, `src/engine.test.ts`, `src/interceptor.test.ts`, `src/output.test.ts`, `src/plugin.test.ts`, `src/registry.test.ts`, `src/snapshotFormatter.test.ts` |
| Pipeline & download | `src/download/index.test.ts`, `src/pipeline/executor.test.ts`, `src/pipeline/template.test.ts`, `src/pipeline/transform.test.ts` |
| Site / adapter logic | `src/clis/apple-podcasts/commands.test.ts`, `src/clis/apple-podcasts/utils.test.ts`, `src/clis/bloomberg/utils.test.ts`, `src/clis/chaoxing/utils.test.ts`, `src/clis/coupang/utils.test.ts`, `src/clis/google/utils.test.ts`, `src/clis/grok/ask.test.ts`, `src/clis/twitter/timeline.test.ts`, `src/clis/weread/utils.test.ts`, `src/clis/xiaohongshu/creator-note-detail.test.ts`, `src/clis/xiaohongshu/creator-notes-summary.test.ts`, `src/clis/xiaohongshu/creator-notes.test.ts`, `src/clis/xiaohongshu/search.test.ts`, `src/clis/xiaohongshu/user-helpers.test.ts`, `src/clis/xiaoyuzhou/utils.test.ts`, `src/clis/youtube/transcript-group.test.ts`, `src/clis/zhihu/download.test.ts` |

Key areas covered by these tests:

- Browser Bridge, DOM snapshot, interceptor, capability routing
- Manifest generation, command discovery, plugin installation and registry
- Output format rendering and snapshot formatting
- Pipeline template evaluation, executor and transform steps
- Data normalization, parameter handling and error tolerance for each site adapter

### E2E Tests (5 files)

| File | Current Coverage |
|---|---|
| `tests/e2e/public-commands.test.ts` | Public commands: `bloomberg`, `apple-podcasts`, `hackernews`, `v2ex`, `xiaoyuzhou`, `google suggest`, etc. |
| `tests/e2e/browser-public.test.ts` | `bbc`, `bloomberg`, `bilibili`, `weibo`, `zhihu`, `reddit`, `twitter`, `xueqiu`, `reuters`, `youtube`, `smzdm`, `boss`, `ctrip`, `coupang`, `xiaohongshu`, `google`, `yahoo-finance`, `v2ex daily` |
| `tests/e2e/browser-auth.test.ts` | Graceful failure for login-required commands: `bilibili`, `twitter`, `v2ex`, `xueqiu`, `linux-do`, `xiaohongshu` |
| `tests/e2e/management.test.ts` | `list`, `validate`, `verify`, `--version`, `--help`, unknown command |
| `tests/e2e/output-formats.test.ts` | Output format validation: `json` / `yaml` / `csv` / `md` |
| `tests/e2e/plugin-management.test.ts` | Full `plugin install` / `list` / `update` / `uninstall` lifecycle |

### Smoke Tests (1 file)

| File | Current Coverage |
|---|---|
| `tests/smoke/api-health.test.ts` | `hackernews`, `v2ex` public API availability, `validate` full adapter validation, and command registry basic integrity |

### Quick Check Commands

To refresh the test file list, use the repository files directly:

```bash
find src -name '*.test.ts' | sort
find tests/e2e -name '*.test.ts' | sort
find tests/smoke -name '*.test.ts' | sort
```

---

## Running Tests Locally

### Prerequisites

```bash
npm ci                # Install dependencies
npm run build         # Compile (required by E2E / smoke tests for dist/main.js)
```

### Run Commands

```bash
# All unit tests
npx vitest run src/

# All E2E tests (makes real external API / browser calls)
npx vitest run tests/e2e/

# All smoke tests
npx vitest run tests/smoke/

# Single test file
npx vitest run src/clis/apple-podcasts/commands.test.ts
npx vitest run tests/e2e/management.test.ts

# All tests
npx vitest run

# Watch mode (recommended during development)
npx vitest src/
```

### Notes for Browser Command Local Testing

- opencli connects to a running Chrome browser via the Browser Bridge extension
- E2E tests invoke the built `dist/main.js` via `runCli()` in `tests/e2e/helpers.ts`
- `browser-public.test.ts` uses `tryBrowserCommand()` — it warns and passes when a site returns empty data due to anti-scraping measures or geo-restrictions
- `browser-auth.test.ts` verifies **graceful failure**: the main concern is no crash, no hang, and a controlled error message
- To test with a full logged-in session, keep Chrome logged in with the Browser Bridge extension installed and run the relevant tests manually

---

## How to Add New Tests

### Adding a New YAML Adapter (e.g. `src/clis/producthunt/trending.yaml`)

1. The `opencli validate` E2E / smoke tests will cover adapter structure validation
2. Based on the adapter type, add an `it()` block to the corresponding test file

```typescript
// If browser: false (public API) → tests/e2e/public-commands.test.ts
it('producthunt trending returns data', async () => {
  const { stdout, code } = await runCli(['producthunt', 'trending', '--limit', '3', '-f', 'json']);
  expect(code).toBe(0);
  const data = parseJsonOutput(stdout);
  expect(Array.isArray(data)).toBe(true);
  expect(data.length).toBeGreaterThanOrEqual(1);
  expect(data[0]).toHaveProperty('title');
}, 30_000);
```

```typescript
// If browser: true but publicly accessible → tests/e2e/browser-public.test.ts
it('producthunt trending returns data', async () => {
  const data = await tryBrowserCommand(['producthunt', 'trending', '--limit', '3', '-f', 'json']);
  expectDataOrSkip(data, 'producthunt trending');
}, 60_000);
```

```typescript
// If browser: true and requires login → tests/e2e/browser-auth.test.ts
it('producthunt me fails gracefully without login', async () => {
  await expectGracefulAuthFailure(['producthunt', 'me', '-f', 'json'], 'producthunt me');
}, 60_000);
```

### Adding a New Management Command (e.g. `opencli export`)

Add a test in `tests/e2e/management.test.ts`; if the new command affects output format, also update `tests/e2e/output-formats.test.ts`.

### Adding a New Internal Module

Create a `*.test.ts` file next to the source file, preferably in the same directory as the module being tested for easy discovery and maintenance.

### Decision Flowchart

```text
New feature → Internal module? → Yes → add *.test.ts under src/
                    ↓ No
             CLI command? → browser: false? → tests/e2e/public-commands.test.ts
                                  ↓ true
                            Public data? → tests/e2e/browser-public.test.ts
                                  ↓ requires login
                            tests/e2e/browser-auth.test.ts
```

---

## CI/CD Pipeline

### `ci.yml`

| Job | Trigger | Description |
|---|---|---|
| `build` | push/PR to `main`, `dev` | `tsc --noEmit` + `npm run build` |
| `unit-test` | push/PR to `main`, `dev` | Run `src/` unit tests on Node `20` and `22`, parallelized across `2` shards |
| `smoke-test` | `schedule` or `workflow_dispatch` | Install real Chrome, run `tests/smoke/` via `xvfb-run` |

### `e2e-headed.yml`

| Job | Trigger | Description |
|---|---|---|
| `e2e-headed` | push/PR to `main`, `dev`, or manual trigger | Install real Chrome, run `tests/e2e/` via `xvfb-run` |

Both E2E and smoke jobs use `./.github/actions/setup-chrome` to prepare real Chrome and inject the browser path via `OPENCLI_BROWSER_EXECUTABLE_PATH`.

### Sharding

Unit tests use vitest's built-in shard feature and run on both Node `20` and `22`:

```yaml
strategy:
  matrix:
    node-version: ['20', '22']
    shard: [1, 2]
steps:
  - run: npx vitest run src/ --reporter=verbose --shard=${{ matrix.shard }}/2
```

---

## Browser Mode

opencli connects to a browser via the Browser Bridge extension:

| Condition | Mode | Use Case |
|---|---|---|
| Extension installed / connected | Extension mode | Local user connected to a logged-in Chrome |
| No extension token | CLI launches its own browser | CI, no login session, or fully automated scenarios |

Use `OPENCLI_BROWSER_EXECUTABLE_PATH` to specify the real Chrome path in CI:

```yaml
env:
  OPENCLI_BROWSER_EXECUTABLE_PATH: ${{ steps.setup-chrome.outputs.chrome-path }}
```

---

## Site Compatibility

On GitHub Actions US runners, some sites return empty data due to geo-restrictions, login requirements, or anti-scraping measures. E2E tests use a warn + pass strategy for these scenarios to prevent intermittent site restrictions from failing the entire CI pipeline.

| Site | CI Behavior | Common Reason |
|---|---|---|
| `hackernews`, `bbc`, `v2ex`, `bloomberg` | Usually returns data | Public API or public page |
| `yahoo-finance`, `google` | Usually returns data | Public page, but may be affected by rate limiting |
| `bilibili`, `zhihu`, `weibo`, `xiaohongshu`, `xueqiu` | Often returns empty data | Geo-restriction, anti-scraping, or login required |
| `reddit`, `twitter`, `youtube` | Often returns empty data | Login state, cookies, or bot detection |
| `smzdm`, `boss`, `ctrip`, `coupang`, `linux-do` | Results vary | Geo-restriction, risk control, or page structure changes |

> For more stable browser E2E results, prefer using a self-hosted runner with network access to the target sites.
