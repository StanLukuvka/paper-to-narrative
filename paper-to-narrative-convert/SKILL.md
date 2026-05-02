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
- Path to workspace directory

## Outputs

- `workspace/SOURCES/INFO_SOURCE.md`
- `workspace/SOURCES/original_info` (provenance record)
- `workspace/SOURCES/metadata.json`

## Instructions

1. Ensure `workspace/SOURCES/` exists. Create it if needed.
2. Check the file extension of the source:
   - If `.md` or `.markdown`: copy it directly to `workspace/SOURCES/INFO_SOURCE.md`.
   - If `.pdf`: run `pdftotext` to extract text, clean form-feed characters (``), and save as markdown.
   - Otherwise: run `pandoc -t gfm -o workspace/SOURCES/INFO_SOURCE.md` on the source file.
3. Write the original path into `workspace/SOURCES/original_info`.
4. Write `workspace/SOURCES/metadata.json` with:
   ```json
   {"info_source": "path/to/original", "info_md": "workspace/SOURCES/INFO_SOURCE.md", "timestamp": "ISO-date"}
   ```
5. Mark step A complete in `workspace/CHECKLISTS/pipeline_checklist.md`:
   ```markdown
   - [X] Step A: Convert
   ```

## Edge Cases

- If `pdftotext` or `pandoc` is not installed, stop and report the missing dependency.
- If the PDF has no text layer, note this in metadata and proceed with best-effort text.
