# Collaborative Reasoning + Test-Driven Documentation
By: Quinn Morley, Unleashed Robotics

A method for performing technical work using AI-simulated expert teams, test-driven quality assurance, and (optionally) programmatic CAD. Runs on Claude Code. Applicable to engineering documents, proposals, hardware design, systems analysis, or anything complex enough that getting it wrong is expensive.

Based on Nelson (2026), "Activating Expertise: Human-AI Teams for Collaborative Reasoning," implemented via spawned sub-agents with clean context windows per persona, with accumulated judgement calls to enable a persistent "pseudo-ego." Nelson's method is further augmented with the Test Driven Documentation method developed by the author in late 2025.

---

## Quickstart

Point Claude Code at CLAUDE.md. The entry point for every session is:

```
Read CLAUDE.md. Work through prompt0. Here is my work order: [paste work order or point to file]
```

Or more simply:

```
Read CLAUDE.md. Work through prompt0. Today we're going to design an airplane.
```

That is the entire entry point. CLAUDE.md bootstraps the session: it tells the orchestrator where to find the operational guide, the gameplan, and the accumulator. prompt0 verifies the environment and tools and loads the document-production toolkit when needed. The team assembles, builds a gameplan for your review, writes a test suite, and begins executing. You approve each step before the next one starts.

After compaction or a new session, the recovery incantation is the same: "Read CLAUDE.md, work through prompt0, and read the gameplan."

LLM-PLM (Product Lifecycle Management performed by the LLM) activates when CAD/geometry creation is expected to occur. If CAD work happens later, the team will pick up the supplement automatically. You can read more about this in the `supplements/llm_plm_cad.md` file.

---

## Why prompt0 Exists

Claude Code arrives relatively unskilled compared to browser Claude. It cannot open PDFs without help, does not know your formatting rules, and has no document production skills out of the box. It relies on web-fetches for capabilities that browser Claude has built in. prompt0 bridges this gap: it verifies tool availability (Python, Node.js, Pandoc, LibreOffice) and loads the document production toolkit. Without it, you spend the first 15 minutes of every session watching Claude Code fumble with basic file operations.

---

## The Team

Twelve personas anchored to biographical reference material form the standing roster. Each runs as a spawned sub-agent with its own clean context window, receiving only the material relevant to their review task. The biographical anchors are the mechanism: a persona grounded in a specific practitioner's published work and documented approach activates that domain knowledge with precision.

| Persona | Role | Wave |
|---------|------|------|
| The Manager | Scope discipline and deliverable accountability. Manages by enabling, not directing. The people closest to the work understand it best. | Bookends |
| The Loftsman | Mathematician. Analytical lofting, conic geometry, surface definition. | 1 (technical) |
| The Software Engineer | Software engineer. Test-driven workflow, test suite design, practical methodology. | 1 (technical) |
| The Engineer | Engineer. Code implementation, hardware interfaces, empirical verification. | 1 (technical) |
| The Topologist | Topologist. B-rep topology, Euler operators, topological consistency verification. | 1 (technical) |
| The Motor Designer | Motor designer. Electromagnetic design, magnetic circuits, winding, loss analysis. | 1 (technical) |
| The Space Resources Engineer | Space resources engineer. ISRU domain accuracy, TRL assessment, evidence gates. | 1 (technical) |
| The Writer | Manuscript composer. Renders reporting copy into readable prose (composition mode); audits the project's own record for fidelity (steward mode). | Writing wave (1st pass); steward mode at step close and session end |
| The Editor | Editor. AI-ism detection and removal, sentence revision, prose tightening. | Writing wave (2nd pass) / Wave 1 audit mode |
| The Designer | Designer. Document design, echo-site tracking, compliance maps. Reviews every change. | 2 (review) |
| The Systems Engineer | Systems engineer. Conceptual integrity, architecture coherence, revision integrity. | 2 (review) |
| The Fact-Checker | Fact-checker. Traces every factual claim to a primary source; catches confabulation invisible to every other persona. | 2 (review) |

Wave 1 agents run in parallel on technical work. Wave 2 agents run sequentially after integration. The Manager bookends the cycle.

