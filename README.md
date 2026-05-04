# From Paper to Page

> *Technical papers are exact. But reading them feels like friction. So we borrowed from fiction.*

A structured pipeline that turns dry nonfiction into prose that reads like a novel — without losing what matters.

---

## The Problem It Solves

Most nonfiction is written to inform. It succeeds at that and fails at everything else. The reader learns what happened, or how something works, but they do not feel it. They do not remember it. They do not care.

Narrative prose does the opposite. It makes the reader live inside the material. But writing it takes a skill most subject-matter experts do not have — and most fiction writers do not want to learn enough science to apply theirs.

This pipeline bridges that gap. It is not a replacement for either kind of writer. It is scaffolding: a structured process that breaks the transformation into inspectable, reversible steps, so that a human with taste can guide an agent throughout the process.

---

## What You Put In, What You Get Out

You give it two things:

1. **A nonfiction source** — research paper, technical report, biography, documentary transcript, anything with facts and structure
2. **A prose style reference** — a novel, memoir, or short story collection whose voice you want to borrow (the full text, or at minimum 3–4 representative chapters)

It produces a narrative version of the source: written in the style of the reference, accurate to the original facts, structured as a readable story. Depending on source length, expect a full run to take 60–300 minutes and produce a draft of roughly the same word count as the source. Time is highly varient on complexity, and size of both texts.

---

## Before You Start

Two system dependencies are required for the conversion step:

- **pandoc** — for DOCX, HTML, and LaTeX sources
- **pdftotext** (from poppler-utils) — for PDF sources

If either is missing when you run the pipeline, the agent stops and tells you. Install them before you begin or let the agent do that for you.

```bash
# macOS
brew install pandoc poppler

# Ubuntu / Debian
sudo apt install pandoc poppler-utils
```

---

## How to Use It

