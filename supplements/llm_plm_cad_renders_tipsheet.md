# Test-Driven Rendering (TDR)
## A Method for Producing CAD Animations and Figures With FreeCAD and ffmpeg Using Claude

---

## The Method in One Sentence

**Treat a render campaign like a build: write the acceptance tests for the shot first, prove the look on full-resolution stills, run one full-resolution pull under a progress-based watchdog, and ship through a scripted encode that has its own size and integrity checks.**

Render work fails in three ways: compute is wasted on frames that cannot be reused, long jobs hang without anyone noticing, and finished files are the wrong size or in the wrong place. Every rule below comes from a real incident on a multi-month robotics-vehicle animation project (about 4,000 frames per hero shot, 3840x2160 output, a heavy assembly of roughly 800 objects). The method assumes Claude is the orchestrator and sub-agents run the long jobs.

---

## Why This Works

Naive approach: "Render the animation and send me the video."
- A draft is rendered to save time, the look is approved on the draft, and the whole shot is rendered again at full size
- A run hangs at frame 448 and reports "healthy" for an hour because the process still exists
- Frames land in a cloud-synced folder and upload gigabytes over a metered link
- The final MP4 is 1.6 GB because nobody set a bitrate

TDR approach:
- The shot has a written spec, a numeric test list, and one approved still before any long run
- Every pull is final resolution, so every frame is reusable
- Liveness is measured from output, never from process existence
- Frames live on a scratch drive; only compressed MP4s and deliverable stills reach the synced tree

---

## The Process (4 Prompts)

Each prompt can be run by a separate agent. Pass the previous output as input.

### Prompt 1: Shot Spec and Test List

```
I need a [SHOT TYPE] of [ASSEMBLY] showing [ACTION].

Requirements:
- [Output resolution, fps, duration]
- [Camera behaviour: fixed, tracking, fly-under, hold, pull-out]
- [Colours and materials that matter, and which parts they apply to]
- [Sequence of events with the frame or second each begins]
- [What the viewer must be able to see at the end]

Write the shot spec and around 20-40 tests. Group them:

GEOMETRY TESTS (numeric, run on the model before rendering)
- Each modified solid is valid, closed, one solid
- Volume removed matches the expected volume within tolerance
- Cut features sit at the specified position and angle, both parts of a pair aligned
- Clearance to neighbouring features is at least the stated minimum
- Every other object in the document is unchanged

TIMELINE TESTS (numeric, run on the frame-state JSON)
- Frame count and duration match the spec
- Each event starts at its stated frame
- Camera position is continuous across every segment boundary (no jump, no unplanned pan)
- Hold segments are actually static; motion segments are not

LOOK TESTS (visual, run on full-resolution stills)
- Named parts show the named colour
- Nothing clips, floats, or z-fights at the moments listed
- The end frame frames the subject as specified
```

The spec is the contract. Ambiguities go to the user as questions before any render (for example: which end of a part a feature belongs on, whether two moving parts move together or one stays fixed, which frames are a "draft" and which are a "final").

### Prompt 2: Full-Resolution Proof Stills

```
Render 3-6 stills at FINAL settings (output 3840x2160, supersample 2, so the
internal render is 7680x4320) in ONE FreeCAD launch. Cover: the wide view,
the close-up of the changed feature, one view along the feature axis, and
each moment the look tests name. Write them to the scratch drive. Copy the
ones the user must approve into the synced review folder and give the full path.
```

Stills are cheap only if batched: scene setup (loading the assembly, building terrain, placing every object) takes 10-15 minutes and dominates a short range. One launch, several camera positions. A still rendered below final resolution proves nothing about the final and pays the setup cost a second time.

### Prompt 3: Full Render Under Watchdog

```
Launch the full render detached (not as a child of the agent's shell), with
frames written to the scratch drive. Run a progress watchdog. Report PIDs,
frame directory, log path, and an ETA that accounts for other jobs sharing
the CPU.
```

