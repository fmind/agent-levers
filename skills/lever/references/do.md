Reference for the **do** step. Opened by `skills/lever/SKILL.md` when `lever.yaml.step: do`.

Drive `lever.yaml.criteria` until every `passes: true`, the budget is exhausted, or a hard stop fires. Always set `lever.yaml.step` (and `pause` when halting) before closing.

Resume cleanly: prior `criteria[i].events` + `passes` are the source of truth. Loop counters (`iter`, `streak`, `t0`) are per-invocation — each re-entry starts a fresh budget window.

## 1. Pre-flight

Reads:

- `lever.yaml` — `criteria` (each `id`, `description`, `verify`, `seed`, `passes`, `events`), `budget`, `step` (must be `do`), `pause` (re-entry).
- LEVER.md — §Plan (Investigation findings, Constraints, Risks); §Brief (Goal, Context).

If any `criteria[i].events` is non-empty, this is a resume — pick up at the first `passes: false` criterion (declared `id` order; honor `seed:`).

If `step != do`, set `pause: blocked` with a chat sentence naming the expected state.

If `pause` was set on entry, the user has confirmed/rejected a visual/manual criterion (§3) or fixed a blocker. Read the latest message; act per the §3 contract; clear `pause` on resolution.

## 2. The loop

```text
iter = 0; streak = 0; t0 = now()

loop:
  if iter   >= budget.max_iterations:    pause = blocked  # max_iterations
  if now()-t0 >= budget.max_minutes*60:  pause = blocked  # max_minutes
  if streak >= budget.max_streak:        pause = blocked  # max_streak

  pick first `passes: false` criterion C (id order; honor seed)
  iter += 1
  drive C within §Plan §Constraints — for type:shell, default TDD:
    write/extend failing test → confirm it fails for the right reason → make source change → re-run test + relevant lint

  run C.verify per its `type` (§3)
  if pass: flip C.passes; set C.output to trimmed verifier line; append `verb: pass` event; cap (§4); streak = 0
  else:    append `verb: fail` event; cap (§4); streak += 1
  rewrite LEVER.md §TL;DR
  if all criteria pass: break
```

One criterion at a time — don't fan out. Park out-of-scope ideas in §Do §Deferred.

**Don't introduce verification beyond what each `verify:` requires.** If the verifier is a grep, don't add a build or extra test suite.

## 3. Verify type dispatch

- **`type: shell`** — execute `verify.cmd` from the workspace root. Pass on exit 0; fail otherwise. Capture trimmed stdout/stderr in the event row's `output`; record rationale in `note` when non-obvious.
- **`type: visual`** — produce or refresh the screenshot at `verify.screenshot`. Halt with `pause: ask`, asking the user to confirm/reject `<Cn>` against `verify.expects`. **Re-entry:** user replies `confirm C<n>` / `reject C<n>` (or unambiguous prose) and re-invokes `/lever <id>`. On confirm, flip `passes: true`, clear `pause`, resume. On reject, treat as a verifier failure (`streak += 1`); continue. **Never flip `passes` based on your own judgment of the screenshot.**
- **`type: manual`** — execute `verify.instructions` step-by-step; capture evidence in the event row. Halt with `pause: ask`; same re-entry contract as visual.

Visual and manual intentionally interrupt the auto-loop.

## 4. Trace events (machine-only, append-then-cap)

Events live in `lever.yaml.criteria[i].events`. Verbs: `pass | fail | fold` (see `lever.schema.json`). Lever-wide non-obvious calls go in top-level `decisions[]`, not events.

**Cap rule.** Each criterion's `events` is capped at the **last 10 rows**. After every append, if `len(events) > 10`:

1. Drop rows from the head until `len(events) == 10`.
2. If the kept window's head is anything other than a `fold`, replace it with one new `fold` row whose `at` is `<oldest-dropped-at>-<head-of-kept-at>`, `verb: fold`, `note: "<N> earlier events dropped"`. If the head is already a `fold`, extend its `at` range and update its `note` to the new total.

Result: each criterion holds at most one head `fold` plus the latest 9 rows.

Rewrite §TL;DR after every loop event — TL;DR is the only LEVER.md section do edits during the loop.

## 5. Final sweep + write the artifacts

When the loop turns green, clean re-run from a fresh shell: run every `verify` per its `type`; on regression, retry once (flake vs. real failure); if it still fails, resume §2. For behavioural / UI changes covered only by `type: shell`, attach a screenshot or log excerpt — type-checks alone don't prove a feature works.

`lever.yaml` — for each criterion that ran: set `passes`, `output` (one trimmed line — prefer the verifier's own summary like `1 passed`; fall back to `exit 0`), and `events` (capped). Lift non-obvious calls into top-level `decisions[]`. On loop success: `step: check`, no `pause`. On halt: `step: do`, set `pause`. Bump `updated_at`. `description`, `verify`, `seed`, `budget` are immutable.

`LEVER.md` — replace §TL;DR with the block below; append §Do below the last existing section using `templates/do.md` (mirrors `criteria` in §Coverage and `decisions` in §Decisions).

§TL;DR after do:

```markdown
## TL;DR

- **Loop:** <iter>/<max_iterations> iters · <minutes>/<max_minutes> min · streak <streak>/<max_streak>.
- **Coverage:** <pass>/<total> criteria green · last attempted: `<Cn>`.
- **What works:** <one sentence — the deliverable in plain English.>
- **State:** `step: check` — every criterion passes.
- **Next:** `/lever <id>` to check.
```

## 6. Hand off

See `skills/lever/SKILL.md` §3 for the shared frame. Do's outcomes:

- **Advance** — every `passes: true`. `step: check`, no `pause`.
- **Pause: ask** — `type: visual` or `type: manual` gate per §3, OR missing info no codebase reading can supply, OR an irreversible architectural choice the plan didn't decide, OR a destructive op the plan didn't sanction (`rm -rf`, force-push, `DROP TABLE` — name it + why).
- **Pause: blocked** — auth/credential failure, OR budget exhausted (name the limit + pass count, e.g., "Budget exhausted: max_streak; 5/6 passing").

**Invariant:** every flipped `passes: true` is backed by a real verifier run captured in `events`; `decisions` records every non-obvious call (mirrored in §Do §Decisions); §Do §Files touched matches `git diff --stat`. Budget exhaustion is the primary stall signal — keep going through flaky tests, lint warnings, doc typos.
