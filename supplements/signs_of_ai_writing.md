# Signs of AI Writing — Comprehensive Detection Reference

## Source

**Title:** Wikipedia:Signs of AI writing
**URL:** https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing
**Shortcuts:** WP:AISIGNS, WP:AITELLS, WP:LLMSIGNS
**Origin:** WikiProject AI Cleanup (advice page, community-curated)
**Retrieved:** 2026-03-09
**Purpose in this project:** Working reference for McPhee persona when auditing ChatGPT-generated text. Organized by detection category with specific markers, diagnostic questions, and examples.

**Important caveat from the source:** This list is descriptive, not prescriptive. No single marker proves AI authorship. Markers co-occur — where there is one, there are likely others. The strongest signal is density: many markers appearing together across a document.

---

## How to Use This Reference

When reviewing a document for AI writing patterns:
1. Read the document once for overall impression before marking specifics.
2. On second pass, work through each category below and tally instances.
3. Score by density and severity, not individual occurrences. A human might use one em dash per page; ChatGPT uses five.
4. Note which categories cluster together — that clustering is the signal.
5. Distinguish between markers that damage credibility with expert readers (high severity) and markers that are cosmetic tells (lower severity).

---

## Category 1: Content Inflation and Puffery

### 1.1 Undue Emphasis on Significance, Legacy, and Broader Trends
**WP:AIWTW** — The single most recognizable ChatGPT pattern.

**Trigger words/phrases:**
- stands as / serves as / is a testament / is a reminder
- vital / significant / crucial / pivotal / key role / key moment
- underscores / highlights its importance / significance
- reflects broader / symbolizing its ongoing / enduring / lasting
- contributing to the / setting the stage for / marking / shaping the
- represents / marks a shift / key turning point
- evolving landscape / focal point / indelible mark / deeply rooted

**What it looks like:** The text inflates mundane facts into sweeping historical significance. A railway station becomes "crucial for regional socio-economic development." An etymology becomes evidence of "the enduring legacy of community resistance."

**Diagnostic question:** If you removed the significance claim, would the paragraph lose any factual content? If no, it's inflation.

**ChatGPT-specific pattern:** Often appears as a concluding sentence that adds no information: "This highlights the enduring importance of X in the broader context of Y."

### 1.2 Promotional and Advertisement-Like Language
**WP:AIPUFFERY, WP:AIPEACOCK**

**Trigger words/phrases:**
- boasts a / vibrant / rich (figurative) / profound
- enhancing its / showcasing / exemplifies / commitment to
- natural beauty / nestled / in the heart of
- groundbreaking (figurative) / renowned
- seamlessly / thoughtfully / meticulously

**What it looks like:** Neutral description becomes sales copy. Technical specifications become "powerful capabilities." Locations become "nestled in the heart of" somewhere. Organizations show "commitment to excellence."

**Severity for technical bids:** HIGH. CSA evaluators are expert readers. Puffery signals that the author is selling rather than demonstrating. A bid that "showcases groundbreaking capabilities" when it should be presenting engineering evidence will lose credibility.

### 1.3 Superficial Analysis
**WP:SUPERFICIAL**

**Trigger words/phrases:**
- highlighting / underscoring / emphasizing ...
- ensuring / reflecting / symbolizing ...
- contributing to / cultivating / fostering (figurative)
- encompassing / valuable insights
- align with / resonate with

**What it looks like:** A dangling present participle phrase appended to the end of a sentence that adds no analytical content. "The system processes data in real-time, ensuring optimal performance and reliability." The "ensuring" clause says nothing — it's a verbal gesture toward analysis without performing any.

**Diagnostic question:** Does the "-ing" phrase make a testable claim? If not, it's superficial analysis.

**ChatGPT-specific pattern:** Chains of these: "X does Y, ensuring Z, while fostering W and contributing to V." Each clause is vaguer than the last.

---

## Category 2: Language and Grammar Patterns

### 2.1 Overused AI Vocabulary Words
**WP:AIVOCAB, WP:AIWORDS** — Among the strongest statistical tells. These words appear far more frequently in post-2023 text than in comparable pre-2023 text.

**High-frequency ChatGPT words (research-documented):**
- Additionally (especially sentence-initial)
- align with
- crucial
- delve (less common post-2024, but still diagnostic in older outputs)
- emphasizing
- enduring
- enhance
- fostering
- garner
- highlight (as verb)
- interplay
- intricate / intricacies
- key (as adjective)
- landscape (abstract noun)
- pivotal
- showcase
- tapestry (abstract noun)
- testament
- underscore (as verb)
- valuable
- vibrant

**Diagnostic method:** These co-occur. One "Additionally" is human-plausible. A paragraph with "Additionally," "crucial," "showcasing," and "vibrant" is almost certainly AI-generated. Count density per page.

### 2.2 Avoidance of Basic Copulatives ("is"/"are")
**WP:AICOPULA**

