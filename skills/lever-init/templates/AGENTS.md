# AGENTS.md

## Overview

- <what the project does — one sentence>
- <primary stack — language, framework, key runtime>
- <load-bearing constraint — deployment target, latency budget, compliance, etc.>

## Conventions

- <code style — formatter, lint command>
- <test command and where tests live>
- <naming, file layout, import rules>
- <commit / branch / PR rules>

## Communication

- <response style — terse vs. verbose, code-first vs. explanation-first>
- <when to ask vs. proceed on ambiguity>
- <tone>
- <how to log assumptions or defaulted choices>

## Workflow

- Use skills (slash commands) for repeating ops — e.g., /deploy-project <env>, /run-tests.
- Use lever skills
  - /lever-new <title> to capture a new task (Session 1: discuss intent, write the brief)
  - /lever <id> to run the chain (Session 2: plan → do → check → act → done, autonomously until paused or finished)
  - /lever-status to inspect levers in flight; /lever-status <id> cancel [<reason>] to retire one
- <other invokable skills the agent should reach for, optional>

## Layout

- `<dir>/` — <what it holds>
- `<dir>/` — <what it holds>
