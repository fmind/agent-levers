---
name: lever-status
description: Inspect or cancel levers. List all (no arg), detail one (TL;DR + criteria + per-criterion events timeline), or cancel one with `<id> cancel [<reason>]`. Never advances the chain.
---

# lever-status

Three modes against `.agents/levers/`: list, detail, cancel. List and detail are read-only; cancel is the only write path. None of them advance the chain — that's `/lever <id>`'s job.

## State model recap

`lever.yaml.step ∈ {plan, do, check, act, done, cancel}` is the single state pointer. `pause ∈ {ask, blocked}` appears only when halted at `step` waiting for the user, and only ever at `step: plan` or `step: do`. The pair `(step, pause)` is the full state.

## 1. Pre-flight

1. Confirm `.agents/levers/` exists. If missing, stop with a chat sentence pointing to `/lever-init`.
2. Branch on the argument:
   - **No argument** → list mode (§2).
   - **`<id-or-slug>`** alone → detail mode (§3).
   - **`<id-or-slug> cancel [<reason>]`** → cancel mode (§4).

## 2. List mode (no argument)

1. Read every `.agents/levers/*/lever.yaml`. Skip directories without one and note them under a `Skipped:` line in the chat reply.
2. For each lever extract: `lever_id`, `slug`, `step`, `pause` (optional), criteria pass count, `updated_at`.
3. Print as a plain-text table sorted by `lever_id` (no markdown table syntax — chat is monospace). State is `<step>` or `<step> · paused: <reason>`:

   ```text
   ID  Slug                     State                  Pass     Updated
    1  add_password_sign_in     check                  3/4      2026-05-03 09:55
    2  fix_dropdown_ios         plan · paused: ask     0/0      2026-05-02 14:12
    3  refactor_api_routes      done                   8/8      2026-05-01 18:30
   ```

4. Trim `slug` to 24 chars with `…` ellipsis if longer. Trim `Updated` to `YYYY-MM-DD HH:MM`.
5. End with a one-line summary (e.g., "3 levers · 1 done · 1 awaiting answer · 1 ready to advance.").

## 3. Detail mode (`<id-or-slug>`)

1. Resolve `<id-or-slug>` → `.agents/levers/<id>-<slug>/`:
   - Pure integer → match the leading `<id>`.
   - String → match a slug fragment; on multi-match pick the lowest `<id>` and note in the chat reply.
   - Missing → stop with a chat sentence; recommend `/lever-status` to list.

2. Read `lever.yaml` and the LEVER.md `## TL;DR` block (the first `## TL;DR` heading and its bullet list — verbatim).

3. Print:

   ```text
   Lever <id> — <slug>
   State: <step>            (or "<step> · paused: <reason>")
   TL;DR:
   <verbatim TL;DR bullets>
   Criteria: C1 ✓ · C2 ✓ · C3 ✗ · C4 ✓ (3/4 passing)
   Next: /lever <id> to advance.
   ```

4. The `Next:` line varies by `(step, pause)`:
   - `step ∈ {plan, do, check, act}` and no `pause` → `/lever <id> to advance`.
   - `pause: ask` → `awaiting answer — see TL;DR`.
   - `pause: blocked` → `blocked — see TL;DR`.
   - `step: done` → `done — human review of the diff`.
   - `step: cancel` → `cancelled`.

5. Render the trace inline whenever any criterion has events (§5). Skip when every `events` is empty (typical at `step: plan`).

## 4. Cancel mode (`<id-or-slug> cancel [<reason>]`)

1. Resolve `<id-or-slug>` → directory (§3 rule).
2. Read `lever.yaml`. If `step ∈ {done, cancel}`, stop with `"Lever <id> already <step>; nothing to do."` — terminal states are immutable.
3. Write `lever.yaml`: `step: cancel`, drop any `pause`, bump `updated_at`. Validated against `lever.schema.json`. Leave `criteria`, `decisions`, `hints` intact for archival.
4. Replace LEVER.md `## TL;DR` in place with the cancellation block. If the user supplied `<reason>` (everything after `cancel`), include it; otherwise omit the line.

   ```markdown
   ## TL;DR

   - **State:** `step: cancel` — lever cancelled.
   - **Next:** none — directory stays for archival.
   - **Cancelled because:** <reason if provided>
   ```

5. Chat reply: one sentence stating the lever was cancelled and naming the directory.

## 5. Trace render (detail mode, when events exist)

For each criterion in `lever.yaml.criteria` (declared `id` order), print:

```text
C<n> — <description> (<pass | fail>)
  HH:MM <verb> · <cmd or "—">
    output: <one trimmed line, when present>
    note:   <one-line rationale or fold root-cause / drop count, when present>
  HH:MM-HH:MM fold (<N> earlier events dropped)
    note:   <drop count or root cause>
  HH:MM pass · <cmd>
    output: <verifier summary>
```

Conventions:

- One block per criterion; if a criterion has no events while others do, print its header followed by `(no events)`.
- Within a criterion, events are in append order — don't reorder by timestamp.
- For `verb: fold`, prefer the `at` range and the `note`; omit `cmd`.
- Trim `output` and `note` to one line each; append `…` if a value contained a newline.

End the chat reply with a one-line summary across criteria (e.g., "4 criteria · 6 events total · 1 cap-fold at C3.").

## 6. Hand off

**Invariant:** list/detail output reflects what's actually in the files (no caching, no inference). Cancel writes only `lever.yaml.step` / `pause` / `updated_at` and the LEVER.md `## TL;DR` block — never criteria, decisions, hints, or the rest of LEVER.md. End the chat reply with the printed table (list), detail block + inline trace (detail), or one-sentence cancellation confirmation (cancel). No fixed line format.
