# Store format

One file per subsystem at `~/.claude/learn-codebase/<repo>/<topic>.md`.

- `<repo>` — the target repo's directory basename (e.g. `django`, `my-service`).
- `<topic>` — the named subsystem, dash-cased (e.g. `job-scheduler`, `auth`).

## Layout

```markdown
---
repo: my-service
remote: git@github.com:acme/my-service.git   # tiebreaker if two repos share a basename
topic: job-scheduler
last-reviewed: 2026-08-07                     # read from context at session start
---

# job-scheduler

## Concepts

- [solid]  Jobs are enqueued via `Scheduler.enqueue` (scheduler/core.py:41)
- [medium] Retries use exponential backoff (scheduler/retry.py:88) — user fuzzy on the cap
- [weak]   Dispatch goes through the event bus, NOT a direct call (scheduler/bus.py:12)

## Diagrams

<!-- grounded ASCII saved from explain-visually during a structural reveal -->
​```
Scheduler.enqueue ──▶ EventBus.publish ──▶ Worker.consume
   (core.py:41)         (bus.py:12)          (worker.py:57)
​```
sources: scheduler/core.py:41, scheduler/bus.py:12, scheduler/worker.py:57

## Notes

Free-form: open questions, things to revisit, links to related topics.
```

## Rules

- **Confidence** is coarse: `weak` / `medium` / `solid`. No numeric scores, no ease
  factors, no computed due dates — this is "resurface what's shaky," not SM-2.
- Each concept line anchors to `file:line` so it stays verifiable and the user can
  click through on review.
- Update confidence and `last-reviewed` at the end of every session.
- If the target code has changed materially since `last-reviewed`, note it — a
  `solid` concept may have gone stale.
