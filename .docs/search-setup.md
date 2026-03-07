# Search Setup Guide

This guide explains how to add **Meilisearch** to your personal context system for instant search across all your files.

## What This Gives You

- **Instant search** across your entire knowledge base
- **Fast results** - milliseconds, not seconds
- **Relevant ranking** - best matches first
- **Used by your skill** to find people, goals, topics, anything

## Prerequisites

- Docker installed
- `fnox` installed (optional - for secret management, see below)
- Your personal context repository set up

## Quick Setup (5 Steps)

### 1. Create Meilisearch Directory

```bash
mkdir -p ~/github/meilisearch
cd ~/github/meilisearch
```

### 2. Create docker-compose.yml

```yaml
version: "3.8"

services:
  meilisearch:
    image: getmeili/meilisearch:v1.5
    container_name: meilisearch
    ports:
      - "7700:7700"
    environment:
      - MEILI_MASTER_KEY=your-master-key-here
      - MEILI_ENV=production
    volumes:
      - ./meili_data:/meili_data.ms
    restart: unless-stopped

volumes:
  meili_data:
```

### 3. Start Meilisearch

```bash
docker-compose up -d
```

Verify it's running:
```bash
curl http://localhost:7700/health
```

Should return: `{"status":"available"}`

### 4. Install Indexer Script

Create `~/github/meilisearch/index.sh`:

```bash
#!/bin/bash

REPO_PATH="$HOME/github/your-name.md"
INDEX_NAME="your_name_documents"

echo "Indexing $REPO_PATH to Meilisearch..."

# Find all markdown files
find "$REPO_PATH" -name "*.md" -type f | while read -r file; do
  # Get relative path
  rel_path="${file#$REPO_PATH/}"

  # Extract title (first heading or filename)
  title=$(grep -m 1 "^# " "$file" | sed 's/^# //' || basename "$file" .md)

  # Get first few lines as preview
  preview=$(head -n 10 "$file" | tr '\n' ' ')

  # Add to index
  curl -s -X POST "http://localhost:7700/indexes/$INDEX_NAME/documents" \
    -H "Authorization: Bearer YOUR_MASTER_KEY" \
    -H "Content-Type: application/json" \
    -d "[
      {
        \"id\": \"$(echo "$rel_path" | md5sum | cut -d' ' -f1)\",
        \"path\": \"$rel_path\",
        \"title\": \"$title\",
        \"content\": \"$(cat "$file" | jq -Rs .)\"
      }
    ]" &
done

wait
echo "Indexing complete!"
```

Make it executable:
```bash
chmod +x ~/github/meilisearch/index.sh
```

### 5. Update Your Skill

Add the Meilisearch search section to your skill file:

```bash
# Search via Meilisearch
curl -s -X POST "http://localhost:7700/indexes/your_name_documents/search" \
  -H "Authorization: Bearer YOUR_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{"q": "query", "limit": 5}' | jq '.hits[] | {path, title}'
```

Add health check to your skill's sync step:

```bash
# Verify Meilisearch
if ! curl -sf http://localhost:7700/health > /dev/null 2>&1; then
  echo "[WARN] Meilisearch not running"
fi
```

## Secret Management with fnox (Optional)

If you use **fnox** for secret management:

### Install fnox

```bash
brew tap yourusername/tap  # Or install fnox from your source
brew install fnox
```

### Store Your Master Key

```bash
fnox set MEILI_MASTER_KEY "your-master-key-here"
```

### Update docker-compose.yml

```yaml
environment:
  - MEILI_MASTER_KEY=${MEILI_MASTER_KEY}
```

### Update Your Skill

```bash
# Verify Meilisearch
if ! curl -sf http://localhost:7700/health > /dev/null 2>&1; then
  echo "[WARN] Meilisearch not running"
elif ! fnox get MEILI_MASTER_KEY > /dev/null 2>&1; then
  echo "[WARN] fnox MEILI_MASTER_KEY not accessible"
fi
```

And use in search:
```bash
-H "Authorization: Bearer $(fnox get MEILI_MASTER_KEY)" \
```

## Running the Indexer

Run the indexer whenever you add or update files:

```bash
~/github/meilisearch/index.sh
```

Or set up a cron job to run automatically:

```bash
# Edit crontab
crontab -e

# Add line to index daily at 2am
0 2 * * * ~/github/meilisearch/index.sh
```

## Using Search from Your Skill

Your skill can now search your context:

**When to search:**
- "Who is [Name]?"
- "What are my goals?"
- "Find where I mention [topic]"

**Don't search for:**
- Content already in today.md, SUMMARY.md, README.md (already loaded)
- File navigation (use Glob instead)

## Troubleshooting

### Meilisearch won't start

```bash
# Check Docker is running
docker ps

# Check logs
docker-compose logs meilisearch

# Restart
docker-compose restart
```

### Search returns no results

```bash
# Check index exists
curl http://localhost:7700/indexes

# Re-run indexer
~/github/meilisearch/index.sh
```

### Permission denied

```bash
# Make script executable
chmod +x ~/github/meilisearch/index.sh

# Make sure master key is correct
curl http://localhost:7700/keys
```

## Advanced Configuration

### Configure Search Settings

```bash
# Update index settings for better ranking
curl -X PATCH 'http://localhost:7700/indexes/your_name_documents/settings' \
  -H "Authorization: Bearer YOUR_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "searchableAttributes": ["title", "content"],
    "rankingRules": ["words", "typo", "proximity", "attribute", "sort", "exactness"],
    "stopWords": ["the", "a", "an"],
    "synonyms": {
      "goal": ["objective", "target", "aim"]
    }
  }'
```

### Add Filters

Add custom fields to filter by:

```bash
"d": {
  "category": "personal",
  "tags": ["important", "reference"]
}
```

Then search with filters:
```bash
-d '{"q": "sleep", "filter": "category = personal"}'
```

## Summary

With Meilisearch set up:
- **Instant search** across your entire knowledge base
- **Used by your skill** to find relevant context
- **Auto-indexing** via cron job
- **Secure** with master key protection

Your personal context becomes searchable in milliseconds instead of scanning files manually.
