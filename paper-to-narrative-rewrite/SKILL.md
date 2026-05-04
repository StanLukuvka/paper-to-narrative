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

Rewrite flagged draft sections using review feedback as additional constraints. The agent performs rewrites **directly** rather than delegating to sub-agents, because voice-sensitive narrative prose requires the agent to maintain tonal continuity across sections.

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
9. Write `workspace/REWRITE_BRIEF.md` with:
   - Narrative frame and character notes from `story_concept.md`
   - Prose rules (sentence variation, sensory details, dialogue, etc.)
   - Metaphor system reminders
   - Fidelity notes from the review (e.g., "Add missing normative rule X to Section Y")
   - Sections to rewrite, organized by priority
   This brief is shared across all batches and serves as the master reference for tonal consistency.

### Phase 2: Build Batch Queue

1. Sort the rewrite queue by section number.
2. Group into batches based on file size. See the main skill's "Batch Size Guidance" table. For rewrite (direct agent work), use the "Rewrite Batch" column.
3. For each section, note its corresponding plan file and section notes file paths.
4. Discover actual filenames on disk using `ls` or `search_files(target='files')`. Do not hardcode filenames.

### Phase 3: Process Batches (Direct Agent Rewrite)

For each batch:

1. **Read batch inputs:** Read the draft files, plan files, and section notes for all sections in this batch. Hold them in working memory.
2. **Write a rewrite brief** to `workspace/REWRITE_BRIEF.md` (if it does not already exist) containing:
   - Narrative frame and character notes from `story_concept.md`
   - Prose rules (sentence variation, sensory details, dialogue, etc.)
   - Metaphor system reminders
   - Fidelity notes from the review (e.g., "Add cache invalidation rule to Section 19")
   This brief is shared across all batches.
3. **Generate rewrites:** For each section in the batch:
   a. Read the current draft directly.
   b. Rewrite it in-place, preserving all technical accuracy while applying the prose rules from the brief.
   c. Address every issue listed in the section notes for this draft.
   d. Overwrite the original file (or write to `NN_title_draft_v2.md` if you want to preserve originals).

**Why direct rewrite instead of sub-agents:** Sub-agents are isolated and cannot maintain tonal continuity across sections. They also time out on voice-sensitive creative work. The direct agent has full access to the brief, exemplars, and previously rewritten sections, allowing consistent voice.

**Pacing:** If a batch contains many large sections (>10KB each), process them sequentially rather than trying to hold all in memory at once. Read one, rewrite one, write one, then move to the next.

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
