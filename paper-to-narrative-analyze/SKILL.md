---
name: paper-to-narrative-analyze
description: "Analyze style source, extract exemplar paragraphs and write style guide."
version: 0.1.0
author: stanl
license: mit
metadata:
  hermes:
    tags: [paper2narrative, analyze, style, subagent]
---

# Step C: Analyze

## Purpose

Read the style source and the nonfiction source, then extract candidate exemplar paragraphs representing different stylistic factors. Score each candidate on both style quality and fit to the nonfiction source's content.

## Inputs

- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/SOURCES/SOURCE.md`
- `workspace/CONFIG/settings.md` (for exemplars_per_factor)

## Outputs

- `workspace/ANALYSIS/style_chapters/chN_analysis.md`
- `workspace/ANALYSIS/exemplars/exemplar_factor_N.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/exemplars_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SOURCES/STYLE_SOURCE.md` exists and is non-empty.
2. `workspace/SOURCES/SOURCE.md` exists and is non-empty.
3. Style source exceeds 1000 words. If shorter, warn: "Style source is very short (<1000 words). Results may be poor." but continue.
4. `workspace/ANALYSIS/` directories exist or can be created.

If STYLE_SOURCE.md or SOURCE.md is missing, abort: "Step C: required source not found. Run Step A first."

## Instructions

**Idempotency guard:** Before doing any work, check if this step's outputs already exist and the checklist marks it complete. If both are true, skip all work and report: "Step C: Analyze: already completed, skipping."

1. Read `workspace/SOURCES/STYLE_SOURCE.md`.
2. Read `workspace/SOURCES/SOURCE.md`.
3. Load config from `workspace/CONFIG/settings.md` (use defaults if missing).
4. Split the style source into chapters or logical blocks (use `##` headings, or group paragraphs if no headings).
5. For each chapter/block, write a brief analysis file to `ANALYSIS/style_chapters/chN_analysis.md` covering:
   - Word and paragraph counts
   - Average sentence length
   - Percentage of short sentences (8 words or fewer)
   - Tone words present
   - Whether dialogue occurs
6. Read every paragraph in the style source and score it against each factor below. A paragraph can score for multiple factors. Assign two scores per paragraph per factor:

   **style_score** (0.0 to 1.0): How strongly and skillfully does this paragraph exhibit the factor? Judge by reading the paragraph and assessing its craft, not by counting words. Trust your understanding of prose.

   | Factor | What to look for |
   |--------|------------------|
   | **tone** | Mood-setting atmosphere; emotional color; atmospheric descriptions |
   | **rhythm** | Sentence length variation; mix of short punchy and long flowing sentences |
   | **imagery** | Sensory language; concrete metaphors; vivid descriptive detail |
   | **dialogue** | Quoted speech; naturalistic back-and-forth; distinct speaker voices |
   | **description** | World-building density; setting exposition; object and place specificity |
   | **pacing** | Action verbs; paragraph length controlling speed; urgency or stillness |
   | **character_voice** | First-person or strongly distinctive narration; personality-laden observations |
   | **humor** | Comedic timing; irony; wit; absurd juxtapositions; light-hearted wordplay |

   **fit_to_source** (0.0 to 1.0): How thematically relevant is this paragraph to the nonfiction source? Does it touch on similar subject matter, metaphors, or conceptual territory? A paragraph about underground networks fits a paper on fungal mycelium better than a paragraph about spaceships. Judge by conceptual overlap, not keyword matching.

7. For each factor, produce a combined score: `(style_score * 0.6) + (fit_to_source * 0.4)`. Keep the top-scoring `exemplars_per_factor` paragraphs per factor using this combined score. If two paragraphs tie, keep the one with higher `fit_to_source`; if still tied, keep the earlier one.

8. Write each kept exemplar to `ANALYSIS/exemplars/exemplar_factor_N.md` with YAML frontmatter:
    ```yaml
    ---
    factor: tone
    chapter: 5
    style_score: 0.85
    fit_to_source: 0.72
    combined_score: 0.80
    rank: candidate
    ---
    <paragraph text>
    ```

9. Write `ANALYSIS/style_profile.md` summarizing the voice characteristics.
10. Write `ANALYSIS/exemplars_index.md` listing all factors and candidate counts.
11. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
    - Find/replace the Step C line to `[X]` with started and completed timestamps.

## Notes

- If the style source is very long, you may use a sub-agent via `delegate_task` to process chapters in parallel. Default max concurrent children is 3; spawn at most 3 sub-agents at a time.
- Keep exemplars concise: one paragraph each.
- A single paragraph can appear under multiple factors if it scores well for each.
- `fit_to_source` ensures the exemplars are not just stylish, but relevant to what the nonfiction source is actually about.
