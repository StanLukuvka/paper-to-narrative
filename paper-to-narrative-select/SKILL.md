---
name: paper-to-narrative-select
description: "Interactive UI to pick best exemplar paragraphs; always keep top 5."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, select, interactive]
---

# Step D: Select

## Purpose

Present candidate exemplars to the user and let them pick the best ones. Always retains at least 5 exemplars.

## Inputs

- `workspace/ANALYSIS/exemplars/` (all candidate files)
- `workspace/ANALYSIS/exemplars_index.md`

## Outputs

- `workspace/ANALYSIS/selected_exemplars.md`

## Instructions

1. Load all exemplar files from `workspace/ANALYSIS/exemplars/`.
2. Group them by factor.
3. For each factor, present the top 3-5 candidates to the user using the `clarify` tool:
   - Show a short preview (first 200 characters) of each candidate.
   - Show its chapter source and score.
   - Ask the user which ones best capture this factor.
4. Accumulate the user's selections.
5. If fewer than 5 exemplars are chosen, automatically add the highest-scoring remaining candidates until the minimum is met. Use text-hash deduplication to avoid duplicates.
6. Write `workspace/ANALYSIS/selected_exemplars.md` organized by factor:
   ```markdown
   # Selected Exemplars

   ## Tone
   - "The castle loomed..." *(Chapter 3)*
   ```
7. Mark step D complete in `workspace/CHECKLISTS/pipeline_checklist.md`.

## Auto Mode

If the user prefers no interaction, select the top 5 exemplars across all factors automatically.

## Notes

- The user can always edit `selected_exemplars.md` manually afterward.
- If no exemplars exist, stop and tell the user to run step C first.
