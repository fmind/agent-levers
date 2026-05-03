---
name: lever-init
description: Bootstrap a repo for agent-levers. Creates .agents/levers/, wires CLAUDE.md/GEMINI.md to import AGENTS.md, writes an AGENTS.md skeleton when missing. Hands off via /lever-new.
---

# lever-init

Bootstrap this repo for agent-levers. Idempotent: only adds files. The one exception is §3 — when `AGENTS.md` exists but doesn't reference the lever skills, init appends a Workflow section pointing at them.

GitHub Copilot reads `AGENTS.md` natively; Claude Code (`CLAUDE.md`) and Gemini CLI (`GEMINI.md`) need a one-line `@`-import shim.

## 1. Pre-flight

1. Confirm the cwd is a git repo: `git rev-parse --is-inside-work-tree`. If not, stop with `"Blocked: not a git repository — run \`git init\` first."`
2. Run `git status --porcelain`. If non-empty, list dirty paths under a `Pre-existing changes:` heading in the chat reply and proceed — init only adds files.
3. Confirm `.agents/levers/` is not git-ignored (handles negations, nested `.gitignore`s, glob patterns, portable across Linux/macOS/Windows Git Bash):

   ```bash
   git check-ignore -q .agents/levers/.gitkeep
   ```

   Exit `0` means the path **is** ignored. Surface the offending line(s) with `grep -n '\.agents' .gitignore` and stop with a chat sentence naming the gitignored path.

## 2. Workspace skeleton

```bash
mkdir -p .agents/levers
```

## 3. AGENTS.md

`AGENTS.md` is the project's contract with its agents — voice, conventions, layout, workflow. Three cases:

- **Missing.** Write `templates/AGENTS.md` (sibling of this `SKILL.md`) verbatim. Surface in chat: `"Wrote AGENTS.md from the lever-init skeleton — fill in the placeholders before running /lever-new."`
- **Exists, mentions any of `/lever-new`, `/lever`, `/lever-status`.** No-op. The workflow is already wired.
- **Exists, no lever mention.** Append a Workflow section pointing at `/lever-new`, `/lever`, `/lever-status` (and `/lever-status <id> cancel [<reason>]` for retirement). Use the Workflow bullets from `templates/AGENTS.md`. If the file already has a `## Workflow` heading, append at its end (before the next `##` or EOF); otherwise add a new `## Workflow` section. Surface in chat: `"Appended a Workflow section to AGENTS.md — review and revert if you'd rather wire the skills differently."`

Detect the lever-mention case with:

```bash
grep -qE '/lever-new|/lever\b|/lever-status' AGENTS.md
```

The append is safe (additive, narrative-only) — no risk of overwriting prose. Init never edits any other AGENTS.md content.

## 4. Wire host context files (write missing, verify existing)

For each of `CLAUDE.md` and `GEMINI.md`:

- **Missing.** Write the import shim. The `[ -f … ] ||` guard is POSIX bash:

  ```bash
  [ -f CLAUDE.md ] || printf '@AGENTS.md\n'   > CLAUDE.md
  [ -f GEMINI.md ] || printf '@./AGENTS.md\n' > GEMINI.md
  ```

- **Exists.** Grep for `@AGENTS.md` (Claude Code) or `@./AGENTS.md` / `@AGENTS.md` (Gemini CLI). If absent, the host file won't pick up `AGENTS.md` — silent break. List each broken file with the suggested one-line edit under a `Drift detected:` heading and stop. Init never edits existing host files.

Copilot needs no shim — it reads `AGENTS.md` natively.

## 5. Surface the gitignore choice

Lever artifacts (`.agents/levers/<id>-<slug>/`) are committed by default — useful audit trail. If the user prefers to ignore them (PII, scratch), they add `.agents/levers/` to `.gitignore` themselves. Surface as one line under `Gitignore choice:` in the chat reply; don't write it for them.

## 6. Hand off

**Invariant:** `.agents/levers/` exists and isn't gitignored; `AGENTS.md` exists (written, appended-to, or pre-existing) and references the lever skills; host shims actually import `AGENTS.md`. End the chat reply with one clear sentence stating what just happened and the next command.

- **Ready** — everything in place. Recommend `/lever-new <short-title-of-first-task>`.
- **Ready, AGENTS.md just written/appended** — same recommendation, plus a note to fill in placeholders or review the appended section.
- **Blocked** — not a git repo; `.agents/levers/` is gitignored; or a host file exists without importing `AGENTS.md`. Name the blocker and the fix.
