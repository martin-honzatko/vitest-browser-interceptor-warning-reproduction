# Vitest Browser Mode: dead `configureServer` hook warning

Minimal reproduction for Vitest 5.0.0 Browser Mode printing, once per run:

```
Plugin "vitest:mocks:interceptor" defines Vite-specific hooks (configureServer) in a plugin returned from applyToEnvironment. These hooks will be ignored.
```

## Steps

```sh
npm install
npx playwright install chromium   # only if Chromium 1.63.0 is not installed yet
npm test
```

## Actual

The warning above is printed at startup. The single test passes.

## Expected

No warning.

## Root cause (as far as I can tell)

- `@vitest/browser` adds `interceptorPlugin({ registerWebSocketEvents: false })` from `@vitest/mocker` to the browser
  contribution; `vitest:browser:loader.applyToEnvironment` returns it for the `client` environment.
- `@vitest/mocker`'s `interceptorPlugin` (`packages/mocker/src/node/interceptorPlugin.ts`) always defines
  `configureServer`; its body returns early when `registerWebSocketEvents === false`, so the hook is a no-op here.
- Vite 8 (`resolveEnvironmentPlugins`) warns for any `config` / `configEnvironment` / `configureServer` /
  `configResolved` hook on a plugin returned from `applyToEnvironment`.

Suggested fix: attach `configureServer` only when `registerWebSocketEvents !== false`, or strip it in the browser
contribution. Nothing is functionally broken; the warning is noise on every run.

Pinned: vitest 5.0.0, vite 8.3.0, @vitest/browser-playwright 5.0.0, playwright 1.63.0.
