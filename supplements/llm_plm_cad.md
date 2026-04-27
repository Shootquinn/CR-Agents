# LLM-PLM with Test-Driven Development: Method Supplement

**Author:** Quinn Morley, Unleashed Robotics
**Date:** February 2026
**Companion to:** *Programmatic CAD and Lightweight PLM for Hardware Startups* (report_draft.md)
**Process:** Collaborative Reasoning (Nelson, 2026), via prompt0.md and operational guide

---

## 1. What This Document Is

This is the method supplement for running LLM-assisted programmatic CAD and lightweight PLM using the collaborative reasoning process and test-driven development. It is not a prompt. It is a reference document that a Claude Code orchestrator reads alongside the operational guide and a work order or change order to understand *what kind of work it is doing and how the CAD/PLM outputs fit together.*

The orchestrator's process comes from prompt0.md and the operational guide. This document provides the domain knowledge: what the project directory looks like, what ASSEMBLY.md is, what a change order propagation document contains, how FCStd and STEP files work, and what the test-driven workflow looks like when the deliverable is geometry rather than prose.

**Typical session start:**
> "Good morning, check out prompt0.md before we get started and then the work order we have today is called NewDevice03.md."

The orchestrator reads prompt0.md (environment and process setup), reads this document (domain method), then reads the work order (what to do today).

---

## 2. The Toolchain

**FreeCAD-native PartDesign** is the primary workflow. Parts are built by headless Python scripts executed inside `FreeCADCmd.exe`. The FCStd file is authoritative for geometry once it exists (see Section 5.2).

### 2.1 Source-of-Truth Hierarchy

**Governing principle:** The most concrete artifact wins. Geometry in an FCStd file is more concrete than numbers in a YAML file, which are more concrete than a script's comments.

**Authority ranking (highest to lowest):**

1. **FCStd files** (`parts_cache/*.FCStd`, assembly FCStd) -- authoritative for geometry once they exist. If an FCStd file and params.yaml disagree, the FCStd is correct. Scripts must read FCStd state and update params.yaml to match before proceeding.

2. **params.yaml** -- authoritative for design parameters when no FCStd exists yet, or when a parameter has no FCStd counterpart (additive change). Pre-flight sync keeps it current with FCStd state. Updated in exactly two circumstances: (a) the engineer edits it directly, (b) pre-flight sync from FCStd.

3. **Build scripts** (`build_parts/*.py`, `assemble.py`) -- generators, not authorities. They read params.yaml, produce or modify FCStd files, and must never overwrite FCStd state that has diverged from params.yaml without first syncing.

**Exception:** An explicit engineer request to rebuild from scratch overrides FCStd authority. Log the authorization.

### 2.2 Component Stack

| Layer | Tool | Format | Notes |
|---|---|---|---|
| Part geometry | FreeCAD PartDesign::Body | `.py` scripts reading `params.yaml` | Sketcher + Pad/Revolution/Pocket/Hole features |
| OCCT bridge shapes | `Part.makeCylinder`, `Part.makeBox`, `BSplineCurve` | Transient in-memory shapes | Tool shapes consumed by `PartDesign::Boolean` |
| Assembly | App::Link + native Assembly workbench joints | `assemble.py` | Links to `parts_cache/*.FCStd` |
| Parameters | YAML | `params.yaml` at project root | Seed parameters; FCStd takes primacy once it exists |
| Assembly description | Markdown | `ASSEMBLY.md` at project root | Inter-part constraints and coordinate system |
| Output | FCStd (authoritative) + STEP (export) | `parts_cache/*.FCStd`, `output/*.step` | FCStd carries the live feature tree; STEP is for sharing |
| Version control | Git | All text files; FCStd as design artifacts | FCStd is binary -- meaningful diffs require pre-flight check (Section 5.2) |
| Verification | FreeCAD GUI | Interactive, editable | GUI is a viewer AND an editor -- not read-only |
| Change orders | Markdown | `CO-NNN.md` + `CO-NNN_propagation.md` | Same as before |

### 2.3 Headless Execution

All build scripts run inside FreeCAD's headless interpreter:

```
"C:/Program Files/FreeCAD 1.0/bin/FreeCADCmd.exe" build_parts/<part_name>.py
```

FreeCADCmd.exe is FreeCAD without the GUI event loop. It loads the full kernel (Part, PartDesign, Sketcher, Assembly modules) and runs any Python script. Multiple instances can run in parallel on independent FCStd files; there is no shared state. Typical build times: 1--5 seconds for simple parts, 10--40 seconds for parts with many booleans or complex profiles.

### 2.4 PartDesign::Body as Primary Container

Every part is a `PartDesign::Body`. This is a hard rule, not a preference. PartDesign::Body provides a parametric feature tree (Pad, Pocket, Revolution, Hole, Fillet, Chamfer), a sequential tip pointer where each feature builds on the previous one, and placement in the Body's local coordinate frame.

**The Body is what the engineer edits in the GUI.** When a human opens an FCStd and modifies a sketch dimension or moves a feature, they are working inside the Body's feature tree. This is why PartDesign::Body matters -- it is editable, unlike a monolithic OCCT solid.

### 2.5 OCCT Authorization

OCCT non-parametric shapes (`Part.makeCylinder`, `Part.makeBox`, `Part.BSplineCurve`, etc.) are authorized in three cases only:

1. **Frozen potted assemblies** that will never be edited (e.g., DumbStatorSolid for mass representation).
2. **Imported third-party geometry** (McMaster STEP files) that arrives as OCCT shapes.
3. **Tool shapes for PartDesign::Boolean** -- cylinders, boxes, or complex profiles (epitrochoids) consumed as cut/fuse tools into a PartDesign::Body.

Case 3 is the most common. If a feature cannot be expressed as a Sketcher profile (periodic B-splines with 500+ control points, radial pockets from a bore surface where datum planes produce delta=0), build an OCCT tool shape and consume it via `PartDesign::Boolean`. The tool is non-parametric, but the Boolean operation sits in the feature tree. Debug the PartDesign approach first; drop to OCCT only when PartDesign genuinely cannot express the geometry.

### 2.6 The Three-Layer Pattern

Parts that combine PartDesign features with OCCT tool shapes follow this pattern:

1. **PartDesign features** -- Pad, Revolution, Pocket, Hole, SubtractiveCylinder, Fillet, Chamfer. These are the parametric backbone.

2. **OCCT tool shapes** -- `Part.makeCylinder(...)`, `Part.makeBox(...)`, `Part.BSplineCurve().interpolate()`. Transient shapes built in Python, assigned to a `Part::Feature`, and immediately consumed by step 3. No parametric history.

