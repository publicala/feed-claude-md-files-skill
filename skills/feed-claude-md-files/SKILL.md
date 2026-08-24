---
name: feed-claude-md-files
description: >
  Surfaces recurring patterns in recent commits and in-session corrections, proposes them as new CLAUDE.md rules in the right file, and writes them only after approval. The inverse of bake-claude-md-files.
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

When the candidate list is long (more than AskUserQuestion comfortably carries), first ask whether the user wants the proposals as an interactive artifact instead: a live doc (`capabilities: {artifact: {}}`) listing each candidate with its exact rule text, repo-relative target path, an approve checkbox, and a free-text input for rewording, plus a copy-decisions control as the fallback path back into the session.

Only write after approval. Create the target file if it does not exist. Append under an appropriate heading; never duplicate or near-duplicate an existing rule.

## Building the decision artifact

The artifact is a decision surface, not a document. The user often decides from a phone, so every item renders as a compact card: path, a one-sentence claim, one evidence line. Compact governs your own prose and never the source: text the user is approving (the line being cut, the rule being written) appears verbatim and whole, however long it runs.

- **Show the line, don't cite it.** Any claim about text the reader cannot see gets a collapsed "See the lines" block quoting the offending line verbatim in diff-removed styling, with the replacement (where one exists) in diff-added styling. The reader must never need the repo open to decide. Abridge very long lines with `[...]` and point at the full diff. Quote blocks hold file lines only: never explanation prose inside the block (that belongs in the card's claim or evidence line), and each quoted line carries its real line number in a muted, unselectable gutter.
- **A note field on every row.** Every decision row carries a free-text note field, always visible and in the tab order, never behind a disclosure the user must click first: opening a control per row breaks the review flow. Keep it one row tall and let it grow with content (`field-sizing: content`, plus `resize: vertical` as the fallback). Give every interactive control a visible `:focus-visible` outline so the whole page is walkable by keyboard. The user may have an opinion or question anywhere, including near-certain items.
- **Keyboard flow.** <kbd>j</kbd>/<kbd>k</kbd> walk the decision rows, <kbd>x</kbd> toggles the row, <kbd>n</kbd> focuses its note, <kbd>e</kbd> toggles its quoted lines, <kbd>Esc</kbd> leaves the note (document the keys on the page, hidden on touch widths). On wide viewports open the quote blocks by default, so deciding needs no clicks at all.
- **Open-in-editor links.** Every file reference gets a small open link built on the user's editor URL scheme, detected from the machine (`$EDITOR`, installed apps): `zed://file{abs}:{line}`, `vscode://file{abs}:{line}`, `cursor://file{abs}:{line}`, `phpstorm://open?file={abs}&line={n}`. Always absolute paths, so the links hold from any worktree, and percent-encode them (keeping `/` in the path, encoding the whole PhpStorm query value) so a `#`, `?`, `%`, or `&` inside a path cannot truncate the target. Evidence references carrying file:line link the same way. Use `target="_blank"`, and stop click propagation on these links in a capture-phase handler, since they sit inside the checkbox label and a click must never toggle the row. The link opens on the machine running the browser, so emit these only when the session runs on the user's own machine: a remote session (SSH, devcontainer, cloud sandbox) reads a different filesystem and a different set of installed editors, so omit the links there rather than pointing an absent editor at a path that does not exist.
- **Word-level diffs.** Pair adjacent removed/added runs (SequenceMatcher on tokens, similarity at or above 0.4) and highlight only the changed tokens. Render each diff line as a `display:block` span and join the spans with no separator: a newline between block spans inside a `<pre>` renders as a phantom blank line. Lint-enforced docs keep whole paragraphs on one physical line, so wrap with `white-space:pre-wrap`, `overflow-wrap:anywhere`, and a hanging indent.
- **Questions as options.** Render each open question as radio options with a "recommended" chip plus a free-text field, mirroring AskUserQuestion. A bare textarea is only for questions with no concrete options.
- **Sticky decision bar.** Section links, a changed-from-default counter, and the copy-decisions control stay reachable while scrolling.
- **A fallback that carries everything.** The copy-decisions control serializes every control on the page: each checkbox, the selected option of each question, and each free-text field. The pasted export is an accepted approval path, so any state it drops is a decision the session then applies wrongly.
- **Triage chips.** Label each diff row by the review effort it needs (cuts-only rows are skimmable, rewrites are worth opening, adds carry new lines) and provide an expand-all-diffs control.
- **Mobile pass.** Verify the page at around 390px width before publishing. Collapse method and other secondary sections by default.
- **Republish safety.** Viewer decisions reach the session only because the page is a live doc (`capabilities: {artifact: {}}`), which saves the DOM a viewer's gesture changed back into the served document. Before any republish, fetch the live artifact and compare its state against defaults. Carry any non-default state into the rebuilt HTML, or do not republish: republishing over live decisions destroys them. Publish without that capability and the state never leaves the viewer's browser, where the session cannot read it: then the clipboard export is the only path back, and republishing is off the table once the user starts deciding.
- **Generate, don't hand-edit.** Build the page from a data-plus-template script in the scratchpad so every iteration regenerates it whole.
- **End with the trigger.** Close the page by telling the user the exact phrase that resumes the session, such as "read the artifact and apply".

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
- Style preferences already enforced by linters/formatters in the repo
- Anything a fresh session derives with a few tool calls (setup commands, stack inventories, directory layouts): inventory, not instruction
- Conventions the codebase already demonstrates nearly everywhere: a new session copies its neighbors without being told; write the rule only where the dominant pattern is the wrong one

The last two reasons are claims about what a fresh session does, and the loaded model making them has already read everything it claims that session would derive. Before skipping a candidate for either, run a clean-context probe: give one fresh low-effort agent the task that surfaced the pattern, without the rule, and record whether it makes the mistake the rule would prevent. The probe decides the skip. The other reasons are checkable directly and never probe. A wrongly written rule gets pruned by a later audit, a wrongly skipped one is gone for good, so the probe guards the skip side.

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
- Verify every symbol an example references against the real codebase: an example calling a method that does not exist teaches a wrong API and is worse than no example
- Write rules precise but generic: keep load-bearing identifiers exact, never enumerate driftable inventories (class lists, file lists, counts)
- Write the rule for the class of mistake, not the incident that revealed it: the correction that prompted a rule is evidence for it, never phrasing to copy
- Never add self-referential document metadata or biography: version stamps, "last updated" lines, rename history, drift-tracking clauses between files, rules phrased against the past ("previously X, now Z"). State the rule present tense. Git is the history, and the audit cuts these on sight

## The quartet

- `feed-claude-md-files` adds rules from observed patterns
- `bake-claude-md-files` converts crystallized rules into tooling and removes the prose
- `audit-claude-md-files` prunes and verifies what remains
- `split-claude-md-files` moves what remains to the scope that reads it

Run `feed` after a working session, `bake` once enough rules have accumulated to be worth automating, `audit` when CLAUDE.md files have grown without review, and `split` after an audit leaves a resident file carrying rules that govern one area.
