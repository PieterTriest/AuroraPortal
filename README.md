# AuroraPortal

AuroraPortal is an ESP32 + FastLED project for driving LED matrix installations with rich visual effects, BLE control, and a browser UI.

The firmware runs on ESP32 boards (currently configured for **Seeed XIAO ESP32S3**) and exposes a custom BLE service that the web interface (`index.html`) uses for real-time control.

## Highlights

- Multiple visualizer programs (rainbow, waves, bubble, dots, radii, animartrix, synaptide, cube, horizons, audioTest, colorTrails)
- Program/mode architecture with dynamic parameter mapping
- Browser-based control panel built with vanilla Web Components and Web Bluetooth
- Preset save/load backed by LittleFS
- Audio capture + processing pipeline for audio-reactive modes
- Matrix mapping support for multiple panel sizes and wiring patterns

## Hardware + matrix configurations

`src/main.cpp` supports two board layouts using `#define BIG_BOARD`:

- **Small board (default):** `22 x 22` matrix, single data pin
- **Big board:** `32 x 48` matrix, 3 data pins (parallel strips)

Default pin assignment in current code:

- `PIN0 = GPIO2`
- `PIN1 = GPIO3` (big board)
- `PIN2 = GPIO4` (big board)

If you use different wiring, update pin and mapping includes in `src/main.cpp` and mapping tables under `src/reference/`.

## Software architecture

### Firmware (ESP32)

Core flow:

1. Boot + LED setup (`FastLED.addLeds(...)`)
2. BLE setup (`bleSetup()`)
3. LittleFS mount
4. Audio input + processing init
5. Main loop:
   - Capture audio sample buffers
   - Route by selected `PROGRAM`/`MODE`
   - Render selected visualizer
   - `FastLED.show()`
   - Manage BLE reconnect behavior

Key firmware files:

- `src/main.cpp` — boot, program dispatch loop, board/matrix selection
- `src/bleControl.h` — BLE service/characteristics, parameter sync, preset storage
- `src/audio/` — audio capture, types, processing helpers
- `src/programs/` — each visualizer module (`*.hpp` + `*_detail.hpp`)
- `src/reference/` — matrix map tables + palettes

### Web UI

- `index.html` is a self-contained Web Bluetooth app (no build toolchain required).
- Connects to BLE device name **"Aurora Portal"**.
- Uses 4 characteristics:
  - button
  - checkbox
  - number
  - string
- UI includes:
  - connection panel
  - program/mode selectors
  - dynamic parameter sliders
  - audio and per-bus controls
  - preset save/load controls

For protocol details and architecture notes, see `BLE_UI_SYSTEM.md`.

## Repo layout

- `platformio.ini` — PlatformIO environment and build config
- `openocd_debug.cfg` — OpenOCD config for debugging
- `index.html` — BLE control UI
- `BLE_UI_SYSTEM.md` — detailed BLE/UI design documentation
- `src/` — firmware source
  - `main.cpp`
  - `bleControl.h`
  - `audio/`
  - `programs/`
  - `reference/`

## Build and flash

### Prerequisites

- [PlatformIO Core](https://docs.platformio.org/en/latest/core/installation/index.html)
- USB connection to your ESP32 board

### Build

```bash
platformio run
```

### Upload firmware

```bash
platformio run -t upload
```

### Serial monitor

```bash
platformio device monitor -b 115200
```

## Using the web UI

Because this project uses **Web Bluetooth**, open `index.html` from a secure context (typically `http://localhost` during development).

Simple local serving examples:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

Then:

1. Power your ESP32 running AuroraPortal firmware.
2. Open the UI in a Chromium-based browser with Web Bluetooth support.
3. Click connect and choose device **Aurora Portal**.
4. Select programs/modes, adjust sliders, and save/load presets.

## Programs and modes (current)

Programs are enumerated in `src/bleControl.h`.

- `rainbow`
- `waves` (modes: `palette`, `pride`)
- `bubble`
- `dots`
- `fxwave2d` (currently disabled in `main.cpp` switch)
- `radii` (modes: `octopus`, `flower`, `lotus`, `radial`, `lollipop`)
- `animartrix` (10 modes)
- `test`
- `synaptide`
- `cube`
- `horizons`
- `audiotest` (10 modes)
- `colortrails` (modes: `orbital`, `lissajous`)

## Presets and persistence

- Presets are stored in LittleFS as JSON files (`/preset_1.json`, etc.).
- Device settings like brightness/program/mode are persisted via `Preferences`.

## Notes

- The project currently uses a local FastLED copy (`lib/FastLED`) as referenced in `platformio.ini`.
- `build_type = debug` and exception decoder monitor filter are enabled by default.
- There are many tunable visual parameters surfaced over BLE; `BLE_UI_SYSTEM.md` documents naming and sync conventions.

## Credits

This project builds on FastLED examples and community-contributed visual ideas. See top-of-file credits in `src/main.cpp` for detailed attributions.
