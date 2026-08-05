# feed-claude-md-files

Claude Code skill — surfaces recurring patterns in recent commits and in-session corrections, proposes them as new CLAUDE.md rules in the right file (root or scoped subdir), and writes them only after approval.

The CLAUDE.md quartet: [feed-claude-md-files](https://github.com/publicala/feed-claude-md-files-skill) adds rules from observed patterns, [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill) converts crystallized rules into tooling, [audit-claude-md-files](https://github.com/publicala/audit-claude-md-files-skill) prunes and verifies what remains, and [split-claude-md-files](https://github.com/publicala/split-claude-md-files-skill) moves what remains to the scope that reads it. Install all four from [publicala/claude-plugins](https://github.com/publicala/claude-plugins).

## How it works

1. Reads recent git history (commits + diffs), in-session corrections, and existing CLAUDE.md files
2. Clusters recurring patterns into candidate rules
3. Drops duplicates and rules better expressed as tooling checks (defers to `bake`)
4. Proposes each remaining rule via `AskUserQuestion`, with a suggested target file (root vs scoped subdir)
5. Writes only after approval — never duplicates an existing rule

## Install

### Via Plugin Marketplace

```
/plugin marketplace add publicala/claude-plugins
/plugin install feed-claude-md-files@publicala
```

### Via skills.sh

```bash
npx skills add publicala/feed-claude-md-files-skill
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

- **skills.sh / manual**: `/feed-claude-md-files`
- **Plugin marketplace**: `/feed-claude-md-files:feed-claude-md-files` (plugin skills are namespaced as `/<plugin>:<skill>`)

Run it after a working session — once you've accumulated commits and corrections worth distilling.

## Resources

- [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill) — converts CLAUDE.md rules into automated checks
- [audit-claude-md-files](https://github.com/publicala/audit-claude-md-files-skill) — prunes CLAUDE.md files with evidence-backed cuts
- [split-claude-md-files](https://github.com/publicala/split-claude-md-files-skill) — moves CLAUDE.md rules to the load scope that reads them
- [CLAUDE.md Guide](https://github.com/publicala/claude-md-guide) — Presentation slides about CLAUDE.md files
- [CLAUDE.md docs](https://docs.anthropic.com/en/docs/claude-code/memory) — Official documentation

## License

MIT
