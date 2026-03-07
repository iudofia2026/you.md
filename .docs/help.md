# Advanced Configuration

This guide covers all the advanced ways to customize your personal context system. For the basics, see README.md.

## Table of Contents

- [Coaching Styles](#coaching-styles)
- [Load Modes](#load-modes)
- [Task → File Mapping](#task--file-mapping)
- [Today.md Structure](#todaymd-structure)
- [Editing Rules](#editing-rules)
- [External Projects](#external-projects)
- [Repository Organization](#repository-organization)
- [Commit Messages](#commit-messages)

---

## Coaching Styles

Define how you want Claude to work with you by adding a **Coaching Style** section to your README.md:

### Example: Ruthless Coach

```markdown
## Coaching Style

**How I work best**:
- Be extremely critical - hold me to very high standards
- Push hard - don't coddle
- Set super high bars, especially in friction areas
- Never say "doing too much" - always assume more is possible
- Maximum accountability - call out when not meeting standards
```

### Example: Supportive Guide

```markdown
## Coaching Style

**How I work best**:
- Be encouraging and supportive
- Celebrate small wins
- Break things down into manageable steps
- Remind me of my strengths when I'm stuck
```

### Example: Direct & Concise

```markdown
## Coaching Style

**How I work best**:
- Be direct and concise
- Don't fluff - just tell me what I need to know
- Push me to finish what I start
- Call out when I'm making excuses
```

---

## Load Modes

Your skill supports three load modes. When you invoke it, you'll be asked:

### Quiet Mode
Loads core files silently, then waits for your request.
- **Files read**: `today.md`, `SUMMARY.md`, `README.md`
- **Best for**: Quick context checks before diving into work

### To-Do Mode
Loads context and presents your immediate priorities as an interactive checklist.
- **Files read**: `today.md`, then parses tasks
- **Best for**: Starting your work day, picking what to focus on

### Question Mode
Asks what type of task you're working on, then loads targeted files.
- **Files read**: Depends on task type (see [Task → File Mapping](#task--file-mapping))
- **Best for**: Specific tasks like writing, career planning, relationship advice

---

## Task → File Mapping

Customize which files are loaded for different task types by editing your skill's file mapping.

### Default Mapping

| Task Type | Files Read |
|-----------|-------------|
| Writing/Communication | `values.md`, `traits.md`, `writing/README.md` |
| Professional/Career | `experience.md`, `skills.md`, `goals.md`, `contacts-to-ask.md` |
| Relationship/Social | `friends/[name].md` or `mentors/[name].md` or `connections/[name].md` |
| Academic/Thesis | `academic-achievements.md`, `thesis-senior-project.md` |
| Development | `README.md` (tech stack), project-specific docs |
| Planning/Strategy | `annual-goals/[YEAR].md`, `values.md`, `goals.md` |

### Customizing Your Mapping

Edit your skill file to add, remove, or reorganize file mappings based on how you work.

---

## today.md Structure

The `today.md` file is your command center. Maintain this structure:

```markdown
# Today - Current Life Context

**Last Updated**: [Month Year]

---

## Tasks

### Today (MMM D)

### Tomorrow (MMM D)
- [ ] Task

### Upcoming
- [ ] Task

---

## Active Projects

| Project | Type | Status | Next Action |
|---------|------|--------|-------------|
| Project Name | Type | Status | Next action |

---

## Calendar

### Upcoming

| Date | Event | Priority | Status |
|------|-------|----------|--------|
| Mar 2 | Event name | High | Scheduled |

---

## This Month's Focus ([Month] [Year])

**Theme**: [Monthly theme]

**Academic**:
- Academic focus item

**Professional**:
- Professional focus item

**Personal**:
- Personal focus item

---

## Quick Reference

[Quick notes and reminders]
```

### Section Purposes

- **Tasks** - Daily task management (Today, Tomorrow, Upcoming)
- **Active Projects** - Track ongoing work with next actions
- **Calendar** - Upcoming events with dates and priorities
- **This Month's Focus** - Monthly theme and priorities across life areas
- **Quick Reference** - Notes, reminders, anything you need quick access to

---

## Editing Rules

### Adding Items

**Calendar table** - Add row with format:
```markdown
| Date | Event | Priority | Status |
```
- Keep sorted by date
- Valid Priority: `High`, `Medium`, `Low`
- Valid Status: `Scheduled`, `Planned`, `In Progress`, `Completed`, `Cancelled`

**Tasks checklist** - Add line:
```markdown
- [ ] Task name
```

**Active Projects table** - Add row:
```markdown
| Project | Type | Status | Next Action |
```

### Removing Items

Use exact string matching in your skill or edit directly:
- Checklist: Remove the entire `- [ ] Task name` line
- Tables: Remove the entire table row

### Updating Items

Replace entire table rows or lines - don't do partial edits.

---

## External Projects

Document projects outside your personal repo in your skill's **Appendices** section:

```markdown
## External Projects

- Project 1: `/path/to/project`
- Project 2: `https://example.com`
- Project 3: Description of project
```

This lets Claude reference your other work when providing assistance.

---

## Repository Organization

### Directory Structure

```
your-name.md/
├── today.md              # READ FIRST - current life context
├── SUMMARY.md            # Overview & recent updates
├── README.md             # Your preferences, coaching style
├── .docs/                # Configuration & setup guides
│   ├── help.md           # This file - advanced configuration
│   └── search-setup.md   # Meilisearch integration guide
├── .claude/skills/       # Setup and template skills
│   ├── setup.md          # Run this first
│   ├── uninstall.md      # Remove your setup
│   └── personal-context.md  # Skill template
├── archive/              # Past events by year/month
│   └── 2026/
│       ├── 01-january/
│       ├── 02-february/
│       └── 03-march/
├── future/               # Future items beyond ~4 weeks
│   ├── README.md
│   └── 2026/
│       ├── 04-april/
│       └── 05-may/
├── personal/             # Values, traits, goals, fitness
├── professional/         # Experience, skills, projects, goals
├── relationships/        # Family, friends, mentors, connections
├── academic/             # Education, achievements, research
├── writing/              # Documents, essays
├── tech/                 # Hardware, setup, productivity
├── projects/             # Side projects
└── other/                # Anything else
```

### Adding New Directories

Add any directories that make sense for your life. The structure is a guide, not a rulebook.

---

## Auto-Maintenance Features

Your skill automatically maintains your context:

### What Happens on Every Load

1. **Git sync** - Pulls latest changes from remote
2. **Rollover check** - Verifies today.md matches current date
3. **Date update** - Updates "Last Updated" to current month
4. **Archive completed** - Moves completed entries to `/archive/`
5. **Future check** - Alerts if future items are ready to migrate

### Rollover Detection

The skill looks for `### Today (MMM D)` format matching today's date. If missing, you'll be alerted to rollover.

### Auto-Archive

Completed items are automatically moved to `/archive/YYYY/MM-month/` with:
- Event name
- Date completed
- Context
- Outcome
- Notes

---

## Future Items Management

### Rule of Thumb

- **~4 weeks (28 days) or less** → Add to `today.md` Calendar
- **4+ weeks out** → Add to `/future/YYYY/MM-month/items.md`

### Future Item Format

```markdown
# April 2026 - Future Items

---

## Calendar

| Date | Event | Priority | Status |
|------|-------|----------|--------|
| Apr 30 | Project milestone | High | Planned |

---

## Details

**Project milestone**:
- Detail 1
- Detail 2
```

### Monthly Migration

When the month arrives, items are flagged for migration to `today.md`.

---

## Commit Messages

### After Any Context Change

Your skill automatically commits with format:
```
update: [brief description] - [YYYY-MM-DD]
```

### Manual Commit Format

```bash
git add -A
git commit -m "update: [brief description] - [YYYY-MM-DD]"
git push origin main
```

### Archive Commit Format

```bash
git add -A
git commit -m "archive: [task name] - YYYY-MM-DD"
git push origin main
```

---

## Marking Items as Done

When you say "mark [X] as done", the skill:

1. **Asks for outcome** - What happened?
2. **Creates archive entry** - Permanent record in `/archive/`
3. **Removes from active** - Deletes from today.md active sections
4. **Commits and pushes** - Saves to git

---

## Custom Writing Style

Add a **Writing Style** section to your README.md to control how Claude writes for you:

```markdown
## Writing Style

- **Concise but thorough** - Avoid fluff, be direct
- **Personal yet professional** - Warmth with competence
- **Evidence-backed** - Use specific examples and data
- **Active voice** - "I did X" not "X was done"
```

---

## Search (Optional)

> **AI Agent Setup**: If you want to add search, tell Claude: "Set up Meilisearch for my personal context following [.docs/search-setup.md](.docs/search-setup.md)" - it can do the entire setup for you.

If you use a search tool like Meilisearch, you can configure your skill to search your context:

### When to Search

- "Who is [Name]?"
- "What are my goals?"
- "Find where I mention [topic]"

### When NOT to Search

- Content already in today.md, SUMMARY.md, README.md (already loaded)
- File navigation (use directory structure instead)

### Agent Setup

Claude can set up Meilisearch for you automatically:
1. Tell it: "Set up Meilisearch for my personal context"
2. It will create the directory structure, docker-compose.yml, and indexer script
3. It will guide you through starting the service
4. It will update your skill with search integration

See [search-setup.md](search-setup.md) for full details.

---

## Uninstalling

### Remove Skill Only

Keeps your repository, just removes the Claude skill:
```bash
@uninstall
```
Then select "Remove skill only"

### Remove Everything

Removes both skill AND repository directory:
```bash
@uninstall
```
Then select "Remove skill and repo"

---

## Troubleshooting

### Can I re-run setup?

**Yes!** Re-running `@setup` is completely safe:
- Existing files are never overwritten
- Setup only creates files that don't exist
- You can add more information anytime by re-running
- The skill is updated if paths have changed

### Skill not loading?

Check the skill path points to your repo:
```bash
cat ~/.claude/skills/your-name/skill.md | grep REPO_PATH
```

Should show your actual repository path.

### Something else wrong?

1. Make sure you're in the repository directory
2. Check the repo is cloned to `~/github/your-name.md`
3. Try re-running `@setup` - won't overwrite your data

---

## Advanced Customization Ideas

### Things You Can Track

- **Habits and routines** - Daily practices, goals
- **Health and fitness** - Workouts, diet, metrics
- **Reading lists** - Books to read, notes
- **Financial goals** - Keep private!
- **Ideas and inspiration** - Capture thoughts
- **Travel plans** - Trips, itineraries
- **Journal entries** - Daily reflections
- **Learning paths** - Courses, skills to acquire

### Custom File Locations

Add files anywhere that makes sense for you. The structure is flexible.

### Multiple Contexts

You can create multiple personal context repos for different areas of life:
- `personal.md` - Life context
- `work.md` - Professional context
- `projects.md` - Side projects

Each gets its own skill: `/personal`, `/work`, `/projects`

---

## Summary

This system is designed to be:
- **Flexible** - Organize information how it makes sense for you
- **Safe** - All local, private until you explicitly push
- **Maintained** - Auto-sync, auto-archive, auto-documentation
- **Personalized** - Coaching style, writing style, preferences

The structure is a guide, not a rulebook. Make it yours.