3. **PartDesign::Boolean bridge** -- `PartDesign::Boolean` (Cut or Fuse) takes the OCCT tool shape and applies it within the Body's feature tree. The result is a named operation the engineer sees in the tree.

**The "Fuse First, Boolean Once" rule:** When applying multiple OCCT cuts (e.g., 40 magnet pockets, 15 coil bores), fuse all tool shapes into a single compound first, then perform one `PartDesign::Boolean` cut. Sequential booleans on the same Body cause volume collapse -- the second cut interacts with or undoes the first.

```python
# Pattern: Combined Tool (mandatory for >1 OCCT cut per Body)
shapes = [Part.makeCylinder(r, h, pos, axis) for pos, axis in features]
fused = shapes[0]
for s in shapes[1:]:
    fused = fused.fuse(s)
tool = doc.addObject("Part::Feature", "CombinedTool")
tool.Shape = fused
bool_cut = body.newObject("PartDesign::Boolean", "AllCuts")
bool_cut.Type = "Cut"
bool_cut.Group = [tool]
doc.recompute()
```

For large feature counts (>15 shapes), use a binary merge tree to fuse in O(n log n) instead of O(n^2).

### 2.7 Assembly via App::Link

Assembly is built in `assemble.py` using `App::Link` objects that reference bodies in `parts_cache/*.FCStd` files. App::Link stores a reference to the source FCStd file and body, not a copy of the geometry; the assembly FCStd is small (kilobytes). Source documents must be open when setting `LinkedObject`, so the assembly script opens all source FCStds before creating links. Placement is set on the Link, not on the source body -- each link has its own position and rotation in the assembly coordinate frame. McMaster STEP imports remain as `Part::Feature` shape copies (they have no source FCStd).

The native Assembly workbench provides joint constraints (Grounded, Fixed, Rigid, Revolute) backed by the Ondsel MbD solver. Current practice: explicit XYZ placements on links (equivalent to grounded joints). Target: upgrade specific interfaces to proper joints for design exploration.

**Multi-level assemblies.** When a project has sub-assemblies (e.g., a module assembly and a system assembly), the system-level script can either: (a) link to the sub-assembly FCStd as a single unit, or (b) link to individual part FCStd files and replicate the sub-assembly's internal placements. Option (b) gives per-part visibility and editability but requires the system script to compose placements: `system_plc.multiply(sub_assembly_internal_plc)`. See FreeCAD API reference "App::Link Placement Behavior" for the three placement categories (explicit, source, identity) and composition rules. **`Link.Placement` does not compose with `LinkedObject.Placement` — the link's placement is the only transform.** Failing to compose placements causes parts to cluster at zone boundaries with correct volumes but wrong positions.

**Assembly script hierarchy.** For multi-level projects, the directory may contain multiple assembly scripts:

```
build_parts/
├── assemble.py                 # Sub-assembly (e.g., one motor module)
├── assemble_<variant>.py       # Variant sub-assembly (e.g., gearbox-less motor)
├── assemble_<system>.py        # System-level assembly (links individual parts)
└── assemble_<system>_applink.py # System-level App::Link assembly (per-part links)
```

The sub-assembly script is the **authoritative reference** for per-part internal placements. The system-level script must read or replicate these placements, not invent its own.

### 2.8 params.yaml as Seed, Not Master

`params.yaml` is the parameter seed. Build scripts read it on first generation to create the initial FCStd. Once the FCStd exists, **the FCStd is the source of truth** for that part's geometry. `params.yaml` remains the source of truth for non-geometric metadata (materials, winding specs, electrical properties) that the FCStd does not carry.

In the FreeCAD-native workflow, the FCStd has a live feature tree that the engineer can and does modify. The script is a generator, not a master.

**Authority hierarchy (for geometry):**

```
FCStd (once it exists)
  > params.yaml
    > build script
```

See Section 5.2 for the Pre-Flight Diff Law that enforces this hierarchy.

---

## 3. Project Directory Structure

```
project-root/
├── params.yaml                 # Parameter seed (geometry, materials, electrical)
├── ASSEMBLY.md                 # Inter-part constraints, coordinate system, assembly sequence
├── build_parts/
│   ├── common.py               # Shared utilities: paths, YAML loader, PartDesign helpers
│   ├── assemble.py             # Sub-assembly builder (App::Link, per-part placements)
│   ├── assemble_<system>.py    # System-level assembly (composes sub-assembly placements)
│   ├── <part_name>.py          # Per-part build script (one FCStd per script)
│   └── <utility>.py            # Support scripts (pregame plots, profile generators)
├── parts_cache/
│   ├── <part_name>.FCStd       # Built part (authoritative once it exists)
│   └── <part_name>_result.json # Build metadata (volume, validity, timing)
├── output/
│   ├── motor_assembly.FCStd    # Assembly document (App::Links to parts_cache/)
│   ├── motor_assembly.step     # STEP export for sharing / external viewers
│   └── *.png, *.mp4            # Renders, cross-section plots, animations
├── RESOURCES/
│   └── <vendor>.STEP           # Third-party geometry (McMaster, etc.)
├── cr_scratch/                 # Working documents, trade studies, draft analyses
├── tests/
│   └── test_borebot.py         # TDD test suite (pytest + conftest.py FreeCAD harness)
├── conftest.py                 # FreeCAD pytest fixture (headless document management)
└── change-orders/
    └── CO-NNN_description/
        ├── CO-NNN.md
        └── CO-NNN_propagation.md
```

### 3.1 Key Conventions

- **One script, one part, one FCStd.** Each `build_parts/<name>.py` produces exactly one `parts_cache/<name>.FCStd`. Do not put two parts in one script.

- **common.py is the shared foundation.** Path constants, the YAML loader, PartDesign helpers, thread specifications, and design constants live here. Every build script imports from it.

- **parts_cache/ is a build cache.** An FCStd becomes the authoritative design artifact the moment it is generated. The `_result.json` sidecar records build metadata (volume, feature count, validity, build time) for automated test consumption.

- **output/ contains assembly-level artifacts.** The assembly FCStd, STEP export, cross-section plots, and renders. The assembly FCStd links into parts_cache/ -- it does not contain copies of part geometry.

- **RESOURCES/ is read-only.** Vendor STEP files live here. Build scripts import from RESOURCES/ but never write to it.

- **params.yaml is singular.** One file at project root, not per-part. Sections are keyed by subsystem (geometry, winding, materials, etc.). Build scripts read what they need.

- **Build scripts are self-contained.** Each script runs independently via `FreeCADCmd.exe <script>.py`. Parallelism comes from launching multiple FreeCADCmd.exe processes. `assemble.py` depends on all part FCStds existing; part scripts depend only on `common.py` and `params.yaml`.

