Reference for the **plan** step. Opened by `skills/lever/SKILL.md` when `lever.yaml.step: plan`.

Convert §Brief into the envelope `do` loops inside: findings (the map), constraints (the walls), an acceptance checklist (the oracle), an explicit budget (the leash). Resolve every §Brief §Open question, cite every load-bearing fact, map every §Success-sketch signal to ≥ 1 criterion.

## 1. Pre-flight

Reads (in order):

- `lever.yaml` — `step` (must be `plan`), `pause` (re-entry), `criteria` (empty after `/lever-new`; populated on re-entry from check).
- LEVER.md — §TL;DR, §Brief. On re-entry from check: also §Plan, §Do, §Check, and `criteria[i].events`.
- `AGENTS.md` §Conventions.

If `step != plan` or §Brief is missing, set `pause: blocked` with a chat sentence naming the expected state.

If `pause` was set on entry, the user has answered or unblocked. Read the latest message; clear `pause` on resolution; otherwise restate the question and keep `pause` set.

**Re-entry from check.** §Do + §Check + `events` pinpoint a spec gap. Plan may revise §Acceptance criteria, §Constraints, §Investigation findings — existing criterion `id`s are immutable, so revisions add new `Cn` rows or replace failing ones in place (don't reorder, don't reuse a dropped id). **Don't silently widen scope past §Brief §Goal** — escalate via `pause: ask` (§5) if the goal itself needs to expand.

## 2. Investigate

Two streams. Push each until it stops yielding load-bearing facts — shallow investigation here surfaces as do-time surprises later.

**Codebase.** Grep / Glob the modules, tests, configs likely relevant. Read each file you'd reasonably touch. Read tests too — they encode invariants the source alone won't show. `git log -p <file>` when recent changes might explain the current shape. Note what's NOT there as carefully as what is.

**External.** When the task touches a library, framework, API, protocol, or domain concept, fetch authoritative sources yourself. Pin exact versions in use (`package.json`, `pyproject.toml`, `go.mod`, lockfiles) before reading docs — version drift is the most common do-time surprise. Capture each fact with URL + retrieval date; inline a short reusable digest when it saves do a fetch.

Resolve every §Brief §Open question; capture answers under §Plan §Investigation findings → Resolved questions. Escalate via `pause: ask` only if investigation genuinely can't resolve one (irreducible business rule, external SLA, missing fixture).

## 3. Distill the operating envelope

Three outputs, each grounded in §2:

- **Findings.** Load-bearing facts do shouldn't re-derive — each carries a citation (`file:line`, commit hash, URL + retrieval date). Subdivide: Codebase / External / Resolved open questions.
- **Constraints.** Invariants do must not violate (public API surfaces, approved/forbidden dependencies, schema rules, security posture, version pins). Risks are _worry_; constraints are _walls_.
- **Acceptance criteria.** 5–10 mechanically verifiable rows in `lever.yaml.criteria` (validated against `lever.schema.json`). Each: stable `id: C<n>` (never reused, never reordered); one-line mechanical `description:` (no "works", "robust", "fast"); typed `verify:` block (§4); `passes: false`; optional `seed:` first-failing-test pointer.

**Budget.** Set `lever.yaml.budget` (defaults `max_iterations: 20`, `max_minutes: 60`, `max_streak: 5`); override per-task when justified. When any limit hits, do stops with `pause: blocked`.

## 4. Verifier hygiene (absolute)

Test the **behavior**, not the agent's past actions. **Anything that re-reads the agent's own diff to assert authorship is bookkeeping, not verification.** If a behavioral test is genuinely infeasible, use `type: visual` or `type: manual`.

- ✅ Behavioral test (TDD), project-level standing check (`npm run typecheck`, `ruff check .`, `eslint .`), benchmark gate, runtime probe.
- ❌ Source-shape grep against the agent's own diff. ❌ File-existence check on a file the agent just created.

Verify types (see `lever.schema.json`):

- `type: shell` — `cmd:`. Passes when `cmd` exits 0. Default for code-level checks.
- `type: visual` — `screenshot:` + `expects:` (one-line rubric). Do attaches the screenshot; passing requires explicit human confirmation via a `pause: ask` cycle.
- `type: manual` — `instructions:` (multi-line). Do runs the instructions, captures evidence, gates on explicit human confirmation.

Mixing types in one plan is fine.

## 5. Write the artifacts

`lever.yaml` — read the seeded file, add `budget`, populate `criteria`. On success: `step: do`, no `pause`. On pause: keep `step: plan`, set `pause: ask | blocked`. Bump `updated_at`. Validated against `lever.schema.json`.

`LEVER.md` — replace §TL;DR with the block below; append §Plan below the last existing section using `templates/plan.md`.

§TL;DR after plan:

```markdown
## TL;DR

- **Goal:** <one sentence — the target deliverable.>
- **Acceptance:** <count> criteria, ids `C1`–`C<n>`. Verify mix: <shell/visual/manual counts>.
- **Budget:** `max_iterations: <n> · max_minutes: <n> · max_streak: <n>`.
- **State:** `step: do` — plan written; criteria defined.
- **Next:** `/lever <id>` to do.
```

## 6. Hand off

See `skills/lever/SKILL.md` §3 for the shared frame. Plan's outcomes:

- **Advance** — investigation complete, criteria populated. `step: do`, no `pause`.
- **Pause: ask** — irreducible question survives investigation (business rule, external SLA, irreversible architectural choice). State the question; list options when there's a fork.
- **Pause: blocked** — codebase reality conflicts with the brief in a way plan can't reconcile, or required tool/credential/fixture is missing.

**Invariant:** every load-bearing fact do will rely on carries a citation; every §Brief §Open question is resolved; every §Success-sketch signal maps to ≥ 1 criterion; every criterion is mechanically verifiable per §4.
