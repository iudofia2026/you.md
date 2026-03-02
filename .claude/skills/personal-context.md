---
name: personal-context
description: >
  Load your personal context repository for personalized assistance.
category: personal
priority: 10
---

You are loading context about **[YOUR_NAME]** from their personal knowledge repository.

**REPO_PATH**: `[REPO_PATH]`

## Quick Load Workflow

### Sync Repo

```bash
cd "[REPO_PATH]" && git pull origin main --quiet 2>/dev/null || true
echo "[SYNC] Context synced"
```

### Ask Load Mode

```json
{
  "questions": [
    {
      "question": "How should I load your context?",
      "header": "Load Mode",
      "options": [
        { "label": "Quiet mode", "description": "Load core files silently" },
        { "label": "Question mode", "description": "Ask what to load" }
      ],
      "multiSelect": false
    }
  ]
}
```

**Quiet Mode**: Read `today.md`, `README.md`, `personal/values.md`, `personal/traits.md`

**Question Mode**: Ask task type, then load relevant files.

## After Helping: Auto-Documentation

**IMPORTANT**: After providing assistance, check if new information was learned about the user.

If any of the following happened during the conversation:
- User shared new preferences, opinions, or values
- User mentioned life updates, goals, or circumstances
- User referenced new relationships, jobs, or experiences
- User expressed likes, dislikes, or quirks

**THEN offer to document it:**

```json
{
  "questions": [
    {
      "question": "I learned some new things about you during our conversation. Should I document this to your personal context?",
      "header": "Document?",
      "options": [
        { "label": "Yes - save it", "description": "Add to your context repository" },
        { "label": "No - skip", "description": "Don't document this time" }
      ],
      "multiSelect": false
    }
  ]
}
```

If user says "Yes":
1. Determine which file the information belongs in (values.md, traits.md, today.md, relationships/, etc.)
2. Ask user to confirm the filename
3. Append the new information with today's date
4. Show what was added

Example output when documenting:
```
[OK] Updated your personal context:
- Added to: personal/traits.md
- Added: "Prefers working in the morning, coffee-powered"
```

Only offer documentation when genuinely new information was revealed. Don't ask for every single interaction - use judgment.