### 3.2 Differences from the CadQuery Directory Structure

- **Per-part directories** are gone. A part is a script + an FCStd + a result JSON. The script lives in `build_parts/`, the outputs in `parts_cache/`.
- **Per-part CHANGELOG.md** is gone. Change history lives in Git and in the accumulator.
- **BOM.yaml as a separate file** is gone. BOM data (mass, material, count) is embedded in `params.yaml` and `ASSEMBLY.md`.
- **build/ subdirectories** are gone. The flat `parts_cache/` directory replaces nested `parts/<name>/build/` trees.

---

## 4. ASSEMBLY.md — The Instance-Awareness Document

ASSEMBLY.md is the structured Markdown file that makes LLM-assisted change propagation possible. It describes the complete assembly for two audiences: the human engineer (single-page system overview) and the LLM processing a change order (structured context for dependency tracing).

**Required sections:**

### 4.1 Parts Table
Every part: name, filesystem path, instance count, brief description.

### 4.2 Parametric Dependencies
The most important section for change order processing. Documents inter-part constraints as scalar relationships:

```
- washer.inner_diameter - bolt.shank_diameter >= 0.2   (clearance fit, min 0.2mm)
- nut.inner_diameter - bolt.shank_diameter in [0.125, 0.625] mm  (thread clearance)
- nut.thread_pitch == bolt.thread_pitch                 (thread compatibility)
- bolt.thread_length == bolt.shank_length               (fully threaded, ISO 4017)
```

Without this, the LLM must infer relationships from isolated `params.yaml` files (which contain no cross-references). With it, the LLM traces documented constraints.

**Limitations of scalar notation:** These constraints project geometric relationships into a lower-dimensional space. `nut.inner_diameter - bolt.shank_diameter in [0.125, 0.625]` records a clearance range but does not express the full thread engagement relationship (helix angle, major/minor diameters, pitch diameter, thread form). The notation is adequate for assemblies where interfaces are geometrically simple. For assemblies with angled interfaces, datum-driven positioning, or standard-table-coupled parameters, the scalar format will not capture the full constraint structure. The LLM should flag geometric ambiguity rather than resolve it.

### 4.3 Coordinate System
Assembly origin, axis conventions, and any global positioning notes.

### 4.4 Assembly Sequence
Build order with joint types and reference geometry for each mate.

### 4.5 Change Order History
Running log of processed change orders.

**Constraint enforcement:** Two layers exist, distinguished by what is computable from scalar parameters. `validate_params()` in the generation scripts enforces all scalar-computable constraints at generation time, including cross-part constraints such as nut-bolt clearance range, washer-bolt clearance, and pitch matching (hard enforcement -- script raises an error). Constraints that resist scalar reduction (coaxiality, thread form compatibility, assembly sequence) are documented in ASSEMBLY.md and enforced by LLM analysis and human review (soft enforcement -- flagged but not automatically blocked). The compensating controls for non-computable constraints are the LLM's analysis and the engineer's review.

**Maintenance:** ASSEMBLY.md is only as useful as it is current. Update it as a mandatory step in every change order propagation. For informal parameter tweaks, update ASSEMBLY.md when committing the parameter change.

**Scalability:** Designed for 5–50 parts. Below 5, the overhead isn't justified. Above 50, decompose into sub-assembly ASSEMBLY.md files with a top-level file referencing them.

---

## 5. Work Orders and Change Orders

### 5.1 Work Orders (New Work)

A work order introduces new parts or assemblies into the project. The work order document should specify:

- **What is being created** — new part(s), new sub-assembly, or new top-level assembly
- **Functional requirements** — what the part/assembly must do
- **Interface constraints** — how it connects to existing parts (mating surfaces, fasteners, clearances)
- **Parameters** — known dimensions, materials, tolerances
- **Acceptance criteria** — how to verify the output is correct

**Work order workflow:**
1. Engineer provides the work order document.
2. Orchestrator reads ASSEMBLY.md (if project exists) to understand the current product tree.
3. CR team (via working loop) defines the test suite for new parts/assemblies, **including functional validity tests and interference checks** (Sections 7.1, 7.4).
4. CR team creates `params.yaml` for each new part. **Norman reviews for functional validity and shape representativeness** (Section 8.4a).
5. **CR team runs sketching study with interference check on sketch geometry. Resolve Design Critique items and interference failures before proceeding** (Section 8.4).
6. CR team writes FreeCAD build scripts, validates against test suite.
7. CR team writes or updates `assemble.py` to integrate new parts.
8. **CR team runs interference check on production assembly. Resolve before proceeding** (Section 7.4).
9. CR team updates ASSEMBLY.md with new parts, dependencies, and assembly sequence.
10. Engineer verifies STEP output in viewer.
11. Commit all text files to Git.

### 5.2 Change Orders (Modifications) -- FreeCAD-Native Workflow

A change order modifies existing parts or assemblies. The change order document (`CO-NNN.md`) records what is changing, why, and which parts are expected to be affected.

#### 5.2.1 The Pre-Flight Diff Law

Before any build script modifies an existing FCStd, it MUST execute a pre-flight check. This is not optional. This is what prevents the agent from destroying the engineer's work.

**The procedure:**

1. **Open the existing FCStd** and extract its current state:
   - Body volume (mm^3)
   - Feature count and feature names
   - Bounding box dimensions
   - File modification timestamp

2. **Compare against params.yaml expected values:**
   - Expected volume (computed from analytical geometry)
   - Expected feature count
   - Expected bounding box

3. **Classify each divergence** into one of three categories:
   - **Match** -- parameter and FCStd agree. No action.
   - **FCStd wins** -- both have the dimension, they disagree. Update params.yaml to match the FCStd.
   - **Additive** -- params.yaml has a parameter with no FCStd counterpart. This is new design intent. Apply the feature to the existing FCStd geometry. Do NOT rebuild from scratch.

4. **If all match:** Proceed with rebuild.

5. **If divergences exist:** The FCStd wins. Read the actual geometry, update params.yaml to match, then apply the requested change on top of the current FCStd state. Never overwrite user edits. Never ask -- just respect the file.

6. **Check cross-part constraints** (from ASSEMBLY.md) after syncing all FCStd states to params.yaml. If constraints are violated, report the violation but do NOT auto-resolve. Two authoritative sources in conflict require human judgment.

7. **Exception:** If the engineer explicitly says "rebuild from scratch" or "this file is broken, regenerate it," that is authorization to overwrite. Log the authorization with the same rigor as a change order.

**Implementation:** A `preflight_check(fcstd_path, expected_params)` function in `common.py` that every build script calls before `FreeCAD.newDocument()`. Returns `True` (safe to rebuild) or raises with a diff report showing what diverged.

