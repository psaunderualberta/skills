---
name: capture-correction
description: Turns a correction the user just made into a durable rule and routes it to the right home — project CLAUDE.md, personal memory, a new skill, or a hook. Refuses one-off or low-signal corrections unless the user overrides, merges duplicates, and surfaces conflicts. Use when the user signals intent to capture a correction as a lasting rule — e.g. "remember this", "make that a rule", "add that to CLAUDE.md", "don't do that again", "note that for next time" — or runs /capture-correction. NOT for ordinary task reminders like "remember to deploy after"; only for turning a correction of the agent's behaviour into a rule that should persist across sessions.
---

# capture-correction

A routing ritual. It does **not** store rules itself — it decides the correct durable home for a correction and delegates. Keep it lightweight: gate → draft → confirm → write → report. Never commit or push.

## 1. Gate (refuse by default)

Not every correction deserves a durable rule — rule-bloat pollutes every future session's context. Capture **only if all three** hold:

1. **Recurs** — plausibly happens again, not a one-off typo or thinko.
2. **Generalises** — states a principle, not just "this one line".
3. **Not already covered** — no existing rule/memory says this.

If it fails any test, **decline**: name which test failed and say *"say 'capture anyway' to override."* On override, capture and acknowledge: *"Capturing anyway per your override."*

Handle multiple corrections in one turn independently — gate and route each on its own.

## 2. Draft the rule

Infer a draft from the recent turns (what the agent did + how the user corrected it); don't make the user phrase it. Then show it for a one-line confirm/edit.

- **Narrowest useful scope** + always a **why** — the rationale is the anti-misfire guard, so future sessions apply the rule only where it holds.
- User's invocation args steer the draft when given.

## 3. Route to one home

Check in order — stop at the first match:

| # | If the rule is… | Home | Action |
|---|---|---|---|
| 1 | deterministic & should always happen mechanically (auto-format, always-lint) | **hook** | tee up [update-config](../update-config) |
| 2 | a repeatable multi-step procedure | **skill** | tee up [write-a-skill](../write-a-skill) |
| 3 | a project-specific convention/fact/constraint | **CLAUDE.md** | write directly |
| 4 | a personal, cross-project preference | **memory** | write directly |

**Tie-breaker** (3 vs 4): project value wins — *"would a teammate on this repo benefit?"* → CLAUDE.md; *"is this just how I work everywhere?"* → memory. If genuinely 50/50, ask.

**No project CLAUDE.md** (not in a repo): fall back to memory. Nearest CLAUDE.md = closest one walking up from cwd.

## 4. Dedup & conflict

Scan the chosen home **before** writing:

- **No overlap** → write.
- **Overlap / near-duplicate** → merge or refine the existing rule; don't add a second phrasing.
- **Direct conflict** (contradicts an existing rule) → **stop and surface it**: show old vs new, ask to replace / re-scope both / keep both. Never auto-resolve.

## 5. Write (delegation split)

- **Light homes — finish inline:**
  - **CLAUDE.md** — concise imperative rule + short rationale, in (or creating) a "Conventions" / "Corrections" section. Edit only; don't commit.
  - **memory** — one file per fact under the memory dir with the memory type that fits (usually `feedback`; use `reference`/`project` for a pure environment/tooling fact), body followed by `**Why:**` and `**How to apply:**` lines; add the one-line `MEMORY.md` pointer.
- **Heavy homes — route, don't author inline:** write a seeded brief (the correction + drafted rule + why) and hand to `write-a-skill` / `update-config`. Don't hand-edit `settings.json` or author a whole skill mid-ritual.

## 6. Report & rewind nudge

- One line: what rule was written, to which home, and what changed on a merge/conflict.
- Nudge to rewind **only** if a failed attempt is still in context *and* hasn't been superseded by completed work: *"lesson captured to `<home>`; you can /rewind to drop the failed attempt."* If the attempt was already resolved (fixed, committed, moved past), skip the nudge — rewinding would destroy good work. Never auto-rewind — it's destructive and you can't judge the other turns.
