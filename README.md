# GitHub Repository Setup Instructions

## STEP 1 — Create the GitHub Repository

1. Go to GitHub → New repository
2. Name: `agilepredict-octopus-rates-card`
3. Check "Public" ✅
4. Check "Add a README" ✅
5. Click "Create repository"

---

## STEP 2 — Replace README.md

1. Open `README.md` in your repo
2. Click the pencil icon (Edit)
3. **Delete everything** and paste the content below
4. Commit changes

### Content to paste into README.md:

```
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
```

---

## STEP 3 — Add Package File

1. In your repo, click "Add file" → "Create new file"
2. Filename: `packages/agilepredict_octopus_card.yaml`
3. Paste the YAML below
4. Commit the file

### Content to paste:

```yaml
# =========================================
# AgilePredict -> Octopus-style Rates Card
# Single-file Home Assistant package
# =========================================

input_number:
  agile_predict_day_offset:
    name: AgilePredict Day Offset
    min: 0
    max: 7
    step: 1
    mode: slider

sensor:
  - platform: rest
    name: Agile Predict
    resource: https://prices.fly.dev/api/A
    timeout: 30
    scan_interval: 900
    value_template: "{{ value_json[0]['name'] }}"
    json_attributes_path: "$[0]"
    json_attributes:
      - created_at
      - prices

template:
  - sensor:
      - name: "AgilePredict Selected Day"
        unique_id: agilepredict_selected_day
        state: >
          {% set off = states('input_number.agile_predict_day_offset') | int(0) %}
          {{ (now().date() + timedelta(days=off)).isoformat() }}
        attributes:
          prices: >
            {% set off = states('input_number.agile_predict_day_offset') | int(0) %}
            {% set target_date = (now().date() + timedelta(days=off)) %}

            {% set raw = state_attr('sensor.agile_predict', 'prices') or [] %}
            {% set map = namespace(d={}) %}
            {% for p in raw %}
              {% set dtl = as_local(as_datetime(p.date_time)) %}
              {% if dtl and dtl.date() == target_date %}
                {% set k = dtl.strftime('%Y-%m-%d %H:%M') %}
                {% set map.d = dict(map.d, **{ k: p }) %}
              {% endif %}
            {% endfor %}

            {% set start = as_local(as_datetime(target_date.isoformat() ~ 'T00:00:00')) %}
            {% set out = namespace(items=[]) %}
            {% for i in range(0, 48) %}
              {% set slot = start + timedelta(minutes=30*i) %}
              {% set key = slot.strftime('%Y-%m-%d %H:%M') %}
              {% set found = map.d.get(key) %}
              {% if found %}
                {% set out.items = out.items + [found] %}
              {% else %}
                {% set out.items = out.items + [{
                  'date_time': slot.isoformat(),
                  'agile_pred': none,
                  'agile_low': none,
                  'agile_high': none
                }] %}
              {% endif %}
            {% endfor %}
            {{ out.items }}

      - name: "AgilePredict Rates (Selected Day)"
        unique_id: agilepredict_rates_selected_day
        icon: mdi:chart-line
        state: >
          {% set off = states('input_number.agile_predict_day_offset') | int(0) %}
          {{ (now().date() + timedelta(days=off)).isoformat() }}
        attributes:
          # Provide both keys for card compatibility across versions
          rates: >
            {% set items = state_attr('sensor.agilepredict_selected_day','prices') or [] %}
            {% set ns = namespace(out=[]) %}
            {% for p in items %}
              {% set dt = as_local(as_datetime(p.date_time)) %}
              {% if dt %}
                {% set start = dt.isoformat() %}
                {% set end = (dt + timedelta(minutes=30)).isoformat() %}
                {% set value = p.agile_pred %}
                {% set ns.out = ns.out + [ {'start': start, 'end': end, 'value_inc_vat': value} ] %}
              {% endif %}
            {% endfor %}
            {{ ns.out }}
          rate: >
            {% set items = state_attr('sensor.agilepredict_selected_day','prices') or [] %}
            {% set ns = namespace(out=[]) %}
            {% for p in items %}
              {% set dt = as_local(as_datetime(p.date_time)) %}
              {% if dt %}
                {% set start = dt.isoformat() %}
                {% set end = (dt + timedelta(minutes=30)).isoformat() %}
                {% set value = p.agile_pred %}
                {% set ns.out = ns.out + [ {'start': start, 'end': end, 'value_inc_vat': value} ] %}
              {% endif %}
            {% endfor %}
            {{ ns.out }}
```

---

## STEP 4 — Add Lovelace Card File

1. In your repo, click "Add file" → "Create new file"
2. Filename: `lovelace/agilepredict_card.yaml`
3. Paste the YAML below
4. Commit the file

### Content to paste:

```yaml
type: vertical-stack
cards:
  - type: custom:mushroom-chips-card
    alignment: center
    chips:
      - type: action
        icon: mdi:chevron-left
        tap_action:
          action: call-service
          service: input_number.decrement
          target:
            entity_id: input_number.agile_predict_day_offset

      - type: entity
        entity: sensor.agilepredict_selected_day
        name: Day
        icon: mdi:calendar

      - type: action
        icon: mdi:restore
        tap_action:
          action: call-service
          service: input_number.set_value
          data:
            entity_id: input_number.agile_predict_day_offset
            value: 0

      - type: action
        icon: mdi:chevron-right
        tap_action:
          action: call-service
          service: input_number.increment
          target:
            entity_id: input_number.agile_predict_day_offset

  - type: custom:octopus-energy-rates-card
    title: AgilePredict Rates (selected day)
    currentEntity: sensor.agilepredict_rates_selected_day
    cols: 2
    hour12: false
    showday: true
    showpast: false
    unitstr: p/kwh
    lowlimit: 15
    mediumlimit: 20
    highlimit: 30
    roundUnits: 2
    cheapest: true
    multiplier: 1
    cardRefreshIntervalSeconds: 1
```

---

## STEP 5 — Verify Your Repository

Your repo should now contain exactly these three files:
- `README.md`
- `packages/agilepredict_octopus_card.yaml`
- `lovelace/agilepredict_card.yaml`

Done! Users can now follow the instructions in your README to install.
