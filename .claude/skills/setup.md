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
1. Ask for your skill name (what you'll type to invoke your context, e.g., /john or /jane-smith)
2. Ask exploratory questions to populate your repository
3. Create your skill at `~/.claude/skills/your-name/`
4. Create initial files based on your answers

**Re-running setup is safe.** Existing files are never overwritten - setup only creates new files.

## Step 0: Doctor Check (Read-Only)

Run environment diagnostics before making any changes. No writes, no commits.

```bash
DOCTOR_PASS=0
DOCTOR_WARN=0
DOCTOR_FAIL=0

echo "=== you.md Setup Doctor ==="
echo ""

# 1. Repo path detection
REPO_PATH="$(pwd)"
REPO_NAME="$(basename "$REPO_PATH")"
if [[ -f "$REPO_PATH/today.md" ]] && [[ -f "$REPO_PATH/README.md" ]]; then
  echo "  [PASS] Repo path detected: $REPO_PATH"
  DOCTOR_PASS=$((DOCTOR_PASS + 1))
else
  echo "  [FAIL] Not a valid you.md repo — today.md or README.md missing"
  echo "         Run setup from inside your cloned you.md repo directory"
  DOCTOR_FAIL=$((DOCTOR_FAIL + 1))
fi

# 2. Git state
if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  if [[ -z "$(git branch --show-current 2>/dev/null)" ]]; then
    echo "  [WARN] Git: detached HEAD state — sync may be unsafe"
    DOCTOR_WARN=$((DOCTOR_WARN + 1))
  elif ! git diff --quiet 2>/dev/null || ! git diff --cached --quiet 2>/dev/null; then
    echo "  [WARN] Git: uncommitted changes present — will skip auto-pull"
    DOCTOR_WARN=$((DOCTOR_WARN + 1))
  else
    git fetch origin main --quiet 2>/dev/null
    LOCAL=$(git rev-parse HEAD 2>/dev/null)
    REMOTE=$(git rev-parse origin/main 2>/dev/null)
    if [[ "$LOCAL" == "$REMOTE" ]]; then
      echo "  [PASS] Git: up to date with remote"
      DOCTOR_PASS=$((DOCTOR_PASS + 1))
    elif git merge-base --is-ancestor "$LOCAL" origin/main 2>/dev/null; then
      echo "  [PASS] Git: behind remote — will pull during setup"
      DOCTOR_PASS=$((DOCTOR_PASS + 1))
    elif git merge-base --is-ancestor origin/main "$LOCAL" 2>/dev/null; then
      echo "  [WARN] Git: local is ahead of remote — will skip pull"
      DOCTOR_WARN=$((DOCTOR_WARN + 1))
    else
      echo "  [WARN] Git: diverged from remote — manual resolution may be needed"
      DOCTOR_WARN=$((DOCTOR_WARN + 1))
    fi
  fi
else
  echo "  [WARN] Git: not a git repository — sync features will be disabled"
  DOCTOR_WARN=$((DOCTOR_WARN + 1))
fi

# 3. Skill install path — detect legacy vs normalized
LEGACY_COUNT=$(find "$HOME/.claude" -name "description.md" -path "*/skills@*" 2>/dev/null | wc -l | tr -d ' ')
NORMALIZED_COUNT=$(find "$HOME/.claude/skills" -name "SKILL.md" 2>/dev/null | wc -l | tr -d ' ')

if [[ "$LEGACY_COUNT" -gt 0 ]]; then
  echo "  [WARN] Skill path: $LEGACY_COUNT legacy install(s) found (skills@name/description.md)"
  echo "         Setup will migrate these to the normalized path (skills/name/SKILL.md)"
  DOCTOR_WARN=$((DOCTOR_WARN + 1))
elif [[ "$NORMALIZED_COUNT" -gt 0 ]]; then
  echo "  [PASS] Skill path: normalized installs detected (skills/name/SKILL.md)"
  DOCTOR_PASS=$((DOCTOR_PASS + 1))
else
  echo "  [PASS] Skill path: no existing installs — fresh setup"
  DOCTOR_PASS=$((DOCTOR_PASS + 1))
fi

# 4. Invocation convention check
if grep -r "/@" "$HOME/.claude/skills" --include="*.md" --include="*.txt" -l 2>/dev/null | grep -q .; then
  echo "  [WARN] Invocation: some skill files still use /@name convention (should be /name)"
  DOCTOR_WARN=$((DOCTOR_WARN + 1))
else
  echo "  [PASS] Invocation: /name convention looks correct"
  DOCTOR_PASS=$((DOCTOR_PASS + 1))
fi

# 5. Optional search readiness
if curl -sf http://localhost:7700/health >/dev/null 2>&1; then
  echo "  [PASS] Search: Meilisearch running at localhost:7700 (optional feature available)"
  DOCTOR_PASS=$((DOCTOR_PASS + 1))
else
  echo "  [INFO] Search: Meilisearch not running — search features will be skipped (optional)"
fi

echo ""
echo "=== Doctor Summary: $DOCTOR_PASS PASS | $DOCTOR_WARN WARN | $DOCTOR_FAIL FAIL ==="
echo ""

if [[ "$DOCTOR_FAIL" -gt 0 ]]; then
  echo "  [STOP] Fix FAIL items above before continuing setup."
  exit 1
elif [[ "$DOCTOR_WARN" -gt 0 ]]; then
  echo "  [OK] Warnings detected — setup will proceed safely, see notes above."
else
  echo "  [OK] All checks passed — safe to proceed."
fi
```

## Step 1: Get Your Skill Name

```json
{
  "questions": [
    {
      "question": "What name would you like for your skill? This is the /name you'll use to invoke your context. (e.g., type 'john' for /john, 'jane-smith' for /jane-smith)",
      "header": "Skill Name",
      "multiSelect": false
    }
  ]
}
```

Store as `SKILL_NAME` (lowercase, with hyphens for spaces).

## Step 1.5: Rerun Detection (now that SKILL_NAME is known)

```bash
# Check normalized path first, then legacy path
SKILL_FILE_NEW="$HOME/.claude/skills/$SKILL_NAME/SKILL.md"
SKILL_FILE_LEGACY="$HOME/.claude/skills@$SKILL_NAME/description.md"

if [[ -f "$SKILL_FILE_NEW" ]]; then
  RERUN=true
  echo "[INFO] Re-running setup — skill already installed at $SKILL_FILE_NEW"
  echo "[INFO] Existing files will NOT be overwritten — only new files will be created"
elif [[ -f "$SKILL_FILE_LEGACY" ]]; then
  RERUN=true
  echo "[INFO] Re-running setup — legacy skill found at $SKILL_FILE_LEGACY"
  echo "[INFO] Will migrate to normalized path during Step 2"
else
  RERUN=false
  echo "[INFO] Fresh install — creating new skill for /$SKILL_NAME"
fi
```

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
# Normalized install path (current convention)
SKILL_DIR="$HOME/.claude/skills/$SKILL_NAME"
SKILL_FILE="$SKILL_DIR/SKILL.md"

# Legacy path (old convention — skills@name/description.md)
LEGACY_DIR="$HOME/.claude/skills@$SKILL_NAME"
LEGACY_FILE="$LEGACY_DIR/description.md"

# Detect and migrate legacy install if present
if [[ -f "$LEGACY_FILE" ]] && [[ ! -f "$SKILL_FILE" ]]; then
  echo "[MIGRATE] Legacy skill detected at $LEGACY_FILE"
  mkdir -p "$SKILL_DIR"
  cp "$LEGACY_FILE" "$SKILL_FILE"
  echo "[MIGRATE] Copied to normalized path: $SKILL_FILE"
  echo "[MIGRATE] Legacy file preserved at $LEGACY_FILE (delete manually once confirmed working)"
elif [[ -f "$SKILL_FILE" ]]; then
  echo "[INFO] Skill already installed at $SKILL_FILE — re-running setup"
fi

# Install or update skill
mkdir -p "$SKILL_DIR"
cp .claude/skills/personal-context.md "$SKILL_FILE"

# Replace placeholders
# Convert skill-name to readable name (e.g., "john-smith" → "John Smith")
READABLE_NAME=$(echo "$SKILL_NAME" | sed -E 's/-/ /g; s/\b(.)/\u\1/g')

if [[ "$(uname -s)" == "Darwin" ]]; then
  sed -i '' "s/\*\*\[YOUR_NAME\]\*\*/$READABLE_NAME/g" "$SKILL_FILE"
  sed -i '' "s|\[REPO_PATH\]|$REPO_PATH|g" "$SKILL_FILE"
  sed -i '' "s/name: personal-context/name: $SKILL_NAME/g" "$SKILL_FILE"
else
  sed -i "s/\*\*\[YOUR_NAME\]\*\*/$READABLE_NAME/g" "$SKILL_FILE"
  sed -i "s|\[REPO_PATH\]|$REPO_PATH|g" "$SKILL_FILE"
  sed -i "s/name: personal-context/name: $SKILL_NAME/g" "$SKILL_FILE"
fi
```

Confirm:
```
[OK] Skill installed at ~/.claude/skills/$SKILL_NAME/SKILL.md
[OK] Invoke with: /$SKILL_NAME
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
- Skill: /$SKILL_NAME
- Repository: $REPO_PATH
- Files created: [count] files based on your answers

Files created:
- [list each file created with brief description, one per line]

Repository path:
$REPO_PATH

Next steps:
1. Use /$SKILL_NAME to load your context in any conversation
2. Edit files anytime to add more information
3. ⚠️ Only push if your repo is PRIVATE: git push origin main
```
