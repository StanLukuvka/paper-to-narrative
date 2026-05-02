---
name: paper-to-narrative
description: "Transform dry research papers into engaging narrative prose using a prose style reference. Pure markdown skill-set. Hermes executes every step by reading and writing workspace files."
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

1. Pick a directory and place your files inside it:
   - `info.pdf` — the research paper
   - `style.md` — the prose style reference

2. Tell the agent:
   ```
   Run paper-to-narrative on info.pdf with style.md, output to book.md
   ```

3. The agent runs steps A-F automatically from your current directory.

## Pipeline Steps

| Step | Skill | What It Does |
|------|-------|--------------|
| A | convert | Convert source document to markdown |
| B | section | Split paper into topical sections |
| C | analyze | Analyze style source and extract exemplars |
| D | select | Pick the best exemplars (interactive or auto) |
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

## Checklist Format

`workspace/CHECKLISTS/pipeline_checklist.md` tracks pipeline state. Format is a markdown task list with started/completed timestamps.

Each step REWRITES its own line in-place (find the line by step name, replace the whole line). If the line is missing, append it to the end.

### Example

```markdown
# Pipeline Checklist

- [X] Step A: Convert
  - started: 2026-05-01T14:30:00Z
  - completed: 2026-05-01T14:31:00Z
- [X] Step B: Section
  - started: 2026-05-01T14:31:00Z
  - completed: 2026-05-01T14:32:00Z
- [ ] Step C: Analyze
  - started: 2026-05-01T14:33:00Z
- [ ] Step D: Select
- [ ] Step E: Plan
- [ ] Step F: Write
```

Rules:
- Set `[ ]` and append `- started: <timestamp>` when a step begins.
- Change `[ ]` to `[X]` and append `- completed: <timestamp>` when a step finishes successfully.
- If a step fails, leave `[ ]` and append `- failed: <timestamp>` plus a note on the next line.
- Timestamps are ISO-8601 UTC (e.g., `2026-05-01T14:30:00Z`).

## Configuration

`workspace/CONFIG/settings.md` stores pipeline settings as YAML frontmatter.

### Schema

```yaml
---
style_anchor_count: 3               # how many style chapters to use as anchors
style_anchor_selection: first_middle_last  # first_middle_last | random | specific
selection_mode: auto                # auto | interactive — Step D behavior
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
| selection_mode | auto |
| max_words_per_section | 2000 |
| min_words_per_section | 200 |
| exemplars_per_factor | 10 |
| exemplars_to_keep | 5 |
| truncate_at | paragraph |

### Loading Pattern

Each step should load config by reading `workspace/CONFIG/settings.md`, parsing the YAML frontmatter, and merging with the defaults dictionary. If the file does not exist, use defaults only.

## Running Steps

Load each sub-skill and follow its instructions. The agent tracks progress in `workspace/CHECKLISTS/pipeline_checklist.md`.

To run the full pipeline, load each sub-skill in order and execute its steps. To resume, check the checklist and start from the first unchecked step.

## System Dependencies

The convert step needs these installed on the host:
- `pandoc` (for DOCX, HTML, LaTeX conversion)
- `pdftotext` from poppler-utils (for PDF extraction)

If either is missing, the agent stops and reports it.

## Notes

- Every file is human-readable markdown.
- The workspace is the audit trail.
- Steps can be rerun independently by reloading their skill.
