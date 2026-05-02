---
name: paper-to-narrative-concept
description: "Interactive story concept selection: present narrative frames for the research paper and let the user choose."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, concept, ideation, interactive]
---

# Step E: Concept

## Purpose

Before planning individual chapters, choose the overarching narrative frame. The agent reads the research sections and style exemplars, then presents 2-3 distinct story concepts. The user picks one, and that concept seeds every chapter plan.

## Inputs

- `workspace/SECTIONS/sections_index.md`
- `workspace/ANALYSIS/selected_exemplars.md`
- `workspace/ANALYSIS/style_profile.md`

## Outputs

- `workspace/CONCEPT/story_concept.md`

## Prerequisites

Before starting, validate:
1. `workspace/SECTIONS/sections_index.md` exists and lists at least one section.
2. `workspace/ANALYSIS/selected_exemplars.md` exists and is non-empty.
3. `workspace/ANALYSIS/style_profile.md` exists.

If any prerequisite is missing, abort with a clear message naming the missing file.

## Instructions

**Idempotency guard:** Before doing any work, check if `workspace/CONCEPT/story_concept.md` already exists and the checklist marks Step E complete. If both are true, skip all work and report: "Step E: Concept: already completed, skipping."

1. Read `workspace/SECTIONS/sections_index.md` to understand the paper's structure and flow.
2. Read `workspace/ANALYSIS/selected_exemplars.md` and `workspace/ANALYSIS/style_profile.md` to understand the target voice.
3. Generate 2-3 distinct narrative concepts that frame the research as a story. Each concept must include:
   - A title (e.g., "The Electric Grove")
   - A one-sentence pitch
   - The narrative perspective (first-person journal, third-person limited, epistolary, etc.)
   - The setting or framing device (laboratory, field expedition, apprenticeship, etc.)
   - How the research sections map to the narrative arc (e.g., Introduction = opening scene, Methods = demonstration scene, Results = discovery scene, Discussion = reflection, Conclusion = closing)
4. Present the concepts to the user using the `clarify` tool. Ask them to pick one or blend elements from multiple.
5. Write `workspace/CONCEPT/story_concept.md` with YAML frontmatter:
   ```yaml
   ---
   concept_title: "The Electric Grove"
   perspective: "third-person limited"
   setting: "A wandering field researcher's enchanted workshop"
   selected_by: user
   ---
   ```
   The body must contain:
   - The full pitch
   - Narrative arc mapping (which research section becomes which story beat)
   - Character or perspective notes
   - Tone and style notes drawn from the exemplars
6. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step E line to `[X]` with started and completed timestamps.

## Notes

- If the user declines all concepts, ask them to describe their own frame and write that into `story_concept.md` instead.
- The concept file is a living document. The user can edit it before Step F runs.
- This step is the creative anchor for the entire book. Everything after it follows the chosen frame.
