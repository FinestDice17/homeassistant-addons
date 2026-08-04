# RemoteTerm for MeshCore — USB Fix

A packaging-only fork of
[bhaggs/ha-remoteterm-app](https://github.com/bhaggs/ha-remoteterm-app) that can talk to
**any** USB serial device the host exposes, not just the first four.

## The problem

Upstream hardcodes a four-entry device map:

```yaml
devices:
  - /dev/ttyUSB0
  - /dev/ttyUSB1
  - /dev/ttyACM0
  - /dev/ttyACM1
```

On a host with several radios attached this fails. In my case a ZBT-2 Zigbee coordinator and
a ZWA-2 Z-Wave stick already held ACM0 and ACM1, so an M5Stack Nesso N1 enumerated as
`ttyACM4`:

```
cdc_acm 1-2.4.2:1.0: ttyACM4: USB ACM device
```

That node does not exist inside the container, so the add-on looped every five seconds:

```
app.radio - WARNING - Could not connect to serial port
/dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_...-if00
app.radio - INFO - Attempting to reconnect to radio...
```

Compounding it, upstream sets neither `udev` nor `uart`, so `/dev/serial/by-id/` symlinks
aren't populated in the container — even a correctly mapped device fails when addressed by
its stable by-id path.

## The fix

The static list is gone, replaced with Supervisor's auto-mapping:

```yaml
uart: true    # map every host UART/serial device into the container
udev: true    # populate /dev/serial/by-id/ symlinks
```

Same approach Zigbee2MQTT uses. Devices are picked up regardless of enumeration order, and
stable by-id paths resolve correctly.

Slug is `remoteterm_usb` so this installs alongside the upstream add-on rather than
colliding with it. Stop and disable the store version before starting this one — two
instances will fight over the same serial port.

## Notes

- Pulls the same prebuilt `ghcr.io/bhaggs/ha-remoteterm-app/remoteterm-app` image as
  upstream, so there's no local build step.
- To build on-device instead: rename `Dockerfile.build-fallback` → `Dockerfile` and remove
  the `image:` line from `config.yaml`. `run.sh` is included unmodified.
- Tracks upstream v3.16.2 (commit `52b8f5c`). Bump `version:` in `config.yaml` to follow new
  releases.
- See [`INSTALL.md`](INSTALL.md) for the full walkthrough and [`DOCS.md`](DOCS.md) for
  add-on usage/options.

## Credit

All application code is Bryan Haggerty's — this add-on changes packaging metadata only.
MIT, same as upstream. If upstream adopts `uart: true`, this fork should be retired in
favour of the store version.
