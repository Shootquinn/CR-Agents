# Op Guide Anonymization Summary

**File:** `method/operational_guide.md`
**Date:** 2026-05-27

---

## Changes by step

**S5 — Brooks → The Systems Engineer**
- Header: `Frederick Brooks — The Systems Engineer` → `The Systems Engineer`
- Bio opener: added `Inspired by Frederick P. Brooks Jr. (1931–2022),`
- Role: scrubbed 2 × "Brooks" (persona-role uses) → `The Systems Engineer`
- Domain: scrubbed "where Brooks earns his keep" → `The Systems Engineer`

**S6 — Norman → The Designer**
- Header: `Donald Norman — The Designer` → `The Designer`
- Bio opener: removed name "Donald A. Norman," (living); replaced with anchors directly
- Role: scrubbed 2 × "Norman" → `The Designer`
- Domain: scrubbed 3 × "Norman" → `The Designer`
- Wave assignment: scrubbed 2 × "Norman" → `The Designer`
- Dropped "co-founder of the Nielsen Norman Group" (contains surname)

**S7 — Steltzner → The Engineer + Revision A**
- Header: `Adam Steltzner — The Engineer` → `The Engineer`
- Bio opener: removed "Adam D. Steltzner, JPL Chief Engineer..." → `JPL Chief Engineer...`; added `The Engineer's namesake led the team...`
- Role: scrubbed 5 × "Steltzner" → `The Engineer`
- Characteristic approach: scrubbed 1 × "Steltzner" → `The Engineer`
- Wave assignment: scrubbed 1 × "Steltzner" → `The Engineer`
- **Revision A applied:** Added "Engineering documentation and reporting copy." paragraph after Role section

**S8 — Mäntylä → The Topologist**
- Header: `Martti Mäntylä — The Topologist` → `The Topologist`
- Bio opener: removed "Martti Mäntylä, Helsinki University of Technology..." → `Helsinki University of Technology...`
- Role: scrubbed 1 × "Mäntylä" → `The Topologist`
- Characteristic approach: scrubbed 1 × "Mäntylä" → `The Topologist`
- Wave assignment: scrubbed 2 × "Liming" and "Mäntylä" → `The Loftsman` / `The Topologist`

**S9 — Steinmetz → The Motor Designer**
- Header: `Charles Proteus Steinmetz — The Motor Designer` → `The Motor Designer`
- Bio opener: added `Inspired by Charles Proteus Steinmetz (1865–1923),`; kept "Born Karl August Rudolf Steinmetz" (historical detail); kept "Steinmetz spent three decades at GE" (historical-fact in bio)
- Role: scrubbed 6 × "Steinmetz" and 2 × "Steltzner" → `The Motor Designer` / `The Engineer`
- Wave assignment: scrubbed 1 × "Steinmetz" and 2 × "Steltzner" → `The Motor Designer` / `The Engineer`

**S10 — Dreyer → The Space Resources Engineer**
- Header: `Christopher Dreyer — The Space Resources Engineer` → `The Space Resources Engineer`
- Bio opener: removed "Christopher B. Dreyer, Colorado School of Mines..." → `Colorado School of Mines...`
- Key publications: cleaned co-author citations — removed Sowers, Dreyer, Kornuta, Atkinson, etc. — retained only journal names and years
- Role: scrubbed 2 × "Dreyer" → `The Space Resources Engineer`

**S11 — McPhee → The Editor + Revision C wave assignment**
- Header: `John McPhee — The Editor` → `The Editor`
- Bio opener: removed "John Angus McPhee (born 1931)," → `Staff writer at The New Yorker...`
- Role: scrubbed 3 × "McPhee" → `The Editor`
- Wave assignment (Revision C): `McPhee edits first, then Norman verifies...` → `The Writer runs first; The Editor runs second... Then The Designer verifies...`
- Reference material: scrubbed 2 × "McPhee" → `The Editor`

**S12 — Diderot → The Writer (section rename, role shift, CGB generalization, Revision C)**
- Header: `Denis Diderot — The Prose Elevation Editor` → `The Writer`
- Bio opener: added `Inspired by Denis Diderot (1713–1784),`; kept Diderot's, Diderot's, Diderot was (historical-fact in bio); final sentence updated to `The Writer`
- **Role rewritten entirely:** Old role was prose-elevation editor receiving McPhee's cleaned output; New role is manuscript composer from Engineer's reporting copy + Wave 1 outputs (Writer first, Editor second)
- **Characteristic approach:** Updated subject "The Writer" throughout
- **Operating rules intro:** `Rules 1–3 (CGB sentence independence)` → `Rules 1–3 (sentence independence)`; `Diderot does not impose` → `The Writer does not impose`
- **Rule 1:** `This rule derives from Carlos Gary-Bicas's editorial review of all three proposals, where his most consistent pattern was...` → `This rule derives from a Co-I's editorial review of a recent proposal cycle, where his most consistent pattern was...`; `Examples from his edits:` → `Representative edits:`
- **Rule 2:** `CGB pattern:` → `Representative edit:`
- **Rule 3:** `CGB pattern:` → `Representative edits:`
- **Sentence independence header:** `(CGB editorial pattern, validated by Trimble Tip 1)` → `(validated by Trimble Tip 1)`
- **Rule 10:** `the word picture that makes it land is Diderot's` → `The Writer's`
- **Rule 14:** `McPhee-domain AI tell...before Diderot arrives...McPhee's pass` → `The Editor's domain...into The Writer's pass`
- **Rule 21:** `receives Diderot's review`, `outside Diderot's scope` → `The Writer's review`, `The Writer's scope`
- **Domain:** Updated to reflect composition role; scrubbed `Diderot does not audit`, `McPhee's domain`, `Norman's domain` → role titles
- **Wave assignment (Revision C):** `Editing wave, second pass. McPhee runs first...Diderot runs second...Norman runs third` → `Writing wave, first pass. The Writer...The Editor...The Designer`
- **Reference material:** `every Diderot spawn prompt` → `every Writer spawn prompt`