### Prompt 4: Encode, Verify, Ship

```
Encode the frames to MP4 with the standard recipe (below). Verify with ffprobe
(resolution, frame count, duration, bitrate, size). Report the numbers.
Place the MP4 in the deliverables folder. Move superseded versions to a
holding folder on the scratch drive; never delete.
```

---

## Runbook: How a Render Is Launched (read this before Prompt 2)

**The human never opens FreeCAD, never runs a macro, never clicks Macro > Execute.** Every render, still or animation, is a Python file launched from a terminal by the agent. If an agent's plan says "open the GUI and run this macro", the plan is wrong; rewrite it to the command below.

### The command

Windows, PowerShell, from the project root:

```
$env:PYTHONDONTWRITEBYTECODE = "1"; $env:PYTHONIOENCODING = "utf-8"
& "C:\Program Files\FreeCAD 1.1\bin\FreeCADCmd.exe" path\to\render_script.py
```

- The executable is `FreeCADCmd.exe` (lowercase `freecadcmd.exe` is the same file). Not `FreeCAD.exe`.
- Variant control is by environment variable set in the same shell before the call (shot name, frame range, resume flag, output name; see Principle 5). No script edits per run.
- Scripts have no `if __name__ == "__main__":` guard. FreeCADCmd never sets `__name__` to `__main__`, so a guard makes the script exit 0 with no output.
- Alternative for scripts that must be read as text: `FreeCADCmd.exe -c "exec(open(r'path\script.py').read())"`.
- Long jobs (over about 30 s) launch detached, with stdout to a log file:

```
Start-Process -FilePath "C:\Program Files\FreeCAD 1.1\bin\FreeCADCmd.exe" `
  -ArgumentList "path\to\render_script.py" `
  -RedirectStandardOutput "<scratch>\logs\shot.log" -RedirectStandardError "<scratch>\logs\shot.err" `
  -WindowStyle Hidden -PassThru
