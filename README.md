# NeoHub (fork) app for Home Assistant

Home Assistant app repository for NeoHub, a bridge between DSC PowerSeries Neo
alarm panels (ITv2/TLink protocol) and Home Assistant.

This is a fork of the NeoHub Home Assistant app
(https://github.com/piotrantosik/neohub-ha-app). NeoHub itself is
https://github.com/BrianHumlicek/NeoHub. All credit for both goes to their
authors. This fork builds a NeoHub fork
(https://github.com/lurisin/NeoHub/tree/toronto) from source at a pinned
commit.

## Installation

The fork's changes are on the `toronto` branch, so the repository URL must end
in `#toronto`. Without it, Home Assistant uses the default branch, which is the
unmodified upstream app.

1. In Home Assistant go to **Settings > Apps > Install App**
2. Click the three-dot menu (top right) and select **Repositories**
3. Add this repository URL: `https://github.com/lurisin/neohub-ha-app#toronto`
4. Find **NeoHub (fork)** in the app store and click **Install**

The app is built on your Home Assistant host when you install or update it.
That needs internet access and takes a few minutes: it downloads the .NET 8
SDK image (about 340 MB compressed), the NuGet packages and the NeoHub source.

Requires Home Assistant Supervisor 2026.04.0 or newer. Older Supervisors pass
an Alpine `BUILD_FROM` when `build.yaml` is missing, which overrides the
Debian base image set in the Dockerfile.

### Only one NeoHub app at a time

This app and the upstream NeoHub app both bind host ports 3072 and 8080, so
only one of them can run. Before starting this one, stop the old NeoHub app and
turn off its **Start on boot**, or uninstall it.

Because it comes from a different repository, this app is installed as a new
app with its own slug and host name, and it starts with empty NeoHub settings.
Enter the panel connection settings again in the NeoHub web UI.

The DSC Neo integration still points at the old app's host name. It cannot
change the host of an existing entry, and it offers no discovery card while an
entry exists. Delete the old entry under **Settings > Devices & services**,
then start or restart this app and accept the discovery card that shows this
app's host name, or add **DSC Neo** by hand with that host name (shown on this
app's **Info** page while it runs) and port 8080.

## How this fork differs from the upstream app

| | Upstream app 0.0.6 | This fork 0.1.0 |
|---|---|---|
| NeoHub code | upstream NeoHub 61a1a0e | NeoHub fork commit 706e7f7: upstream 61a1a0e + upstream pull request 45 (remove the exact disconnected WebSocket client) + `arm_home_instant` / `arm_away_instant` WebSocket messages + "Stay Instant" arm button |
| How NeoHub gets into the app | copied out of the prebuilt image `ghcr.io/brianhumlicek/neohub:sha-61a1a0e` | built from source in the app's Dockerfile |
| .NET | whatever `dotnet/aspnet:8.0` was when that image was built | `dotnet/sdk:8.0.428` and `dotnet/aspnet:8.0.34` |
| Base image | `{amd64,aarch64}-base-debian:latest` (moving tag) | `base-debian:trixie-2026.11.0` |
| `build.yaml` | yes | removed (deprecated since Supervisor 2026.04.0) |
| Updates | a daily workflow proposes a new image tag and version | re-pin by hand (see below) |

### How the upstream app is built

NeoHub's own GitHub Actions workflow runs on every push to its `master` (or
`main`) branch. It builds NeoHub's Dockerfile for `linux/amd64` and
`linux/arm64`: an SDK stage (`mcr.microsoft.com/dotnet/sdk:8.0`) runs
`dotnet restore` and `dotnet publish -c Release -o /app/publish
/p:UseAppHost=false` with the commit SHA and the run number, and a runtime
stage (`mcr.microsoft.com/dotnet/aspnet:8.0`) holds the result. The image is
pushed as `ghcr.io/brianhumlicek/neohub:sha-<short sha>`.

The upstream app has no `image` in `config.yaml`, so the Supervisor builds it
locally. Its `build.yaml` passes that NeoHub image as `NEOHUB_IMAGE` and the
Home Assistant Debian base image as `BUILD_FROM`, and its Dockerfile copies
`/usr/share/dotnet` and `/app` out of the NeoHub image into the base image.

### How this fork is built

The Supervisor builds the app locally with `docker buildx build`, using the
`neohub` folder as the only build context. `neohub/Dockerfile` has three
stages:

1. `dotnet/sdk:8.0.428`: fetches commit `NEOHUB_COMMIT` from `NEOHUB_REPO`
   (`git fetch --depth 1` by full SHA) and stops the build unless exactly that
   commit is checked out. It then runs the same `dotnet restore` and
   `dotnet publish` as upstream NeoHub's Dockerfile, with `GITHUB_SHA` set to
   the pinned commit and `GITHUB_RUN_NUMBER` set to the app version.
2. `dotnet/aspnet:8.0.34`: only the source of the .NET runtime.
3. `BUILD_FROM` (default `ghcr.io/home-assistant/base-debian:trixie-2026.11.0`):
   the same layout as the upstream app (`/usr/share/dotnet`, `/app`,
   `NEOHUB_PERSIST_PATH=/config`, the s6 run script that registers discovery
   and starts NeoHub).

NeoHub's menu then shows the pinned commit (`GitHub 706e7f7`) and the app
version as the build number (`Build 0.1.0`).

.NET 8 is end of life. `8.0.428` and `8.0.34` are the last .NET 8 image tags
and get no more fixes; moving to a newer .NET is a change to NeoHub itself.

### Re-pinning NeoHub

1. Push the new NeoHub commit to `https://github.com/lurisin/NeoHub` and keep
   it reachable from a branch or, safest, a tag (for example `v0.2.0`): every
   install, update and rebuild fetches the pinned commit from there.
2. Set `NEOHUB_COMMIT` in `neohub/Dockerfile` to its full 40-character SHA.
   Short SHAs, branch and tag names make the build fail.
3. Bump `version` in `neohub/config.yaml` and add a `neohub/CHANGELOG.md`
   entry. Home Assistant offers the update, which rebuilds the app, only when
   the version changes.
4. Push to `toronto`, then in Home Assistant check for updates and update the
   app.

## Apps provided by this repository

- **[NeoHub (fork)](neohub/DOCS.md)** - DSC PowerSeries Neo alarm panel management via ITv2/TLink protocol
