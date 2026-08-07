---
name: consolidate-memory
description: Groom the session's memory/ dir and MEMORY.md index — merge duplicate memories, flag contradictions for you to resolve, and repair the index and broken [[links]]. Like a "dream" pass that consolidates and dedupes accumulated memory. Invoke by hand (/consolidate-memory); not auto-triggered.
disable-model-invocation: true
argument-hint: "(optional) type or topic to focus on"
---

# consolidate-memory

A periodic grooming ritual for the file-based memory system — the agent's analogue of REM sleep's "dream" pass: read everything that has accumulated, **merge true duplicates**, **flag contradictions** for a human to settle, and reconcile the index. Boris Cherny's page describes auto-dream as memory that "consolidates, dedupes, cleans" itself; this is the hand-invoked version of the consolidate-and-dedupe half.

The memory system this grooms is described in the harness memory rules: one fact per file under `memory/`, frontmatter (`name`, `description`, `metadata.type ∈ {user, feedback, project, reference}`), `**Why:**`/`**How to apply:**` bodies for `feedback`/`project`, `[[name]]` cross-links, and a `MEMORY.md` index of one-line pointers.

## Boundary (read first)

- **`capture-correction`** *writes* new memories/rules from a correction. This skill never creates net-new facts — it only reorganizes what already exists.
- **Non-goals — explicitly out of scope:** pruning stale/wrong memories, normalizing format/quality, and splitting overloaded files. Deleting a *wrong* memory is a human call (or belongs to the manual delete path); this skill only deletes a duplicate whose content is fully absorbed by its survivor.

## Safety model (tiered)

- **Safe ops auto-apply:** rebuilding the `MEMORY.md` index, repairing broken `[[links]]`.
- **Destructive ops (merges) require confirmation.** Each merge is proposed with its **full blast radius** and nothing is deleted until the user approves the batch.

## Ritual

### 1. Load the memory dir

Find the session memory dir (`…/memory/`). **If it is missing or has no `*.md` files, say so and exit** — there is nothing to consolidate. Otherwise read every memory file and the current `MEMORY.md`. If an argument was given, restrict attention to that `type` or topic.

### 2. Cluster by underlying fact

Read for meaning, not string similarity. Group files that assert the **same underlying fact** (regardless of `type`). A cluster of 2+ is a merge candidate. A single file stands alone.

### 3. Separate merges from contradictions

For each multi-file cluster, decide:
- **Duplicate** — the files say the *same* thing (one may be a strict superset). → merge candidate (step 4).
- **Contradiction** — same topic, *disagreeing* content. → **flag only.** Report the pair and stop there; never edit, pick a winner, or delete. Resolving it is the user's call.

### 4. Propose each merge with its full blast radius

For every merge, present — before touching anything:
- the **survivor**: keep the **clearer of the existing slugs** (never coin a new one); show the merged body, which unions the `[[links]]` and keeps the most-informative content (superset of each source's `**Why:**`/`**How to apply:**`);
- the **file(s) to delete**;
- every **inbound `[[link]]`** elsewhere in memory that points at a deleted slug and will be **repointed** to the survivor.

The user approves the batch. Only then delete the losers, write the survivor, and repoint the inbound links. The repoint rides *with* the merge (it is a consequence of the deletion), not as a silent afterward step.

### 5. Reconcile safe ops (auto)

After merges (or if there were none):
- **Repair broken `[[links]]`** — any link whose target slug has no file (and isn't resolved by a step-4 repoint) is reported; obvious slug typos are fixed.
- **Rebuild `MEMORY.md`** — one `- [Title](file.md) — hook` line per surviving memory, dropping pointers to deleted files and adding any missing ones.

### 6. Report

Summarize: N merges applied, M contradictions flagged (with the file pairs, for the user to resolve), links repaired, index reconciled. The flagged contradictions are the only follow-up the user owns.
