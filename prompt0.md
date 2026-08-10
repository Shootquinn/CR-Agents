# Collaborative Reasoning Session -- Environment Setup (prompt0)

**Run this prompt at session start and after any compaction event**

## CRITICAL: You are on Windows via Git Bash (MINGW64)

You are running inside Claude Code on a Windows machine. Your bash shell is Git Bash (MINGW64), NOT WSL.

**Key implications:**
- Windows drives are mounted at `/c/`, `/d/`, etc. — NOT `/mnt/c/`.
- Windows-installed tools (node, pandoc, python) should all be on PATH if installed correctly. Run them directly.
- For LibreOffice console output, use `soffice.com` (not `soffice.exe`, which launches the GUI).
- **Run all checks ONE AT A TIME. Do not fire them in parallel.** If one fails, continue to the next.

### Windows Encoding (cp1252 errors)

Windows consoles default to cp1252 encoding, which cannot display Unicode characters like `≈` (U+2248), `°` (degree), or smart quotes. Python scripts that print such characters will fail with `UnicodeEncodeError: 'charmap' codec can't encode character`.

**Fix: Set UTF-8 encoding before running Python scripts that may output Unicode.**

```bash
# In Git Bash (MINGW64) — use inline environment variable
PYTHONIOENCODING=utf-8 python script.py

# Or reconfigure stdout inside the script
import sys
sys.stdout.reconfigure(encoding='utf-8')
```

**Common triggers:**
- Reading xlsx files with openpyxl (cells may contain `≈`, `°`, `±`, em dashes, smart quotes)
- Running validate.py from claude-docx-bundle (XML contains Unicode punctuation)
- Any script that prints non-ASCII characters

**PowerShell equivalent:**
```powershell
$env:PYTHONIOENCODING = 'utf-8'; python script.py
```

Do not use `set PYTHONIOENCODING=utf-8` in bash — bash `set` does not export to child processes.

---

## 1. Locate the Document Toolkit (Optional)

If the session work order, change order, user request, or gameplan involves producing `.docx` or `.pdf` deliverables, locate the document toolkit:

1. Search for a folder named `claude-docx-bundle` (with or without hyphens/underscores between words). Search the current directory, one level down, and up to **two levels up** from the working directory. No further.
2. If found, read `docx_SKILL.md` and `PDF_SKILL.md` from the bundle before creating or modifying any documents.
3. If not found, check for `claude-docx-bundle.zip` nearby and extract it with Python's zipfile module.
4. If the bundle is not found and the gameplan does not require document production, skip this step entirely — it is not blocking.

**Document production rules (when toolkit is active):**
- **Creating docx files:** Use Node.js with the docx library (docx-js). Always validate output with validate.py from the bundle.
- **Reading docx files:** Use `pandoc filename.docx -t plain` for text extraction.
- **Reading PDF files:** Use Python with pypdf or pdfplumber.
- **Converting docx to PDF:** Use `soffice.com --headless --convert-to pdf filename.docx`
- **Never use python-docx** for creating documents.
- **US Letter** (8.5 x 11), Times New Roman 12pt unless told otherwise.
- **Use style guide**: `claude-docx-bundle/docx_style.md` is provided and usage is mandatory. Should be prescribed in gameplans for document creation as part of the final docx generation step. 

---

## 2. Self-Check

Run these checks sequentially and report results as a table. Categorize each as PASS, FAIL, or SKIP (if not needed for this session).

**Core (must pass):**
1. `python --version` — Python installed
2. Operational guide found and readable
3. TDD method document found and readable

**For document production (must pass if gameplan requires .docx/.pdf):**
4. `node --version` — Node.js installed
5. `node -e "const d = require('docx'); console.log('OK');"` — docx-js available
6. `pandoc --version` — Pandoc installed
7. `soffice.com --version` — LibreOffice installed
8. Document toolkit (`claude-docx-bundle`) found

**Nice to have (warn if missing, don't block):**
9. `python -c "import pypdf; print('OK')"` — pypdf available
10. `python -c "import pdfplumber; print('OK')"` — pdfplumber for table extraction

If a tool is not found, try the full Git Bash path as a fallback (e.g. `"/c/Program Files/LibreOffice/program/soffice.com" --version`).