# Rigor Checklist

Apply only the items that fit the artifact. Each is a concrete thing to verify, not a generic caution.

## Plots (PDF/PNG)

- **Axes**: Are both axes labelled with units? Is the scale linear or log — and does the chosen scale flatter or exaggerate the effect?
- **Axis range**: Does a truncated/non-zero baseline or a zoomed range make a small difference look large (or vice versa)?
- **Uncertainty**: Are there error bars, confidence bands, or shaded regions? What do they represent (std, SEM, CI, min–max)? If none, differences cannot be called significant.
- **Sample size / seeds**: Is n shown anywhere (legend, caption, points)? Trends from few runs are fragile.
- **Baselines / references**: Is there a comparison point (prior method, random, chance, theoretical bound)? An absolute curve with no baseline is hard to judge.
- **Cherry-picking**: Does the x-range, subset of conditions, or chosen metric look selected to favour one method? What's plausibly omitted?
- **Overlap / occlusion**: Are series distinguishable, or do markers/lines hide each other? Could a hidden curve change the read?
- **Metric choice**: Is the plotted metric the one that matters, or a proxy? Higher = better, or lower = better — is that obvious?
- **Aggregation**: Mean vs median vs best-of? "Best run" curves overstate typical performance.

## Tables (LaTeX)

- **Variance**: Is dispersion reported (±std, CI, IQR)? A bold "best" number without spread may be within noise.
- **What's bolded**: Is the highlighted winner actually distinguishable from runners-up, given the (reported or missing) variance?
- **Seeds / trials**: How many runs back each cell? Stated in caption or assumed?
- **Baselines & ablations**: Are the obvious comparison rows present? Is the strongest competitor included or quietly dropped?
- **Precision consistency**: Are decimal places consistent and not implying false precision?
- **Aggregation method**: Mean across what? Are failed/diverged runs excluded (survivorship)?
- **Metric direction**: Does the table state whether ↑ or ↓ is better per column?
- **Missing cells**: Are there gaps (— or N/A)? Why are those configurations absent?
- **Multiple comparisons**: Many columns/rows with one bolded winner each — is "best on at least one metric" being oversold?

## Cross-cutting

- **Claim vs evidence**: Does the strongest interpretation actually need an assumption the artifact doesn't justify?
- **Consistency**: Do a plot and a table of the same experiment agree? Disagreement is a red flag.
- **Confounds**: Could something other than the named variable explain the trend (compute budget, tuning effort, data differences)?
