---
name: paper-to-narrative-section
description: "Split INFO_SOURCE.md into topical sections."
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
- `workspace/CONFIG/settings.md` (for max_words_per_section, min_words_per_section)

## Outputs

- `workspace/SECTIONS/NN_title.md` (one per section)
- `workspace/SECTIONS/sections_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SOURCES/INFO_SOURCE.md` exists and is non-empty.
2. `workspace/SECTIONS/` exists or can be created.

If INFO_SOURCE.md is missing or empty, abort: "Step B: INFO_SOURCE.md not found or empty. Run Step A first."

## Instructions

1. Read `workspace/SOURCES/INFO_SOURCE.md`.
2. Load config from `workspace/CONFIG/settings.md` (use defaults if missing; see main skill for defaults).
3. Split the text into sections based on markdown headings (`#`, `##`, `###`).
4. If the document has fewer than 2 headings, split by paragraph boundaries instead. Group paragraphs into chunks of roughly 500-2000 words (respecting `max_words_per_section` and `min_words_per_section`). Preserve paragraph boundaries.
5. Merge adjacent tiny sections (under `min_words_per_section`) into the next section until the minimum is met. Never exceed `max_words_per_section`.
6. For each section, write a file in `workspace/SECTIONS/` named `NN_title.md` with YAML frontmatter:
   ```yaml
   ---
   section: 1
   title: "Introduction"
   words: 452
   ---
   ```
   Use 2-digit zero-padded numbers for `NN`.
7. Write `workspace/SECTIONS/sections_index.md` listing each section with title, file link, and word count.
8. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step B line to `[X]` with started and completed timestamps.

## Rules

- Preserve all content. Nothing is dropped.
- Section numbers must be monotonic integers starting at 1.
- The `title` frontmatter field must exist and be non-empty.
