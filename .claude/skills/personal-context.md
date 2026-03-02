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
ISIAH_MD="[REPO_PATH]"
[ ! -d "$ISIAH_MD" ] && ISIAH_MD="[REPO_PATH]"
cd "$ISIAH_MD" && git pull origin main --quiet

# Get current date info
CURRENT_DATE=$(date "+%Y-%m-%d")
CURRENT_MONTH=$(date "+%B %Y")
CURRENT_MONTH_NUM=$(date "+%m")
CURRENT_MONTH_LOWER=$(date "+%B" | tr '[:upper:]' '[:lower:]')
ARCHIVE_DIR="$ISIAH_MD/archive/$(date +%Y)/$(date +%m)-$(date +%B | tr '[:upper:]' '[:lower:]')"
mkdir -p "$ARCHIVE_DIR"

# Update Last Updated header in today.md
sed -i '' "s/\*\*Last Updated\*\*: .*/\*\*Last Updated\*\*: $CURRENT_MONTH/" "$ISIAH_MD/today.md"

# Find and archive past completed entries (before today)
grep -n "Completed.*2026" "$ISIAH_MD/today.md" 2>/dev/null | while read -r line; do
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
sed -i '' '/Completed.*2026/d' "$ISIAH_MD/today.md" 2>/dev/null || true

# Check if future/ has items for current month
FUTURE_MONTH_DIR="$ISIAH_MD/future/$(date +%Y)/${CURRENT_MONTH_NUM}-${CURRENT_MONTH_LOWER}"
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
        { "label": "Quiet mode", "description": "Load core files (today.md, SUMMARY.md, README.md) silently, wait for your request" },
        { "label": "Work on to-do items", "description": "Load context and show your immediate priorities as a checklist from today.md" },
        { "label": "Question mode", "description": "Ask about task type and context needs, then load targeted files" }
      ],
      "multiSelect": false
    }
  ]
}
```

### Step 2: If Quiet Mode

Read core files in parallel (use detected path from Step 0):
```bash
$ISIAH_MD/today.md          # Current life context (read FIRST)
$ISIAH_MD/SUMMARY.md        # Overview & recent updates
$ISIAH_MD/README.md         # Coaching style, tech stack, structure
```

Then summarize:
```
[OK] Loaded personal context
  Current: [from today.md]
  Ready
```

### Step 2.5: If "Work on to-do items" Mode

After loading core files (same as Quiet Mode), present the immediate priorities from today.md as a checklist using AskUserQuestion.

Parse the "This Week" section and "PRISM Frontend Tasks" (if exists) from today.md to build the checklist dynamically.

After selection, summarize:
```
[OK] Selected: [items user selected]
Ready to work. What's first?
```

### Step 3: If Question Mode

Use **AskUserQuestion** to understand task type:

```json
{
  "questions": [
    {
      "question": "What type of task are you working on?",
      "header": "Task Type",
      "options": [
        { "label": "Writing/Communication", "description": "Emails, messages, documents, personal writing" },
        { "label": "Professional/Career", "description": "Resume, applications, networking, job search" },
        { "label": "Relationship/Social", "description": "Communication with specific person, social context" },
        { "label": "Academic/Thesis", "description": "School work, research, thesis" }
      ],
      "multiSelect": false
    },
    {
      "question": "What's most important for me to know for this task?",
      "header": "Priority Context",
      "options": [
        { "label": "Current situation", "description": "What's happening right now (today.md)" },
        { "label": "Goals & values", "description": "What matters most, long-term direction" },
        { "label": "Specific person", "description": "Context about someone you're working with" }
      ],
      "multiSelect": true
    }
  ]
}
```

### Step 4: Conditionally Load Based on Task Type

**For writing/communication tasks:**
- `personal/values.md` - Core principles
- `personal/traits.md` - Personality traits
- `writing/README.md` - Writing style reference

**For professional/career tasks:**
- `professional/experience.md` - Work history
- `professional/skills.md` - Capabilities
- `professional/goals.md` - Career objectives
- `professional/contacts-to-ask.md` - Networking targets

**For relationship/communication tasks:**
- Ask which person, then read their profile:
- `relationships/friends/[name].md` - Specific friend context
- `relationships/mentors/[name].md` - Mentor context
- `relationships/connections/[name].md` - Professional connections

**For academic tasks:**
- `academic/academic-achievements.md` - Research, publications

**For development work:**
- See README.md for "Preferred Tech Stack" section

**For planning/strategy tasks:**
- `personal/annual-goals/[YEAR].md` - Current year goals
- `personal/values.md` - Core principles
- `professional/goals.md` - Career objectives

### Step 5: Confirm Context Loaded

After reading, summarize:
```
[OK] Loaded personal context
  Task: [selected]
  Current: [from today.md]
  Read [X] files
```

## today.md Structure

The `today.md` file is your current life context. Maintain this structure:

```markdown
# Today - Current Life Context

**Last Updated**: [Month Year]

---

## Calendar

### Upcoming

| Date | Event | Priority | Status |
|------|-------|----------|--------|
| Mar 2 | Event name | High | Scheduled |

### This Week (Mar 2-8)

- [ ] Task name
- [ ] Another task

### Project Tasks (Optional)

- [ ] Project specific task
- [ ] Another project task

---

## Active Projects

| Project | Type | Status | Next Action |
|---------|------|--------|-------------|
| Project Name | Type | Status | Next action |

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

2. **This Week checklist** - Add line: `- [ ] Task name`

3. **Active Projects table** - Add row with format: `| Project | Type | Status | Next Action |`

**When REMOVING items:**

- Use exact string matching
- For checklist: `sed -i '' '/- \[ \] Exact task text/d' "$ISIAH_MD/today.md"`
- For tables: `sed -i '' '/| Exact Project Name |/d' "$ISIAH_MD/today.md"`

**When UPDATING items:**

- Replace entire table row, not partial edits
- For status changes: rewrite the full row

## Repository Structure

```
your-name.md/
├── today.md              # READ FIRST - current life context
├── SUMMARY.md            # Overview & recent updates
├── README.md             # Coaching style, tech stack, structure
├── archive/              # Past events by year/month
│   └── 2026/
│       ├── 01-january/
│       ├── 02-february/
│       └── 03-march/
│           └── event-name.md
├── future/               # Future items beyond ~4 weeks
│   └── 2026/
│       ├── 04-april/
│       ├── 05-may/
│       └── README.md
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
cd "$ISIAH_MD"
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
