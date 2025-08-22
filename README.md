# SauceDemo Playwright E2E Suite

A production-style Playwright TypeScript test suite for https://www.saucedemo.com with
- Page Object Model
- Fixtures to avoid repetitive login
- Utilities for pricing and parsing
- Tests: auth, sorting, full checkout (with price verification), and advanced scenarios for `problem_user` and `performance_glitch_user`.

## Quick setup
1. Clone repo
2. `npm ci`
3. `npx playwright install`
4. Run tests: `npm test`
5. See report: `npx playwright show-report`

## Project architecture
- `src/pages/*` — Page Objects. Encapsulate selectors and page-level actions so tests remain readable and stable.
- `tests/fixtures.ts` — Playwright fixtures. Exposes `loggedInPage` (standard_user) to avoid repeated login steps.
- `tests/*.spec.ts` — Tests (spec files) that use POM and fixtures.
- `src/utils/price.ts` — Small utility to parse and format prices.
- `playwright.config.ts` — CI-friendly configuration with sensible timeouts and retries.

### Why this structure — "Five-year" rationale
- Clear separation of concerns (POM keeps selectors and flows out of tests).
- Fixtures centralize state creation and teardown (e.g., authenticated pages) and make the tests both faster and more expressive.
- Utilities are isolated for easier unit-tests and reuse.
- Tests focus on *what* is tested, not *how* — easier for a future teammate to extend.

## Strategic decisions (Advanced scenarios)
### problem_user (wrong images)
- **Ideal approach**: Visual regression testing (pixel-diff) against approved baselines in CI (Win/Chromium). Store golden images in repo or a remote artifact store and use Playwright’s screenshot + pixel-diff tool (e.g., `pixelmatch`) in CI.
- **Implemented functional check**: Since baselines are not available here, the implemented test collects the `product name -> image src` mapping for `standard_user` and compares it with `problem_user`. A mismatch indicates images are swapped/incorrect. This is reliable and non-flaky and demonstrates the detection capability without heavy image storage.

### performance_glitch_user (lag)
- **Approach**: Never use brittle sleeps. Use:
  - Playwright's built-in waiting (`expect(locator).toBeVisible`, `waitForLoadState('networkidle')`) with increased timeouts for this scenario.
  - Where appropriate, wait for UI state changes (e.g., button enabled, cart item visible).
- **Why**: Robust to variable network / rendering delays. Avoids fragile fixed `sleep()` statements.

## Notes & improvements
- Add visual regression (Golden images) in CI for pixel-level verification of UI (especially recommended for the `problem_user` scenario).
- Extract commonly used flows (e.g., `addMultipleProducts`) into higher-level helpers if more tests require them.
- Consider parallel test execution strategies and per-worker storageState caching to speed up runs in CI.

## Running locally
- `npm ci`
- `npx playwright install`
- `npm test`
- `npx playwright show-report`
