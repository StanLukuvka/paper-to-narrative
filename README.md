# Paper-to-Narrative

A Hermes skill-set that transforms dry research papers into engaging narrative prose using a style reference.

## What This Is

Pure markdown instructions. No Python. No CLI. The Hermes agent reads these SKILL.md files and performs every step by reading and writing files in a workspace directory.

## How to Use

1. Place your source files in a working directory:
   - `info.pdf` (or .md, .docx, etc.) — the research paper
   - `style.md` — the prose style reference

2. Tell the agent:
   ```
   Run paper-to-narrative on info.pdf with style.md
   ```

3. The agent loads `SKILL.md`, creates `workspace/`, and executes steps A-F.

## The Pipeline

| Step | Skill | Action |
|------|-------|--------|
| A | convert | Convert source to markdown |
| B | section | Split into topical sections |
| C | analyze | Analyze style and extract exemplars |
| D | select | Pick best exemplars (interactive) |
| E | plan | Create rewrite blueprints |
| F | write | Generate narrative chapters |

## Workspace

Every intermediate state is a human-readable markdown file in `workspace/`:
- `SOURCES/` — converted inputs
- `SECTIONS/` — split paper
- `ANALYSIS/` — style breakdown and exemplars
- `PLAN/` — rewrite blueprints
- `DRAFTS/` — generated chapters and saved prompts
- `CHECKLISTS/` — pipeline progress

## Structure

```
paper-to-narrative/
├── SKILL.md              # main orchestrator
├── README.md             # this file
└── skills/
    ├── convert/SKILL.md
    ├── section/SKILL.md
    ├── analyze/SKILL.md
    ├── select/SKILL.md
    ├── plan/SKILL.md
    └── write/SKILL.md
```

Each sub-skill is self-contained instructions for one pipeline step.
