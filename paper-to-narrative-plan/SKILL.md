---
name: paper-to-narrative-plan
description: "Create rewrite plan: how each research section becomes a narrative chapter."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, plan, blueprint]
---

# Step E: Plan

## Purpose

Design the narrative transformation before writing. For each research section, decide the narrative frame, style anchors, metaphor ideas, and structural hooks.

## Inputs

- `workspace/SECTIONS/*.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/selected_exemplars.md`

## Outputs

- `workspace/PLAN/rewrite_strategy.md`
- `workspace/PLAN/chapter_plans/NN_section_plan.md` (one per section)
- `workspace/PLAN/chapter_plan_index.md`

## Instructions

1. Read every section file from `workspace/SECTIONS/`.
2. Read `workspace/ANALYSIS/style_profile.md` and `workspace/ANALYSIS/selected_exemplars.md`.
3. For each section, infer a narrative frame based on its title:
   - Introduction → opening scene establishing the world
   - Methods/Protocol → practitioner demonstrating technique to an apprentice
   - Results/Findings → discovery scene where truths are unveiled
   - Discussion → evening conversation reflecting on what was learned
   - Conclusion → closing scene tying threads together
   - Other → narrative exposition
4. Write a plan file to `PLAN/chapter_plans/NN_title_plan.md` with YAML frontmatter:
   ```yaml
   ---
   section: 1
   original_title: "Introduction"
   plan_title: "The First Lesson"
   frame: "Opening scene establishing the world and stakes"
   words_target: 450
   ---
   ```
   The body of the plan should contain:
   - Narrative frame description
   - Original concepts to preserve (key claims, methods, numbers)
   - Style anchors (relevant exemplars)
   - Opening hook suggestion
   - Closing hook suggestion
   - Word target (plus or minus 10 percent of original length)
5. Write `PLAN/rewrite_strategy.md` containing overall rules:
   - Preserve all factual claims
   - Translate jargon into consistent metaphors
   - Match sentence rhythm and imagery to selected exemplars
   - Never add information not in the research section
6. Write `PLAN/chapter_plan_index.md` mapping sections to plan files.
7. Mark step E complete in `workspace/CHECKLISTS/pipeline_checklist.md`.

## Notes

- Plans are living documents. The user can edit them before running step F.
- Word targets are guides, not hard constraints.
