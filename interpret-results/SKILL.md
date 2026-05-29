---
name: interpret-results
description: Interpret and critique research results presented as plots (PDF/PNG) or LaTeX tables — read off trends, compare conditions, and flag rigor problems (missing error bars, absent baselines, misleading axes, unsupported claims). Writes a per-artifact interpretation plus a cross-cutting synthesis to results/interpretations/<name>.md. Use when the user wants to interpret, analyze, sanity-check, or critique figures or tables of experimental results, or mentions result plots, .pdf/.png figures, or LaTeX result tables. Does NOT draft paper or thesis prose, and does NOT extract from source code — for documenting codebase internals for a write-up, use thesis-extract instead.
---

# Interpret Results

Help a researcher **process large volumes of results faster** by reading plots and tables, stating what they appear to show, and flagging where rigor is thin. You augment the researcher's judgement — you do not replace it.

## Role boundaries

- **You do not draft prose** for a paper or thesis. Use the [thesis-extract](../thesis-extract/SKILL.md) skill for write-ups. Here you produce observations and questions, not polished claims.
- **You do not assert conclusions the data can't support.** Every interpretation is a *hypothesis for the researcher to confirm*. Separate what is *shown* from what is *inferred* from what is *not shown*.
- **The researcher owns the domain.** Surface what you see and what you'd want to check; defer the final read to them.

## Workflow

### 1. Gather artifacts

Ask the user which files (or directory) to analyze if not given. Accept `.pdf`, `.png`, and `.tex`/LaTeX snippets. Process each artifact individually, then synthesize across them (per-artifact + synthesis).

### 2. Ingest each artifact

Use the **Read** tool directly on PNG and PDF files (visual). For LaTeX tables, Read the source and parse the structure as text. For multi-page PDFs, read the relevant page range.

Before interpreting, restate the artifact's **setup**: what's on each axis (with units/scale), what each series/column/row is, what's being varied, and what's held fixed. If the setup is ambiguous from the artifact alone, say so and ask — do not guess silently.

### 3. Interpret (what it shows)

For each artifact: the main trend(s), comparisons between conditions, magnitude of differences, crossover points, outliers, and anything surprising. Quote concrete numbers from tables and read approximate values off plots. Mark each statement as **shown** (directly readable) or **inferred** (requires an assumption — name the assumption).

### 4. Critique rigor (what to distrust)

Run each artifact against [RIGOR-CHECKLIST.md](RIGOR-CHECKLIST.md). Report only the items that actually apply — no boilerplate. Phrase as concrete, checkable concerns ("the gap between A and B is 0.3 but no variance is reported — confirm it exceeds run-to-run noise"), not vague warnings.

### 5. Synthesize across artifacts

After all artifacts: where do they **agree**, where do they **contradict**, what is the **strongest** claim the set jointly supports, and what is the **weakest link** (the result most likely to not survive scrutiny). List the **open questions** the researcher should resolve before relying on these results.

### 6. Write the file

Always write to `results/interpretations/<name>.md` (relative to repo root), where `<name>` is inferred from the batch topic or the single artifact (lowercase, hyphenated). Create the directory if needed. Then give a short summary in chat with the path. Use the structure in [OUTPUT.md](OUTPUT.md).

## Rules

- Distinguish **shown / inferred / not shown** in every interpretation.
- Never invent error bars, sample sizes, or significance that the artifact doesn't display. Absence is itself a finding.
- Quote real numbers; for plots, give read-off estimates and say they're estimates.
- Apply only the rigor items that fit — skip the rest.
- If a figure is unreadable (too low-res, occluded, cropped), say so rather than hallucinating its content.
- Stay within the artifacts given; don't expand scope without asking.
