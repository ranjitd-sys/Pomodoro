# Daily Task Log — Google OAuth Login Debugging

**Date:** July 12, 2026
**Project:** `de-workflow` (DeepTrack — `@de-workflow/web`)
**Environment:** AKS deployment, `workflow.deepecom.com`

---

## Summary

Google OAuth login worked fine on `localhost:3000` but failed in the deployed AKS environment. Debugged through three distinct failure stages, each with a different root cause.

---

## Issue 1: Popup closes immediately (`popup_closed`)

**Symptom**

```
Google auth error: _.Cd {message: 'Popup window closed', type: 'popup_closed'}
```

Login worked locally, but on the deployed site the Google sign-in popup opened and immediately reported as closed, even without user interaction.

**Root cause**

`web-nginx.conf` was setting a strict Cross-Origin-Opener-Policy header:

```nginx
add_header Cross-Origin-Opener-Policy "same-origin" always;
```

This fully isolates the page's window from any popup it opens. Google's GSI popup couldn't `postMessage` back to the opener or be read/closed correctly, so the browser tore it down — surfaced by GSI as a false "popup closed" error.

**Fix**

Changed COOP to allow popup communication, in both the top-level `server` block and the `/assets/*.js` location block:

```nginx
add_header Cross-Origin-Opener-Policy "same-origin-allow-popups" always;
```

Also temporarily removed `Cross-Origin-Embedder-Policy: require-corp` to eliminate it as a confounding variable (not currently needed unless SharedArrayBuffer/WASM threading is added later).

**File changed:** `docker/web-nginx.conf`

---

## Issue 2: `Error 401: invalid_client` — "OAuth client was not found"

**Symptom**

After fixing COOP, the popup reached Google but returned:

```
Access blocked: Authorization Error
Error 401: invalid_client
```

**Root cause**

The frontend's Google OAuth Client ID env var (e.g. `VITE_GOOGLE_CLIENT_ID`) was not being passed into the Docker build. The `Dockerfile` only defined:

```dockerfile
ARG VITE_SYNC_BASE_URL=http://web-backend-svc:3000
ENV VITE_SYNC_BASE_URL=$VITE_SYNC_BASE_URL
```

With no client ID build arg, Vite baked in `undefined` for the client ID at build time — this only worked locally because a `.env` file supplied it there, but that `.env` wasn't part of the Docker build context / CI pipeline.

**Fix**

Added the client ID as a build arg/env var in the Dockerfile and passed it explicitly during the image build (CI/CD or `docker build --build-arg`):

```dockerfile
ARG VITE_GOOGLE_CLIENT_ID
ENV VITE_GOOGLE_CLIENT_ID=$VITE_GOOGLE_CLIENT_ID
```

Verified the value matches exactly the "Web client 1" OAuth Client ID registered in Google Cloud Console.

**File changed:** `Dockerfile` (+ CI/CD build args)

---

## Issue 3 (in progress): `Unauthorized Google account: samruddhik@deepecom.com`

**Symptom**

Login now completes successfully with Google (token returned), but the app itself rejects the account:

```
Unauthorized Google account: samruddhik@deepecom.com
```

**Root cause (suspected)**

Application-level allowlist or domain check in the auth callback handler rejecting this specific account — not an OAuth/Google config issue. Need to locate the exact check (likely a hardcoded email list or domain-suffix match) that's failing for this account.

**Next step**

- Grep backend/frontend for the string `"Unauthorized Google account"` to find the exact validation logic.
- Confirm whether `samruddhik@deepecom.com` should be added to an allowlist, or whether there's a domain-matching bug.

**Status:** Not yet resolved — carrying over to next session.

---

## Secondary / non-blocking observation

```
POST https://20.87.39.192.nip.io/v1/traces  net::ERR_CERT_AUTHORITY_INVALID
```

OpenTelemetry traces endpoint failing due to an invalid/self-signed cert on that IP. Unrelated to auth flow — deferred, revisit if tracing data is needed.

---

## Files touched today
- `docker/web-nginx.conf`
- `Dockerfile`

## Carried over to next session
- Locate and fix the backend email/domain allowlist check causing "Unauthorized Google account" for `samruddhik@deepecom.com`.

---

## Task 2: Added CSV Export feature

**Project:** DeepTrack (`@de-workflow/web`) — daily task tracking

**What was done**

Added the ability to export daily tasks as a CSV file from the task tracking view, alongside the existing CSV import support.

- Implemented export logic in `daily-tasks.ts` (materialized view layer), reusing the same row-shape used for import so export/import stay symmetric.
- Mapped SQLite EventJournal-backed task records to flat CSV rows (date, task title, status, time spent, notes, etc.).
- Wired up a UI action (export button) to trigger a browser download of the generated CSV, scoped to the currently selected date range/grouping.
- Verified round-trip: exported CSV re-imports cleanly without data loss or type mismatches.

**Status:** Done ✅

**Follow-up ideas (not started)**
- Add column selection / filtering before export.
- Support export of a custom date range instead of just the current view.

---

## Task 3: Read up on `ax` (Necmttn) — agent memory/observability tool

**Context**

Investigative reading, connected to the pattern of repeatedly re-debugging the same infra issues across sessions (AKS, PgBouncer, DNS, shard mismatches) — looking at whether a tool like `ax` could reduce that repetition by giving agents structured memory of past sessions.

**What `ax` is**

`ax` (github.com/Necmttn/ax) bills itself as an "agent experience layer" — observability + memory for AI coding agents (Claude Code, Codex). Key points:

- **Local-first, typed, yours** — not a hosted service; data stays local.
- Core idea: instead of a giant rolling context window (expensive, slow, lossy) or vague vector-based retrieval (no structure, no grounding in real events), `ax` builds a **typed graph of evidence** from the agent's own logs.
- Provides **skill triage** — tracks which installed skills actually get used, which never fire, and which correlate with sessions where the agent got stuck.
- Ships as:
  - `npx skills add Necmttn/ax` — installs agent skills (setup, retro, extract-workflow, dojo, etc.)
  - `claude mcp add ax -- ax mcp` — registers an MCP server exposing read-only graph queries as tools
- Recommended agent loop:
  1. `ax project context --json` before starting work — surfaces stack info, recent friction points, verification commands.
  2. Do the work.
  3. `ax project verify --json` before reporting done.
- For Codex, `ax mcp` is added to `~/.codex/config.toml` instead.
- Sessions can be shared via `ax share <session-id>`, producing a link like `ax.necmttn.com/s/<owner>/<session>`.

**Relevance to current workflow**

The "recent friction" and "verification commands" surfaced by `ax project context` map closely to the kind of repeated debugging pattern seen this week (COOP headers, env var propagation, allowlist checks) — worth evaluating whether wiring this into the DeepTrack/de-workflow dev loop would cut down re-discovery time on recurring AKS/infra issues.

**Status:** Reading/investigation only — not yet installed or trialed.

**Next step (if pursued)**
- Try `npx skills add Necmttn/ax` on a local branch and see what `ax project context --json` surfaces for the `de-workflow` repo.
- Compare against manually maintained debugging notes (like this doc) to see if it captures the same friction points automatically.