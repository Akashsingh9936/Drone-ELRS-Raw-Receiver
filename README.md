# ELRS Raw Receiver

Read raw RC channel values from an **ExpressLRS (ELRS)** receiver on an **ESP32** over the CRSF protocol, and print them to the serial monitor. This is the "hello world" step for building anything that reacts to RC input — rovers, drones, custom gimbals, etc.

## What this code does

- Initialises CRSF on ESP32 UART2 at 420000 baud (handled by the library).
- Listens for packets from an ExpressLRS receiver (tested with **RadioMaster RP1**).
- Prints all **16 channel values** to Serial Monitor every 100 ms.
- Warns on the serial monitor if the RC link is down (TX off, not bound, bad wiring).

Nothing is actuated — no motors, no servos. This is purely a receiver-side reader you can build on.

## Hardware

| Part | Example |
|---|---|
| MCU | ESP32 DevKit V1 (any ESP32 with UART2 works) |
| RC receiver | RadioMaster RP1 (ExpressLRS, 2.4 GHz) |
| Transmitter | Any ELRS-compatible radio (e.g. RadioMaster Pocket / TX16S) |

### Wiring

```
   RadioMaster RP1              ESP32 DevKit V1
   ───────────────              ───────────────
   TX  ───────────────────▶     GPIO16 (RX2)
   RX  ◀───────────────────     GPIO17 (TX2)   [optional, telemetry]
   5V  ───────────────────      5V / VIN
   GND ───────────────────      GND
```

> The RX↔TX2 wire is only needed if you later want to send telemetry **back** to the radio. For reading channels only, you can skip it.

## Channel values

CRSF raw range:

| | Value |
|---|---|
| Min (stick fully one way) | **172** |
| Center (stick neutral) | **992** |
| Max (stick fully other way) | **1811** |

To convert to microseconds (standard servo PWM 1000–2000 µs):

```cpp
int us = map(val, 172, 1811, 1000, 2000);
```

## Build & flash (PlatformIO)

1. Clone this repo and open the `ELRS-Raw-Receiver/` folder in VS Code with the PlatformIO extension.
2. Connect the ESP32 by USB.
3. Hit **Upload** (or `pio run -t upload`).
4. Open **Serial Monitor** at **115200 baud**.

### Dependencies

Declared in `platformio.ini`:

```ini
lib_deps = zz-cat/CRSFforArduino
```

PlatformIO fetches it automatically on first build.

## Expected serial output

```
=================================
ROVER Step 1 — CRSF Channel Read
=================================
[OK] CRSF initialised — waiting for RC link...
--- Channels ---
  CH01 : 1500  CH02 :  992  CH03 :  172  CH04 :  992
  CH05 :  992  CH06 :  992  CH07 :  992  CH08 :  992
  CH09 :  992  CH10 :  992  CH11 :  992  CH12 :  992
  CH13 :  992  CH14 :  992  CH15 :  992  CH16 :  992
```

If you see `[WARN] No RC link`, check:

- Transmitter is on and bound to the RP1.
- RP1 has a solid LED (not blinking).
- TX/RX pins aren't swapped.
- 5V and GND are solid.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| `CRSF init failed` on boot | GPIO16/17 unavailable (reused by another peripheral), bad wiring |
| `No RC link` forever | TX not bound to RX, or RX TX pin not connected to ESP32 GPIO16 |
| Garbage channel values | GND not shared between ESP32 and RP1 |
| Values stuck at 992 | TX is off or in failsafe |

## Next steps

This is step 1 of a bigger build. Possible next modules in this repo:

- Mix channels → PWM to brushed/brushless ESCs (rover drive).
- Mix channels → servo PWM (gimbal, steering).
- Send telemetry back via the TX2 line (battery voltage, RSSI).
