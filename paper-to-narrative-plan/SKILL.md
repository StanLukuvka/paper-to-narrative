---
name: paper-to-narrative-plan
description: "Create rewrite plan: how each source section becomes a narrative chapter."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, plan, blueprint]
---

# Step F: Plan

## Purpose

Design the narrative transformation before writing. For each source section, decide the narrative frame, style anchors, metaphor ideas, and structural hooks. This step is rule-based.

## Inputs

- `workspace/SECTIONS/*.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/CONCEPT/story_concept.md`
- `workspace/SOURCES/SOURCE.md` (to extract key claims)
- `workspace/CONFIG/settings.md` (for fidelity and integration)

## Outputs

- `workspace/PLAN/rewrite_strategy.md`
- `workspace/PLAN/chapter_plans/NN_section_plan.md` (one per section)
- `workspace/PLAN/chapter_plan_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SECTIONS/` exists and contains at least one `.md` file.
2. `workspace/ANALYSIS/selected_exemplars.md` exists and is non-empty.
3. `workspace/ANALYSIS/selected_exemplars.md` contains `selected_by: user` in its YAML frontmatter. If this field is missing or set to `auto`, abort: "Step D (Select) must be completed with user input before planning. Run Step D and use the clarify tool."
4. `workspace/ANALYSIS/style_profile.md` exists.
5. `workspace/CONCEPT/story_concept.md` exists and contains `selected_by: user` in its YAML frontmatter. If `selected_by` is missing or not `user`, abort: "Step E (Concept) must be completed with user input before planning. Run Step E and use the clarify tool."

## Instructions

**Idempotency guard:** Before doing any work, check if this step's outputs already exist and the checklist marks it complete. If both are true, skip all work and report: "Step F: Plan: already completed, skipping."

1. Read every section file from `workspace/SECTIONS/`.
2. Read `workspace/ANALYSIS/style_profile.md` and `workspace/ANALYSIS/selected_exemplars.md`.
3. Read `workspace/SOURCES/SOURCE.md`.
4. Read `workspace/CONCEPT/story_concept.md`. Use the chosen narrative frame, perspective, and setting as the base for all plans.
5. For each section, map it to a story beat using the arc from `story_concept.md`. If the concept does not specify a mapping for a section title, infer one:
   - Introduction → opening scene establishing the world and stakes
   - Methods/Process → demonstration or practice scene
   - Results/Findings → discovery or revelation scene
   - Discussion → reflection or debate scene
   - Conclusion → closing scene tying threads together
   - Other → narrative exposition or transition
6. Extract key claims from the section using this rule-based method:
   - Take the first sentence of every paragraph in the section. These are the "original concepts to preserve."
   - If the section has fewer than 3 paragraphs, take the first and last sentence of each paragraph instead.
   - Never invent claims. Only copy verbatim from the section text.
7. Write a plan file to `PLAN/chapter_plans/NN_title_plan.md` with YAML frontmatter:
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
   - Fidelity note: how precise the numbers and methods should be in this section
   - Integration note: how deeply the science should drive the scene's action
8. Write `PLAN/rewrite_strategy.md` containing overall rules:
   - Preserve all factual claims (never invent, never omit)
   - Translate jargon into consistent metaphors
   - Match sentence rhythm and imagery to selected exemplars
   - Respect `fidelity` and `integration` settings from config:
     - **fidelity: low** — numbers become approximations ("three quarters" not "74.21%"); methods are sketched not detailed.
     - **fidelity: medium** — key numbers survive in rounded or narrativized form; methods are mentioned but not instruction-manual precise.
     - **fidelity: high** — exact figures stay exact where possible; methods remain intact and traceable.
     - **integration: low** — story is window-dressing around explanations; characters observe the science more than live it.
     - **integration: medium** — science and story interleave; characters discuss findings in narrative scenes.
     - **integration: high** — the science IS the plot; characters discover results in real time; the reader learns as the characters learn, through action and consequence.
9. Write `PLAN/chapter_plan_index.md` mapping sections to plan files.
10. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step F line to `[X]` with started and completed timestamps.

## Notes

- Plans are living documents. The user can edit them before running step G.
- Word targets are guides, not hard constraints.
