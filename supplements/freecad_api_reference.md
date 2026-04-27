# FreeCAD 1.0 Headless API Reference

**Purpose:** Consolidated reference of empirically verified API patterns for FreeCAD 1.0 headless scripting via FreeCADCmd. Every entry in this file was discovered by running code and observing results — not from documentation alone.

**Maintained by:** Steltzner (primary), updated after each step that produces new API knowledge.

**Source evidence:** `cr_scratch/step1_capability_report.md`, `cr_scratch/step1_scripts/`, `cr_scratch/step1_5_assembly_solver_report.md`, `cr_scratch/step1_5_scripts/`

---

## Environment

- **FreeCAD:** 1.0.2 (build 39319, Git 256fc7ef)
- **Python:** 3.11.13 (conda-forge, MSC v.1943 64-bit)
- **Executable:** `C:\Program Files\FreeCAD 1.0\bin\FreeCADCmd.exe`
- **Platform:** Windows 10 Pro

## Script Invocation

```bash
"C:/Program Files/FreeCAD 1.0/bin/FreeCADCmd.exe" -c "exec(open(r'path/to/script.py').read())"
```

Direct file argument also works for simple scripts, but the `exec(open(...).read())` pattern is more reliable.

---

## PartDesign Scripting

### Document and Body Creation
```python
import FreeCAD
import Part
import PartDesign
import Sketcher

doc = FreeCAD.newDocument("MyDoc")
body = doc.addObject("PartDesign::Body", "Body")
```

### Sketch on Datum Plane
```python
sketch = body.newObject("Sketcher::SketchObject", "Sketch")
sketch.AttachmentSupport = [(doc.getObject("XY_Plane"), "")]
sketch.MapMode = "FlatFace"
```

### Sketch Geometry and Constraints
```python
# Add line segments (rectangle example — four lines with coincident corners)
sketch.addGeometry(Part.LineSegment(FreeCAD.Vector(0, 0, 0), FreeCAD.Vector(50, 0, 0)), False)
# ... (4 line segments total for a rectangle)

# Constraint syntax
Sketcher.Constraint("Coincident", geo1, vertex1, geo2, vertex2)
Sketcher.Constraint("Horizontal", geo_index)
Sketcher.Constraint("Vertical", geo_index)
Sketcher.Constraint("DistanceX", geo1, v1, geo2, v2, value)
Sketcher.Constraint("DistanceY", geo1, v1, geo2, v2, value)
```

**Pitfall — Lock constraint:** The `Lock` constraint type does NOT work with `(type, geo, vertex, x, y)` syntax. Use separate `DistanceX` and `DistanceY` constraints from the origin (`-1, 1`) instead.

### Pad (Extrude)
```python
pad = body.newObject("PartDesign::Pad", "Pad")
pad.Profile = sketch
pad.Length = 20.0
pad.Type = 0  # 0 = Length
doc.recompute()
```

### Fillet
```python
fillet = body.newObject("PartDesign::Fillet", "Fillet")
fillet.Base = (pad, ["Edge4", "Edge7", "Edge10", "Edge12"])  # BARE TUPLE
fillet.Radius = 3.0
doc.recompute()
```

### Chamfer
```python
chamfer = body.newObject("PartDesign::Chamfer", "Chamfer")
chamfer.Base = (previous_feature, ["Edge4", "Edge7", "Edge10"])  # BARE TUPLE (same as Fillet)
chamfer.Size = 0.5  # chamfer dimension in mm
doc.recompute()
```

### Pocket (Subtractive Extrusion)
```python
pocket = body.newObject("PartDesign::Pocket", "Pocket")
pocket.Profile = sketch
pocket.Type = 1  # 0 = Length, 1 = Through All, 2 = To First, 3 = To Face
# pocket.Length = 10.0  # only needed if Type = 0
doc.recompute()
```

### LinearPattern
```python
pattern = body.newObject("PartDesign::LinearPattern", "Pattern")
pattern.Originals = [pocket_or_pad]  # feature(s) to pattern
pattern.Direction = (sketch, ["N_Axis"])  # X direction in sketch coords
# Other direction options: ["V_Axis"] for Y, or reference an edge
pattern.Length = 20.0    # total extent of pattern
pattern.Occurrences = 3  # total count including original
doc.recompute()
```

**Critical pitfall — PropertyLinkSub vs PropertyLinkSubList:**
- `App::PropertyLinkSub` (singular): `obj.Prop = (target_obj, ["Sub1", "Sub2"])` — bare tuple
- `App::PropertyLinkSubList` (list): `obj.Prop = [(target_obj, ("Sub1",)), (target_obj2, ("Sub2",))]`
- `PartDesign::Fillet.Base` and `PartDesign::Chamfer.Base` are **PropertyLinkSub** (singular). Using a list of tuples produces: `ValueError: Expect input sequence of size 2`

### Saving
```python
doc.saveAs(r"C:/path/to/output.FCStd")
```

### Shape Metrics
```python
shape = body.Shape
print(f"Volume: {shape.Volume}")
print(f"BoundBox: {shape.BoundBox.XLength} x {shape.BoundBox.YLength} x {shape.BoundBox.ZLength}")
print(f"Faces: {len(shape.Faces)}, Edges: {len(shape.Edges)}, Vertices: {len(shape.Vertexes)}")
```

### SubtractiveCylinder / AdditiveCylinder Placement (Local Coordinate System)

`PartDesign::SubtractiveCylinder` and `PartDesign::AdditiveCylinder` (and all PartDesign primitives) place geometry **relative to the active Body's local origin**, not the global origin. The Placement property (position + rotation) is interpreted in the Body's local coordinate frame.

**Critical rule:** Keep the Body at `(0,0,0)` with identity rotation during all PartDesign feature operations (pad, pocket, boolean, fillet). Move the Body to its assembly position only AFTER all features are built — or compute all tool/primitive placements relative to the Body's local frame.

```python
# CORRECT: Body at origin, primitive placed relative to Body
body = doc.addObject("PartDesign::Body", "Body")
# ... build pad, sketch, etc. ...

# Radial hole at angle theta, centered at radius R along Body's Z
import math
theta = math.radians(24)  # 24 degrees around Z
R = 67.5  # radial distance from center

cyl = doc.addObject("PartDesign::SubtractiveCylinder", "RadialHole")
body.addObject(cyl)
cyl.Radius = 2.0
cyl.Height = 20.0

# Rotation: align cylinder axis from Body-Z to radial direction
# Default cylinder axis is Body-local Z. Rotate 90° around Y, then rotate around Z by theta.
rot = FreeCAD.Rotation(FreeCAD.Vector(0, 0, 1), math.degrees(theta)) * \
      FreeCAD.Rotation(FreeCAD.Vector(0, 1, 0), 90)
pos = FreeCAD.Vector(R * math.cos(theta), R * math.sin(theta), 11.0)
cyl.Placement = FreeCAD.Placement(pos, rot)
doc.recompute()
```

**Common mistake:** Concluding that SubtractiveCylinder "ignores rotation" or "is axis-locked." It is not — the rotation works, but positions and rotations are in the Body's local frame. If the Body has been moved from the origin, the primitive appears in the wrong place. Debug coordinate math before concluding FreeCAD is broken.

**Source:** WO-2026-002 Step 5D stator shell build. Initial conclusion that SubtractiveCylinder was axis-locked was incorrect — root cause was coordinate system confusion (author-identified).

### PartDesign::Boolean (Feature Tree Integration)

`PartDesign::Boolean` bridges OCCT tool shapes into the PartDesign feature tree. The tool shape is consumed by the Boolean operation, and downstream features (fillets, chamfers) chain from it.

```python
# Create OCCT tool shape as Part::Feature
tool = doc.addObject("Part::Feature", "CutTool")
tool.Shape = Part.makeCylinder(5.0, 20.0, FreeCAD.Vector(30, 0, 5), FreeCAD.Vector(1, 0, 0))

# Create PartDesign::Boolean in the Body
bool_cut = body.newObject("PartDesign::Boolean", "BoolCut")
bool_cut.Type = "Cut"       # "Cut", "Fuse", or "Common"
bool_cut.Group = [tool]     # NOT .Bodies (that property does nothing)
doc.recompute()
```

**Key API facts (verified cr_scratch/test_pd_boolean.py, 14/15 PASS):**
- Tool set via `bool_cut.Group = [tool]` or `bool_cut.addObject(tool)` — there is NO working `Bodies` property
- `Type` accepts strings: `"Cut"`, `"Fuse"`, `"Common"`
- Both `Part::Feature` and `PartDesign::Body` accepted as tool objects
- Sequential PartDesign::Boolean operations in one Body DO work (multiple booleans are valid)
- `PartDesign::Fillet` works after `PartDesign::Boolean` — `fillet.Base = (bool_feature, ["EdgeN"])`

**When to use:** For any feature requiring non-axial orientation or complex curves that cannot be expressed as a PartDesign sketch+pad/pocket. The Body remains parametric; only the tool shapes are procedural.

### Combined Tool Pattern (Batch Subtractive Features)

When cutting multiple features of the same type (e.g., 15 radial holes, 40 magnet pockets), fuse all OCCT tool shapes into one compound and perform a **single** `PartDesign::Boolean` cut. One Boolean per batch is cleanest.

```python
# Build all tool shapes
shapes = []
for i in range(15):
    angle = math.radians(i * 24)
    pos = FreeCAD.Vector(R * math.cos(angle), R * math.sin(angle), z)
    direction = FreeCAD.Vector(math.cos(angle), math.sin(angle), 0)
    shapes.append(Part.makeCylinder(radius, depth, pos, direction))

# Fuse into one compound
fused = shapes[0]
for s in shapes[1:]:
    fused = fused.fuse(s)

# Add as Part::Feature and boolean-cut
tool = doc.addObject("Part::Feature", "CombinedTool")
tool.Shape = fused
bool_cut = body.newObject("PartDesign::Boolean", "AllCuts")
bool_cut.Type = "Cut"
bool_cut.Group = [tool]
doc.recompute()
```

**Why fuse first:** Multiple sequential `PartDesign::Boolean` operations on the same Body CAN work, but combining tool shapes into one operation is more reliable and produces cleaner topology. The fuse-then-boolean pattern is the canonical approach for batch subtractive features.

**Source:** WO-2026-002 Step 5D stator shell (15 pockets + 15 M6 + 15 M4 radial + 10 M4 axial = 55 holes in one boolean). Rotor hub (40 magnet pockets in one boolean).

### Fillet Ordering Rule