1. Install the pipeline skills (see [Installation](#installation))
2. Create a working directory and place your files:
   - `source.pdf` (or `.md`, `.txt`, `.docx`) — the nonfiction source
   - `style.md` — the prose style reference (full text or representative chapters)
3. Tell the agent:
   ```
   Run paper-to-narrative on source.pdf with style.md
   ```
4. Steps A and B run automatically — conversion and sectioning.
5. Step C analyzes the style reference and extracts candidate paragraphs.
6. **Step D stops and asks you to pick the best exemplars.** Respond before the pipeline continues.
7. **Step E stops and asks you to choose a narrative concept.** Respond before the pipeline continues.
8. Steps F through I run automatically — planning, writing, review, and rewriting.
9. Step J exports a clean PDF to `workspace/DRAFTS/`.

---

## The Philosophy

### Agents are scaffolding, not authors

The agent does not write the book. It prepares the materials, proposes options, and executes mechanical work. Every creative decision. From which paragraphs best capture the style and what narrative frame to use, is gated behind human approval. The pipeline has two mandatory Human Gates (Steps D and E) that force the agent to stop and ask. It cannot proceed without your answer. (Unless you explicitly let it.)

### Every state is a file

There is no hidden context, no session memory that matters. Every intermediate output is a human-readable markdown file in `workspace/`. You can open any file, edit it by hand, and resume the pipeline. The agent does not own the state. You do.

### Quality over speed

There are ten steps, two of which stop for your input. This is slower than asking a single LLM prompt to "rewrite this paper like Hemingway." It is also better. The output is inspectable at every stage. If the style analysis misses something, you can edit it. If the narrative concept is wrong, you reject it before a single chapter is written. You are not debugging a finished draft, you are steering the process.

### Deterministic enough to discard

The entire skill-set is pure markdown instructions. No Python, no CLI, no dependencies beyond pandoc and pdftotext for the initial conversion. If the agent disappears tomorrow, the instructions are still readable. Another agent, a human, or a future tool can follow them.

---

## What Makes It Different

| Approach | What Happens | The Problem |
|---|---|---|
| Single prompt | "Rewrite this paper like [author]" | Hallucinates, flattens nuance, produces generic pastiche |
| Fine-tuning | Train a model on the author's work | Expensive, legally murky, no structural guidance |
| Human ghostwriting | Hire a writer | Slow, expensive, writer may not understand the source |
| **This pipeline** | Structured transformation with human gates | Facts stay intact, prose mechanics are real, you steer every creative choice |

---

## The Pipeline

★ marks steps that require your input before the pipeline can continue.

| Step | Type | What Happens |
|---|---|---|
| A — Convert | Automated | Source document becomes markdown |
| B — Section | Automated | Split into topical sections by content and size |
| C — Analyze | Automated | Style source is read chapter by chapter; paragraphs are scored on tone, rhythm, imagery, dialogue, description, and fit |
| D — Select ★ | **Human Gate** | Agent presents top candidates per factor. You pick the ones that best represent the style. |
| E — Concept ★ | **Human Gate** | Agent presents 2–3 narrative frames. You choose the story concept that will drive every chapter. |
| F — Plan | Automated | Rewrite blueprints built per section, using your chosen exemplars and concept |
| G — Write | Automated | Narrative chapters generated, one per section, with saved prompts for audit |
| H — Review | Automated | Quality gate: continuity, style consistency, and factual accuracy checked against the source |
| I — Rewrite | Automated | Flagged sections rewritten using review feedback as additional constraints |
| J — Export | Automated | Clean final PDF produced with no pipeline metadata |

Steps D and E are the core of the system. They exist because an agent cannot know what you like, and it cannot invent a story frame you believe in. It can only propose. You decide.

---

## Configuration

`workspace/CONFIG/settings.md` controls the pipeline behavior:

```yaml
style_anchor_count: 3               # how many style chapters to keep for context
style_anchor_selection: first_middle_last  # first_middle_last | random | specific
max_words_per_section: 2000
min_words_per_section: 200
exemplars_per_factor: 10
exemplars_to_keep: 5
truncate_at: paragraph              # paragraph | sentence | char
```

---

## Workspace

Every file is markdown. Every state is inspectable and editable.

```
workspace/
├── SOURCES/
│   ├── SOURCE.md              # converted source
│   ├── STYLE_SOURCE.md        # style reference
│   ├── STYLE_ANCHORS.md       # trimmed anchor chapters for context efficiency
│   └── metadata.md
├── SECTIONS/
│   ├── 01_introduction.md
│   └── sections_index.md
├── ANALYSIS/
│   ├── style_chapters/        # per-chapter breakdowns
│   ├── exemplars/             # candidate paragraphs with scores
│   ├── selected_exemplars.md  # your chosen exemplars
│   └── style_profile.md
├── CONCEPT/
│   └── story_concept.md       # your chosen narrative frame
├── PLAN/
│   ├── rewrite_strategy.md
│   ├── chapter_plans/
│   └── chapter_plan_index.md
├── DRAFTS/
│   ├── prompts/               # saved prompts for every chapter (for audit)
│   ├── 01_introduction_draft.md
│   ├── rewrite_log.md
│   └── drafts_index.md
├── REVIEW/
│   ├── review_report.md
│   ├── rewrite_decision.md
│   ├── section_notes/
│   └── rewrite_requests.md
├── CHECKLISTS/
│   └── pipeline_checklist.md  # tracks what is done and where to resume
└── CONFIG/
    └── settings.md
```

You can edit any file and resume. The checklist tells the agent where to pick up.

---

## Installation

### Install from the flattened repo. Each sub-skill is self-contained.

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
hermes skills install https://raw.githubusercontent.com/StanLukuvka/paper-to-narrative/main/paper-to-narrative-rewrite/SKILL.md
```

### Settings to configure

- Increase the delegate_task timeout for agents to an amount greater than 10 minutes or else its likely it will time out.


---

## Notes

- Handles any nonfiction — not just academic papers. The name stuck.
- Any step can be rerun independently by reloading its skill.
- The story concept file is a living document. Edit `workspace/CONCEPT/story_concept.md` before Step F if you change your mind.
- After Step H (Review), the agent presents three options: proceed, rewrite flagged sections, or halt. Step I handles rewrites if you choose that path.
