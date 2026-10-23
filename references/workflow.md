# Matchmove implementation reference

Use the relevant sections for the current operation. This workflow combines a global 3D camera solve with a separately verified surface insert. It can also finish an existing matchmove without rebuilding it.

Contents: 1. Intake and timing; 2. Tracking and solving; 3. Surface placement and projection; 4. Blender–AE transfer; 5. Residual corrections; 6. Native automation and verification.

## 1. Intake and timing

Probe the source with ffprobe or the available media API. Record dimensions, pixel aspect, exact frame-rate fraction, frame count, timecode, and any crop/retime. Extract frame-accurate working images without altering the source. Grayscale half-resolution frames can speed tracking; retain the scale factor and inspect the original resolution for edge selection.

Write down the mapping before measuring drift. At rate `N/D`, an untrimmed source frame numbered from 1 has comp time:

```text
t = (source_frame - 1) * D / N
source_frame = zero_based_comp_counter + 1
```

This assumes the clip starts at comp time zero and has no time remapping. For existing comps inspect display start, footage in-point, stretch and time remap. A user's screenshot counter may be zero-based, one-based or timecode. Probe the current time to resolve it when possible.

Identify the image-to-frame boundary for a painting replacement. Wood frame, gray bevel, shadow, and canvas edge can be several tens of final-resolution pixels apart. Save a labeled reference crop if ambiguity is likely.

## 2. Tracking and solving

Generate Shi–Tomasi corners and track with pyramidal Lucas–Kanade, or use suitable native Blender tracks. Seed at multiple points in time to maintain coverage when a new wall enters. Track forward and backward; reject large forward/backward discrepancies, poor photometric matches, occluded points and image-margin exits. Remove duplicate seeds. Use conservative foreground masks, including moving shadows/reflections and blinking lights where applicable.

Plot tracks and per-frame coverage. A large total count does not guarantee useful overlap through the reveal. Look for shared static points bridging keyframe pairs and depth diversity; avoid a solve dominated by one plane. Stop and reseed when a point disappears rather than inventing motion across an actor. Choose numerical thresholds for the working resolution, blur and texture; document them with their pixel scale.

Import top-origin image coordinates into Blender clip markers as:

```text
marker.co = (x_tracking / tracking_width, 1 - y_tracking / tracking_height)
```

The clip retains the source dimensions. Keep track run boundaries explicit; use muted boundary markers when necessary to prevent unwanted endpoint extrapolation. Use `scene.active_clip` where supported. In background scripting, the solve operator needs a valid clip-editor area, window region and context override; inspect the installed Blender API rather than assigning undocumented properties.

Use a translating-camera solve when there is parallax. Use a tripod solver only for rotation-only footage. If automatic keyframe choice stalls or chooses poor overlap, select two frames with sufficient shared points and baseline. Solve focal length from a plausible initial guess; refine principal point or distortion only when coverage and evidence support it. Unknown sensor width means that a millimeter focal length is conditional; image-space focal length and field of view are the meaningful calibrated quantities.

Save the first reconstruction. Compute point/track residuals, remove supported outliers and re-solve. Verify that the new reconstruction is valid and includes the expected camera frames; do not accept stale results from a failed solve. Warnings during an optimizer's internal trial steps require investigation but are not by themselves the final solve status.

For withheld checks, remove tracks from the cleaned camera optimization, triangulate their fixed 3D positions from a subset of observations, and evaluate other observations. Report whether selection was independent: tracks selected using the initial solve are useful consistency checks but give an optimistic validation set.

## 3. Surface placement and projection

Fit a plane from reconstructed points that belong to the actual target image. A robust reprojection fit can optimize plane parameters and reference image positions jointly, reducing the influence of weak bundles. Inspect spatial and temporal coverage; good early canvas residuals may leave the late edge poorly constrained.

For the conventions used here, `M_f` is Blender camera-to-world, camera forward is local negative Z, image origin is top-left, and pixels are square. For a homogeneous world point `X`:

```text
p = inverse(M_f) @ [X.x, X.y, X.z, 1]
u = cx - focal_px * p.x / p.z
v = cy + focal_px * p.y / p.z

K = [[focal_px, 0, -cx],
     [0, -focal_px, -cy],
     [0, 0, -1]]
P_f = K @ inverse(M_f)[:3, :]
```

Check this sign convention with a known camera-space point before using `P_f` for triangulation. It differs from the common positive-Z camera convention.

For reference camera identity, a ray from pixel `(u,v)` is `r=[(u-cx)/f, -(v-cy)/f, -1]`. A plane parameterized by `n·X=1` intersects at `X=r/(n·r)`. With another reference pose, transform the ray and its camera origin into world coordinates and use the general ray-plane intersection. Guard near-parallel rays and points behind the camera.

Choose corner locations against the intended crop. Deriving a hidden fourth corner as `TR + BL - TL` in world space assumes a parallelogram; it does not establish the true boundary. Render a sparse diagnostic grid and inspect visible edges through the whole shot. Keep target geometry error separate from camera reprojection error.

## 4. Blender–AE transfer

Bake location and rotation at every source frame and store exact rate/duration. For square-pixel footage with no camera shift, one tested convention is:

```text
AE_position = [W/2 + S*X, H/2 - S*Z, S*Y]
e = M_f.to_euler('ZYX')
AE_X_rotation = degrees(e.x) - 90
AE_Y_rotation = -degrees(e.y)
AE_Z_rotation = -degrees(e.z)
AE_zoom = focal_px
AE_Orientation = [0,0,0]
camera_auto_orientation = disabled
```

`S` is an arbitrary shared world scale, used consistently for camera and all 3D controls. Apply the position mapping to surface corner controls too. Unwrap equivalent Euler angles if needed to avoid unintended inter-frame spins. Validate this conversion for any camera shift, nonsquare pixels or changed transform conventions.

