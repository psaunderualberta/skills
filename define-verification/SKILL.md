---
name: define-verification
description: Interview the user to author a durable, project-specific verification policy — a change-type→method routing table written into CLAUDE.md — and draft any enforcement hooks. Deliberate one-time-ish setup ritual, hand-invoked.
disable-model-invocation: true
argument-hint: "(optional) area or change types to focus on"
---

# Define Verification

Boris Cherny calls this the single most important practice: **give Claude a durable, domain-specific way to verify its own work** ("this 2-3x the quality of final results"). This skill runs a setup ritual that decides what *"verified"* means in **this** project and records it durably.

## Boundary with `/verify` and `tdd` (read first)

- **`/verify`** *runs* verification for a given change and bootstraps a project verify skill (the *runner* — how to drive this app). It is change-scoped and reactive.
- **`define-verification`** (this skill) *authors the policy*: the change-type→method routing that says what verification each kind of change requires. It does **not** write the runner — it defers execution to `/verify` and references it.
- **`tdd`** is a specific test-first loop, orthogonal to both.

You produce the **policy**; `/verify` owns the **runner**; `update-config` installs any **hooks**. Three tools, three non-overlapping outputs.

## Ritual

### 1. Inspect the repo first (do not ask what you can find)

Ground the interview in reality before asking anything. Detect:
- test runner / command, language, framework;
- whether a UI surface exists, a migrations dir, a CLI/API entrypoint, existing CI config;
- a formatter/linter (candidate for a PostToolUse hook);
- whether a `/verify`-bootstrapped project verify skill already exists.

### 2. Interview using the inline scaffolding

Use these menus only to *prompt* the conversation; fill the table from what step 1 found and confirm gaps with the user.

**Verification surfaces** (from Boris's page): bash tests · test suite · simulator · browser (Chrome extension) · mobile simulator · computer use · external/staging step · manual smoke procedure.

**Starter change-type taxonomy:** logic/backend · schema/migration · UI · CLI or API contract · docs/config-only.

### 3. Apply the two-bar fallback gate

Only build a policy if the project clears **at least one** bar:
- **≥2 change-types** that need *materially different* verification (real branching in the table); **or**
- **≥1 verification surface beyond** "run the default test command" (simulator, browser, computer-use, staging, manual smoke).

If it clears neither (just "run `pytest`/`npm test` and you're done"), **bail**: write one durable line into CLAUDE.md —
`Verification: run \`/verify\` before committing; no domain-specific policy needed.` — and stop. This records the question was considered and closed.

### 4. Author the `## Verification` section in CLAUDE.md

Three parts (keep it tight — a table plus a few lines, not paragraphs):

1. **Routing table** — `Change type | What "verified" means | How to run it`. The *how* column points at the `/verify` project skill or a concrete command, never prose.
2. **Surfaces line(s)** — name the non-obvious verification tools this project uses, so an agent knows a surface exists before consulting the table.
3. **Hooks note** — one line stating what's auto-enforced (only if hooks were installed in step 5).

If gaps remain, add a **nudge line** here: `No policy yet for <change types>; run \`/define-verification\` to author one.` — the only always-on surface we own (`/verify` is compiled and cannot nudge).

See [EXAMPLES.md](EXAMPLES.md) for a full worked run.

### 5. Draft hooks, then delegate to `update-config` (opt-in, grounded)

Surface hook candidates **only when the repo warrants them**:
- **PostToolUse** auto-format/lint — *if* a formatter/linter was detected.
- **Stop hook** running a verification agent — *if* there's a cheap-enough smoke/test command to justify it.

When one fits, **draft the exact `settings.json` snippet** (concrete matcher + command), show it, then instruct invoking **`update-config`** to install it. **Never write `settings.json` directly** — only the harness can. Record whatever is installed in the step-4 hooks note.
