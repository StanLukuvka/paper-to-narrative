---
name: paper-to-narrative-select
description: "Interactive or automatic selection of best exemplar paragraphs."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, select, interactive]
---

# Step D: Select

## Purpose

Present candidate exemplars to the user (interactive mode) or automatically select the best ones (auto mode). Always retains at least `exemplars_to_keep` exemplars.

## Inputs

- `workspace/ANALYSIS/exemplars/` (all candidate files)
- `workspace/ANALYSIS/exemplars_index.md`
- `workspace/CONFIG/settings.md` (for selection_mode, exemplars_to_keep)

## Outputs

- `workspace/ANALYSIS/selected_exemplars.md`

## Prerequisites

Before starting, validate:
1. `workspace/ANALYSIS/exemplars/` exists and contains at least one `.md` file.
2. `workspace/ANALYSIS/exemplars_index.md` exists.

If no exemplar files exist, abort: "No exemplars found — run Step C first."

## Instructions

1. Load config from `workspace/CONFIG/settings.md` (use defaults if missing).
2. Load all exemplar files from `workspace/ANALYSIS/exemplars/`.
3. Group them by factor.
4. Deduplicate within each factor: compute SHA256 of `paragraph.strip().lower()`. Keep only the first occurrence of each hash per factor.

### Interactive Mode (selection_mode == interactive)

5. For each factor, present the top 3-5 candidates to the user using the `clarify` tool:
   - Show a short preview (first 200 characters) of each candidate.
   - Show its chapter source and score.
   - Ask the user which ones best capture this factor.
6. Accumulate the user's selections.
7. If fewer than `exemplars_to_keep` exemplars are chosen, automatically add the highest-scoring remaining candidates until the minimum is met.

### Auto Mode (selection_mode == auto)

5. Across all factors, collect all deduplicated candidates.
6. Sort by `score` descending. Break ties by `chapter` ascending.
7. Select the top `exemplars_to_keep` candidates from this sorted list.
8. No user interaction required.

### Output

8. Write `workspace/ANALYSIS/selected_exemplars.md` with two sections:

   **Human-readable section:**
   ```markdown
   # Selected Exemplars

   ## Tone
   - "The castle loomed..." *(Chapter 3, score: 0.85)*
   ```

   **Machine-readable YAML array (after a `---` separator):**
   ```yaml
   ---
   selected:
     - factor: tone
       chapter: 3
       score: 0.85
       text: "The castle loomed..."
       hash: "a3f2b1..."
     - factor: rhythm
       chapter: 1
       score: 0.92
       text: "Wind howled..."
       hash: "c7d8e9..."
   ```

9. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step D line to `[X]` with started and completed timestamps.

## Notes

- The user can always edit `selected_exemplars.md` manually afterward.
- If no exemplars exist, the prerequisite check aborts before any work begins.
