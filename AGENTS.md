# star-wars-family Project Handoff

This file is the maintenance entry point for new sessions. Read it before changing the project.

## Current State

- Project path: `D:\Codex\Project\star-wars-family`
- Repository: `https://github.com/kevin198156/star-wars-family.git`
- Intended checkout: root of that path, branch `main`
- Last known stable commit: `2aaedc0` (`Restore stats history rendering`)
- App source: root `index.html`; there is no build system, package manifest, or test suite.
- Read-only Firebase snapshot on 2026-08-09: `girl=75`, `boy=85`, `614` transactions, latest transaction date `2026-07-26`. Re-check before relying on these values.

## Project Goal

This is a small mobile-first family reward app for two children. Each user can complete tasks for stars, spend stars on rewards, review recent history and charts, edit tasks/rewards, change profile settings, and export a CSV report. Parents can add manual score adjustments.

## Architecture and Data

- Everything is in `index.html`: markup, CSS, and browser JavaScript.
- External runtime dependencies are Chart.js and Firebase v9 modules loaded from CDNs.
- Firebase Realtime Database root: `star_wars_data_v2`.
- Local fallback key: `star_wars_local`.
- Main data shape: `users.girl`, `users.boy`, and `transactions`.
- A user has `name`, `theme`, `avatar`, `balance`, `tasks`, and `rewards`.
- A transaction has `id`, `timestamp`, `dateStr`, `user`, `type`, `name`, `amount`, and `note`.
- `transactions` may be an old array or a Firebase object containing numeric and push-generated keys. Keep `getTransactions()` compatible with both.

## Important Decisions

- Score actions append a transaction with Firebase `push()` and change the shared balance with `runTransaction()`. Preserve this concurrency-safe path.
- Do not restore whole-root Firebase writes for score actions. A stale page using `set(window.dbRef, db)` can erase another device's newer data.
- `saveDB()` still uses a whole-root `set()` for profile, task/reward, and avatar edits. Treat that as a known limitation; if those edits need concurrent safety, migrate them to path-scoped updates before changing behavior.
- The stats page renders history independently before Chart.js, uses fixed chart wrappers, and uses `maintainAspectRatio: false`. Keep chart failures from hiding history.
- The Firebase client config is intentionally in the static page. Treat Firebase Database Rules and the data path as the security boundary.

## Known Issues and Open Work

- The root checkout currently contains an untracked nested duplicate at `star-wars-family\star-wars-family\`, byte-identical to the root checkout at `2aaedc0`. Do not edit or stage the nested copy. Cleanup was intentionally not performed in this handoff.
- There is no automated test or build pipeline. Verification is currently script parsing, `git diff --check`, read-only Firebase checks, and manual browser testing.
- Score transaction creation and balance update are two client operations, so a network failure between them can still leave an orphan transaction or a temporary balance mismatch. Do not expand this flow without testing failure recovery.
- CDN availability is required for Chart.js and Firebase in normal browser use.

## Safe Session Startup

1. Work only from `D:\Codex\Project\star-wars-family`, never from the nested duplicate.
2. Read this file, then inspect `git status --short --branch`, the remote, and the current `index.html` before editing.
3. For data-related work, read the current Firebase schema and values first. Do not infer live data from this document.
4. Keep changes surgical. Preserve the single-file architecture unless the user explicitly asks for a larger migration.
5. Verify the intended user flow, JavaScript parsing, and `git diff --check` before handoff. Report skipped or unavailable browser tests.
6. Update this file only when architecture, safety boundaries, known issues, or unfinished work materially changes. Git history holds detailed historical context.

## Non-Destructive Boundaries

- Never delete or rewrite historical transactions without explicit user approval and a verified backup.
- Never overwrite `star_wars_data_v2` wholesale for a score operation or schema migration.
- Before changing Firebase data, use narrow paths and verify the exact target and resulting totals.
- Do not stage, commit, or push the nested duplicate directory.