Productive tensions are structural, spanning both domain pairings and process-level pairings specific to the complement protocol — full list at `method/operational_guide.md` A.8. Do not resolve these. Disagreement between tension pairs is information.

**The Recruiter.** Spawned when a task requires expertise outside the standing roster. The Recruiter identifies a historical figure whose published work covers the capability gap and produces a persona specification for human approval.

**The complement protocol.** A conditional fifth wave (`method/operational_guide.md` A.14): before a step's work begins, a criteria agent writes down what bad output on this step would look like. After the work is done, four checkers — error, coherence, tradeoff, horizon — converge on that criteria file, not on the work itself, each in a clean context with no accumulator and no view of the build's own reasoning. Error and coherence findings block the step's close until fixed or ruled; tradeoff and horizon findings write straight into the gameplan's record and block nothing.

---

## Workflow

```
Work Order / Change Order / Ask
        |
        v
   CLAUDE.md ---------> Bootstraps session, pointers to all files
        |
        v
   prompt0.md --------> Environment setup, tool checks, document toolkit
        |
        v
   Gameplan -----------> Created BEFORE execution; user reviews
        |
        v
   Test Suite ---------> Pass/fail criteria; The Software Engineer reviews before production
        |
        v
   Execution ----------> Wave 1 technical, integration, writing wave, Wave 2 review
        |
        v
   Complement Wave ----> (conditional) four checkers converge on a pre-written criteria
        |                file, not the work itself; error/coherence block close
        v
   Inter-Step Gate ----> The Manager closes; user approves before next step
        |
        v
   Delivery
```

Every step produces reviewable documents on disk. The accumulator preserves persona history across spawns and sessions. If the context window compacts mid-session, CLAUDE.md provides the recovery sequence.

---

## Files Included

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Bootstrap. Read first. File pointers, process rules, recovery protocol. |
| `prompt0.md` | Environment setup. Tool checks, document-production toolkit. |
| `method/operational_guide.md` | The method, in full: standing roster (quick reference and full specifications), spawn templates, working loop, wave rules, complement protocol. |
| `method/tdd_method.md` | Test-driven documentation. Defines how test suites are built and used. |
| `supplements/llm_plm_cad.md` | CAD/geometry supplement. Active only during parametric modeling work. |
| `supplements/signs_of_ai_writing.md` | AI writing detection. 7 categories, severity ratings. Loaded into every Editor spawn. |
| `supplements/freecad_api_reference.md` | FreeCAD headless scripting. Project-specific; swap in your own for other CAD tools. |
| `templates/gameplan.md` | Blank gameplan. Objectives, steps, progress log, design notes. |
| `templates/accumulator.md` | Blank accumulator. Per-persona contribution history. |
| `templates/work_order_drafter.md` | Prompt for drafting work orders in web chat before handing to Claude Code. |
| `claude-docx-bundle/` | Document production toolkit. docx and pdf creation and validation. |
| `claude-docx-bundle/docx_style.md` | Formatting rules. Default: Times New Roman 12pt, Unleashed heading hierarchy. |

---

## Customizing the Document Style

The default formatting style is defined in `claude-docx-bundle/docx_style.md` (Times New Roman 12pt, specific heading hierarchy, APA 7 citations). To use your own style:

1. Place a .docx file in the project root with "STYLE" in the filename (e.g., "USE THIS STYLE.docx").
2. The toolkit detects it automatically and uses it as the formatting reference.
3. If no user style file is found, the default applies.

You can also edit `claude-docx-bundle/docx_style.md` directly to change the defaults.

---

## What You Need to Know

The human is a team member, not a director. Your job is creative origination, evaluative judgment, and reflective practice. The team handles execution. You approve or redirect at each gate.

Gameplans are created before execution and may take a couple iterations to get right. This is normal and cheap. Fixing a missing section in a gameplan costs minutes; fixing it in a draft costs hours.

The one-step gate is enforced: after The Manager closes a step, Claude Code stops and reports. It does not open the next step until you say so. Use this time to provide feedback, adjust the gameplan, or flag issues.

Work orders can be drafted in Claude web chat using `templates/work_order_drafter.md` before handing them to Claude Code for execution.
