# Signs of AI Writing: Comprehensive Detection Reference

## Source
Wikipedia:Signs of AI writing (WP:AISIGNS, WP:AITELLS, WP:LLMSIGNS), https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing, an advice page from WikiProject AI Cleanup. Retrieved 2026-09-06; trimmed to CSA bids written with Claude and Gemini. Descriptive, not prescriptive; density is the signal. Read Category 12 before accusing on style alone.

---

## Ordering and Use
Worst first, not by taxonomy. **Performative Epistemics is Category 1**, this project's own addition. **Negative Parallelism is BLOCKING** (2), not NOTE; its reversed form is at 2.3. Tally by category, score by density, and run Category 11 and 12 first.

---

# TIER 1: BLOCKING
Damages credibility with expert evaluators.

---

## Category 1: Performative Epistemics (Epistemic Theater)
**No Wikipedia shortcut. This project's own category.**

This pattern is grammatically clean. The sentence's only job is to perform about the document's own rigor. It is a song and a dance instead of good faith rigor or a testable claim.

**The test:** delete the sentence. If nothing is lost but word count, it was performative, so cut it. If deleting it would let a false or unsupported claim stand, it was a real caveat, so keep it.

### 1.1 Self-Narrating Honesty
The document asserts its own good faith instead of making a claim.
Examples: "and now heavier than the last draft admitted, because burying a correction that hurts us would defeat the purpose of publishing at all." / "That is an honest cost to our position." / "The consequences are real and we state them, not waiting to be told."

### 1.2 Restated Conclusions
A sentence repeats a number or finding just given, dressed as insight.
Example: "...which is to say that at the heaviest fixed-site sizing case the three prospecting cells sit essentially ON that boundary" immediately after the margin was already stated as 1.02.
**Fix:** Cut the second statement. Keep the number.

### 1.3 Throat-Clearing Before the Point
A clause announces that a point is coming, or frames why it matters, before making it, where the fact itself, stated first, would have done both jobs.
Example: "the mast is a storage-reduction device, not structural overhead, and this is why elevation matters more than it looks" used as a section's opening sentence.
**Fix:** Cut the throat-clearing clause. Keep the fact.

### 1.4 Meta-Commentary About the Argument Itself
The document coaches the reader on how to argue with it, instead of making the argument.
Examples: "which a critic will likely go to first," "Attack the duty cycle first," staging an "Invitation to comment" section as "Push first... Push second... Push third."
**Fix:** Delete the staging and meta-commentary to the user.

### 1.5 Redundant Table Framing
Prose explains what a table is about to show, the table shows it, then a closing sentence restates what the table just showed.
**Fix:** Let the table stand alone. Cut the sentence before and the sentence after.
NOTE: it is true tables need to be mentioned in the text. Just not theatrically!

### 1.6 Hedged Qualifier Stacking (Changelog Leakage)
A qualifier records the history of a number instead of the number itself, a revision-log note that leaked into the document.
Examples: "though the margin is narrower than the last draft claimed," "and now heavier than the last draft admitted."
**Fix:** Cut the qualifier. Keep the new number.

### 1.7 Procedural Statements About Your Own Work
**Corresponds to Wikipedia's edit-summary sign "Specific mentions of 'preserved' or 'retained' information, 'avoided' mistakes, and other procedural statements". Wikipedia files it under edit summaries; the mechanism is identical in body prose, status reports and commit messages.**

**Words to watch:** preserved / preserving, retained / retaining, avoided / avoiding, ensured / ensuring, aimed / aiming to, verified independently, checked, confirmed, genuinely, actually.

**Wikipedia's diagnostic, quoted:** *"It is unusual for a human edit summary to include mention of material that was not edited or things that they didn't do."*

**What it looks like:** The writer reports the care taken instead of the result obtained. "Removed promotional language and revised the passage to use a more neutral, balanced tone while preserving the original meaning and technical details." "Reworded lead and body for neutrality and balance... while preserving sourced criticisms."

