# Comprehensive Book Summary — Pipeline Spec

Produces one comprehensive markdown summary per book-length source PDF.

## Roles

| Role | Model | Count |
|---|---|---|
| Orchestrator | — | one |
| Chapter agent | Sonnet | one per chapter |
| Visual agent | Opus | one per visual page cluster |
| Merge agent | Sonnet | one |

## 0. Source constraints

Work only from the source PDF. No web search, no outside sources, no filling gaps from background knowledge about the book.

Bibliographic details come from the document itself, not from PDF metadata, which is frequently wrong or corrupt. 

## 1. File naming

First author surname, publication year, a two to four word slug. Example: `trimble-2011-writing-with-style.md`.

## 2. Comprehensiveness and length

Comprehensive means every chapter gets a subsection, every named rule or test the book states gets recorded, every worked example gets reproduced and explained, every figure and table gets read and reported, and every term the book defines gets its definition. Nothing is dropped for being minor or for making the document long. If a chapter contains eleven tips, the summary contains eleven tips.

Expect 1,250 to 3,500 lines, but use your engineering judgement to set the length of the comprehensive summary based on the complexity and length of the material.

Completeness test: a reader consulting this document a year on, without the book at hand, should not need to return to the book for anything but pleasure.

## 3. Structure

In this order:

1. **Citation block.** Full APA 7 reference for the book or edition, ending in a DOI link where one exists, otherwise a stable publisher URL, followed by a separate Publisher URL line. Adapt APA to the source type: whole book, edited volume, chapter, translation, later edition.
2. **Abstract.** Laconic, in the register of a package docstring. What the book establishes, over what scope, by what method. A few lines.
3. **Architecture.** How the book is built: its parts, its chapter count, the sequence of the argument, where the load-bearing chapters sit. The map the rest of the document hangs from. Half a page.
4. **Chapter-by-chapter treatment.** The bulk of the document, one subsection per chapter, in book order. Shape defined in section 4.
5. **Cross-cutting synthesis.** The book's recurring moves collected out of chapter order: techniques it returns to, terms it defines and reuses, through-lines that a linear read leaves scattered for the reader to reassemble. This section justifies the document's length, and it is the part unavailable from the table of contents.
6. **Scope and limitations.** What the book covers, what it explicitly declines to cover, what its advice assumes about its reader, discipline, or era. The book's own boundaries, not complaints about them.
7. **Topic mapping.** Sections touched, stated neutrally.

## 4. Chapter subsections

Every chapter takes the same internal shape, so subsections stay scannable and comparable:

- **Claim.** What the chapter argues, in one to three sentences.
- **Mechanism.** How the chapter says to do the thing: the procedure, test, or heuristic. Where the chapter gives a rule, the rule appears in full, including its stated exceptions.
- **Evidence and examples.** What the chapter offers in support: studies, before/after demonstrations, worked cases, figures, tables. Each example reproduced in full per section 6, then what it demonstrates and why the move works. Every example in the chapter, not a representative sample.
- **Terms introduced.** Vocabulary the book coins or repurposes, with the book's definition.
- **Connections.** Which chapters this one depends on or feeds, by number.

Chapters vary in weight. A chapter carrying the central argument may run 80 lines; a short transitional chapter may run 15.

## 5. Preserve the operable content

These are instructional books. The transferable content is the rules, tests, and procedures. "Schimel argues that structure matters more than word choice" summarizes a position. "Schimel's test: cut the paper to its opening and closing paragraphs and check whether the argument still resolves" summarizes something usable.

Where the book gives a procedure, the procedure gets captured. Where it gives a diagnostic question, the question gets captured. Where it gives a numbered list of moves, the list stays numbered and complete.

## 6. Quoting

Summaries are private reading notes from a purchased copy, not a publication. Quote whatever serves them. A summary that will be committed to a shared or public repository takes the article prompt's section 9 limits instead: original wording, quotes under fifteen words, none reproduced at example length.

- **Reproduce example passages verbatim.** Craft books are built out of before/after pairs, specimen paragraphs, and worked revisions. Paraphrase destroys them; the point of a before/after pair is the exact wording that changed. Both halves in full, then what the revision did.
- Quote the author's own formulation of a rule, test, or definition where the phrasing is what makes it precise or memorable. A rule restated loosely is a rule someone has to reconstruct later.
- Blockquote anything over a line or two, with a page number, so a quotation is never mistaken for the note-taker's prose.
- Attribute epigraphs and second-hand quotations to their original speaker as well as to the page they appear on. Craft books quote other writers heavily; an epigraph from another author must not end up filed as a line of the book's own author.
- Where quoting and describing both fit, do both: the passage, then what it demonstrates. The description is the part that will not be in the book when someone goes looking for it.

## 7. Neutrality

