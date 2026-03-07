---
name: personal-context
description: >
  Load your personal context repository for personalized assistance.
  Auto-syncs, updates dates, manages todos/future items, and archives completed work.
category: personal
priority: 10
---

You are loading context about **[YOUR_NAME]** from their personal knowledge repository.

**REPO_PATH**: `[REPO_PATH]`

**IMPORTANT**: This skill includes auto-maintenance that runs every time:
- Syncs repo with git pull
- Updates "Last Updated" to current month
- Archives past completed entries (moves to /archive/)
- Checks for future items ready to migrate

## Purpose

Load the **minimum context needed** to provide personalized help, then direct to relevant files for deeper information.

## Quick Load Workflow

### Step 0: ALWAYS Sync & Auto-Maintain First

**CRITICAL**: Before loading any context, sync the repository and run maintenance.

```bash
# Detect OS and sync
REPO="[REPO_PATH]"
cd "$REPO" && git pull origin main --quiet

# Get current date info
CURRENT_DATE=$(date "+%Y-%m-%d")
CURRENT_MONTH=$(date "+%B %Y")
CURRENT_MONTH_NUM=$(date "+%m")
CURRENT_MONTH_LOWER=$(date "+%B" | tr '[:upper:]' '[:lower:]')
ARCHIVE_DIR="$REPO/archive/$(date +%Y)/$(date +%m)-$(date +%B | tr '[:upper:]' '[:lower:]')"
mkdir -p "$ARCHIVE_DIR"

# Update Last Updated header in today.md
if [[ "$(uname -s)" == "Darwin" ]]; then
  sed -i '' "s/\*\*Last Updated\*\*: .*/\*\*Last Updated\*\*: $CURRENT_MONTH/" "$REPO/today.md"
else
  sed -i "s/\*\*Last Updated\*\*: .*/\*\*Last Updated\*\*: $CURRENT_MONTH/" "$REPO/today.md"
fi

# Find and archive past completed entries (before today)
grep -n "Completed.*2026" "$REPO/today.md" 2>/dev/null | while read -r line; do
  LINE_NUM=$(echo "$line" | cut -d: -f1)
  COMPLETED_CONTENT=$(echo "$line" | cut -d: -f2-)

  # Parse the date from the completed entry
  COMPLETED_DATE=$(echo "$COMPLETED_CONTENT" | sed -E 's/.*Completed ([A-Za-z]+ [0-9]+, [0-9]+).*/\1/')

  # Create archive entry for this completed item
  TASK_NAME=$(echo "$COMPLETED_CONTENT" | sed -E 's/.*Completed [A-Za-z]+ [0-9]+, [0-9]+: (.+) - .*/\1/' | cut -d'-' -f1 | sed 's/:$//')
  OUTCOME=$(echo "$COMPLETED_CONTENT" | sed -E 's/.*- (.+)$/\1/')

  cat > "$ARCHIVE_DIR/${TASK_NAME// /-}-$(date +%Y%m%d).md" << EOF
# ${TASK_NAME}

**Date Completed**: ${COMPLETED_DATE}
**Status**: COMPLETED

## Context

Auto-archived from today.md completion entry.

## Outcome

${OUTCOME}

## Notes

Auto-archived on $(date "+%B %-d, %Y")
EOF
done

# Remove all completed entries (they're now archived)
if [[ "$(uname -s)" == "Darwin" ]]; then
  sed -i '' '/Completed.*2026/d' "$REPO/today.md" 2>/dev/null || true
else
  sed -i '/Completed.*2026/d' "$REPO/today.md" 2>/dev/null || true
fi

# Check if future/ has items for current month
FUTURE_MONTH_DIR="$REPO/future/$(date +%Y)/${CURRENT_MONTH_NUM}-${CURRENT_MONTH_LOWER}"
if [ -d "$FUTURE_MONTH_DIR" ] && [ -f "$FUTURE_MONTH_DIR/items.md" ]; then
  echo "[AUTO] Found future items for $CURRENT_MONTH in $FUTURE_MONTH_DIR"
  echo "[TODO] Migrate items from $FUTURE_MONTH_DIR/items.md to today.md"
fi

echo "[AUTO] Context synced and refreshed"
```

