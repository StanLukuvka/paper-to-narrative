---
name: paper-to-narrative-write
description: Write final chapters using plan, exemplars, and style-source chapters for tonal context.
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, write, generation]
---

# Step F: Write

## Purpose

Transform each research section into a first draft of narrative prose.

## Inputs

- `workspace/PLAN/chapter_plans/*.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/SECTIONS/*.md`

## Outputs

- `workspace/DRAFTS/NN_title_draft.md` (one per section)
- `workspace/DRAFTS/prompts/NN_title_prompt.txt`
- `workspace/DRAFTS/drafts_index.md`
- Final merged file (e.g., `workspace/book.md`)

## Instructions

1. Read the style source from `workspace/SOURCES/STYLE_SOURCE.md`.
2. Pick 3 chapters or blocks from the style source to use as tonal anchors. If the user wants reproducibility, pick the same ones every time (e.g., first, middle, last, or whatever the CONFIG specifies). These serve as tonal anchors.
3. Read `workspace/ANALYSIS/selected_exemplars.md`.
4. For each plan file in `PLAN/chapter_plans/`:
   a. Match it to the corresponding section file in `SECTIONS/` by section number.
   b. Strip any YAML frontmatter from the section text before including it in the prompt.
   c. Assemble a prompt containing:
      - The narrative frame from the plan
      - The research section text (truncated to ~2000 words if very long)
      - The rewrite plan body
      - The selected exemplars (truncated to ~1200 words)
      - The 3 style-source chapters (truncated to ~3000 words combined)
   d. Save the full prompt to `DRAFTS/prompts/NN_title_prompt.txt`.
   e. Send the prompt to the LLM and capture the response.
   f. Write the response to `DRAFTS/NN_title_draft.md` with YAML frontmatter:
      ```yaml
      ---
      section: 1
      plan_title: "The First Lesson"
      status: draft
      ---
      ```
   g. If the LLM call fails, write a placeholder with `status: pending` so the user knows to retry.
5. When all sections are processed, merge every draft file into the final output file.
6. Write `DRAFTS/drafts_index.md` listing all drafts and their status.
7. Mark step F complete in `workspace/CHECKLISTS/pipeline_checklist.md`.

## Resume Behavior

If the user says "resume", check `DRAFTS/` for existing draft files. Skip any with `status: draft`. Regenerate any with `status: pending`.

## Single Section Mode

If the user asks for a single section (e.g., "write section 3 only"), generate only that section's draft. Useful for prompt iteration.

## Notes

- All prompts are saved for inspection. The user can copy a prompt, run it through any LLM, and paste the response back into the draft file.
- Scientific accuracy is the highest priority. The narrative frame serves the facts.
- Truncation prevents exceeding LLM context limits while preserving enough material for the writer to work with.
