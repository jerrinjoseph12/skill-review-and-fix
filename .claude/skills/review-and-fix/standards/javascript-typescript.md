# JavaScript / TypeScript Standards

Legend: `[C]` Critical, `[H]` High, `[M]` Medium, `[L]` Low (default severity; adjust per [../severity.md](../severity.md)).

## Authorities
- ECMAScript specification and the TypeScript Handbook
- OWASP Node.js Security, DOM-based XSS Prevention and Prototype Pollution Prevention cheat sheets
- typescript-eslint `recommended` / `strict` rule sets
- Framework docs: React, Angular, Vue, Express, NestJS, Next.js

## Tooling to look for and run
- `npm test`, `npx tsc --noEmit`, `npx eslint .`, `npx prettier --check .`, or the project's own scripts in `package.json`
- `npm audit` / `pnpm audit` / `yarn npm audit`

## Security
- `[C]` `eval`, `new Function`, or `setTimeout` / `setInterval` with a string argument, on data that isn't fully trusted.
- `[C]` `child_process.exec` or `execSync` with interpolated input. Use `execFile` or `spawn` with an argument array and no shell.
- `[C]` SQL or NoSQL built from template strings or concatenation. Also flag Mongo queries that take request objects directly (`{ $where }` or `$ne` operator injection).
- `[H]` Prototype pollution: a deep merge, `Object.assign` or a path-setter fed untrusted JSON without blocking `__proto__`, `constructor` and `prototype`.
- `[H]` DOM XSS: `innerHTML`, `insertAdjacentHTML`, `document.write`, `dangerouslySetInnerHTML` or `v-html` with untrusted data (see [html-css.md](html-css.md)).
- `[H]` JWTs: `jwt.decode` used for authorization, or `jwt.verify` without pinning `algorithms`.
- `[H]` Path traversal in `res.sendFile`, `fs.readFile` or a static file server using request paths.
- `[H]` Secrets in client bundles: `NEXT_PUBLIC_*`, `VITE_*` or `REACT_APP_*` variables holding private keys.
- `[H]` ReDoS: regexes with nested quantifiers, such as `(a+)+`, applied to user input.
- `[M]` Express without `helmet` or similar security headers when sibling services use it; `cors({ origin: true, credentials: true })`.

## Async correctness
- `[H]` Floating promises: a missing `await`, or a promise neither returned nor `.catch`-ed (`@typescript-eslint/no-floating-promises`).
- `[H]` `array.forEach(async ...)` doesn't wait for the callbacks. Use `for...of` or `Promise.all`.
- `[H]` `Promise.all` over an unbounded list hitting an API or database. Limit concurrency.
- `[M]` Unhandled rejections in event handlers or streams; `async` functions in Express before v5 not forwarding errors to `next`.

## Correctness
- `[H]` `.sort()` on numbers without a comparator. The default sort is lexicographic, so `[10, 2, 1]` becomes `[1, 10, 2]`.
- `[H]` Falsy checks that reject valid `0` or `""` (`if (!count)`, `x || default`). Use `??` or explicit checks.
- `[H]` `==` or `!=` coercion bugs, apart from the intentional `== null`.
- `[H]` Money in floating point. Use integer cents or a decimal library.
- `[H]` `Date` mistakes: months are 0-based; `new Date("2024-01-02")` is parsed as UTC but `new Date("2024/01/02")` as local time; timezone assumptions.
- `[M]` `JSON.parse` on external input without `try`; `parseInt` without a radix; `for...in` over arrays.
- `[M]` TypeScript: `any`, `as` casts, or non-null `!` used to hide real nullability; `@ts-ignore` without a reason. Prefer `@ts-expect-error` with a comment.

## React, Vue and Angular
- `[H]` State or props mutated in place, so no re-render happens or updates are stale.
- `[H]` `useEffect` or `useMemo` / `useCallback` with missing dependencies, which leads to stale closures. Effects with no cleanup for subscriptions, intervals or listeners leak memory.
- `[M]` Array index used as `key` in lists that reorder, insert or delete items.
- `[M]` Fetching in render or in effects without abort or race handling (a late response overwrites a newer one).
- `[M]` Angular subscriptions never unsubscribed. Use `takeUntilDestroyed` or the `async` pipe.

## Node performance
- `[H]` Synchronous `fs` or `crypto` calls, or CPU-heavy loops, in request handlers.
- `[M]` N+1 requests or queries inside loops; whole files or result sets loaded into memory where a stream would do.
- `[L]` Importing a whole library (`import _ from "lodash"`) in frontend code when the bundle size matters.

## Maintainability
- `[M]` New modules with no tests; reimplementing utilities the project already has.
- `[L]` `var` in new code, unused exports or variables, `console.log` left in production code (if the project has a logger).
