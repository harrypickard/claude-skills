---
name: data_dict_convert
description: Convert a PDF data dictionary/codebook into a searchable .md file. Use when asked to convert, parse, or index a data dictionary or codebook PDF. Extracts all text page-by-page, writes a brief intro header, then produces a single .md file alongside the original PDF. Split files are cleaned up automatically.
allowed-tools: Bash(python*), Bash(pip*), Bash(mkdir*), Bash(ls*), Bash(rm*), Read, Write, Edit
argument-hint: [path-to-pdf]
---

# data_dict_convert: Convert a PDF Data Dictionary to Markdown

## Purpose

Convert a PDF data dictionary or codebook into a plain `.md` file that can be searched with Grep across sessions. The output is raw text — no interpretation, no summarization — so it faithfully represents the original document and can be queried directly.

## When This Skill Is Invoked

The user provides a path to a PDF data dictionary or codebook (e.g., `data/raw/bes/panel/Bes_wave30Documentationv30.1.pdf`). The output `.md` file is written to the **same folder** as the original PDF.

If the user does not provide a path, ask for one. Do not guess.

## Step 1: Verify the PDF and install dependencies

- Confirm the file exists
- Check for PyPDF2; install if missing: `pip install PyPDF2`

## Step 2: Split the PDF into 4-page chunks

Split into 4-page chunks in a **temporary subdirectory** inside the same folder as the original PDF (e.g., `_splits_tmp/`). This avoids loading the full PDF into memory at once and prevents context crashes.

```python
from PyPDF2 import PdfReader, PdfWriter
import os

def split_pdf(input_path, split_dir, pages_per_chunk=4):
    os.makedirs(split_dir, exist_ok=True)
    reader = PdfReader(input_path)
    total = len(reader.pages)
    prefix = os.path.splitext(os.path.basename(input_path))[0]
    chunks = []
    for start in range(0, total, pages_per_chunk):
        end = min(start + pages_per_chunk, total)
        writer = PdfWriter()
        for i in range(start, end):
            writer.add_page(reader.pages[i])
        out_path = os.path.join(split_dir, f"{prefix}_pp{start+1}-{end}.pdf")
        with open(out_path, "wb") as f:
            writer.write(f)
        chunks.append(out_path)
    return total, chunks
```

## Step 3: Extract text from each split and write to .md

Extract text from each split PDF in order, appending to the output `.md` file. Do this in a single Python script — do NOT use the Read tool on splits (too slow for large documents).

```python
from PyPDF2 import PdfReader
import os, glob

def extract_all(split_dir, output_path, intro_text):
    splits = sorted(glob.glob(os.path.join(split_dir, "*.pdf")))
    with open(output_path, "w", encoding="utf-8") as out:
        out.write(intro_text + "\n\n---\n\n")
        for split in splits:
            reader = PdfReader(split)
            for page in reader.pages:
                text = page.extract_text()
                if text:
                    out.write(text)
                    out.write("\n\n")
```

## Step 4: Write the intro header

Before writing extracted text, prepend a header block to the `.md` file. This header should include:

1. **Document name** — the filename of the source PDF
2. **Source path** — full path to the original PDF
3. **Page count** — total pages in the PDF
4. **Date converted** — today's date
5. **Brief description** — 2–4 sentences you write summarizing what this document is, based on its filename and any prior context the user has provided. Focus on: what dataset it describes, the time period, the unit of observation, and anything notable about the variable structure (e.g., wave suffixes, grid questions, profile variables).
6. **Usage note** — remind yourself (in future sessions) how to query this file: use Grep with `output_mode: content` and `context` lines to retrieve variable definitions.

Format the header as:

```markdown
# [Document name]

**Source**: [full path to original PDF]
**Pages**: [N]
**Converted**: [YYYY-MM-DD]

## About this document

[2–4 sentence description you write]

## How to use

Search with Grep on this file using `output_mode: "content"` with `context: 5–10` lines to retrieve variable definitions and response options. Variable names are typically followed by their label and wave availability on the next line.

---
```

## Step 5: Delete all split files and the temp directory

After extraction is complete, delete the entire `_splits_tmp/` subdirectory.

```python
import shutil
shutil.rmtree(split_dir)
```

## Step 6: Report completion

Tell the user:
- Path to the output `.md` file
- Total pages processed
- File size of the output

## Output convention

| Input | Output |
|-------|--------|
| `data/raw/bes/panel/Bes_wave30Documentationv30.1.pdf` | `data/raw/bes/panel/Bes_wave30Documentationv30.1.md` |
| `data/raw/some/Codebook.pdf` | `data/raw/some/Codebook.md` |

The `.md` file is always written to the **same directory as the source PDF**, with the same base name.

## Important constraints

- **Never read the full PDF directly.** Always split first, then extract via Python script.
- **Never interpret or summarize** variable content — copy text verbatim.
- **Always clean up** the `_splits_tmp/` directory when done.
- **The original PDF is never modified or deleted.**