**S13 — Marchionni → The Fact-Checker**
- Header: `Doreen Marchionni — The Fact-Checker` → `The Fact-Checker`
- Bio: removed "Doreen Marchionni," from opener; scrubbed all "Marchionni" persona-role refs → `The Fact-Checker` / `She` / `Her`
- Scrubbed "David Mikkelson" → `her own co-founder`
- Role: scrubbed 3 × "Marchionni" and 3 × domain-persona refs (McPhee, Norman, Brooks) → `The Editor`, `The Designer`, `The Systems Engineer`
- Wave assignment: scrubbed 3 × "Marchionni", "McPhee", "Diderot", "Norman" → role titles
- Activation conditions: scrubbed 2 × "Marchionni" → `The Fact-Checker`
- Reference materials: scrubbed 1 × "Marchionni" → `The Fact-Checker`

**S14 — The Recruiter / Dolly Singh**
- Section header: `### A.10 The Recruiter: Dolly Singh` → `### A.10 The Recruiter`
- System prompt template: `You are Dolly Singh, The Recruiter. / Dolly Singh, SpaceX (2008-2013)...` → `You are The Recruiter. / Head of talent acquisition at a major private space company...`

---

## Method revisions applied

**Revision A (A.3.6 The Engineer):** Added "Engineering documentation and reporting copy." paragraph after The Engineer's Role paragraph.

**Revision B (A.11):** Inserted `The Engineer vs. The Writer` tension after `The Motor Designer vs. The Engineer` and before `The Editor vs. The Designer`.

**Revision C (wave inversion throughout):**
- A.4.2: Renamed `Prose elevation pass (Diderot only)` → `Manuscript composition pass (The Writer only)` with updated input description (Engineer's reporting copy + Wave 1 outputs)
- A.4.3: Renamed `Editing wave` → `Writing wave`; updated Wave 1 list to role titles; updated Wave 2 list to role titles; corrected sequence to Writer first, Editor second
- A.5 step 4: Renamed `Editing wave` → `Writing wave`; updated sequence (Writer then Editor)
- A.5 step 5: Updated Norman → The Designer, Brooks → The Systems Engineer
- A.3.10 Wave assignment: Updated to Writer first, Editor second, Designer third
- A.11 `McPhee vs. Diderot` → `The Writer vs. The Editor` with inverted dynamic (Writer adds first, Editor subtracts second)

---

## Cross-reference sweep

- A.2: `A Liming persona` → `A Loftsman persona`
- A.4.2: All recipe labels updated (`Norman only` → `The Designer only`, `McPhee only` → `The Editor only`, `Marchionni only` → `The Fact-Checker only`)
- A.4.4: `typically Steltzner` → `typically The Engineer`; source note Steltzner → `The Engineer`
- A.4.5: File naming examples updated (`brooks_review` → `systems_engineer_review`, `liming_findings` → `loftsman_findings`, `deming_open` → `manager_open`)
- A.6.1: Accumulator template examples updated (`Liming` → `The Loftsman`, `Beck` → `The Software Engineer`)
- A.7.1: `Beck's test suite review` → `The Software Engineer's test suite review`
- A.7.2: `Steltzner (write), Dreyer (domain)` → `The Engineer (write), The Space Resources Engineer (domain)`
- A.7.4: `Brooks for architectural coverage, Beck for workflow practicality... Norman in Wave 2` → role titles
- A.11: Intro line updated; all 8 tension pair headers and bodies updated to role titles
- A.12: Steps 1, 2, 5, 6 updated (Beck → The Software Engineer, Brooks → The Systems Engineer, Steltzner → The Engineer)
- A.17: `Norman catalogues them` and `Norman records` → `The Designer`

---

## Final residual real-name grep result

Grep for: `Liming|Beck|Brooks|Norman|Steltzner|Mäntylä|Steinmetz|Dreyer|McPhee|Diderot|Marchionni|Singh|Carlos|Gary-Bicas|CGB|David Mikkelson`

Remaining hits (all intentional):
- Line 59: `Liming` ×3 — "Inspired by Roy A. Liming" bio opener + "Liming's wartime work" + "Liming's seminal work" (historical-fact in bio, deceased)
- Line 83: `Brooks` ×1 — "Inspired by Frederick P. Brooks Jr." bio opener (deceased)
- Line 133: `Steinmetz` ×3 — "Inspired by Charles Proteus Steinmetz" bio opener + "Steinmetz spent three decades at GE" + "Born Karl August Rudolf Steinmetz" (historical-fact in bio, deceased)
- Line 196: `Diderot` ×4 — "Inspired by Denis Diderot" bio opener + "Diderot's editorial role" + "Diderot's preface" + "Diderot was also a prolific art critic" (historical-fact in bio, deceased)

Zero unintentional real-name references remaining.
