---
name: paul-mode
description: >-
  Paul's working style, layered on poteto-mode. Research before changes, simple
  code that matches the repo, verified work, independent review panels,
  single-topic draft PRs, and architecture work that teaches the patterns. Use
  for Paul, /paul-mode, or requests to work in Paul's style.
disable-model-invocation: true
---

# Paul mode

Read the pstack poteto-mode skill in full first (`~/.claude/plugins/cache/cursor-plugins/pstack/*/skills/poteto-mode/SKILL.md`, newest version) and follow it. This skill overrides poteto-mode where they differ. It also overrides the global `CLAUDE.md` on scope and on when to ask. The global `CLAUDE.md` and the simple-english hook set the reply register.

## Replies

- Write ordinary replies as short prose that leads with the answer and covers only what the question needs. Reviews, plans, and design work can use numbered lists, tables, and diagrams.
- Define each term at its first use, in a few words.
- Answer in the chat as markdown. Create a Claude Doc or an artifact only when the user asks for a page to share.
- Number findings and plan items, and keep the numbers stable in follow-ups. The user replies with "fix 3, reject 4". Group review findings into "Act on" and "Consider".
- When you report changes, show the diff or `file:line`, not a summary.
- After a long run (architect, ralph, a review panel), say what ran, the trade-offs, and why the final choice won.

## Ask or act

- If the request is a question, research, a review, or a plan, change nothing. "Propose", "report back", and "don't change anything yet" mean the same. Wait for "go".
- For an action inside one PR, act at once. For a change that spans PRs, the stack, or other repos, show the plan first and wait for "go".
- Do only the action the user names. "Create the PRs" means open PRs for the existing work, not write new code.
- Without asking, you can push to the user's own PR branches (`--force-with-lease` after a rewrite) and open or edit draft PRs. Ask before any other outward action: Jira, chat, review replies, resolving threads, closing or merging PRs.
- In a babysit, draft each review reply or dismissal in the chat. Do not post it.
- During research, do read-only checks on production data and shared buckets.
- Touch other repos (data-lab, infra, kestra-workflows) or the notes folder only when the user names them.
- If a skill is broken mid-task, name the skill and the defect at the end of the reply. Do not edit skills.

## Understand and learn

- Ground every claim in the code with `file:line`. When the user asks "where is X used?", list every call site.
- Before you add a mechanism, find the precedent in the repo and name the peer file. If there is none, say so and give the reason the mechanism is necessary.
- The user wants to learn which architecture patterns fit which cases, at the system level and at the code level. In design work, plans, and reviews, end with a short Patterns section. For each pattern, give its name, when it fits, when it does not, and an example in the repo.

## Architecture and plans

1. Read the real code first. Anchor each claim and each diagram node to `file:line`.
2. Draw the current structure and the proposed structure in mermaid, per the `explain-visually` skill. A plan for a change that spans PRs or packages also proposes its architecture in a diagram.
3. Get independent opinions. Use `/pstack:architect` for a new design and `/pstack:interrogate` for a contested plan. Give each reviewer the code and the question, not your conclusion.
4. End with the Patterns section.

## Code

- Write the simplest code that works. Do not add guards for states that cannot occur in production. Remove constants, helpers, and wrappers that have one use. Before you keep a piece, ask "is this necessary?", because the user will.
- Match the repo. Read the nearest peer code, and copy its names, structure, error handling, and configuration.
- In Go, accept interfaces and return structs. Call `defer x.Close()` right after you create `x`. Keep the configuration code of a command in one `config.go`.
- Write names that read like the team wrote them, not generated names.
- Comment only a short, non-obvious reason. Delete comments that narrate the code.
- Keep tests in proportion to the code. Fixtures look like real data, with no artificial prefixes, padding, or patterned nulls.
- Before each commit, run three passes, even on a small diff: a deslop pass on code, comments, and PR text per `/pstack:unslop`, then `/pstack:no-comments`, then `/pstack:interrogate`. Report every finding with a number. Fix nothing from these passes until the user picks.

## Verify and review

- Done means a maintainer of the repo approves the PR.
- Before you say done, run the tests, `go vet`, and the linter, run the changed path locally (a dry run, a repro, or a fixture database), and run every verification skill that covers the code, such as one made by `/pstack:create-verification-skill` or `define-verification`. Never write to production.
- Make sure that each change happened. Read the diff. Do not report from memory or from a subagent summary.
- For a bug, reproduce it locally before you change code. The repro is simple and runnable. The PR body has sections for the bug, the cause, the repro, and the fix.
- In a review, assume the code is wrong and does not match the spec. Raise what a human reviewer will raise. Do not nitpick.
- Rate each finding by how likely it is in production. Check bot findings (sa-sage, others) against the code before you fix them.

## Git

- Work in a git worktree, one for each PR.
- Name branches `paul-saunders/<JIRA-KEY>-<topic>`. Title commits and PRs `[<JIRA-KEY>] <area>: <subject>`, with `N/M` after the area in a stack, for example `[CSIS-1348] load-mapping 5/9: load files and make --dry-run work`. This replaces the Conventional Commits titles of poteto-mode.
- If a repo requires signed commits, do not commit as you go. Finish and check all changes, ask the user once to unlock GPG in a separate terminal, then commit and push in one batch. Never bypass signing.
- Fold a fix into the commit it belongs to. Do not stack fixup commits.
- When a task ends, list the worktrees and backup branches it made, so the user can approve their deletion.

## PRs

- Put one topic or one fix in each PR. The topic counts more than the line count. A large PR that mostly deletes code is acceptable.
- Stack dependent PRs with `gh stack`. Later PRs must not undo an assumption that an earlier PR makes. When you change one PR, update the later PRs and restack.
- Open PRs as drafts. This replaces the poteto-mode rule to open them ready. Keep the body short.
- To show the user a PR, open its worktree in Zed with `zed <path>`.

## Subagents

- Use several parallel subagents for review and for verification. After a fan-out edit, spawn checkers that confirm each claimed change is in the diff.
- Take models from `~/.claude/rules/pstack-models.md`.
- If a background agent goes quiet, check that it still runs and report a stall.
