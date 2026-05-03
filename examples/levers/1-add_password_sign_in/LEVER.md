# Lever — `1-add_password_sign_in`

## TL;DR

- **Outcome:** `POST /auth/login` issues a JWT on valid credentials, 401s on wrong password (constant timing on missing user), 429s after 5 attempts in 1 min/IP. OAuth path untouched.
- **Coverage:** 4/4 verifiers passed (rerun green from a fresh shell). Verify mix: 4 shell (3 behavioral, 1 project-level).
- **Chain alignment:** Brief→Plan pass · Plan→Do pass · Do→Result pass · Result→Intent pass.
- **Act outcome:** 2 hints kept · 1 project edit (`AGENTS.md` +5/-0) · 0 skill edits · 1 follow-up `/lever-new` recommended (`enforce_route_middleware_lint`).
- **State:** `step: done` — proposals staged; lever finished.
- **Next:** human review of the diff and the staged `AGENTS.md` edit. User invokes `/lever-new enforce_route_middleware_lint` to spin up the follow-up lever.

## Brief

### Goal

Add `POST /auth/login` that accepts `{ email, password }` and returns `{ accessToken: <jwt> }` on valid credentials, or `401` otherwise. Reuse the existing JWT issuance helper. The OAuth login path stays untouched. Password reset, account lockout, 2FA, and any frontend work are out of scope for this task.

### Context

- bcrypt is already a runtime dependency — source: `package.json` (existing).
- JWT issuance helper exists at `src/auth/jwt.ts` — source: user message.
- Login attempts must be rate-limited — source: user message ("the API has a public IP").
- Test framework is Vitest — defaulted from `package.json` (`"test": "vitest"`).
- TypeScript strict mode is on — defaulted from `tsconfig.json`.

### Success sketch

- A client POSTs `{ email, password }` to `/auth/login`, gets back a JWT, and uses it to call existing protected routes.
- Wrong-password attempts return 401; repeated failures from the same IP eventually return 429.
- The existing OAuth login flow still works exactly as it did before.

### Open questions