```python
# Pattern: Pre-flight check at script top
fcstd = os.path.join(PARTS_CACHE, "stator_shell.FCStd")
if os.path.exists(fcstd):
    preflight_check(fcstd, expected_volume=55200, expected_features=8)
    # If we get here, FCStd matches params -- safe to rebuild
doc = FreeCAD.newDocument("StatorShell")
```

**Why this matters:** FCStd files are binary. You cannot `git diff` them meaningfully. The pre-flight check is the agent's equivalent of a human opening the file and looking at it. Without it, every rebuild is a potential data loss event.

#### 5.2.2 Change Order Workflow (6 steps)

1. **Engineer writes CO-NNN.md** -- the change request with reason and expected affected parts.

2. **Agent reads ASSEMBLY.md + CO-NNN.md** -- traces parametric dependencies to identify all affected parts. Dependencies include both params.yaml cross-references and assembly-level relationships (App::Link placements, joint references).

3. **Agent drafts CO-NNN_propagation.md** -- for each affected part:
   - Whether a rebuild is required
   - Which parameters change (before/after values)
   - Whether the change is **dimensional** (topology preserved) or **topological** (face/edge structure changes)
   - Whether the FCStd has been user-edited (pre-flight check result)
   - Completion status

4. **Engineer reviews** -- approves or modifies the propagation document. This is where the engineer catches what the agent missed: standard-table implications (M4 to M6 implies pitch, head size, washer changes), tolerance stack-ups, manufacturing constraints.

5. **Execute and verify:**
   - Run pre-flight check on each affected FCStd
   - For user-edited FCStds: open existing file, apply change incrementally
   - For unedited FCStds: rebuild from updated params.yaml
   - Run test suite (volume checks, interference, constraints)
   - Rebuild assembly (assemble.py)
   - Export STEP

6. **Update documentation** -- ASSEMBLY.md, params.yaml, accumulator, gameplan.

#### 5.2.3 Dimensional vs. Topological Changes in FreeCAD

This distinction is more consequential in FreeCAD than in script-and-export workflows.

**Dimensional changes** (e.g., increase endplate thickness from 9.5mm to 11mm):
- Topology is preserved. The same faces, edges, and vertices exist; they just moved.
- All PartDesign features survive. A Pad length change does not invalidate downstream Pockets or Holes.
- Assembly App::Link placements may need updating (the part grew, so its neighbors shift).
- Face selectors in downstream features remain valid.

**Topological changes** (e.g., change bolt count from 5 to 6, add a new pocket, change thread size):
- The B-rep face/edge structure changes. Face and edge indices renumber.
- **Any feature that references a face or edge by name may break.** This includes Hole features referencing a datum plane, Fillet features referencing edge names, and Boolean features positioned relative to face geometry.
- Assembly joints that reference specific faces will break.
- The propagation document must flag topological changes explicitly and list every downstream feature and assembly reference that must be re-verified.

**The fillet ordering rule:** Fillets must be applied BEFORE boolean operations that fragment the target edge. A fillet on a clean circular edge succeeds. The same fillet after 40 magnet-pocket booleans have split that edge into 40+ segments will fail. Plan the feature tree so fillets precede booleans that touch adjacent geometry.

#### 5.2.4 The FCStd-First Workflow for Interactive Changes

When the engineer edits a part in the FreeCAD GUI:

1. **Engineer opens FCStd**, makes changes (adjusts a sketch dimension, moves a feature, adds a fillet).
2. **Engineer saves FCStd.** The file is now the source of truth.
3. **Agent detects the edit** via pre-flight check (volume or feature count diverges from params.yaml expectations).
4. **Agent reads the FCStd** to extract the new state: updated dimensions, new features, changed topology.
5. **Agent updates params.yaml** to reflect the FCStd's current state. This keeps the text representation in sync for future builds, test validation, and documentation.
6. **If additional scripted changes are needed** (e.g., the engineer adjusted shell thickness and the agent must now update bolt hole positions), the agent applies them on top of the current FCStd -- opening the existing file, not creating a new document.

The data flow runs in both directions:

```
First build:     params.yaml  -->  script  -->  FCStd
GUI edit:         FCStd  -->  agent reads  -->  params.yaml updated
Scripted change:  params.yaml  -->  script opens existing FCStd  -->  FCStd updated
```

#### 5.2.5 FreeCAD-Specific Change Propagation Concerns

- **Face selector stability.** FreeCAD identifies faces and edges by index (Face1, Edge3, etc.). These indices change when topology changes. Any feature or assembly joint that references a face or edge by name is fragile across topological changes. The propagation document must identify all face-selector dependencies.

- **Feature tree ordering.** PartDesign features are sequential. Inserting a feature in the middle of the tree can invalidate everything below it. The propagation document must specify whether the change is an append (new feature at the end) or an insert (requiring revalidation of downstream features).

- **App::Link placement updates.** When a part's dimensions change, its neighbors in the assembly may need placement updates. Joint-based assemblies propagate such changes through the solver; explicit-placement assemblies require manual coordinate updates in assemble.py.

- **Binary file conflict.** FCStd files cannot be merged in Git. If two changes touch the same part, one must land first and the second must rebuild on top. Change orders on the same part must be serialized. The propagation document must check for in-flight changes to the same FCStd.

---

## 6. STEP Assembly Structure

Understanding STEP structure is essential for verifying output.

A STEP assembly (AP203/AP214/AP242) carries:
- **PRODUCT** entities — one per component, with human-readable names
- **NEXT_ASSEMBLY_USAGE_OCCURRENCE (NAUO)** — parent-child structural links
- **CONTEXT_DEPENDENT_SHAPE_REPRESENTATION (CDSR)** — geometric placements (transformations between coordinate systems)
- **STYLED_ITEM + COLOUR_RGB** — color assignments for visual identification

FreeCAD's STEP export writes this structure through OCCT's XCAF document model. The output is the same structure that SolidWorks, Creo, and other professional tools produce — each component is a separate body with its own geometry, linked by NAUOs and placed by CDSRs. Not Boolean-united into a single solid.

**AP selection:** AP214 by default (carries color). AP203 for maximum compatibility. AP242 if semantic PMI is needed. Controlled by a single variable in the assembly script.

**Viewer verification:** Open the STEP in at least two viewers. Check that the product tree shows individually selectable, named components. Check that colors match assignments. Measure critical dimensions. This catches gross errors; sub-millimeter positioning issues may require closer inspection.

---

## 7. Test-Driven Development for CAD

The TDD method (see method/tdd_method.md) applies to CAD work with adaptations for geometry.

### 7.1 Test Categories

