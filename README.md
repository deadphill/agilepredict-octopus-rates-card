# AgilePredict Forecast Rates Card (Octopus-style) for Home Assistant

Show AgilePredict predicted half-hour electricity rates (up to 7 days) in Home Assistant using the **Octopus Energy Rates Card** look + colour thresholds, with a **day selector** (left/right).

---

## Prerequisites

### Home Assistant

- YAML access to your HA config (File editor / VS Code add-on / Samba etc.)
- Restart capability

### HACS (recommended)

Install HACS:
- https://hacs.xyz/

### Install these cards (HACS → Frontend)

These are required for the UI:

- **Mushroom Cards** (day selector UI)  
  https://github.com/piitaya/lovelace-mushroom

- **Octopus Energy Rates Card** (Octopus-style coloured rate display)  
  [https://github.com/clee/home-assistant-octopus-energy-rates-card]

### Data source

This uses a public forecast endpoint:
- https://prices.fly.dev/api/A

---

## Installation

### 1) Enable packages in configuration.yaml

If you **already** use packages, skip this step.

Add this to your `configuration.yaml`:

​```yaml
homeassistant:
  packages: !include_dir_merge_named packages/
​```

Then restart Home Assistant.

### 2) Copy the package file into your HA config folder

Download `packages/agilepredict_octopus_card.yaml` from this repo and copy it to:

​```
config/packages/agilepredict_octopus_card.yaml
​```

Then restart Home Assistant.

### 3) Add the Lovelace card

In your dashboard:

1. Edit dashboard → Add card → Manual (YAML)
2. Paste the contents of: `lovelace/agilepredict_card.yaml`

---

## Entities created

### Helper

- `input_number.agile_predict_day_offset` (0–7)

### Sensors

- `sensor.agile_predict` (REST sensor, attribute `prices`)
- `sensor.agilepredict_selected_day` (selected day, attribute `prices` = 48 half-hours)
- `sensor.agilepredict_rates_selected_day` (Octopus-card compatible, attributes include `rate` and `rates`)

---

## Troubleshooting

### The card doesn't feel "instant" when changing day

Set `cardRefreshIntervalSeconds: 1` in the Lovelace card YAML.

### Sensor goes unavailable sometimes

The REST sensor includes:
- `timeout: 30`
- `scan_interval: 900`

to reduce flapping.

---

## Disclaimer

Not affiliated with Octopus Energy or AgilePredict.
