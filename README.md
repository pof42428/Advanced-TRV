# 🌡️ Advanced TRV Room Controller for Home Assistant

[![Home Assistant Required Version](https://img.shields.io/badge/Home%20Assistant-2024.10.0%2B-blue.svg)](https://homeassistant.io)

A production-grade, highly sophisticated automation blueprint for a single room equipped with a Thermostatic Radiator Valve (TRV) that supports an **external temperature input** and a **temperature accuracy** (start threshold) setting—such as the **Sonoff TRVZB** (firmware ≥ 1.3) managed via Zigbee2MQTT.

---

## ✨ Features

- **Native TRV Regulation:** The TRV regulates temperature natively by receiving the target setpoint, the external room temperature sensor reading, and its start threshold (accuracy, sent as-is, e.g., `−0.4` °C). It opens when `room ≤ setpoint + accuracy` and closes when the room reaches the target setpoint.
- **Seamless Auto & Manual Sync:** Schedule-driven *auto mode* and *manual mode* driven by a UI slider or the physical knob, kept fully in sync both ways. Turning the knob or moving the slider while in auto mode automatically switches the room to manual mode.
- **Ignore Physical Rotation:** Child-lock synchronization and knob-change reversion capabilities to prevent unwanted manual physical adjustments.
- **Global & Room Overrides:**
  - Global *master switch* (summer mode handling).
  - *Maintenance switch* to safely halt the automation.
  - *Laundry drying mode* overrides to force valves fully open.
- **Window/Door Integration:** Pauses heating automatically after a configurable delay when windows or doors are left open.
- **Hardware Keep-Alive:** Automatically re-sends the external temperature periodically to prevent the TRV from timing out its remote sensor (~30 min hardware timeout prevention).
- **Boiler Boost Integration:** Optional automated boiler boost features based on room temperature differences.

---

## 🚦 Control Priority Order

The controller evaluates states based on the following priority hierarchy (highest to lowest):
0. **Maintenance switch ON** → Automation is completely halted.
1. **Window / Door open** → Heating paused, TRV dropped to the minimum window temperature.
2. **Laundry override active** → Valve forced 100% open.
3. **Heating master switch OFF (Summer Mode):**
   - If global laundry switch is ON: Non-laundry valves close (`5 °C`).
   - If global laundry switch is OFF: All valves forced 100% open (prevents valve sticking).
4. **Manual open switch ON** → Valve forced 100% open.
5. **Auto mode ON** → Follows the schedule setpoint (or boiler boost if active).
6. **Auto mode OFF** → Follows the manual target (UI slider or physical knob).

---

## 📦 Requirements & Prerequisites

- **Home Assistant:** Version `2024.10.0` or newer.
- **Integration:** Zigbee2MQTT (recommended for Sonoff TRVZB or equivalent TRVs supporting external temperature and start accuracy attributes/number entities).
- **Optional Helpers:** Input numbers, input booleans, and binary sensors can be configured depending on which features you want to utilize. All optional inputs can be left empty to skip functionality.

---

## 🚀 Installation

You can import this blueprint directly into your Home Assistant instance using the button below (once hosted on GitHub):

[![Open your Home Assistant instance and show the blueprint import dialog](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=YOUR_GITHUB_RAW_URL_HERE/trv.yaml)

### Manual Installation
1. Copy the raw contents of `trv.yaml`.
2. Save the file inside your Home Assistant configuration directory under:
   `config/blueprints/automation/helryser/trv.yaml`
3. Go to **Settings > Automations & Scenes > Blueprints** and click **Reload Blueprints**.

---

## ⚙️ Inputs Configuration Overview

### 1. Room Entities
- **Climate entity (TRV):** The physical radiator climate device.
- **Room temperature sensor:** Reference temperature sensor fed to the TRV.
- *Optional:* External temperature number entity, temperature accuracy number entity, child lock switch, and window/door binary sensors.

### 2. Room Controls (Helpers)
- *Optional:* Target temperature UI slider (`input_number`), start offset entity, auto mode switch, manual open switch, ignore physical rotation switch, laundry override boolean, and boiler boost configuration entities.

### 3. Global System Entities
- *Optional:* House-wide heating master switch, maintenance switch, and laundry drying mode switch.

### 4. Tuning & Offsets
- Adjust parameters like the valve "fully open" temperature, artificial cold external temperature for forced opening, window temperature fallback, window open delay, trigger debounce duration, and the hardware keep-alive interval.

---

## 📄 License

This project is open-source and available under the terms of the [MIT License](LICENSE).
