---
name: blender-ae-matchmove
description: Solve moving-camera footage in Blender for painting, sign, or screen inserts in After Effects, and finish residual Corner Pin drift. Use when occlusion or a wall reveal defeats a local planar or point track, or when refining an existing Blender-to-AE matchmove.
metadata:
  version: "0.1.0"
---

# Blender to After Effects matchmove

Deliver a reusable camera solve, editable artwork placement, and a versioned native AE project. For an existing solve, inspect the saved work and address the requested residual; rebuilding the camera is often unnecessary.

## Choose the operation

- **New solve:** Read [workflow.md](references/workflow.md), sections 1–4 and 6. Track static features across scene depth and across the reveal, solve in Blender, fit the target surface, and verify the native AE export.
- **Drift or placement correction:** Read workflow sections 5–6. Determine whether the error is in the camera/export, target plane, absolute placement, or residual movement before changing anything.
- **Worked example:** Read [synthetic-example.md](references/synthetic-example.md) for a fictional drift-correction scenario. Its frame numbers illustrate the method and are not defaults.

## Working method

1. **Inspect and preserve.** Find the footage, existing `.blend`/`.aep`, scripts, and latest version. Probe dimensions, pixel aspect, exact rational rate, frame count, and source timecode. Identify the intended painting and its inner image boundary. Read the current AE project path and active comp before scripting changes. Save updates to a new version and retain the user's artwork edits.
2. **Define coordinates and timing.** Declare source frame numbering and the comp's display convention. With an untrimmed comp at time zero, one-based source frame `f` maps to `(f-1)/fps`; a zero-based AE counter shows `f-1`. Check comp start time, trims and time remapping rather than assuming this mapping. Keep tracking-resolution and final-resolution pixels distinct.
3. **Solve from background coverage.** Use static, distributed features on both sides of a reveal; reject actors, reflections and flashing lights. Stop unreliable tracks at occlusion. Establish overlapping tracks across time and depth. Estimate unknown intrinsics with documented assumptions; do not turn an assumed sensor width into a claimed physical lens measurement.
4. **Separate camera and surface validation.** Check reconstruction validity and per-frame coverage, inspect residuals, and prune supported outliers. Validate withheld observations where possible. Fit the painting plane using its image surface rather than its thick frame. Occluded corners and extrapolated geometry remain uncertain even with a low mean camera error.
5. **Export and read back.** Bake the camera per source frame. Verify Blender projections independently, then compare the same 3D points using native AE `toComp`. Separate scalar X/Y/Z rotations with zero Orientation are a tested fallback if native readback loses an Orientation axis. Confirm all axes and projections in the target AE version; successful script execution is insufficient.
6. **Finish against the actual edge.** Compare the insert to the inner canvas boundary before and after the reported onset. A correction anchored to an already misplaced frame preserves its absolute error. When the edge is hidden, use nearby reliable observations and camera-driven transfer, then blend during occlusion; label inferred intervals. Prefer a small additive 2D corner correction when the global solve remains sound. Maintain both camera-driven and baked insert comps if both are supplied.
7. **Verify the saved result.** Read back expressions and corner values on every frame, confirm earlier protected frames remain unchanged, render native AE frames at the transition and worst drift, and inspect the final visible edge. Report camera error, export agreement, surface fit, and correction interpolation separately. Preserve the original solve and footage.

## Delivery details that matter

- Keep base motion and four editable corner-offset controls separate. When revising a correction, recover the base and replace the total offsets; do not accidentally add the previous correction twice.
- Use linear temporal interpolation for measured offsets unless another curve is justified. Choose key density from a measured pixel tolerance and check intermediate frames. Hold after exit only if the surface stays offscreen.
- Corner Pin order is TL, TR, BL, BR. Keep artwork layer coordinates consistent with comp pixels, or convert through `fromComp`; otherwise a correct projected point can drive the wrong effect coordinate.
- Retain a native `.blend`, a new `.aep`, reusable artwork precomp, diagnostics, and a short timing/assumptions note. Distinguish current corrected projects from older camera-only import scripts and previews.
- Camera tracking does not supply foreground roto, final artwork cropping, blur, grain, or color matching. State remaining work according to what was actually completed.

Application versions and APIs change. Discover installed executables and inspect current Blender RNA/AE property behavior. The references describe a tested approach and its observed failure modes; they are not a promise that one API signature works in every version.
