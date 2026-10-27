# Test-Driven Documentation (TDD)
## A Method for Technical Document Drafting

---

## Synopsis

**Claude excels at software engineering, so treat your writing project like an Agile software project which defines acceptance tests first, then builds content that passes the tests.**

---

## The Process (3 Prompts)

Each prompt can be run in a separate chat. Attach the output of each step as input to the next.

### Prompt 1: Generate Test Suite

```
I need to write a [DOCUMENT TYPE] about [TOPIC].

Requirements:
- [Audience]
- [Purpose]
- [Constraints: length, format, deadline]
- [Questions it must answer]
- [Source materials]

Create a comprehensive test suite (around 150 tests; use your 
engineering judgment based on document complexity) which this 
document must pass.

Organize tests by scope:

WHOLE-DOCUMENT TESTS
- Requirements that apply to the complete document
- Examples: "All sections use consistent terminology," "Executive abstract should be concise," "No contradictory claims between sections"
- These are validated once, against the finished draft

SECTION-LEVEL TESTS  
- Requirements specific to each major section
- Examples: "Section 3 states the selection criteria explicitly," 
  "Section 6 includes the calculation showing 78%"
- These are validated while drafting each section

REQUIRED:
- Thesis statement in the test plan. This can be provided by the user or synthesized by you from a preponderance of the requirements. 

For each test, specify:
- What is being tested
- Pass/fail criteria  
- Evidence required (if applicable)

```

### Prompt 2: Generate Outline

```
Generate a paragraph-level outline which, if followed correctly, will 
result in a document that passes all of the tests in the attached 
TDD test suite.

Each paragraph in the outline should be represented by its topic 
sentence: the claim that paragraph will make. It's often easier to 
just write the sentence than to expend effort crafting a terse 
bullet point that you'll later have to re-expand anyway.

What Is a Topic Sentence?
"A topic sentence introduces the main idea of a paragraph and supports the main thesis of a paper or essay. The topic sentence is typically the first sentence of a paragraph, and it introduces the subject of a paragraph by laying out a controlling idea or particular point of view on the topic. The rest of the paragraph will include supporting sentences with examples and evidence to support the main point." (source: https://www.masterclass.com/articles/topic-sentence-guide)

[Attach: TDD test suite from Prompt 1]
```

### Prompt 3: Write the Document

```
Write the document using the attached outline. For each paragraph, 
a topic sentence is provided. Develop it into a complete paragraph.

[Attach: Outline from Prompt 2]
[Attach: Any reference materials needed]
```

---

## Key Principles

1. **Tests before content.**

2. **Organize by location, tag by severity.** When drafting Section 3, you need Section 3's tests, and you check to make sure they support the overall thesis.

3. **Topic sentences, not headings.** "Introduction" tells you nothing. "This report demonstrates that X is feasible because Y" tells you exactly what to write.

4. **Trace everything.** Every test maps to content. Every claim maps to evidence. Orphans indicate gaps. Careful traceability can help detect hallucinations or temperature drifts.

5. **Validate at outline stage.** Running the tests is cheap.

6. **Audit tests on architecture migration.** When the system under test changes structure, every existing test that opens or references the old structure must be audited in the same step. We want to avoid missing tests that were broken by the migration and not fixed.

7. **Verify test sources against primary materials.** Any test that cites a source document (work order, SOW, RFP, customer requirement, specification, peer-reviewed paper) must be verified against that source before the test suite is accepted as the contract. Does the source say what the test claims it says? A test that attributes a requirement to a numbered SOW clause must be checked against the SOW to confirm the clause exists and contains the claimed language. A test that attributes a number to a paper must be checked against the paper. This rule exists because LLMs confidently fabricate plausible-sounding source attributions. The verification burden is on the agent writing or modifying the test. However, reviewers of the test suite must also verify any source claims they encounter that they have not personally checked. See 8 for how the deliverable must have facts mapped back to tests as well.

8. **Every claim of fact gets a test linked to a primary source.** Any factual claim in the document (a number, a measurement, a regulatory threshold, a personnel commitment, a cited result) should have a corresponding test in the suite, and that test should name the primary source it will be validated against. The source can be a paper, a regulation, a budget spreadsheet, a letter of commitment, meeting minutes, or a markdown summary of any of these. If the source is on disk or retrievable, the test is verifiable. If the claim has a test and the test maps to an accessible source, verification is straightforward and the claim is low-risk. If the claim has no test and no source, or is simply referencing an abstract or pubmed metadata, it is high-risk. High-risk claims can either be LLM confabulation (plausible-sounding details generated because specificity correlates with factual content in training data) or a human assertion that was inserted without sourcing and never revisited. Both failure modes produce the same result: a specific, plausible, internally consistent claim that no one can trace to reality. Analytical conclusions, design rationale, and argumentative framing are the author's territory. This principle (No. 8) targets empirical claims: numbers, dates, measurements, regulatory classifications, personnel hours, cited results.

9. **Every detector and numeric gate runs a zero control and a known-bad control before its output is trusted.** The zero control is a known-clean case that must read zero. The known-bad control is a synthetic case with a planted defect that must fire. Record both results next to the gate's output. The rule covers an AI-tell grep, a fact-check pass and an echo-site audit as much as a geometry probe. A gate that was never shown to fire passes everything. Its green result carries no information. A count with no zero control cannot be told from noise, so a nonzero reading may be the detector's own floor.

10. **Run the suite red first.** Run every test on the untouched baseline before building anything. A test that passes at baseline is labelled an invariant, and gates that cannot fail are replaced.

---

## On Test Count

"Around 150" is a starting point. Use engineering judgment:

- A short article might need 30 tests
- A complex regulatory submission might need 400+
- Software projects routinely use thousands of tests

The right number is whatever comprehensively covers your deliverable's requirements without inventing tests for their own sake. Avoid creating 5 tests for this document and 10 for this document. The true deliverable of the [day, hour, whatever] is what the test plan should be written to, and it should be dedicated to the support of that single deliverable.

---

## The Revision Pass (Optional Prompt 4)

After the document passes its test suite, a dedicated editing pass can improve prose quality without changing technical content. This addresses a specific LLM failure mode: text that passes all technical tests but reads like statistical average prose (correct but lifeless). Using the analogy to Agile software development, we have the tests green, now we need to go back and add the "elegance." 

The revision pass operates under strict constraints:
- **Subtract or replace whenever possible.** Replacements should be shorter than originals.
- **Preserve all technical meaning.** Technical terms and precise jargon survive. Decorative language and performative candor/epistemics do not.
- **Test suite remains the contract.** Edits cannot introduce new test failures. Run the suite again after revision.

This pass runs after content is frozen, so there is no risk of scope creep. A reviewer familiar with the domain (not just consistency checking, but evaluating whether a cognizant reader can follow the technical design intent) should verify that edits did not break the document's communication architecture.

Prompt 4:
```
Do a review/editing pass on the document. Using an analogy to Agile software development, we have the tests green, now we need to go back and add the "elegance." 

The revision/editing pass should operate under strict constraints:
- **Subtract or replace whenever possible.** Replacements should be shorter than originals.
- **Preserve all technical meaning.** Technical terms and precise jargon survive. Decorative language and performative candor/epistemics do not.
- **Test suite remains the contract.** Edits cannot introduce new test failures. Run the suite again after revision.

This pass runs after content is frozen, so no scope creep. After editing the copy and running the tests, imagine you are a reviewer familiar with the domain (not just consistency checking, but evaluating whether a cognizant reader can follow the technical design intent) and verify that edits did not break the document's communication architecture.
```
