---
name: paper-to-narrative
description: "Transform dry nonfiction sources into engaging narrative prose using a prose style reference. Pure markdown skill-set. Hermes executes every step by reading and writing workspace files."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [writing, research, transformation, pipeline, markdown-only]
    dependencies:
      - paper-to-narrative-convert
      - paper-to-narrative-section
      - paper-to-narrative-analyze
      - paper-to-narrative-select
      - paper-to-narrative-concept
      - paper-to-narrative-plan
      - paper-to-narrative-write
      - paper-to-narrative-review
      - paper-to-narrative-rewrite
---

# Paper-to-Narrative

Turn a nonfiction source into readable narrative prose. Feed it a style reference (any novel or prose text) and it rewrites the source in that voice. Hermes executes every step by reading and writing markdown files in a workspace directory.

## Quick Start

1. Pick a directory and place your files inside it:
   - `source.pdf` (or .md, .txt, .docx) — the nonfiction source
   - `style.md` — the prose style reference

2. Tell the agent:
   ```
   Run paper-to-narrative on info.pdf with style.md, output to book.md
   ```

3. The agent runs the full pipeline from your current directory, stopping at Human Gates D and E for your input.

## Pipeline Steps

| Step | Skill | What It Does |
|------|-------|--------------|
| A | convert | Convert source document to markdown |
| B | section | Split paper into topical sections |
| C | analyze | Analyze style source and extract exemplars |
| D | select | Pick the best exemplars (interactive or auto) |
| E | concept | Choose the narrative frame interactively |
| F | plan | Create rewrite plan per section |
| G | write | Write narrative chapters |
| H | review | Quality gate: continuity, style, accuracy |
| I | rewrite | Rewrite flagged sections using review feedback |
| J | *(inline)* | Export finalized book to clean PDF (no pipeline metadata) |

## Workspace Structure

```
workspace/
├── SOURCES/
│   ├── SOURCE.md       # converted nonfiction source
│   ├── STYLE_SOURCE.md      # style reference copy
│   ├── STYLE_ANCHORS.md     # extracted anchor chapters (trimmed)
│   └── metadata.md          # run metadata (YAML frontmatter)
├── SECTIONS/
│   ├── 01_introduction.md   # split source sections
│   └── sections_index.md
├── ANALYSIS/
│   ├── style_chapters/      # per-chapter breakdowns
│   ├── exemplars/           # candidate paragraphs
│   ├── selected_exemplars.md
│   └── style_profile.md
├── CONCEPT/
│   └── story_concept.md     # chosen narrative frame
├── PLAN/
│   ├── rewrite_strategy.md
│   ├── chapter_plans/       # per-section blueprints
│   └── chapter_plan_index.md
├── DRAFTS/
│   ├── prompts/             # saved prompts for every chapter
│   ├── 01_introduction_draft.md
│   ├── rewrite_log.md       # tracks what was rewritten and why
│   └── drafts_index.md
├── REVIEW/
│   ├── review_report.md
│   ├── rewrite_decision.md  # user choice after review
│   ├── section_notes/
│   └── rewrite_requests.md
├── CHECKLISTS/
│   └── pipeline_checklist.md
└── CONFIG/
    └── settings.md
```

## Checklist Format

`workspace/CHECKLISTS/pipeline_checklist.md` tracks pipeline state. Format is a markdown task list with started/completed timestamps.

Each step REWRITES its own line in-place (find the line by step name, replace the whole line). If the line is missing, append it to the end.

### Format

Markdown task list. Each step is one line with started/completed timestamps.

```markdown
- [X] Step A: Convert
  - started: 2026-05-01T14:30:00Z
  - completed: 2026-05-01T14:31:00Z
- [ ] Step J: Export
```

Rules:
- Set `[ ]` + `started` when a step begins. Change to `[X]` + `completed` when it finishes.
- If a step fails, leave `[ ]` and append `failed: <timestamp>` plus a note.
- Timestamps are ISO-8601 UTC.

