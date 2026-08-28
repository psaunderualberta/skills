---
name: learn-codebase
description: Build and verify your mental model of an unfamiliar codebase through retrieval practice — you explain how a subsystem works, the agent reads the real code, probes the gaps, and persists your weak spots so they resurface next session. Use when the user wants to learn a codebase, be quizzed on how existing code works, test their understanding of a subsystem, or resume a previous learning session. Invoke by hand (/learn-codebase); not auto-triggered.
disable-model-invocation: true
argument-hint: "Which subsystem or area do you want to learn?"
---

# learn-codebase

Help the user build a durable mental model of code **they did not write**, through
retrieval practice. The user recalls first; you find the gaps against the real
source; the gaps persist and resurface later. This is the inverse of
[teach](../teach/SKILL.md) (which delivers knowledge *to* the user) and unlike
[grill-me](../grill-me/SKILL.md) (which has no persistence).

**You do not lecture.** Your job is to elicit the user's model, compare it to ground
truth you read from the code, and probe the delta — one question at a time.

## The store

Per-subsystem state lives at `~/.claude/learn-codebase/<repo>/<topic>.md`, where
`<repo>` is the target repo's directory basename and `<topic>` is the named
subsystem. This is the user's personal mental-model tracker — it deliberately does
**not** live in the target repo (never dirty a repo you don't own).

At session start, read the current date from context and record it as the
`last-reviewed` marker. Each concept carries a coarse confidence: `weak` / `medium`
/ `solid`. See [store-format.md](./store-format.md) for the exact layout.

This is **not** true spaced-repetition scheduling (no intervals/ease factors) — it
is "resurface what you're shaky on." Do not over-claim otherwise to the user.

## The loop

1. **Scope.** Confirm the subsystem (the argument, or ask). A topic is a *subsystem*
   ("the job scheduler"), coarser than a file, narrower than the whole repo. If a
   `<topic>.md` already exists, load it and tell the user their weak spots — resume,
   don't restart.
2. **User explains first.** Ask the user to explain their current understanding in
   their own words *before* you show any framing. This is genuine recall, not
   recognition — do not read them the answer first.
3. **You read the code.** Now trace the real source for that subsystem. Build a
   ground-truth checklist: public entry points, key invariants, surprising control
   flow. Ground everything in `file:line` — never quiz from parametric guesses.

   **Never cite a bare `:NNN`.** A line number without a path is unusable to someone
   who does not yet know the codebase — which is, definitionally, the person you are
   teaching. Write `user_snapshot_ingester.go:547`, not `:547`. Carry the filename on
   *every* reference, even when it repeats: a reveal usually spans several files, and
   the reader cannot infer which one is "current". Re-anchor with the full path the
   first time each file appears in a message, and again whenever you switch files
   mid-paragraph.
4. **Probe the gaps, one question at a time.** Order:
   1. **Contradictions** — things the user stated that the code contradicts (active
      misconceptions; highest priority).
   2. **Silent omissions** — load-bearing things the user didn't mention.
   3. **Depth probes** — where the user was vague, push for the *why*.
5. **Reveal.** After each probe, reveal the correct understanding grounded in the
   code. When the gap is **structural** (how components connect, how data/control
   flows), use [explain-visually](../explain-visually/SKILL.md) to draw a grounded
   ASCII diagram of the correct model, and **save that diagram into `<topic>.md`** —
   this is a deliberate override of explain-visually's ephemeral default, legitimate
   because the user is in a persistent-learning context. For non-structural gaps
   (wrong invariant, missed edge case), a prose reveal is better.

   The bare-`:NNN` ban from step 3 applies hardest here. Under reveal, you are moving
   fast across files you have read and the user has not, and dropping the path is the
   default failure. Same rule in the store: every concept line in `<topic>.md` needs a
   path, so it still resolves next month.
6. **Update the store.** Record/adjust each concept's confidence and the
   `last-reviewed` date.

## Selecting what to quiz on resume

When resuming an existing topic, surface `weak` concepts first, then `medium`, and
re-test `solid` ones only occasionally (or when the code has changed since
`last-reviewed`). Always still open with step 2 (user explains) for the concept in
question — recall before reveal, every time.
