# RemoteTerm for MeshCore — local fork with USB/serial fix

## What was broken

Upstream `config.yaml` hardcodes the device map:

```yaml
devices:
  - /dev/ttyUSB0
  - /dev/ttyUSB1
  - /dev/ttyACM0
  - /dev/ttyACM1
```

Your M5 Nesso N1 enumerated as **`ttyACM4`** (kernel: `cdc_acm 1-2.4.2:1.0: ttyACM4: USB ACM device`).
ACM0/ACM1 are already occupied by the ZBT-2 and ZWA-2, so the Nesso will never land in that
static window. The node simply does not exist inside the container, which produced the
5-second reconnect loop:

```
app.radio - WARNING - Could not connect to serial port
/dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_48:F6:EE:C4:AF:F0-if00
```

Second problem: upstream sets neither `udev: true` nor `uart: true`, so `/dev/serial/by-id/`
symlinks aren't populated in the container either — even a correctly mapped device would
fail when addressed by its stable by-id path.

## The fix

Static `devices:` list removed, replaced with:

```yaml
uart: true    # auto-map every host UART/serial device into the container
udev: true    # populate /dev/serial/by-id/ symlinks
```

This is exactly what Zigbee2MQTT does. Any USB serial device HA can see, RemoteTerm can now
open — current, future, and after any renumber.

Slug changed to `remoteterm_usb` (installs as `local_remoteterm_usb`) so it coexists with the
store version instead of colliding.

## Install

1. **Copy the folder.** Via Samba, drop `remoteterm-usb/` into `\\homeassistant\addons\`.
   Or in Studio Code Server, create `/addons/remoteterm-usb/` and paste the files in.
   Final path should be `/addons/remoteterm-usb/config.yaml`.

2. **Stop and disable the store version.** Settings → Add-ons → RemoteTerm for MeshCore →
   Stop, and set Start on boot = off. Two instances fighting over the same serial port will
   both fail. (Leave it installed for now as a rollback.)

3. **Refresh the store.** Settings → Add-ons → Add-on Store → ⋮ → Check for updates.
   Hard-refresh the browser if it doesn't appear. "RemoteTerm for MeshCore (USB Fix)" shows
   up under **Local add-ons**.

4. **Install it.** It pulls the same prebuilt ghcr.io image upstream uses — no local build,
   should take under a minute.

5. **Configure.** Set `serial_port` to the stable by-id path:

   ```
   /dev/serial/by-id/usb-Espressif_USB_JTAG_serial_debug_unit_48:F6:EE:C4:AF:F0-if00
   ```

   Baudrate 115200. Carry over `disable_bots: true` and anything else you had customized.

6. **Start**, then check the log. You want the reconnect warnings gone and a successful
   connect. Enable Watchdog and Start on boot once it's confirmed working.

## If the image pull fails

Fall back to building locally on the Pi:

1. Rename `Dockerfile.build-fallback` → `Dockerfile`
2. Delete the `image:` line from `config.yaml`
3. Reinstall

`run.sh` is included and unmodified for that path. The build pulls
`docker.io/jkingsman/remoteterm-meshcore` and takes a few minutes on the Pi 5.

## Keeping up with upstream

When bhaggs releases a new version, bump `version:` in `config.yaml` to match and reinstall.
If upstream ever adds `uart: true` themselves, drop this fork and go back to the store copy.
Worth opening an issue on `bhaggs/ha-remoteterm-app` — the static four-device map is a real
bug that hits anyone with more than a couple of serial devices.
