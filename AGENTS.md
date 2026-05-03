# AGENTS.md

## Overview

- agent-levers ships a small set of slash-command skills (`/lever-init`, `/lever-new`, `/lever`, `/lever-status`) that give a coding agent a structured plan → do → check → act loop.
- Skills are file-based and harness-agnostic: the same `skills/` tree works in Claude Code, Gemini CLI, and GitHub Copilot.
- Repo state is `.agents/levers/<id>-<slug>/{lever.yaml, LEVER.md}` — `lever.yaml` validated against `lever.schema.json`.

## Conventions

- **Line caps.** Skill files exist to be loaded by an agent on every run; verbosity directly costs context budget. Targets:
  - `SKILL.md` — ≤ 110 lines
  - `skills/lever/references/*.md` — ≤ 100 lines (each step procedure)
  - `skills/lever/templates/*.md` — ≤ 50 lines (each LEVER.md section template)
  - `skills/*/templates/*.md` — ≤ 50 lines (other artifact templates)
  - Project-root markdown (`README.md`, `AGENTS.md`) — uncapped, but tighten when it sprawls

  When a file approaches its cap, prefer cutting motivational/explanatory prose over removing structural rules. Each rule should be stated once, in the imperative, in the file the agent loads when it needs it. The "why" lives in commit messages and PR descriptions.

- **Lint.** Run `npm run lint` and `npm run validate` to validate changes — `lint` covers Prettier formatting and markdownlint-cli2; `validate` runs ajv against every `examples/**/lever.yaml`. Both gate `pre-commit` locally and CI.

- **Voice.** Skill bodies address the agent in the imperative ("Read X, then write Y"). Avoid second-person flavor text aimed at the human reader — that belongs in the README.

## Workflow

- Use lever skills (this repo dogfoods its own framework on non-trivial changes):
  - `/lever-new <title>` — capture a new task (Session 1: discuss intent, write the brief)
  - `/lever <id>` — run the chain (Session 2: plan → do → check → act → done, autonomously until paused or finished)
  - `/lever-status` — inspect levers in flight; `/lever-status <id> cancel [<reason>]` — retire one

## Layout

- `skills/` — Agent Skills (`lever-init`, `lever-new`, `lever`, `lever-status`); the `lever` dispatcher loads `references/<step>.md` on demand.
- `lever.schema.json` — JSONSchema for end-user `.agents/levers/*/lever.yaml`.
- `examples/levers/` — runnable walkthroughs validated by CI.
- `.claude-plugin/`, `gemini-extension.json`, `plugin.json` — harness manifests for Claude Code, Gemini CLI, GitHub Copilot.
