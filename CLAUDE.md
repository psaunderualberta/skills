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

The `description` is the **only** text the agent sees when deciding whether to load the skill — it must state the capability and the trigger conditions (keywords, contexts, intents). See `write-a-skill/SKILL.md` for the canonical rules (max 1024 chars, third person, "Use when …" clause).

Optional sibling files in a skill directory:
- `REFERENCE.md` / `EXAMPLES.md` / topic-named `.md` files — progressive disclosure for content that would push `SKILL.md` past ~100 lines (see `tdd/` which splits into `interface-design.md`, `refactoring.md`, `deep-modules.md`, `mocking.md`, `test.md`; and `improve-codebase-architecture/REFERENCE.md`).
- `scripts/` — utility scripts for deterministic operations.

`SKILL.md` should stay under ~100 lines; link out to siblings rather than inlining long content.

## Conventions enforced across skills

- **Triggers must be explicit and discriminating.** Skills like `tdd-coach` carry a "ONLY when explicitly invoked" clause and call out the sibling skill (`tdd`) that handles the non-coached path. When adding a new skill that overlaps an existing one, disambiguate in both descriptions.
- **Prime-directive skills declare role boundaries up front.** `tdd-coach` opens with "You do not write tests or implementation code" — sets the contract before workflow steps. Mirror this pattern for any skill that constrains agent behavior rather than expanding it.
- **Output paths are part of the contract.** Skills that produce artifacts name the destination in the description (`thesis-extract` → `thesis/notes/<topic>.md`; `ubiquitous-language` → `UBIQUITOUS_LANGUAGE.md`). Keep these stable; downstream users rely on them.
- **Cross-skill links use relative paths** (e.g. `[tdd](../tdd/SKILL.md)`).

## Authoring workflow

When adding or editing a skill, follow `write-a-skill/SKILL.md`. After drafting, run through its review checklist (description has triggers, `SKILL.md` under 100 lines, no time-sensitive info, concrete examples, references one level deep).

## Repo-specific notes

- `.claude/settings.local.json` contains permission allowlists scoped to an *unrelated* project (`differentiable-privacy-percentages`) that this user works on. It is not configuration for this repo's content — leave it alone unless explicitly asked.
- There is no CI, linter, or formatter. Validation is by reading the markdown and confirming frontmatter parses.