One tested AE 26.5 export path lost Y/Z when writing vector Orientation values. Separate scalar rotation properties resolved it. This is an observed failure of that export path, not a proven universal AE defect. Read back every axis and verify native projections in the target application; use the fallback when the observed behavior warrants it.

Use linear temporal interpolation. Prevent spatial Bezier interpolation from introducing motion between camera samples where inappropriate. Reopen the saved Blender file and compare `world_to_camera_view` with exported numeric projections. In AE evaluate `corner_null.toComp([0,0,0])` at the same times, using null anchor `[0,0,0]`; compare every corner and frame.

For a 2D artwork insert, the Corner Pin coordinates are in layer space. A camera-driven corner expression can use:

```javascript
var p = thisComp.layer("Painting_TL").toComp([0,0,0]);
var q = fromComp(p);
[q[0], q[1]];
```

Keep the artwork layer's anchor/position at zero and scale at 100 if directly baking comp pixel values into Corner Pin, or perform the proper comp-to-layer conversion. Corner order is TL, TR, BL, BR; a homography polygon normally uses TL, TR, BR, BL. Provide a camera-driven comp and a baked Corner Pin alternative when useful, sharing one editable artwork precomp.

## 5. Residual corrections

Diagnose the error before fitting keys:

| Observation | Investigate |
| --- | --- |
| Many static background points drift together | Track quality, intrinsics, camera solve, timing |
| Blender is stable but native AE disagrees | Transform conversion, rotation axes, zoom, pixel aspect |
| Background is stable but insert boundary misses | Target plane, corner placement, crop, local residual |
| Relative movement improves but a constant gap remains | An already misplaced correction reference |

Measure the physical inner edge at several heights and times. A vertical edge can constrain horizontal offset and shear; it does not independently determine every component of a four-corner homography. Use reliable corners or other edges for vertical/depth constraints. Do not claim all four corners are independently tracked when only one edge is visible.

An economical correction is `x' = x + a*y + b`, `y' = y + c`. Normalize a visible near-vertical line to `l=[1, m, k]`, so `x + m*y + k = 0`. If the base line is `l_base` and the observed line is `l_actual`:

```text
a = l_base.m - l_actual.m
b = -l_actual.k - l_actual.m*c + l_base.k
dx_i = a*y_i + b
dy_i = c
```

If individual corners retain different known `dy_i`, substitute each in `dx_i = a*y_i - l_actual.k - l_actual.m*dy_i + l_base.k`. These equations place the projected right-edge endpoints on the observed line. Normalize differently for near-horizontal edges where the first line coefficient approaches zero.

When comparing against a reference via a homography `H` that maps reference pixels to the current frame, transfer a line with `inverse(H).T @ l_reference` and renormalize. A generic line detector needs geometric/spatial filtering and visual identity checks; it can lock onto the outer bevel or an actor's hair. During occlusion, transfer from the nearest trustworthy edge measurement. A neighboring border can support a constrained continuation after the canvas exits, but label that inference.

Choose an unchanged anchor before the requested onset. Blend to the measured correction during a suitable occlusion if that preserves the visible track. An occlusion that begins earlier does not authorize changing frames before the requested onset. For revisions, preserve the previously evaluated corners and offsets there; unchanged does not mean resetting existing good corrections to zero. Frame numbers, blend duration and smoothing are per-shot choices. Retain existing good vertical corrections when the user only requests a horizontal adjustment.

Store four additive Point Control offsets separately from the camera or baked base motion. For baked Corner Pin use `value + offset`; for the camera-driven expression add the offset to the projected base. If replacing an old correction, subtract its evaluated offset from captured corners first, then install the new total correction. Rename/reuse controls instead of layering the same correction twice.

Reduce the measured per-frame curve to editable linear keys by inserting the frame with the largest interpolation deviation until every corner is within a chosen final-pixel tolerance. Check intermediate frames, zero-change frames and the final visible frame. A held offset after exit is appropriate only while the surface remains offscreen. Distinguish interpolation deviation from accuracy against the physical scene.

## 6. Native automation and verification

For deterministic AE changes, native JSX can be preferable to UI clicking when available and permitted. On Windows, one execution option is `AfterFX.exe -r <absolute-script>`; otherwise use an available supported script runner, such as File > Scripts > Run Script File. Discover the executable or runner and test a harmless probe. Use completion/error files and bounded waits. Do not launch repeated blind instances when a script produces no output.

Before mutation, probe `app.project.file`, comp names, dimensions, rate and target layer/property identities. Refuse to write if an unrelated project is active. Capture the required base values and save a new project version before applying edits in an undo group. After validation, save again. Do not call `app.newProject()` or close an unrelated live project to simplify scripting.

Use runtime-supported ExtendScript syntax and property match names. Adding an effect to an indexed group can invalidate previously held property objects; reacquire them after changes. Use correct tangent dimensionality if setting spatial interpolation. Emit scalar/vector logs explicitly, such as `vector.join(',')`. Node syntax checks do not establish AE execution correctness. Parse data safely; do not evaluate untrusted file contents as code.

Check all frames numerically, including expression errors and both insert alternatives. Render representative frames with native AE `saveFrameToPng`, inspect edge crops at useful magnification, and retain the results. Subpixel numeric agreement between Blender and AE proves the export, not the fit of the plane to the actual image. Likewise machine-precision agreement with intended offsets only proves that the offsets were installed correctly.

Deliver versioned native files, portable source references where feasible, the artwork precomp, diagnostics and a note explaining timing, lens assumptions, selected boundary, inferred intervals and remaining compositing work. Clearly flag baseline import scripts or previews that do not contain later corrections.
