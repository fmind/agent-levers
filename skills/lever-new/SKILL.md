---
name: lever-new
description: Start a new lever — discuss the ask in chat, then capture intent (Goal, Success sketch, Context, Open questions) as §Brief. Hands off via /lever <id>.
---

# lever-new

Capture user intent — what to build, what success looks like — as the lever's inaugural §Brief. The brief is a contract about WHAT to build; the chain that follows (`/lever <id>` → plan → do → check → act) owns mechanism, work, and verification.

§Brief is the **input** to the chain. The chain reads it, investigates, builds the spec, executes, and verifies. The chat session before the brief lands is the human's last guaranteed touchpoint with the lever — capture enough that the chain can run end-to-end without pausing to re-ask.

**Dialog first (hard rule).** First-turn output is chat-only: clarifying questions, proposed enhancements, and a draft brief inline. Do not create `.agents/levers/<id>-<slug>/`, do not write `LEVER.md`, do not seed `lever.yaml` until the user explicitly agrees (typed acknowledgment, "looks good", edits incorporated). Files commit on the second turn at earliest. Brief is the cheapest place to improve intent.

## 1. Pre-flight (read-only)

First-turn pre-flight is reads only — no file writes, no directory creation. Read in order: `AGENTS.md` (and any nested ones), `README.md`, the user ask, any spec the ask points at (e.g., `TASK.md`, design doc, link).

Goal: know enough to ask sharp questions and sketch a draft. Stop when you can classify §2's ambiguities and propose a draft.

## 2. Discuss and propose

For each ambiguity, classify:

- **User-only** — irreducible, only the user can answer (business rules, scope boundaries, irreversible architectural choices, security posture, deadlines, new third-party dependencies). → escalate as a clarifying question in chat.
- **Investigation-resolvable** — the chain can answer it by reading the codebase or external docs (which file owns this, which test framework is wired, what the existing pattern looks like). → note for §Open questions; the chain resolves these without coming back.
- **Conventional default** — project context already dictates an answer (a rule in `AGENTS.md` §Conventions, a preset in `package.json`). → fold into §Context implicitly.

Bias toward asking when stakes are real; default only for low-stakes choices that align with explicit conventions. Re-attempt classifying every user-only question as investigation-resolvable or conventional-default before raising it — a user-only question that slips past the dialog forces the chain to pause and re-ask later. Cap user-only questions at 5 (10 for greenfield); if you'd need more, the brief is premature.

**Propose enhancements.** Look for sibling capabilities, parallel features, or edge cases the goal implies but the user didn't ask for. One enhancement question is fine; don't pad. Extend where extension is obvious ("you specified Google OAuth — want GitHub too?").

**First-turn output.** Reply in chat with: (a) clarifying questions, (b) proposed enhancements, (c) a draft brief inline (Goal, Success sketch, Context, Open questions). No file writes.

## 3. Commit on agreement

Files commit only after explicit agreement — typed acknowledgment, "looks good", "ship it", or edits incorporated.

Once agreement is reached:

1. Pick `<id>`: highest leading integer among existing `.agents/levers/<n>-*` directories, plus 1 (start at 1 if none exist).
2. Slug the title to lower snake_case, ≤ 40 chars (drop punctuation, collapse whitespace).
3. `mkdir -p .agents/levers/<id>-<slug>/`.

§Success sketch is informal — one to three bullets describing what a human would observe when this works, in plain English. Not acceptance criteria (those come from chain investigation), not mechanism — a north star anchoring the chain's downstream criteria. If you can't sketch it, the goal is still ambiguous; loop back to §2.

## 4. Write the artifacts

Two files in `.agents/levers/<id>-<slug>/` — read both templates (siblings of this `SKILL.md`) before writing:

**`LEVER.md`** — match `templates/LEVER.md`. Initial state: title (`# Lever — <id>-<slug>`), §TL;DR (≤ 20 lines: state + intent + what to do or decide next), and §Brief (Goal, Context, Success sketch, Open questions). No frontmatter. Future sections (§Plan, §Do, §Check, §Act) appear when their step runs.

**`lever.yaml`** — match `templates/lever.yaml`. Seed:

```yaml
lever_id: <id>
slug: <slug>
created_at: <ISO8601 UTC current timestamp>
updated_at: <same as created_at on first write>
step: plan # brief captured; the chain enters plan next
criteria: []
decisions: []
hints: []
```

`budget` is absent — plan adds it. `pause` is absent on the happy path; set only if a user-only question survives the dialog and lands unanswered (`pause: ask`) — rare. `step: plan` means the next invocation, `/lever <id>`, runs the chain autonomously through plan → do → check → act → done.

## 5. Hand off

Two outcomes:

- **Advance** — brief captured cleanly. Write `step: plan`, no `pause`. Closing: `"Brief captured; lever.yaml + LEVER.md written. In a new session, run /lever <id> — the chain runs autonomously from there."`
- **Pause** — a user-only question slipped past the dialog and only surfaced after files were written (irreducible business rule, external SLA, technical fork). Rare. Write `step: plan`, `pause: ask`; state the question and list options when there's a fork.

If the user abandons before agreement, stay in dialog and write nothing. To abandon after files are written, use `/lever-status <id> cancel [<reason>]`.

`lever-new` never invokes `/lever <id>` itself. The two-session split is deliberate: brief-time is the only place the human is in the loop; chain execution is mechanical and reads from files.

**Invariant:** §Brief answers WHAT to build and why, names every signal a human would observe when this works, and lists every ambiguity the chain must resolve through investigation — so the chain investigates and executes, never re-debates intent or re-asks the human.