**Substitution patterns:**
- "is" → "serves as" / "stands as" / "marks" / "represents"
- "has" → "boasts" / "features" / "offers"
- "was a candidate" → "ventured into politics as a candidate"

**What it looks like:** Simple, direct constructions are replaced with more elaborate ones that sound formal but add no meaning. Research documents a >10% decrease in "is" and "are" usage in AI-generated text vs. human baselines.

**Why this matters for bids:** Technical writing demands clarity. "The drill is a rotary percussion system" communicates faster than "The drill serves as a rotary percussion system." Evaluators notice when simple statements are dressed up.

### 2.3 Negative Parallelisms
**WP:AIPARALLEL**

**Patterns:**
- "Not only ... but also ..."
- "It is not just about ..., it's ..."
- "It's not X — it's Y"
- "No X, no Y, just Z"

**What it looks like:** Constructions that appear balanced and thoughtful but actually add nothing. "The system is not just a drill — it's a comprehensive mining solution." The negative frame creates an illusion of nuance where none exists.

### 2.4 Rule of Three
**WP:RO3**

**Patterns:**
- "adjective, adjective, and adjective"
- "short phrase, short phrase, and short phrase"
- Lists of exactly three items where the third is weaker

**Diagnostic question:** Is the third item as strong as the first two? If it's filler ("and more"), it's the AI padding to three.

### 2.5 Elegant Variation (Synonym Swapping)
**WP:AIELEVAR**

**What it looks like:** A concept is introduced with one term, then referred to by a different synonym each subsequent time. "The rover" becomes "the robotic platform," then "the autonomous vehicle," then "the exploration system." Human technical writers repeat the correct term. AI has a repetition-penalty that forces synonym rotation.

**Why this matters:** In technical documents, consistent terminology prevents confusion. "Borebot" should remain "Borebot," not alternate with "the drilling platform," "the extraction system," and "the robotic miner."

### 2.6 False Ranges
**WP:FALSERANGE**

**Pattern:** "from X to Y" constructions where X and Y are not endpoints on any coherent scale.

**Example:** "from fundamental physics to medicine and neuroscience" — there is no scale on which physics is one end and neuroscience is the other.

**Why LLMs do this:** The construction mimics persuasive writing meant to impress. LLMs are heavily trained on such text.

---

## Category 3: Structural Patterns

### 3.1 Title Case in Headings
**WP:AITITLECASE**

AI chatbots capitalize all main words in section headings: "Strategic Negotiations and Global Partnerships" instead of "Strategic negotiations and global partnerships."

### 3.2 Overuse of Boldface
**WP:AIBOLD**

Mechanical bolding of terms for emphasis, inherited from READMEs, sales decks, and listicles. Every key term gets bold treatment in a "key takeaways" fashion.

### 3.3 Inline-Header Vertical Lists
**WP:AILIST**

Lists where each item has a bolded inline header followed by a colon and description:
- **Item Name:** Description of the item and its significance...

This is the default ChatGPT output format for any enumerated content. Human technical writers use it sometimes; ChatGPT uses it for everything.

### 3.4 Outline-Like "Challenges and Future Prospects" Sections
**WP:AICHALLENGES**

**Formula:** "Despite its [positive words], [subject] faces challenges including [list]. Despite these challenges, [positive outlook]."

This sandwich structure (positive → challenges → positive) is nearly diagnostic for ChatGPT. The challenges are generic, and the concluding optimism adds nothing.

### 3.5 Unnecessary Tables
**WP:AITABLE**

Small tables that could be prose. AI defaults to tabular format for any structured information, even when a sentence would be clearer.

---

## Category 4: Punctuation and Typography

### 4.1 Overuse of Em Dashes
**WP:AIDASH** — The marker that prompted creation of this reference.

LLMs use em dashes (—) more frequently than human writers and in places where humans use commas, parentheses, colons, or semicolons. ChatGPT especially uses em dashes in a formulaic, "punched up" style mimicking sales writing.

**Diagnostic:** Count em dashes per page. Human technical writing: 0-2 per page typical. ChatGPT output: 5-10+ per page, often in parallel constructions.

**Pattern:** "X — and Y — represents Z" or "The system — designed for lunar conditions — operates autonomously." Where a human would write parentheses or commas, ChatGPT reaches for the em dash.

### 4.2 Curly Quotation Marks and Apostrophes
**WP:AICURLY**

