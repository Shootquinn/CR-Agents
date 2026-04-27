# Test-Driven Documentation (TDD)
## A Method for Technical Document Creation Using Claude

---

## The Method in One Sentence

**Claude excels at software engineering, so treat your writing project like an Agile software project—define acceptance tests first, then build content that passes them.**

This approach plays to Claude's strengths: systematic reasoning, test validation, structured problem decomposition, and maintaining consistency across large contexts. It transforms an ambiguous creative task ("write a report") into a well-specified engineering task ("build something that passes these tests").

---

## Why This Works

Traditional approach: "Write me a report about X."
- LLM guesses at structure, depth, and completeness
- Reviewer finds gaps, requests revisions
- Iterate until acceptable (inefficient)

TDD approach: "Here are 100 tests the report must pass. Create an outline that passes all of them."
- Requirements are explicit before writing begins
- Gaps are identified at outline stage (cheap to fix)
- Final draft is validated against known criteria

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
- [Key questions it must answer]
- [Source materials available]

Create a comprehensive test suite (around 150 tests—use your 
engineering judgment based on document complexity) that this 
document must pass.

Organize tests by scope:

WHOLE-DOCUMENT TESTS
- Requirements that apply to the complete document
- Examples: "All sections use consistent terminology," "Executive summary 
  stands alone," "No contradictory claims between sections"
- These are validated once, against the finished draft

SECTION-LEVEL TESTS  
- Requirements specific to each major section
- Examples: "Section 3 states the selection criteria explicitly," 
  "Section 6 includes the calculation showing 78%"
- These are validated while drafting each section

For each test, specify:
- What is being tested
- Pass/fail criteria  
- Evidence required (if applicable)
```

### Prompt 2: Generate Outline

```
Generate a paragraph-level outline which, if followed correctly, will 
result in a document that passes all of the tests in the attached 
TDD test plan.

Each paragraph in the outline should be represented by its topic 
sentence—the claim that paragraph will make. It's often easier to 
just write the sentence than to expend effort crafting a terse 
bullet point that you'll later have to re-expand anyway.

[Attach: TDD test suite from Prompt 1]
```

### Prompt 3: Write the Document

```
Write the document using the attached outline. For each paragraph, 
a topic sentence is provided—develop it into a complete paragraph.

[Attach: Outline from Prompt 2]
[Attach: Any reference materials needed]
```

---

## Key Principles

1. **Tests before content.** Always.

2. **Organize by location, tag by severity.** When drafting Section 3, you need Section 3's tests—not a mixed list sorted by importance. Severity (critical/major/minor) is useful metadata, but scope is the organizing principle.

3. **Topic sentences, not headings.** "Introduction" tells you nothing. "This report demonstrates that X is feasible because Y" tells you exactly what to write.

4. **Trace everything.** Every test maps to content. Every claim maps to evidence. Orphans indicate gaps.

5. **Validate at outline stage.** Fixing a missing section in an outline costs minutes. Fixing it in a draft costs hours.

6. **Audit tests on architecture migration.** When the system under test changes structure (monolithic script to per-part modules, spreadsheet parameters to file-based params, single assembly to App::Link federation), every existing test that opens or references the old structure must be audited in the same step. A test that references a deleted object is not a pre-existing failure — it is a test that was broken by the migration and not fixed. Beck owns this audit as part of the migration step, not as deferred debt.

7. **Verify test sources against primary materials.** Any test that cites a source document (work order, SOW, RFP, customer requirement, specification, peer-reviewed paper) must be verified against that source before the test suite is accepted as the contract. The verification is simple: does the source actually say what the test claims it says? A test that attributes a requirement to a numbered SOW clause must be checked against the actual SOW to confirm the clause exists and contains the claimed language. A test that attributes a number to a paper must be checked against the paper. This rule exists because LLMs confidently fabricate plausible-sounding source attributions — a test can cite a requirement ID for language the source never states, and downstream agents will treat the test as authoritative without re-checking. The test suite is the contract; a contract built on fabricated requirements is worse than no contract at all, because it creates false confidence. The verification burden is on the agent writing or modifying the test, not on downstream consumers. Reviewers of the test suite (Beck's review at A.12 step 1, or any agent modifying the suite at A.12 step 4) must also verify any source claims they encounter that they have not personally checked.

---

## On Test Count

"Around 150" is a starting point. Use engineering judgment:

- A short article might need 30 tests
- A complex regulatory submission might need 400+
- Software projects routinely use thousands of tests

Claude isn't afraid of large test suites. The right number is whatever comprehensively covers your document's requirements without inventing tests for their own sake.

---

## Why Claude Specifically

This method leverages Claude's architectural strengths:

- **Large context window**: Hold entire test suite + outline + draft simultaneously
- **Systematic reasoning**: Test validation is natural for a model trained on code
- **Consistency maintenance**: Cross-reference checking across long documents
- **Structured output**: Generates well-organized tests and outlines without drift

The method may be less effective with models optimized primarily for conversational fluency rather than systematic task completion.

---

## The Revision Pass (Optional Prompt 4)

After the document passes its test suite, a dedicated editing pass can improve prose quality without changing technical content. This addresses a specific LLM failure mode: text that passes all technical tests but reads like statistical average prose — correct but lifeless. Wikipedia's WP:AISIGNS (Signs of AI Writing) catalogues the patterns: overused transition words ("Additionally," "Furthermore"), dangling present participles ("ensuring," "highlighting"), puffery ("groundbreaking," "crucial"), elegant variation (synonyms that shift meaning), and superficial analysis ("reflecting broader trends").

The revision pass operates under strict constraints:
- **Subtract or replace, never add.** Replacements must be shorter than originals.
- **Preserve all technical meaning.** Technical terms and precise jargon survive. Decorative language does not.
- **Test suite remains the contract.** Edits cannot introduce new test failures. Run the suite again after revision.

This pass runs after content is frozen, so there is no risk of scope creep. A reviewer familiar with the domain (not just consistency checking, but evaluating whether a cognizant reader can follow the technical design intent) should verify that edits did not break the document's communication architecture.

```
Revise the attached document for prose quality. Do not add content
or change technical meaning. Only subtract or replace words.
Every replacement must be shorter than the original.

Remove: decorative adjectives, dangling participle phrases
("ensuring...", "highlighting..."), puffery, elegant variation
(synonyms that shift meaning), superficial analysis
("reflecting broader trends"), and transition words that add
no logical connection ("Additionally", "Furthermore", "Moreover").

Preserve: all technical terms, quantitative claims, and
specific facts.

[Attach: Document that passed all tests from Prompt 3]
[Attach: Test suite from Prompt 1 (for re-validation)]
```

---

## That's It

Define what "done" looks like. Ask Claude to build it. Validate against your definition.

The method is simple because you're working with the model's strengths instead of against them.

---

*TDD for Documentation, v1.2*
