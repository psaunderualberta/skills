# Store format

One file per subsystem at `~/.claude/learn-codebase/<repo>/<topic>.md`.

- `<repo>` — the target repo's main checkout name (e.g. `django`, `my-service`).
  SKILL.md gives the command.
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
- [weak]   Failed jobs go to the dead-letter queue (scheduler/retry.py:120)
  level-set: 2026-08-07 (revealed)
  misconception: the retry loop has a max count, so a job must be dropped after it

## Diagrams

<!-- grounded ASCII saved from explain-visually during a structural reveal -->
​```
Scheduler.enqueue ──▶ EventBus.publish ──▶ Worker.consume
   (core.py:41)         (bus.py:12)          (worker.py:57)
​```
sources: scheduler/core.py:41, scheduler/bus.py:12, scheduler/worker.py:57

## Next session

- Re-test: dead-letter queue (revealed)

## Notes

Free-form: open questions, things to revisit, links to related topics.
```

## Rules

- **Confidence** is coarse: `weak` / `medium` / `solid`. No numeric scores, no ease
  factors, no computed due dates — this is "resurface what's shaky," not SM-2.
- Each concept line anchors to `file:line` so it stays verifiable and the user can
  click through on review.
- Update confidence and `last-reviewed` at the end of every session.
- `misconception: <the user's reasoning>` records the reasoning behind a wrong answer.
- `(revealed)` means the user did not find the cause without help. Remove it after a
  later hit.
- `level-set: <date>` is the day the concept was added or last changed level. A line
  without one gets today's date the first time it is graded.
- `## Next session` lists what to re-test first. Read it before `## Concepts`. The
  user also writes plans there, so never replace the section. Remove the items you
  re-tested, add new items, and keep the rest. A `build started at` line is not a
  re-test item.
- If the target code has changed materially since `last-reviewed`, note it — a
  `solid` concept may have gone stale.
