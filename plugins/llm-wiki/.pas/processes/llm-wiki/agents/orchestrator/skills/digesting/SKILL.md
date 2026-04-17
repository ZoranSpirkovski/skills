---
name: digesting
description: Use when the user says "digest this", "preprocess these files", drops files into `.wiki/raw/` and asks to prepare them, or says "digest all undigested files in raw/". Also triggered automatically by the ingesting skill when a source has no existing digest.
---

# Digesting

Preprocess a raw source into clean, structured markdown so that ingest works from a normalized, inspectable artifact instead of wrestling with file formats.

## Steps

1. **Identify the source** in `.wiki/raw/` and derive a slug for it.
2. **Create the digest directory**, mirroring the source's folder path from `raw/`. If the source is at `.wiki/raw/Project/Subfolder/file.pdf`, create `.wiki/digested/Project/Subfolder/<slug>/`. The folder hierarchy matches `raw/`; only the leaf directory uses the slug name.
3. **Extract text.** Use the best available tool for the format:
   - `.md`, `.txt`, `.html` — Read tool, copy content as-is.
   - `.pdf` — `pdftotext` (poppler-utils) if available, otherwise Read tool.
   - `.docx` — Read tool or `python-docx` if available.
   - `.xlsx` — `openpyxl` to render markdown tables if available, otherwise note as unextractable.
   - `.csv` — Read tool, copy as-is.

   Save extracted text to the digest directory as `text.md`. If a tool isn't installed, note what was unavailable and proceed with what works. Digest degrades gracefully — never fail because an optional tool is missing.
4. **Extract images** (PDF and docx only). Use `pdfimages` (poppler-utils) or equivalent if available. Save extracted images in the digest directory as `img-001.png`, `img-002.png`, etc. If extraction tools aren't available, skip this step and note it.
5. **Describe images visually.** For each extracted image (or for the source file itself if it's a JPEG/PNG), use the Read tool to view it and write a description capturing labels, annotations, spatial relationships, and layout details.
6. **Write `digest.md`** in the digest directory, combining everything:
   ```markdown
   # <Source Title>

   **Raw file:** `<path to original>`
   **Format:** <format> | **Digested:** <date>
   **Extraction notes:** <what tools were used, what was lossy, what was unavailable>

   ## Text Content
   <extracted text, cleaned up>

   ## Image 1: <filename>
   <visual description>

   ## Image 2: <filename>
   ...
   ```
   For text-only sources, the `## Image` sections are omitted.
7. **Append to `.wiki/wiki/log.md`** with the canonical prefix:
   ```
   ## [YYYY-MM-DD] digest | <source title>
   ```
   Note the format, tools used, and any extraction gaps.
8. **Report back.** Show the digest path, format detected, extraction method used, and any gaps the user should review.

## Output

```
Digested: <source title>

Output: .wiki/digested/<path>/<slug>/digest.md
Format: <format detected>
Extraction: <tools used, e.g. "pdftotext + pdfimages + visual Read">
Images: <N> extracted and described (or "none")
Gaps: <anything lossy or unreadable> (or "none")

Log entry:
## [YYYY-MM-DD] digest | <title>
```

## Common Mistakes

- **Running text-only extraction on image-heavy sources.** Floor plans, annotated diagrams, scanned documents produce empty or degraded text output. Always extract images and describe them visually.
- **Failing because a tool is missing.** Digest degrades gracefully — note what was unavailable and produce what you can.
- **Digesting sources that don't need it.** Plain `.md` or `.txt` sources pasted in chat don't need a digest; ingest reads them directly.
- **Skipping the log entry.** Every digest gets a log entry, even if it's just "pdftotext failed, text extraction skipped."
