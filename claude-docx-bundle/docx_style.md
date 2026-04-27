## User Style Override Detection

When the document toolkit activates, check for user-provided style files before applying the default style below.

**Detection procedure:**
1. Search the project root directory and the claude-docx-bundle directory for any `.docx` file whose name contains the word "style" (case-insensitive).
2. Use word-boundary matching, not substring matching. "USE THIS STYLE.docx" and "project_style.docx" match. "styleside chevy essay.docx" does not, because "style" appears as part of the compound word "styleside," not as a standalone word.
3. If a clear match is found (the filename unambiguously indicates a style document), use it as the formatting reference instead of the default style below.
4. If the match is ambiguous (the word "style" could be incidental to the document's purpose), ask the user whether they intended it as a style reference before applying it.
5. If no style file is found, use the default Unleashed Robotics style defined in the rest of this document.

**Recommended convention for users:** Place a .docx file in the project root with "STYLE" in the name (e.g., "USE THIS STYLE.docx"). The toolkit will detect it automatically.

---

This section provides instructions for Claude. Read this section before generating any docx files.

Always your docx skill before creating documents.

  Tips for next time:

  1. Single tool discovery command:
  powershell -Command "Get-Command pandoc, python, node, npm -EA Silent"
  2. On Windows, assume Word exists - Skip straight to COM automation since most Windows machines have Office installed
  3. Check for Word first on Windows:
  Test-Path 'C:\Program Files\Microsoft Office\*\WINWORD.EXE'
  4. Browser Claude uses docx-js - It generates the .docx file directly in JavaScript memory and serves it as a download artifact. No external tools needed. I don't have that capability in CLI mode.
  5. If I had Node.js available, I could install docx package and generate programmatically - similar to browser Claude but from CLI.

  The core lesson: don't hunt for tools sequentially - check what's available in one go, then pick the best option.

### Our Style

The heading hierarchy below differs from docx-js defaults. You must manually apply the correct formatting to each heading rather than relying on built-in HeadingLevel styles. Pay attention to the spacing values, including the 1.08 line spacing.

Tables: Use ShadingType.CLEAR (not SOLID) to prevent black backgrounds. Cell margins are internal padding, not added to cell width. Match columnWidths to cell widths exactly.

### Document Formatting

#### Page Setup

Page size: US Letter (8.5 x 11 inches). Margins: 1 inch on all sides. These translate to docx-js values of width: 12240, height: 15840, and margins: 1440 for each side.

#### Body Text

Font: Times New Roman, 12pt (size: 24 in half-points). Paragraph spacing: 0pt before, 4pt after (before: 0, after: 80 in twips). Line spacing: 1.08 (line: 259 in docx-js units).

Paragraph indent: Each body text paragraph must begin with a tab character. Insert a literal tab (`\t`) as the first character of the first TextRun in each body paragraph. This mimics a human pressing Tab at the start of a new paragraph. Do NOT use paragraph-level indent styles (e.g., `indent: { firstLine: ... }`) because this would also indent non-paragraph "Normal" text elements like captions, subtitles, and other items that should not be indented. The tab character approach is manual and selective --- apply it only to actual body paragraphs, not to headings, captions, figure labels, table cells, or other non-paragraph content.

#### Heading Hierarchy

**Title:** 14pt Bold, Times New Roman. Use for document title only.

**Level 1 Heading:** 12pt Bold Underline, Times New Roman, "Small Caps" font style. Use for major sections (Part 1, Part 2, Appendix X).

**Level 2 Heading:** 12pt Bold, Times New Roman, title case. Use for subsections (1.1, 1.2, 2.1).

**Level 3 Heading:** 12pt Underline, Times New Roman, title case. Use for sub-subsections or named items within a subsection.

**Level 4 Heading:** 12pt Italic, Times New Roman, title case. Use for sub-sub-subsections or named items within a sub-subsection.

No headings below level 4 are defined. If used, they should be normal text with title caste. If numbering headings, include a period after the final number, i.e. "4.2.1. Salmon Burgers"

#### Dashes

Em dashes and en dashes are forbidden. They create inconsistency across platforms and editors, and most uses are better served by other punctuation.

Preferred alternatives: use commas for parenthetical phrases, use hyphens for compound modifiers, use colons to introduce explanations, restructure the sentence if none of these work.

If a dash is absolutely necessary and no alternative exists, use three hyphens (---) to stand in for an em dash and two hyphens (--) to stand in for an en dash.

#### Bold Text

Bold is reserved for headings only. Do not use bold for emphasis within paragraphs.

Do not bold the first phrase of a bullet point simply because it contains a colon. If the sentence has a natural structure of "label: explanation" the label does not automatically deserve bold treatment. Bold for header rows/columns in tables is acceptable and should be used.

#### Italic Text

Italic may be used sparingly for emphasis when it adds clarity and crispness to the sentence. If removing the italic would not change the reader's understanding, remove it.

Acceptable uses: titles of works, a single word that genuinely requires stress for the sentence to parse correctly.

Unacceptable uses: italicizing entire phrases for dramatic effect, using italics as a substitute for clear writing, italicizing anything you think is important.

### Citations

#### In-Text Citations (APA 7)

Basic format: (Author, Year). Example: (Morley, 2021).

With page number: (Author, Year, p. X). Example: (Vella, 2025, p. 12).

Two authors: (Author1 & Author2, Year). Example: (Morley & Bowen, 2022).

Three or more authors: (First Author et al., Year). Example: (Zacny et al., 2019).

Multiple citations: separate with semicolons, alphabetical order. Example: (Annan et al., 2002; Morley, 2021; Vogt, 2006).

Direct quote: include page number and use quotation marks. Example: According to the report, "the system is feasible" (Morley & Bowen, 2022, p. 5).

#### Reference Page (APA 7)

**Journal article:** Author, A. A., & Author, B. B. (Year). Title of article. Title of Periodical, volume(issue), page--page. https://doi.org/xxxxx

**Book:** Author, A. A. (Year). Title of work: Capital letter also for subtitle. Publisher.

**Report:** Author, A. A. (Year). Title of report (Report No. xxx). Publisher. URL

**Website:** Author, A. A. (Year, Month Day). Title of page. Site Name. URL

Format notes: use hanging indent (0.5 inch), double-space entries, alphabetize by first author's last name, italicize titles of longer works (journals, books), do not italicize article titles.
