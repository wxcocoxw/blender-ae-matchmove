# Hosting and publication

Source: [GitHub repository](https://github.com/wxcocoxw/blender-ae-matchmove).

Downloads: [Release 0.1.0](https://github.com/wxcocoxw/blender-ae-matchmove/releases/tag/v0.1.0), including the `.skill`, `.zip`, and SHA-256 checksums.

Public introduction: [Blender to After Effects Matchmove Gist](https://gist.github.com/wxcocoxw/b520c47f164df363c9d05b594ccdf1e9). The Gist links to the source and release. The experiment article can be linked after it is published; no article URL is assumed here.

Use three complementary destinations:

1. **Personal website:** the experiment story, screenshots, captions, and embedded synthetic demonstration. Link to the source repository and a specific release version.
2. **GitHub repository:** the skill source, installation instructions, changelog, issue reports, and the small reviewed synthetic examples in `examples/`.
3. **GitHub Releases:** the versioned `.skill` and `.zip` downloads and their SHA-256 checksums. Keep release binaries out of Git history. The Gist serves as a short introduction that links to the repository and release.

GitHub documents [Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases) as a way to distribute versioned packages and binary assets. For larger future demonstrations, keep videos on the website's existing media host/CDN and link to them from the repository.

## Local preparation

This directory is a separate sanitized Git root. The surrounding production workspace is deliberately excluded. The repository contains only the allowlisted text files and three reviewed synthetic examples. Its history begins with the reviewed public files and uses a GitHub noreply commit address.

`release/` is the ignored local staging directory for the 0.1.0 `.skill`, `.zip`, and checksum file. Downloads are distributed as release assets. The packaged skill contains only its five instruction and metadata files, not this repository's example media or publication notes. The archive README describes the self-contained package; the repository README also contains repository and release links.

Before publishing updates, review the exact staged files with `git diff --cached --stat` and `git ls-files`. Select a reuse license for the instructions and examples before presenting the repository as open source; no license has been assigned by this publication.

## Media policy

The repository `.gitignore` uses an explicit allowlist. All unreviewed files are ignored, including movie footage, native AE/Blender projects, images, renders, caches, logs, and generated releases. Only the three documented synthetic media files in `examples/` are exceptions.

An ignore rule does not remove an already tracked file or erase history. Check the index and full history before publishing any existing repository. This repository was initialized separately with only reviewed public files from the production workspace.

See GitHub's [ignore documentation](https://docs.github.com/en/get-started/git-basics/ignoring-files).
