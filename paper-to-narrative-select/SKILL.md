---
name: paper-to-narrative-select
description: "Interactive selection of best exemplar paragraphs. Human gate — always stops for user input."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, select, interactive]
---

# Step D: Select

## Purpose

Present candidate exemplars to the user and collect their selections. Always retains at least `exemplars_to_keep` exemplars. This is a Human Gate: the agent must use the `clarify` tool and wait for user response.

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

**Idempotency guard:** Before doing any work, check if this step's outputs already exist and the checklist marks it complete. If both are true, skip all work and report: "Step D: Select: already completed, skipping."

1. Load config from `workspace/CONFIG/settings.md` (use defaults if missing).
2. Load all exemplar files from `workspace/ANALYSIS/exemplars/`.
3. Group them by factor.
4. Deduplicate within each factor: compute SHA256 of `paragraph.strip().lower()`. Keep only the first occurrence of each hash per factor.

### Human Selection (REQUIRED)

5. For each factor, present the top 3-5 candidates to the user using the `clarify` tool:
   - Show a short preview (first 200 characters) of each candidate.
   - Show its chapter source and score.
   - Ask the user which ones best capture this factor.
6. After calling `clarify`, STOP. Wait for the user's response in the next message. Do not proceed to any other work.
7. When the user responds, accumulate their selections.
8. If fewer than `exemplars_to_keep` exemplars are chosen across all factors, automatically add the highest-scoring remaining candidates until the minimum is met.
9. If the user rejects all candidates for a factor, keep the single highest-scoring candidate for that factor to ensure the pool is not empty.

### Output

10. Write `workspace/ANALYSIS/selected_exemplars.md` with two sections:

    **Human-readable section:**
    ```markdown
    # Selected Exemplars

    ## Tone
    - "The castle loomed..." *(Chapter 3, score: 0.85)*
    ```

    **Machine-readable YAML array (after a `---` separator):**
    ```yaml
    ---
    selected_by: user
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

11. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
    - Find/replace the Step D line to `[X]` with started and completed timestamps.

## Notes

- The user can always edit `selected_exemplars.md` manually afterward.
- If no exemplars exist, the prerequisite check aborts before any work begins.
- This step is a Human Gate. The agent MUST use `clarify` and wait for a response. Never auto-select.
