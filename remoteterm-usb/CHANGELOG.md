# Changelog

All notable changes to this Home Assistant app are documented in this file.

Version numbers match the upstream RemoteTerm for MeshCore release this app packages.
This is a packaging-only fork of [bhaggs/ha-remoteterm-app](https://github.com/bhaggs/ha-remoteterm-app)
that adds `udev: true` so `/dev/serial/by-id/` paths resolve. All app code is upstream's.

## 3.26.9

_Updated from 3.16.2. This one release covers every upstream release from 3.17.0 to 3.26.9._

**Before updating:** this version upgrades the RemoteTerm database, and the upgrade can't be undone.
Take a Home Assistant backup of this app first (**Settings → System → Backups**), or turn on
"Keep backup of the last version" in the update dialog. Restoring the backup is the way back to 3.16.2.

### Packaging changes

- **Upstream source moved** to [statico/remoteterm-meshcore](https://github.com/statico/remoteterm-meshcore). The original jkingsman project is paused after 3.17.1; the fork continues from it with the same configuration and data format.
- The upstream image now runs on Python 3.14 (was 3.12).
- Stopping the app no longer shows "error" in Home Assistant. A real crash is still reported as an error.
- The app no longer downloads upstream's dev tools on every start, so it starts faster and doesn't need internet to start.
- Stopping the app during its first minute now shuts down cleanly, and a crash at start-up is reported immediately instead of after 60 seconds.
- Kept from this fork: `uart: true` and `udev: true`, the `remoteterm_usb` slug, and all your options, so your serial by-id path and settings carry over unchanged. (Upstream now ships `uart: true` itself; `udev: true` is still only here.)
- The build-fallback `Dockerfile` now uses the statico image, pinned to 3.26.9.

### RemoteTerm changes (3.17.0 → 3.26.9)

**Radio and sending**
- Outgoing messages (UI, API, bots) wait for the previous message to be repeated plus a few seconds of quiet air, so back-to-back messages stop colliding at repeaters. New "Max Gap Between Sent Messages" setting in Settings > Radio (default 10 seconds, 0 turns it off).
- Sending no longer locks the message box: messages queue and go out in order while you keep typing.
- Channel auto-resend waits longer (6 seconds) and for quiet air, so messages no longer go out twice.
- Sending text over the radio's byte limit now returns an error instead of being silently truncated.
- Contacts of unknown type are no longer loaded onto the radio, which stops the repeating "Could not load all desired contacts" error.
- Radio contact sync no longer stalls on large message databases (it could hang post-connect setup for 5+ minutes and stop bots replying).
- Per-contact regional override: the globe in the chat header now works for DMs, not just channels.

**Repeaters and telemetry**
- Telemetry checks on monitored repeaters retry up to 3 times, 30 seconds apart, so distant repeaters on weak links still get logged.
- Latest battery reading (volts or percent) and temperature can show in the sidebar for monitored repeaters (enable in Local Configuration).
- Repeater login auto-retries with flood routing when no response is heard.
- CSV export for repeater telemetry.

**Rooms**
- Rooms log in automatically (saved password or guest), sync in the background, and open straight to chat. Added Sync Now and Re-enter Password.
- Room status and controls moved into the chat header.
- Several fixes for room login state leaking between rooms and duplicated toolbars.

**Messaging and UI**
- Mentions list in the sidebar with an unread badge.
- Send emoji reactions and GIFs (paste a Giphy link) compatible with MeshCore Open; MeshCore One reactions now render properly.
- Optional Ollama-powered summaries of unread channel messages (off by default).
- Contact pane has a "Send direct message" button.
- Much faster loading of messages, unread counts and contacts, smoother scrolling on mobile, skeleton placeholders instead of spinners, and a large frontend upgrade (React 19).
- Fixed the empty band at the top on iPhone when shown inside Home Assistant.
- Web push notifications now work in Microsoft Edge.
- Toggle for showing hop width inline on channel messages, packet analyzer from the visualizer feed, and customizable date binning on the map.

**Bots**
- Bots can opt into `message_id`, `paths` (with SNR/RSSI/time) and `is_room` kwargs.
- Bot replies can be routed with `"dm"` or `"channel"`, long replies are split on word boundaries, and each bot has an execution timeout (default 30 seconds).
- Replies to room posts now go to the room instead of a DM.

**Security and dependencies**
- certifi bumped to remove compromised emSign root certificates, plus several dependency bumps for security alerts.

For the full details see the [upstream changelog](https://github.com/statico/remoteterm-meshcore/blob/main/CHANGELOG.md).

## 3.16.2

- Tracks upstream RemoteTerm for MeshCore 3.16.2 (zero-hop repeater region discovery).
