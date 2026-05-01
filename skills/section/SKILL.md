---
name: paper-to-narrative-section
description: Split INFO_SOURCE.md into topical sections.
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, section, split]
---

# Step B: Section

## Purpose

Break the converted research paper into manageable sections that will become narrative chapters.

## Inputs

- `workspace/SOURCES/INFO_SOURCE.md`

## Outputs

- `workspace/SECTIONS/NN_title.md` (one per section)
- `workspace/SECTIONS/sections_index.md`

## Instructions

1. Read `workspace/SOURCES/INFO_SOURCE.md`.
2. Split the text into sections based on markdown headings (`#`, `##`, `###`).
3. If the document has fewer than 2 headings, split by paragraph boundaries instead. Group paragraphs into chunks of roughly 500-2000 words. Preserve paragraph boundaries.
4. Merge adjacent tiny sections (under 200 words) into the next section until the minimum is met. Never exceed 2000 words per section.
5. For each section, write a file in `workspace/SECTIONS/` named `NN_title.md` with YAML frontmatter:
   ```yaml
   ---
   section: 1
   title: "Introduction"
   words: 452
   ---
   ```
   Use 2-digit zero-padded numbers for `NN`.
6. Write `workspace/SECTIONS/sections_index.md` listing each section with title, file link, and word count.
7. Mark step B complete in `workspace/CHECKLISTS/pipeline_checklist.md`.

## Rules

- Preserve all content. Nothing is dropped.
- Section numbers must be monotonic integers starting at 1.
- The `title` frontmatter field must exist and be non-empty.
