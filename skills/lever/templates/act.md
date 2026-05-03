# §Act section template

Appended below the last existing LEVER.md section by the act step. Empty sections: write a single `- none` bullet rather than omit the section. No trailing status-formatted line.

````markdown
## Act

### Summary

<one paragraph — what gaps the trail surfaced, where each finding landed, what the project (and the levers) now teach themselves.>

### Source trail

- `lever.yaml.hints` — <count> hint(s); read first.
- `lever.yaml.criteria[i].events` — cap-fold rows, repeated `fail` rows aligned with each hint.
- `lever.yaml.decisions` — non-obvious calls do made.
- §Do — Summary, Deferred.
- §Plan / §Brief — re-read on demand when a hint pointed back.

### Hints reviewed

- <hint summary> — kept; target: `<file>` (project) / `skills/<name>/SKILL.md` / follow-up `/lever-new <title>`.
- <hint summary> — dropped; reason: <one-off / project-domain-knowledge / not real / skills not in tree>.

### Findings

#### <finding title>

- Symptom: <one line — cite `Cn.events HH:MM` / §Do / diff line>.
- Root cause: <one line — why it happened, not just what>.
- Catches: <signal + two future levers this rule would catch — proves it generalizes>.
- Target: `<file>` or `/lever-new <title>` — <why it beat alternatives>.
- Edit / recommendation: `<file>` +<n>/-<m> — <one line on what bullet/section changed>; or `/lever-new <title> (<reason>)` for follow-up work.

### Alternatives considered

- <surface or framing rejected> — <one-line reason>.

### Files touched

```text
<git diff --stat — empty when only follow-up /lever-new recommendations were made>
```
````
