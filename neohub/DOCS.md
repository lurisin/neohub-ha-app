# Home Assistant App: NeoHub (fork)

A real-time web portal for monitoring and controlling DSC PowerSeries Neo alarm panels
via the ITv2 (TLink) protocol.

This app builds NeoHub from source at a pinned commit of the NeoHub fork
https://github.com/lurisin/NeoHub/tree/toronto (90bc9d2): upstream NeoHub 61a1a0e plus
upstream pull request 45 (remove the exact disconnected WebSocket client), the
`arm_home_instant` / `arm_away_instant` WebSocket messages, a "Stay Instant" arm button,
`command_result` replies to WebSocket commands that carry a `request_id`, the partition
`ready` flag in `full_state` / `partition_update`, and the partition trouble flag
`troubles` in `full_state` / `partition_update`, kept current by re-reading it from the panel
(**Trouble Refresh Interval** on NeoHub's Connections page, default 60 seconds, 0 = off). The
[repository README](https://github.com/lurisin/neohub-ha-app/blob/toronto/README.md) explains
how the app is built and how to pin a different NeoHub commit.

Requires Home Assistant Supervisor 2026.04.0 or newer. Older Supervisors pass an Alpine
`BUILD_FROM` when `build.yaml` is missing, which overrides the Debian base image set in the
Dockerfile.

## Only one NeoHub app at a time

This app and the upstream NeoHub app both bind host ports 3072 and 8080. Before starting
this app, stop the old NeoHub app and turn off its **Start on boot**, or uninstall it.
This app starts with empty NeoHub settings, so enter the panel connection settings again
in the NeoHub web UI.

The DSC Neo integration still points at the old app's host name. It cannot change the host
of an existing entry, and it offers no discovery card while an entry exists. Delete the old
entry under **Settings > Devices & services**, then start or restart this app and accept the
discovery card that shows this app's host name, or add **DSC Neo** by hand with that host
name (shown on this app's **Info** page while it runs) and port 8080.

## Support

- [This app fork](https://github.com/lurisin/neohub-ha-app/tree/toronto)
- [NeoHub fork](https://github.com/lurisin/NeoHub/tree/toronto) - the fork-only features
  (`arm_home_instant`, `arm_away_instant`, the "Stay Instant" button, `command_result`, the
  partition `ready` and `troubles` flags, the trouble re-reads) are maintained there; do not report problems with them upstream
- [Upstream app](https://github.com/piotrantosik/neohub-ha-app)
- [NeoHub issues](https://github.com/BrianHumlicek/NeoHub/issues) - only for problems that also
  occur with upstream NeoHub
