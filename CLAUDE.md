# IGCSE Planner — working rules

Offline-first PWA (React + Vite + TypeScript) for IGCSE / O Level / A Level revision: exam
countdown, past-paper tracker, study plan, flashcards. All user data lives in the browser
(`localStorage`, keys prefixed `igcse-planner:`); there is no backend.

## Repo state — read first
This repo currently holds only **build output** (`index.html`, `sw.js`, manifest, icons).
`index.html` references `/assets/*.js` and `sw.js` references `src/theme/engine.ts` and
`vite.config.ts`, none of which are committed. Do not hand-edit `sw.js` or hashed asset names:
they are generated. If a task needs source changes and `src/` is absent, stop and ask for it.

## Commands (once source is present)
- `npm ci` · `npm run dev` · `npm run build` · `npm run preview`
- `npm test` (Vitest) · `npx tsc --noEmit` · `npm run lint`
- Check `package.json` scripts before assuming any of the above exist.

## How to work
1. **Explore before editing.** `rg` for text, `npx @ast-grep/cli run -p '<pattern>' -l ts` for
   structure. (`sg` on Linux is often the shadow-utils binary, not ast-grep.)
2. **TDD, every behaviour change.** Write a failing Vitest test, run it and confirm it fails for the
   right reason, then write the minimum code to pass, then refactor. Bug fixes start with a
   regression test that reproduces the bug.
3. **Verify UI in a real browser.** For anything visible, run `npm run build && npm run preview`
   and check it with the `playwright` MCP (navigate, snapshot, console messages). Also test
   offline (service worker) and the light and dark themes.
4. **Finish clean.** Typecheck, lint and tests must all pass before you commit. Report failures
   verbatim; never claim green without running the checks.

## Code style
- Small, single-purpose modules; pure logic (dates, grade thresholds, scheduling) lives
  outside React components so it can be unit-tested without a DOM.
- Strict TypeScript: no `any`, no non-null `!` without a comment saying why it's safe.
- Match the existing patterns for naming, file layout and comment density in neighbouring files.
- Storage: add new keys under the `igcse-planner:` prefix and migrate old shapes, so existing
  users' data keeps loading.

## Dependencies — no hallucinations
- Never import a package that isn't in `package.json`. Before adding one, confirm it exists
  and is maintained (`npm view <pkg> version`) and ask whether it is worth its weight; prefer
  the platform or existing deps.
- Look up APIs through `context7` (`resolve-library-id` → `query-docs`) or `fetch` on
  official docs instead of relying on memory, especially for React, Vite, Workbox/SW APIs.
- Pin exact versions and commit the lockfile.

## Don'ts
- Don't change the cache name or update flow in the service worker without an explicit ask:
  breaking it strands installed users on an old build.
- Don't add analytics, trackers or network calls that send user data anywhere.
