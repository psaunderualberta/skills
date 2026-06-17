---
name: make-presentation
description: Build a research presentation from the current project — combing the README/docs, source code, result figures, and any paper/thesis notes — and emit a self-contained HTML file that generates a .pptx in the browser (PptxGenJS via CDN, no local install). Use when the user runs /make-presentation and wants a slide deck for a talk, lab meeting, or conference.
disable-model-invocation: true
argument-hint: "<minutes> <what the talk is about / audience>"
---

# make-presentation

Build a polished, academic-but-attractive research talk **from the project you are invoked in**. The output is a single `.html` file that, when opened in any browser, downloads a `.pptx`. No Node/npm install — PptxGenJS loads from a CDN and runs client-side.

The argument is a duration plus a free-text description, e.g. `15m a lab-meeting talk on our DP-percentages results`. Treat the description as the framing/audience and the minutes as the pacing budget.

## Process

Work in four phases. **Do not skip the outline sign-off (phase 3).**

### 1. Parse the argument
- Extract the duration in minutes (`15m`, `15 min`, `15`) and the free-text description.
- Derive a slide budget: **~1 content slide per minute** for a research talk, then add a title slide, an agenda/outline slide, and section dividers. So 15 min ≈ 15 content + ~4 structural ≈ 19 slides. Adjust down for very short talks (don't pad).

### 2. Gather material (comb the project)
Read, don't guess. Pull from:
- **Result figures/tables** — first check `CLAUDE.md` and any context/config files for a stated results location. **If none is documented, ask the user where result figures live** (and offer to record it in `CLAUDE.md`). Do not assume a path. Once known, look there for `.png`/`.pdf`/tables and any `interpretations/*.md`.
- **README, CLAUDE.md, `docs/`** — motivation, problem statement, method overview.
- **`thesis/notes/`, paper drafts** — existing framing and prior phrasing to stay consistent with.
- **Source code** — for method/contribution/architecture details not written up elsewhere.

Note which figures actually exist and their paths — you will embed them (see [REFERENCE.md](REFERENCE.md) on base64 images). If a claimed result has no figure, prefer a PptxGenJS native chart built from the underlying numbers over an empty placeholder.

### 3. Propose an outline, get sign-off
Present a slide-by-slide outline: for each slide give the **one-line takeaway headline** (the point of the slide) and what visual/evidence sits under it. Map it to the structure arc: Motivation → Problem → Method → Results → Conclusion, with section dividers. Confirm the slide count fits the minutes. **Wait for the user's approval or edits before generating.**

### 4. Generate the HTML deck
Write all files for **this** presentation into a folder **specific to this talk** — `presentations/<slug>/` at the project root, where `<slug>` is a short kebab-case name derived from the talk's topic/audience (e.g. `presentations/dp-percentages-lab-meeting/`). `make-presentation` may be run several times for different purposes, so each deck gets its own folder and never overwrites a previous one; if the chosen slug already exists, pick a distinct one (or confirm with the user). The deck itself is `presentations/<slug>/presentation.html`, and any supporting assets (e.g. PNGs converted from PDFs) live alongside it. Follow [REFERENCE.md](REFERENCE.md) for the PptxGenJS boilerplate, layout masters, palette, charts, tables, base64 image embedding, equation formatting, speaker notes, and the build-reveal technique. Then tell the user: *open `presentations/<slug>/presentation.html` in a browser; it downloads the `.pptx` automatically.*

## Design principles (academic, but pleasing)

- **One idea per slide**, fronted by a takeaway headline that states the conclusion — not a topic label ("DP noise dominates below ε=1", not "Results").
- **A real palette**, applied consistently via a master slide: one dark ink colour, one accent, one muted background tint — not black-on-white only. See [REFERENCE.md](REFERENCE.md) for a ready palette and `defineSlideMaster` usage.
- **Generous structure, not empty space**: section dividers, consistent margins, a footer with talk title + slide number, accent rules/shapes to anchor the eye.
- **Evidence-forward**: every results slide shows the figure/chart large; text annotates it, never competes with it.
- **Preserve figure aspect ratios**: never stretch or squash a figure to fill a box. Always embed with `sizing: { type: "contain" }` so the original proportions are kept (see [REFERENCE.md](REFERENCE.md) §4).
- **Format equations properly**: render mathematical equations with LaTeX or PowerPoint's built-in equation formatting — never as plain ASCII like `1/sqrt(2*pi)`. See [REFERENCE.md](REFERENCE.md) §10.

## Constraints
- Keep everything local: the only network dependency is the CDN `<script>` for the library; all content (text, base64 figures) stays in the subdirectory.
- Give each presentation its own folder (`presentations/<slug>/`) so repeated runs don't collide or scatter files across the project root; never overwrite a prior deck's folder.
- Pin the CDN version (e.g. `pptxgenjs@4`) so the deck reproduces.
- Never assume a results-figure location. Use the path documented in `CLAUDE.md`/context files, or one the user gives you; if neither exists, ask before generating.
