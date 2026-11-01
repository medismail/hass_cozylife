# CozyLife Local — HA 2026.1 recovery branch

This branch is a **staged, unverified device-recovery upgrade**, based on the upstream
CozyLife Local 0.2.1 implementation. It is intended for Home Assistant Core 2026.1
but has **not** been tested on the owner's hardware.

## Goals

- Keep the original integration domain `hass_cozylife_local_pull`, configured
  device IPs, entity unique IDs, and normal switch/light control values.
- When a device becomes unresponsive, a state poll fails, and Home Assistant
  reports its light/switch entity as **unavailable**, rather than pretending
  it is online.
- Retry a known device TCP connection every **15 seconds** after failure
  (upstream defaults to 60 seconds). After a valid handshake, entity
  polling/notifications can restore availability and device state.
- Discover devices every 60 seconds, including devices with changed IPs.
- Allow **Reload** for the CozyLife Local integration in
  **Settings → Devices & services → CozyLife Local → ⋮ → Reload** without
  restarting Home Assistant (uses upstream config-entry setup/unload).
- Retain 0.2.0's static work-mode reset on ordinary light turn-on, only
  on devices that report support for that work-mode DPID.

## Installation / migration (manual)

**Do not overwrite the working installation until you have a backup.**

1. Back up `/config/custom_components/hass_cozylife_local_pull/` and
   `/config/configuration.yaml`. Note current entity IDs / automations.
2. Check Home Assistant Core is **2026.1**. Keep the YAML configuration below.
3. Copy this branch's
   `custom_components/hass_cozylife_local_pull/` directory over the
   currently installed same-named directory; remove outdated .py files
   that are no longer present.
4. Restart **Home Assistant once** to migrate from the legacy YAML
   platform to the config-entry-based integration. Your settings are
   automatically imported from YAML into its config entry.
5. After that, use integration **Reload** to recover manually; try the
   automatic reconnect first by powering a test device off, waiting
   for it to become unavailable, then turning it on again.
6. On failure, restore the backup directory and restart Home Assistant.
   The original implementation remains on the repository's `main` branch.

```yaml
hass_cozylife_local_pull:
  lang: en
  ip:
    - "192.168.1.99"  # optional static IP addresses
```

**Important:** Keep this YAML section until you intentionally migrate away.
The current upstream config flow is YAML-import-only (not an interactive
Add Integration form). Additional upstream 0.2.1 motor/countdown entities
may appear; these do not replace the existing light/switch entity unique IDs.

## Troubleshooting / verification

- For a known IP, expect offline status after the next failed entity poll
  (normally around 30 s), not instant notification.
- A TCP retry runs about every 15 s after a failed connection. Discovery
  of a new address runs every 60 s. The exact time to recover varies.
- The integration reload also re-establishes its clients; it is not a
  substitute for power-cycling a physically frozen device.
- Check **Settings → System → Logs** and filter for
  `custom_components.hass_cozylife_local_pull`.
- For detailed logs, temporarily configure:

```yaml
logger:
  logs:
    custom_components.hass_cozylife_local_pull: debug
```

- Some CozyLife models or network isolation/firewall rules may prevent
  UDP broadcast discovery. Configure static IPs to improve reliability.
- No code can recover a device that remains powered on but does not
  respond to new network connections; the entity should stay unavailable.
- The upstream tests included in this branch are *not* proof of successful
  testing against your Home Assistant instance.

Upstream: https://github.com/cozylife/hass_cozylife_local_pull
