# Blender to After Effects Matchmove

Version **0.1.0** · Experimental agent workflow

[Shareable Gist introduction](https://gist.github.com/wxcocoxw/b520c47f164df363c9d05b594ccdf1e9)

[Download .skill](https://github.com/wxcocoxw/blender-ae-matchmove/releases/download/v0.1.0/blender-ae-matchmove-0.1.0.skill) · [Download .zip](https://github.com/wxcocoxw/blender-ae-matchmove/releases/download/v0.1.0/blender-ae-matchmove-0.1.0.zip) · [Release 0.1.0 and checksums](https://github.com/wxcocoxw/blender-ae-matchmove/releases/tag/v0.1.0)

A reusable agent skill for camera solves and surface inserts when a local planar or point track loses useful coverage. It covers static-background tracking, Blender camera reconstruction, target-plane placement, native AE export checks, and editable residual Corner Pin corrections.

This is a workflow guide with formulas and API examples. It does not include an automatic tracker, a one-click exporter, application binaries or footage.

## Package contents

This repository also includes a [short shareable introduction](GIST.md), [synthetic examples](examples/README.md), and [hosting notes](PUBLISHING.md). The installable skill archive contains only the five files listed below. Repository example media is kept separate from that archive.

- `SKILL.md` — skill entry point and operation selection.
- `agents/openai.yaml` — Codex display metadata and example invocation.
- `references/workflow.md` — implementation details and verification.
- `references/synthetic-example.md` — an entirely fictional correction example.
- `README.md` — installation and package overview.

All required references are included and use relative links. No source media, native project files, production logs, private paths or production case study are included.

## Install and use

The `.skill` file is a ZIP archive with a different extension. Rename a copy to `.zip` if your uploader or extraction utility requires it. Keep `SKILL.md`, `agents/` and `references/` together inside the `blender-ae-matchmove` folder.

For Codex, extract that folder into `~/.agents/skills/` (Windows: `%USERPROFILE%\.agents\skills\`). The result should be `~/.agents/skills/blender-ae-matchmove/SKILL.md`. If it does not appear, restart Codex. See the [official OpenAI skill documentation](https://learn.chatgpt.com/docs/build-skills).

For Claude, use the ZIP copy. Enable Code execution and file creation in Settings > Capabilities, then go to Customize > Skills > + > Create skill > Upload a skill. Upload the ZIP and toggle the skill on. See the [official Claude skill instructions](https://support.claude.com/en/articles/12512180-use-skills-in-claude).

Example request:

```text
Use $blender-ae-matchmove to solve this shot in Blender, export an editable
painting insert to After Effects, and check the late drift against the canvas edge.
```

For an existing solve, provide its native files and identify the problem frame range. The skill directs the agent to inspect timing and project identity before editing and to retain the current artwork in a new saved version.

Other agent systems can use `SKILL.md` as the workflow entry point; adapt their installation mechanism and tool access as needed.

Installing these instructions does not give an agent access to desktop applications. Running the complete workflow requires access to the footage, project files and local Blender/After Effects tools; an agent without that access can help prepare scripts and guide the work.

## Tools and compatibility

The core workflow uses Blender and Adobe After Effects. ffmpeg/ffprobe and Python with OpenCV, NumPy and SciPy can support frame extraction, tracking and geometry fitting; equivalent native tools may be substituted. These dependencies are not bundled. Discover installed versions and validate their APIs before executing scripts.

The rotation convention and expressions are examples with stated assumptions, including square pixels and no camera shift. Verify the actual native projections for every export. The guidance includes a fallback for an Orientation-vector write failure observed in AE 26.5; it does not assume every version has that behavior.

The skill supports matchmove and editable artwork placement. Foreground roto and final artwork blur, grain and color treatment remain separate compositing work unless explicitly included in the request.

## Validation status

The underlying workflow resolved one difficult production shot. This public instruction package has been checked for valid skill structure, portable references and removal of production identifiers. A separate hypothetical revision scenario was also used to review its timing and project-preservation decisions.

The production workflow was used on Windows with Blender 5.2.2 LTS, After Effects 2026 / 26.5, Python 3.11, OpenCV, NumPy, SciPy and FFmpeg. These are the observed tools, not a compatibility guarantee or bundled dependencies.

Broader testing on a second independent real-world shot remains outstanding. Public illustrations and the animation use an authored synthetic scene with intentionally introduced drift; they demonstrate the concepts, not an independent tracking benchmark. Lens assumptions, feature thresholds and correction onset must be determined for each new shot.

## Release 0.1.0

Initial public experimental release: camera/surface validation, Blender-to-AE projection checks, additive corner correction, a fictional worked example, and installation instructions. No production footage, project files or shot-specific tracking scripts are included.
