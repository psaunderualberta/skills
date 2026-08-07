---
name: explain-visually
description: Draw a grounded ASCII (or mermaid) diagram of an unfamiliar subsystem, protocol, or flow to aid on-the-fly comprehension — read the real code first, anchor every node to a file:line, and keep it throwaway. Use when the user asks to "diagram this", "visualize the architecture", "draw the request flow", "map how these modules connect", or "how does X flow through the system" — i.e. visual/structural understanding of code, right now. NOT for generic "explain this function" (that's plain answering) and NOT for a polished deck to show others (that is make-presentation, ../make-presentation/SKILL.md).
---

# explain-visually

Produce a **throwaway visual** that helps the user understand an unfamiliar subsystem, protocol, or flow *right now*. This is comprehension-on-the-fly, not a deliverable. The moment a diagram is meant to be *shown to other people*, that is [make-presentation](../make-presentation/SKILL.md)'s job — hand off, don't compete.

## Medium

- **ASCII is the default** and the always-available fallback — it renders in a plain terminal, in the transcript, everywhere, with zero dependencies.
- **Upgrade to mermaid only when it earns its keep** — the structure is complex enough that hand-drawn ASCII gets unwieldy (large state machines, dense entity/component graphs) *and* the user is reading somewhere mermaid renders. When in doubt, ASCII.
- **HTML is out of scope.** That is the boundary with `make-presentation`. Never emit an HTML artifact here.

## Grounding is mandatory (the cardinal rule)

A wrong diagram is worse than none — it teaches false structure with the authority of a picture. So:

- **Read before you draw.** Trace the real entry point and follow the calls in the actual source. Never draw from parametric guesses about how "a system like this usually works".
- **Anchor every node to a real symbol** — each box/participant names the file/function/module it represents, so the user can click through and verify.
- **Mark inference explicitly.** Anything you inferred rather than read (e.g. "probably retries here") is flagged as an assumption, not drawn as settled fact.
- **Be honest about gaps.** If a part couldn't be traced, label it "not traced" — never invent a plausible box to fill the hole.
- **Always emit a `sources:` list** under the diagram — the `file:line` references the diagram is built from, so it stays verifiable.

## Scope

Diagram **one bounded subject**: a single flow/protocol, a single subsystem and its immediate collaborators, or one module's internal structure. Never try to draw the whole repo as one diagram — that is a hairball, not comprehension.

When the request is **broad** ("visualize the architecture"), don't recurse into every file. Identify the **3–6 top-level components and draw the relationships between them** — one high-level component diagram. A broad ask usually also means the user is still forming their question, so treat the diagram as a springboard (see below).

## Choosing the diagram type

Let the content pick, using this mapping:

| Intent | Type |
| --- | --- |
| Protocol / request lifecycle / "how does X flow" | **sequence** (actors + ordered messages) |
| "How is this architected" / "how do these connect" | **component** (boxes + relationships) |
| "How does data move / get transformed" | **data-flow** |
| "What states can this be in" | **state** |

When ambiguous, default to **sequence for anything time/flow-shaped, component for anything structure-shaped.** State which type you chose and why in one line, so the user can override.

## After you draw (lightweight springboard)

Keep this self-contained — no heavy machinery:

- **Offer to zoom.** "Point at a box and I'll redraw that piece in detail" — this is just the skill re-running, focused on one node.
- **Stay in normal Q&A** about what you drew. That's just conversation, no special tooling.
- If the user signals they want a *sustained, multi-session* learning arc, drop a **one-line pointer**: for learning *this codebase* over time, `/learn-codebase` sets up recall practice with persistent weak-spot tracking; for a concept or skill, `/teach` sets up a workspace. Never auto-escalate into either.

## Persistence

**Ephemeral by default** — the diagram lives in the conversation; write nothing to disk. Do **not** proactively save (that's a deliberate act, and `make-presentation`'s territory). Only if the user explicitly asks ("save that", "drop it in a file"), write the raw ASCII/mermaid to a plain `.md` — no ceremony, no dedicated directory.