### Step 1: Ask Mode First

**ALWAYS use AskUserQuestion tool to ask the user which mode they want:**

```json
{
  "questions": [
    {
      "question": "How should I load your context?",
      "header": "Load Mode",
      "options": [
        { "label": "Quiet", "description": "Load core files (today.md, SUMMARY.md, README.md) silently, wait for your request" },
        { "label": "To-do", "description": "Load context and show your immediate priorities as a checklist from today.md" },
        { "label": "Question", "description": "Ask about task type and context needs, then load targeted files" }
      ],
      "multiSelect": false
    }
  ]
}
```

### Step 2: Load Based on Mode

| Mode | Files to Read | Presentation |
|------|---------------|--------------|
| Quiet | today.md, SUMMARY.md, README.md (parallel) | Summarize, wait for request |
| Question | Use Task→File mapping below | Ask follow-up with AskUserQuestion |
| To-do | today.md, parse tasks | Present tasks with AskUserQuestion |

**For To-do mode:**
1. Parse today.md tasks
2. Present with AskUserQuestion (not text list):
```json
{
  "questions": [{
    "question": "What do you want to tackle?",
    "header": "Your Tasks",
    "options": [
      {"label": "Task name", "description": "Brief context"},
      ...
    ],
    "multiSelect": false
  }]
}
```
3. On selection, load relevant files and begin work

### Step 3: Confirm

```
✓ Loaded [YOUR_NAME] context
  Current: [from today.md Active Projects/This Month]
  Read [X] files
```

## Key Context (CUSTOMIZE IN YOUR README)

**Coaching Style**: [Customize in your README - e.g., "Ruthless, high standards" or "Supportive and encouraging"]

**Current Situation**: ALWAYS read fresh from today.md sections:
- Active Projects table
- This Month's Focus
- Calendar > Upcoming
- Today tasks

**Writing Style**: [Customize in your README - e.g., "Concise but thorough, personal yet professional, evidence-backed, active voice"]

## Task → File Mapping

| Task Type | Read These Files |
|-----------|------------------|
| Writing/Communication | values.md, traits.md, writing/README.md |
| Professional/Career | experience.md, skills.md, goals.md, contacts-to-ask.md |
| Relationship/Social | friends/[name].md or mentors/[name].md or connections/[name].md |
| Academic/Thesis | academic-achievements.md, thesis-senior-project.md |
| Development | README.md (tech stack), project-specific docs |
| Planning/Strategy | annual-goals/[YEAR].md, values.md, goals.md |

## Task Archiving

When user says "Archive: [task]" or "Mark done":

```markdown
# [Task Name]
**Date Archived**: YYYY-MM-DD
**Status**: COMPLETED

## Context
[Brief description]

## Action Taken
[What happened - full transcripts/messages/links]

## Notes
[Follow-up needed, lessons learned]
```

1. Create file in `archive/YYYY/MM-monthname/[kebab-task-name].md`
2. Remove from today.md
3. `git add -A && git commit -m "archive: [task] - YYYY-MM-DD" && git push origin main`

## Appendices

**External Projects** (customize in your README with your actual projects):
- Project 1: `[path/to/project]`
- Project 2: `[path/to/project]`
- Project 3: `[url or description]`

**After ANY changes to your repo**:
```bash
cd "$REPO" && git add -A && git commit -m "update: [brief]" && git push origin main --quiet
```

## today.md Structure

The `today.md` file is your current life context. Maintain this structure:

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

### Editing Rules

**When ADDING items:**

1. **Calendar table** - Add row with format: `| Date | Event | Priority | Status |`
   - Keep sorted by date
   - Valid Priority: High, Medium, Low
   - Valid Status: Scheduled, Planned, In Progress, Completed, Cancelled