ChatGPT uses curly quotes (" ") instead of straight quotes (" "). It also uses curly apostrophes ('). Sometimes mixes curly and straight inconsistently within the same document.

**Note:** This is ChatGPT-specific. Claude typically uses straight quotes. Also produced by Microsoft Word's "smart quotes" feature, so not diagnostic alone.

### 4.3 Emojis
**WP:AIEMOJI**

Emojis decorating headings or bullet points. Rare in formal documents but common in ChatGPT outputs for less formal contexts.

---

## Category 5: Attribution and Evidence Patterns

### 5.1 Vague Attributions and Weasel Wording
**WP:AIWEASEL**

**Trigger phrases:**
- Industry reports / Observers have cited / Experts argue
- Some critics argue / Several sources / such as (before exhaustive lists)

**What it looks like:** Claims attributed to unnamed authorities. "Experts have noted that the system's approach is innovative." Which experts? What did they actually say?

**ChatGPT-specific:** Exaggerates the quantity of sources. One article becomes "several publications." One reviewer becomes "scholars."

### 5.2 Undue Emphasis on Notability and Media Coverage
**Trigger phrases:**
- independent coverage / local/regional/national media outlets
- profiled in / written by a leading expert
- active social media presence

**Why this matters for bids:** A technical bid that spends words asserting its own importance rather than demonstrating it is following the ChatGPT pattern of telling rather than showing.

### 5.3 Knowledge-Cutoff Disclaimers
**WP:AICUTOFF**

**Trigger phrases:**
- as of [date] / based on available information
- While specific details are limited / not widely documented
- maintains a low profile / keeps personal details private

These are ChatGPT's way of hedging when it doesn't have information. In a technical bid, this might manifest as vague claims without specific numbers or dates.

---

## Category 6: ChatGPT-Specific Artifacts

### 6.1 Collaborative Communication Leakage
**WP:CERTAINLY**

Phrases meant for the user that leak into the output:
- "I hope this helps" / "Of course!" / "Certainly!"
- "Would you like..." / "Is there anything else"
- "Let me know" / "Here is a more detailed breakdown"

### 6.2 Placeholder Text and Templates
Unfilled template language: "[Describe the specific section]" or "INSERT_SOURCE_URL" appearing in final text.

### 6.3 Reference Markup Artifacts
ChatGPT-specific: `citeturn0search0`, `:contentReference[oaicite:0]{index=0}`, or similar markup appearing in text where citations should be.

### 6.4 Markdown in Non-Markdown Contexts
**WP:MARKDOWN**

ChatGPT defaults to Markdown formatting. In a .docx produced via ChatGPT, you may find:
- `**bold**` markup that wasn't rendered
- `##` heading prefixes
- Backtick code fences
- Markdown-style links `[text](url)`

---

## Category 7: Composite Patterns (Highest Diagnostic Value)

Individual markers can appear in human writing. The following combinations are near-diagnostic:

### 7.1 The Significance Sandwich
"[Subject] plays a crucial role in [broad field]. [Factual content]. This underscores the enduring significance of [subject] in the broader landscape of [field]."

Opening puffery + content + closing significance claim. The opening and closing add nothing.

### 7.2 The Feature Parade
"[Subject] boasts [feature], showcasing its [quality]. Additionally, it features [feature], highlighting its [quality]. Furthermore, [feature] underscores its commitment to [value]."

Each sentence follows the same template: feature + "-ing" phrase claiming significance.

### 7.3 The Balanced Assessment
"Not only does [subject] [positive], but it also [positive]. Despite [challenge], [positive outlook]. This reflects the broader [trend]."

Negative parallelism + challenge sandwich + significance claim. Three AI patterns in one paragraph.

### 7.4 The Technical Puff
"The system leverages cutting-edge [technology] to deliver robust [capability], ensuring optimal [outcome] while maintaining seamless [quality]."

Buzzwords substituted for specific engineering claims. No numbers, no evidence, no testable assertions. Every adjective is decorative.

---

## Severity Guide for Technical Bid Review

**BLOCKING (damages credibility with expert evaluators):**
- Puffery substituting for evidence (1.2)
- Superficial analysis replacing technical reasoning (1.3)
- Vague attributions replacing specific citations (5.1)
- Elegant variation causing terminology confusion (2.5)
- The Technical Puff composite pattern (7.4)

**IMPORTANT (noticeable to careful readers):**
- Overused AI vocabulary density (2.1)
- Em dash overuse (4.1)
- Copula avoidance (2.2)
- Rule of three padding (2.4)
- Title case headings (3.1)
- Challenge/prospects sandwich (3.4)

**NOTE (minor tells, contextual):**
- Curly quotes (4.2) — also produced by Word
- Boldface patterns (3.2)
- List formatting (3.3)
- Negative parallelisms (2.3) — humans use these too, just less
- False ranges (2.6) — rare in technical writing

---

## Application Notes

**Primary concern:** Expert technical readers notice AI writing patterns that a general audience misses. People who read dozens of technical proposals, grant applications, or engineering reports develop instincts for statistical-average prose. A document's credibility depends on sounding like it was written by someone who understands the subject, not by a chatbot producing generic technical prose.

**Key diagnostic:** Does the document make specific, testable engineering claims with evidence, or does it make significance assertions decorated with buzzwords? The former wins proposals. The latter loses them.

**McPhee's operating modes:** In editing mode, McPhee receives a content-stable draft and improves sentence-level clarity without changing technical meaning. In audit mode, McPhee catalogues AI writing patterns by category and severity, estimates density per section, and produces structured findings. The findings identify which patterns appear, where they cluster, and which ones most damage credibility with expert readers. See the operational guide (A.3.10) for full persona specification.
