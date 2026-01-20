# AgilePredict Forecast Rates Card (Octopus-style) for Home Assistant

Show AgilePredict predicted half-hour electricity rates (up to 7 days) in Home Assistant using the **Octopus Energy Rates Card** look + colour thresholds, with a **day selector** (left/right).

---

## Prerequisites

### 1. Home Assistant

- YAML access to your HA config (File editor / VS Code add-on / Samba etc.)
- Restart capability

### 2. HACS (recommended)

Install HACS:
- https://hacs.xyz/

### 3. Install these cards (HACS → Frontend)

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

### 2) Download and configure the package file

The package file contains everything needed:
- **AgilePredict REST sensor** - fetches forecast data from the API (default: Region A)
- **Template sensors** - process the data for display
- **Input number helper** - for day selection

**Download the file:**

1. Click here: [`packages/agilepredict_octopus_card.yaml`](packages/agilepredict_octopus_card.yaml)
2. Click the "Raw" button (or right-click and "Save Link As")
3. Save to your computer

**IMPORTANT - Change to your region:**

Before copying to Home Assistant, you MUST edit the file to use your region:

1. Open agilepredict_octopus_card.yaml in a text editor (Notepad, VS Code, etc.)
2. Find this line (near the top of the file):

    resource: https://prices.fly.dev/api/A

3. Change /api/A to your region code:
   - /api/A - National Average (default)
   - /api/B - East Midlands
   - /api/G - North West England
   - /api/C - West Midlands
   - See full list: https://prices.fly.dev/api_how_to

4. Save the file

**Copy to Home Assistant:**

Copy the edited file to your Home Assistant config folder as:

    config/packages/agilepredict_octopus_card.yaml

Then restart Home Assistant.

**Verify it's working:**

After restart, go to Developer Tools → States and check that these entities exist:
- sensor.agile_predict (should show your region name)
- sensor.agilepredict_selected_day
- sensor.agilepredict_rates_selected_day
- input_number.agile_predict_day_offset

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
