# Upstream proposals for the NeoHub Home Assistant app

Upstream: https://github.com/piotrantosik/neohub-ha-app

**Nothing from this fork is proposed upstream, and nothing has been submitted**
(no pull request, issue or comment). Nothing may be submitted without the
owner's explicit OK.

## Why nothing is proposed

Every commit on the `toronto` branch is specific to this fork:

| Commit subject | Why it stays in the fork |
|---|---|
| `Build NeoHub from source at a pinned commit` | It builds a commit of the NeoHub fork (https://github.com/lurisin/NeoHub/tree/toronto), not upstream NeoHub. The upstream app uses the image that NeoHub's own CI publishes for each upstream commit. |
| `Rename the app to NeoHub (fork), version 0.1.0` | Fork name, fork URL and fork version. |
| `Point the repository and docs at the fork` | Fork URLs and fork-specific instructions. |
| `Add UPSTREAM_PRS.md` | This file. |
| `Remove the scheduled NeoHub image bump workflow` | The workflow edits `build.yaml`, which this fork removed; upstream still needs it. |

## Generic ideas that could be offered later

These are ideas only, not drafts. Each would have to be written as a separate,
minimal change against upstream `main`, and only with the owner's OK.

1. **Move the build parameters out of the deprecated `build.yaml`.**
   Evidence: Supervisor 2026.04.0 deprecated `build.yaml`. When the file is
   present, `supervisor/apps/build.py` logs "uses build.yaml which is
   deprecated. Move build parameters into the Dockerfile directly." Without
   `build.yaml`, Supervisor does not pass `BUILD_FROM` at all (it still passes
   `BUILD_VERSION` and `BUILD_ARCH`). Upstream could keep its prebuilt-image
   approach and give the Dockerfile defaults, `ARG BUILD_FROM=...` and
   `ARG NEOHUB_IMAGE=...`. Its daily workflow would then edit the Dockerfile
   instead of `build.yaml`. This also removes the two BuildKit
   `InvalidDefaultArgInFrom` warnings that the current Dockerfile produces,
   because its `ARG`s have no defaults. Supervisor versions before 2026.04.0
   still pass `BUILD_FROM` when `build.yaml` is missing (default
   `ghcr.io/home-assistant/{arch}-base:latest`, which is Alpine), and that
   overrides the Dockerfile default, so this change would raise the minimum
   Supervisor version to 2026.04.0.

2. **Pin the Home Assistant base image to a dated tag.**
   Evidence: `build.yaml` uses `ghcr.io/home-assistant/{amd64,aarch64}-base-debian:latest`.
   In https://github.com/home-assistant/docker-base, `latest` follows
   `DEBIAN_LATEST`, which is now `trixie` (Debian 13), so the base changed
   under the app without a commit. Dated multi-arch tags such as `base-debian:trixie-2026.11.0` exist,
   and the docker-base README prefers the multi-arch `base-debian` image over
   the arch-prefixed ones since 2026.03.1. A pinned tag would make rebuilds
   reproducible. Base-image bumps would then need their own update path.

3. **Use the `app_config` map type instead of `addon_config`.**
   Evidence: validating `neohub/config.yaml` with the Supervisor's own
   `SCHEMA_APP_CONFIG` logs "App 'NeoHub' uses legacy map type
   'addon_config'; use 'app_config' instead." The legacy map types are
   deprecated as of Supervisor 2026.07 (the warning is in
   `supervisor/apps/validate.py` since 2026.07.1). In current Supervisor both
   types mount the same app config folder at `/config`
   (`supervisor/docker/app.py`). Older Supervisor versions do not know
   `app_config`, so this change would raise the minimum Supervisor version.
   This fork keeps `addon_config` unchanged.