```

Record the returned PID and the log path in the plan file.

### "One GUI session" means one window that the script opens itself

Stills and frames need FreeCAD's 3D view (`view.saveImage`), which exists only with the GUI libraries loaded. The render script does this in code, from inside the FreeCADCmd process:

```python
import FreeCADGui as Gui
Gui.showMainWindow()          # a window appears by itself; nobody clicks anything
...
view.saveImage(png, W*SS, H*SS, "Transparent")
...
sys.exit(0)                   # the script ends the process
```

A window appearing on screen during a run is expected. Leave it alone. The script opens it once per launch, loops over every camera position or frame inside that one window, and exits.

Two separate things carry the word "GUI" and are not the same:

| Term | Meaning | Who does it |
|---|---|---|
| Headless build | Geometry edits, tests, the stripped render copy. No window at all. | FreeCADCmd, no `FreeCADGui` import |
| Script-opened GUI session | The render step: `Gui.showMainWindow()` inside the script | The script, once per launch |
| Hand-run macro in the FreeCAD GUI | Not part of this method | Nobody |

### If the run does not work

Send the log, not a request to run it by hand. Check in this order:
1. The log file exists and has content. Empty log and exit 0 means a main-guard in the script.
2. `FreeCADCmd.exe` path resolves (`Test-Path`).
3. The scratch output folder exists and is writable.
4. Liveness per Principle 3: newest-frame age and CPU delta.

---

## Principles

### 1. Render every pull at full resolution

Output 3840x2160 with supersample 2. No 720p drafts from the CAD renderer, no "quick look" runs at reduced size. Measured on the project: about 2.3 s per frame at draft versus 15 s at final, but the scene setup cost (10-15 min per launch) is identical, so a short draft range costs nearly as much as a final one and its frames cannot be reused. If a fast preview is needed, make it by downsampling frames that were already rendered at full size.

Keep the supersampled frames (7680x4320) when disk allows. Downsample to 3840x2160 in the encode step with a Lanczos filter, not in the render script. Frames that were downsampled in-script look like "not full resolution" to a reviewer opening the PNG, and cannot be re-cropped later.

Label every artifact as draft or final in its filename. "Approved the draft" means the look is approved; it does not authorise using the draft as the deliverable.

### 2. Storage discipline

- Frames, textures, model files (FCStd, FCBak, BREP), caches, and intermediate encodes live on a scratch drive outside any cloud-synced folder.
- The synced project tree holds code, logs, JSON, markdown, finished compressed MP4s, and stills the user asked to see.
- A run must scan the synced tree at start and end and report new media in MB (target: 0 for frames).
- Where legacy scripts expect the old path, use a directory junction (`mklink /J`, works without admin on Windows); symlinks fail without admin.
- Before moving any files, grep the code for every filename or directory being moved (include `os.path.join` forms). A broad move once took a dozen small input images that a pipeline still read.
- Cleanup means moving superseded files to a holding folder on the scratch drive for the user to delete. Do not delete versions without explicit approval. Keep one milestone version per stage (before a change, first version with it, latest).

### 3. Run jobs in parallel; measure liveness by output

The machine is CPU-limited and the OS schedules the threads. Two or three FreeCAD command-line processes at once is fine; each runs slower (plan for roughly 20 %+). Do not serialise renders, do not wait for "the renderer to be free", do not kill one job to start another.

Liveness is defined by output:
- newest-frame modification time, and
- CPU-time delta over the polling interval.

A process that exists with zero CPU delta and no new frames is hung. Kill it and resume from the existing frames. Never report "healthy" from process existence. The project saw a hang at frame 448 (no frames, 0 CPU, over an hour) coincide with a GPU driver kernel event; the fix was kill, then resume.

Watchdog limits:
- First-frame grace of 60 minutes (scene setup under CPU sharing exceeded 15 minutes and a 15-minute limit killed two healthy jobs before frame 1).
- After the first frame, 10 minutes without a new frame counts as a stall.
- The runner logs a heartbeat each minute: frames n/N, newest-frame age, CPU delta, free disk.
- Resume reads existing frames, re-renders the newest one, and compares it to the file on disk; a difference above 0.05 (mean pixel) means the resume is not reproducing the render and must stop.

### 4. Detached runners and state on disk

Anything over about 30 seconds runs in the background so the user can keep talking to the orchestrator. Long jobs run as detached processes with a log file, a state JSON, and a documented command to check progress. Agents are told the exact output directories and forbidden ones. Record PIDs, output dirs, and ETAs in the plan file at launch, because context compaction can erase the orchestrator's memory at any time.

FreeCAD's command-line launcher can execute the script a second time in the same launch. Guard against duplicated work by checking for existing frames before rendering.

### 5. One config file, environment-variable control

Every knob (colours, camera heights, frame ranges, output names, modes such as final versus draft) lives in one parameters module and is overridable by environment variable so one script serves every variant. Shot variants are selected by variables such as shot name, frame range, resume flag, and output name, not by copying the script.

Recolouring must be keyed by object index (a census of the assembly), not by material name. A name-keyed table recoloured two unrelated parts that shared a material. Changing a colour requires re-rendering every shot that contains the part; list those shots in the plan before starting.

### 6. Modify the model with tests, backups, and a disposable render copy

- Render from a disposable copy stripped of hardware that the shot does not show (coils, fasteners, internals). Keep a master and a render copy; a heavy model rendered in the GUI once per frame is unusable.
- Before any geometry edit, back up the master, the render copy, and the cache files, and record SHA-1s.
- Write the measurement first: pull the reference feature's dimensions, angles, and pitch from the source model, then build the cut from the measurement. When copying a feature between parts, check that the helix pitch or angle of the receiving part matches; orient the new feature by the receiving part's own geometry and copy only the dimensions that are independent of it.
- Build with tests (Prompt 1 geometry tests). Run the tests on the untouched model first; a test that also passes on the baseline does not discriminate.
- Cross-review geometry with two independent reviewers (one for analytic geometry, one for topology). Then re-measure on the model of record with an independently coded check (ray casts through the feature centres plus control rays 100 mm away that must hit).
- Known kernel failure: a boolean cutter that crosses the parametric seam of a revolved solid can fail. Split the solid at the seam plane, cut each half, fuse. Expect face counts to rise (233 to 1140 on one barrel) and check for leftover seam edges, naked edges, and internal skins after the fuse.
- Check that downstream code keys by object index, not face count or face index, before installing the modified solid.

### 6a. Physical logic gets asked, not guessed

Motion sequences that involve which part stays fixed (a pipe rising with the vehicle attached, versus a vehicle driving into a stationary pipe) are stated by the author. Two wrong physical models were built before the author's plain-language description resolved it. Ask early, with a one-line question and concrete options.

### 7. Camera rules

- A tracking camera follows a Gaussian-smoothed centroid of the moving assembly's bounding box, not a chosen point such as a tool tip.
- Segment boundaries must be continuous in position; a pan appearing before an event is a bug in the boundary, not a stylistic choice. Check by diffing camera positions across every boundary in the frame-state JSON.
- Holds at the end of a tracked segment use a fixed camera pose captured from the last tracked frame.
- New opening or closing sections are inserted by offsetting the frame index of the native timeline (seamless index = native + offset); keep the mapping in the state JSON.
- Draw zoom-outs from the end pose so the transition is one continuous move.

### 8. Compositing overlays

- Render overlays (logo, plaque) with a transparent background and straight alpha. Composite with alpha-over.
- Blur only the alpha mask, never the RGB channels, or a dark halo forms at the edge.
- Lens flares, glints, and bokeh are 2D layers built in PIL/NumPy and combined additively or with screen; the geometry render underneath stays sharp. A flare layer built at half resolution and enlarged 2x is acceptable for soft light; anything with edges is built at final size.
- Verify with numbers: alpha min/max/mean, no negative RGB after compositing, and that frames without overlay are pixel-identical to the plate.

### 9. Who looks at images

Visual judgment needs a model that can look at images. A text-only orchestrator or worker that "confirms" a look is producing a false green. Send frames to an image-capable reviewer, batch several frames in one call, and give it a rubric (colour, symmetry, clipping, z-fighting, framing). Everything else uses numbers: pixel diffs between frames, luminance, bounding boxes, alpha statistics, frame counts, CPU and new-frame progress. Review budgets are finite; use the image-capable reviewer once per decision, not once per iteration.

### 10. Encode recipe (ffmpeg discipline)

Deliverable MP4 in the synced folder:

```
ffmpeg -i input -an -c:v libx264 -preset slow -crf 24 -maxrate 10M -bufsize 20M \
  -pix_fmt yuv420p -movflags +faststart output.mp4
