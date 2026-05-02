---
name: paper-to-narrative-convert
description: "Convert source document to markdown."
version: 0.1.0
author: stanl
license: mit
requires_toolsets: [terminal]
metadata:
  hermes:
    tags: [paper2narrative, convert, markdown]
---

# Step A: Convert

## Purpose

Convert the research paper from any pandoc-supported format into clean markdown and place it in the workspace.

## Inputs

- Path to source file (PDF, HTML, DOCX, LaTeX, or Markdown)
- Path to style reference file (Markdown)

## Outputs

- `workspace/SOURCES/INFO_SOURCE.md`
- `workspace/SOURCES/STYLE_SOURCE.md`
- `workspace/SOURCES/metadata.md`

## Prerequisites

Before starting, validate:
1. Source file exists and is readable.
2. Source file extension is one of: `.pdf`, `.md`, `.markdown`, `.docx`, `.html`, `.tex`, `.txt`.
3. Style reference file exists and is readable.
4. `pandoc` and/or `pdftotext` are available on PATH (for non-markdown sources).

If any check fails, abort with a clear error message. Do not proceed.

## Instructions

1. Ensure `workspace/SOURCES/` exists. Create it if needed.
2. Check the file extension of the source:
   - If `.md` or `.markdown`: copy it directly to `workspace/SOURCES/INFO_SOURCE.md`.
   - If `.pdf`: run `pdftotext` to extract text, clean form-feed characters (`\f`), and save as markdown.
   - Otherwise: run `pandoc -t gfm -o workspace/SOURCES/INFO_SOURCE.md` on the source file.
3. Copy the style reference to `workspace/SOURCES/STYLE_SOURCE.md`.
4. Write `workspace/SOURCES/metadata.md` with YAML frontmatter:
   ```yaml
   ---
   info_source: "path/to/original"
   style_source: "path/to/style"
   info_md: "workspace/SOURCES/INFO_SOURCE.md"
   timestamp: "2026-05-01T14:30:00Z"
   ---
   ```
5. Update `workspace/CHECKLISTS/pipeline_checklist.md`:
   - Find the line `- [ ] Step A: Convert` (or append if missing).
   - Replace it with:
     ```markdown
     - [X] Step A: Convert
       - started: <timestamp>
       - completed: <timestamp>
     ```

## Edge Cases

- If `pdftotext` or `pandoc` is not installed, stop and report the missing dependency.
- If the PDF has no text layer, note this in metadata and proceed with best-effort text.
