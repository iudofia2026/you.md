# you.md - Personal Context Template

A template for creating a personal knowledge repository that **Claude Code** can reference for personalized assistance.

## Version

**v0.0.3**

## What This Does

This creates **two things** that work together:

1. **A personal repository** - Your life context (goals, relationships, projects, communication style)
2. **A Claude Code skill** - Lets Claude access your context in any conversation

Instead of generic responses, Claude knows *you* and provides personalized help.

## Quick Start (4 Steps)

1. **Fork** this repository → rename to `your-name.md` (e.g., `john.smith.md`)
2. **Make it PRIVATE** → Settings → Danger Zone → Change visibility → Make private
3. **Clone** → `git clone https://github.com/YOUR_USERNAME/john.smith.md.git ~/github/john.smith.md`
4. **Run `/setup`** in Claude Code from that directory

Then use: `/john` to load your context in any Claude Code conversation.

## What Gets Created

| Thing | What It's For |
|------|---------------|
| **Repository** | Your personal knowledge base |
| **Claude skill** | Access via `/your-name` command |
| **today.md** | Current tasks, calendar, projects |
| **SUMMARY.md** | Overview of your life |
| **README.md** | How you want Claude to work with you |
| **archive/** | Completed tasks (auto-saved) |
| **future/** | Stuff coming up beyond ~4 weeks |
| **personal/** | Your values, goals, traits |
| **professional/** | Work history, skills, projects |
| **relationships/** | People database |

## Key Features

- **Auto-sync** - Always up to date with git
- **Rollover detection** - Alerts you to update daily tasks
- **Auto-archive** - Completed tasks saved automatically
- **Future items** - Backlog for stuff beyond ~4 weeks
- **Auto-documentation** - Claude asks before saving new info

## How You Use It

```
/john                           → Load context, see priorities
/john mark "Interview" as done   → Complete and archive task
/john help me email Mom          → Context-aware help
```

## Privacy

**Keep your repository PRIVATE.** Your personal data stays on your machine unless you explicitly `git push`. Claude Code conversations are ephemeral and never stored.

## Customize It

Add to your `README.md` to tell Claude how to work with you:

```markdown
## Coaching Style
- Be direct and concise
- Push me to finish what I start
- Celebrate small wins

## Writing Style
- Professional but warm
- Active voice only
```

## Need Help?

- **Advanced config** - See [.docs/help.md](.docs/help.md) for customization options
- **Search setup** - See [.docs/search-setup.md](.docs/search-setup.md) for Meilisearch integration
- **Re-run setup anytime** - Safe, won't overwrite anything
- **Edit files directly** - Just text files
- **Uninstall**: Run `/uninstall` from the repo directory

---

## What Changed in v0.0.3

| Fix | Details |
|-----|---------|
| **Doctor check** | New read-only diagnostic runs before any setup writes — reports PASS/WARN/FAIL for git state, skill path, invocation convention, and optional search |
| **Skill install path** | Normalized to `~/.claude/skills/your-name/SKILL.md` (was `skills@your-name/description.md`). Legacy installs detected and migrated non-destructively |
| **Invocation convention** | All docs updated from `/@name` → `/name` and `@setup` → `/setup` |
| **Rerun detection bug** | Fixed — skill name is now collected before rerun check runs (was undefined at check time) |
| **Git sync hardened** | Bare `git pull` replaced with state-aware sync: dirty/detached/ahead/diverged all handled safely |
| **Archive logic** | Hardcoded `2026` year removed — dynamic year used. Empty/malformed entries skipped. Preview+confirm before deletion |
| **Optional search** | Health-check first, fallback to file reads silently — zero blocking dependency for non-search users |

## Migration Guide

### Fresh install (new user)
No action needed. Run `/setup` as normal — you'll land on the normalized path automatically.

### Legacy install upgrade (existing user with `skills@name/description.md`)
1. Run `/setup` from your repo directory
2. The doctor check (Step 0) will detect your legacy install and report `[WARN]`
3. Step 2 will automatically copy your skill to the normalized path and keep the legacy file intact
4. Test that `/your-name` loads correctly
5. Once confirmed, delete the legacy file manually: `rm -rf ~/.claude/skills@your-name`

### Dirty git state
The sync step will detect uncommitted changes and skip the pull — your local changes are never clobbered. Sync manually when ready: `git pull origin main`.

### No search installed
Nothing to do. Search is fully optional — the skill falls back to file reads silently. No errors, no setup required.

## Rollback (if something breaks)

1. **Restore legacy skill file** (if you had one): `cp ~/.claude/skills@your-name/description.md ~/.claude/skills/your-name/SKILL.md`
2. **Pin to prior version**: In your fork, revert to the `v0.0.2.2` tag: `git checkout v0.0.2.2 -- .claude/skills/`
3. **Doctor-only mode**: Delete `~/.claude/skills/your-name/SKILL.md` and re-run `/setup` — it will only run diagnostics until you complete the full setup flow

---

Based on a production personal context system in daily use.
