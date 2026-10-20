# 🌡️ Advanced TRV Room Controller for Home Assistant

[![Home Assistant Required Version](https://img.shields.io/badge/Home%20Assistant-2024.10.0%2B-blue.svg)](https://homeassistant.io)

A production-grade, highly sophisticated automation blueprint for a single room equipped with a Thermostatic Radiator Valve (TRV) that supports an **external temperature input** and a **temperature accuracy** (start threshold) setting—such as the **Sonoff TRVZB** (firmware ≥ 1.3) managed via Zigbee2MQTT[cite: 1].

---

## ✨ Features

- **Native TRV Regulation:** The TRV regulates temperature natively by receiving the target setpoint, the external room temperature sensor reading, and its start threshold (accuracy, sent as-is, e.g., `−0.4` °C)[cite: 1]. It opens when `room ≤ setpoint + accuracy` and closes when the room reaches the target setpoint[cite: 1].
- **Seamless Auto & Manual Sync:** Schedule-driven *auto mode* and *manual mode* driven by a UI slider or the physical knob, kept fully in sync both ways[cite: 1]. Turning the knob or moving the slider while in auto mode automatically switches the room to manual mode[cite: 1].
- **Ignore Physical Rotation:** Child-lock synchronization and knob-change reversion capabilities to prevent unwanted manual physical adjustments[cite: 1].
- **Global & Room Overrides:**
  - Global *master switch* (summer mode handling)[cite: 1].
  - *Maintenance switch* to safely halt the automation[cite: 1].
  - *Laundry drying mode* overrides to force valves fully open[cite: 1].
- **Window/Door Integration:** Pauses heating automatically after a configurable delay when windows or doors are left open[cite: 1].
- **Hardware Keep-Alive:** Automatically re-sends the external temperature periodically to prevent the TRV from timing out its remote sensor (~30 min hardware timeout prevention)[cite: 1].
- **Boiler Boost Integration:** Optional automated boiler boost features based on room temperature differences[cite: 1].

---

## 🚦 Control Priority Order

The controller evaluates states based on the following priority hierarchy (highest to lowest):
0. **Maintenance switch ON** → Automation is completely halted[cite: 1].
1. **Window / Door open** → Heating paused, TRV dropped to the minimum window temperature[cite: 1].
2. **Laundry override active** → Valve forced 100% open[cite: 1].
3. **Heating master switch OFF (Summer Mode):**
   - If global laundry switch is ON: Non-laundry valves close (`5 °C`)[cite: 1].
   - If global laundry switch is OFF: All valves forced 100% open (prevents valve sticking)[cite: 1].
4. **Manual open switch ON** → Valve forced 100% open[cite: 1].
5. **Auto mode ON** → Follows the schedule setpoint (or boiler boost if active)[cite: 1].
6. **Auto mode OFF** → Follows the manual target (UI slider or physical knob)[cite: 1].

---

## 📦 Requirements & Prerequisites

- **Home Assistant:** Version `2024.10.0` or newer[cite: 1].
- **Integration:** Zigbee2MQTT (recommended for Sonoff TRVZB or equivalent TRVs supporting external temperature and start accuracy attributes/number entities)[cite: 1].
- **Optional Helpers:** Input numbers, input booleans, and binary sensors can be configured depending on which features you want to utilize. All optional inputs can be left empty to skip functionality[cite: 1].

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
- **Climate entity (TRV):** The physical radiator climate device[cite: 1].
- **Room temperature sensor:** Reference temperature sensor fed to the TRV[cite: 1].
- *Optional:* External temperature number entity, temperature accuracy number entity, child lock switch, and window/door binary sensors[cite: 1].

### 2. Room Controls (Helpers)
- *Optional:* Target temperature UI slider (`input_number`), start offset entity, auto mode switch, manual open switch, ignore physical rotation switch, laundry override boolean, and boiler boost configuration entities[cite: 1].

### 3. Global System Entities
- *Optional:* House-wide heating master switch, maintenance switch, and laundry drying mode switch[cite: 1].

### 4. Tuning & Offsets
- Adjust parameters like the valve "fully open" temperature, artificial cold external temperature for forced opening, window temperature fallback, window open delay, trigger debounce duration, and the hardware keep-alive interval[cite: 1].

---

## 📄 License

This project is open-source and available under the terms of the [MIT License](LICENSE).
