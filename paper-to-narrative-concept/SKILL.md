---
name: paper-to-narrative-concept
description: "Interactive story concept selection: present narrative frames for the nonfiction source and let the user choose. Human gate -- always stops for user input."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, concept, ideation, interactive]
---

# Step E: Concept

## Purpose

Before planning individual chapters, choose the overarching narrative frame. The agent reads the source sections and style exemplars, then presents 2-3 distinct story concepts. The user picks one, and that concept seeds every chapter plan.

This is a Human Gate. The agent MUST use the `clarify` tool and wait for the user's response. It cannot invent a concept on the user's behalf.

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

## Instructions

**Idempotency guard:** Before doing any work, check if `workspace/CONCEPT/story_concept.md` exists, contains `selected_by: user` in its YAML frontmatter, AND the checklist marks Step E complete. If all three are true, skip all work and report: "Step E: Concept: already completed, skipping."

If the file exists but `selected_by: user` is missing, treat the step as incomplete. Delete the file and restart.

### Phase 1: Generate and Present (Agent Action)

1. Read `workspace/SECTIONS/sections_index.md` to understand the source's structure and flow.
2. Read `workspace/ANALYSIS/selected_exemplars.md` and `workspace/ANALYSIS/style_profile.md` to understand the target voice.
3. Generate 2-3 distinct narrative concepts that frame the source as a story. Each concept must include:
   - A title (e.g., "The Electric Grove")
   - A one-sentence pitch
   - The narrative perspective (first-person journal, third-person limited, epistolary, etc.)
   - The setting or framing device (classroom, field expedition, apprenticeship, memoir, etc.)
   - How the source sections map to the narrative arc (e.g., Introduction = opening scene, Methods = demonstration scene, Results = discovery scene, Discussion = reflection, Conclusion = closing)
4. Present the concepts to the user using the `clarify` tool. Ask them to pick one or blend elements from multiple.
5. After calling `clarify`, STOP. Wait for the user's response in the next message. Do not proceed to any other work. Do not write any files yet.

### Phase 2: Write Output (After User Responds)

6. When the user responds, capture their choice (or blended choice).
7. Write `workspace/CONCEPT/story_concept.md` with YAML frontmatter:
   ```yaml
   ---
   concept_title: "<user's chosen title>"
   perspective: "<user's chosen perspective>"
   setting: "<user's chosen setting>"
   selected_by: user
   ---
   ```
   The body must contain:
   - The full pitch (exactly as chosen by the user)
   - Narrative arc mapping (which source section becomes which story beat)
   - Character or perspective notes
   - Tone and style notes drawn from the exemplars
8. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find/replace the Step E line to `[X]` with started and completed timestamps.

## Notes

- If the user declines all concepts, ask them to describe their own frame and write that into `story_concept.md` instead.
- The concept file is a living document. The user can edit it before Step F runs.
- This step is a Human Gate. The agent MUST use `clarify` and wait for a response. Never auto-generate a concept without user approval.
- This step is the creative anchor for the entire book. Everything after it follows the chosen frame.
