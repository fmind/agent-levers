Reference for the **check** step. Opened by `skills/lever/SKILL.md` when `lever.yaml.step: check`.

Walk the chain — §Brief → §Plan → `criteria[i].events` → §Do — and confirm three things: (1) **internal consistency** (each step honored the previous); (2) **result aligns with user intent** (no silent scope drift, no non-goals built); (3) **every verifier passes** (re-run from a fresh shell). Trust the chain audit + verifier rerun as ground truth.

**Check never pauses.** It either advances forward (`step: act` or `step: done`) or routes back (`step: do` or `step: plan`).

## 1. Pre-flight

Reads (chain order):

1. `lever.yaml` — `criteria` (each `id`, `description`, `verify`, `seed`, `passes`, `output`, `events`), `budget`, `decisions`, `step` (must be `check`).
2. LEVER.md §Brief, §Plan, §Do (TL;DR, Summary, Coverage, Decisions, Deferred, Files touched). §Coverage is the human-readable mirror of `lever.yaml.criteria`.
3. `git status` and `git diff --stat`.

If `step != check`, stop with a chat sentence naming the expected state. (Check has no `pause` — structural error.)

## 2. Audit the chain

Capture one bullet per handoff under §Check §Chain audit (`pass` or `drift` with a one-line gap):

- **Brief → Plan.** Did plan resolve every §Brief §Open question with a citation? Does §Plan §Goal match §Brief §Goal? Carrying §Brief §Context constraints into §Plan §Constraints? Every §Success-sketch signal reflected in ≥ 1 criterion?
- **Plan → Do.** Did `events` show every `Cn` driven (not skipped)? Did `decisions` stay inside §Plan §Constraints? Are §Do §Deferred items genuinely out-of-scope or quietly dropped requirements?
- **Do → Result.** Does §Do §Summary describe what `events` shows? For each `Cn` in §Do §Coverage, does `events` carry a matching `pass` row? Do §Do §Files touched match `git diff --stat`?
- **Result → User intent.** Does the diff (or visible behavior) deliver §Brief §Goal and §Success sketch? Did anything in §Goal's non-goals get built anyway?

Minor `drift` may be benign (a deferral the human will accept) — note and move on. Serious `drift` (a non-goal got built; an §Open question was never resolved; §Do claims success on a `Cn` whose events log shows only failures) becomes a `step` reroute in §4.

## 3. Re-run verifiers

Run each `verify:` per its `type`; capture exit code + relevant output. **Overwrite** `lever.yaml.criteria[i].passes` and `output` with the rerun result; bump `updated_at`. Then **edit §Do §Coverage in place** to reflect the latest pass/fail. Cite any regression by `Cn` in §Check §Summary.

For `type: visual` and `type: manual`, the rerun's "ground truth" is the user-confirmation event in `events`. Cite the timestamp; don't re-prompt unless events show confirmation never happened.

Any `fail` row blocks ship. Either apply the smallest fix to the implementation (≤ 5 lines and obvious), or hand back with the failing rows pinpointed. **Never edit `AGENTS.md` / in-tree `SKILL.md` / `skills/lever/references/<step>.md` to make a verifier pass — those are act's surfaces.**

## 4. Pick one primary action

- **Ship it (default).** Every rerun is `pass` AND every chain-audit row is `pass` (or only benign drift). Continue to §5.
- **Hand back to do.** A verifier failed, OR chain audit surfaced in-scope drift the doer can fix. Spec is fine; execution is off. `step: do`, no `pause`. Skip §5.
- **Hand back to plan.** `events` + §Do show the _spec_ was wrong: criteria don't capture intent, §Plan §Constraints missed an invariant, or findings led do astray. Execution faithfully followed a wrong plan. `step: plan`, no `pause`. Skip §5.

**How to tell do-vs-plan apart:** ask "if do re-ran the loop with the current `criteria`, would it converge?" Yes → hand back to do. No → hand back to plan.

**Scope guard.** Plan re-runs may revise criteria/constraints/findings — but not silently widen scope past §Brief §Goal. If the goal itself needs to expand, plan will escalate via `pause: ask` when re-entered.

## 5. Scan for act hints (Ship-only)

Walk the chain, flagging moments where the agent's output drifted from human expectation, where a decision deserved more guidance, or where a recurring pattern is starting to show. **You're pointing, not investigating, and never editing — act owns root-cause AND every edit to `AGENTS.md` / in-tree `SKILL.md` / `references/<step>.md`.**

Sources to skim: `events` (retried `fail` rows; cap-fold summaries), `decisions` and §Do §Deferred vs §Brief §Goal/§Open questions, §Do §TL;DR (off-track items), `git log -p AGENTS.md` (a third occurrence of the same pattern is stronger than a first).

For each promotable moment, append one bullet to `lever.yaml.hints`. Reference events by `(Cn, at)`:

```yaml
hints:
  - "<source — Cn.events HH:MM, lever.yaml.decisions, §Brief §Goal, ...> — <one-line symptom>"
```

Hint count routes `step`:

- **Hints non-empty** → `step: act`, no `pause`. Auto-loop continues into act on the same invocation.
- **Hints empty** → `step: done`, no `pause`. Closing: `"All <p>/<t> verifiers passed. No hints. Done."`

## 6. Write the artifacts

`lever.yaml` — already updated in §3 with rerun results. Lift hints into `hints`. Set `step` per §4/§5 (no `pause`). Bump `updated_at`.

`LEVER.md` — replace §TL;DR with the block below; append §Check below the last existing section using `templates/check.md` (mirrors `lever.yaml.hints` in §Hints).

§TL;DR after check:

```markdown
## TL;DR

- **Verifiers:** <pass>/<total> rerun green.
- **Chain alignment:** Brief→Plan <p/d> · Plan→Do <p/d> · Do→Result <p/d> · Result→Intent <p/d>.
- **Hints:** <count> for act (or "none").
- **State:** `step: <done|act|do|plan>` — <one-line reason>.
- **Next:** <`/lever <id>` to act / do / plan, or "human review of the diff" on `step: done`.>
```

## 7. Hand off

See `skills/lever/SKILL.md` §3 for the shared frame. Check never pauses; outcomes:

- **Advance to done** — closing: `"All <p>/<t> verifiers passed. No hints. Done — diff is staged."`
- **Advance to act** — auto-loop continues into act on the same invocation, so the user typically sees act's closing.
- **Hand back to do** — closing: `"Failing on Cn; <p>/<t> passing. Run /lever <id> to resume do."`
- **Hand back to plan** — closing: `"Plan needs revision: <gap>; <p>/<t> passing. Run /lever <id> to re-spec."`

**Invariant:** rerun reflects ground truth (every `passes` overwritten from a real verifier run); §Do §Coverage edited in place; `hints` names every promotable moment as a one-line symptom (mirrored in §Check §Hints); §Diff summary matches `git diff --stat`.
