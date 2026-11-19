# Hardware plan: sensor buoy and night siren

**Status: planned. Nothing in this folder is built yet.** The parts below will be
assembled and programmed during the Resurrect 2026 hackathon finale (VCET Puttur,
27–28 November 2026). The web app in the rest of this repo was built before that.

AquaNexus works today with readings typed in by the farmer or sent by any device to the
sensor API. This folder is the low-cost device that will send real readings, plus an
alarm that wakes the farmer at night without needing the internet.

## 1. Sensor buoy (at the pond)

A floating, waterproof box that measures the water every 5 minutes and sends the
readings to the live app.

| Part | Job | Rough price (Rs) |
|---|---|---|
| ESP32 DevKit | Reads the sensors, sends readings over Wi-Fi | 450 |
| pH sensor + probe (PH-4502C) | Acidity of the water | 1,300 |
| DS18B20 waterproof probe | Water temperature | 150 |
| Analog TDS sensor | Dissolved salts (a proxy for water quality) | 700 |
| 6 V solar panel + 18650 cell + charger | Power, day and night | 800 |
| IP65 box, float, cable glands | Keeps it dry and floating | 600 |
| **Total** | | **about 4,000** |

Prices are rough estimates from Indian online stores (Oct 2026). Check before ordering.

**No oxygen probe in the low-cost kit.** A dissolved-oxygen probe alone costs Rs 18,000+.
The app already estimates tonight's oxygen-crash risk from water temperature, weather and
the trend, so the buoy stays affordable. An oxygen probe can be added later for bigger farms.

### Planned pin map

| Sensor | ESP32 pin | Notes |
|---|---|---|
| pH module (analog out) | GPIO 34 | Input-only ADC pin; module powered at 5 V, output divided down to 3.3 V |
| TDS module (analog out) | GPIO 35 | Input-only ADC pin |
| DS18B20 (data) | GPIO 4 | 4.7 kΩ pull-up to 3.3 V |
| Status LED | GPIO 2 | Green / amber / red blink codes |

### How it talks to the app

The buoy uses the existing sensor API, no backend changes needed:

```
POST /api/sensor/{device_id}
X-Sensor-Key: <the key shown once when the pond's device is added>
{"ph": 7.4, "temperature_c": 28.6, "tds_ppm": 310}
```

See [../docs/sensor-api.md](../docs/sensor-api.md) for the full format, keys and refusals.
The key goes in a `secrets.h` file on the device that is **never committed**.

### Calibration

- pH: two-point calibration with buffer 4.0 and 7.0 solutions before every demo.
- TDS: one-point check against a known solution.
- Readings the app finds impossible (for example pH 0) are shown as "sensor fault",
  not as a danger alert.

## 2. Night siren (in the farmer's house)

Phone alerts are easy to sleep through, and many ponds have weak internet. The siren
fixes both.

| Part | Job | Rough price (Rs) |
|---|---|---|
| 2 × SX1278 LoRa modules (433 MHz) | Radio link from pond to house, works without internet | 700 |
| ESP32 or Arduino Nano (house unit) | Receives the alarm | 300 |
| 12 V buzzer siren + red LED + "silence" button | Wakes the farmer | 250 |
| 5 V adapter + backup cell | Keeps it on during power cuts | 300 |

**How it works:** when a reading crosses into Danger, the buoy sends a short LoRa message
to the house unit, which sounds the siren until the farmer presses "silence". The app
alert and the siren work independently, so either one still warns the farmer if the
other fails.

## 3. Build order at the finale

| Hours | Task |
|---|---|
| 0–4 | Wire the sensors, calibrate pH |
| 4–10 | ESP32 firmware: read sensors, post to the live sensor API |
| 10–16 | LoRa pond node to house node, siren logic |
| 16–21 | Full test in a water tank (a little vinegar lowers the pH to trigger an alert) |
| 21–24 | Fix, tidy, rehearse the demo |

Firmware will go in `hardware/buoy/` and `hardware/siren/`.
