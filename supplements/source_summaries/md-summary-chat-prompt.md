# Comprehensive Source Summary Prompt

Produces one comprehensive markdown summary per source PDF, so later work can read clean text without reopening the original. Book-length sources use `md-book-summary-chat-prompt.md`. Pasted into a chat, this file is self-contained.

## 1. Stance: summaries are clean references

A summary is a reference artifact and stays strictly descriptive. State what the source says and found, not whether it helps or hurts any argument, hypothesis, or proposal. Relevance is inferred later, during synthesis, and is not written into the summary. A downstream reader must be able to trust the summary as a neutral mirror of the source.

- No relevance or applicability section. If a relevance read is requested, it is a separate synthesis file.
- If you catch yourself writing "this supports", "this matters because", or "this fills the gap in", stop. That is synthesis.
- Where the source disagrees with another source in the set, record the source's own position and adjudicate nothing.

## 2. Reading protocol

- Work only from the source. No web search, no outside knowledge. Content claims come from the source text.
- Bibliographic details come from the document itself (title page, headers, copyright line), not PDF metadata. If a title, author, or publication detail cannot be confirmed from the document, ask. With no person available, record the field as unconfirmed in the citation block. A guessed field is a wrong output.
- Text with page markers is the primary input. A PDF with no text layer is read page by page as images.
- Large texts (over 2,000 lines): read the table of contents and structure first, then the sections that carry the substance. Do not pull the whole text into context.
- Process one source at a time, one summary file per source.

## 3. File naming

First author surname (organization for institutional documents), year, a two to four word slug: `defronzo-1981-insulin-sodium-retention.md`. Meeting transcripts keep the source's dated name plus `_summary.md`.

## 4. Structure (in order)

1. **Title line:** `# Author Year - Title`, then `## Comprehensive Technical Summary`.
2. **Citation:** full APA 7 adapted to the source type (article, book, chapter, report), ending in the DOI link where one exists or a stable URL where not, then a separate `Publisher URL:` line.
3. **Metadata:** study type, institution, publication date, open-access status.
4. **Abstract:** laconic, in the register of a package docstring. What the source establishes, over what scope, by what method, in as few lines as the content allows. Neutral.
5. **Body:** the fixed subsections below by default. Where the source's own structure carries the meaning better, follow it. Either way: descriptive, complete, neutral.
   - Background and objective
   - Methods and scope
   - Key findings: the source's actual figures, dates, units, and specifics, not the general direction. Reproduce tables that are the finding.
   - Limitations
   - Topic mapping: sections touched, stated neutrally.

## 5. Calibration by source type

- Target 200 to 350 lines, scaled to complexity.
- Agency strategy documents: timelines, milestones, capability gaps. Skip organizational process.
- Gap-assessment and technology reports: assessment levels, specific gaps, recommendations. Reproduce findings tables.
- Peer-reviewed papers: full methods, results, and discussion.
- Student and conference papers: 150 to 200 lines, centered on the data reported.
- Documents over 300 pages: summarize the structure, go deep on the chapters carrying the substance, and list which chapters were read in full and which skimmed.

## 6. Meeting transcripts

A distinct genre. Stay descriptive and reorganize:

- Header: participants, date and time, duration, stated purpose.
- Executive abstract: neutral synopsis of what the meeting covered and decided.
- Body by topic, using the transcript's own outline where present.
- Decisions and action items: a table of item, owner, status. This is the transcript's key findings.
- Carry-forward: open threads and stated next steps, descriptive.
- Omit personal, off-topic, and sensitive material, and note that it was excluded.

## 7. Figures

Work mainly from the text. When the body attributes a quantitative finding to a figure, read the figure too. Prose compresses a distribution into a range and drops the conditions, per-series spread, and axis definitions: a text citing Fig. 9 for 70 to 95 percent, where the figure plots 77 to 95 percent against a denominator the prose never names. Use an Opus agent for visual reading; availability depends on the plan, so check the original prompt for whether Opus is allowed, or ask. Sonnet agents handle everything else. Fold each figure read into the summary itself. A figure that cannot be read is marked unread at that point, never inferred from surrounding text.

## 8. Include and exclude

Include: specific numbers, dates, targets, power or performance figures; reusable tables; key definitions and findings with page numbers. Exclude: acknowledgments, funding disclaimers, author bios, methodology for analyses the project does not use, speculation beyond the source's evidence.

## 9. Prose and copyright

- Original wording throughout. Numbers, statistics, and findings are reported directly, since data is not copyrightable. No large verbatim block of the source's prose; a short quote is rare, under fifteen words, used at most once per source. Cite specific claims as (Author, Year, p. N).
- Performative epistemics governs the summary's own prose (`supplements/performative-epistemics.md` where the repo carries it). Cut on sight: sentences whose only content is that the summary is careful or candid; a finding restated in a "which is to say" clause; headers that tell the reader how to feel; staging of the argument ("note that", "it is worth observing"); narration before and after a table; references to earlier drafts; statements of what was preserved, retained, or avoided. State the result. Test: delete the sentence; if only word count goes, it was decorative.
- Paragraphs for narrative, bullets for enumerated findings. Straight quotes, no emojis, the source's units and precision.

## 10. Template

```
# [Author] [Year] - [Short Title]
## Comprehensive Technical Summary

## Citation

[APA 7 reference, ending in https://doi.org/10.xxxx or a stable URL]

Publisher URL: [https://...]

## Metadata

Study type · Institution · Publication date · Open-access status

---

## Abstract

[Laconic, docstring-register synopsis. Neutral. A few lines.]

---

### Background and objective
[...]

### Methods and scope
[...]

### Key findings
[Actual figures, results, specifics.]

### Limitations
[...]

### Topic mapping
[Section(s) touched, stated neutrally.]
```
