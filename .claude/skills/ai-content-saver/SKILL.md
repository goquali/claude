---
name: ai-content-saver
description: Save AI-related content from URLs or pasted text into an organized library. Use when the user wants to save, bookmark, or organize AI content. Also supports browsing and searching saved entries.
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

# AI Content Saver Skill

Save, browse, and search AI-related content in a structured local library.

## Storage Structure

All content is stored under `ai-content-library/` in the repository root:

```
ai-content-library/
├── README.md              # Browsable index (auto-maintained)
├── entries/               # Individual content files
│   └── YYYY-MM-DD-<slug>.md
└── images/                # Saved visuals
    └── YYYY-MM-DD-<slug>.<ext>
```

**First run:** If `ai-content-library/` does not exist, create the directory structure and seed `README.md` with the index template before proceeding.

## Operations

Parse the argument to determine the operation:

- **URL** (starts with `http`) → Save from URL
- **`browse`** → Browse saved content
- **`search <query>`** → Search saved content
- **Text blob or no argument** → Save from pasted text (ask user for text if not provided)

---

### Operation 1: Save from URL

1. **Check for duplicates** — Use `Grep` to search for the URL across `ai-content-library/entries/`. If found, inform the user and ask whether to update or skip.
2. **Fetch content** — Use `WebFetch` to retrieve the page content. If fetch fails (paywall, 404, timeout), inform the user and offer to save from pasted text instead.
3. **Extract metadata:**
   - **Title**: Extract from the page content (HTML title, h1, or og:title)
   - **Summary**: Generate a 2-3 sentence summary of the AI-related content
   - **Key Points**: Extract 3-5 bullet points capturing the main takeaways
   - **Tags**: Auto-assign 1-3 tags from the taxonomy below based on content analysis
4. **Generate filename** — Create slug from title: lowercase, replace spaces with hyphens, remove special characters, truncate to 50 characters. Format: `YYYY-MM-DD-<slug>.md`
   - If file exists, append `-2`, `-3`, etc.
5. **Create entry file** — Write to `ai-content-library/entries/YYYY-MM-DD-<slug>.md` using the entry template below
6. **Update index** — Read `ai-content-library/README.md`, prepend a new row to the entries table (newest first)
7. **Report** — Tell the user: title, tags assigned, and file path

### Operation 2: Save from Pasted Text

Same as Save from URL, but:
- Skip the WebFetch step
- Use the pasted text as the content to analyze
- Set source to `Manual Entry` in the entry
- If the title is not obvious from the text, ask the user to provide one

### Operation 3: Save Image/Visual

When the user provides an image URL or local file path:

1. **Determine source type:**
   - **Image URL** (ends in `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.svg`, or is an image hosting URL) — Use `WebFetch` to download, then write to `ai-content-library/images/YYYY-MM-DD-<slug>.<ext>`
   - **Local file path** — Use `Bash(cp ...)` to copy to `ai-content-library/images/YYYY-MM-DD-<slug>.<ext>`
2. **Validate format** — Only accept: png, jpg, jpeg, gif, webp, svg
3. **Create entry file** — Use the entry template with a `## Visuals` section containing a relative markdown image link: `![description](../images/<filename>)`
4. **Ask user** for a title and brief description if not provided
5. **Update index** — Same as URL flow

### Operation 4: Browse Saved Content

1. Read `ai-content-library/README.md`
2. Present the index table to the user, sorted by date (newest first)
3. If no entries exist, inform the user the library is empty

### Operation 5: Search Saved Content

1. Use `Grep` to search across `ai-content-library/entries/` for the query term
2. Read matching files and present results with: title, date, tags, and matching excerpt
3. If no matches, suggest alternative search terms or offer to browse all content

---

## Tag Taxonomy

Assign 1-3 tags per entry from this list:

| Tag | Use When Content Covers |
|-----|------------------------|
| `AI Models` | Model releases, benchmarks, capabilities, architecture |
| `Prompting` | Prompt engineering, techniques, templates |
| `Agents` | Agentic workflows, autonomous systems, tool use |
| `Research` | Papers, academic findings, experiments |
| `Tools` | Developer tools, frameworks, libraries, SDKs |
| `Ethics` | Safety, alignment, regulation, bias |
| `Industry` | Business applications, market trends, adoption |
| `Tutorials` | How-to guides, walkthroughs, courses |
| `Opinion` | Thought pieces, predictions, commentary |

## Entry Template

```markdown
---
title: "<Title>"
source: "<URL or Manual Entry>"
date_saved: "YYYY-MM-DD"
tags: [tag1, tag2]
---

# <Title>

**Source:** [link](<URL>) or "Manual Entry"
**Date Saved:** YYYY-MM-DD
**Tags:** tag1, tag2

## Summary

<2-3 sentence summary>

## Key Points

- <point 1>
- <point 2>
- <point 3>

## Visuals

<!-- Include only if images were saved with this entry -->
![description](../images/<image-filename>)

## Original Content

<Relevant excerpt or full pasted text>
```

## Index Template

The file `ai-content-library/README.md` should follow this format:

```markdown
# AI Content Library

A curated collection of AI-related content saved from across the web.

## Entries

| Date | Title | Tags | Source |
|------|-------|------|--------|
| YYYY-MM-DD | [Title](entries/YYYY-MM-DD-slug.md) | tag1, tag2 | [link](URL) |
```

New entries are **prepended** to the table (newest first). If the file does not exist, create it with the header and an empty table.

## Rules

- Always use today's date (YYYY-MM-DD format) for `date_saved`
- Ensure directories exist before writing files (`mkdir -p` if needed)
- Keep summaries concise — 2-3 sentences maximum
- Preserve original content accurately — do not alter quotes or excerpts
- When saving images, always create a corresponding entry file that references the image
