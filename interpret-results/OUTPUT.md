# Output Structure

Write to `results/interpretations/<name>.md`. One section per artifact, then a synthesis.

```md
# Interpretation: <batch topic or artifact name>

**Artifacts:** <list of files analyzed>
**Date:** <today's date from environment>

---

## <artifact-1 filename>

**Setup:** <axes/columns/rows, units, scale, what's varied, what's fixed>

### What it shows
- [shown] <directly readable observation, with numbers>
- [inferred] <interpretation> — assumes: <assumption>
- [not shown] <relevant thing the artifact does not display>

### Rigor concerns
- <concrete, checkable concern from the checklist>

---

## <artifact-2 filename>
...

---

## Synthesis

- **Agreements:** <where artifacts reinforce each other>
- **Contradictions:** <where they disagree, and which to trust>
- **Strongest supported claim:** <the claim the set jointly justifies>
- **Weakest link:** <result least likely to survive scrutiny, and why>
- **Open questions:** <what the researcher should resolve before relying on these>
```

Notes:
- Tag every observation `[shown]`, `[inferred]`, or `[not shown]`.
- Omit the **Rigor concerns** block for an artifact if nothing applies (rare — usually at least sample size or variance is worth a line).
- Keep it scannable: bullets over paragraphs, real numbers over adjectives.
