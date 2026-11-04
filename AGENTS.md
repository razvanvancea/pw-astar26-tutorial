# AGENTS.md

## Project Overview

Workshop repository ("Scalable Playwright Framework with AI") containing Playwright end-to-end tests written in TypeScript against the demo shop `https://practicesoftwaretesting.com/`. It uses the Page Object Model with custom fixtures, and ships Copilot agents/skills (installed via APM) to demonstrate AI-assisted test authoring.

- Stack: Node.js (LTS), `@playwright/test`, TypeScript (strict, CommonJS), ESLint + typescript-eslint, oxfmt, `@faker-js/faker`.
- Only browser project: `Google Chrome` (`channel: 'chrome'`) — Chrome must be installed.
- Some spec files (`01-…`, `03-…`) contain commented-out exercises for workshop participants; do not uncomment or "fix" them unless asked.

## Repository Layout

- `tests/e2e/` — spec files (`testDir`), e.g. `setup-check.spec.ts`, `01-playwright-fundamentals.spec.ts`
- `pages/` — page objects, `<feature>.page.ts` (`LoginPage`, `HeaderPage`)
- `page-fixtures.ts` — registers page objects as fixtures and re-exports `expect`
- `playwright.config.ts` — config (baseURL, reporters, `testIdAttribute: 'test-data'`, expect timeout 15s)
- `.github/copilot-instructions.md` — detailed Playwright/POM rules (follow them)
- `.github/agents/`, `.github/skills/` — Copilot agents and skills
- `.github/workflows/playwright.yml` — CI
- `apm.yml`, `apm.lock.yaml`, `apm_modules/` — APM dependencies (`apm_modules/` is git-ignored; don't edit)

## Setup Commands

- Install dependencies: `npm install` (CI uses `npm ci`)
- Install browsers: `npx playwright install --with-deps`
- Verify setup: `npm run test:check`
- Install APM agent packages: `npm run apm:install`

## Development Workflow

- No dev server; tests run against the remote site. Override target with `BASE_URL`, e.g. `npm run test:dev:env` or `BASE_URL=https://host npx playwright test`.
- Record tests: `npm run codegen`
- Reports: `npm run show:report` (HTML reports in `playwright-report/`, blob in `blob-report/`; both git-ignored)
- Locator test id attribute is `test-data`, so `page.getByTestId('x')` matches `test-data="x"`.

## Testing Instructions

- All tests: `npm test` (`npx playwright test`)
- Single file: `npx playwright test tests/e2e/setup-check.spec.ts`
- By title: `npx playwright test -g "<title>"`; smoke: `npm run test:smoke` (tag tests with `@smoke`)
- Debug/UI: `npm run test:debug`, `npm run test:ui`, `npm run test:headed`
- Flakiness checks: `npm run test:ten:times`, `npm run test:shuffle`, `npm run test:single-worker`
- Tests run fully parallel; keep them independent (no shared state or order dependence).
- Add or update tests for behavior you change; run the affected spec until it passes consistently.

## Code Style

- Formatting via oxfmt (`.oxfmtrc.json`): semicolons, single quotes, 2 spaces, print width 100, trailing commas `es5`, LF.
- Run `npm run format` then `npm run lint` (`tsc --noEmit && eslint tests/**`). ESLint enforces `no-floating-promises` and `await-thenable` — always `await` Playwright calls.
- Specs: import `test`/`expect` from `../../page-fixtures` (not `@playwright/test`), group in `test.describe()`, use `beforeEach` for common navigation, title format `Feature - Specific scenario`, file name `<feature-or-page>.spec.ts`.
- Locators: prefer `getByRole` > `getByLabel` > `getByText` > `getByTestId`; avoid CSS/XPath.
- Assertions: web-first, auto-retrying (`toHaveText`, `toHaveURL`, `toHaveCount`, `toMatchAriaSnapshot` when structure matters). No `waitForTimeout` or raised timeouts.
- Page objects: class `<Feature>Page` in `pages/<feature>.page.ts`, constructor takes only `page: Page`, `readonly` typed `Locator` properties, async verb-first action methods. Register each new page object in `page-fixtures.ts` (interface + `base.extend`).
- Comments only where logic is non-obvious.

## Build and CI

- No build step. CI (`.github/workflows/playwright.yml`) runs on push to `main`, manual dispatch and a monthly schedule: `npm ci`, `npx playwright install --with-deps`, `npx playwright test --pass-with-no-tests`, then uploads the blob report. In CI: 2 retries, 1 worker, `test.only` forbidden.

## Pull Request Guidelines

Before committing, run `npm run format`, `npm run lint` and `npm test`; all must pass. Keep commit messages short and imperative (e.g. `add login page object`).

## Security Considerations

- Never commit credentials or tokens; use environment variables (e.g. `BASE_URL`). The demo credentials already in the workshop specs are for the public demo shop only.
- Don't commit generated output (`test-results/`, `playwright-report/`, `blob-report/`, `.playwright-mcp/`).
