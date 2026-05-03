# Lever — `2-fix_dropdown_ios_close`

## TL;DR

- **Verifiers:** 3/4 rerun green · `C2` regressed during check.
- **Chain alignment:** Brief→Plan pass · Plan→Do pass · Do→Result pass · Result→Intent drift — fix introduced a regression on the close-button path.
- **Hints:** none — check routes back to do; act will run after the green rerun.
- **State:** `step: do` — execution gap; `pointerdown` listener races with the close-button handler.
- **Next:** `/lever 2` to resume do and fix the `C2` regression.

## Brief

### Goal

The site-header dropdown menu does not close on iOS Safari when the user taps outside it (works on Chrome, Firefox, Android Safari, desktop Safari). Add the missing close behavior without regressing close-button or keyboard-escape paths. Visual styling and the menu's open/scroll animation are out of scope for this task.

### Context

- The dropdown's outside-tap listener lives at `src/components/Dropdown.tsx:42` and currently uses a `click` event on `document`. — source: codebase grep.
- iOS Safari does not synthesize `click` on `document.body` for taps that originated on a child element. — source: user report ("only iOS").
- E2E framework is Playwright with the `webkit` profile aliased to "iOS Safari". — defaulted from `playwright.config.ts`.
- TypeScript strict mode is on. — defaulted from `tsconfig.json`.

### Success sketch

- A user opens the dropdown on an iPhone, taps elsewhere on the page, the dropdown closes — same as on every other platform.
- Tapping the dropdown's close button still closes it (regression check).
- Pressing Escape still closes it (regression check).

### Open questions

- What event should replace `click` to fire reliably on iOS — `touchend` (legacy) or `pointerdown` (modern)? Plan to investigate browser support and any project-level pointer-event convention.

## Plan

### Goal

Replace the dropdown's outside-tap listener with an event that fires reliably on iOS Safari, without regressing the close-button or keyboard-escape paths.

### Investigation findings

#### Codebase

- Outside-tap listener: `useEffect` on `document` with `addEventListener('click', handleOutsideClick)` registered while `isOpen`. source: `src/components/Dropdown.tsx:42`.
- Close-button handler: `onClick={(e) => { e.stopPropagation(); close(); }}`. source: `src/components/Dropdown.tsx:71`.
- Project uses Pointer Events for drag interactions in `src/components/Reorderable.tsx:18`; no global "use pointer events" convention codified in `AGENTS.md`. source: codebase grep + `AGENTS.md`.

#### External

- Pointer Events are supported on iOS Safari ≥ 13 (project's `browserslist` requires ≥ 14). source: <https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events> (retrieved 2026-05-04).
- `touchend` works but is legacy and known to fire after `click` on devices that synthesize both. source: <https://www.w3.org/TR/pointerevents3/> (retrieved 2026-05-04).

#### Resolved questions

- Which event → `pointerdown`. Modern, reliable on iOS Safari ≥ 13, supported across all platforms in `browserslist`. source: MDN (above).

### Constraints

- No new runtime dependencies.
- The close-button and keyboard-escape paths must keep working unchanged — covered by `C2` and `C3`.
- Don't introduce a global pointer-events policy in `AGENTS.md`; this fix is local to `Dropdown.tsx`.

### Risks & mitigations

- `pointerdown` fires earlier in the gesture pipeline than `click` — it can race with the close button's own `onClick`. Mitigation: add a `pointerdown` capture-phase guard that bails out when the event target is inside the dropdown bounds. (`C2` is the regression check.)

### Deferred

- Migrating the rest of the codebase to pointer events — out of scope; project has no global convention.

## Do

### Summary

Replaced the `document` `click` listener at `src/components/Dropdown.tsx:42` with a `pointerdown` listener and added a guard that bails when `event.target` is inside the dropdown root. All 4 criteria flipped to `passes: true`: outside-tap closes the dropdown on iOS Safari (`C1`), the close button still closes it (`C2`), Escape still closes it (`C3`), type-check clean (`C4`). One decision captured in `lever.yaml.decisions` (event choice + capture-phase guard). Files: `src/components/Dropdown.tsx`, `tests/dropdown.spec.ts`.

### Coverage

(Single human-readable mirror of `lever.yaml.criteria`. Do writes it on green; check edits it in place after the rerun. `lever.yaml` is the source of truth for `passes` / `output`.)

- `C1` — Dropdown closes when user taps outside on iOS Safari. verify: `npx playwright test tests/dropdown.spec.ts -g 'closes on outside tap (iOS)'` → `1 passed`. **pass**.
- `C2` — Dropdown still closes via close button. verify: `npx playwright test tests/dropdown.spec.ts -g 'closes via close button'` → `expected dropdown.hidden=true, got false`. **fail (regressed during check rerun — see §Check §Summary)**.
- `C3` — Dropdown closes on Escape. verify: `npx playwright test tests/dropdown.spec.ts -g 'closes on escape'` → `1 passed`. **pass**.
- `C4` — typecheck clean. verify: `npm run typecheck` → `exit 0`. **pass**.

### Decisions

(Mirror of `lever.yaml.decisions`. One bullet per decision, verbatim.)

- Switched outside-listener event from `click` to `pointerdown` — Safari does not synthesize click on document body for taps that originated on a child element, and capture-phase pointerdown fires reliably on iOS. Trade-off recorded as a §Plan §Risk.

### Deferred

- Migrating other components to pointer events — out of scope per brief.

### Files touched

```text
 src/components/Dropdown.tsx  | 12 +++++++-----
 tests/dropdown.spec.ts       | 28 ++++++++++++++++++++++++++++
 2 files changed, 35 insertions(+), 5 deletions(-)
```

## Check

### Summary

Chain holds at the brief / plan / do level — `C2` was correctly declared as a regression check, and §Plan §Risks called out the `pointerdown` race with the close-button handler. Re-running every `verify:` from a fresh shell, `C2` regressed: the new `pointerdown` listener fires before the close button's `onClick` and re-opens the dropdown via a `stopPropagation` race that doesn't trigger in same-event-loop click handling. Updated §Do §Coverage in place: `C2` now reads **fail**. Routing `step: do` to resume — execution gap, not a spec gap; the criteria are right and the planned mitigation (capture-phase guard) just isn't wired correctly.

### Chain audit

- Brief → Plan: pass — outside-tap event question resolved with MDN + W3C citations; close-button and Escape paths declared as regression checks (`C2`, `C3`).
- Plan → Do: pass — all four criteria driven; `decisions` records the `pointerdown` + capture-phase choice and stays inside §Plan §Constraints (no new deps, no `AGENTS.md` edit).
- Do → Result: pass at the time §Do was written — every `Cn.events` had a matching `pass` row; §Files touched matches `git diff --stat`.
- Result → User intent: drift — the fix delivers the iOS outside-tap behavior (`C1`), but the rerun shows it regresses the close-button path (`C2`). Brief explicitly required the close-button to keep working; that promise is currently broken.

### Hints

(Mirror of `lever.yaml.hints`. Empty here — check routed back to do on the C2 regression and never reached its §Scan-for-act-hints sub-step.)

- none

### Diff summary

```text
 src/components/Dropdown.tsx  | 12 +++++++-----
 tests/dropdown.spec.ts       | 28 ++++++++++++++++++++++++++++
 2 files changed, 35 insertions(+), 5 deletions(-)
```
