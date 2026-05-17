# `custom` branch

Tracks `paperclipai/paperclip:master` plus a small patch:

- Dockerfile bakes in `chromium` + `chromium-sandbox` + `@earendil-works/pi-coding-agent@latest`
  on top of upstream's production image (alongside the claude-code / codex /
  opencode CLIs upstream already installs).
- Adds `CHROME_BIN` / `PUPPETEER_EXECUTABLE_PATH` env so headless-browser
  tooling finds the system chromium.

## How sync works

`.github/workflows/custom-image.yml` runs daily at 05:30 UTC and on any push
to `custom`. Each run:

1. Rebases the local patch onto `upstream/master` (force-pushes the result).
2. Builds the image and pushes two tags:
   - `ghcr.io/nferro/paperclip:custom-YYYY.MM.DD-<short-sha>` (immutable)
   - `ghcr.io/nferro/paperclip:custom` (moving pointer for convenience)
3. Cuts a GitHub Release whose body lists upstream commits since the previous
   custom build.
4. If `ARGOCD_REPO_TOKEN` is set (PAT with write to `dotpt-private/argocd`),
   commits a tag bump to `applications/paperclip.yaml`.

This branch is **never merged to `master`**. To roll back, deploy a previous
`custom-*` tag.
