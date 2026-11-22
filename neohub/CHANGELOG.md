# Changelog

## 0.4.0

- Build NeoHub fork commit a40f98e https://github.com/lurisin/NeoHub/compare/90bc9d2..a40f98e
- Trouble types: NeoHub decodes the panel's trouble detail messages. Every partition in `full_state` and `partition_update` carries `trouble_types`, the list of active troubles: `ac_failure`, `tamper` (cabinet tamper), `battery_disconnected`, `unknown_<A>_<B>_<C>` (in hex) for any other trouble the panel names, and `unknown`, always last, when the panel reports troubles that no trouble message explains (for example ones that were active before NeoHub connected). The list is the same on every partition whose `troubles` is true and `[]` otherwise; the names do not change in later versions. A new trouble now turns `troubles` on at once instead of at the next re-read. A restore never turns it off by itself: about 3 s after the last trouble is restored NeoHub reads the partition status again, and `troubles` turns off when a read reports no troubles (with a Trouble Refresh Interval of 0 there is no extra read, and the flag follows the troubles the panel names). `full_state` lists the capability `trouble_types`
- Keypad alarms: a keypad alarm (the F / A / P keys) is sent to WebSocket clients as a new message `keypad_alarm` with `partition`, `alarm_type` (`fire` for code 3, `unknown` for any other code, such as 9, which is not identified), the panel's raw `code` and `time` (NeoHub's clock, UTC), also for a partition the client does not know. An `unknown` keypad alarm is not a confirmed Auxiliary or Panic key press. Each session in `full_state` carries the last one as `last_keypad_alarm` (or `null`); it is kept in memory while the panel stays connected and is lost when the panel disconnects or NeoHub restarts. `full_state` lists the capability `keypad_alarms`
- Siren and alarm memory: every partition in `full_state` and `partition_update` carries `siren` (the partition's siren is sounding, from the panel's bell messages; any arm or disarm of the partition also turns it off, so a missed "off" cannot hide the next alarm) and `alarm_memory` (an alarm is in the partition's memory; DSC panels clear it at the next arming). The status re-reads now update the trouble flag, the siren and the alarm memory, still nothing else. `full_state` lists the capability `partition_siren_memory`
- A keypad alarm does not change the partition `status`: it stays `disarmed` or `armed_*`, and clients see no `triggered` for keypad or other alarms. Detect alarms with `keypad_alarm` and `siren`
- A change of `trouble_types`, `siren` or `alarm_memory` alone sends one `partition_update`; no change sends nothing. Decoded trouble, keypad alarm, bell and alarm memory messages are logged at Information. Their layouts are inferred from what one PowerSeries Neo panel sent and are not documented; a message that does not fit is logged at Information with its bytes and ignored
- Clearer rejection texts: `command_result` carries `code_kind` next to `code`: `"response"` when the panel rejected the command, `"nack"` when the panel interface refused it (the two numberings overlap with different meanings), `null` when there is no panel code (session not found, invalid request, no answer in time). `message` is a short English text, for example "Invalid access code", "Panel busy" or "Keypad locked out"; response code 1 reads "Partition not ready to arm" only for arm commands. Replies to commands without `request_id` are unchanged
- A NACK now carries its NACK code in `code` (it was `null`). The DSC Neo integration up to 1.3.0 does not read `code_kind`, so it shows a NACK 1 on an arm as "Partition not ready to arm" (the arm still fails; a working panel is not expected to send that NACK). Update NeoHub and the integration together
- Session takeover fix: when the panel opens a new connection before NeoHub notices that the old one is gone, the new connection now takes over the session ("Session replaced by a newer connection" in the log), keeps the partition and zone state and reads the panel status again; commands go to the new connection. Before, the teardown of the old connection removed the session: the panel stayed connected, but its state was gone and every command failed with "Session ... not found" until the next reconnect, which could be hours later
- Experimental FAP command (Fire / Auxiliary / Panic), off by default. **It raises a REAL alarm: the panel sounds it and reports it to the monitoring station.** The WebSocket message `activate_fap` (`session_id`, `partition`, `fap_type` `fire` / `auxiliary` / `panic` and `request_id`, all required) works only after **Allow FAP commands** is turned on and saved on NeoHub's Settings page (Application Settings, "Panel control"); while it is off, `activate_fap` is refused with "FAP commands are disabled in the NeoHub settings" and nothing is sent to the panel. Only the value saved in NeoHub's settings file counts. While it is on, every page of NeoHub's web UI shows a red banner and `full_state` lists the capability `fap`; NeoHub sends a new `full_state` when the setting changes. The command is sent once and never retried; with no answer the reply is "No answer from the panel: the alarm may still have been triggered". There is no FAP button in the web UI, and the Dev Console does not offer the command. Untested on a real panel: the message layout comes from another open-source ITv2 integration. Turn the setting on only for a test agreed with the monitoring station, and off again afterwards
- The DSC Neo integration fork v1.4.0 adds, per partition, a keypad alarm event entity and "siren" and "alarm in memory" binary sensors, shows `trouble_types` as an attribute of the trouble sensor, uses NeoHub's new rejection texts and `code_kind`, and adds the experimental action `neohub.activate_fap` (no button or other entity). Integration 1.3.0 and older keep working; they ignore the new fields and log a warning "Unknown message type" for each `keypad_alarm` message

## 0.3.0

- Build NeoHub fork commit 90bc9d2 https://github.com/lurisin/NeoHub/compare/77784fa..90bc9d2
- NeoHub re-reads each partition's trouble flag from the panel while the panel is connected. The new per-connection setting "Trouble Refresh Interval" on the Connections page sets the seconds between re-reads: default 60, 0 = off, values from 1 to 9 count as 10. Settings saved before this version load with 60. The re-reads change only the trouble flag, and are skipped while a command, a configuration operation or programming mode is in progress
- `full_state` partitions and `partition_update` carry `troubles` (true while the panel reports a trouble for the partition). A change of `troubles` alone sends one `partition_update`; a re-read that finds no change sends nothing. `full_state` lists the capability `partition_troubles`
- After an arm is rejected with panel code 1, NeoHub re-reads that partition's trouble flag right away, in the background; the reply to the arm command does not wait for it. A Trouble Refresh Interval of 0 turns this re-read off too
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
