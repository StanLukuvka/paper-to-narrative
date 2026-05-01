---
name: paper-to-narrative
description: >
  Transform dry research papers into engaging narrative prose using a prose style reference.
  Pure markdown skill-set. Hermes executes every step by reading and writing workspace files.
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

Turn a research paper into readable narrative prose. Feed it a style reference (any novel or prose text) and it rewrites the research in that voice. Hermes executes every step by reading and writing markdown files in a workspace directory.

## How It Works

Six-step pipeline. Each step reads files from the workspace, performs work, and writes new files back. The workspace is the single source of truth. No hidden state.

```
INFO_SOURCE  --A--> SOURCES/INFO_SOURCE.md
STYLE_SOURCE --B--> SECTIONS/*.md
                          |
                   C --v   ANALYSIS/ (style analysis + exemplars)
                          |
                   D --v   selected_exemplars.md
                          |
                   E --v   PLAN/ (rewrite blueprints)
                          |
                   F --v   DRAFTS/ + final output
```

## Quick Start

1. Place files in a working directory:
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
│   └── metadata.md          # run metadata (YAML frontmatter)
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
    └── settings.md
```

## Running Steps

Load each sub-skill and follow its instructions. The agent tracks progress in `workspace/CHECKLISTS/pipeline_checklist.md`.

To run the full pipeline, load each sub-skill in order and execute its steps. To resume, check the checklist and start from the first unchecked step.

## Configuration

Set in `workspace/CONFIG/settings.md` (YAML frontmatter) or tell the agent directly:

- `style_anchor_count`: how many style-source chapters to use as tonal anchors (default: 3)
- `max_words_per_section`: upper bound when splitting source (default: 2000)
- `min_words_per_section`: lower bound when merging tiny sections (default: 200)
- `exemplars_per_factor`: candidates to extract per style factor (default: 10)
- `exemplars_to_keep`: minimum exemplars after selection (default: 5)

## System Dependencies

The convert step needs these installed on the host:
- `pandoc` (for DOCX, HTML, LaTeX conversion)
- `pdftotext` from poppler-utils (for PDF extraction)

If either is missing, the agent stops and reports it.

## Notes

- Every file is human-readable markdown.
- The workspace is the audit trail.
- Steps can be rerun independently by reloading their skill.
- The agent does not choose the LLM. The user runs Hermes with whatever model they prefer.
