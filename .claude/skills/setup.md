---
name: setup
description: >
  Initialize your personal context repository. Creates your skill,
  and runs through exploratory questions to populate your repository with your information.
category: setup
priority: 100
---

# Personal Context Setup

This skill sets up your personal context repository. It will:
1. Ask for your skill name (what you'll type to invoke your context, e.g., /@john)
2. Ask exploratory questions to populate your repository
3. Create your skill at `~/.claude/skills/your-name/`
4. Create initial files based on your answers

**Re-running setup is safe.** Existing files are never overwritten - setup only creates new files.

## Step 0: Detect if Re-running

```bash
# Check if setup has been run before
SKILL_DIR="$HOME/.claude/skills@$SKILL_NAME"
if [[ -d "$SKILL_DIR" ]]; then
  RERUN=true
  echo "[INFO] Re-running setup - will add new files without overwriting existing ones"
else
  RERUN=false
fi
```

## Step 1: Get Your Skill Name

```json
{
  "questions": [
    {
      "question": "What name would you like for your skill? This is the /@name you'll use to invoke your context. (e.g., type 'john' for /@john, 'jane-smith' for /@jane-smith)",
      "header": "Skill Name",
      "multiSelect": false
    }
  ]
}
```

Store as `SKILL_NAME` (lowercase, with hyphens for spaces).

## Privacy Check

**Before continuing, confirm your repository is PRIVATE:**

```json
{
  "questions": [
    {
      "question": "Is your GitHub fork set to PRIVATE? Your personal data should never be public. (If unsure, go to your fork on GitHub → Settings → Change visibility → Make private)",
      "header": "Privacy Check",
      "options": [
        { "label": "Yes, it's private", "description": "My fork is set to private, continue setup" },
        { "label": "I'll do it now", "description": "Pause setup while I make it private" }
      ],
      "multiSelect": false
    }
  ]
}
```

IF USER SELECTS "I'll do it now": Show instructions and wait for them to confirm they've made it private before continuing.

Detect repository path:
```bash
REPO_PATH="$(pwd)"
REPO_NAME="$(basename "$REPO_PATH")"
```

## Step 2: Create the Skill

```bash
SKILL_DIR="$HOME/.claude/skills@$SKILL_NAME"
mkdir -p "$SKILL_DIR"
cp .claude/skills/personal-context.md "$SKILL_DIR/description.md"

# Replace placeholders
# Convert skill-name to readable name (e.g., "john-smith" → "John Smith")
READABLE_NAME=$(echo "$SKILL_NAME" | sed -E 's/-/ /g; s/\b(.)/\u\1/g')

if [[ "$(uname -s)" == "Darwin" ]]; then
  sed -i '' "s/\*\*\[YOUR_NAME\]\*\*/$READABLE_NAME/g" "$SKILL_DIR/description.md"
  sed -i '' "s|\[REPO_PATH\]|$REPO_PATH|g" "$SKILL_DIR/description.md"
  sed -i '' "s/name: personal-context/name: $SKILL_NAME/g" "$SKILL_DIR/description.md"
else
  sed -i "s/\*\*\[YOUR_NAME\]\*\*/$READABLE_NAME/g" "$SKILL_DIR/description.md"
  sed -i "s|\[REPO_PATH\]|$REPO_PATH|g" "$SKILL_DIR/description.md"
  sed -i "s/name: personal-context/name: $SKILL_NAME/g" "$SKILL_DIR/description.md"
fi
```

Confirm:
```
[OK] Skill created as /@$SKILL_NAME
```

## Step 3: Exploratory Questions

Ask questions with only two options: "Skip for now" and "Remove this section".
User types their answer directly in the text input.

**CRITICAL**: When user provides an answer (not "Skip" or "Remove"), CREATE the corresponding file with their answer.

**Re-run safety**: If a file already exists, skip the question and note that it exists.

### Personal Information

**Core Values:**
```json
{
  "questions": [
    {
      "question": "What are your core values? These are the principles that guide your decisions and actions. (Type your answer below, or select Skip/Remove)",
      "header": "Core Values",
      "options": [
        { "label": "Skip for now", "description": "Skip this question" },
        { "label": "Remove this section", "description": "Don't include values" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `personal/values.md` with their answer.

**Personality Traits:**
```json
{
  "questions": [
    {
      "question": "What are your key personality traits? How would you describe yourself and how you work? (Type below or select Skip/Remove)",
      "header": "Personality Traits",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include traits" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `personal/traits.md` with their answer.

**Contact:**
```json
{
  "questions": [
    {
      "question": "What's your preferred email address? (Type below or select Skip/Remove)",
      "header": "Contact",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include contact" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `personal/contact.md` with their answer.

### Professional

**Current Role:**
```json
{
  "questions": [
    {
      "question": "What's your current work situation? (Job title, company, student status. Type below or select Skip/Remove)",
      "header": "Current Role",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include professional info" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `professional/experience.md` with their answer.

**Skills:**
```json
{
  "questions": [
    {
      "question": "What are your key skills? (Type below or select Skip/Remove)",
      "header": "Skills",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include skills" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `professional/skills.md` with their answer.

**Career Goals:**
```json
{
  "questions": [
    {
      "question": "What are your career goals? (Type below or select Skip/Remove)",
      "header": "Career Goals",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include goals" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `professional/goals.md` with their answer.

### Education

```json
{
  "questions": [
    {
      "question": "What's your educational background? (Type below or select Skip/Remove)",
      "header": "Education",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include education" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `academic/education.md` with their answer.

### Current Context

```json
{
  "questions": [
    {
      "question": "What's happening in your life right now? (Type below or select Skip/Remove)",
      "header": "Current Context",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't create today.md" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ANSWERS: Create `today.md` with their answer.

### Relationships

```json
{
  "questions": [
    {
      "question": "Are there key people in your life you'd like to document now? (Type names below or select Skip/Remove)",
      "header": "Relationships",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Remove this section", "description": "Don't include relationships" }
      ],
      "multiSelect": false
    }
  ]
}
```
IF USER ENTERS NAMES: For each person, ask category (family/friend/mentor/connection), ask for details, then create file in appropriate `relationships/` subdirectory.

### Custom Files

Ask if they want to create any additional files:

```json
{
  "questions": [
    {
      "question": "Do you want to create any custom files? (Type filename like 'habits.md' or select Skip/Remove. You can create files in other/, or anywhere in the repo)",
      "header": "Custom Files",
      "options": [
        { "label": "Skip for now", "description": "Skip" },
        { "label": "Done adding files", "description": "No more custom files needed" }
      ],
      "multiSelect": false
    }
  ]
}
```

IF USER ENTERS A FILENAME: Ask what content to put in it, then create the file. Loop back and ask if they want to add another custom file.

## Step 4: Commit and Confirm

```bash
cd "$REPO_PATH"
git add -A
git commit -m "Initial setup: Personal context" 2>/dev/null || true
```

Final confirmation:
```
[OK] Personal context is up and running!

Your personal context is ready:
- Skill: /@$SKILL_NAME
- Repository: $REPO_PATH
- Files created: [count] files based on your answers

Files created:
- [list each file created with brief description, one per line]

Repository path:
$REPO_PATH

Next steps:
1. Use /@$SKILL_NAME to load your context in any conversation
2. Edit files anytime to add more information
3. ⚠️ Only push if your repo is PRIVATE: git push origin main
```