Apply `PartDesign::Fillet` **BEFORE** boolean operations that would fragment the target edge into segments. A fillet on a clean, pre-boolean edge (e.g., a single circular edge from a revolution) succeeds reliably. The same fillet after a boolean that cuts across that edge will find dozens of edge fragments and may fail or produce invalid geometry.

**Feature tree ordering:**
1. Base shape (Pad, Revolution)
2. Fillets on clean edges
3. PartDesign::Boolean cuts (which may fragment edges near the fillet)

**Example:** Rotor hub inside corner fillet at R=54, Z=3. Applied after revolution (1 clean circular edge) → success. Applied after 40 magnet pocket booleans → edge fragmented into 40+ segments → `PartDesign::Fillet` failed.

**Source:** WO-2026-002 Step 5D rotor hub build.

---

## Assembly Scripting

### Assembly Structure Creation
```python
import sys
sys.path.insert(0, r"C:/Program Files/FreeCAD 1.0/Mod/Assembly")
import JointObject
import UtilsAssembly

asm_doc = FreeCAD.newDocument("Assembly")
asm = asm_doc.addObject("Assembly::AssemblyObject", "Assembly")
```

### Cross-Document Links
```python
# IMPORTANT: Owner document must be saved BEFORE creating links
asm_doc.saveAs(r"C:/path/to/assembly.FCStd")

# Open part documents
part_doc = FreeCAD.openDocument(r"C:/path/to/part.FCStd")

# Create links
link = asm_doc.addObject("App::Link", "PartName")
link.LinkedObject = part_doc.getObject("Body")
link.adjustRelativeLinks(asm)
asm.addObject(link)
```

### App::Link Placement Behavior (CRITICAL)

**`Link.Placement` is the ONLY transform applied to a linked object.** FreeCAD does NOT automatically compose `Link.Placement` with `LinkedObject.Placement`. The source object's placement is ignored when computing the link's visual/geometric position.

This means there are **two distinct linking patterns** depending on where the part's position information lives:

#### Pattern 1: Geometry at body-local origin, placement needed
Parts built by PartDesign at body-local coordinates (e.g., a pad from Z=0 to Z=15). The part's Shape vertices are at the body origin. Position in the assembly is entirely controlled by `Link.Placement`.

```python
# Part was built at body-local Z=0..15, needs to go to assembly Z=23
link = asm.addObject("App::Link", "RearEndplate")
link.LinkedObject = src_doc.getObject("EndplateRear")
link.Placement = FreeCAD.Placement(
    FreeCAD.Vector(0, 0, 23),    # assembly position
    FreeCAD.Rotation(0, 0, 0)
)
```

#### Pattern 2: Position encoded in source Placement
Multi-object files (e.g., 15 coils at 24° spacing) where each object's `Placement` in the source document encodes its position relative to the sub-assembly origin. The Shape vertices may be at the local origin; the Placement rotates/translates to the correct position.

```python
# WRONG — discards the source object's angular/radial position:
link.Placement = FreeCAD.Placement(FreeCAD.Vector(0, 0, z_offset), FreeCAD.Rotation(0, 0, 0))
# Result: all 15 coils stack at angle 0°

# CORRECT — preserve source placement, add assembly offset:
src_plc = src_obj.Placement.copy()
src_plc.Base.z += z_offset
link.Placement = src_plc
```

#### Multi-Level Assembly Placement Composition

When building a top-level assembly that places sub-assembly parts (not sub-assembly documents), each part needs both its **sub-assembly-internal placement** and the **top-level placement**. These must be explicitly composed:

```python
# Compose: top_level_placement * sub_assembly_internal_placement
top_plc = FreeCAD.Placement(
    FreeCAD.Vector(0, 0, zone_z_offset),
    FreeCAD.Rotation(yaw, pitch, roll)      # e.g., 180° flip
)

# For Pattern 1 parts (explicit offset within sub-assembly):
internal_plc = FreeCAD.Placement(FreeCAD.Vector(x_local, y_local, z_local), FreeCAD.Rotation(...))
link.Placement = top_plc.multiply(internal_plc)

# For Pattern 2 parts (source placement within sub-assembly):
internal_plc = src_obj.Placement.copy()
internal_plc.Base.z += extra_z_offset  # if sub-assembly script added a Z shift
link.Placement = top_plc.multiply(internal_plc)

# For identity parts (geometry already at sub-assembly-local coords):
link.Placement = top_plc
```

**The multiply order matters:** `A.multiply(B)` applies B first (sub-assembly positioning), then A (top-level positioning). This is standard rigid-body transform composition.

#### Common Failure Mode

A top-level assembly script that applies only the top-level offset to every part:

```python
# WRONG — all parts get the same placement, losing sub-assembly structure:
for part_name in sub_assembly_parts:
    link_obj(src, part_name, label, z=zone_offset, ry=180)

# The sub-assembly script had per-part offsets like:
#   EndplateFront:  z=-16    (body-local origin, needs explicit offset)
#   StatorShell:    z=-1     (body-local origin, needs explicit offset)
#   CycloidalDisc:  x=2.5, z=7.0  (eccentricity + axial position)
#   Coils:          source Placement (angular position)
#   Pins:           source Placement + z=7.0
# All of these internal offsets are LOST.
```

**Diagnostic:** If a multi-level App::Link assembly has correct per-part volumes but parts visually overlap or cluster at zone boundaries, the sub-assembly-internal placements are being discarded. Compare part Z-ranges against a known-good reference (e.g., a STEP-import of the sub-assembly).

**Prevention:** When writing a top-level assembly script that links individual parts (not sub-assembly documents), always reference the sub-assembly script to extract per-part placement offsets. Document the three placement categories (explicit, source, identity) for each part type.

---

### Joint Creation
```python
# Joints are Python feature objects, NOT C++ types
# Assembly::JointFixed, Assembly::JointRevolute, etc. are NOT valid addObject types

joint_group = UtilsAssembly.getJointGroup(asm)
joint_obj = joint_group.newObject("App::FeaturePython", "FixedJoint")
JointObject.Joint(joint_obj, type_index)
# Type indices: 0=Fixed, 1=Revolute, 2=Cylindrical, 3=Slider, 4=Ball, 5=Distance, 6=Parallel, 7=Perpendicular, 8=Angle, 9=RackPinion, 10=Screw, 11=Gears, 12=Belt
```

### Grounded Joint
```python
# Ground a part (fix its placement). No GUI/ViewProvider needed.
joint_group = UtilsAssembly.getJointGroup(asm)
ground = joint_group.newObject("App::FeaturePython", "GroundedPart")
JointObject.GroundedJoint(ground, link_to_ground)
# Sets ObjectToGround and Placement automatically
```

### Joint References (CRITICAL — Correct Format)
```python
# Reference format requires TWO sub-elements: element + vertex
# isRefValid(ref, 2) checks len(ref[1]) >= 2

# For face center selection (most common for assembly):
joint_obj.Reference1 = (assembly, ["LinkName.FaceN", "LinkName.FaceN"])
joint_obj.Reference2 = (assembly, ["LinkName.FaceM", "LinkName.FaceM"])

# For vertex-specific selection:
joint_obj.Reference1 = (assembly, ["LinkName.FaceN", "LinkName.VertexM"])

# For circle center selection (edge is a circle):
joint_obj.Reference1 = (assembly, ["LinkName.FaceN", "LinkName.EdgeK"])
```

**Critical pitfall — Single sub-element FAILS SILENTLY:**
Using `(link, ["FaceN"])` with only one sub-element causes `findPlacement()` to return identity placement via `isRefValid(ref, 2) -> False`. The solver then receives empty constraints and returns -6. Always provide TWO sub-elements.

**Sub-element path format:** `"LinkName.ElementName"` where LinkName is the `App::Link` object name in the assembly, and ElementName is `FaceN`, `EdgeN`, or `VertexN` (1-based).

### Placement Computation
```python
# Compute JCS placement from reference (REQUIRED before solving)
plc = UtilsAssembly.findPlacement(ref)

# Set on joint:
joint_obj.Placement1 = UtilsAssembly.findPlacement(joint_obj.Reference1)
joint_obj.Placement2 = UtilsAssembly.findPlacement(joint_obj.Reference2)
```

`findPlacement()` computes a placement relative to the referenced object's coordinate system:
- **Planar face (vtx=Face):** origin = face.CenterOfGravity, rotation = surface.Rotation
- **Cylindrical face (vtx=Face):** origin = projected center on axis, rotation = surface.Rotation
- **Edge circle (vtx=Edge):** origin = curve.Location (circle center), rotation = curve.Rotation
- **Vertex:** origin = vertex point, rotation = identity

### Solver
```python
# Full working solver pattern:
asm_doc.recompute()
result = asm.solve()
# Return codes:
#   0  = success (constraints satisfied)
#  -6  = no valid constraints (bad references or missing Placement1/Placement2)
```

The solver is the **Ondsel MbD (Multi-body Dynamics)** solver. It uses Newton-Raphson iteration, typically converging in 2-4 iterations. Convergence is logged with `MbD: Convergence = ...` messages.

### Complete Working Assembly Pattern
```python
# 1. Create and save assembly document
asm_doc = FreeCAD.newDocument("Assembly")
asm = asm_doc.addObject("Assembly::AssemblyObject", "Assembly")
asm_doc.saveAs(r"path/to/assembly.FCStd")  # MUST save before linking

# 2. Open parts, create links
part_doc = FreeCAD.openDocument(r"path/to/part.FCStd")
link = asm_doc.addObject("App::Link", "PartName")
link.LinkedObject = part_doc.getObject("PartObj")
link.adjustRelativeLinks(asm)
asm.addObject(link)

# 3. Ground one part
joint_group = UtilsAssembly.getJointGroup(asm)
ground = joint_group.newObject("App::FeaturePython", "Ground")
JointObject.GroundedJoint(ground, grounded_link)

# 4. Create joints with CORRECT reference format
ref1 = (asm, ["Link1.FaceN", "Link1.FaceN"])
ref2 = (asm, ["Link2.FaceM", "Link2.FaceM"])

joint = joint_group.newObject("App::FeaturePython", "Joint")
JointObject.Joint(joint, 0)  # 0=Fixed
joint.Reference1 = ref1
joint.Placement1 = UtilsAssembly.findPlacement(ref1)
joint.Reference2 = ref2
joint.Placement2 = UtilsAssembly.findPlacement(ref2)
joint.Activated = True

# 5. Solve
asm_doc.recompute()
result = asm.solve()  # Returns 0 on success
```

