---
name: lever
description: Advance an existing lever. Reads lever.yaml.step and runs the chain (plan → do → check → act → done) autonomously until it pauses or finishes.
---

# lever

Advance the lever chain for `<id-or-slug>`. Auto-chains through plan → do → check → act → done; pauses only if a step needs human input or hits a budget; finishes on `done`. To cancel a lever, use `/lever-status <id> cancel [<reason>]`.

## State model

`lever.yaml.step ∈ {plan, do, check, act, done, cancel}` is the single state pointer. `lever.yaml.pause ∈ {ask, blocked}` is set only when halted at `step` waiting for the user, and only ever at `step: plan` or `step: do`. The pair `(step, pause)` is the full state. Check and act always advance, route back, or finish — they never pause.

References under `references/` are loaded on demand — only the one matching `step`, never all four.

## 1. Pre-flight

1. **Resolve `<id-or-slug>`** to `.agents/levers/<id>-<slug>/`:
   - Pure integer → match the leading `<id>`.
   - String → match a slug fragment; on multi-match pick the lowest `<id>` and note in chat.
   - Missing → stop with a chat sentence naming the expected path. Recommend `/lever-status` to list, or `/lever-new <title>` to create.

2. **Read `lever.yaml`.** Branch on `step`:
   - `done` or `cancel` (terminal) → stop with a chat sentence reporting the lever is finished/cancelled. Recommend `/lever-status <id>` for detail.
   - `plan | do | check | act` (with or without `pause`) → §2.

   On re-entry with `pause` set, the procedure for `step` reads the latest user message and clears `pause` when the question/blocker resolves.

## 2. Run the chain

1. Open `skills/lever/references/<step>.md` and execute it end-to-end against this lever directory. The procedure reads its inputs, writes its LEVER.md section + TL;DR rewrite, updates owned `lever.yaml` fields, advances `step` (or sets `pause`), and bumps `updated_at`.
2. After the step returns, decide whether to continue:
   - `pause` set → stop. Hand off via §3.
   - `step ∈ {done, cancel}` → stop (terminal). Hand off via §3.
   - `step ∈ {plan, do, check, act}` and no `pause` → re-read `lever.yaml`, loop §2.

Don't iterate without re-reading `lever.yaml` between turns — each step writes state the next must observe. The dispatcher doesn't interpret procedure output; the procedure's own §Hand off owns the chat-reply sentence. The dispatcher prepends one line naming the steps that ran (`Ran: plan → do → check.`).

## 3. Hand off

The procedure owns the final state write. **Invariant:** `step` advanced as expected (or stayed put on a within-step pause). If a procedure's `lever.yaml` write fails schema validation, surface the underlying error rather than mask it.

End the chat reply with one prepended line — `Ran: <step1> → <step2> → ...` — followed by the procedure's own closing sentence stating what happened, the current state, and the next command. No fixed line format.

Three categories of outcome:

- **Advancing** — chain ran one or more steps; next step is queued. If the auto-loop is still running, this won't be the final state surfaced. Closing names `/lever <id>` as the next command.
- **Paused** — the procedure set `pause: ask` or `pause: blocked` (plan or do only). The chat reply states the question (`ask`) or the blocker and how to fix it (`blocked`). Re-invoke `/lever <id>` after answering / fixing.
- **Terminal** — `step: done` (lever finished; user reviews the diff and any staged act edits) or `step: cancel` (set via `/lever-status <id> cancel [<reason>]`).
