# Future Log

**Purpose**: Backlog for items beyond ~4 weeks out. Items here auto-migrate to `today.md` when the month arrives.

---

## Directory Structure

```
future/
├── README.md           # This file
└── 2026/
    ├── 04-april/
    │   └── items.md    # Future items for April 2026
    ├── 05-may/
    │   └── items.md    # Future items for May 2026
    └── 06-june/
        └── items.md    # Future items for June 2026
```

---

## Migration Process

**On the first day of each month**, items from the corresponding month folder in `/future/` are migrated to `today.md`.

- Example: On April 1, items from `future/2026/04-april/items.md` move to `today.md`
- After migration, the month folder can be archived or removed

---

## Adding Future Items

**Rule of thumb**:
- **~4 weeks (28 days) or less** → Add to `today.md` Calendar
- **4+ weeks out** → Add to `future/YYYY/MM-month/items.md`

**Item format** (same as today.md Calendar):
```markdown
| Date | Event | Priority | Status |
|------|-------|----------|--------|
| Apr 30 | Project milestone | High | Planned |
```

Include a `## Details` section for context:
```markdown
## Details

**Project milestone**:
- Detail 1
- Detail 2
- Detail 3
```
