---
name: paper-to-narrative-review
description: "Review drafts for continuity, style fidelity, worldbuilding depth, and technical accuracy."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, review, quality, audit]
---

# Step H: Review

## Purpose

Quality gate before final output. Check every draft for consistency across the book, fidelity to the style source, depth of worldbuilding, and accuracy against the nonfiction source.

## Inputs

- `workspace/DRAFTS/*.md`
- `workspace/DRAFTS/drafts_index.md`
- `workspace/PLAN/chapter_plans/*.md`
- `workspace/PLAN/rewrite_strategy.md`
- `workspace/SOURCES/SOURCE.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/CONCEPT/story_concept.md`

## Outputs

- `workspace/REVIEW/review_report.md`
- `workspace/REVIEW/section_notes/NN_title_notes.md` (one per draft)
- `workspace/REVIEW/rewrite_requests.md` (optional, only if issues found)

## Prerequisites

Before starting, validate:
1. `workspace/DRAFTS/` exists and contains at least one draft file.
2. `workspace/PLAN/` exists with corresponding plan files.
3. `workspace/SOURCES/SOURCE.md` exists.

If any prerequisite is missing, abort with a clear message naming the missing file.

## Instructions

**Idempotency guard:** Before doing any work, check if `workspace/REVIEW/review_report.md` exists and the checklist marks Step H complete. If both are true, skip all work and report: "Step H: Review: already completed, skipping."

1. Read every draft in `workspace/DRAFTS/` and its matching plan in `workspace/PLAN/chapter_plans/`.
2. Read `workspace/CONCEPT/story_concept.md` and `workspace/PLAN/rewrite_strategy.md`.
3. For each draft, write a review note to `workspace/REVIEW/section_notes/NN_title_notes.md` assessing these five areas:

   ### 1. Metaphor Consistency
   Does the draft use the same metaphor regime as earlier sections? Flag any new magic/technical vocabulary that was not established in previous sections. A book should commit to one metaphor system per domain, not invent new terms section-by-section.

   ### 2. Narrative Continuity
   Do characters recur with consistent names and roles? Is there a through-line of discovery or escalating mystery? Does each section connect to the next, or does it open/close in isolation?

   ### 3. Style Fidelity
   Does the prose rhythm, dialogue patterns, and tone match the selected exemplars? Flag passages that feel flat, too modern, too technical, or tonally inconsistent with the style source.

   ### 4. Worldbuilding Depth
   Is the setting lived-in with its own traditions and history, or is it a surface translation (e.g., "Faraday cage → warding circle" with no further texture)? Flag one-to-one swaps that could be deepened.

   ### 5. Technical Accuracy
   Cross-check every numerical value, name, date, figure, process step, and reference against `workspace/SOURCES/SOURCE.md`. Flag any missing numbers, altered ranges, or introduced details not in the source.

4. Write `workspace/REVIEW/review_report.md` summarizing:
   - Overall verdict: pass, pass with notes, or needs rewrite
   - Per-section grades (pass / minor issues / major issues)
   - Cross-cutting issues (metaphor drift, continuity gaps, etc.)
   - Recommended fixes, organized by priority

5. If any section has major issues, write `workspace/REVIEW/rewrite_requests.md` listing:
   - Section number and title
   - Issue category (continuity / style / accuracy / worldbuilding)
   - Specific instruction for the rewrite

6. Present the review report to the user. If rewrite_requests.md exists, ask whether to:
   - Proceed to final merge anyway
   - Rewrite specific sections now (specify which)
   - Halt for manual editing

7. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step H line to `[X]` with started and completed timestamps.

## Notes

- This step is a gate, not an automatic rewrite trigger. The user decides what to fix.
- Minor style notes can be addressed in a future revision; missing data or broken continuity should block release.
- The agent can perform the technical audit by comparing draft text against SOURCE.md directly.