**In a status report or a chat reply it reads as:** "I did not tell it what to find." "I verified this independently." "The improved wording survived." "I did not relax a row to reach green." "Measured, not asserted."

**Why it survives the throat-clearing ban:** it is disguised as substance. It looks like a fact about the work rather than a claim about the writer, so a writer told to cut preambles will keep it.

**Fix:** State the result. The reader assumes the work was done. If the procedure genuinely matters, it belongs in a method section with an owner, not welded to the finding.

**Severity:** BLOCKING. Same failure class as 3.2/3.3/5.4: assertion substituting for evidence.

---

## Category 2: Negative Parallelism
**WP:AIPARALLEL**, promoted from NOTE to BLOCKING for this project.

The output reads as though it is correcting a misconception the reader never held. It contrasts the real claim against a rejected alternative that nobody proposed, which makes the claim sound weighed rather than merely asserted.

**The test, and it is the whole category:** *did anyone actually propose Y?* If nobody did, cut to X and the sentence loses nothing. The construction is legitimate only when Y is a live option someone raised, or one the reader is likely holding.

**The plain-language version of the defect:** "The cat caught a mouse rather than the dog." The dog was never in the room.

### 2.1 "Not only X, but also Y"
Constructions using "not", "but" or "however" to stage a contrast: "Not only ... but ...", "It is not just ..., it's ...".

Example, from an April 2025 revision to *Eugenio Duse*, and note it runs across two sentences:

> He hailed from the esteemed Duse family, renowned for their theatrical legacy. Eugenio's life, however, took a path that intertwined both personal ambition and familial complexities.

### 2.2 "Not X, but Y"
The stronger form, denying the first characteristic outright: "It's not ..., it's ...", "no ..., no ..., just ...".

Example: "The system is not just a drill — it's a comprehensive mining solution."

### 2.3 "X rather than Y": THE REVERSED FORM
**Absent from the 2026-03 retrieval of this reference. Added 2026-09-06.**

Wikipedia's note: *"This pattern may also be reversed, a construction particularly common in Grok output."*

Their example, from an April 2026 revision to *First Battle of Guilin*, which explicitly states it is from Grokipedia:

> Chiang's strategy emphasized military suppression of these holdouts to enforce subordination, prioritizing empirical consolidation of power amid fragmented loyalties **rather than** ideological purity.

**Worked examples from this project:**

| Written | Nobody proposed | Fix |
|---|---|---|
| "It assumes the polar grade rather than measuring it" | measuring it; no instrument exists | "It assumes the polar grade. Nobody has measured it." |
| "the axis re-scores the ground rather than relocating it" | relocating the ground | "the axis argues about how good that ground is, not where it is" |
| "measured by command rather than assumed" | assuming | state the measurement |
| "reported as a zero rather than omitted" | omitting it | report the zero |
| "keyed on the property rather than a phrase" | keying on a phrase | "keyed on the property" |

**Why this one is worth its own category here:** it is the highest-frequency form in this project's output, it reads as considered judgement, and a writer told only to "remove hedging" will walk straight past it. In one Step 51 audit, nineteen of thirty-two clause-level strikes were this single shape.

---

## Category 3: Content Inflation and Puffery

**3.1** (WP:AILEGACY) stands as, testament, crucial, pivotal, underscores, landscape. **Diagnostic question:** If you removed the significance claim, would the paragraph lose any factual content? If no, it's inflation.
**3.2** (WP:AIPUFFERY) boasts, showcasing, exemplifies, groundbreaking, seamlessly, meticulously; HIGH severity, reads as selling, not demonstrating.
**3.3** (WP:SUPERFICIAL) a dangling "-ing" clause claims analysis without performing any, e.g. "ensuring optimal performance." **Diagnostic question:** Does the "-ing" phrase make a testable claim? If not, it's superficial analysis.

---

## Category 4: Attribution and Evidence Patterns