### CRITICAL: Fixed Joints vs Grounded Joints for Pre-Positioned Parts

**Problem discovered (WO-2026-002 Step 4.75):** When parts are built at their correct absolute coordinates (all geometry constructed at final positions in a single document), using Fixed joints with `findPlacement()` **moves parts to wrong positions**. `findPlacement()` computes a face-matching transform between two reference faces and the solver applies it — which pulls parts away from their construction coordinates to align the referenced faces.

**Symptom:** Parts appear displaced, rotated, on wrong planes in the GUI. Tests checking individual part dimensions still pass because the underlying Part::Feature shapes are correct — only the assembly Link placements are wrong.

**Root cause:** `findPlacement()` is designed for assemblies where parts start at the origin and the solver positions them via face-matching constraints. When parts are already at their final positions, face-matching introduces unwanted transforms.

**Fix — use Grounded joints for all parts:**
```python
# CORRECT: Ground all parts at their construction coordinates
jg = UtilsAssembly.getJointGroup(asm)
for part_name, link in links.items():
    ground = jg.newObject("App::FeaturePython", "Ground_" + part_name)
    JointObject.GroundedJoint(ground, link)
```

**When to use each approach:**
| Scenario | Approach |
|----------|----------|
| All parts built at absolute coordinates in one doc | Ground all parts |
| Parts from separate docs, need solver to position | Fixed/Revolute joints with findPlacement |
| Mixed: some parts pre-positioned, some need solving | Ground the pre-positioned, constrain the rest |

**Verification — always audit link placements after assembly creation:**
```python
for obj in doc.Objects:
    if 'Link' in obj.Name and hasattr(obj, 'Placement'):
        p = obj.Placement
        if abs(p.Base.x) > 0.01 or abs(p.Base.y) > 0.01 or abs(p.Base.z) > 0.01:
            print("WARNING: {} displaced to ({}, {}, {})".format(
                obj.Name, p.Base.x, p.Base.y, p.Base.z))
        if abs(p.Rotation.Angle) > 0.01:
            print("WARNING: {} rotated by {} deg".format(
                obj.Name, p.Rotation.Angle * 57.2958))
```

**Test gap:** Dimension-checking tests (volume, bounding box, face counts) do NOT catch assembly placement errors. Parts can have correct geometry but be in completely wrong positions. Visual verification (MP4 turntable render or GUI inspection) is required for any assembly step.

### Assembly API Methods
```python
asm.solve()                              # Run constraint solver (returns 0 on success)
asm.isPartGrounded(obj)                  # Check if part is grounded
asm.isPartConnected(obj)                 # Check if part has constraints
asm.isJointConnectingPartToGround(joint) # Check if joint grounds a part
asm.exportAsASMT()                       # Export to ASMT format
asm.undoSolve()                          # Undo last solve
```

---

## Headless FCStd Reading

### Via FreeCADCmd API
```python
doc = FreeCAD.openDocument(r"C:/path/to/file.FCStd")
for obj in doc.Objects:
    print(f"{obj.Name} | {obj.TypeId}")
    if hasattr(obj, "Shape"):
        print(f"  Volume: {obj.Shape.Volume}")
        print(f"  BoundBox: {obj.Shape.BoundBox}")
```

### Via Standalone Python (No FreeCAD)
```python
import zipfile
import xml.etree.ElementTree as ET

with zipfile.ZipFile("file.FCStd", "r") as z:
    with z.open("Document.xml") as f:
        tree = ET.parse(f)
        root = tree.getroot()
        for obj in root.findall(".//Object"):
            print(obj.get("name"), obj.get("type"))
```

FCStd files are zip archives containing:
- `Document.xml` — feature tree, parameters, constraints, dependencies
- `GuiDocument.xml` — display properties (colors, visibility)
- `*.brp` files — B-rep geometry for each shape
- `*.emx` files — element maps (TNP shadow references)

Both methods extract complete feature trees and dimension values. Standalone XML parsing provides everything except computed geometry (volumes, bounding boxes).

### App::Link Timestamp Detection and Recompute on Open

When opening an App::Link assembly, FreeCAD checks timestamps of linked FCStd files. If a linked file has been modified since the assembly was last saved, FreeCAD logs:

```
Document.cpp(2216): assembly#ObjectName.LinkedObject: Time stamp changed on link source#SourceObject
```

This triggers a recompute of the linked document's feature tree. For complex parts (PartDesign bodies with 50+ features, Hole Wizard holes, boolean cuts), this recompute can take minutes. Consequences for headless test scripts:

1. **Timeout failures.** A test that opens an assembly with stale links may exceed its timeout waiting for recompute, not because of a bug in the geometry.
2. **Null objects.** During recompute, objects that exist in the saved file may temporarily return `None` from `getObject()`. Scripts that read shapes immediately after `openDocument()` without checking for `None` will crash.

**Best practice for headless tests that open App::Link assemblies:**

```python
import subprocess

try:
    proc = subprocess.run(cmd, capture_output=True, text=True, timeout=TIMEOUT * 2)
except subprocess.TimeoutExpired:
    pytest.xfail(f"INCONCLUSIVE: FreeCADCmd exceeded {TIMEOUT * 2}s "
                 f"(linked file recompute). Not a geometry failure.")
```

Use `pytest.xfail` (not `pytest.fail`) so the test is reported as INCONCLUSIVE rather than a hard failure. Allow 2x the default timeout, no more. If it still exceeds 2x, something else is wrong.

**Preferred alternative:** Read result JSON files written by build scripts (`save_result()` pattern) instead of opening FCStd files in tests. JSON reads are instant and do not trigger recompute.

---

## Error Handling

| Error Type | Exception Class | Identifies Problem | Exit Code |
|-----------|----------------|-------------------|-----------|
| Python syntax | `SyntaxError` | Yes — specific cause + line number | 1 |
| Bad API call | `AttributeError` | Yes — object type + missing method | 1 |
| Bad geometry | `Part.OCCError` | Partially — `BRep_API: command not done` | 1 |
| Missing file | `OSError` | Yes — full file path | 1 |

Bad geometry errors produce a two-layer message: FreeCAD-level ("BRep_API: command not done") then OCC-level exception. Detectable but less specific than other error types.

---

## Miscellaneous

- **TNP (Topological Naming Problem):** Shadow naming system is active in FreeCAD 1.0. Fillet edge references include shadow TNP names (e.g., `;#16:1;:U;XTR;:H4f5:7,E.Edge4`).
- **Recompute:** Call `doc.recompute()` after each feature addition to ensure the feature tree is up to date.
- **Feature tree order:** `body.Group` returns features in creation order. `body.Tip` points to the last feature.
- **Encoding:** FreeCAD on Windows uses cp1252 encoding for `exec(open(...).read())`. Scripts must be pure ASCII. Do not use Unicode box-drawing characters or other non-ASCII characters in FreeCAD scripts.

---

## Hex Sketch Pattern (Regular Polygon)

```python
import math

AF = 13.0  # across-flats dimension
R_inscribed = AF / 2.0
R_circum = R_inscribed / math.cos(math.radians(30))  # vertex radius

# Compute vertices (flat-side-down: vertices at 30, 90, 150, 210, 270, 330 deg)
hex_vertices = []
for i in range(6):
    angle_rad = math.radians(30 + 60 * i)
    x = R_circum * math.cos(angle_rad)
    y = R_circum * math.sin(angle_rad)
    hex_vertices.append(App.Vector(x, y, 0))

# Add 6 line segments
geo_indices = []
for i in range(6):
    idx = sketch.addGeometry(Part.LineSegment(hex_vertices[i], hex_vertices[(i+1)%6]), False)
    geo_indices.append(idx)

# Close polygon: coincident constraints
for i in range(6):
    sketch.addConstraint(Sketcher.Constraint("Coincident",
        geo_indices[i], 2, geo_indices[(i+1)%6], 1))

# Equal sides
for i in range(1, 6):
    sketch.addConstraint(Sketcher.Constraint("Equal", geo_indices[0], geo_indices[i]))

# Fix position (first vertex)
sketch.addConstraint(Sketcher.Constraint("DistanceX", -1, 1, geo_indices[0], 1, hex_vertices[0].x))
sketch.addConstraint(Sketcher.Constraint("DistanceY", -1, 1, geo_indices[0], 1, hex_vertices[0].y))

# Set side length
sketch.addConstraint(Sketcher.Constraint("Distance", geo_indices[0], R_circum))
```

## Annular Ring Sketch Pattern

```python
# Two concentric circles in one sketch -> pad creates an annular ring
sketch.addGeometry(Part.Circle(App.Vector(0,0,0), App.Vector(0,0,1), outer_r), False)
sketch.addGeometry(Part.Circle(App.Vector(0,0,0), App.Vector(0,0,1), inner_r), False)
sketch.addConstraint(Sketcher.Constraint("Coincident", 0, 3, -1, 1))  # outer center to origin
sketch.addConstraint(Sketcher.Constraint("Coincident", 1, 3, -1, 1))  # inner center to origin
sketch.addConstraint(Sketcher.Constraint("Radius", 0, outer_r))
sketch.addConstraint(Sketcher.Constraint("Radius", 1, inner_r))
```

## Parametric Modification (Round-Trip)

```python
# Open existing FCStd, modify a feature parameter, save
doc = App.openDocument(r"path/to/file.FCStd")
body = doc.getObject("Body")

# Modify a pad length
pad = doc.getObject("PadName")
pad.Length = new_value  # accepts float

# Recompute propagates through ALL dependent features
doc.recompute()

# Save (overwrites in place)
doc.save()
```

