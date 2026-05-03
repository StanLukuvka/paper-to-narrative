---
name: paper-to-narrative-write
description: "Write final chapters using plan, exemplars, and style-source chapters for tonal context. Batched execution: 4-5 sections per turn."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, write, generation, batch]
---

# Step G: Write

## Purpose

Transform each source section into a first draft of narrative prose. Process sections in batches of 4-5 to minimize file I/O and context switching.

## Inputs

- `workspace/PLAN/chapter_plans/*.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/SECTIONS/*.md`
- `workspace/CONFIG/settings.md` (for style_anchor_count, style_anchor_selection, truncate_at, fidelity, integration)

## Outputs

- `workspace/DRAFTS/NN_title_draft.md` (one per section)
- `workspace/DRAFTS/prompts/NN_title_prompt.txt`
- `workspace/DRAFTS/drafts_index.md`
- Final merged file (e.g., `workspace/book.md`)

## Prerequisites

Before starting, validate:
1. `workspace/PLAN/chapter_plans/` exists and contains at least one `.md` file.
2. Every plan file has required YAML frontmatter: `section`, `plan_title`, `frame`.
3. `workspace/ANALYSIS/selected_exemplars.md` exists and contains `selected_by: user` in its YAML frontmatter. If `selected_by` is missing or not `user`, abort: "Step D (Select) must be completed with user input. Run Step D and use the clarify tool."
4. `workspace/SOURCES/STYLE_SOURCE.md` exists.
5. `workspace/CONCEPT/story_concept.md` exists and contains `selected_by: user` in its YAML frontmatter. If `selected_by` is missing or not `user`, abort: "Step E (Concept) must be completed with user input. Run Step E and use the clarify tool."
6. `workspace/SECTIONS/` exists with corresponding section files.

If any prerequisite fails, abort with a message naming the missing file or field.

## Instructions

**Idempotency guard:** Before doing any work, check if this step's outputs already exist and the checklist marks it complete. If both are true, skip all work and report: "Step G: Write: already completed, skipping."

### Phase 1: Load Shared Context (Read Once)

Read these files ONCE and hold their contents in working memory. Do not re-read them for each section or batch:

1. Read `workspace/SOURCES/STYLE_SOURCE.md`.
2. Read `workspace/ANALYSIS/selected_exemplars.md`.
3. Read `workspace/CONCEPT/story_concept.md`.
4. Load config from `workspace/CONFIG/settings.md`.
5. Select `style_anchor_count` chapters or blocks from the style source using `style_anchor_selection`:
   - `first_middle_last` — first chapter, middle chapter, last chapter.
   - `random` — pick randomly (not seeded; varies per run).
   - `specific` — use chapters listed in config (not yet implemented; fallback to first_middle_last).
   These serve as tonal anchors and are held in memory for all batches.

### Phase 2: Build Batch Queue

1. List all plan files in `workspace/PLAN/chapter_plans/`.
2. Sort by section number.
3. Group into batches of 4-5 sections. The final batch may be smaller.
4. For each plan file, note its corresponding section file path in `workspace/SECTIONS/` (match by section number).

### Phase 3: Process Batches

For each batch:

1. **Read batch inputs:** Read the plan files and section files for all 4-5 sections in this batch. Hold them in working memory.
2. **Generate chapters:** For each section in the batch:
   a. Strip any YAML frontmatter from the section text before including it in the prompt.
   b. Truncate the section text to respect LLM context limits. Use `truncate_at` setting:
      - `paragraph` — truncate at paragraph boundary, keep full paragraphs up to limit.
      - `sentence` — truncate at sentence boundary.
      - `char` — hard truncate at character count.
   c. Assemble a prompt using this exact template:

      ```
      You are a narrative rewriter. Your task is to transform a source section into engaging prose.

      NARRATIVE FRAME:
      {frame}

      SOURCE SECTION (to preserve and rewrite):
      {section_text}

      KEY CLAIMS TO PRESERVE (verbatim facts):
      {original_concepts}

      STYLE EXEMPLARS (match this voice):
      {exemplars_text}

      TONAL ANCHORS (style-source chapters for voice reference):
      {style_anchors}

      RULES:
      1. Preserve every factual claim from KEY CLAIMS TO PRESERVE.
      2. Never add information not present in the source section.
      3. Match sentence rhythm, imagery, and tone to the STYLE EXEMPLARS.
      4. Use the NARRATIVE FRAME as the scene structure.
      5. Write in the voice of the TONAL ANCHORS.
      6. Target approximately {words_target} words.
      7. FIDELITY LEVEL: {fidelity}
         - low: approximate numbers, sketch methods, prioritize flow over precision
         - medium: round key figures, mention methods without full detail
         - high: keep exact numbers intact, methods stay precise and traceable
      8. INTEGRATION LEVEL: {integration}
         - low: story decorates the explanation; characters observe
         - medium: story and science interleave; characters discuss findings
         - high: the science IS the plot; characters discover in real time; reader learns through action

      Write the narrative chapter now.
      ```

   d. Generate the narrative chapter by following the prompt. The agent acts as the writer.

3. **Validate each generated chapter** before writing any files. For each chapter in the batch, verify:
   - Word count exceeds `min_words_per_section` from settings.
   - The prose references the narrative frame from `story_concept.md` (e.g., mentions the setting, perspective, or framing device).
   - The prose preserves at least one key claim from the corresponding plan file.
   If any check fails, regenerate that chapter before proceeding. Do not write invalid drafts to disk.

4. **Write batch outputs:** Only after all chapters in the batch pass validation:
   a. Save each assembled prompt to `DRAFTS/prompts/NN_title_prompt.txt` for audit.
   b. Write each generated chapter to `DRAFTS/NN_title_draft.md` with YAML frontmatter:
      ```yaml
      ---
      section: 1
      plan_title: "The First Lesson"
      status: draft
      ---
      ```
   c. If generation produces no output for a section, write a placeholder with `status: pending` so the user knows to retry.
   d. **Verify each written file:** Read back the file and confirm it is non-empty and contains YAML frontmatter. If empty or malformed, retry once. Only move to the next batch after verification passes.

### Phase 4: Merge and Index

1. When all batches are processed, merge every draft file into the final output file (`workspace/book.md`).
2. Write `DRAFTS/drafts_index.md` listing all drafts and their status.

### Phase 5: Update Checklist

Update `workspace/CHECKLISTS/pipeline_checklist.md`:
- Find/replace the Step G line to `[X]` with started and completed timestamps.

## Resume Behavior

If the user says "resume", check `DRAFTS/` for existing draft files. Skip any with `status: draft`. Regenerate any with `status: pending`. Resume from the next unprocessed batch.

## Single Section Mode

If the user asks for a single section (e.g., "write section 3 only"), treat it as a batch of one. Generate only that section's draft. Useful for prompt iteration.

## Notes

- All prompts are saved for inspection. The user can copy a prompt, run it through any LLM, and paste the response back into the draft file.
- Factual accuracy is the highest priority. The narrative frame serves the facts.
- Truncation prevents exceeding LLM context limits while preserving enough material for the writer to work with.
- Batching reduces wall time by keeping the agent in writer mode across multiple sections.
