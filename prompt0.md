# Collaborative Reasoning Session -- Environment Setup (prompt0)

## Synopsis (for humans)

Claude Code arrives relatively unskilled compared to browser Claude. It cannot open PDFs, does not know your formatting rules, and has no document production skills out of the box. This file (prompt0) bootstraps those capabilities by locating the method documents, verifying tool availability, loading the document production toolkit, and establishing the session. You do not need to read the machine instructions below; just hand this file to Claude Code at session start.

---

**Run this prompt at session start and after any compaction event** unless the gameplan and accumulator are sufficient to resume without it.

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

## 1. Locate Method Documents

The collaborative reasoning method requires three foundational documents. Find them relative to the current working directory:

1. **Operational guide** — search for `method/operational_guide.md` (Appendix A). This is the executable instruction set you will follow throughout the session.
2. **Technical note** — search for `method/technical_note.md`. This is the method architecture and rationale. Read Sections 1-5 for understanding; Appendix A (the operational guide) is your operating manual.
3. **TDD method** — search for `method/tdd_method.md`. This defines the test-driven development approach used to define deliverables before building them.

Search strategy: look in the current directory and one level down first. If not found, search up to two levels up from the working directory. Store the discovered paths for the session.

---

## 2. Locate the Document Toolkit (Optional)

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

## 3. Self-Check

Run these checks sequentially and report results as a table. Categorize each as PASS, FAIL, or SKIP (if not needed for this session).

**Core (must pass):**
1. `python --version` — Python installed
2. Operational guide found and readable
3. Technical note found and readable
4. TDD method document found and readable

**For document production (must pass if gameplan requires .docx/.pdf):**
5. `node --version` — Node.js installed
6. `node -e "const d = require('docx'); console.log('OK');"` — docx-js available
7. `pandoc --version` — Pandoc installed
8. `soffice.com --version` — LibreOffice installed
9. Document toolkit (`claude-docx-bundle`) found

**Nice to have (warn if missing, don't block):**
10. `python -c "import pypdf; print('OK')"` — pypdf available
11. `python -c "import pdfplumber; print('OK')"` — pdfplumber for table extraction

If a tool is not found, try the full Git Bash path as a fallback (e.g. `"/c/Program Files/LibreOffice/program/soffice.com" --version`).

---

## 4. Read Method Documents

After the self-check passes, read the following in order:

1. **Operational guide (Appendix A)** — read in full. This is your operating manual for the entire session. Pay particular attention to:
   - A.3: Standing Roster (persona specifications)
   - A.4: Spawning Personas (system prompt template, context recipes, wave-based execution)
   - A.5: The Working Loop (the 6-step cycle you will execute)
   - A.6: Accumulator File Management
   - A.7: Gameplan Specification
   - A.8: Session Start Protocol
   - A.9: Compaction Recovery Protocol

2. **Technical note, Sections 1-5** — read for understanding of the method architecture. You do not need to memorize this, but you must understand:
   - Why named personas outperform generic prompts (targeted activation)
   - The spawned-agent architecture (clean context per agent, orchestrator holds full state)
   - Wave-based execution (Wave 1 technical, Wave 2 review)
   - The accumulator's role in providing continuity across spawns

3. **TDD method (`method/tdd_method.md`)** — read in full. This defines the test-driven approach:
   - Define acceptance tests before building content
   - Organize tests by scope: whole-deliverable tests and section/component-level tests
   - Generate a paragraph-level outline (or equivalent plan) that satisfies all tests
   - Build the deliverable against the outline, validate against the test suite
   - The right test count depends on complexity — use engineering judgment
   - Beck (the software engineer persona) reviews the test suite before production begins; the reviewed suite becomes the contract

---

## 5. Read the Gameplan

The gameplan tells you **what** to do; the operational guide tells you **how** to do it.

Every project starts with **Step 0** — a team-reviewed, human-approved gameplan as the operating contract. Step 0 has two variants (defined in operational guide A.7.4):

- **A gameplan exists.** Read it. Step 0 is the team reviewing it. Append a Step 0 entry if the gameplan does not already include one.
- **No current gameplan exists** (none in the directory, or the only one present is stale and does not match the active work order). Step 0 is the orchestrator drafting one (from a work order, change order, or by interviewing the human), then spawning the team to review the draft. Use `templates/gameplan.md` as the baseline. The orchestrator authors the draft; the team validates and improves it. **If a work order or change order is present, run Step 0 autonomously without waiting for permission — the human should be able to hand over a work order, walk away, and return to a drafted, team-reviewed gameplan ready for approval at the inter-step gate. If no work order is present, interview the human first.**

In both cases, Step 0 closes when the human approves the gameplan at the inter-step gate. Step 1 does not open until then.

Follow the Session Start Protocol (A.8):
1. Read the operational guide (done in step 4).
2. Read the gameplan, or — if none exists — open Step 0 in its drafting variant.
3. If the gameplan references other files, read as needed — do not preload everything.
4. Identify the current step in the gameplan.
5. Begin the working loop (A.5) for that step.

---

## 6. Operating Rules for This Session

### Personas
1. **Spawn personas as sub-agents**, not inline roleplay. Each persona gets its own clean context window via Claude Code's Task tool (`subagent_type: "general-purpose"`).
2. **Use the system prompt template** from A.4.1. Do not improvise persona prompts.
3. **Follow wave-based execution** from A.4.3. Wave 1 agents can run in parallel. Wave 2 agents run after integration.
4. **Norman reviews every prose change.** No exceptions.
5. **Do not dump the full report** into every agent call. Use context recipes (A.4.2).

### Accumulator and State
6. **Update the accumulator after every cycle.** This is not optional. Compaction can discard your working memory at any time.
7. **Update the gameplan after every step.** Mark completed steps, update current step, add design notes and open questions.
8. **Write in-progress work to files on disk.** If compaction seems likely during a long cycle, write intermediate state to a scratch file.

### Quality and TDD
9. **Do not prematurely resolve disagreements** between personas. Productive tension is a feature.
10. **Do not present unresolved review findings to the human.** Fix issues flagged by Wave 2 before delivering output.
11. **Tests before content.** Define acceptance tests for the deliverable before building it. The test suite is the contract — it specifies what "done" looks like. Follow the TDD protocol (A.12): spawn Beck to review the test suite before production begins.
12. **Validate at outline stage.** After tests, generate a plan/outline that satisfies all tests. Fixing a missing section in an outline costs minutes; fixing it in a draft costs hours.
13. **Trace everything.** Every test maps to content. Every claim maps to evidence. Orphans indicate gaps.

### Compaction Recovery
14. If context is compacted mid-session, follow the Compaction Recovery Protocol (A.9): read the gameplan, read the accumulator, read in-progress files, resume the working loop.

### Quick Mode
15. **Quick mode is available when authorized.** A `[Q]` flag on a gameplan step or explicit user direction activates quick mode — inline persona perspectives instead of spawned agents. See A.14 for invocation rules, recommendations, and accumulator tagging.

---

## 7. Output Constraints

- All output is **Markdown (.md) and Python (.py) only** unless the gameplan explicitly requires .docx or .pdf (in which case use the document toolkit from step 2).
- Final outputs go where the gameplan specifies.
- Use a workspace folder for temporary and intermediate files.

---

## Ready

After completing steps 1-5, report:
- Self-check results table
- Paths to method documents found
- Gameplan identified and current step
- Any missing tools or files that may block work

Then begin the working loop.