```

- Target about 100 MB for a 2-3 minute 4K shot; CRF 22-28 with a maxrate cap of 8-20 Mbit/s covers the range. A first encode near lossless (95 Mbit/s) produced a 1.66 GB file; recompression to 100 MB at CRF 24 was visually acceptable to the reviewer.
- If a size guess is uncertain, encode with the cap, check the size, and adjust CRF; do not ship the first result unchecked.
- Intermediate encodes (crf 16, per-segment clips, cross-fade chains) live on the scratch drive; only the final pass is written to the synced tree.
- Compose multi-shot cuts by encoding each segment at the intermediate quality, joining with xfade, then one final compression pass.
- Phone or email copies: 1080p, about 25 MB.
- Report size, resolution, frame count, and duration for every MP4 before calling it done. Read the encoder logs; a guard that flags static or duplicate frames (holds are expected) needs a human decision.
- A frame that looks fine in a review player can still be soft if the pipeline upscaled it; assemble supercuts from native-resolution segments only.

### 11. Communication

- Executive summary first: what exists (paths, sizes), what is running (PIDs, frame counts), what is next. Open decisions go in an interactive question, one or two at a time.
- No polling chatter. When a background task completes, report the result.
- Own mistakes in one sentence with the fix.
- The plan file carries a top-of-file status block that a new session can read cold: rules, what exists, what is running, what is paused, and known caveats (for example, "any render launched now includes an unapproved change").

---

## Test Suite Skeleton (render campaign)

| ID | Test | Method |
|---|---|---|
| R1 | Every pull is 3840x2160 SS=2 (7680x4320 internal) | PNG header of the first and last frame |
| R2 | Frame count equals spec | Directory count vs state JSON |
| R3 | No frame is missing in the sequence | Index gap scan |
| R4 | Camera continuous at every segment boundary | Diff camera pose across boundary frames |
| R5 | Hold segments static, motion segments not | Pixel diff between adjacent frames |
| R6 | Named part colours match the spec | Sample pixels in a proof still; image reviewer for the rest |
| R7 | Modified geometry passes its test list | Geometry test JSON |
| R8 | Frames stored outside the synced tree | Scan of the synced tree: new media = 0 MB |
| R9 | Watchdog logs heartbeat with newest-frame age and CPU delta | Log inspection |
| R10 | MP4 resolution, frames, duration, bitrate, size | ffprobe |
| R11 | MP4 size within target | File size |
| R12 | Overlay frames: alpha stats sane, base frames unchanged | Numeric compositing checks |
| R13 | Superseded versions moved, not deleted; milestone versions kept | Directory listing |
| R14 | Plan file status block current | Read the top of the plan |

---

## Failure Catalogue (what went wrong, and the rule it produced)

| Failure | Rule |
|---|---|
| Draft-resolution runs redone at full size | Every pull is final resolution |
| Supersampled frames downsampled inside the render script | Keep supersampled frames; downsample in the encode step |
| "Healthy" reported for a hung job | Liveness = new-frame time + CPU delta |
| 15-minute watchdog killed jobs during scene setup | 60-minute first-frame grace |
| Runner died silently | Heartbeat log, check it |
| Serialised renders behind a long one | Parallel jobs are fine |
| Frames uploaded to a cloud-synced folder | Scratch drive for frames; scan the synced tree at start and end |
| Broad file move broke pipeline inputs | Grep filenames before moving |
| 1.66 GB deliverable MP4 | Standard encode recipe and size check |
| Name-keyed recolour changed the wrong part | Key colours by object index |
| Cutter across a solid's seam failed | Split at the seam, cut, fuse, re-check topology |
| Feature copied with the wrong helix angle | Orient by the receiving part's own geometry |
| Camera panned before an event | Diff camera pose at every boundary |
| Two wrong motion models | Ask which part is fixed |
| Dark halo on overlays | Blur alpha only |
| Ship step looked in the wrong frame folder | Check folder-name suffixes; junction if needed |
| Upscaled tail sections inside a 4K cut | Native-resolution segments only |
| Non-image model "confirmed" the look | Image-capable reviewer for looks, numbers for the rest |

---

## Quick Reference

- Launch = `FreeCADCmd.exe script.py` from a terminal (see Runbook). No hand-run macros, no clicking in the GUI. The script opens its own single window for stills and frames and exits itself
- Full resolution only (3840x2160, SS=2); keep 7680x4320 frames
- Parallel jobs OK; watchdog on output; 60 min first-frame grace, 10 min stall
- Frames off the synced drive; MP4 ~100 MB, x264 slow, CRF 22-28, maxrate cap, faststart
- One params file; environment variables for variants; colours by index
- Model edits: backup, measure, tests, two-reviewer geometry check, independent re-measure
- Move, do not delete; keep milestone versions; update the plan's status block