**4.1** (WP:AIWEASEL) "Experts have noted..." Which experts? **4.2** (WP:AICONNECT) "was the CEO of" softened to "associated with leadership of." **4.3** (WP:AICUTOFF) hedges it cannot find something, then speculates what the gap "likely" means; in a bid, vague claims without numbers.

---

## Category 5: Composite Patterns

**5.1 The Significance Sandwich:** puffery opener, fact, significance close adding nothing. "X plays a crucial role in Y. [Content]. This underscores the enduring significance of X."

**5.2 The Feature Parade:** every sentence is feature plus an "-ing" phrase claiming significance. "X boasts Y, showcasing Z. Additionally, it features W, highlighting V."

**5.3 The Balanced Assessment:** negative parallelism, then a challenge sandwich, then a significance claim. Three patterns in one paragraph.

**5.4 The Technical Puff:** buzzwords substitute for engineering claims. "The system leverages cutting-edge [technology] to deliver robust [capability], ensuring optimal [outcome] while maintaining seamless [quality]." No numbers, no evidence, every adjective decorative.

---
# TIER 2: IMPORTANT
Noticeable to careful readers; density is the signal.

---

## Category 6: Language and Grammar Patterns

**6.1** (WP:AIVOCAB, WP:AIWORDS), the strongest statistical tell: additionally, align with, boasts, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight, interplay, intricate, key, landscape, meticulous, pivotal, robust, showcase, tapestry, testament, underscore, valuable. Delve and tapestry are the most documented of these; the paper is titled "Why Does ChatGPT 'Delve' So Much?" One word is human-plausible; several together are not.
**6.2** (WP:AINOCOPULA) "is" becomes "serves as"; "has" becomes "boasts." **6.3** (WP:RO3) three items, the third weaker filler. **Diagnostic question:** Is the third item as strong as the first two? If it's filler ("and more"), it's padding to three.
**6.4** (WP:AIELEVAR) a new synonym each mention: "the rover," then "the robotic platform." Keep "Borebot" as "Borebot."

---

## Category 7: Structural Patterns

**7.1** (WP:AITITLECASE) title-case headings. **7.2** (WP:AIBOLD) bolding every key term. **7.3** (WP:AILIST) bullets opening with a bolded header and colon, for everything. **7.4** (WP:FACESCHALLENGES) "Despite its [strengths], faces challenges. Despite these, [outlook]." The sign is the rigid formula. **7.5** (WP:AITABLE) small tables that could be prose. **7.6** headings at level 3, or `----` breaks between sections.

---

## Category 8: Punctuation and Typography

**8.1** (WP:AIDASH), the marker that prompted this reference: LLMs use em dashes where humans use commas. The author's scale: novels run about five per book, Moby-Dick about five per page, Claude about five per paragraph. Only Claude, of contemporary models, uses more em dashes than professional writers (July 2026); ChatGPT uses fewer, so it's a live tell here.
**8.2** (WP:AIEMOJI) decorating headings or bullets.

---
# TIER 3: FORENSIC
Near-conclusive when present; absence proves nothing.

---

## Category 9: Model-Specific Artifacts and Leakage

**9.1** (WP:CERTAINLY) "I hope this helps," reviewer-addressed submission notes. **9.2** (WP:AIPLACEHOLDER) "[Describe the specific section]," dates like `2025-xx-xx`. **9.3** (WP:STARTSPAN) Gemini's leak: `[cite: 1]`. **9.4** (WP:MARKDOWN) unrendered `**bold**`/`##` in a .docx. **9.5** (WP:AIFICTREF) invalid ISBN/DOI, wrong-article DOIs, unverifiable page numbers.

---

## Category 10: Edit Summaries and Commit Messages
**WP:AISUMMARY**, applies to commit messages; rigid formulae, and one suggests the edit is AI. **The rigidity is the sign.** **10.1** ensured, adheres to, refined, streamlined, in compliance with, neutrality, clarity, flow. **10.2** see 1.7, BLOCKING here since it appears in body prose too. **10.3** "added sourced content," not what the content is.

