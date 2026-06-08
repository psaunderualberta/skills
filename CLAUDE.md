# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal collection of Claude Code **skills** (origin: `git@github.com:psaunderualberta/skills.git`). Each top-level directory is one skill, loaded by Claude Code from `~/.claude/skills/`. There is no build, no test suite, and no package manifest — content is markdown (plus occasional helper scripts) consumed directly by the agent harness.

## Skill anatomy

Every skill lives in its own directory with a required `SKILL.md` that begins with YAML frontmatter:

```yaml
---
name: skill-name              # must match the directory name
description: <what it does>. Use when <specific triggers>.
---
```

The `description` is the **only** text the agent sees when deciding whether to load an auto-invoked skill, so it helps for it to state the capability and the trigger conditions (keywords, contexts, intents). `write-a-skill/SKILL.md` has the canonical guidance (max 1024 chars, third person, a "Use when …" clause). Treat these as defaults to reach for, not gates — command-style skills meant to be invoked by hand (`disable-model-invocation: true`, often with an `argument-hint`, e.g. `teach/`) legitimately skip the "Use when …" trigger clause, since the agent never auto-selects them.

Optional sibling files in a skill directory:
- `REFERENCE.md` / `EXAMPLES.md` / topic-named `.md` files — progressive disclosure for content that would push `SKILL.md` past ~100 lines (see `tdd/` which splits into `interface-design.md`, `refactoring.md`, `deep-modules.md`, `mocking.md`, `test.md`; and `improve-codebase-architecture/REFERENCE.md`).
- `scripts/` — utility scripts for deterministic operations.

Aim to keep `SKILL.md` reasonably short (~100 lines is a good target); link out to siblings rather than inlining long content when it grows past that.

## Conventions worth following

These are patterns that have worked well here, not hard rules. Imported or command-style skills may deviate, and that's fine — apply judgement.

- **Discriminating triggers help auto-invoked skills.** For skills the agent selects on its own, explicit triggers reduce misfires — e.g. `tdd-coach` carries a "ONLY when explicitly invoked" clause and names the sibling `tdd` that handles the non-coached path. When two auto-invoked skills overlap, disambiguating in both descriptions is worthwhile. (Command-style skills invoked by hand don't need this.)
- **Prime-directive skills declare role boundaries up front.** `tdd-coach` opens with "You do not write tests or implementation code" — sets the contract before workflow steps. A good pattern to mirror for skills that constrain agent behavior rather than expanding it.
- **Output paths tend to be part of the contract.** Skills that produce artifacts often name the destination in the description (`thesis-extract` → `thesis/notes/<topic>.md`; `ubiquitous-language` → `UBIQUITOUS_LANGUAGE.md`). Where downstream users rely on a path, keep it stable.
- **Cross-skill links use relative paths** (e.g. `[tdd](../tdd/SKILL.md)`).

## Authoring workflow

When adding or editing a skill, `write-a-skill/SKILL.md` is a useful guide, and its review checklist (description has triggers, `SKILL.md` under 100 lines, no time-sensitive info, concrete examples, references one level deep) is worth a pass. Use it as a sanity check rather than a gate — skills imported from elsewhere can keep their own style.

## Repo-specific notes

- `.claude/settings.local.json` contains permission allowlists scoped to an *unrelated* project (`differentiable-privacy-percentages`) that this user works on. It is not configuration for this repo's content — leave it alone unless explicitly asked.
- There is no CI, linter, or formatter. Validation is by reading the markdown and confirming frontmatter parses.
