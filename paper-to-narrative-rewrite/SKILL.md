---
name: paper-to-narrative-rewrite
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, rewrite, revision, batch]
---

# Step I: Rewrite

## Purpose

Rewrite flagged draft sections using review feedback as additional constraints. Process sections in batches of 4-5 to minimize file I/O and context switching.

## Inputs

- `workspace/REVIEW/rewrite_requests.md` (if it exists)
- `workspace/REVIEW/review_report.md`
- `workspace/REVIEW/section_notes/*.md`
- `workspace/DRAFTS/*.md`
- `workspace/DRAFTS/drafts_index.md`
- `workspace/PLAN/chapter_plans/*.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/CONCEPT/story_concept.md`
- `workspace/CONFIG/settings.md`

## Outputs

- `workspace/DRAFTS/NN_title_draft_v2.md` (one per rewritten section)
- `workspace/DRAFTS/prompts/NN_title_rewrite_prompt.txt` (one per rewritten section)
- `workspace/DRAFTS/rewrite_log.md`

## Prerequisites

Before starting, validate:
1. `workspace/REVIEW/rewrite_decision.md` exists. If not, abort: "Step I: Rewrite cannot run without a decision from Step H. Run Step H first."
2. Read `workspace/REVIEW/rewrite_decision.md`. If the decision is `proceed` or `halt`, skip this step and report: "Step I: Rewrite: skipped (user decision was '{decision}')."
3. `workspace/DRAFTS/` contains at least one draft file.
4. `workspace/PLAN/chapter_plans/` exists with corresponding plan files.

If the decision is `rewrite` but `workspace/REVIEW/rewrite_requests.md` does not exist or is empty, read `workspace/REVIEW/review_report.md` and extract any sections with major issues. If none, skip this step and report: "No rewrite requests found. Step I: Rewrite: skipped."

## Instructions

**Idempotency guard:** Before doing any work, check `workspace/DRAFTS/rewrite_log.md`. If it exists and lists all sections from `rewrite_requests.md` as `status: completed`, skip all work and report: "Step I: Rewrite: already completed, skipping."

### Phase 1: Load Shared Context (Read Once)

Read these files ONCE and hold their contents in working memory. Do not re-read them for each section or batch:

1. Read `workspace/REVIEW/rewrite_decision.md` to confirm the user chose `rewrite`.
2. Read `workspace/REVIEW/rewrite_requests.md` (if it exists). If it does not exist, read `workspace/REVIEW/review_report.md` and extract any sections with major issues.
3. Read `workspace/REVIEW/section_notes/*.md` for detailed per-section feedback.
4. Read `workspace/ANALYSIS/selected_exemplars.md`.
5. Read `workspace/SOURCES/STYLE_SOURCE.md`.
6. Read `workspace/CONCEPT/story_concept.md`.
7. Load config from `workspace/CONFIG/settings.md`.
8. Build a rewrite queue: a list of sections that need rewriting, with their associated issues.

### Phase 2: Build Batch Queue

1. Sort the rewrite queue by section number.
2. Group into batches of 4-5 sections. The final batch may be smaller.
3. For each section, note its corresponding plan file and section notes file paths.

### Phase 3: Process Batches

For each batch:

1. **Read batch inputs:** Read the draft files, plan files, and section notes for all 4-5 sections in this batch. Hold them in working memory.
2. **Generate rewrites:** For each section in the batch:
   a. Assemble a rewrite prompt using the Step G template **with the following additions**:

      ```
      REWRITE INSTRUCTIONS (from review):
      {specific_issues_for_this_section}

      CONSTRAINTS:
      - Address every issue listed above.
      - Do not introduce new factual claims not present in the source.
      - Preserve the narrative frame and character voice.
      - Match the style exemplars more closely than the previous draft.
      ```

   b. Generate the rewritten narrative chapter by following the prompt. The agent acts as the writer.

3. **Validate each generated chapter** before writing any files. For each chapter in the batch, verify:
   - Word count exceeds `min_words_per_section` from settings.
   - The prose references the narrative frame from `story_concept.md`.
   - The prose preserves at least one key claim from the corresponding plan file.
   - Every issue from the review feedback is visibly addressed in the new draft.
   If any check fails, regenerate that chapter before proceeding. Do not write invalid drafts to disk.

4. **Write batch outputs:** Only after all chapters in the batch pass validation:
   a. Save each assembled prompt to `workspace/DRAFTS/prompts/NN_title_rewrite_prompt.txt` for audit.
   b. Write each rewritten chapter to `workspace/DRAFTS/NN_title_draft_v2.md` with YAML frontmatter:
      ```yaml
      ---
      section: 1
      plan_title: "The First Lesson"
      status: draft-v2
      rewritten_from: "NN_title_draft.md"
      rewrite_reason: "{summary_of_issues}"
      ---
      ```
   c. If generation produces no output for a section, write a placeholder with `status: pending`.
   d. **Verify each written file:** Read back the file and confirm it is non-empty and contains YAML frontmatter. If empty or malformed, retry once. Only move to the next batch after verification passes.

### Phase 4: Update Drafts Index and Log

1. Update `workspace/DRAFTS/drafts_index.md`:
   - List rewritten sections with `status: draft-v2`.
   - Link to both the original and rewritten files.
2. Write `workspace/DRAFTS/rewrite_log.md`:
   - YAML frontmatter with `started` and `completed` timestamps.
   - List of rewritten sections with original file, new file, and issue summary.
   - Any sections that failed to rewrite.

### Phase 5: Update Checklist

Update `workspace/CHECKLISTS/pipeline_checklist.md`:
- Find/replace the Step I line to `[X]` with started and completed timestamps.

## Single Section Mode

If the user asks for a single section rewrite (e.g., "rewrite section 3 only"), treat it as a batch of one. Process only that section. Useful for targeted fixes.

## Resume Behavior

If the user says "resume rewrite", check `rewrite_log.md`. Skip any sections already marked `completed`. Regenerate any marked `pending`. Resume from the next unprocessed batch.

## Notes

- Rewritten drafts preserve the original. The original `NN_title_draft.md` is never overwritten.
- If a section is rewritten multiple times, use `v3`, `v4`, etc.
- All rewrite prompts are saved for inspection, just like first-draft prompts.
- Factual accuracy remains the highest priority. Rewrite instructions from the Fidelity Auditor take precedence over style instructions from the Style Adversary.
- Batching reduces wall time by keeping the agent in writer mode across multiple sections.
