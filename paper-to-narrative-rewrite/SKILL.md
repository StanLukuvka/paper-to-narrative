---
name: paper-to-narrative-rewrite
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, rewrite, revision]
---

# Step I: Rewrite

## Purpose

Rewrite flagged draft sections using review feedback as additional constraints. This step consumes the review reports and produces improved drafts.

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
1. `workspace/REVIEW/review_report.md` exists.
2. `workspace/DRAFTS/` contains at least one draft file.
3. `workspace/PLAN/chapter_plans/` exists with corresponding plan files.

If `rewrite_requests.md` does not exist or is empty, but `review_report.md` exists, check the review report for any major issues. If none, skip this step and report: "No rewrite requests found. Step I: Rewrite: skipped."

## Instructions

**Idempotency guard:** Before doing any work, check `workspace/DRAFTS/rewrite_log.md`. If it exists and lists all sections from `rewrite_requests.md` as `status: completed`, skip all work and report: "Step I: Rewrite: already completed, skipping."

### Phase 1: Read Review Reports

1. Read `workspace/REVIEW/rewrite_requests.md` (if it exists). If it does not exist, read `workspace/REVIEW/review_report.md` and extract any sections with major issues.
2. Read `workspace/REVIEW/section_notes/*.md` for detailed per-section feedback.
3. Build a rewrite queue: a list of sections that need rewriting, with their associated issues.

### Phase 2: Rewrite Flagged Sections

For each section in the rewrite queue:

1. Read the current draft file (`workspace/DRAFTS/NN_title_draft.md`).
2. Read the corresponding plan file (`workspace/PLAN/chapter_plans/NN_title_plan.md`).
3. Read the section notes (`workspace/REVIEW/section_notes/NN_title_notes.md`).
4. Read `workspace/ANALYSIS/selected_exemplars.md` and `workspace/SOURCES/STYLE_SOURCE.md` for style context.
5. Read `workspace/CONCEPT/story_concept.md` for narrative frame.
6. Load config from `workspace/CONFIG/settings.md`.

7. Assemble a rewrite prompt using the Step G template **with the following additions**:

   ```
   REWRITE INSTRUCTIONS (from review):
   {specific_issues_for_this_section}
   
   CONSTRAINTS:
   - Address every issue listed above.
   - Do not introduce new factual claims not present in the source.
   - Preserve the narrative frame and character voice.
   - Match the style exemplars more closely than the previous draft.
   ```

8. Save the assembled prompt to `workspace/DRAFTS/prompts/NN_title_rewrite_prompt.txt` for audit.
9. Generate the rewritten narrative chapter by following the prompt. The agent acts as the writer.
10. Write the new prose to `workspace/DRAFTS/NN_title_draft_v2.md` with YAML frontmatter:
    ```yaml
    ---
    section: 1
    plan_title: "The First Lesson"
    status: draft-v2
    rewritten_from: "NN_title_draft.md"
    rewrite_reason: "{summary_of_issues}"
    ---
    ```
11. If generation produces no output, write a placeholder with `status: pending`.

### Phase 3: Update Drafts Index and Log

1. Update `workspace/DRAFTS/drafts_index.md`:
   - List rewritten sections with `status: draft-v2`.
   - Link to both the original and rewritten files.
2. Write `workspace/DRAFTS/rewrite_log.md`:
   - YAML frontmatter with `started` and `completed` timestamps.
   - List of rewritten sections with original file, new file, and issue summary.
   - Any sections that failed to rewrite.

### Phase 4: Update Checklist

Update `workspace/CHECKLISTS/pipeline_checklist.md`:
- Find/replace the Step I line to `[X]` with started and completed timestamps.

## Single Section Mode

If the user asks for a single section rewrite (e.g., "rewrite section 3 only"), process only that section. Useful for targeted fixes.

## Resume Behavior

If the user says "resume rewrite", check `rewrite_log.md`. Skip any sections already marked `completed`. Regenerate any marked `pending`.

## Notes

- Rewritten drafts preserve the original. The original `NN_title_draft.md` is never overwritten.
- If a section is rewritten multiple times, use `v3`, `v4`, etc.
- All rewrite prompts are saved for inspection, just like first-draft prompts.
- Factual accuracy remains the highest priority. Rewrite instructions from the Fidelity Auditor take precedence over style instructions from the Style Adversary.
