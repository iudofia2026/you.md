---
name: uninstall
description: >
  Remove your personal context skill and optionally delete the repository.
category: setup
priority: 90
---

# Uninstall Personal Context

This skill removes your personal context setup.

## Step 1: Detect Repository

```bash
if [[ -f "README.md" ]] && grep -q "Personal Context Template" README.md 2>/dev/null; then
  REPO_PATH="$(pwd)"
  REPO_NAME="$(basename "$REPO_PATH")"
else
  # Try to find repo by name pattern
  for dir in ~/github/*.md; do
    if [[ -f "$dir/README.md" ]] && grep -q "Personal Context Template" "$dir/README.md" 2>/dev/null; then
      REPO_PATH="$dir"
      REPO_NAME="$(basename "$dir")"
      break
    fi
  done
fi

if [[ -z "$REPO_PATH" ]]; then
  echo "[ERROR] Could not find your personal context repository."
  echo "Please run this from within your repository directory."
  exit 1
fi

# Get skill name from repo name (remove .md extension)
SKILL_NAME="${REPO_NAME%.md}"
```

Show what will be removed:
```
[INFO] Found personal context:
- Repository: $REPO_PATH
- Skill name: $SKILL_NAME
- Skill location: ~/.claude/skills@$SKILL_NAME/
```

## Step 2: Ask What to Remove

```json
{
  "questions": [
    {
      "question": "What would you like to remove?",
      "header": "Uninstall Options",
      "options": [
        {
          "label": "Remove skill only",
          "description": "Keep the repository, just remove the ~/.claude/skills/ directory"
        },
        {
          "label": "Remove skill and repo",
          "description": "Delete both the skill AND the repository directory"
        },
        {
          "label": "Cancel",
          "description": "Keep everything, exit uninstall"
        }
      ],
      "multiSelect": false
    }
  ]
}
```

## Step 3: Confirm Deletion

```json
{
  "questions": [
    {
      "question": "This will permanently delete data. Are you sure?",
      "header": "Confirm",
      "options": [
        {
          "label": "Yes, delete it",
          "description": "Permanently remove selected items"
        },
        {
          "label": "No, cancel",
          "description": "Keep everything"
        }
      ],
      "multiSelect": false
    }
  ]
}
```

## Step 4: Execute Removal

**Remove skill:**
```bash
SKILL_DIR="$HOME/.claude/skills@$SKILL_NAME"
if [[ -d "$SKILL_DIR" ]]; then
  rm -rf "$SKILL_DIR"
  echo "[OK] Removed skill: $SKILL_DIR"
else
  echo "[INFO] Skill directory not found: $SKILL_DIR"
fi
```

**Remove repository (if selected):**
```bash
if [[ "$REMOVE_REPO" == "true" ]]; then
  cd ~
  rm -rf "$REPO_PATH"
  echo "[OK] Removed repository: $REPO_PATH"
fi
```

## Final Confirmation

```
[OK] Uninstall complete!

Removed:
- Skill: ~/.claude/skills@$SKILL_NAME/
$([ "$REMOVE_REPO" == "true" ] && echo "- Repository: $REPO_PATH")

Your personal context has been removed.
The @$SKILL_NAME command is no longer available.
```
