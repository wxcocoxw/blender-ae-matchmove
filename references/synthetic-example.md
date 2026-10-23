# Fictional example: fix an insert after a wall reveal

This scenario is entirely illustrative. It contains no production footage, project identifiers, measured production results or camera-report metadata. The timing and control names below are invented teaching examples.

## Request and available evidence

A user has a stable camera solve but the inserted artwork sits too far right from AE counter frame 120 onward. They want an updated project that preserves their artwork and earlier alignment. Both a camera-driven insert and a baked Corner Pin alternative share an artwork precomp.

The hypothetical source is untrimmed, square-pixel, 25 fps, with source frame 1 at comp time zero and a zero-based AE display. The inner image edge is obscured over source frames 119–126 and clear at source frame 127. An outer frame bevel remains partly visible, but it is not the artwork boundary. Old additive corner controls already exist.

| Event | AE counter | One-based source frame | Comp time |
| --- | --- | --- | --- |
| Last protected frame | 119 | 120 | 4.76 s |
| First allowed correction | 120 | 121 | 4.80 s |
| Clear edge observation | 126 | 127 | 5.04 s |

Confirm these conventions in the real project before applying equivalent reasoning to another shot.

## Diagnose before changing the camera

Compare several static background points and the insert corners in Blender and native AE. If native projections agree and the background stays aligned, inspect the target plane and crop rather than rebuilding the camera immediately. A low camera reprojection error alone does not establish exact canvas placement.

Inspect the inner image edge when clear. A correction that merely preserves the insert's position at an already misplaced reference frame can improve relative movement while retaining the visible gap. The requested finish needs an absolute edge alignment.

## Prepare the correction

Measure the inner edge at source frame 127 at multiple heights. Use the base quad homography to transfer that line backward through the occlusion. Mark those earlier estimates as inferred. A lone edge supports a constrained horizontal offset/shear adjustment; it does not independently determine all four corner coordinates.

Retain the evaluated insert through source frame 120. Blend the new horizontal correction beginning at source frame 121, using the remaining occlusion to reach the trustworthy visible measurement. The earlier start of the occlusion does not authorize modifying protected frames. Retain existing good vertical offsets.

Capture the base by subtracting the existing evaluated corner-control offsets from the evaluated corners. Install the new total offset curves in reusable controls such as `Insert TL offset` through `Insert BR offset`. Do this for both alternatives, preserving the shared artwork precomp. Replacing total offsets avoids adding an earlier correction twice.

Select linear key density from a stated final-pixel interpolation tolerance. Test between keys and at the first/last visible frames. Holding offsets after exit is appropriate only while the target remains offscreen. Exact magnitudes and keyframes require the real source observations; this example supplies none.

## Save and verify

Probe the current project path and expected comp/layer identities before editing. If a different project is open, leave it untouched; measurements, a timing map and guarded update code can be prepared from the intended saved assets. Execute the update only in the intended project, preserving unsaved artwork changes when saving a new version.

Read back both alternatives over all frames. Confirm protected frames match the pre-edit evaluated result, expressions are valid, and installed offsets match the intended curves. Render the transition, a clear-edge frame and the worst visible error for inspection. Report export agreement, edge-fit error and interpolation deviation separately.

Deliver the versioned project and a short note identifying controls, timing conventions, inferred intervals and remaining roto/finishing work. A native script that successfully applies the keys has established implementation correctness; the rendered edge review establishes whether the chosen correction looks right.
