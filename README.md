# Paper-to-Narrative

Turn dry nonfiction into prose that reads like a novel, without losing what matters.

## Why This Exists

Most nonfiction is written to inform. It succeeds at that and fails at everything else. The reader learns what happened, or how something works, but they do not feel it. They do not remember it. They do not care.

Narrative prose does the opposite. It makes the reader live inside the material. But writing it takes a skill most subject-matter experts do not have, and most fiction writers do not want to learn enough science to apply theirs.

This pipeline exists to bridge that gap. It is not a replacement for either kind of writer. It is scaffolding: a structured process that breaks the transformation into inspectable, reversible steps, so that a human with taste can guide an agent with stamina.

## What It Actually Does

You give it two things:

1. A nonfiction source (research paper, technical report, biography, documentary transcript, whatever has facts and structure)
2. A prose style reference (a novel, a memoir, a short story collection -- any text whose voice you want to borrow)

It produces a narrative version of the source, written in the style of the reference, accurate to the original facts, structured as a readable story.

## The Philosophy

### Agents Are Scaffolding, Not Authors

The agent does not write the book. It prepares the materials, proposes options, and executes mechanical work. Every creative decision -- which paragraphs best capture the style, what narrative frame to use -- is gated behind human approval. The pipeline has two mandatory Human Gates (Steps D and E) that force the agent to stop and ask. It cannot proceed without your answer.

### Every State Is a File

There is no hidden context, no session memory that matters. Every intermediate output is a human-readable markdown file in `workspace/`. You can open any file, edit it by hand, and resume the pipeline. The agent does not own the state. You do.

### Deterministic Enough to Discard

The entire skill-set is pure markdown instructions. No Python, no CLI, no dependencies beyond pandoc and pdftotext for the initial conversion. If Hermes disappears tomorrow, the instructions are still readable. Another agent, a human, or a future tool can follow them. The pipeline is designed to make itself obsolete.

### Quality Over Speed

There are eight steps, two of which stop for your input. This is slower than asking a single LLM prompt to "rewrite this paper like Hemingway." It is also better. The output is inspectable at every stage. If the style analysis misses something, you can edit it. If the narrative concept is wrong, you reject it before a single chapter is written. You are not debugging a finished draft. You are steering the process.

## Who This Is For

- Writers who want to turn research into readable prose without losing accuracy
- Researchers who want their work to reach people outside their field
- Anyone who has read a great novel and thought "I wish this author had written my textbook"
- People who do not trust black-box AI to make creative decisions for them

## What Makes It Different

| Approach | What Happens | The Problem |
|----------|-------------|-------------|
| Single prompt | "Rewrite this paper like [author]" | The model hallucinates, flattens nuance, and produces generic pastiche |
| Fine-tuning | Train a model on an author's work | Expensive, legally murky, and still produces output without structural guidance |
| Human ghostwriting | Hire a writer | Expensive, slow, and the writer may not understand the source |
| **This pipeline** | Structured transformation with human gates | Keeps facts intact, captures actual prose mechanics, and lets you steer every creative choice |

## The Pipeline

| Step | Type | What Happens |
|------|------|-------------|
| A -- Convert | Automated | Source document becomes markdown |
| B -- Section | Automated | Split into topical sections by content and size |
| C -- Analyze | Automated | Read style source chapter by chapter, score paragraphs on tone, rhythm, imagery, dialogue, description, and fit to source |
| D -- Select | **Human Gate** | Agent presents top candidates per factor. You pick which ones best represent the style. Agent cannot proceed without your answer. |
| E -- Concept | **Human Gate** | Agent presents 2-3 narrative frames. You choose the story concept that will drive every chapter. Agent cannot proceed without your answer. |
| F -- Plan | Automated | Build rewrite blueprints per section, using your chosen exemplars and concept |
| G -- Write | Automated | Generate narrative chapters, one per section, with saved prompts for audit |
| H -- Review | Automated | Quality gate: check continuity, style consistency, and factual accuracy against the source |

