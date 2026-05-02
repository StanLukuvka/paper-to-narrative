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

Design the narrative transformation before writing. For each research section, decide the narrative frame, style anchors, metaphor ideas, and structural hooks. This step is rule-based; it does not call an LLM.

## Inputs

- `workspace/SECTIONS/*.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/SOURCES/INFO_SOURCE.md` (to extract key claims)

## Outputs

- `workspace/PLAN/rewrite_strategy.md`
- `workspace/PLAN/chapter_plans/NN_section_plan.md` (one per section)
- `workspace/PLAN/chapter_plan_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SECTIONS/` exists and contains at least one `.md` file.
2. `workspace/ANALYSIS/selected_exemplars.md` exists and is non-empty.
3. `workspace/ANALYSIS/style_profile.md` exists.

If any prerequisite is missing, abort with a clear message naming the missing file and which step produces it.

## Instructions

1. Read every section file from `workspace/SECTIONS/`.
2. Read `workspace/ANALYSIS/style_profile.md` and `workspace/ANALYSIS/selected_exemplars.md`.
3. Read `workspace/SOURCES/INFO_SOURCE.md`.
4. For each section, infer a narrative frame based on its title:
   - Introduction → opening scene establishing the world
   - Methods/Protocol → practitioner demonstrating technique to an apprentice
   - Results/Findings → discovery scene where truths are unveiled
   - Discussion → evening conversation reflecting on what was learned
   - Conclusion → closing scene tying threads together
   - Other → narrative exposition
5. Extract key claims from the section using this rule-based method:
   - Take the first sentence of every paragraph in the section. These are the "original concepts to preserve."
   - If the section has fewer than 3 paragraphs, take the first and last sentence of each paragraph instead.
   - Never invent claims. Only copy verbatim from the section text.
6. Write a plan file to `PLAN/chapter_plans/NN_title_plan.md` with YAML frontmatter:
   ```yaml
   ---
   section: 1
   original_title: "Introduction"
   plan_title: "The First Lesson"
   frame: "Opening scene establishing the world and stakes"
   words_target: 450
   ---
   ```
   The body of the plan must contain:
   - Narrative frame description
   - Original concepts to preserve (the extracted first sentences, verbatim)
   - Style anchors (relevant exemplars from selected_exemplars.md)
   - Opening hook suggestion
   - Closing hook suggestion
   - Word target (plus or minus 10 percent of original section length)
7. Write `PLAN/rewrite_strategy.md` containing overall rules:
   - Preserve all factual claims (the extracted first sentences)
   - Translate jargon into consistent metaphors
   - Match sentence rhythm and imagery to selected exemplars
   - Never add information not in the research section
8. Write `PLAN/chapter_plan_index.md` mapping sections to plan files.
9. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step E line to `[X]` with started and completed timestamps.

## Notes

- Plans are living documents. The user can edit them before running step F.
- Word targets are guides, not hard constraints.
- No LLM is used in this step. All extraction is rule-based.