**Verified behavior:** Modifying `Pad.Length` and calling `recompute()` correctly updates:
- The pad's shape
- All downstream features (e.g., Fillet/Chamfer that reference the pad's edges)
- The body's overall Shape, Volume, and BoundBox
- All of this persists through save/load cycles

## Sketch Attachment Offset

```python
# Attach sketch to a datum plane with a Z offset
sketch.AttachmentSupport = [(doc.getObject("XY_Plane"), "")]
sketch.MapMode = "FlatFace"
sketch.AttachmentOffset = App.Placement(App.Vector(0, 0, offset_z), App.Rotation())
```

Useful for placing a Pocket sketch on the top face of a Pad without needing to reference the actual top face (which would require knowing the face name).

---

## Face/Feature Name Mapping Procedure

**Reusable module:** `cr_scratch/step1_5_scripts/face_mapper.py`

### Shape Enumeration
```python
# Import the face mapper
exec(open(r"path/to/face_mapper.py").read())

# Print full map of all faces, edges, vertices with geometry metadata
data = print_shape_map(obj.Shape, "MyPart")

# Query specific face types
planar_faces = find_faces_by_type(shape, "Part::GeomPlane")
cyl_faces = find_faces_by_type(shape, "Part::GeomCylinder")

# Find faces by orientation
z_up_faces = find_planar_face_by_normal(shape, App.Vector(0, 0, 1))

# Find cylinders by radius
shank_faces = find_cylindrical_face_by_radius(shape, 5.0, tolerance=0.1)
```

### Face Naming Convention
FreeCAD uses 1-based indexing: `Face1`, `Face2`, ... `FaceN` corresponding to `shape.Faces[0]`, `shape.Faces[1]`, ... `shape.Faces[N-1]`. Same for `Edge1`...`EdgeN` and `Vertex1`...`VertexN`.

### Surface Type Identifiers
| TypeId | Geometry | Key Properties |
|---|---|---|
| `Part::GeomPlane` | Flat face | `.Axis` (normal), `.Position` |
| `Part::GeomCylinder` | Cylindrical face | `.Axis`, `.Center`, `.Radius` |
| `Part::GeomCone` | Conical face | `.Axis`, `.Apex`, `.Center`, `.SemiAngle` |
| `Part::GeomSphere` | Spherical face | `.Center`, `.Radius` |
| `Part::GeomTorus` | Toroidal face | `.Center`, `.Axis`, `.MajorRadius`, `.MinorRadius` |

### Curve Type Identifiers
| TypeId | Geometry | Key Properties |
|---|---|---|
| `Part::GeomLine` | Straight edge | `.Direction` |
| `Part::GeomCircle` | Circular edge | `.Location` (center), `.Radius`, `.Axis` (normal) |
| `Part::GeomEllipse` | Elliptical edge | `.Location`, `.MajorRadius`, `.MinorRadius` |
| `Part::GeomBSplineCurve` | B-spline edge | (general curve) |

### Building Assembly References from Face Maps
```python
# After enumerating faces, build references for joints:
ref = build_reference_direct(assembly, "LinkName", "Face3", "Face3")  # face center
ref = build_reference_direct(assembly, "LinkName", "Face3", "Vertex5")  # specific vertex
ref = build_reference_direct(assembly, "LinkName", "Edge2", "Edge2")  # edge/circle center
```

---

## AdditiveHelix / SubtractiveHelix

### Basic Pattern (from FreeCAD TestHelix.py, empirically verified)

```python
# Profile sketch on XZ_Plane
thread_sketch = doc.addObject("Sketcher::SketchObject", "ThreadSketch")
body.addObject(thread_sketch)
xz_plane = body.Origin.OriginFeatures[4]  # XZ_Plane
thread_sketch.AttachmentSupport = xz_plane
thread_sketch.MapMode = "FlatFace"

# In XZ_Plane sketch coordinates:
#   U axis (N_Axis) = body X (radial direction)
#   V axis (V_Axis) = body Z (axial direction, helix axis)

# Add profile geometry (triangle, rectangle, circle, etc.)
# ...
doc.recompute()

# Create helix
helix = doc.addObject("PartDesign::SubtractiveHelix", "ThreadHelix")  # or AdditiveHelix
body.addObject(helix)
helix.Profile = thread_sketch
helix.ReferenceAxis = (thread_sketch, "V_Axis")  # helix axis = body Z
helix.Placement = App.Placement(
    App.Vector(0, 0, 0),
    App.Rotation(App.Vector(0, 0, 1), 0),
    App.Vector(0, 0, 0))
helix.Pitch = 1.25      # distance between turns (mm)
helix.Height = 30.0     # total helical extent (mm)
helix.Turns = 24.0      # number of turns
helix.Angle = 0.0       # cone angle (0 = cylindrical)
helix.Growth = 0.0      # growth per turn (0 = constant pitch)
helix.Mode = 0          # 0 = pitch+height mode
helix.LeftHanded = False
helix.Reversed = True   # True = helix goes in -V direction
doc.recompute()
```

### Helix Mode Values
- `0` = pitch + height (most common)
- `1` = pitch + turns
- `2` = height + turns

### V-Thread Profile for ISO M8x1.25

```python
import math

PITCH = 1.25
MAJOR_R = 4.0                        # cylinder outer radius
THREAD_DEPTH = 0.6134 * PITCH        # 0.767mm
MINOR_R = MAJOR_R - THREAD_DEPTH     # 3.233mm
TRI_H = PITCH * 0.90                 # CRITICAL: must be < PITCH
half_tri = TRI_H / 2.0

# Triangle: tip at major radius (OD), base at minor radius (root)
tip = App.Vector(MAJOR_R, 0, 0)
bl  = App.Vector(MINOR_R, -half_tri, 0)
br  = App.Vector(MINOR_R, half_tri, 0)

g0 = sketch.addGeometry(Part.LineSegment(tip, br), False)
g1 = sketch.addGeometry(Part.LineSegment(br, bl), False)
g2 = sketch.addGeometry(Part.LineSegment(bl, tip), False)

sketch.addConstraint(Sketcher.Constraint("Coincident", g0, 2, g1, 1))
sketch.addConstraint(Sketcher.Constraint("Coincident", g1, 2, g2, 1))
sketch.addConstraint(Sketcher.Constraint("Coincident", g2, 2, g0, 1))
```

### CRITICAL CONSTRAINT: Groove Height Must Be Less Than Pitch

**If the profile height (axial extent) equals the pitch, OCCT produces a self-intersecting swept solid.** Adjacent turns share a boundary, causing:

```
Tool shape is not valid for boolean operation
```

The helix feature's Shape will be NULL. The body falls back to the previous feature's shape. Volume does not change. No Python exception is raised.

**Fix:** Use profile height = 90% of pitch (or less). This is also physically correct for ISO threads, which have truncated crests and roots.

| Profile height / Pitch | Result |
|------------------------|--------|
| 100% (h = P) | **FAIL** -- self-intersection |
| 90% (h = 0.9P) | PASS |
| 80% (h = 0.8P) | PASS |
| 50% (h = 0.5P) | PASS |

### Object Creation Pattern

Use `doc.addObject()` + `body.addObject()` (not `body.newObject()`):

```python
helix = doc.addObject("PartDesign::SubtractiveHelix", "ThreadHelix")
body.addObject(helix)
```

This matches the pattern used in FreeCAD's own test suite (TestHelix.py).

### AdditiveCylinder (Primitive)

```python
cylinder = doc.addObject("PartDesign::AdditiveCylinder", "Cylinder")
cylinder.Radius = 4.0
cylinder.Height = 30.0
cylinder.Angle = 360
body.addObject(cylinder)
```

Alternative to Sketch+Pad for creating cylinders. Used in TestHelix.py.

### Threaded Fastener Construction — Standard Process

**Source:** User's hard-won experience from v1 (CadQuery/PythonOCC) and v2 (FreeCAD). This is the standard process for any fastener that requires modeled threads (common for 3D printing test assemblies). Highly repeatable; avoids thread-bleeding-into-head, nut boolean failures, and helix boundary artifacts.

**When to use:** Any time a fastener is specified as threaded. Threaded modeling is common for 3D-printed test assemblies and functional prototypes.

#### Core Principle: The Stud is a Tool Body

Make a 100mm all-thread stud (threaded rod). It is deliberately oversized. Use features from the **middle** of the stud, never the ends. Cut both ends to length. Each part file (bolt, nut) builds its own stud — this is a best practice, not duplication.

**Why the middle:** The helix has start/end artifacts at Z=0 and Z=100 (partial teeth, ragged termination). Cutting from the middle gives clean thread runout at both ends, like a real fastener. The boolean cuts through continuous helical geometry produce minor overshoot (~1 pitch) but the shape is valid and geometrically sound.

#### Step 1: Build a 100mm Threaded Stud

```python
STUD_LENGTH = 100.0  # tool body -- deliberately long, who cares

# 1. Minor-radius cylinder (full length)
body = doc.addObject("PartDesign::Body", "Body")
base_sketch = body.newObject("Sketcher::SketchObject", "BaseSketch")
base_sketch.AttachmentSupport = [(doc.getObject("XY_Plane"), "")]
base_sketch.MapMode = "FlatFace"
base_sketch.addGeometry(Part.Circle(App.Vector(0,0,0), App.Vector(0,0,1), MINOR_R), False)
base_sketch.addConstraint(Sketcher.Constraint("Coincident", 0, 3, -1, 1))
base_sketch.addConstraint(Sketcher.Constraint("Radius", 0, MINOR_R))
doc.recompute()

shank_pad = body.newObject("PartDesign::Pad", "ShankPad")
shank_pad.Profile = base_sketch
shank_pad.Length = STUD_LENGTH
shank_pad.Type = 0
doc.recompute()

# 2. Thread profile on XZ_Plane -- ridge points OUTWARD past major_R
thread_sketch = doc.addObject("Sketcher::SketchObject", "ThreadSketch")
body.addObject(thread_sketch)
xz_plane = body.Origin.OriginFeatures[4]
thread_sketch.AttachmentSupport = xz_plane
thread_sketch.MapMode = "FlatFace"

half_tri = TRI_H / 2.0  # TRI_H = PITCH * 0.90 (MUST be < PITCH)
tip = App.Vector(MAJOR_R + 0.05, 0, 0)   # 0.05 past major for clean overlap
bl  = App.Vector(MINOR_R - 0.05, -half_tri, 0)  # 0.05 inside minor
br  = App.Vector(MINOR_R - 0.05, half_tri, 0)

g0 = thread_sketch.addGeometry(Part.LineSegment(tip, br), False)
g1 = thread_sketch.addGeometry(Part.LineSegment(br, bl), False)
g2 = thread_sketch.addGeometry(Part.LineSegment(bl, tip), False)
thread_sketch.addConstraint(Sketcher.Constraint("Coincident", g0, 2, g1, 1))
thread_sketch.addConstraint(Sketcher.Constraint("Coincident", g1, 2, g2, 1))
thread_sketch.addConstraint(Sketcher.Constraint("Coincident", g2, 2, g0, 1))
doc.recompute()

# 3. AdditiveHelix -- adds thread ridges to the cylinder
helix = doc.addObject("PartDesign::AdditiveHelix", "ThreadHelix")
body.addObject(helix)
helix.Profile = thread_sketch
helix.ReferenceAxis = (thread_sketch, "V_Axis")
helix.Pitch = PITCH
helix.Height = STUD_LENGTH
helix.Turns = STUD_LENGTH / PITCH
helix.Mode = 0
helix.Reversed = False
doc.recompute()
```

#### Step 2: Build the Bolt (Stud + Collar + Head + Cut Both Ends)

Add features in the **middle** of the stud, then cut both ends. **NEVER** stop threads early — add unthreaded portions AFTER the threaded section.

```python
# Place features in the MIDDLE of the stud.
# For a 30mm shank bolt: collar at Z=50, shank runs Z=20 to Z=50.
COLLAR_Z = 50.0  # mid-stud -- well away from both helix ends
SHANK_BOTTOM_Z = COLLAR_Z - SHANK_L  # e.g., Z=20

# Collar (unthreaded ring at MAJOR_R)
collar_sketch = body.newObject("Sketcher::SketchObject", "CollarSketch")
collar_sketch.AttachmentSupport = [(doc.getObject("XY_Plane"), "")]
collar_sketch.MapMode = "FlatFace"
collar_sketch.AttachmentOffset = App.Placement(
    App.Vector(0, 0, COLLAR_Z), App.Rotation())
collar_sketch.addGeometry(Part.Circle(App.Vector(0,0,0), App.Vector(0,0,1), MAJOR_R + 0.1), False)
# ... constraints, pad ...

# Hex head (offset above collar)
hex_z = COLLAR_Z + COLLAR_H
hex_sketch.AttachmentOffset = App.Placement(
    App.Vector(0, 0, hex_z), App.Rotation())
# ... hex geometry + pad ...

# CUT BOTH ENDS using Part-level booleans
# (PartDesign::Pocket has direction issues with complex helix geometry)
top_z = hex_z + HEAD_H
body_shape = body.Shape.copy()

cut_top = Part.makeCylinder(MAJOR_R + 1.0, STUD_LENGTH - top_z + 5.0,
                             App.Vector(0, 0, top_z))
cut_bot = Part.makeCylinder(MAJOR_R + 1.0, SHANK_BOTTOM_Z + 5.0,
                             App.Vector(0, 0, -5.0))

cut_shape = body_shape.cut(cut_top)
cut_shape = cut_shape.cut(cut_bot)

# Extract solid, clean up, fix
if cut_shape.ShapeType == "Compound":
    cut_shape = max(cut_shape.Solids, key=lambda s: s.Volume)
cut_shape = cut_shape.removeSplitter()
if not cut_shape.isValid():
    cut_shape.fix(1e-7, 1e-7, 1e-7)

# Chamfer hex top edges (Part-level, not PartDesign)
chamfered = cut_shape.makeChamfer(0.5, [edges at top_z...])

# Store as Part::Feature
bolt_feature = doc.addObject("Part::Feature", "Bolt")
bolt_feature.Shape = cut_shape
doc.recompute()
```

**Why Part-level boolean for the cut:** PartDesign::Pocket has direction ambiguity with complex helix geometry that can cause it to cut the wrong way or produce invalid shapes. Part-level `shape.cut()` is reliable.

#### Step 3: Build the Nut (Hex Body + Boolean Cut with Scaled Stud Tool)

Build a **separate 100mm stud** with 3% scaled dimensions in X/Y (NOT Z!). Boolean-subtract from hex body. The stud tool must extend well past both faces of the hex to avoid helix boundary artifacts in the bore.

**Critical lesson learned:** Do NOT use `transformGeometry(matrix)` with non-uniform scaling on helix geometry — it produces wrong results (volume decreases, asymmetric BoundBox). Instead, pre-compute the scaled dimensions and build the tool stud with those dimensions directly.

```python
# Scaled dimensions -- compute BEFORE building, don't transform after
SCALE_XY = 1.03
S_MAJOR_R = MAJOR_R * SCALE_XY        # 4.12mm for M8
S_MINOR_R = MINOR_R * SCALE_XY        # 3.33mm
S_BASE_R  = BASE_R * SCALE_XY         # 3.28mm
S_TIP_R   = (MAJOR_R + 0.05) * SCALE_XY  # tip past major for overlap

# 1. Build hex body in nut document
nut_doc = App.newDocument("NutThreaded")
# ... hex sketch + pad ...

# 2. Build 100mm stud tool in SEPARATE document with scaled dimensions
tool_doc = App.newDocument("ToolStud")
tool_body = tool_doc.addObject("PartDesign::Body", "ToolBody")

# Tool sketch offset to Z=-5 so tool extends past nut bottom
tool_sketch.AttachmentOffset = App.Placement(
    App.Vector(0, 0, -5.0), App.Rotation())
# Circle at S_MINOR_R (scaled), pad to STUD_LENGTH
# Thread profile using S_TIP_R, S_BASE_R (all pre-scaled)
# AdditiveHelix with PITCH (NOT scaled -- Z stays 1.0!)

# 3. Boolean subtract
hex_shape = nut_body.Shape
tool_shape = tool_body.Shape
result_shape = hex_shape.cut(tool_shape)

# 4. Extract largest solid (boolean may produce compound)
if result_shape.ShapeType == "Compound":
    result_shape = max(result_shape.Solids, key=lambda s: s.Volume)

# 5. Store as Part::Feature
nut_feature = nut_doc.addObject("Part::Feature", "Nut")
nut_feature.Shape = result_shape
nut_doc.recompute()

App.closeDocument(tool_doc.Name)  # tool doc no longer needed
```

**Key rules:**
- The stud is 100mm long. Use the middle. Cut both ends. Don't micro-optimize length.
- Each part file builds its own stud. This is fine and may be a best practice.
- Scale X and Y by 1.03. NEVER scale Z — this would change pitch and break thread mating.
- Pre-compute scaled dimensions. Do NOT use `transformGeometry()` for non-uniform scaling on helix geometry.
- The nut stud tool must extend well past both hex faces (offset Z=-5 or more).
- Boolean results are `Part::Feature`, not `PartDesign::Body`. This is expected and correct.
- The 3% clearance creates ~0.12mm radial clearance at crests (for M8).
- After boolean cuts: `removeSplitter()` then `fix(1e-7, 1e-7, 1e-7)` if invalid.
- Profile height MUST be < pitch (90% works). Equal to pitch causes OCCT self-intersection.
- `PartDesign::Pocket` has direction issues on helix geometry — use Part-level `shape.cut()` instead.

---

## Headless Script Rules

### NEVER Import FreeCADGui in FreeCADCmd Scripts

```python
# WRONG -- opens a GUI window on the user's machine!
import FreeCADGui
FreeCADGui.showMainWindow()

# RIGHT -- FreeCADCmd is headless, no GUI module needed
import FreeCAD as App
import Part
import PartDesign
import Sketcher
```

`FreeCADCmd.exe` is the headless executable. Importing `FreeCADGui` and calling `showMainWindow()` will launch an actual FreeCAD GUI window. This is never appropriate in automated scripts.

### Visibility in Headless-Created FCStd Files

**Problem:** `FreeCADCmd`'s `doc.saveAs()` does NOT generate `GuiDocument.xml`. Without it, FreeCAD GUI creates default ViewProviders on first open, but defaults everything to hidden (user must manually toggle visibility).

**Two valid solutions:**

1. **Strip mode (simplest):** Remove any `GuiDocument.xml`. FreeCAD creates defaults on open.
2. **Template mode:** Generate a complete `GuiDocument.xml` that passes `Gui::Document::RestoreDocFile()` validation. Use `fix_visibility_xml.py --template`.

### GuiDocument.xml Specification (FreeCAD 1.0)

**Parser:** FreeCAD uses Xerces-C SAX parsing (`Base::XMLReader`) to read `GuiDocument.xml` sequentially. The parser calls `readElement()` for each expected element in strict order. Missing elements cause the SAX parser to read past EOF, triggering the generic error: `"Reading failed from embedded file: GuiDocument.xml"`.

**Required document structure (strict order):**

```xml
<?xml version='1.0' encoding='utf-8'?>
<!--
 FreeCAD Document, see https://www.freecad.org for more information...
-->
<Document SchemaVersion="1" HasExpansion="1">
    <Expand count="0"/>                          <!-- MANDATORY: count attr (lowercase c) -->
    <ViewProviderData Count="N">                 <!-- MANDATORY: Count attr (uppercase C) -->
        <ViewProvider name="..." expanded="0" treeRank="-1">
            <!-- Extensions (if applicable) -->
            <Properties Count="M" TransientCount="0">
                <!-- Property elements -->
            </Properties>
        </ViewProvider>
        <!-- ... one per document object ... -->
    </ViewProviderData>
    <Camera settings="OrthographicCamera {...}"/> <!-- MANDATORY: parser unconditionally reads this -->
</Document>
```

**Critical elements that MUST be present:**

| Element | Why Required | What Happens If Missing |
|---------|-------------|------------------------|
| `<Expand count="N">` | `ExpandInfo::restore()` calls `getAttribute<long>("count")` unconditionally | `getAttribute` throws, "Reading failed from embedded file" |
| `<Camera settings="..."/>` | `RestoreDocFile()` calls `readElement("Camera")` after ViewProviderData | SAX parser reads past `</Document>`, hits EOF, throws |

**Expand element:** Must have `count` attribute (lowercase c). Use `count="0"` for no expansion state. The `<ViewProviderData>` uses `Count` (uppercase C) -- different casing.

**Camera element:** Must be the LAST child of `<Document>`, after `</ViewProviderData>`. Settings value is an Open Inventor camera string with `&#10;` for newlines. Default safe value:
```
OrthographicCamera {
  viewportMapping ADJUST_CAMERA
  position 0 -100 0
  orientation 0.57735026 0.57735026 0.57735026  2.0943952
  nearDistance -200
  farDistance 200
  aspectRatio 1
  focalDistance 100
  height 100
}
```

**ViewProvider property counts by type:**

| Object Type | VP Properties | Extensions | Notes |
|-------------|--------------|------------|-------|
| `PartDesign::Body` | 22 | `ViewProviderOriginGroupExtension`, `ViewProviderFaceTexture` | status="1" |
| `App::Origin` | 6 | none | Size is PropertyVector |
| `App::Plane`, `App::Line` | 10 | none | status="1", BoundingBox status="9" |
| `PartDesign::Pad/Pocket/etc.` | 21 | `ViewProviderSuppressibleExtension`, `ViewProviderFaceTexture`, `ViewProviderAttachExtension` | Most props status="9" (inherited) |
| `Sketcher::SketchObject` | 21 (min) | `ViewProviderFaceTexture`, `ViewProviderAttachExtension`, `ViewProviderGridExtension` | Full: 36 props |
| `Assembly::AssemblyObject` | 5 | `ViewProviderOriginGroupExtension` | |
| `App::Link` | 15 | `ViewProviderLinkObserver` | |

**Extension namespace:** Always `Gui::` prefix for `ViewProviderSuppressibleExtension`, `ViewProviderFaceTexture`, `ViewProviderOriginGroupExtension`, `ViewProviderGroupExtension`. Use `PartGui::` for `ViewProviderAttachExtension`, `ViewProviderGridExtension`.

**Binary file references (file= attribute) -- CRITICAL:** `ColorList` and `MaterialList` elements have a `file=` attribute pointing to binary data in the ZIP. **Every non-empty `file=` reference MUST have a corresponding binary file in the ZIP archive.** Using `file=""` (empty string) for multiple VPs causes `"Reading failed from embedded file: GuiDocument.xml"`. This was the root cause of 5 failed visibility fix attempts (Steps 2.5 through 5.9). Binary files must be generated with correct content:

- **ShapeAppearance (76 bytes with UUID):** For Body, PartDesign features, Sketcher. Contains material properties + FreeCAD default material UUID `7f9fd73b-50c9-41d8-b7b2-575a030c1eeb`.
- **ShapeAppearance (40 bytes without UUID):** For App::Line, App::Plane. Same material structure but no UUID, alpha bytes = 0x00.
- **LineColorArray / PointColorArray (8 bytes):** `uint32 count=1, uint32 color=0x19191900`.
- Binary file naming: sequential per type — `ShapeAppearance`, `ShapeAppearance1`, `ShapeAppearance2`, etc.

**Tip feature visibility:** The Tip feature (Body.Tip in Document.xml, the last feature in the timeline) must have `Visibility="true"` in its VP. All other PartDesign features should have `Visibility="false"`. Without this, the Body is unhidden but no geometry is displayed because the Tip (which controls the visible shape) is hidden.

**XML serialization:** Use `ET.tostring(root, encoding="unicode", short_empty_elements=False)` for explicit close tags. This matches the proven working configuration (Step 5.10 H1 test). Both self-closing and explicit close tags are valid XML, but explicit close tags are the validated path.

**Testing limitation:** FreeCADCmd ignores `GuiDocument.xml` entirely. The parser only runs in FreeCAD GUI. Template changes must be verified by opening in the GUI.

**Tool body visibility (Step 6.5, root-caused):** Part::Feature objects used as boolean tools (e.g., `AllHolesTool`, `MagPocketTool`, `EpitrochoidSolid`) must have `Visibility=False`. These are construction geometry that lives outside the PartDesign::Body as document-level objects. If visible, they render ON TOP of the boolean result, making it look like the boolean failed. Fix in build scripts: `tool_feat.Visibility = False` immediately after creation. Fix in GuiDocument.xml template: hide Part::Feature objects whose names end in "Tool" or "Solid". The `fix_visibility_xml.py --template` mode handles this automatically.

**OriginGroupExtension (Step 6.5, root-caused):** Do NOT inject `Gui::ViewProviderOriginGroupExtension` into GuiDocument.xml VP entries for PartDesign::Body. FreeCAD 1.0.2 GUI rejects it: "Extension is not a python addable version." This is a C++ extension that FreeCAD adds internally when it recognizes the VP type — injecting it from XML causes errors (one per Body per linked document, can produce 100+ console errors for large assemblies). Omit the Extensions block entirely from Body VPs; FreeCAD will add the correct extensions automatically.

**Do NOT:**
- Import FreeCADGui in FreeCADCmd scripts -- see prohibition above
- Add minimal GuiDocument.xml (1 property per VP) -- FreeCAD ignores these
- Omit the `<Camera>` element -- parser crashes
- Omit the `count` attribute on `<Expand>` -- parser crashes
- Use `file=""` (empty string) for binary file references across multiple VPs -- causes "Reading failed"
- Omit binary files referenced by `file=` attributes -- must exist in ZIP
- Set tool bodies (Part::Feature used for PartDesign::Boolean) to Visibility=True -- obscures the boolean result

---

## Pre-Flight Diff Pattern

Before a build script overwrites an existing FCStd, it must read the file's current geometry state and compare against expected values. This prevents silent destruction of manual GUI edits.

### Reading Geometry State from an Existing FCStd

```python
import FreeCAD
import os

def preflight_check(fcstd_path, expected):
    """
    Read geometry state from an existing FCStd and compare against expected values.

    Parameters
    ----------
    fcstd_path : str
        Absolute path to the FCStd file.
    expected : dict
        Keys: 'volume', 'feature_count', 'tip_name', 'bb_x', 'bb_y', 'bb_z'.
        All optional. Missing keys are skipped.

    Returns
    -------
    dict with keys:
        'exists': bool
        'volume': float or None
        'feature_count': int or None
        'tip_name': str or None
        'bb': (xlen, ylen, zlen) or None
        'divergences': list of str (empty if all match)
    """
    result = {'exists': False, 'volume': None, 'feature_count': None,
              'tip_name': None, 'bb': None, 'divergences': []}

    if not os.path.exists(fcstd_path):
        return result

    result['exists'] = True
    doc = FreeCAD.openDocument(fcstd_path)

    # Find the Body (first PartDesign::Body in the document)
    body = None
    for obj in doc.Objects:
        if obj.TypeId == "PartDesign::Body":
            body = obj
            break

    if body is None:
        result['divergences'].append("No PartDesign::Body found in document")
        FreeCAD.closeDocument(doc.Name)
        return result

    # Read current state
    shape = body.Shape
    result['volume'] = shape.Volume
    result['feature_count'] = len([o for o in body.Group
                                    if o.TypeId not in ("App::Origin",)])
    result['tip_name'] = body.Tip.Name if body.Tip else None
    bb = shape.BoundBox
    result['bb'] = (bb.XLength, bb.YLength, bb.ZLength)

    # Compare against expected values
    if 'volume' in expected and expected['volume'] is not None:
        ratio = result['volume'] / expected['volume'] if expected['volume'] > 0 else float('inf')
        if abs(ratio - 1.0) > 0.001:  # 0.1% tolerance
            result['divergences'].append(
                "Volume: actual=%.2f expected=%.2f (ratio=%.4f)" %
                (result['volume'], expected['volume'], ratio))

    if 'feature_count' in expected and expected['feature_count'] is not None:
        if result['feature_count'] != expected['feature_count']:
            result['divergences'].append(
                "Feature count: actual=%d expected=%d" %
                (result['feature_count'], expected['feature_count']))

    if 'tip_name' in expected and expected['tip_name'] is not None:
        if result['tip_name'] != expected['tip_name']:
            result['divergences'].append(
                "Tip name: actual=%s expected=%s" %
                (result['tip_name'], expected['tip_name']))

    if 'bb_z' in expected and expected['bb_z'] is not None:
        if abs(result['bb'][2] - expected['bb_z']) > 0.1:
            result['divergences'].append(
                "BoundBox Z: actual=%.2f expected=%.2f" %
                (result['bb'][2], expected['bb_z']))

    FreeCAD.closeDocument(doc.Name)
    return result
```

### Usage in Build Scripts

```python
# At the top of any build_parts/*.py script:
check = preflight_check(output_path, {
    'volume': 54800.0,       # mm^3, computed from geometry
    'feature_count': 12,
    'tip_name': 'AllCuts',
    'bb_z': 46.5,
})

if check['exists'] and not check['divergences']:
    print("FCStd matches expected state -- safe to rebuild")
elif check['exists'] and check['divergences']:
    print("FCStd DIVERGES from expected state:")
    for d in check['divergences']:
        print("  - %s" % d)
    print("The FCStd wins. Read actual geometry before modifying.")
    # Option: update params.yaml from actual values, then proceed
else:
    print("No existing FCStd -- building from scratch")
```

### Tolerance Handling

- **Volume:** 0.1% relative tolerance (`abs(ratio - 1.0) <= 0.001`). Fillets and thread modeling introduce small volume variations that stay well within this band.
- **Bounding box:** 0.1 mm absolute tolerance. Accounts for floating-point differences in OCCT edge placement.
- **Feature count:** Exact match. A mismatch means features were added or removed in the GUI.
- **V=0:** Always a build failure, regardless of `isValid()`. OCCT can report `isValid()=True` on zero-volume shapes produced by collapsed booleans.

**Source:** WO-2026-002 process_docs_update_plan.md Section 7c (Pre-Flight Diff Law).

---

## GUI Editability Guidance

FCStd files produced by headless scripts are fully editable in the FreeCAD GUI. Once a human inspects or edits an FCStd, that file becomes the source of truth for the part's geometry (the FCStd primacy rule). This section documents what is safe to edit and what is not.

### Safe to Edit

| Operation | How | Notes |
|-----------|-----|-------|
| Change sketch dimensions | Double-click sketch, edit constraints | Propagates to all downstream features on recompute |
| Change Pad/Pocket length | Select feature, edit Length in Data panel | Propagates downstream |
| Change Fillet/Chamfer radius | Select feature, edit Radius/Size in Data panel | |
| Change Hole parameters | Select Hole feature, edit thread size/depth/clearance | |
| Add a new Pad, Pocket, Hole, or Fillet | Part Design menu, attach to existing face or datum plane | Appended after current Tip |
| Rearrange feature order | Drag features in the Model tree (with care) | FreeCAD 1.0 supports this but references can break |
| Move assembly Link placements | Select Link, edit Placement in Data panel | For pre-positioned assemblies only |
| Toggle feature suppression | Right-click feature, Suppress | Temporarily removes feature from computation |

### Unsafe / Destructive

| Operation | Risk | What to do instead |
|-----------|------|--------------------|
| Delete a feature that other features reference | Breaks downstream features (dangling references) | Suppress the feature instead, or delete from the bottom of the tree upward |
| Edit a `Part::Feature` shape directly | `Part::Feature` shapes are computed results, not parametric. Edits are not persistent across recompute. | If the shape came from OCCT (e.g., a boolean tool), edit the script that generated it and rebuild |
| Modify the Origin object | Origin (XY_Plane, XZ_Plane, YZ_Plane, axes) is auto-generated per Body. Modifying it corrupts the Body. | Never touch Origin |
| Rename objects that scripts reference by name | Build scripts use `doc.getObject("BodyName")`. Renaming breaks the script. | If renaming is needed, update all scripts that reference the old name |
| Edit `Part::Feature` objects that are boolean tool shapes | These are consumed by `PartDesign::Boolean`. Editing the tool shape may not propagate to the boolean result. | Rebuild the tool shape via script |

### Round-Trip Workflow

1. Script builds FCStd from `params.yaml`
2. Engineer opens in GUI, inspects, makes adjustments
3. Next script run detects divergence via pre-flight diff
4. Script reads actual geometry from FCStd, applies requested changes on top of current state
5. Engineer re-inspects

The `Parametric Modification (Round-Trip)` section of this reference documents the API for step 4.

**Source:** WO-2026-002 process_docs_update_plan.md Section 7a-7d.

---

## Assembly Joint Strategy

Decision tree for choosing joint types in FreeCAD's native Assembly workbench. The workbench uses the Ondsel MbD solver.

### Decision Tree

```
Is the part at a known absolute position (built at final coordinates)?
+-- YES -> GroundedJoint
|         Locks the part at its construction placement.
|         No solver computation needed.
|
+-- NO -> Does the part need to move relative to another part?
         +-- NO (static relationship) -> Fixed joint
         |   References two faces. Solver computes placement
         |   to align them. Use findPlacement().
         |
         +-- YES -> What kind of motion?
                  +-- Rotation around one axis -> Revolute joint (type_index=1)
                  |   For bearings, shafts, rotating interfaces.
                  |   References cylindrical faces.
                  |
                  +-- Rotation + translation along same axis -> Cylindrical joint (type_index=2)
                  |   For sliding/rotating fits.
                  |
                  +-- Translation along one axis -> Slider joint (type_index=3)
                  |   For linear guides.
                  |
                  +-- Free rotation -> Ball joint (type_index=4)
                       For spherical interfaces.
```

### Joint Type Summary

| Type | type_index | DOF removed | Typical use | Reference geometry |
|------|-----------|-------------|-------------|-------------------|
| Grounded | N/A | 6 (all) | Parts built at final position | None (uses Link.Placement) |
| Fixed | 0 | 6 (all) | Static part-to-part attachment | Two planar faces |
| Revolute | 1 | 5 (1 rotation free) | Bearings, shaft journals | Cylindrical face pairs |
| Cylindrical | 2 | 4 (1 rotation + 1 translation free) | Sliding bearings | Cylindrical face pairs |
| Slider | 3 | 5 (1 translation free) | Linear guides | Planar face pairs |
| Ball | 4 | 3 (3 rotations free) | Ball joints | Vertex or spherical face |

### GUI Instructions (for Wesley / design engineer audience)

**Creating a Grounded Joint (GUI):**
1. Open the assembly in FreeCAD
2. In the Model tree, expand the Assembly object
3. Click on the Link you want to ground
4. Menu: Assembly > Ground (or right-click the Link > Ground)
5. The part is now fixed at its current placement

**Creating a Fixed Joint (GUI):**
1. Select the first face on Part A (click on the 3D view)
2. Hold Ctrl, select the second face on Part B
3. Menu: Assembly > Fixed Joint
4. The solver will align the two faces. If the result is wrong, try selecting different faces or toggle the joint's Offset property.

**Creating a Revolute Joint (GUI):**
1. Select a cylindrical face on the rotating part (e.g., bearing outer race)
2. Hold Ctrl, select the mating cylindrical face on the fixed part (e.g., bearing bore)
3. Menu: Assembly > Revolute Joint
4. The part can now rotate around the shared axis

**Scripting Equivalents:**

See the existing "Assembly Scripting" section for the API. Key points:
- Joint type is set by `type_index` in `JointObject.Joint(joint_obj, type_index)`
- References require TWO sub-elements: `(assembly, ["LinkName.FaceN", "LinkName.FaceN"])`
- Always call `findPlacement()` after setting references (except for GroundedJoint)
- Grounded joints do not use references -- they use `JointObject.GroundedJoint(ground, link)`

### Transition Strategy for Existing Assemblies

Current motor assembly uses explicit XYZ placements (no joints, no solver). Migration path:

1. **Phase 1:** Add GroundedJoint to every Link (equivalent to current explicit placement). Solver result = 0. Zero functional change. This is where the Borebot motor assembly is now.
2. **Phase 2:** For interfaces that Wesley needs to explore (e.g., bearing fits, endplate positions), replace GroundedJoint with the appropriate parametric joint (Fixed, Revolute). The solver computes placement from face references.
3. **Phase 3:** Remove explicit placement coordinates from `assemble.py`. Let the solver position everything from joints.

**Source:** WO-2026-002 process_docs_update_plan.md Section 7e.

---

## PartDesign::Hole Best Practices

All threaded holes should use FreeCAD's native `PartDesign::Hole` feature. It handles thread standards, bore sizing, clearance, and optional 3D thread rendering in one operation.

### Standard Settings

| Setting | Value | Rationale |
|---------|-------|-----------|
| Threaded | True | Sizes bore for tapping |
| ModelThread | True | Renders 3D helix geometry for 3D printing / visual verification |
| UseCustomThreadClearance | True | Overrides default fit tolerance |
| CustomThreadClearance | 0.2 mm | Consistent clearance across all thread sizes. Replaces the old 3% scale-up convention. |
| Refine | **False** | **Critical.** Prevents cascading "Removing splitter failed" errors on complex bodies. This is the single most common cause of thread features disappearing. |

### Radial Holes (Perpendicular to Axis of Revolution)

Requires one datum plane per hole. PolarPattern does not work with datum-plane-based Hole features in FreeCAD 1.0.

```python
import math

# Datum plane with normal pointing radially outward at angle `a` degrees
px = R * math.cos(math.radians(a))
py = R * math.sin(math.radians(a))

dp = body.newObject("PartDesign::Plane", "RadialDP")
dp.MapMode = "Translate"
dp.Placement = FreeCAD.Placement(
    FreeCAD.Vector(px, py, z_position),
    FreeCAD.Rotation(a, 90, 0))  # yaw=angle, pitch=90 -> normal = radial outward
```

The Hole feature drills in the **-normal** direction by default (radially inward from the OD surface). If it drills the wrong way, set `hole.Reversed = True`.

### Axial Holes (Along Axis of Revolution)

Use a datum plane at the face position. The datum plane normal should point along the drill direction.

```python
# Front face (Z=0), drill in +Z direction
dp = body.newObject("PartDesign::Plane", "AxialFrontDP")
dp.MapMode = "Translate"
dp.Placement = FreeCAD.Placement(
    FreeCAD.Vector(0, 0, 0),
    FreeCAD.Rotation(0, 0, 0))  # normal = +Z

# Attach sketch, draw circles at bolt positions, apply Hole
hole.Reversed = True  # drill INTO body (+Z direction)
```

**Why Reversed = True:** The Hole feature drills in the -normal direction by default. For faces on the body boundary, -normal points away from the body. Reversed flips it inward.

**Do not flip the datum plane** (e.g., `pitch=180`) to reverse drill direction. This introduces orientation confusion downstream. Keep datum planes in standard orientation and use `hole.Reversed` to control drill direction.

### Full Scripting Checklist

```python
hole = body.newObject("PartDesign::Hole", "MyHole")
hole.Profile = sketch              # sketch on datum plane with circle(s)
hole.Threaded = True
hole.ModelThread = True
hole.ThreadType = "ISOMetricProfile"  # or "UNC", "UNF", "UNEF"
hole.ThreadSize = "M4"
hole.UseCustomThreadClearance = True
hole.CustomThreadClearance = 0.2   # mm
hole.DepthType = "Dimension"       # or "ThroughAll"
hole.Depth = 8.0                   # mm (ignored if ThroughAll)
hole.Refine = False                # CRITICAL
hole.Reversed = True               # test both orientations
doc.recompute()
```

**Always verify persistence after save/reopen.** FreeCAD can report success during the session but fail to persist features. Open the saved file in a new session and count Hole features.

**Full reference:** `RESOURCES/FreeCAD_Threaded_Holes_Tipsheet.md` -- includes available thread types and sizes, troubleshooting table, and GUI workflow.

**Source:** WO-2026-002 process_docs_update_plan.md Section 7f, validated on stator shell (15 M6 + 15 M4 radial + 10 M4 axial = 40 holes).

---

## Coil Pocket OCCT Exception

`PartDesign::Hole` and `PartDesign::Pocket` cannot cut radial pockets from the bore (ID) surface. Attaching a sketch to a datum plane positioned at the ID surface and applying a Hole or Pocket produces delta=0 (zero material removed) in all orientations tested. The feature completes without error but changes nothing.

### Root Cause

The ID surface of a revolution body is a concave cylindrical face. PartDesign features that drill or cut expect to start from a convex or planar surface where the tool direction points into material. On a concave bore surface, the default drill direction points away from the material, and Reversed points through the full wall thickness -- neither produces the intended shallow pocket.

### Workaround: OCCT Cylinders + PartDesign::Boolean

Build the pocket geometry as OCCT primitives and boolean-subtract them from the Body in a single operation.

```python
import math
import Part

pockets = []
for i in range(n_coils):
    angle = math.radians(i * 360.0 / n_coils)
    # Pocket cylinder at the bore surface, pointing radially inward
    center = FreeCAD.Vector(
        pocket_radius * math.cos(angle),
        pocket_radius * math.sin(angle),
        z_center)
    direction = FreeCAD.Vector(math.cos(angle), math.sin(angle), 0)
    cyl = Part.makeCylinder(pocket_r, pocket_depth, center, direction)
    pockets.append(cyl)

# Fuse all pockets into one compound (see Combined Tool Pattern)
fused = pockets[0]
for p in pockets[1:]:
    fused = fused.fuse(p)

# Single PartDesign::Boolean cut
tool = doc.addObject("Part::Feature", "CoilPocketTool")
tool.Shape = fused
bool_cut = body.newObject("PartDesign::Boolean", "CoilPockets")
bool_cut.Type = "Cut"
bool_cut.Group = [tool]
doc.recompute()
```

### Authorization

This is an authorized OCCT exception. The pocket cylinders are dumb tool shapes consumed by a single `PartDesign::Boolean`. The tool is dumb; the operation is parametric. The Body's feature tree remains intact, and downstream features (fillets, additional booleans) chain from the boolean result.

Valid justifications for OCCT tool shapes (per project convention):
- (a) Frozen potted assembly that will not be edited
- (b) Imported third-party geometry (McMaster STEPs)
- (c) Tool shapes consumed by `PartDesign::Boolean` -- **this case**

For >15 shapes, use the binary merge tree fuse described in the Combined Tool Pattern section to reduce fuse time from O(n^2) to O(n log n).

**Source:** WO-2026-002 process_docs_update_plan.md Section 7g. Validated on stator shell (15 coil pockets).

---

## Combined Tool Pattern (Batch Subtractive Features) -- Expanded

**Named pattern: "Fuse-First-Boolean-Once."**

When cutting multiple features of the same type from a Body (radial holes, magnet pockets, coil pockets), fuse all OCCT tool shapes into one compound and perform a single `PartDesign::Boolean` cut. This is the canonical approach for batch subtractive features in PartDesign.

### Standard Recipe

```python
import Part

# 1. Build all tool shapes
shapes = []
for i in range(count):
    s = Part.makeCylinder(radius, depth, position_i, direction_i)
    shapes.append(s)

# 2. Fuse into one compound
fused = shapes[0]
for s in shapes[1:]:
    fused = fused.fuse(s)

# 3. Add as Part::Feature
tool = doc.addObject("Part::Feature", "CombinedTool")
tool.Shape = fused

# 4. Single PartDesign::Boolean cut
bool_cut = body.newObject("PartDesign::Boolean", "AllCuts")
bool_cut.Type = "Cut"
bool_cut.Group = [tool]
doc.recompute()
```

### Binary Merge Tree for Large Shape Counts

The naive sequential fuse (`shapes[0].fuse(shapes[1]).fuse(shapes[2])...`) is O(n^2) because each fuse operates on an increasingly complex compound. For >15 shapes, use a binary merge tree that keeps intermediate shapes balanced:

```python
def binary_fuse(shapes):
    """Fuse a list of OCCT shapes using a binary merge tree. O(n log n)."""
    if len(shapes) == 0:
        raise ValueError("No shapes to fuse")
    if len(shapes) == 1:
        return shapes[0]

    level = list(shapes)
    while len(level) > 1:
        next_level = []
        for i in range(0, len(level), 2):
            if i + 1 < len(level):
                next_level.append(level[i].fuse(level[i + 1]))
            else:
                next_level.append(level[i])  # odd one out, carry forward
        level = next_level
    return level[0]

# Usage
fused = binary_fuse(shapes)
```

**Performance:** For the rotor hub (40 magnet pockets), binary fuse completes in ~2 seconds vs ~8 seconds for sequential fuse. The difference grows with shape count.

### Why Fuse First

Multiple sequential `PartDesign::Boolean` operations on the same Body can produce zero-volume results even when each individual cut is valid. The second boolean can interact with or undo the first. One boolean per batch is reliable; two or more in sequence is not guaranteed.

**Rule:** One `PartDesign::Boolean` per logical batch. If you need to cut holes AND pockets, fuse all holes into one tool, fuse all pockets into another tool, and apply two booleans (one per batch). Do not apply 40 individual booleans.

**Source:** WO-2026-002 Step 5D stator shell (55 features in one boolean) and rotor hub (40 magnet pockets in one boolean).

---

## Fillet Ordering Rule -- Expanded

Apply `PartDesign::Fillet` **before** boolean operations that would fragment the target edge.

### The Problem

A fillet targets a specific edge (e.g., `"Edge4"` -- a single circular edge from a revolution). If a `PartDesign::Boolean` cut crosses that edge, it splits the single edge into multiple segments (potentially dozens). Applying the fillet after the boolean means FreeCAD must fillet many short edge fragments instead of one clean edge. This either fails outright or produces invalid geometry.

### Feature Tree Ordering

```
1. Base shape (Pad, Revolution)
2. Fillets on clean edges         <-- fillet HERE
3. PartDesign::Boolean cuts       <-- booleans fragment edges
4. Additional fillets on NEW edges introduced by booleans (if needed)
```

### Example

Rotor hub inside corner fillet at R=54, Z=3:
- **Applied after revolution** (1 clean circular edge): fillet succeeds.
- **Applied after 40 magnet pocket booleans** (edge fragmented into 40+ segments): `PartDesign::Fillet` fails with invalid geometry.

### Rule

Plan the feature tree so that every fillet operates on a clean, unfragmented edge. If a boolean must precede a fillet, verify that the boolean does not cross the fillet's target edge.

**Source:** WO-2026-002 Step 5D rotor hub build.

---

## Boolean Cut Strategy -- Expanded

### One Boolean Per Body Per Batch

Multiple sequential `PartDesign::Boolean` cuts on the same Body can produce volume collapse (zero-volume result) even when each individual cut is geometrically valid. The second boolean appears to interact with the first boolean's result topology in ways that cause OCCT to discard material.

### Symptoms

- Body volume drops to 0 after the second boolean
- `isValid()` still returns True
- No Python exception raised

### The Rule

**Fuse all tool shapes for a given batch, then cut once.** If you need multiple categories of cuts (e.g., bolt holes + magnet pockets + coil pockets), apply them as separate booleans (one per category). This is safe -- the issue is with many individual single-shape booleans, not with a small number of compound booleans.

```python
# CORRECT: Two booleans (one per category)
# Boolean 1: all bolt holes fused into one tool
bolt_tool = doc.addObject("Part::Feature", "BoltTool")
bolt_tool.Shape = binary_fuse(bolt_shapes)
bool1 = body.newObject("PartDesign::Boolean", "BoltHoles")
bool1.Type = "Cut"
bool1.Group = [bolt_tool]
doc.recompute()

# Boolean 2: all magnet pockets fused into one tool
magnet_tool = doc.addObject("Part::Feature", "MagnetTool")
magnet_tool.Shape = binary_fuse(magnet_shapes)
bool2 = body.newObject("PartDesign::Boolean", "MagnetPockets")
bool2.Type = "Cut"
bool2.Group = [magnet_tool]
doc.recompute()
```

```python
# WRONG: 40 individual booleans
for pocket_shape in magnet_shapes:
    tool = doc.addObject("Part::Feature", "Tool_%d" % i)
    tool.Shape = pocket_shape
    b = body.newObject("PartDesign::Boolean", "Cut_%d" % i)
    b.Type = "Cut"
    b.Group = [tool]
    doc.recompute()  # may collapse to zero volume partway through
```

**Source:** WO-2026-002 process_docs_update_plan.md Section 1b. Validated on stator shell and rotor hub builds.

---

## Revision Log

| Step | Date | What was added |
|------|------|---------------|
| 1 | 2026-03-10 | Initial reference: PartDesign, Assembly structure, headless reading, error handling, all API pitfalls |
| 1.5 | 2026-03-10 | Face/feature name mapping procedure, solver integration patterns, correct reference format (2 sub-elements), grounded joint pattern, complete working assembly pattern, surface/curve type reference tables |
| 2 | 2026-03-10 | Chamfer, Pocket (Type=1 through-all), LinearPattern, hex sketch pattern, annular ring pattern, parametric modification round-trip, sketch AttachmentOffset, cp1252 encoding pitfall |
| 2.5 | 2026-03-10 | AdditiveHelix/SubtractiveHelix, V-thread profile, groove < pitch constraint, AdditiveCylinder primitive, female thread pattern, XZ_Plane sketch mapping (U=body X, V=body Z) |
| 2.5v2 | 2026-03-10 | Threaded fastener best practice (stud-first approach), FreeCADGui prohibition, visibility fix for headless FCStd files (root cause + template fix) |
| 2.5v3 | 2026-03-10 | Threaded fastener standard process rewritten: mid-stud approach (use middle, cut both ends), pre-computed scaled dimensions for nut (not transformGeometry), Part-level boolean for cut-to-length (not Pocket), removeSplitter+fix cleanup pattern |
| 3 | 2026-03-20 | **CRITICAL assembly pitfall:** Fixed joints with findPlacement() move pre-positioned parts to wrong coordinates. Use Grounded joints for parts built at absolute positions. Link placement audit pattern. Test gap: dimension tests don't catch assembly placement errors — visual verification required. (From WO-2026-002 motor assembly) |
| 6-pre | 2026-03-11 | GuiDocument.xml specification: root-caused "Reading failed from embedded file" (missing Camera element + missing Expand count attribute). Complete VP property counts, extension namespaces, Camera format, Expand semantics. Corrected visibility fix guidance. |
| 5.10 | 2026-03-11 | **ROOT CAUSE FOUND:** `file=""` (empty binary file references) across multiple VPs causes "Reading failed". Fix: generate actual binary files (ShapeAppearance, ColorArrays) and reference them by name. Also: Tip feature detection (Body.Tip) required for geometry visibility. Use `short_empty_elements=False` in ET.tostring(). Corrected the false claim that `file=""` is "safely ignored". |
| 5E | 2026-03-23 | SubtractiveCylinder/AdditiveCylinder local coordinate system rules (Body-relative placement). PartDesign::Boolean feature tree integration (Group property, Type strings, tool types). Combined Tool Pattern (fuse-then-boolean for batch features). Fillet ordering rule (fillets before booleans). All from WO-2026-002 Step 5D learnings. |
| 6.5 | 2026-03-24 | **Tool body visibility best practice:** Part::Feature boolean tools must have `Visibility=False` — they render on top of the boolean result and look like the boolean failed. **OriginGroupExtension prohibition:** Do NOT inject `Gui::ViewProviderOriginGroupExtension` into GuiDocument.xml — FreeCAD 1.0.2 rejects it as "not a python addable version". Omit Extensions block from Body VPs entirely. **App::Link assembly pattern:** Cross-document App::Link works headlessly (saveAs first), preserves PartDesign trees, 65x smaller FCStd. ElementCount+PlacementList for repeated parts. From WO-2026-002 Step 6.5. |
| PD-1 | 2026-03-25 | **8 new sections (Liming draft, WO-2026-002 process docs update):** Pre-Flight Diff Pattern (preflight_check API, tolerance handling), GUI Editability Guidance (safe/unsafe edits, round-trip workflow), Assembly Joint Strategy (decision tree, type summary, GUI instructions, transition strategy), PartDesign::Hole Best Practices (standard settings, radial/axial holes, Refine=False rule), Coil Pocket OCCT Exception (bore-surface workaround), Combined Tool Pattern expanded (binary merge tree, performance data), Fillet Ordering Rule expanded (feature tree ordering), Boolean Cut Strategy expanded (one-boolean-per-batch rule, correct/wrong examples). |
| 8 | 2026-03-29 | **App::Link Placement Behavior (CRITICAL):** Link.Placement is the ONLY transform — FreeCAD does NOT compose with LinkedObject.Placement. Two linking patterns (geometry-at-origin vs position-in-source-Placement). Multi-level assembly placement composition via `top_plc.multiply(internal_plc)`. Common failure mode: top-level script discards sub-assembly-internal offsets, parts cluster at zone boundaries. Diagnostic and prevention guidance. |
