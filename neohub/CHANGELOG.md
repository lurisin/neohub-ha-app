# Changelog

## 0.3.0

- Build NeoHub fork commit 90bc9d2 https://github.com/lurisin/NeoHub/compare/77784fa..90bc9d2
- NeoHub re-reads each partition's trouble flag from the panel while the panel is connected. The new per-connection setting "Trouble Refresh Interval" on the Connections page sets the seconds between re-reads: default 60, 0 = off, values from 1 to 9 count as 10. Settings saved before this version load with 60. The re-reads change only the trouble flag, and are skipped while a command, a configuration operation or programming mode is in progress
- `full_state` partitions and `partition_update` carry `troubles` (true while the panel reports a trouble for the partition). A change of `troubles` alone sends one `partition_update`; a re-read that finds no change sends nothing. `full_state` lists the capability `partition_troubles`
- After an arm is rejected with panel code 1, NeoHub re-reads that partition's trouble flag right away, in the background; the reply to the arm command does not wait for it
- Panel notifications that NeoHub cannot decode are logged at Information once per command word per session, and with their payload in hex at Debug (length only for commands that can carry a code)
- Arm rejections with panel code 1 read "Partition not ready to arm" (was "Partition not ready to arm (zone open)"); the panel response code is still 1
- Live trouble notifications from the panel (0x023F) and the trouble detail messages (0x0817, 0x0823) are not implemented: their payload formats are not known, so `trouble_types` is not sent
- The DSC Neo integration fork v1.3.0 adds a trouble binary sensor per partition

## 0.2.0

- Build NeoHub fork commit 77784fa https://github.com/lurisin/NeoHub/compare/706e7f7..77784fa
- WebSocket commands that carry a `request_id` get a `command_result` reply (success, panel response code, message). Replies to commands without `request_id` are unchanged
- Arm rejections with panel code 1 read "Partition not ready to arm (zone open)"
- `full_state` and `partition_update` carry the partition `ready` flag, which is false while the partition is not disarmed. `full_state` lists `capabilities`
- The wait for the panel's answer is limited to 8 s
- Access codes are redacted from logged client messages
- A refusal by the panel interface (NACK) is reported as a failed `command_result`; replies to commands without `request_id` are unchanged
- After an unanswered request to enter programming for a zone bypass, NeoHub sends the exit request so programming is left
- The DSC Neo integration fork v1.1.0 keeps working unchanged; v1.2.0 uses the new replies

## 0.1.0

- Rename the app to "NeoHub (fork)". It installs as a new app; stop the old NeoHub app and turn off its Start on boot first (both use host ports 3072 and 8080)
- Build NeoHub from source instead of copying the prebuilt upstream image
- Build NeoHub fork commit 706e7f7 https://github.com/lurisin/NeoHub/compare/61a1a0e..706e7f7 - upstream 61a1a0e + upstream pull request 45 (remove the exact disconnected WebSocket client) + `arm_home_instant` / `arm_away_instant` WebSocket messages + "Stay Instant" arm button
- Pin the build images: dotnet/sdk 8.0.428, dotnet/aspnet 8.0.34, base-debian trixie-2026.11.0
- Remove the deprecated build.yaml

## 0.0.6

- Update NeoHub https://github.com/BrianHumlicek/NeoHub/compare/4604126..61a1a0e - Accesscode/User configuration

## 0.0.5

- Update NeoHub https://github.com/BrianHumlicek/NeoHub/compare/c4fb28a..4604126 - Installer configuration

## 0.0.4

- Update NeoHub https://github.com/BrianHumlicek/NeoHub/compare/b5e2b56..c4fb28a

## 0.0.3

- Use Home Assistant Base Images
- Add auto discovery

## 0.0.2

- Use image from official repository

## 0.0.1

- Initial release as Home Assistant add-on
