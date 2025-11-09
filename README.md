# KellAI – HeySalad Camera PlatformIO Port

This PlatformIO project contains a first-pass C++ port of the original CircuitPython HeySalad camera server for the Seeed XIAO ESP32S3 Sense.

## Features included

- GC9A01 round display bring-up via `Arduino_GFX`
- ESP32-S3 camera configuration for 240×240 JPEG streaming
- Wi-Fi station + fallback access-point logic with HTTP + WebSocket endpoints
- Lightweight BLE UART bridge (NimBLE) for remote control commands
- Web dashboard (`data/index.html`) for viewing the live stream

## Display assets

GC9A01 splash screens are stored in SPIFFS under `data/assets` as 16-bit RGB565
streams (`*.rgb565`). If you update the source BMP artwork, regenerate the raw
files with:

```bash
python3 scripts/convert_images_to_rgb565.py assets_src data/assets
```

The converter validates that assets are 240×240 pixels and writes little-endian
RGB565 data that can be streamed directly to the display.

Original 32-bit BMP artwork lives in `assets_src/` so it doesn't bloat the
SPIFFS image.

## Security & configuration

- Secrets are intentionally scrubbed. Edit `include/Config.h` (Wi-Fi
  fallbacks, AP passphrase, BLE pairing secret, Laura cloud endpoints) or
  provision them at runtime via the `/api/settings` REST handlers.
- The boot-time admin password uses `Config::DEFAULT_AUTH_PASSWORD`. Change it
  before distributing firmware—the value is hashed into NVS on first boot.
- Cloud upload logic (`LauraClient`) stays dormant until
  `Config::LAURA_API.enabled` is set to `true` and the required URLs / keys are
  populated.
- Avoid storing production credentials in-source. Instead, inject them via the
  provisioning API or keep a private header that overrides the defaults.

## To build / upload

1. Install PlatformIO (PIO Core or VS Code extension).
2. Plug in the XIAO ESP32S3 Sense.
3. From this folder run `pio run -t upload`.
4. Use `pio device monitor -b 115200` to watch logs.

> **Heads-up**: Camera and display pin assignments use Seeed defaults. Adjust the constants in `include/Config.h` if you have modified wiring or use a different ESP32-S3 module.

## Follow-up TODOs

- Port the Laura cloud client and peer device APIs if they are still required.
- Add persistent credentials / secrets handling instead of hard-coding values in `Config.h`.
- Harden the WebSocket binary streaming path (rate limiting, error handling, watch-dogging).