---
# TIER 4: COUNTER-EVIDENCE
Run before concluding AI authorship; false accusations are expensive.

---

## Category 11: Signs of Human Writing

**Age.** ChatGPT launched 30 November 2022. Text demonstrably written before that date is not AI-generated, however much it resembles the patterns above. The corpus of human writing is vast enough to produce coincidences.

**Ability to explain an editorial choice.** A writer who can say why they made a mistake, supply the correct link, or quote the passage they were working from, made a human error.

**Syntax LLMs avoid.** Empirically more common in human writing than in AI output:
- Simple *is* and *has* phrases: "there is a", "it has a"
- Plain words where a stiff synonym exists: wrote (not authored), moved (not relocated), used (not utilized), tried (not attempted), died (not passed away)
- Superlative or definitive statements: "one of the best", "is the only", "was the first"
- Hedging qualifiers and intensifiers: very, perhaps, tends to
- Wordy constructions: "as a result of", "in order to", "all of the", "a part of", "the fact that"

**Consistency with pre-2023 work.** If a writer's current output matches their own writing from before chatbots existed, including quirks like heavy bolding, that consistency argues for human authorship.

---

## Category 12: Ineffective Indicators

These are commonly cited and do not work. Some point the opposite way.

- **Perfect grammar.** Many people write well, professionally.
- **Mixed casual and formal register.** Indicates a technical person writing casually, youth, playfulness, neurodivergence, code-switching, or several authors on one page.
- **"Bland" or "robotic" prose.** LLM output skews positive and verbose. It is formulaic in specific ways, and may not read as robotic to someone unfamiliar with it.
- **"Fancy" or academic prose.** LLMs over-use *specific* words. The correlation does not extend to formal vocabulary generally.
- **Transition words in isolation.** Only a few are over-used, and essay-writing humans use them too.
- **Unsourced content.** Most unsourced material predates LLMs. Modern chatbots add citations readily, inaccurate ones but present.
- **Bizarre markup.** Misplaced HTML tags and mangled syntax point to browser extensions, translation tools, or visual-editor mistakes.
- **Correct markup.** Getting complex formatting right is normal for anyone using a visual editor or a preview button.

**Before alleging AI use, consider whether confirmation bias is operating.** False accusations drive away good contributors and create an atmosphere of suspicion.

---

## Differences Between Models, and History

**ChatGPT and Grok** dwell on broader context more than Gemini and Claude. **Claude** alone, of contemporary models, uses more em dashes than professional writers (July 2026). **Grok** over-uses pseudo-scientific vocabulary and the reversed form in 2.3. Historical, rare now: didactic disclaimers, formulaic summaries ("In summary"), generation cut off mid-thought.

---

## Severity Guide

**BLOCKING:** Category 1, incl. 1.7; negative parallelism (2, esp. 2.3); puffery (3.2); superficial analysis (3.3); vague attributions (4.1); the Technical Puff (5.4).
**IMPORTANT:** AI vocabulary (6.1); copula avoidance (6.2); rule-of-three (6.3); elegant variation (6.4); em dashes (8.1, elevated for Claude); title case (7.1); challenge sandwich (7.4).
**FORENSIC:** markup (9.3); citation forensics (9.5); leakage (9.1); placeholder text (9.2).
**NOTE:** boldface (7.2); list formatting (7.3); emojis (8.2).

---

## Application Notes

**Key diagnostic:** testable engineering claims with evidence win proposals; buzzword claims lose them (operational guide A.12.10).

**The dashboard case, 2026-09-06:** a register pass can clear every marker in Categories 1 through 8 and still fail, since the defect was never which words sat beside the numbers, but that words sat there at all. Test: if the prose could be lorem ipsum with no change to the reader's experience, the fix is deletion. This file documents prose tells, not information design.
