# you.md - Personal Context Template

A template for creating a personal knowledge repository that AI agents can reference for personalized assistance.

## How It Works

1. **Fork this repository** → rename your fork to `your-name.md` (e.g., `john.smith.md`)
2. **Clone** → your repo becomes `~/github/your-name.md`
3. **Run `@setup`** → creates your personal skill at `~/.claude/skills/your-name/` pointing to your repo
4. **Use** → type `/your-name` to load your personal context in any conversation

## Quick Start

### 1. Fork and Rename

1. Fork this repository on GitHub
2. Rename your fork to `your-name.md` (e.g., `john.smith.md`, `jane-doe.md`)

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

### 4. Use Your Context

```
/john-smith help me write an email to my mom
/john-smith what should I prioritize this week?
```

## What Gets Created

After setup, you'll have:

| Location | What It Is |
|----------|------------|
| `~/github/your-name.md/` | Your personal context repository |
| `~/.claude/skills/your-name/` | Your personal skill (global, accessible from anywhere) |
| `personal/values.md` | Your core values (if you answered) |
| `personal/traits.md` | Your personality traits (if you answered) |
| `professional/experience.md` | Work history (if you answered) |
| etc. | Other files based on your answers |

## Repository Structure

```
your-name.md/
├── .claude/skills/
│   ├── setup.md             # The @setup skill
│   ├── uninstall.md         # The @uninstall skill
│   └── personal-context.md  # Skill template (copied during setup)
├── README.md                # This file
│
├── personal/                # Your personal information
├── professional/            # Work & career
├── relationships/           # People database
│   ├── family/              # Family member profiles
│   ├── friends/             # Friend profiles
│   ├── mentors/             # Professional mentor profiles
│   └── connections/         # Professional connections
├── academic/                # Education background
├── writing/                 # Your documents
├── tech/                    # Setup and tools
├── projects/                # Side projects
└── other/                   # Anything else you want to track
```

## Example Workflow

```
1. Fork you.md → rename fork to "isiah.md"
2. Clone: git clone ... ~/github/isiah.md
3. Run @setup in Claude Code
4. Skill created at: ~/.claude/skills/isiah/
5. Skill points to: ~/github/isiah.md
6. Use @isiah in any conversation to load context
```

## Privacy & Security

**IMPORTANT: Keep your repository PRIVATE**

This repository contains your personal information - values, relationships, career details, goals, and more. Before running setup:

1. **Set your fork to PRIVATE** - After forking, go to your fork's Settings → Danger Zone → Change visibility → Make private
2. **Never push personal data to public repos** - Your personal context is for you and AI assistants only
3. **Local use only** - The skill reads from your local machine; no data is sent externally

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
cat ~/.claude/skills/your-name/description.md | grep REPO_PATH
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

## Inspiration

Based on the `isiah.md` repository structure.
