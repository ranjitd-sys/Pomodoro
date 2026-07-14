# Google Auth Refactor — July 14, 2026

## Summary
Converted the DeepTrack Google integration from a Tasks-scoped auth flow to a generic, reusable Google auth module — then chased down two downstream bugs the rename surfaced.

## 1. Stripped Tasks-specific scope from the auth module
- Narrowed the OAuth scope from `https://www.googleapis.com/auth/tasks openid email profile` down to just `openid email profile` — no more Tasks API access requested.
- Removed `UnauthorizedUserError` and the `STATIC_USERS` whitelist check (that gating logic was specific to the Tasks-project use case).
- Renamed `GoogleTasksAuth` → `GoogleAuth`. Its `connect()` now just returns `{ accessToken, expiresAt, email, name, picture }` — no `project` lookup, no unauthorized-user failure path.
- `GoogleTokenClient` and `GoogleUserInfo` were already task-agnostic, so left unchanged.

## 2. Fixed the consuming `connectGoogleAndRedirect` flow
- Renamed `GoogleTasksAuth` references to `GoogleAuth` to match the module rename (both the `yield*` pull and the `Layer.provide` call).
- Fixed a pre-existing bug: the code was resolving `GoogleTokenClient` from context but calling `.connect()` on it — that method only exists on the higher-level auth service. Now correctly resolves `GoogleAuth`.
- Renamed localStorage keys from `google_tasks_access_token` / `google_tasks_token_expiry` to `google_access_token` / `google_token_expiry` since the flow is no longer Tasks-scoped.

## 3. Diagnosed a TypeScript structural-typing error
- Error: `Effect<...>` argument not assignable due to missing `[TypeId]`/`[NodeInspectSymbol]` — classic symptom of **two resolved copies of the `effect` package** (one in the app, one nested under `@de-purchase/auth`'s own `node_modules`).
- Recommended fixes:
  - Make `effect` a `peerDependency` of `@de-purchase/auth` instead of a regular dependency.
  - Or force a single version workspace-wide via `overrides` (npm) / `pnpm.overrides` / `resolutions` (yarn).
- No code change needed once versions are deduped — `connect-google.ts` was correct as written.

## 4. Fixed `ReferenceError: google is not defined` at runtime
- Root cause: `declare const google: any` is a TypeScript-only declaration — it doesn't load the actual Google Identity Services (GIS) script (`https://accounts.google.com/gsi/client`) onto the page.
- Fix: added a `loadGis()` helper inside the auth module that:
  - Checks if `google.accounts.oauth2` is already available.
  - Otherwise injects the GIS `<script>` tag (or reuses an existing one already on the page) and waits for its `load` event.
  - De-dupes concurrent calls via a shared load promise.
- `requestAccessToken` now awaits `loadGis()` before touching `window.google`, so the flow is self-sufficient regardless of whether the script is preloaded in `index.html`.

## Files touched
- `google-auth.ts` — scope narrowing, `GoogleAuth` rename, GIS script loader
- `connect-google.ts` — `GoogleAuth` rename, fixed service-resolution bug, localStorage key rename

## Open item
- `error_callback` in GIS doesn't fire for all failure modes (e.g. a silently blocked popup) — flagged as a separate issue to look at later, not yet fixed.