# §Check section template

Appended below the last existing LEVER.md section by the check step. Empty sections: write a single `- none` bullet rather than omit the section. No trailing status-formatted line.

In addition to appending §Check, the check step **updates §Do §Coverage in place** with the rerun results — flipping `pass` ↔ `fail` rows and refreshing `output` lines. §Coverage is the single human-readable mirror of `lever.yaml.criteria`. Cite criteria by `id` (`C2 regressed`); the reader scrolls up to §Coverage for the latest pass/fail.

§Hints mirrors `lever.yaml.hints` so humans read LEVER.md without opening YAML.

````markdown
## Check

### Summary

<one paragraph — does the chain hold; do verifiers pass; ship recommendation in one line. Cite any rerun regressions by `Cn`.>

### Chain audit

- Brief → Plan: pass / drift — <one-line finding>
- Plan → Do: pass / drift — <one-line finding>
- Do → Result: pass / drift — <one-line finding>
- Result → User intent: pass / drift — <one-line finding>

### Hints

(Mirror of `lever.yaml.hints`. One bullet per promotable moment, verbatim. Empty when no hints surfaced.)

- <hint — source + one-line symptom>
- <hint — source + one-line symptom>

### Diff summary

```text
<git diff --stat trimmed to relevant lines>
```
````