Steps D and E are the core of the system. They exist because an agent cannot know what you like, and it cannot invent a story frame you believe in. It can only propose. You decide.

## How to Use It

1. Install the skills (see Installation below)
2. Create a working directory and place your files:
   - `source.pdf` (or .md, .txt, .docx) -- the nonfiction source
   - `style.md` -- the prose style reference
3. Tell the agent:
   ```
   Run paper-to-narrative on source.pdf with style.md
   ```
4. The agent runs Steps A and B automatically.
5. At Step C, it analyzes the style source and extracts candidate paragraphs.
6. At Step D, it stops and asks you to pick the best ones. Respond.
7. At Step E, it stops and asks you to pick a narrative concept. Respond.
8. Steps F, G, and H run automatically. You get a finished draft in `workspace/DRAFTS/`.

## Workspace

Every file is markdown. Every state is inspectable.

```
workspace/
├── SOURCES/
│   ├── SOURCE.md              # your converted source
│   ├── STYLE_SOURCE.md        # your style reference
│   ├── STYLE_ANCHORS.md       # trimmed anchor chapters for context efficiency
│   └── metadata.md            # run metadata
├── SECTIONS/
│   ├── 01_introduction.md     # split source sections
│   └── sections_index.md
├── ANALYSIS/
│   ├── style_chapters/        # per-chapter breakdowns
│   ├── exemplars/             # candidate paragraphs with scores
│   ├── selected_exemplars.md  # your chosen exemplars
│   └── style_profile.md       # aggregate style notes
├── CONCEPT/
│   └── story_concept.md       # your chosen narrative frame
├── PLAN/
│   ├── rewrite_strategy.md    # overall approach
│   ├── chapter_plans/         # per-section blueprints
│   └── chapter_plan_index.md
├── DRAFTS/
│   ├── prompts/               # saved prompts for every chapter
│   ├── 01_introduction_draft.md
│   └── drafts_index.md
├── REVIEW/
│   ├── review_report.md       # continuity, style, accuracy checks
│   ├── section_notes/
│   └── rewrite_requests.md
├── CHECKLISTS/
│   └── pipeline_checklist.md  # tracks what is done and what is not
└── CONFIG/
    └── settings.md            # fidelity and integration knobs
```

You can edit any file and resume. The checklist tells the agent where to pick up.

## Configuration

`workspace/CONFIG/settings.md` controls the pipeline:

```yaml
---
style_anchor_count: 3               # how many style chapters to keep for context
style_anchor_selection: first_middle_last  # first_middle_last | random | specific
max_words_per_section: 2000
min_words_per_section: 200
exemplars_per_factor: 10
exemplars_to_keep: 5
truncate_at: paragraph              # paragraph | sentence | char
fidelity: medium                   # low | medium | high -- technical precision
integration: high                   # low | medium | high -- how tightly facts drive the plot
---
```

Fidelity and integration are the two axes that matter most. High fidelity means the narrative does not simplify or distort the source material. High integration means the plot is driven by the facts, not decorated around them. Both are adjustable per project.

## Installation

Install from the flattened repo. Each sub-skill is self-contained.

```bash
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-convert/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-section/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-analyze/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-select/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-concept/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-plan/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-write/SKILL.md
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-review/SKILL.md
```

## System Dependencies

The convert step needs these installed on the host:
- `pandoc` (for DOCX, HTML, LaTeX)
- `pdftotext` from poppler-utils (for PDF)

If either is missing, the agent stops and tells you.

## Notes

- The name is a joke that stuck. It handles any nonfiction, not just papers.
- You can rerun any step independently by reloading its skill.
- The story concept file is a living document. Edit it before Step F if you change your mind.
- Review (Step H) generates a report, not auto-rewrites. You read it and decide what to fix.
