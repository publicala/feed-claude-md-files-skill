# feed-claude-md-files

Claude Code skill — surfaces recurring patterns in recent commits and in-session corrections, proposes them as new CLAUDE.md rules in the right file (root or scoped subdir), and writes them only after approval.

The inverse of [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill): `feed` adds prose rules from observed patterns; `bake` converts crystallized prose rules into tooling once they're stable.

## How it works

1. Reads recent git history (commits + diffs), in-session corrections, and existing CLAUDE.md files
2. Clusters recurring patterns into candidate rules
3. Drops duplicates and rules better expressed as tooling checks (defers to `bake`)
4. Proposes each remaining rule via `AskUserQuestion`, with a suggested target file (root vs scoped subdir)
5. Writes only after approval — never duplicates an existing rule

## Install

The repo doubles as a plugin marketplace (required by Claude Code for `plugin install` to work). `marketplace.json` points to the plugin in this same repo.

### Via skills.sh

```bash
npx skills add publicala/feed-claude-md-files-skill
```

### Via Plugin Marketplace

```
/plugin marketplace add publicala/feed-claude-md-files-skill
/plugin install feed-claude-md-files@publicala
```

### Manual

Copy `skills/feed-claude-md-files/SKILL.md` into your skills directory:

```bash
# Global (all projects)
mkdir -p ~/.claude/skills/feed-claude-md-files
cp skills/feed-claude-md-files/SKILL.md ~/.claude/skills/feed-claude-md-files/

# Project-level
mkdir -p .claude/skills/feed-claude-md-files
cp skills/feed-claude-md-files/SKILL.md .claude/skills/feed-claude-md-files/
```

## Usage

Both installation methods invoke the same skill:

- **skills.sh / manual**: `/feed-claude-md-files`
- **Plugin marketplace**: `/feed-claude-md-files:feed`

Run it after a working session — once you've accumulated commits and corrections worth distilling.

## Pairs with `bake`

| Direction | Skill | What it does |
|-----------|-------|--------------|
| Patterns → prose rules | `feed` | Watches what you do, proposes new CLAUDE.md rules |
| Prose rules → tooling | `bake` | Replaces prose rules with linter / static-analysis / CI checks |

The loop: `feed` after a session, `bake` once rules have stabilized.

## Resources

- [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill) — the inverse skill
- [CLAUDE.md Guide](https://github.com/publicala/claude-md-guide) — Presentation slides about CLAUDE.md files
- [CLAUDE.md docs](https://docs.anthropic.com/en/docs/claude-code/memory) — Official documentation

## License

MIT
