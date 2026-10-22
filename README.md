# claude-skills

General-purpose [Claude Code](https://claude.com/claude-code) skills.

| Skill | What it does |
|---|---|
| [mac-disk-cleanup](mac-disk-cleanup/SKILL.md) | Frees disk space on macOS: finds "System Data" bloat (Docker VM disks, caches, Xcode), deletes safe items and asks before risky ones |

## Install

Copy a skill folder into `~/.claude/skills/` (personal) or `.claude/skills/` (per project):

```bash
git clone https://github.com/deBilla/claude-skills.git
cp -r claude-skills/mac-disk-cleanup ~/.claude/skills/
```

Restart Claude Code. The skill loads automatically when relevant, or you can run it as `/mac-disk-cleanup`.
