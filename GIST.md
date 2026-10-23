# Blender to After Effects Matchmove

**Version 0.1.0 · Experimental agent skill**

[GitHub repository](https://github.com/wxcocoxw/blender-ae-matchmove) · [Release 0.1.0](https://github.com/wxcocoxw/blender-ae-matchmove/releases/tag/v0.1.0)

Downloads: [.skill](https://github.com/wxcocoxw/blender-ae-matchmove/releases/download/v0.1.0/blender-ae-matchmove-0.1.0.skill) · [.zip](https://github.com/wxcocoxw/blender-ae-matchmove/releases/download/v0.1.0/blender-ae-matchmove-0.1.0.zip). SHA-256 checksums are included with the release.

A reusable skill for AI assistants working on difficult painting, sign, or screen inserts. When a camera move or wall reveal leaves a planar or point track without useful features, it guides a Blender camera solve, editable After Effects placement, and residual Corner Pin corrections checked against the actual image boundary.

The workflow covers timing and coordinate checks, native AE export verification, preserving existing artwork, and saving corrections in a new project version. It contains instructions, formulas, and technical examples. Blender, After Effects, and suitable access to local files and applications are required.

**Install:** The `.skill` download is a ZIP archive. For Codex, extract the `blender-ae-matchmove` folder into `~/.agents/skills/` (Windows: `%USERPROFILE%\.agents\skills\`). Keep `SKILL.md`, `agents/`, and `references/` together. For Claude, upload the ZIP copy through its Skills settings and enable it.

**Try:** “Use the Blender to After Effects Matchmove skill to track this artwork and fix the drift near the end of the shot.”

The underlying workflow resolved one production shot; broader independent testing remains outstanding. Public examples use a synthetic scene. Production footage, native projects, private paths, and production identifiers are excluded.

By [Paul Melluzzo](https://www.paulmelluzzo.com/).
