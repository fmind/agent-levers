# §Do section template

Appended below the last existing LEVER.md section by the do step on the final write (after the loop turns green). Empty sections: write a single `- none` bullet rather than omit the section. No trailing status-formatted line.

§Coverage mirrors `lever.yaml.criteria`. §Decisions mirrors `lever.yaml.decisions`. `lever.yaml` remains the source of truth for `passes` / `output` / decision content; humans read LEVER.md without opening YAML.

````markdown
## Do

### Summary

<one paragraph>

### Coverage

(Mirror of `lever.yaml.criteria` after this run. Pull `description` and `verify` from `lever.yaml` so humans can read LEVER.md standalone. `lever.yaml` is the source of truth for `passes` / `output`.)

- `C1` — <criterion>. verify: `<cmd or type:visual/manual summary>` → <one-line output>. **pass**.
- `C2` — <criterion>. verify: `<cmd>` → <one-line output>. **pass**.

### Decisions

(Mirror of `lever.yaml.decisions`. One bullet per decision, verbatim.)

- <non-obvious call do made>
- <another non-obvious call>

### Deferred

- <bullet>

### Files touched

```text
<git diff --stat>
```
````
