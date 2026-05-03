# Examples

Two worked examples of the lever chain. Both describe fictional projects — the artifacts are illustrative. The path layout matches what `/lever-init` creates in your own project (`.agents/levers/<id>-<slug>/`).

## `levers/1-add_password_sign_in/` — happy path, end-to-end

Add `POST /auth/login` accepting email + password and returning a JWT, rate-limited, reusing the existing JWT helper. The chain runs cleanly: brief → plan → do → check → act → done. Two hints surface during check; act lands one as an `AGENTS.md` edit and surfaces a follow-up `/lever-new` recommendation for an ESLint rule.

- `lever.yaml` — machine state with stable IDs `C1`–`C4`, per-criterion `events` (the sole do-loop audit trail; `C3.events 09:38` fail recovers at `C3.events 09:51`), `budget`, `decisions`, `hints`. Validated against `lever.schema.json` at the repo root.
- `LEVER.md` — single narrative artifact. Sections appear when their step ran:
  - `## TL;DR` — rewritten on every step; the latest version is what humans skim.
  - `## Brief` — captured intent (Goal, Context, Success sketch, Open questions). Written by `/lever-new`.
  - `## Plan` — investigation + constraints (Investigation findings, Constraints, Risks & mitigations, Deferred). Written by the plan step (run via `/lever 1`); criteria themselves live in `lever.yaml.criteria` — there is no §Acceptance checklist mirror.
  - `## Do` — final state + per-criterion coverage (Summary, Coverage, Deferred, Files touched). §Coverage is the single human-readable mirror of `lever.yaml.criteria`; check edits it in place after the rerun. Decisions live in `lever.yaml.decisions` only — LEVER.md does not mirror them. The per-event audit lives in `lever.yaml.criteria[i].events`; render it via `/lever-status 1` (the trace appears inline whenever any criterion has events).
  - `## Check` — chain audit + diff summary (Summary, Chain audit, Diff summary). Written by the check step; check re-runs each `verify`, overwrites `lever.yaml.criteria[i].passes`/`output`, and edits §Do §Coverage in place. Hints live in `lever.yaml.hints` only — LEVER.md does not mirror them.
  - `## Act` — proposals lifted from the trail. Two findings: one project-side edit to `AGENTS.md` (sensitive-endpoint rate-limit convention), one follow-up `/lever-new enforce_route_middleware_lint` recommendation (codify the rule as an ESLint check rather than relying on agent memory).

## `levers/2-fix_dropdown_ios_close/` — check routes back to do

Fix a dropdown menu that doesn't close on iOS Safari when tapping outside. The do loop turns all four criteria green — but check's rerun catches a regression on `C2` (the close-button path) caused by the `pointerdown` listener racing with the close-button's own click handler. Check sets `step: do` and updates §Do §Coverage in place to reflect the regressed `C2`. The lever stops at `step: do`, waiting for the next `/lever 2` to resume the loop.

This example illustrates:

- An execution gap (criteria are correct; the implementation just doesn't satisfy `C2`) — distinct from a spec gap, which would route to plan.
- Check editing §Do §Coverage in place rather than appending a parallel §Verifier rerun mirror.
- A lever in a non-terminal `step: do` state — `/lever-status 2` would show `step: do` with the trace inline.
