---
name: paper-to-narrative-analyze
description: "Analyze style source, extract exemplar paragraphs and write style guide."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, analyze, style, subagent]
---

# Step C: Analyze

## Purpose

Read the style source and extract candidate exemplar paragraphs representing different stylistic factors.

## Inputs

- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/CONFIG/settings.md` (for exemplars_per_factor)

## Outputs

- `workspace/ANALYSIS/style_chapters/chN_analysis.md`
- `workspace/ANALYSIS/exemplars/exemplar_factor_N.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/exemplars_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SOURCES/STYLE_SOURCE.md` exists and is non-empty.
2. Style source exceeds 1000 words. If shorter, warn: "Style source is very short (<1000 words). Results may be poor." but continue.
3. `workspace/ANALYSIS/` directories exist or can be created.

If STYLE_SOURCE.md is missing, abort: "Step C: STYLE_SOURCE.md not found. Run Step A first."

## Instructions

1. Read `workspace/SOURCES/STYLE_SOURCE.md`.
2. Load config from `workspace/CONFIG/settings.md` (use defaults if missing).
3. Split it into chapters or logical blocks (use `##` headings, or group paragraphs if no headings).
4. For each chapter/block, write a brief analysis file to `ANALYSIS/style_chapters/chN_analysis.md` covering:
   - Word and paragraph counts
   - Average sentence length
   - Percentage of short sentences (8 words or fewer)
   - Tone words present
   - Whether dialogue occurs
5. Extract candidate exemplar paragraphs for each of these factors:
   - **tone** — mood-setting atmosphere
   - **rhythm** — sentence length variation
   - **imagery** — sensory and descriptive language
   - **dialogue** — naturalistic speech patterns
   - **description** — world-building and detail density
   - **pacing** — action verbs and short paragraphs
   - **character_voice** — first-person or distinctive speech
   - **humor** — light or comedic markers
6. For each candidate, score it with a simple heuristic (0.0 to 1.0) and keep the top `exemplars_per_factor` per factor (default 10).
7. Write each exemplar to `ANALYSIS/exemplars/exemplar_factor_N.md` with YAML frontmatter:
   ```yaml
   ---
   factor: tone
   chapter: 5
   score: 0.85
   rank: candidate
   ---
   <paragraph text>
   ```
8. Write `ANALYSIS/style_profile.md` summarizing the voice characteristics.
9. Write `ANALYSIS/exemplars_index.md` listing all factors and candidate counts.
10. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
    - Find/replace the Step C line to `[X]` with started and completed timestamps.

## Notes

- If the style source is very long, you may use a sub-agent via `delegate_task` to process chapters in parallel.
- Keep exemplars concise: one paragraph each.
