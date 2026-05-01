---
name: paper-to-narrative
description: Transform dry research papers into engaging narrative prose using a style reference. Pure markdown skill-set. Hermes agent executes every step.
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
      - paper-to-narrative-plan
      - paper-to-narrative-write
---

# Paper-to-Narrative

Academic paper to readable narrative. Uses a style reference (any novel or prose text) to rewrite research in an engaging voice. The Hermes agent executes every step by reading and writing markdown files in a workspace directory.

## How It Works

The agent follows a 6-step pipeline. Each step reads files from the workspace, performs work, and writes new files back. The workspace is the single source of truth. There is no hidden state.

```
INFO_SOURCE  ──A──► SOURCES/INFO_SOURCE.md
STYLE_SOURCE ──B──► SECTIONS/*.md
                          ↓
                   C ──► ANALYSIS/ (style analysis + exemplars)
                          ↓
                   D ──► selected_exemplars.md
                          ↓
                   E ──► PLAN/ (rewrite blueprints)
                          ↓
                   F ──► DRAFTS/ + final output
```

## Quick Start

1. Place your files in a working directory:
   - `info.pdf` — the research paper
   - `style.md` — the prose style reference

2. Tell the agent:
   ```
   Run paper-to-narrative on info.pdf with style.md, output to book.md
   ```

3. The agent creates `workspace/` and runs steps A-F automatically.

## Pipeline Steps

| Step | Skill | What It Does |
|------|-------|--------------|
| A | convert | Convert source document to markdown |
| B | section | Split paper into topical sections |
| C | analyze | Analyze style source and extract exemplars |
| D | select | Pick the best exemplars (interactive) |
| E | plan | Create rewrite plan per section |
| F | write | Write narrative chapters |

## Workspace Structure

```
workspace/
├── SOURCES/
│   ├── INFO_SOURCE.md       # converted research paper
│   ├── STYLE_SOURCE.md      # style reference copy
│   └── metadata.json        # run metadata
├── SECTIONS/
│   ├── 01_introduction.md   # split paper sections
│   └── sections_index.md
├── ANALYSIS/
│   ├── style_chapters/      # per-chapter breakdowns
│   ├── exemplars/           # candidate paragraphs
│   ├── selected_exemplars.md
│   └── style_profile.md
├── PLAN/
│   ├── rewrite_strategy.md
│   ├── chapter_plans/       # per-section blueprints
│   └── chapter_plan_index.md
├── DRAFTS/
│   ├── prompts/             # saved prompts for audit
│   ├── 01_introduction_draft.md
│   └── drafts_index.md
├── CHECKLISTS/
│   └── pipeline_checklist.md
└── CONFIG/
    └── settings.yaml
```

## Running Steps

Load each sub-skill and follow its instructions. The agent tracks progress in `workspace/CHECKLISTS/pipeline_checklist.md`.

To run the full pipeline, load each sub-skill in order and execute its steps.

## Configuration

Set in `workspace/CONFIG/settings.yaml` or environment variables:
- `model`: which LLM to use for generation steps
- `random_seed`: seed for style chapter sampling (default: 42)
- `max_words_per_section`: 2000
- `min_words_per_section`: 200

## Dependencies

- pandoc (for non-markdown source conversion)
- pdftotext (poppler-utils, for PDF extraction)

## Notes

- Every file is human-readable markdown.
- The workspace is the audit trail.
- Steps can be rerun independently by reloading their skill.
