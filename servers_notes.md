# Servers notes

This file complements `servers_manual.md`.

- `servers_manual.md` should describe stable facts: hardware, access rules, storage layout, scheduler rules, shell defaults, and common environment/cache conventions.
- This file should record short project-local notes that are not stable enough for the manual yet, or pitfalls that repeatedly affect multiple projects.

Do not put credentials, private hostnames, private usernames, internal paths, or full incident logs here. If a note only matters to one workspace, put it in that workspace's `STATE.md`, `experiment-log.md`, or local runbook instead.

## Entry format

Prepend new entries under the relevant section, newest first.

```md
#### YYYY-MM-DD - Short title
[3-5 lines: reusable symptom / likely cause / fix. Prefer a command or checklist that another user can copy after filling in their own paths.]
```

Stale entries can be marked with `~~strikethrough~~`. Delete them only after they are clearly obsolete.

---

## Global notes

### Configuration

#### YYYY-MM-DD - Title
Template entry.

### Pitfalls

#### YYYY-MM-DD - Title
Template entry.

## Server group: <name>

### Configuration

#### YYYY-MM-DD - Title
Template entry.

### Pitfalls

#### YYYY-MM-DD - Title
Template entry.

## External APIs / hosted compute

### Provider: <name>

#### YYYY-MM-DD - Title
Template entry.
