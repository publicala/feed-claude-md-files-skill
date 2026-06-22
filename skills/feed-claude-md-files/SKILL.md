---
name: feed-claude-md-files
description: >
  Surfaces recurring patterns in recent commits and in-session corrections, proposes them as new CLAUDE.md rules in the right file (root or scoped subdir), and writes them only after approval. The inverse of bake-claude-md-files.
user-invocable: true
disable-model-invocation: true
---

Read recent git history (commits + diffs), the current conversation's user corrections and feedback, and all existing CLAUDE.md files in the project.

Cluster recurring patterns into candidate rules. A pattern is anything that recurs: the same correction asked for twice, the same kind of edit across multiple commits, the same nit, the same naming or style choice, the same architectural decision applied repeatedly.

For each candidate, decide:

- Whether it duplicates an existing CLAUDE.md rule (drop)
- Whether it would be better expressed as a tooling check (lint, static analysis, CI, hook) — if so, propose deferring to `/bake-claude-md-files` instead of writing prose
- Which CLAUDE.md file it belongs in: a scoped subdir (preferred when the pattern only applies there) or root (only when the rule genuinely cuts across the project)

Use `AskUserQuestion` to propose each rule with:

- The exact rule text
- The target file path (suggested; let the user pick another)
- Optionally: defer-to-bake instead of writing prose

Only write after approval. Create the target file if it does not exist. Append under an appropriate heading; never duplicate or near-duplicate an existing rule.

## Signal priority

1. **Current conversation corrections** — strongest. Direct "no, do it like X" / "stop doing Y" / "always do Z" carries explicit intent.
2. **Recent commit diffs** — next. If the same kind of change recurs (consistent param ordering, error-handling shape, test layout, commit-message style), it is a pattern.
3. **Auto-memory feedback files** — read-only input. Look for `feedback*.md` (and any file with `type: feedback` in its frontmatter) under `~/.claude/projects/*/memory/` — both the user-home-encoded directory and the project-cwd-encoded directory if present. Treat these like prior corrections but with lower confidence (they may be older or stale). Skip silently if none exist.
4. **Existing CLAUDE.md files** — used to deduplicate and to pick the right home for new rules, not as a source of new ones.

## Scope: project files only

This skill writes **only** to CLAUDE.md files inside the current project tree. Never write to:

- `~/.claude/` — global skills, settings, or user memory
- `~/.claude/projects/*/memory/` — auto-memory files (read-only input)
- Any path outside the project root

Reads from `~/.claude/projects/*/memory/` are fine; writes only ever land in the project's root `CLAUDE.md` or a scoped subdir `CLAUDE.md`.

## What to skip

- One-off corrections that look situational
- Rules better expressed as automated checks → point at `/bake-claude-md-files`
- Anything already covered by an existing CLAUDE.md rule
- Style preferences already enforced by linters/formatters in the repo

## Targeted vs root

Default to the smallest scope that still captures the rule. Promote to root only when the rule genuinely cuts across the project.

Examples:

- React component pattern → `src/components/CLAUDE.md`
- Eloquent model convention → `app/Models/CLAUDE.md`
- Test layout rule → `tests/CLAUDE.md`
- Commit message style → root `CLAUDE.md`
- Cross-cutting safety rule (e.g. "never log PII") → root `CLAUDE.md`

## Rule writing

- Lead with the rule, imperative voice
- One short paragraph, or a tight bullet list
- Add a `Why:` line when the reason is non-obvious — future-you will thank you when judging edge cases
- Match the existing CLAUDE.md voice in the file you're appending to

## Pairs with bake

`feed` and `bake` are a loop:

- `feed` adds prose rules from observed patterns
- `bake` converts crystallized prose rules into tooling and removes them from CLAUDE.md

Run `feed` after a working session; run `bake` once enough rules have accumulated to be worth automating.