**Part-level tests:**
- Parameter validation (intra-part constraints enforced by `validate_params()`)
- Geometry existence (does the FCStd file generate without error? Does STEP export succeed?)
- Dimensional accuracy (do key measurements match `params.yaml`?)
- Feature presence (does the part have the expected features — threads, chamfers, holes?)
- Volume/mass sanity checks (is the volume in the expected range?)

**Assembly-level tests:**
- Product tree structure (correct number of components, correct names)
- Component placement (are parts in expected positions?)
- Interference (do parts overlap where they shouldn't?)
- Clearance (are required gaps maintained?)
- Constraint satisfaction (do ASSEMBLY.md inter-part constraints hold?)
- Color assignment (are components visually distinguishable?)

**Functional validity tests:**
- Shape representativeness — does the geometry represent a real, manufacturable part, or is it a dimensionally correct placeholder? A woodruff key is a half-moon. A pressure plate finger is a cantilevered beam that can deflect. A drive dog has a realistic engagement profile. If the shape wouldn't be recognized by a machinist or a mechanical engineer as the part it claims to be, it fails.
- Functional feature presence — parts that perform mechanical functions (springs, detents, dogs, keys, splines) must have geometry that could plausibly perform those functions. A flat ring with rectangular tabs is not a diaphragm spring. A rectangular block is not a woodruff key.
- Manufacturing plausibility — could this geometry actually be manufactured by the specified process? Stamped sheet metal has bends. Machined parts have tool-access constraints. Castings have draft angles.
- Parametric-to-geometry fidelity — does the script faithfully represent what params.yaml describes, or did it take a shortcut? If the params say "6 cantilevered fingers with root radius," the geometry must have 6 cantilevered fingers with root radii — not 6 rectangular tabs.

**Workflow-level tests:**
- Reproducibility (does re-running scripts from a commit produce equivalent geometry?)
- Diffability (does `git diff` on params.yaml show the expected changes?)
- Change propagation (does a parameter change in part A correctly trigger updates in parts B, C?)

### 7.2 When to Write Tests

**Before creating parts or assemblies.** The test suite is the contract. It defines what "done" looks like before any geometry is generated.

For a work order:
1. Read the work order requirements.
2. Write the test suite covering part-level, assembly-level, and workflow-level criteria.
3. Review the test suite (Beck persona, via CR working loop).
4. Generate geometry that passes the tests.

For a change order:
1. Read the change order and existing test suite.
2. Identify which tests are affected by the change.
3. Update the test suite (new pass criteria for changed parameters, new tests for new constraints).
4. Execute the change. Validate against updated suite.

### 7.3 Test Execution

Some tests are automated (script runs without error, `validate_params()` passes, FCStd file is non-empty and valid). Others require human verification (visual inspection in viewer, dimensional measurement, interference checking). The test suite should mark each test as **automated** or **manual** and specify the evidence required for a pass.

### 7.4 Interference Checking and Spatial Validation

Interference checking is a mandatory automated gate — not a best-effort recommendation. If parts overlap, the process failed. The user should never be the one to discover that parts occupy the same space.

**Method:** Boolean intersection (`BRepAlgoAPI_Common` or `Part.Shape.common()` in FreeCAD) between neighboring part pairs at their assembly positions. Any intersection volume above tolerance (~0.01 mm³) is a fail. This is computationally inexpensive and catches the class of errors that visual inspection catches too late.

**When to run:**
1. **After sketch study assembly** (Section 8.4) — sketch geometry provides the spatial baseline for early detection. Interference at the sketch level is cheap to fix; interference discovered after production geometry is expensive. If parts overlap in the sketch, they will overlap in production.
2. **After production assembly generation** — mandatory gate before user review. No assembly ships to the user with overlapping parts.
3. **After any change order affecting dimensions or positions** — re-validate spatial relationships whenever the assembly envelope changes.

**Scope:** All neighbor pairs for small assemblies (<20 parts). For larger assemblies, use graph-based neighbor selection (parts sharing an axial zone, radial zone, or documented interface in ASSEMBLY.md).

**Reporting:** Per-pair results: part A name, part B name, intersection volume (mm³), pass/fail. Include in test suite output. Any single fail blocks the gate.

**Principle:** Interference is caught by the process, not by the user. The engineering team's credibility depends on delivering assemblies where every part occupies its own space. A user who opens an assembly and finds overlapping parts loses confidence in everything else the team produced.

### 7.5 Assembly Placement Verification

**Problem:** Dimension-checking tests (volume, bounding box, face count) pass even when assembly parts are displaced, rotated, or on wrong planes. A part can have geometrically correct dimensions but be in a completely wrong position within the assembly. This class of error is invisible to parametric tests and only visible in 3D visualization or explicit placement auditing.

**Mandatory gate for any step that creates or modifies an assembly:**

1. **Placement audit.** After assembly creation, verify every link placement programmatically. For assemblies where parts are built at absolute coordinates, all link placements should be identity (position 0,0,0, rotation 0°). Any non-identity placement is suspect.

2. **Visual verification.** Generate a turntable MP4 or multi-view PNG render of the assembled model. Norman (or equivalent design reviewer) must confirm "this looks like the design." Comparing against a 2D cross-section plot is insufficient — the 2D plot validates geometry, not assembly placement. The 3D render validates both.

3. **Include in test suite.** At minimum, add a test that checks all assembly link placements are within tolerance of their expected positions. This catches the case where a constraint solver or face-matching algorithm silently moves parts.

**Root cause pattern (from WO-2026-002):** CAD tool constraint solvers use face-matching to compute part positions. When parts are already at their final coordinates, face-matching can INTRODUCE displacement rather than correct it. The safe pattern for pre-positioned parts: lock (ground) all parts at their construction coordinates rather than constraining them via face references.

### 7.6 Exploratory and Production Build Passes

CAD geometry is built in two passes within each step. This is not optional.

**Pass 1: Exploratory build.** The purpose of the exploratory build is to surface problems. Every functional feature the part needs must be present in 3D, even if simplified: mounting provisions, torque transfer features, bearing retention, fastener holes, seal grooves, alignment features. A bolt hole can be a plain through-hole (no counterbore). A snap ring groove can be a rectangular cut (no toleranced dimensions). A D-flat can be a boolean cut (no press-fit analysis). But every feature must *exist*, because a missing feature is a missing experiment. You cannot learn anything about bolt-to-split-plane clearance from a flange with no bolt holes. You cannot validate a torque path through a sun gear with no shaft interface.

The exploratory build produces *data*: interference findings, clearance measurements, axial stack-up verification, mass estimates, and a list of design issues discovered during construction. This data informs the production pass.

**Norman gates the exploratory build on completeness:** "Can we learn from this? Are there features we intended to include but didn't, creating blind spots?" A part missing a functional feature fails the exploratory gate. It is not a draft. It is an incomplete experiment. A part with all features present but geometrically simplified passes.

**Pass 2: Production build.** The production build refines every feature to deployment-ready quality, informed by the findings from Pass 1. Counterbores replace plain holes. Retention features get proper dimensions. Fits and tolerances are specified. Chamfers and fillets are added where they serve function (assembly guidance, stress relief). The production build is not "add the stuff we skipped." It is "fix what the prototype taught us."

**Norman gates the production build on quality:** "Would you build this? Would a machinist or printer operator produce a functional part from this geometry?" This is the pride test applied to individual parts, not just the assembly.

**What "deployment-ready" means for a part:**
- Every interface to another part is geometrically defined (bolt holes, register bosses, press fits, snap rings, keyways, splines)
- Every retention mechanism is modeled (bearing shoulders, snap ring grooves, retaining features)
- Every torque/force transfer feature is present (D-flats, keyways, splines, friction surfaces)
- Fastener provisions are complete (through-holes with counterbores/countersinks on the access side, tapped holes on the mating side)
- Seal provisions are modeled where sealing is required (O-ring grooves, face seal surfaces)
- The part could be manufactured by its intended process without additional design work

**Failure mode this prevents:** The AI builds "concept-level" geometry that passes dimensional tests but lacks the mechanical features needed for fabrication or assembly. A corrective sub-step gets created to add what should have been there from the start. By requiring all features in Pass 1 (even simplified), the process catches missing features early and produces useful data for Pass 2.

---

## 8. Collaborative Reasoning Integration

This section describes how the CR team's personas map to LLM-PLM work.

### 8.1 Persona Assignments for CAD Work

| Persona | CAD/PLM Role |
|---|---|
| **Deming** | Opens/closes each cycle. Scopes the work, evaluates output readiness. |
| **Steltzner** | Primary executor. Builds FreeCAD PartDesign parts, runs build scripts, reports results. Validates FCStd and STEP output empirically. |
| **Liming** | Geometric reasoning. Reviews placement math, thread engagement geometry, surface definitions. Validates that parametric dependencies in ASSEMBLY.md correctly represent the geometric relationships. |
| **Beck** | Test suite design and review. Ensures tests validate the right things. Flags ceremony. |
| **Norman** | Design critic for both physical products and documentation. Reviews params.yaml for functional validity and shape representativeness before geometry exists — "will these parameters produce parts that belong in a world-class product?" Reviews sketch and production geometry for the pride test: would this part look at home in a product catalog from a top design house? Reviews ASSEMBLY.md, change orders, and all prose for consistency. Norman catches lazy placeholders, non-functional geometry, and design shortcuts (see functional validity tests, Section 7.1). Every update that touches documentation or physical design. |
| **Brooks** | Architecture-level review. Product tree structure, method coherence, scalability of the assembly approach. |

### 8.2 Wave Assignments

**Wave 1 (technical, parallel):**
- Steltzner: builds/modifies FreeCAD PartDesign parts, runs build scripts, generates FCStd and STEP
- Liming: reviews geometry math, placement calculations, constraint definitions
- Beck: reviews/updates test suite
- Norman: reviews params.yaml for functional validity, shape representativeness, and pride test (params-level review — lightweight, non-blocking, findings feed into Wave 1 scripts)

**Wave 2 (review, after integration):**
- Norman: reviews sketch/production geometry for pride test and functional validity; reviews interference check results (Section 7.4); reviews all documentation changes (ASSEMBLY.md, change orders, changelogs)
- Brooks: reviews product tree architecture, method-level concerns

### 8.3 Recruiter (Dolly Singh)

If a work order involves domain expertise outside the standing roster — e.g., GD&T for tolerance analysis, materials science for thermal properties, electrical engineering for PCB-mechanical interfaces — spawn the recruiter to identify and spec a temporary team member.

### 8.4 Sketching Phase (Visual Design Exploration)

Before the first production geometry step — or at any point where the team has parameters but no visual confirmation of design intent — run a sketching study cycle. This is a lightweight cycle that produces fast geometry for visual validation before committing to detailed scripts. The sketching phase is where design problems are cheapest to fix and where the team's design standards are first enforced.

**Why:** Parametric definitions (params.yaml, ASSEMBLY.md) describe a design numerically, but nobody can evaluate proportions, spatial relationships, or whether a part "looks right" from a spreadsheet of dimensions. The gap between parameter definition and production geometry is where design mistakes hide. A sketching study fills this gap cheaply — and with the addition of Norman's params review and Design Critique, it also catches functional validity failures (Section 7.1) before any geometry investment.

**Cycle structure (3 personas):**

1. **Norman (designer) — three sub-roles:**

   **a. Params review (before geometry):** Norman reviews each part's params.yaml entry before any sketch geometry is generated. The question is not "are the numbers right?" — Beck handles that. Norman's question is: "Will these parameters produce a part I'd show to a client? Does this parameter set describe a real part or a lazy placeholder? Are functional features described with enough specificity to generate representative geometry?" A parameter set that describes a pressure plate as "flat ring, OD × ID × thickness" without mentioning fingers, deflection, or spring rate is already a design failure at the params level. Norman catches it here, before a single line of geometry code is written. This review is lightweight and non-blocking — findings feed directly into the sketch scripts.

   **b. Sketch production (representative shapes):** For each part, Norman writes a quick script that represents the part as a simplified but *representative* shape. Not arbitrary boxes. A woodruff key sketch is a half-moon. A pressure plate finger sketch is a cantilevered beam. A drive dog sketch has a realistic engagement profile. The sketch should make someone say "I can see what this part does" — not "what is this rectangle?" Simplification means no fillets, no bolt holes, no cosmetic features — it does not mean stripping away the functional character of the part. Norman also describes the design intent in natural language: what each part should look like, how parts relate visually, what the overall proportions should convey. Norman's output is both geometry (quick STEP files) and prose (design notes).

   **c. Design Critique (after sketch assembly):** After the sketch assembly is built and interference checks have run (Section 7.4), Norman produces a Design Critique document — the "this doesn't look right" mechanism. This is the feedback that real design teams give in sketch reviews: spatial concerns, functional concerns, pride test failures, and aesthetic issues captured as actionable items. Examples: "These pressure plate fingers need to be cantilevered beams, not flat plates, or they cannot produce spring force." "The drive dog profile should have draft angles for engagement/disengagement." "The woodruff key is a rectangle — it should be a half-moon shape that a machinist would recognize." Without Design Critique, problems hide until production geometry is complete and expensive to fix.

2. **Steltzner (engineer):** Reviews Norman's sketching study for engineering feasibility. Runs interference checks on the sketch assembly (Section 7.4). Flags any visual concepts that conflict with mechanical requirements, material constraints, or assembly sequence. Provides engineering feedback before the study goes to the human. This is a collaborative exchange, not a gate — Norman and Steltzner iterate until the concept is both visually sound and mechanically feasible.

3. **Deming (manager):** Reviews the Norman-Steltzner output, evaluates whether it is ready for the human, and presents the sketching study at the gate.

**Output:**
- Norman's params review findings (per-part functional validity assessment)
- One assembly STEP file with all parts as simplified but representative shapes in their assembly positions
- PNG render + MP4 turntable of the sketch assembly
- Optional: 2D matplotlib section plots (XZ side view, XY plan view) showing part outlines
- Norman's design notes with natural language descriptions of each part's visual intent
- Norman's Design Critique document (actionable items from sketch review)
- Steltzner's engineering feedback
- Interference check results on sketch geometry (Section 7.4)

**Gate:** Norman's Design Critique items and interference check failures must be resolved — by parameter change, placement change, or explicit deferral with rationale — before proceeding to production geometry. The human reviews the sketch renders, Norman's descriptions, the Design Critique, and the interference report. Feedback shapes the detailed geometry that follows. The sketching study is fast enough (~30 minutes) that multiple iterations are practical.

**When to use:**
- Before the first production geometry step of any multi-part assembly project
- When a design change affects the visual character of multiple parts
- When the human requests visual validation before detailed work

**When to skip:**
- Single-part projects where the part geometry is obvious from dimensions
- Steps that modify existing geometry without changing the visual concept
- When the human explicitly says to proceed without visual exploration

#### 8.4.1 Cross-Section Projection Checklist

Pregame cross-section plots (matplotlib) must use correct engineering drawing projection. Cylindrical features are the most common source of error: an LLM generating plot code defaults to circle patches for cylinders regardless of view orientation.

**Rule:** In axial cross-sections (R vs Z, side view), ALL cylindrical features — pins, shafts, bearings, bolts, bores — appear as **rectangles** (width = diameter, height = length along Z). Circles appear ONLY in radial cross-sections (X vs Y, looking along the bore axis). Spheres appear as circles in both views. Tapered features appear as trapezoids in the view showing the taper.

**Checklist (enforce in every pregame plot prompt):**
- [ ] Identify which view direction (axial side view R-vs-Z, or radial end view X-vs-Y)
- [ ] For each cylindrical feature, determine its axis orientation
- [ ] Cylinder axis parallel to view direction → **circle** (you see the end)
- [ ] Cylinder axis perpendicular to view direction → **rectangle** (you see the side)
- [ ] Never use a circle patch for a cylinder seen from the side
- [ ] Bearings in side view: nested rectangles (outer race, inner race), not circles
- [ ] Bolts/studs in side view: rectangle with width = nominal diameter

**Source:** WO-2026-002 Step 5D pregame plots. Axial cross-sections used circles for cylindrical objects that should have been rectangles.

#### 8.4.2 Pregame Plot Conventions

Standard conventions for all pregame matplotlib cross-section plots:

- **Figure size:** `figsize=(14, 10)` minimum for axial, `figsize=(10, 10)` for radial (square)
- **DPI:** 150 for screen review, 300 for documentation
- **Color scheme:** Distinct colors per material/part-type. Label in legend.
- **Labeling:** Every feature labeled with name + key dimension (e.g., "Bearing 6006-2RS, OD=55"). Use `annotate()` with arrows for crowded regions.
- **Scale:** Equal aspect ratio (`ax.set_aspect('equal')`). Never distort proportions.
- **Grid:** On, with minor gridlines for reading dimensions
- **Title:** Include view type ("Axial Cross-Section R vs Z" or "Radial Cross-Section X vs Y")

---

## 9. Visualization and Rendering

The CAD workflow generates FCStd files as the authoritative geometric artifacts and STEP files as the export format for sharing and verification. Neither format is visually inspectable without a viewer application. The visualization pipeline bridges this gap: it produces PNG renders and MP4 turntable videos from STEP or FCStd files without requiring a GUI, making visual output available in the terminal, in pull requests, and in documentation deliverables.

### 9.1 Why Visualization Matters

Git diffs show that a fillet radius changed from 2 mm to 3 mm. They do not show what the resulting shape looks like. A headless rendering pipeline partially addresses this gap by generating visual output automatically after every geometry change. Automated renders are not a complete substitute for interactive inspection — sub-millimeter positioning errors, degenerate faces, and interference between closely mated parts still require loading the STEP file in a viewer and measuring directly. But renders catch gross errors (missing features, wildly wrong dimensions, parts in the wrong place) without requiring the engineer to open a viewer for every regeneration.

For documentation deliverables (assembly instructions, print guides), rendered images are essential. Assembly instructions require step-by-step diagrams showing each assembly operation from the student's or technician's perspective. These must be producible programmatically so they update when the design changes.

### 9.2 The Rendering Pipeline

The standard rendering tool is `render_step.py`, which uses VTK (Visualization Toolkit) for headless 3D rendering from STEP files.

**Dependencies:**
- Python with VTK (`import vtk`)
- ffmpeg (for MP4 turntable video generation)

**Outputs per STEP file:**
- **PNG render:** A single wireframe-from-edges image. Provides quick visual verification without opening a viewer. Generated alongside the STEP file in the project's `output/` directory.
- **MP4 turntable video:** A 360-degree rotation showing the part or assembly from all angles. Useful for sharing and review. Requires ffmpeg. Use H.264 codec (libx264), not H.265/HEVC — H.265 does not embed in Discord and many other platforms.

**Usage:**
```bash
python render_step.py path/to/part.step
# Outputs: path/to/part.png, path/to/part.mp4
```

**When to render:**
- After generating or regenerating any STEP file (part or assembly)
- After assembly script changes that affect component placement
- When producing documentation that requires visual output

### 9.3 Instructional Rendering

Assembly instructions and print guides require more than turntable renders. They need specific camera angles showing specific assembly operations (e.g., "attach part A to part B from this direction"). This requires either:

1. **Scripted instructional views:** Extend the rendering script to accept camera position, target, and part visibility parameters. The assembly script or a separate documentation script defines a sequence of views (one per assembly step), each specifying which parts are visible, where the camera is, and what annotations to show. This is the most maintainable approach — views update automatically when the design changes.

2. **Manual STEP viewer screenshots:** Open the assembly in FreeCAD or CAD Assistant, hide/show parts to match each assembly step, and capture screenshots. This requires no scripting but is manual labor that must be repeated if the design changes. Acceptable for a first pass but does not scale.

3. **FreeCAD SVG export:** FreeCAD's TechDraw workbench can export 2D projected views as SVG line art. Clean and scriptable but lacks shading and color, which can make it hard to distinguish parts in an assembly view.

**Recommendation:** Start with scripted VTK rendering for turntable and beauty-shot views. For instructional step-by-step views, extend the renderer with camera and visibility parameters during the assembly step (not during documentation). If the rendering pipeline is not available, fall back to manual screenshots from a STEP viewer and document the procedure.

### 9.4 Render Tooling in the Project

`render_step.py` should be present in the project repository or accessible from the working directory. The prompt0 self-check verifies its availability. If it is not present:

1. Check if it exists in a sibling project or shared tools directory.
2. If not found, write a minimal VTK rendering script that loads a STEP file via OCCT, converts to VTK polydata, and renders to PNG. The core logic is approximately 50 lines of Python using `OCP` for STEP loading and `vtk` for rendering.
3. If VTK is not available, fall back to manual STEP viewer screenshots and document the procedure for reproducibility.

The rendering pipeline is recommended but not blocking. FCStd files are the authoritative design artifacts; STEP files are the export format. Renders provide convenience and documentation support. A project that produces correct FCStd and STEP files but no renders is complete; a project that produces beautiful renders from incorrect geometry is not.

---

## 10. Outputs Checklist

After processing a work order or change order, the following outputs should exist or be updated:

**For work orders (new parts/assemblies):**
- [ ] `params.yaml` for each new part
- [ ] FreeCAD build script (`.py`) for each new part
- [ ] Generated FCStd files in `parts_cache/` and STEP exports in `output/`
- [ ] `assemble.py` updated (or created) with new components
- [ ] Assembly FCStd and STEP export regenerated
- [ ] `ASSEMBLY.md` updated with new parts, dependencies, assembly sequence
- [ ] `BOM.yaml` regenerated (if applicable)
- [ ] Test suite written and validated
- [ ] Exploratory build completeness gate passed (Section 7.6) — **mandatory** — all functional features present
- [ ] Production build quality gate passed (Section 7.6) — **mandatory** — all features deployment-ready
- [ ] Interference check passed on production assembly (Section 7.4) — **mandatory automated gate**
- [ ] Functional validity tests passed (Section 7.1) — **mandatory**
- [ ] FCStd verified in FreeCAD GUI; STEP verified in viewer (product tree, dimensions, visual inspection)
- [ ] All text files committed to Git
- [ ] PNG renders generated via render_step.py (**Recommended** -- requires VTK; FCStd is the authoritative artifact, PNG provides quick visual verification without opening FreeCAD)
- [ ] MP4 turntable video generated via render_step.py (**Recommended** -- requires VTK + ffmpeg; useful for sharing and review but not required for geometry correctness)

**For change orders:**
- [ ] `CO-NNN.md` (change request)
- [ ] `CO-NNN_propagation.md` (impact analysis, reviewed and approved)
- [ ] `params.yaml` updated for all affected parts
- [ ] FCStd files regenerated for all affected parts; STEP exports updated
- [ ] `assemble.py` updated if assembly positioning affected
- [ ] Assembly FCStd and STEP export regenerated
- [ ] `ASSEMBLY.md` updated (current values, change order history)
- [ ] `BOM.yaml` regenerated (if applicable)
- [ ] Test suite updated and validated
- [ ] Exploratory build completeness gate passed (Section 7.6) — **mandatory** — all functional features present
- [ ] Production build quality gate passed (Section 7.6) — **mandatory** — all features deployment-ready
- [ ] Interference check passed on production assembly (Section 7.4) — **mandatory automated gate**
- [ ] Functional validity tests passed (Section 7.1) — **mandatory**
- [ ] FCStd verified in FreeCAD GUI; STEP verified in viewer (product tree, dimensions, visual inspection)
- [ ] All text files committed to Git
- [ ] For topological changes: face selectors verified, all parent assemblies re-verified
- [ ] PNG renders regenerated via render_step.py (**Recommended**)
- [ ] MP4 turntable video regenerated via render_step.py (**Recommended**)

---

## 11. Known Boundaries

These are the limits of this method. Do not pretend they don't exist.

1. **Geometry complexity.** Validated only on prismatic and revolved forms. Helical swept forms were attempted but revealed an OCCT boolean limitation: BSpline-swept solids cut against analytical surfaces can silently fail (imprint edges without removing material). When performing these operations, calculate the volume of both boolean solids before the operation to ensure the addition or subtraction works as expected. If possible, cut the part primatives before combining/unioning them (i.e., cut the threads into a "stud" before adding the prismatic bolt head). Flag complex operations like this for user review at the appropriate step gate. 

2. **Assembly scale.** FreeCAD's native Assembly solver (Ondsel MbD) tested with up to 16 parts and grounded joints. Behavior with dozens of interdependent parametric constraints (revolute, fixed) is an open question. Fallback: explicit XYZ placements on App::Link objects (deterministic but requires manual placement math).

3. **Scalar constraints.** ASSEMBLY.md's parametric dependency notation cannot express derived relationships, conditional constraints (ISO table lookups), tolerance bands, or summation constraints (stack-up calculations). The LLM should flag ambiguity, not resolve it.

4. **Standard-table coupling.** A diameter change from M8 to M10 implies ISO standard-table lookups that the LLM may or may not handle correctly. The engineer's review at step 4 of the change order workflow is the safety net.

5. **Silent geometry errors.** Some errors (inverted thread profiles, insufficient BSpline sampling, subtle misplacement) produce valid FCStd and STEP files that look correct at a glance. Volume comparison and careful viewer inspection are mandatory.

6. **Partial automated cross-part validation.** `validate_params()` enforces scalar-computable cross-part constraints (clearance ranges, pitch matching) at generation time. Non-scalar constraints (coaxiality, thread form compatibility, assembly sequence) are checked by LLM analysis and human review, not by code. Full automated enforcement of non-scalar constraints would require a constraint solver and formal schema that do not exist yet.

7. **Functional validity is a judgment call, not fully automatable.** A reviewing persona must correctly identify non-functional geometry — there is no algorithm for "does this look like a real part." The defense-in-depth: (1) Norman's params review catches placeholder descriptions before geometry exists (Section 8.4a), (2) Norman's Design Critique catches non-representative shapes after the sketch assembly (Section 8.4c), (3) functional validity tests define "representative" per part type with specific criteria (Section 7.1), (4) human review at the step gate catches what the LLM missed. This layered approach acknowledges that no single check is sufficient. The standard is not "does this pass a test?" but "would this part look at home in a product from a world-class design house?"
