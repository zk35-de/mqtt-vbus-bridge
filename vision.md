# vbus2mqtt – Vision

## Why this exists

RESOL solar controllers broadcast their data over a proprietary serial bus (VBus).
RESOL sells their own gateways and cloud services to read this data — but the VBus
protocol specification is publicly available and the data is yours.

vbus2mqtt reads the VBus stream directly from a cheap USB serial adapter,
decodes it using the open RESOL specification, and publishes everything to a
local MQTT broker — no RESOL gateway, no cloud account, no subscription.

## Goal

Full solar controller data in Home Assistant (or any MQTT consumer) with:

- No cloud dependency
- No vendor hardware required beyond the controller itself
- A standard USB serial adapter (~5 €) instead of a 150 € RESOL gateway
- Full local control, data stays on your network

## Design decisions

### Runtime configuration via web UI

All settings (MQTT broker, serial port, publish interval, log level) are editable
at runtime through the embedded web UI. Changes take effect immediately without
a container restart.

**Why:** In a home automation context, broker addresses and credentials change.
A restart-free config update prevents gaps in telemetry data.

### File-based config with atomic writes

Settings saved via the web UI are written atomically (write-then-rename) to a
JSON file on a mounted volume.

**Why:** Atomic writes prevent a half-written config from corrupting the startup
sequence after a power loss.

### Device registry generated from the official VSF

The 360+ device definitions are generated from the official RESOL VBus
Specification (`.vsf`) file by Daniel Wippermann (MIT). Custom overrides in
`registry_custom.go` take precedence for devices where the spec disagrees with
the actual hardware payload.

## Scope

**In scope:**
- All VBus broadcast packets (dst=0x0010, cmd=0x0100)
- Home Assistant MQTT Autodiscovery
- Multi-arch container (amd64 · arm64 · arm/v7)
- Rootless Podman and Docker

**Out of scope:**
- VBus write commands (controller configuration)
- Non-serial VBus interfaces (LAN, KM2 gateway passthrough)
- Devices that don't broadcast standard VBus packets