2. **Tasks checklist** - Add line: `- [ ] Task name`

3. **Active Projects table** - Add row with format: `| Project | Type | Status | Next Action |`

**When REMOVING items:**

- Use exact string matching
- For checklist: `sed -i '' '/- \[ \] Exact task text/d' "$REPO/today.md"`
- For tables: `sed -i '' '/| Exact Project Name |/d' "$REPO/today.md"`

**When UPDATING items:**

- Replace entire table row, not partial edits
- For status changes: rewrite the full row

## Repository Structure

```
your-name.md/
├── today.md              # READ FIRST - current life context
├── SUMMARY.md            # Overview & recent updates
├── README.md             # Coaching style, structure, customization
├── .docs/                # Configuration & setup guides
│   ├── help.md           # Advanced configuration guide
│   └── search-setup.md   # Search integration guide
├── archive/              # Past events by year/month
│   └── 2026/
│       ├── 01-january/
│       ├── 02-february/
│       └── 03-march/
├── future/               # Future items beyond ~4 weeks
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

## Archive Directory

Completed items are archived to `/archive/YYYY/MM-month/` with this format:

```markdown
# [Event Name]

**Date Completed**: YYYY-MM-DD
**Status**: COMPLETED

## Context

[Brief description - what was this event?]

## Outcome

[What actually happened - update after event]

## Notes

[Follow-up, lessons learned, action items]
```

## Future Log (/future/)

**Purpose**: Backlog for items beyond ~4 weeks out. Auto-migrated to `today.md` when the month arrives.

**Directory structure**: Mirrors `/archive/` but for future months
```
future/
├── README.md           # Rules and index
└── 2026/
    ├── 04-april/
    │   └── items.md   # Calendar + details for April
    ├── 05-may/
    │   └── items.md   # Calendar + details for May
    └── 06-june/
        └── items.md
```

**Rule of thumb**:
- **~4 weeks (28 days) or less** → Add to `today.md` Calendar
- **4+ weeks out** → Add to `future/YYYY/MM-month/items.md`

**Item format in items.md**:
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

## Monthly Migration Process

**On the first day of each month, when skill is run:**

1. Check if `future/YYYY/MM-month/items.md` exists for current month
2. If yes, notify user that items need migration
3. User/agent migrates Calendar items to `today.md` Calendar table
4. Add Details to appropriate sections
5. Remove migrated items file

## Marking Items as Done

When user says "mark [X] as done":

1. **Ask for outcome** — Use AskUserQuestion
2. **Create archive entry** (permanent record)
3. **Remove from active sections** (Calendar, Active Projects, This Week)
4. **Add completion note** to today.md (auto-archived next run)
5. **Commit and push** to repository

## Auto-Documentation

**IMPORTANT**: When new information emerges during conversation that should be preserved, document it.

Ask about documenting when:
- New emails/messages are drafted and sent
- New professional connections are made
- Updates to existing relationships
- Career developments (interviews, offers, rejections)
- Project updates or milestones

**ALWAYS use AskUserQuestion before adding documentation:**

```json
{
  "questions": [
    {
      "question": "Should I document this to your personal context?",
      "header": "Document?",
      "options": [
        { "label": "Yes - add update", "description": "Add this information with today's date" },
        { "label": "No - skip", "description": "Don't document this time" }
      ],
      "multiSelect": false
    }
  ]
}
```

## Committing Changes

After making changes:

```bash
cd "$REPO"
git add -A
git commit -m "update: [brief description] - [YYYY-MM-DD]" --quiet
git push origin main --quiet
echo "[OK] Updated personal context"
```

## Privacy & Security

**CRITICAL**:
- Keep your repository PRIVATE
- Never push personal data to public repos
- The skill reads from your local machine; data is ephemeral in conversations
- Your personal context NEVER leaves your machine unless you explicitly `git push`
