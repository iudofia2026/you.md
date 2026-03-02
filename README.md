# you.md - Personal Context Template

A template for creating a personal knowledge repository that AI agents can reference for personalized assistance.

## Version

**v0.0.3**
- **today.md** - Current life context with todos, calendar, active projects
- **archive/** - Auto-archived completed items
- **future/** - Future items beyond ~4 weeks, auto-migrate to today.md
- **Auto-maintenance** - Sync, rollover check, date updates, archiving on every skill load
- **Mark as done** - One-command workflow to complete and archive tasks
- **Rollover detection** - Checks if today.md matches current date
- **Auto-documentation prompts** - Ask before preserving new information

## How It Works

1. **Fork this repository** → rename your fork to `your-name.md` (e.g., `john.smith.md`)
2. **Clone** → your repo becomes `~/github/your-name.md`
3. **Run `@setup`** → creates your personal skill at `~/.claude/skills/your-name/` pointing to your repo
4. **Use** → type `/your-name` to load your personal context in any conversation

## Quick Start

### 1. Fork and Rename

1. Fork this repository on GitHub
2. Rename your fork to `your-name.md` (e.g., `john.smith.md`, `jane-doe.md`)
3. **Make it PRIVATE** → Settings → Danger Zone → Change visibility → Make private

### 2. Clone

```bash
git clone https://github.com/YOUR_USERNAME/john.smith.md.git ~/github/john.smith.md
cd ~/github/john.smith.md
```

### 3. Run Setup

In Claude Code, while in the repository directory:
```
@setup
```

The setup skill will:
- Ask for your skill name (e.g., `john` for `/john`)
- Ask exploratory questions to populate your repository
- **Create your personal skill** at `~/.claude/skills/your-name/`
- Configure the skill to point to your repository at `~/github/your-name.md`
- Create `today.md`, `SUMMARY.md`, `README.md` with your info

### 4. Use Your Context

```
/john-smith help me write an email to my mom
/john-smith what should I prioritize this week?
/john-smith mark "Job interview" as done
```

## What Gets Created

After setup, you'll have:

| Location | What It Is |
|----------|------------|
| `~/github/your-name.md/` | Your personal context repository |
| `~/.claude/skills/your-name/` | Your personal skill (global, accessible from anywhere) |
| `today.md` | Current todos, calendar, active projects (auto-maintained) |
| `SUMMARY.md` | Overview & recent updates |
| `README.md` | Your coaching style, tech stack, structure |
| `archive/` | Completed items (auto-archived) |
| `future/` | Future items beyond ~4 weeks |
| `personal/` | Your values, traits, goals |
| `professional/` | Work history, skills, projects |
| `relationships/` | People database (family, friends, mentors) |

## Repository Structure

```
your-name.md/
├── today.md              # READ FIRST - current life context
├── SUMMARY.md            # Overview & recent updates
├── README.md             # Your preferences, tech stack, structure
├── .claude/skills/       # Setup and template skills
│   ├── setup.md          # Run this first
│   ├── uninstall.md      # Remove your setup
│   └── personal-context.md  # Skill template (copied during setup)
│
├── archive/              # Past events by year/month
│   └── 2026/
│       ├── 01-january/
│       ├── 02-february/
│       └── 03-march/
│           └── event-name.md
│
├── future/               # Future items beyond ~4 weeks
│   ├── README.md
│   └── 2026/
│       ├── 04-april/
│       │   └── items.md
│       └── 05-may/
│           └── items.md
│
├── personal/             # Your personal information
│   ├── values.md         # Core principles
│   ├── traits.md         # Personality traits
│   ├── goals/            # Annual goals
│   └── fitness/          # Routines, health
│
├── professional/         # Work & career
│   ├── experience.md     # Work history
│   ├── skills.md         # Capabilities
│   ├── projects.md       # Project overview
│   └── goals.md          # Career objectives
│
├── relationships/        # People database
│   ├── family/           # Family member profiles
│   ├── friends/          # Friend profiles
│   ├── mentors/          # Professional mentor profiles
│   └── connections/      # Professional connections
│
├── academic/             # Education background
├── writing/              # Your documents
├── tech/                 # Setup and tools
├── projects/             # Side projects
└── other/                # Anything else
```

## Key Features

### today.md - Your Command Center

The `today.md` file includes:
- **Tasks** - Today, Tomorrow, and Upcoming sections
- **Calendar** - Upcoming events with dates, priorities, status
- **Active Projects** - Ongoing work with next actions
- **This Month's Focus** - Theme and priorities for the month
- **Quick Reference** - Notes and reminders

**Auto-maintained** every time you load your skill:
- Syncs with git pull
- Checks if today.md matches current date (rollover detection)
- Updates "Last Updated" to current month
- Past completed entries moved to `/archive/`
- Future items flagged when ready to migrate

### Archive - Completed Items

When you mark items as done, they're automatically archived to `/archive/YYYY/MM-month/` with:

- Event name
- Date completed
- Context (what it was)
- Outcome (how it went)
- Notes (follow-up, lessons learned)

No more losing track of what you've accomplished.

### Rollover Detection

The skill checks if your `today.md` is stale:
- Looks for `### Today (MMM D)` format matching today's date
- If missing, alerts you that rollover is needed

### Future - Backlog Beyond ~4 Weeks

Items beyond ~4 weeks go in `/future/YYYY/MM-month/items.md`:

- **~4 weeks or less** → `today.md` Calendar
- **4+ weeks out** → `/future/YYYY/MM-month/items.md`

When the month arrives, items are flagged for migration to `today.md`.

### Auto-Documentation

The skill asks before preserving new information:
- New emails/messages drafted and sent
- New professional connections made
- Updates to existing relationships
- Career developments (interviews, offers, rejections)
- Project updates or milestones

## Skill Commands

| Command | What It Does |
|---------|--------------|
| `/your-name` | Load your context, see priorities |
| `/your-name mark "X" as done` | Complete and archive a task |
| `/your-name what's on my todo list?` | See current tasks |
| `/your-name help me write an email` | Context-aware assistance |

## Customization

### Coaching Style

Add to your `README.md`:

```markdown
## Coaching Style

**How I work best**:
- Be direct and concise
- Push me to finish what I start
- Call out when I'm making excuses
- Celebrate small wins

**Writing Style**:
- Professional but warm
- Active voice only
- Evidence-backed claims
```

### External Projects

Add to your skill's Appendices:

```markdown
## External Projects
- Project 1: `/path/to/project`
- Project 2: `https://example.com`
```

## Example Workflow

```
1. Fork you.md → rename fork to "john.smith.md"
2. Clone: git clone ... ~/github/john.smith.md
3. Run @setup in Claude Code
4. Skill created at: ~/.claude/skills/john/
5. Skill points to: ~/github/john.smith.md
6. Use /john in any conversation to load context

Daily usage:
- /john → See today's priorities
- /john mark "Task" as done → Complete and archive
- /john what's on my todo list? → See current tasks
```

## Privacy & Security

**IMPORTANT: Keep your repository PRIVATE**

This repository contains your personal information - values, relationships, career details, goals, and more.

### How Your Data Stays Private

| Component | Location | Privacy |
|-----------|----------|---------|
| Your repository | `~/github/your-name.md` | Local only (unless you push) |
| Your skill | `~/.claude/skills/your-name/` | Local only, never synced |
| AI conversations | Claude Code | Ephemeral, not stored |

**Your personal context NEVER leaves your machine unless you explicitly `git push` to a remote.**

### GitHub Fork Visibility

**GitHub doesn't allow templates to enforce private-only forking.** When you fork this repo, GitHub defaults to PUBLIC visibility. **You must manually change it to PRIVATE** before adding any personal information.

Steps to make your fork private:
1. Go to your fork on GitHub
2. Settings → Scroll to "Danger Zone"
3. "Change repository visibility" → Make private
4. Confirm the change

**Do this BEFORE running `@setup` and adding your personal information.**

## Uninstall

To remove your personal context setup:

```
@uninstall
```

Run this from within your repository directory. The uninstall skill will ask what you want to remove:

| Option | What Gets Deleted |
|--------|------------------|
| Remove skill only | `~/.claude/skills/your-name/` - keeps your repository |
| Remove skill and repo | Both skill AND `~/github/your-name.md/` directory |
| Cancel | Nothing, exit uninstall |

**Warning:** Deletion is permanent. Make sure you have backups if you want to keep your data.

## Troubleshooting

### Can I re-run setup?

**Yes!** Re-running `@setup` is completely safe:

- **Existing files are never overwritten**
- Setup only creates files that don't exist
- You can add more information anytime by re-running
- The skill is updated if paths have changed

### What if I mess up a file?

Edit any file directly - they're just markdown files. No need to re-run setup unless you want to add new sections.

### Skill not loading?

Check the skill path points to your repo:
```bash
cat ~/.claude/skills/your-name/skill.md | grep REPO_PATH
```

Should show your actual repository path (e.g., `/Users/you/github/your-name.md`).

### Something else wrong?

1. Make sure you're in the repository directory
2. Check the repo is cloned to `~/github/your-name.md`
3. Try re-running `@setup` - it won't overwrite your data

## Customizing Your Repo

The template is just a starting point. Add whatever you need:

### Ways to Customize

1. **Add files anywhere** - Create new markdown files in any directory
2. **Use the `other/` directory** - For things that don't fit existing categories
3. **Re-run setup** - Use the custom files prompt to add more via `@setup`
4. **Edit directly** - These are just markdown files - edit them however you want

### What Else Can I Track?

Check `other/README.md` for ideas like:
- Habits and routines
- Health and fitness
- Reading lists and book notes
- Financial goals (keep private!)
- Ideas and inspiration
- Travel plans
- Journal entries
- Anything else that matters to you

### No Wrong Way to Use This

The structure is a guide, not a rulebook. Organize your information in whatever way makes sense for you.

## Skill Commands

| Command | What It Does |
|---------|--------------|
| `/your-name` | Load your context, see priorities |
| `/your-name mark "X" as done` | Complete and archive a task |
| `/your-name what's on my todo list?` | See current tasks |
| `/your-name help me write an email` | Context-aware assistance |

## Customization

### Coaching Style

Add to your `README.md`:

```markdown
## Coaching Style

**How I work best**:
- Be direct and concise
- Push me to finish what I start
- Call out when I'm making excuses
- Celebrate small wins

**Writing Style**:
- Professional but warm
- Active voice only
- Evidence-backed claims
```

### External Projects

Add to your skill's Appendices:

```markdown
## External Projects
- Project 1: `/path/to/project`
- Project 2: `https://example.com`
```

## Example Workflow

Once set up, use these commands:

| Command | What It Does |
|---------|--------------|
| `/your-name` | Load your context, see priorities |
| `/your-name mark "X" as done` | Complete and archive a task |
| `/your-name what's on my todo list?` | See current tasks |
| `/your-name help me write an email` | Context-aware assistance |

## Inspiration

Based on the `isiah.md` repository structure - a production personal context system in daily use.
