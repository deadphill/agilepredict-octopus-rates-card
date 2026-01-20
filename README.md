STEP 3 — README.md content (copy/paste everything)

# AgilePredict Forecast Rates Card (Octopus-style) for Home Assistant

Show AgilePredict predicted half-hour electricity rates (up to 7 days) in Home Assistant using the **Octopus Energy Rates Card** look + colour thresholds, with a **day selector** (left/right).

---

## Prerequisites

### 1) HACS (recommended)
Install HACS:
- https://hacs.xyz/

### 2) Install these Frontend (Lovelace) cards via HACS → Frontend

**Required**
- Mushroom Cards (for the day selector UI)  
  https://github.com/piitaya/lovelace-mushroom

- Octopus Energy Rates Card (for the coloured “pill” rate display)  
  https://github.com/clee/home-assistant-octopus-energy-rates-card

### 3) API used
This guide uses the public forecast endpoint:
- https://prices.fly.dev/api/A

---

## What this creates

### Helper
- `input_number.agile_predict_day_offset` (0–7)

### Sensors
- `sensor.agile_predict` (REST sensor, attributes include `prices`)
- `sensor.agilepredict_selected_day` (48 half-hours for selected day, attributes include `prices`)
- `sensor.agilepredict_rates_selected_day` (Octopus-card compatible entity, attributes include `rate` and `rates`)

---

## Install (recommended): Home Assistant Package

### Step A — Enable packages in configuration.yaml

If you don’t already have it, add:

```yaml
homeassistant:
  packages: !include_dir_merge_named packages/
```
Restart Home Assistant.
