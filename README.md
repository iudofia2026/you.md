# you.md - Personal Context Template

A template for creating a personal knowledge repository that **Claude Code** can reference for personalized assistance.

## Version

**v0.0.2**

## What This Does

This creates **two things** that work together:

1. **A personal repository** - Your life context (goals, relationships, projects, communication style)
2. **A Claude Code skill** - Lets Claude access your context in any conversation

Instead of generic responses, Claude knows *you* and provides personalized help.

## Quick Start (4 Steps)

1. **Fork** this repository → rename to `your-name.md` (e.g., `john.smith.md`)
2. **Make it PRIVATE** → Settings → Danger Zone → Change visibility → Make private
3. **Clone** → `git clone https://github.com/YOUR_USERNAME/john.smith.md.git ~/github/john.smith.md`
4. **Run `@setup`** in Claude Code from that directory

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

- **Advanced config** - See [help.md](help.md) for customization options
- **Re-run setup anytime** - Safe, won't overwrite anything
- **Edit files directly** - Just text files
- **Uninstall**: Run `@uninstall` from the repo directory

---

Based on the `isiah.md` repository structure - a production personal context system in daily use.
