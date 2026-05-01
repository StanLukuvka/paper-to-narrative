---
name: paper-to-narrative-analyze
description: Analyze style source, extract exemplar paragraphs and write style guide.
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

## Outputs

- `workspace/ANALYSIS/style_chapters/chN_analysis.md`
- `workspace/ANALYSIS/exemplars/exemplar_factor_N.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/exemplars_index.md`

## Instructions

1. Read `workspace/SOURCES/STYLE_SOURCE.md`.
2. Split it into chapters or logical blocks (use `##` headings, or group paragraphs if no headings).
3. For each chapter/block, write a brief analysis file to `ANALYSIS/style_chapters/chN_analysis.md` covering:
   - Word and paragraph counts
   - Average sentence length
   - Percentage of short sentences (8 words or fewer)
   - Tone words present
   - Whether dialogue occurs
4. Extract candidate exemplar paragraphs for each of these factors:
   - **tone** — mood-setting atmosphere
   - **rhythm** — sentence length variation
   - **imagery** — sensory and descriptive language
   - **dialogue** — naturalistic speech patterns
   - **description** — world-building and detail density
   - **pacing** — action verbs and short paragraphs
   - **character_voice** — first-person or distinctive speech
   - **humor** — light or comedic markers
5. For each candidate, score it with a simple heuristic (0.0 to 1.0) and keep the top 10 per factor.
6. Write each exemplar to `ANALYSIS/exemplars/exemplar_factor_N.md` with YAML frontmatter:
   ```yaml
   ---
   factor: tone
   chapter: 5
   score: 0.85
   rank: candidate
   ---
   <paragraph text>
   ```
7. Write `ANALYSIS/style_profile.md` summarizing the voice characteristics.
8. Write `ANALYSIS/exemplars_index.md` listing all factors and candidate counts.
9. Mark step C complete in `workspace/CHECKLISTS/pipeline_checklist.md`.

## Notes

- If the style source is very long, you may use a sub-agent via `delegate_task` to process chapters in parallel.
- Keep exemplars concise: one paragraph each.
