# make-presentation — PptxGenJS recipes

Everything here targets the **browser bundle** of PptxGenJS so the user needs no Node/npm. The whole deck lives in one `presentation.html`; opening it triggers a `.pptx` download.

## 1. HTML skeleton (no install)

```html
<!doctype html>
<html>
<head><meta charset="utf-8"><title>Generate deck</title></head>
<body>
  <p>Your download should start automatically. If not, <button onclick="build()">click here</button>.</p>
  <!-- pin the version so the deck reproduces -->
  <script src="https://cdn.jsdelivr.net/npm/pptxgenjs@4/dist/pptxgen.bundle.js"></script>
  <script>
    function build() {
      const pptx = new PptxGenJS();
      pptx.defineLayout({ name: "W16x9", width: 13.333, height: 7.5 });
      pptx.layout = "W16x9";
      defineMasters(pptx);
      buildSlides(pptx);
      pptx.writeFile({ fileName: "talk.pptx" }); // browser → download
    }
    window.addEventListener("load", build);
  </script>
</body>
</html>
```

Coordinates are **inches**. 16:9 is 13.333 × 7.5.

## 2. Palette + master slides

Pick one ink, one accent, one tint. A safe academic-but-warm set:

```js
const C = { ink: "1F2933", accent: "2F6F4E", accent2: "C2553B", tint: "F4F1EA", muted: "7B8794", white: "FFFFFF" };
```

Define masters once, then every slide stays consistent:

```js
function defineMasters(pptx) {
  pptx.defineSlideMaster({
    title: "CONTENT",
    background: { color: C.white },
    objects: [
      { rect: { x: 0, y: 0, w: "100%", h: 0.12, fill: { color: C.accent } } },      // top accent rule
      { text: { text: "Talk title", options: { x: 0.5, y: 7.05, w: 9, h: 0.3, fontSize: 9, color: C.muted } } },
    ],
    slideNumber: { x: 12.6, y: 7.05, fontSize: 9, color: C.muted },
  });
  pptx.defineSlideMaster({
    title: "SECTION",
    background: { color: C.ink },
  });
}
const s = pptx.addSlide({ masterName: "CONTENT" });
```

## 3. Takeaway-headline content slide

```js
function takeawaySlide(pptx, headline, bullets) {
  const s = pptx.addSlide({ masterName: "CONTENT" });
  s.addText(headline, { x: 0.5, y: 0.4, w: 12.3, h: 1.0, fontSize: 28, bold: true, color: C.ink });
  s.addShape(pptx.ShapeType.line, { x: 0.5, y: 1.45, w: 3.0, h: 0, line: { color: C.accent, width: 2 } });
  s.addText(bullets.map(t => ({ text: t, options: { bullet: true } })),
            { x: 0.5, y: 1.7, w: 6.0, h: 4.8, fontSize: 16, color: C.ink, lineSpacingMultiple: 1.2 });
  return s;
}
```

## 4. Embedding result figures as base64

In the browser you cannot read disk paths — embed each figure as a data-URI. **You (the agent) read the figure file and inline its base64** when writing the HTML:

```js
s.addImage({ data: "image/png;base64,iVBORw0KGgo...", x: 6.8, y: 1.7, w: 6.0, h: 4.5, sizing: { type: "contain", w: 6.0, h: 4.5 } });
```

- **Always preserve the aspect ratio.** Pass `sizing: { type: "contain", w, h }` so the figure is scaled to fit the box without stretching or squashing — the original proportions are kept and the figure is letterboxed within `w × h`. Never set `w`/`h` to values that imply a different aspect ratio than the source without `contain`. If you know the source pixel dimensions, you can instead give a `w` and let `h = w * (srcH / srcW)` to size it exactly.
- **PDFs are not images** — PptxGenJS embeds PNG/JPG/GIF only. If a result is a `.pdf`, convert to PNG first (`pdftoppm -png -r 200 fig.pdf fig` or ImageMagick `convert -density 200 fig.pdf fig.png`) and embed that. Note this in the outline if conversion is needed.
- Keep the base64 inline but consider that many large figures make the HTML big — that's fine, it still opens.

## 5. Native charts (when you have numbers, not a figure)

Prefer this over an empty placeholder when raw data exists:

```js
s.addChart(pptx.ChartType.line,
  [{ name: "Method A", labels: xs, values: ysA }, { name: "Baseline", labels: xs, values: ysB }],
  { x: 6.8, y: 1.7, w: 6.0, h: 4.5, showLegend: true, legendPos: "b",
    chartColors: [C.accent, C.accent2], showTitle: false, valAxisTitle: "metric" });
```

Types: `line`, `bar`, `scatter`, `area`, `pie`. Always label axes; include error bars only if the data has them (don't fabricate).

## 6. Tables (results tables)

```js
s.addTable(
  [[{ text: "Method", options: { bold: true, fill: { color: C.tint } } }, { text: "Acc", options: { bold: true, fill: { color: C.tint } } }],
   ["Ours", "0.91"], ["Baseline", "0.84"]],
  { x: 0.5, y: 1.7, w: 6.0, fontSize: 14, border: { type: "solid", color: "DDDDDD", pt: 1 }, color: C.ink });
```

## 7. Speaker notes

Put the talking points behind each slide so the deck is presentable:

```js
s.addNotes("Lead with the motivation; the figure shows X; land on the takeaway in the headline.");
```

## 8. "Animation" / build effect (honest)

PptxGenJS has **no reliable per-element entrance animations**. To get a build/reveal feel:

- **Fragment technique**: emit N near-identical slides, each revealing one more bullet/element. Stepping through them looks like an incremental build. Factor the slide body into a function that takes `revealCount`.
- **Slide transitions**: set per-slide `transition` (e.g. `{ type: "fade" }`) where the running PptxGenJS version supports it; treat it as best-effort and never rely on it for meaning.

```js
function revealBuild(pptx, headline, bullets) {
  for (let k = 1; k <= bullets.length; k++) takeawaySlide(pptx, headline, bullets.slice(0, k));
}
```

## 9. Section dividers

```js
function sectionSlide(pptx, title) {
  const s = pptx.addSlide({ masterName: "SECTION" });
  s.addText(title, { x: 0.8, y: 3.0, w: 11.7, h: 1.5, fontSize: 40, bold: true, color: C.white });
  s.addShape(pptx.ShapeType.line, { x: 0.85, y: 4.4, w: 2.5, h: 0, line: { color: C.accent, width: 3 } });
  return s;
}
```

## 10. Equations (LaTeX or PowerPoint native — never ASCII)

Mathematical equations must render as real math, not plain text like `1/sqrt(2*pi)`. Two acceptable routes:

**A. PowerPoint-native equation (editable in PowerPoint).** PptxGenJS can emit Office Math (OMML) via the `addText` math option where supported, but the most reliable cross-version path is to author the equation as OMML and add it as a text run flagged as math. Keep equations short and prefer this when the user will edit the deck in PowerPoint.

**B. LaTeX rendered to an image (most portable).** Render the LaTeX to an SVG/PNG and embed it like any figure — this always displays correctly regardless of the PowerPoint version. Two ways to produce the image:

- **At authoring time (you, the agent):** render with a LaTeX/`dvisvgm` or `matplotlib` mathtext pipeline to a PNG, then base64-inline it (see §4). Best for fixed equations.
- **In the browser at build time:** load KaTeX/MathJax from a CDN, render the LaTeX to SVG, serialize it to a data-URI, and pass it to `addImage`. Example with MathJax:

```js
// <script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js"></script>
function eqImage(latex) {
  const svg = MathJax.tex2svg(latex).querySelector("svg").outerHTML;
  return "image/svg+xml;base64," + btoa(unescape(encodeURIComponent(svg)));
}
s.addImage({ data: eqImage("\\frac{1}{\\sqrt{2\\pi}\\sigma}e^{-\\frac{(x-\\mu)^2}{2\\sigma^2}}"),
             x: 1, y: 2, w: 5, h: 1.2, sizing: { type: "contain", w: 5, h: 1.2 } });
```

Render math white-on-dark on SECTION masters by setting the LaTeX/MathJax color so it stays legible. Whichever route, keep the equation's aspect ratio with `contain` (§4).

## Recommended slide arc

Title → Agenda → Motivation (why care) → Problem/gap → Method (1–3 slides) → Results (figure-forward, one finding each) → Limitations → Conclusion/takeaways → (optional) Backup. Keep one takeaway headline per slide throughout.