## Configuration

`workspace/CONFIG/settings.md` stores pipeline settings as YAML frontmatter.

### Schema

```yaml
---
style_anchor_count: 3               # how many style chapters to use as anchors
style_anchor_selection: first_middle_last  # first_middle_last | random | specific
max_words_per_section: 2000
min_words_per_section: 200
exemplars_per_factor: 10
exemplars_to_keep: 5
truncate_at: paragraph              # paragraph | sentence | char
---
```

### Defaults

If `settings.md` is missing, use these defaults:

| Key | Default |
|-----|---------|
| style_anchor_count | 3 |
| style_anchor_selection | first_middle_last |
| max_words_per_section | 2000 |
| min_words_per_section | 200 |
| exemplars_per_factor | 10 |
| exemplars_to_keep | 5 |
| truncate_at | paragraph |

### Loading Pattern

Read `workspace/CONFIG/settings.md`, parse YAML frontmatter, merge with defaults. If the file does not exist, use defaults only.

## Running Steps

Steps fall into two categories: **Automated** and **Human Gate**.

| Type | Steps | Behavior |
|------|-------|----------|
| Automated | A, B, C, F, G, H, I, J | Agent runs these without stopping. Checklist tracks progress. Step J is inline in this file. |
| Human Gate | D, E | Agent MUST stop and use the `clarify` tool. Do not proceed to the next step until the user has responded. |

### Execution Rules

1. Load the sub-skill for the current step.
2. Follow its instructions exactly.
3. Update `workspace/CHECKLISTS/pipeline_checklist.md` when the step begins and when it completes.
4. **For Human Gates (D, E):**
   - Present options to the user with the `clarify` tool.
   - After calling `clarify`, STOP. Do not load the next skill.
   - Wait for the user's response in the next message.
   - Only after receiving the user's choice, write the output file and mark the step complete.
   - Then proceed to the next step.
5. **To resume:** Read the checklist. If Step D or E is marked `[ ]` with a `started` timestamp but no `completed` timestamp, re-run that gate step from the beginning (idempotency guards prevent duplicate work).

### Never Skip a Human Gate

Steps D and E exist because the agent cannot judge style preference or story concept on behalf of the user. If you are tempted to auto-select or auto-generate a concept to save time, do not. Always stop and ask.

## System Dependencies

The convert step needs these installed on the host:
- `pandoc` (for DOCX, HTML, LaTeX conversion)
- `pdftotext` from poppler-utils (for PDF extraction)

If either is missing, the agent stops and reports it.


## Step J: Export (Inline)

When Step I is complete, produce a final PDF.

1. Read `workspace/DRAFTS/drafts_index.md` for draft order.
2. Read `workspace/CONCEPT/story_concept.md` and extract `concept_title` for the book title.
3. Concatenate all drafts in order into a single temporary markdown string.
4. Strip all non-narrative content:
   - Remove YAML frontmatter blocks
   - Remove any line containing `workspace/`, `DRAFTS/`, `PLAN/`, `ANALYSIS/`, `REVIEW/`, `CONFIG/`, `CHECKLISTS/`
   - Remove internal pipeline headers (`# Draft`, `# Chapter Plan`, etc.)
5. Add a title page: `# <concept_title>`
6. Write cleaned markdown to a temp file, then run:
   ```bash
   pandoc /tmp/book_clean.md -o workspace/BOOK.pdf --pdf-engine=xelatex -V geometry:margin=1in
   ```
   Fall back to default PDF engine if xelatex is missing.
7. Delete the temp file.
8. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Mark Step J `[X]` with timestamps.

Prerequisites: `pandoc` on PATH. If missing, abort and tell the user to install it.

## Notes

- Every file is human-readable markdown.
- The workspace is the audit trail.
- Steps can be rerun independently by reloading their skill.
- The story concept step (E) is the creative anchor. Everything after it follows the chosen frame.
