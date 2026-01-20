# AgilePredict Forecast Rates Card (Octopus-style) for Home Assistant

Show AgilePredict predicted half-hour electricity rates (up to 7 days) in Home Assistant using the **Octopus Energy Rates Card** look + colour thresholds, with a **day selector** (left/right).

---

## Prerequisites

### 1. Home Assistant

- YAML access to your HA config (File editor / VS Code add-on / Samba etc.)
- Restart capability

### 2. AgilePredict Sensor (REQUIRED)

**You must set up the AgilePredict sensor first.** This card requires the sensor to be working.

Follow the official setup instructions here:
- **https://prices.fly.dev/api_how_to**

Quick setup: Add this to your `configuration.yaml`:
```yaml
sensor:
  - platform: rest
    resource: https://prices.fly.dev/api/G
    scan_interval: 3600
    name: Agile Predict
    value_template: "{{ value_json[0]['name']}}"
    json_attributes_path: "$[0]"
    json_attributes:
      - "created_at"
      - "prices"
```

**Important:** Change `/api/G` to your region code. See the API documentation link above for all available regions.

After adding this, restart Home Assistant and verify the sensor exists in Developer Tools → States (`sensor.agile_predict`).

### 3. HACS (recommended)

Install HACS:
- https://hacs.xyz/

### 4. Install these cards (HACS → Frontend)

These are required for the UI:

- **Mushroom Cards** (day selector UI)  
  https://github.com/piitaya/lovelace-mushroom

- **Octopus Energy Rates Card** (Octopus-style coloured rate display)  
  https://github.com/clee/home-assistant-octopus-energy-rates-card

---

## Installation

### 1) Enable packages in configuration.yaml

If you **already** use packages, skip this step.

Edit your configuration.yaml file and add the following:

    homeassistant:
      packages: !include_dir_merge_named packages/

Save the file and restart Home Assistant.

### 2) Copy the package file into your HA config folder

Download the package file:

1. Click here: [`packages/agilepredict_octopus_card.yaml`](packages/agilepredict_octopus_card.yaml)
2. Click the "Raw" button (or right-click the link above and "Save Link As")
3. Save the file

Copy the downloaded file to your Home Assistant config folder as:

    config/packages/agilepredict_octopus_card.yaml

Then restart Home Assistant.

**Note:** After restarting, the sensors and helper will be automatically created by the package. You don't need to configure them manually.

### 3) Add the Lovelace card

1. Open the file [`lovelace/agilepredict_card.yaml`](lovelace/agilepredict_card.yaml) in this repository
2. Copy all the YAML content
3. In your Home Assistant dashboard:
   - Edit dashboard → Add card → Manual (YAML)
   - Paste the YAML you copied
   - Save

---

## Entities created

### Helper

- input_number.agile_predict_day_offset (0–7)

### Sensors

- sensor.agile_predict (REST sensor, attribute prices)
- sensor.agilepredict_selected_day (selected day, attribute prices = 48 half-hours)
- sensor.agilepredict_rates_selected_day (Octopus-card compatible, attributes include rate and rates)

---

## Troubleshooting

### The card doesn't feel "instant" when changing day

Set cardRefreshIntervalSeconds: 1 in the Lovelace card YAML.

### Sensor goes unavailable sometimes

The REST sensor includes timeout: 30 and scan_interval: 900 to reduce flapping.

---

## Disclaimer

Not affiliated with Octopus Energy or AgilePredict.
