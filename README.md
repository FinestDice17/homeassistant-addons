# HAOS ADD-ON's

A single Home Assistant add-on repository holding all of my Supervisor add-ons, so they
install and update together instead of living behind separate repository URLs.

## Add to Home Assistant

Settings → Add-ons → Add-on Store → ⋮ → Repositories → add:

```
https://github.com/FinestDice17/homeassistant-addons
```

Both add-ons below will appear under **HAOS ADD-ON's** in the store.

## Add-ons

| Add-on | Slug | What it does |
|--------|------|---------------|
| [RemoteTerm for MeshCore (USB Fix)](remoteterm-usb/) | `remoteterm_usb` | Packaging-only fork of [bhaggs/ha-remoteterm-app](https://github.com/bhaggs/ha-remoteterm-app) adding `udev: true` so `/dev/serial/by-id/` paths resolve (upstream now ships `uart: true` itself). |
| [Bee Backend](bee_backend/) | `bee_backend` | Local backend API bridging Bee wearable data into Home Assistant for the [Bee HA integration](https://github.com/FinestDice17/ha-bee-integration). |

Each add-on's folder has its own `README.md` (background/rationale) and `DOCS.md` (the
in-store usage/options doc shown by Home Assistant).

## Adding future add-ons

New add-ons go in as a new top-level folder with its own `config.yaml` — Supervisor
discovers every subdirectory in this repo automatically. Keep slugs unique across folders.