Descriptive throughout. State what the book claims, establishes, and recommends — not whether it is right, not whether it supports any position the owner may later take. Where two books in the same batch disagree, record both positions in their own summaries without adjudicating.

## 8. Prose constraints

The summary's own prose, not only the books it describes, is held to the Editor's AI-writing list and its performative-epistemics failures:

- **Self-narrating honesty.** No sentence whose only content is that the document is being careful, thorough, or candid.
- **Restated conclusions.** State a finding once. No "which is to say" clause re-delivering it.
- **Throat-clearing headers.** A header names its section; it does not tell the reader how to feel about what follows.
- **Meta-commentary.** No staging the argument: no "note that," no "it is worth observing that," no directing which part to attack first.
- **Table sandwiching.** A table stands alone, without narration before and after it.
- **Changelog leakage.** No references to earlier drafts, to initial impressions, or to what got revised.

## 9. Mechanics

- Summarize one book start to finish before the next begins.
- Sequential chunk reads, not sampling. A chapter-by-chapter document requires every chapter to have been read.
- Reconcile the table of contents against actual chapter breaks before drafting. Scanned and OCR'd books carry front matter that shifts the printed-to-PDF page offset.
- Each summary is written to its own file.

## 10. Division of labor

| Step | Role | Work |
|---|---|---|
| 1. Inventory | Orchestrator | Page map, chapter ranges, offset, visual-content list |
| 2. Text pass | Chapter agents | Chapter subsections |
| 3. Visual pass | Visual agents | Figures, tables, marked-up examples, OCR repair |
| 4. Merge | Merge agent | Assembly, ordering, page-ref normalization, gap report |
| 5. Synthesis | Orchestrator | Architecture and cross-cutting synthesis |

**Step 1: Inventory.** Before dispatch, walk the whole PDF and build a page map: chapter start and end pages, the printed-to-PDF offset, and every page carrying visual content. A page extracting empty or near-empty mid-chapter is an image page. So is any page the text layer references as a figure, table, box, or exhibit. This list is an explicit artifact; it drives steps 2 and 3.

**Step 2: Text pass.** One chapter agent per chapter, given that chapter's page range. Each returns its chapter subsection in the section 4 shape, examples quoted in full, page numbers on every quote. Chapters are independent, so the pass runs fully parallel.

**Step 3: Visual pass.** One visual agent per flagged page cluster, reading those pages as images rather than through extracted text. Visual agents take anything where layout rather than wording carries the meaning:

- Figures, charts, diagrams: what the figure shows, including axis labels, units, and any value the surrounding text omits.
- Tables: transcribed in full as markdown. A table is never summarized.
- Marked-up revision examples: strikethroughs, carets, insertions, editing marks. These rarely survive text extraction; on a scanned book they arrive as garbage or vanish. They are also what a craft book is built from.
- Typographic demonstrations: indentation, spacing, line breaks, emphasis used as the example itself.
- Boxed sidebars and pull quotes, which extraction splices into the middle of body paragraphs.
- OCR repair: where a scanned page's text layer is too damaged to trust, the visual agent supplies the correct text.

**Step 4: Merge.** The merge agent receives every chapter subsection from step 2 and every visual result from step 3, and assembles the document:

- Fold each visual result into its chapter at the point the chapter refers to it, never appending figures at the end.
- Order chapters by book order and normalize their internal shape, so every subsection carries the same five elements in the same sequence.
- Normalize page references to printed page numbers using the step 1 offset. Chapter agents return a mix of printed and PDF numbering.
- Reconcile duplicates. Adjacent chapters often approach one rule from different angles; keep both treatments and make them point at each other rather than silently repeat.
- Return a gap report beside the merged draft: thin chapters, examples referenced but never reproduced, step 1 figures no visual result covers, quotes missing page numbers. This is the punch list for a second pass.

**Step 5: Synthesis.** The orchestrator reads the merged draft and writes the architecture and cross-cutting synthesis sections onto it. Both depend on holding the whole book at once and are not delegated. The gap report is cleared before the document is called done.

## 11. Template

```
# [Short Title]

## Citation

[Author, A. A. (Year). *Title of book* (Nth ed.). Publisher. https://doi.org/10.xxxx]

Publisher URL: [https://...]

## Abstract

[Laconic synopsis, a few lines.]

## Architecture

[Parts, chapter count, sequence of the argument, location of the load-bearing chapters.]

## Chapters

### Chapter 1 — [Title]

**Claim.** [...]

**Mechanism.** [...]

**Evidence and examples.** [...]

**Terms introduced.** [...]

**Connections.** [...]

### Chapter 2 — [Title]

[...same shape...]

## Cross-cutting synthesis

### [Recurring move or theme]

[...]

## Scope and limitations

[...]

## Topic mapping

[Sections touched, stated neutrally.]
```