- What rate-limit policy applies to the login route specifically? (Plan to investigate `src/middleware/rate.ts` and existing rate-limited routes for the project's convention.)

## Plan

### Goal

Add `POST /auth/login` (email + password → JWT) with rate limiting, reusing the existing JWT helper.

### Investigation findings

#### Codebase

- bcrypt is a runtime dependency, version `^5.1.1`. source: `package.json:14`
- JWT issuance helper: `issueAccessToken(userId: string): Promise<string>`, reads `JWT_SECRET` and `JWT_TTL` from env. source: `src/auth/jwt.ts:42`
- Rate-limit factory: `createLimiter({ windowMs, max, keyGenerator })`, returns Express middleware. Existing OAuth route uses `5 req/min/IP` (the project's "sensitive endpoint" convention). source: `src/middleware/rate.ts:18`, `src/routes/oauth.ts:7`
- Auth routes mounted at `/auth/*` in `src/app.ts:31`. source: `src/app.ts:31`
- `users` table has columns `id`, `email`, `password_hash`, `oauth_provider`. source: `src/db/schema.ts:8`
- No existing login route; OAuth callback is the only auth entry today. source: `git log -p src/routes/` (no `login.ts`)

#### External

- bcrypt v5 API: `compare(plain: string, hash: string): Promise<boolean>`. source: <https://www.npmjs.com/package/bcrypt> (retrieved 2026-05-02)

#### Resolved questions

- Rate-limit policy for login → 5 req/min/IP, matching the project's existing sensitive-endpoint convention; source: `src/routes/oauth.ts:7`.

### Constraints

- No new runtime dependencies (bcrypt + existing rate-limit factory cover it).
- bcrypt rounds = 12 (project convention from user-creation flow).
- JWT TTL = 1h (existing `JWT_TTL` env value).
- OAuth login flow must remain untouched.

### Risks & mitigations

- Timing attack on `compare` when user is missing → run `compare` against a fixed dummy hash on the user-not-found branch so timing doesn't leak account existence.
- Rate-limit keyed on `req.ip` — behind a proxy this might collapse to one key. Existing OAuth route relies on `trust proxy = 1` set in `app.ts` — same setup applies here.

### Deferred

- Account lockout / progressive delays (out of scope per brief).
- 2FA / WebAuthn (out of scope).
- Password-reset flow (out of scope).

## Do

### Summary

Added `POST /auth/login` to `src/routes/auth.ts`: looks up the user by email, runs `bcrypt.compare` against a fixed dummy hash on the not-found branch (constant timing), issues a JWT via the existing `issueAccessToken` helper, and is gated by a `rate_limit_login` middleware (5 req/min/IP) before the handler. Three Vitest tests cover happy-path JWT issuance, wrong-password 401, and the 429 after the 5-attempt window. All 4 criteria flipped to `passes: true`; type-check clean. Decisions captured in `lever.yaml.decisions` (per-route limiter choice, constant-timing user-not-found branch).

### Coverage

(Single human-readable mirror of `lever.yaml.criteria`. Do writes it on green; check edits it in place after the rerun. `lever.yaml` is the source of truth for `passes` / `output`.)

- `C1` — Login returns 200 + JWT on valid credentials. verify: `npx vitest run tests/auth/login.test.ts -t 'returns jwt on valid credentials'` → `1 passed`. **pass**.
- `C2` — Login returns 401 on wrong password. verify: `npx vitest run tests/auth/login.test.ts -t 'returns 401 on wrong password'` → `1 passed`. **pass**.
- `C3` — Login returns 429 after 5 failures in 1 min. verify: `npx vitest run tests/auth/login.test.ts -t 'rate-limits at 5 per minute'` → `1 passed`. **pass**.
- `C4` — typecheck clean. verify: `npm run typecheck` → `exit 0`. **pass**.

### Decisions

(Mirror of `lever.yaml.decisions`. One bullet per decision, verbatim.)

- Constant-timing user-not-found path — chose dummy-hash compare over early throw. Why brief required no account-existence leakage; the dummy compare is one extra bcrypt op only on the missing-user branch.
- Per-route limiter — applied rate_limit_login directly on the login route rather than as global middleware. Why keeps the OAuth path under its own (independent) limit and matches the convention of src/routes/oauth.ts:7.

### Deferred

- Account lockout / progressive delays — out of scope per brief.
- 2FA / WebAuthn — out of scope per brief.
- Password-reset flow — out of scope per brief; promote via `/lever-new password_reset`.

### Files touched

```text
 src/routes/auth.ts        | 47 ++++++++++++++++++++++++++++
 src/middleware/rate.ts    |  4 +++
 tests/auth/login.test.ts  | 64 ++++++++++++++++++++++++++++++++++++++
 3 files changed, 115 insertions(+)
```

## Check

### Summary

Chain holds end-to-end. §Brief was crisp (one open question, cleanly resolved in §Plan). §Plan's four criteria covered the spec without over-specifying mechanism — three behavioral tests for the observable behavior, one project-level type-check; no source-shape grep against the agent's own diff. `lever.yaml.criteria[i].events` shows TDD-style progression with one expected miss (limiter wiring on `C3.events 09:38`, recovered at `09:51`). §Do accurately reflects the events log; coverage and files-touched line up with `git diff --stat`. Re-running every `verify:` from a fresh shell, all four exit zero. Recommend: ship; 2 hints surfaced for act (lifted into `lever.yaml.hints`).

### Chain audit

- Brief → Plan: pass — rate-limit open question resolved with `src/routes/oauth.ts:7` citation.
- Plan → Do: pass — all four criteria driven in `lever.yaml.criteria[i].events`; no skipped rows; constant-timing decision recorded under `lever.yaml.decisions` stays inside §Plan §Constraints.
- Do → Result: pass — §Do §Coverage matches each `Cn.events` `pass` row; §Do §Files touched matches `git diff --stat`.
- Result → User intent: pass — OAuth flow not touched; deferred items match §Brief's non-goals.

### Hints

(Mirror of `lever.yaml.hints`. One bullet per promotable moment, verbatim.)

- `C3.events 09:38 fail` — limiter-wiring missed in first handler pass; "attach middleware" deserves an explicit sub-step in the do TDD pattern.
- `lever.yaml.decisions` — per-route-limiter choice cites `src/routes/oauth.ts:7` directly, but no project-level rule exists; future sensitive endpoints will re-discover (or re-miss) the convention.

### Diff summary

```text
 src/routes/auth.ts        | 47 ++++++++++++++++++++++++++++
 src/middleware/rate.ts    |  4 +++
 tests/auth/login.test.ts  | 64 ++++++++++++++++++++++++++++++++++
 3 files changed, 115 insertions(+)
```

## Act

### Summary

Walked the trail behind `lever.yaml.hints`. Two findings landed on different surfaces. (1) The repo treats per-route rate-limit middleware as a convention but only documents it in code (`src/routes/oauth.ts:7`); §Plan cited it but didn't lift it into §Constraints, so do had to re-derive it. Codified the rule under a new §Project conventions block in `AGENTS.md`. (2) The handler shipped before the limiter was wired (`C3.events 09:38` fail). The TDD pattern catches the failure on the first verifier run, but a static check could catch it at lint time before any test runs — that's its own piece of project work, not a rule. Recommended `/lever-new enforce_route_middleware_lint` to add an ESLint rule that flags sensitive route declarations missing their guard middleware. One project edit + one follow-up recommendation staged.

### Source trail

- `lever.yaml.hints` — 2 hints; read first.
- `lever.yaml.criteria[i].events` — `C3.events 09:38` fail (limiter not yet attached); recovered at `C3.events 09:51` after wiring.
- `lever.yaml.decisions` — per-route limiter choice with a citation to `src/routes/oauth.ts:7`, no project-level rule.
- §Do — Summary, Deferred.
- §Plan — §Investigation cited the convention but didn't lift it into §Constraints; re-read to confirm root cause.

### Hints reviewed

- `lever.yaml.hints[0]` (limiter-wiring missed; "attach middleware" needs sub-step) — kept; target: follow-up `/lever-new enforce_route_middleware_lint`. Reason: the gap is in the project's lint coverage, not in a rule the next plan should remember.
- `lever.yaml.hints[1]` (sensitive-endpoint rate-limit convention only lives in code) — kept; target: `AGENTS.md`.

### Findings

#### Document the sensitive-endpoint rate-limit convention

- Symptom: `C3.events 09:38` fail shows handler landed before `rate_limit_login` was wired; `lever.yaml.decisions` records the per-route limiter choice but cites only `src/routes/oauth.ts:7`, no project-level rule.
- Root cause: §Plan §Investigation found the convention at `src/routes/oauth.ts:7` but didn't lift it into §Constraints, so do treated rate-limit wiring as derivable rather than required.
- Catches: any future sensitive endpoint — e.g., `/lever-new password_reset` (will need its own `rate_limit_password_reset`); a future `/lever-new two_factor` endpoint with the same pattern. Without the rule, each future plan re-discovers (or re-misses) the convention.
- Target: `AGENTS.md` §Project conventions — chosen because the rule is cross-task and project-owned. Rejected `src/routes/README.md` (none exists; `AGENTS.md` guidance discourages new top-level docs from inside skills).
- Edit: `AGENTS.md` +5/-0 — added new `## Project conventions` section with one bullet: "Sensitive endpoints (login, OAuth, password-reset, 2FA) each declare their own `rate_limit_<endpoint> = createLimiter(...)` and wire it inline at route declaration: `router.post(path, rate_limit_<endpoint>, handler)`. Don't rely on global middleware for these routes."

#### Catch unwired guard middleware at lint time

- Symptom: `C3.events 09:38` fail — handler shipped on first pass without the limiter wired; the failure was caught only by `C3`'s integration-style test, one full vitest run later.
- Root cause: the project has no static check that asserts a sensitive-route declaration includes its guard middleware. The convention lives in code (`src/routes/oauth.ts:7`) and now in `AGENTS.md`, but nothing fails the build when a new sensitive route is added without its guard. The TDD pattern catches the regression eventually; a lint rule catches it before any test runs.
- Catches: future sensitive endpoints (password_reset, 2FA, admin actions) — same shape, same risk, same one-iteration recovery cost. A lint rule converts each of those one-iteration costs to zero.
- Target: follow-up `/lever-new enforce_route_middleware_lint` — chosen because the gap is in the project's lint coverage, not in a rule the next plan should memorize. Adding an ESLint rule (e.g., `no-unguarded-sensitive-route`) is its own piece of project work, with its own brief, plan, criteria, and verifier. Rejected an `AGENTS.md` rule like "remember to wire the middleware" — that fold leans on agent memory; a lint rule fails the build.
- Edit / recommendation: `/lever-new enforce_route_middleware_lint (add ESLint rule that flags sensitive route declarations missing their guard middleware)`.

### Alternatives considered

- Adding the sensitive-endpoint rule to `src/routes/README.md` — rejected; no such file exists, and `AGENTS.md` guidance discourages new top-level docs from inside skills.
- Editing `skills/lever/references/do.md` to add a "wire middleware first" TDD sub-step — not applicable in this project (skills installed via plugin, not vendored in tree). Even if vendored, the lint-rule route catches more cases more reliably than a procedure note.
- Folding the lint-rule recommendation into `AGENTS.md` as a remember-to-wire bullet — rejected; a build failure beats an agent reminder.

### Files touched

```text
 AGENTS.md | 5 +++++
 1 file changed, 5 insertions(+)
```
