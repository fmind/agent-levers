Reference for the **act** step. Opened by `skills/lever/SKILL.md` when `lever.yaml.step: act` (check surfaced hints). When `hints[]` is empty, check advances directly to `step: done` and act does not run.

Walk the trail — §Brief → §Plan → `criteria[i].events` → §Do → §Check (and `lever.yaml.hints`) — and decide which moments are worth codifying. Deliverable: one or more proposals, each landed on the right surface — or a follow-up lever recommendation when the finding is bigger than a rule.

**Act never pauses.** It either advances to `step: done` (proposals staged) or stops on a structural error.

## 1. Pre-flight

Reads:

1. `lever.yaml` — confirm `step: act`. Read final `criteria`, `hints`, `decisions`, `criteria[i].events`.
2. LEVER.md §Check (chain context, §Hints mirror), §Do (TL;DR, Summary, Decisions, Deferred), §Plan / §Brief (re-open only when a hint points back).

If `step != act`, stop with a chat sentence naming the expected state. If `hints` is empty, this skill shouldn't have run — surface and stop.

## 2. Investigate each hint

For every hint, in order:

1. **Is it real?** Confirm against `events`, §Do, or the diff. Drop hints that don't reproduce.
2. **Does it generalize?** Name two plausible future levers it would catch. If you can't, drop as one-off.
3. **What surface owns it?** Use §3.

New patterns spotted while investigating are fair game — add as additional findings even if check didn't flag them. Dropped hints go in §Hints reviewed with the reason.

## 3. Pick the target for each finding

Top-to-bottom; first matching symptom wins:

- **Cross-cutting project rule** (convention drift, recurring mistake) → `AGENTS.md` (default). A dedicated rules/memory skill in the project takes precedence if one exists.
- **Skill flaw or gap** → edit an in-tree `SKILL.md`, or scaffold a new one under `skills/`. `skills/lever/references/<step>.md` are valid when agent-levers itself is the tree. When no skills are in the tree, fold into `AGENTS.md` or drop.
- **Concrete project work, not a rule** → follow-up `/lever-new <new-title>` — surfaced inline in §6 (e.g., "add an ESLint rule that catches this", "refactor X to remove the trap"). Act writes no file.
- **One-off mistake** → drop. Note `dropped; reason: one-off`.
- **Project domain knowledge** (auth flow, schema details) → drop from act — that knowledge belongs in whatever notes/docs/memory mechanism the project uses, captured during plan.

A single act may span any combination of the three keep-targets. Note the chosen target (and why it beat alternatives) for each finding in §Findings.

## 4. Propose the change

For `AGENTS.md` and `skills/` targets: edit, add, or restructure bullets/sections — or scaffold a new `skills/<name>/SKILL.md` when the lesson lands cleaner as its own skill. If a file feels at capacity, evict stale bullets in the same edit and explain in §Findings.

For follow-up `/lever-new` targets: write nothing. Recommendation rides inline in §6. Title in short snake_case, paired with a one-line reason.

Edits sit in the working tree as proposals — keep each minimal so the user can accept, revise, or reject independently.

## 5. Write the artifacts

`lever.yaml` — set `step: done`, no `pause`, bump `updated_at`. `criteria`, `budget`, `decisions`, `hints` are immutable here.

`LEVER.md` — replace §TL;DR with the block below; append §Act below the last existing section using `templates/act.md`.

§TL;DR after act:

```markdown
## TL;DR

- **Gaps surfaced:** <count> kept · <count> dropped.
- **Project edits:** <files (or "none")>.
- **Skill edits:** <`skills/<name>/SKILL.md` paths (or "none — skills not in tree" / "none")>.
- **Follow-up levers:** <count recommended (or "none")>.
- **State:** `step: done` — proposals staged; lever finished.
- **Next:** user reviews staged edits — accept, revise, or reject each independently. For follow-up `/lever-new` recommendations, user invokes `/lever-new <title>`.
```

## 6. Hand off

See `skills/lever/SKILL.md` §3 for the shared hand-off frame. Act never pauses; only outcome is **advance to done** — proposals staged. Mention each follow-up `/lever-new` inline with a one-line reason (e.g., `"Optional follow-ups: /lever-new enforce_route_middleware_lint to catch missing guard middleware at lint time"`). If hints don't survive §2, still mark the lever done and write a §Act stub explaining hints were investigated but didn't promote.

**Invariant:** each edit is small enough for independent accept/revise/reject; each finding cites a source (`Cn.events HH:MM`, §Do, diff line); each follow-up `/lever-new` pairs the title with a one-line reason.
