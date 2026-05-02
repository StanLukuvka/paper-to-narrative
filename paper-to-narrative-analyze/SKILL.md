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

Read the style source and extract candidate exemplar paragraphs representing different stylistic factors.

## Inputs

- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/CONFIG/settings.md` (for exemplars_per_factor)

## Outputs

- `workspace/ANALYSIS/style_chapters/chN_analysis.md`
- `workspace/ANALYSIS/exemplars/exemplar_factor_N.md`
- `workspace/ANALYSIS/style_profile.md`
- `workspace/ANALYSIS/exemplars_index.md`

## Prerequisites

Before starting, validate:
1. `workspace/SOURCES/STYLE_SOURCE.md` exists and is non-empty.
2. Style source exceeds 1000 words. If shorter, warn: "Style source is very short (<1000 words). Results may be poor." but continue.
3. `workspace/ANALYSIS/` directories exist or can be created.

If STYLE_SOURCE.md or INFO_SOURCE.md is missing, abort: "Step C: required source not found. Run Step A first."

## Instructions

**Idempotency guard:** Before doing any work, check if this step's outputs already exist and the checklist marks it complete. If both are true, skip all work and report: "Step C: Analyze: already completed, skipping."

1. Read `workspace/SOURCES/STYLE_SOURCE.md`.
2. Load config from `workspace/CONFIG/settings.md` (use defaults if missing).
3. Split it into chapters or logical blocks (use `##` headings, or group paragraphs if no headings).
4. For each chapter/block, write a brief analysis file to `ANALYSIS/style_chapters/chN_analysis.md` covering:
   - Word and paragraph counts
   - Average sentence length
   - Percentage of short sentences (8 words or fewer)
   - Tone words present
   - Whether dialogue occurs
5. For each chapter/block, scan every paragraph and score it against each factor below. A paragraph can score for multiple factors. Use a simple 0.0-1.0 heuristic: count sentences that match the factor's markers, divided by total sentences in the paragraph.

   | Factor | What to look for |
   |--------|------------------|
   | **tone** | Mood-setting words (gloomy, serene, tense, joyful); atmospheric descriptions; emotional color |
   | **rhythm** | Noticeable variation in sentence length within the paragraph; mix of short punchy and long flowing sentences |
   | **imagery** | Sensory language (sight, sound, smell, touch, taste); concrete metaphors; vivid adjectives |
   | **dialogue** | Quoted speech; naturalistic back-and-forth; distinct speaker voices |
   | **description** | World-building details; setting exposition; object/place specificity; historical or cultural texture |
   | **pacing** | Action verbs; short paragraphs; quick scene changes; urgency markers |
   | **character_voice** | First-person narration; distinctive diction or syntax; internal monologue; personality-laden observations |
   | **humor** | Comedic timing; irony; wit; absurd juxtapositions; light-hearted wordplay |

6. For each factor, keep the top-scoring `exemplars_per_factor` paragraphs (default 5). If two paragraphs from the same chapter tie on score, keep the earlier one.
7. Write each kept exemplar to `ANALYSIS/exemplars/exemplar_factor_N.md` with YAML frontmatter:
   ```yaml
   ---
   factor: tone
   chapter: 5
   score: 0.85
   rank: candidate
   ---
   <paragraph text>
   ```
8. Write `ANALYSIS/style_profile.md` summarizing the voice characteristics.
9. Write `ANALYSIS/exemplars_index.md` listing all factors and candidate counts.
10. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
    - Find/replace the Step C line to `[X]` with started and completed timestamps.

## Notes

- If the style source is very long, you may use a sub-agent via `delegate_task` to process chapters in parallel.
- Keep exemplars concise: one paragraph each.
- A single paragraph can appear under multiple factors if it scores well for each.
