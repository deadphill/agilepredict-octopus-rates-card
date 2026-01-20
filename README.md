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
  https://github.com/clee/home-assistant-octopus-energy-rates-card

### Data source
This uses a public forecast endpoint:
- https://prices.fly.dev/api/A

---

## Install (recommended): Package file + Lovelace card

### 1) Enable packages (configuration.yaml)

If you **already** use packages, skip this step.

Add this to your `configuration.yaml`:
```yaml
homeassistant:
  packages: !include_dir_merge_named packages/
```

Restart Home Assistant.

### 2) Copy the package file into your HA config folder

Copy this file from the repo:
