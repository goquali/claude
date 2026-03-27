# Plan: AI Content Saver

## Context

The user finds valuable AI-related content across social media and websites but has no organized way to save and retrieve it. This plan adds an **AI Content Saver** system to the repository — a skill + command that captures URLs or pasted text into a structured, browsable markdown library.

## Architecture: Command + Skill

Following the repo's guidance ("Use commands for workflows instead of standalone agents"), this uses a **Command → Skill** pair without a dedicated agent. The workflow is linear (fetch → extract → write), so an agent layer would add unnecessary indirection.

| Component | File | Purpose |
|-----------|------|---------|
| **Skill** | `.claude/skills/ai-content-saver/SKILL.md` | Core logic: fetch, summarize, save, browse, search |
| **Command** | `.claude/commands/save-ai-content.md` | Entry point with interactive menu |
| **Library** | `ai-content-library/README.md` | Auto-maintained index of saved entries |
| **Entries** | `ai-content-library/entries/*.md` | Individual content files |

## Implementation

### Step 1: Create the Skill (`.claude/skills/ai-content-saver/SKILL.md`)

Frontmatter:
```yaml
---
name: ai-content-saver
description: Save AI-related content from URLs or pasted text into an organized library. Use when the user wants to save, bookmark, or organize AI content.
argument-hint: [URL or "browse" or "search <query>"]
allowed-tools:
  - WebFetch(*)
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(date *)
  - Bash(mkdir *)
  - Bash(cp *)
---
```

Body sections:
1. **Storage Structure** — Define `ai-content-library/entries/YYYY-MM-DD-<slug>.md` for entries, `ai-content-library/images/` for visuals, and `ai-content-library/README.md` for the index
2. **Save from URL** — WebFetch the URL, extract title, generate 2-3 sentence summary, auto-assign tags, create entry file, update index
3. **Save from Pasted Text** — Same flow but skip WebFetch; infer title from content
4. **Save Image/Visual** — Accept image URL (download via WebFetch and save to `ai-content-library/images/YYYY-MM-DD-<slug>.<ext>`) or local file path (copy to images dir). Reference in entry with relative markdown image link.
5. **Browse** — Glob entries, read index, present table sorted newest-first
6. **Search** — Grep across entries for query, present matching results
7. **Tag Taxonomy** — Predefined tags: `AI Models`, `Prompting`, `Agents`, `Research`, `Tools`, `Ethics`, `Industry`, `Tutorials`, `Opinion`
8. **Entry Template** — YAML frontmatter (title, source, date_saved, tags, has_image) + markdown body (Summary, Key Points, Visuals, Original Content)
9. **Index Template** — Table with Date, Title (linked), Tags, Source columns; newest first
10. **Edge Cases** — First run creates directories; duplicate URL detection via Grep; WebFetch failure fallback to paste; slug collision handling; image format validation (png, jpg, gif, webp, svg)

### Step 2: Create the Command (`.claude/commands/save-ai-content.md`)

Frontmatter:
```yaml
---
description: Save and browse AI-related content from URLs or pasted text
argument-hint: [save|browse|search <query>]
allowed-tools:
  - Skill
  - AskUserQuestion
model: sonnet
---
```

Workflow:
1. Parse argument — if URL, browse, or search → invoke skill directly
2. If no argument → AskUserQuestion with menu (Save URL / Paste text / Save image / Browse / Search)
3. For paste mode → collect text via AskUserQuestion, then invoke skill
4. Report result to user

### Step 3: Seed the Library (`ai-content-library/README.md`)

Create initial directory and index file with header + empty table. The skill handles this on first run, but seeding it ensures the directory exists in git.

### Step 4: Update CLAUDE.md

Add a brief section under "Key Components" documenting the AI Content Saver system, following the Weather System documentation pattern.

## Entry File Format

```markdown
---
title: "Example Title"
source: "https://example.com/article"
date_saved: "2026-03-27"
tags: [AI Models, Research]
---

# Example Title

**Source:** [link](https://example.com/article)
**Date Saved:** 2026-03-27
**Tags:** AI Models, Research

## Summary

2-3 sentence summary of the content.

## Key Points

- Key takeaway 1
- Key takeaway 2
- Key takeaway 3

## Visuals

<!-- Optional: images saved with this entry -->
![description](../images/2026-03-27-example-title-1.png)

## Original Content

Relevant excerpt or full pasted text.
```

## Files to Create/Modify

| Action | File |
|--------|------|
| **Create** | `.claude/skills/ai-content-saver/SKILL.md` |
| **Create** | `.claude/commands/save-ai-content.md` |
| **Create** | `ai-content-library/README.md` |
| **Modify** | `CLAUDE.md` (add AI Content Saver section) |

## Verification

1. Run `/save-ai-content` — verify interactive menu appears with 5 options (Save URL, Paste text, Save image, Browse, Search)
2. Run `/ai-content-saver https://some-ai-article-url` — verify entry file and index are created
3. Run `/ai-content-saver browse` — verify saved entries are listed
4. Run `/ai-content-saver search agents` — verify grep-based search works
5. Save an image URL — verify it downloads to `ai-content-library/images/` and entry references it
6. Check `ai-content-library/` for correct file structure and formatting
