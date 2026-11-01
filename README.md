# 🌡️ Advanced TRV Room Controller for Home Assistant

[![Home Assistant Required Version](https://img.shields.io/badge/Home%20Assistant-2024.10.0%2B-blue.svg)](https://homeassistant.io)
[![Author](https://img.shields.io/badge/Author-pof42428-orange.svg)]()

A production-grade, highly sophisticated automation blueprint for a single room equipped with a Thermostatic Radiator Valve (TRV) that supports an **external temperature input** and a **temperature accuracy** (start threshold) setting—such as the **Sonoff TRVZB** (firmware ≥ 1.3) managed via Zigbee2MQTT.

---

## ✨ Features

- **Native TRV Regulation:** The TRV regulates temperature natively by receiving the target setpoint, the external room temperature sensor reading, and its start threshold (accuracy, sent as-is, e.g., `−0.4` °C). It opens when `room <= setpoint + accuracy` and closes when the room reaches the target setpoint.
- **Seamless Auto & Manual Sync:** Schedule-driven *auto mode* and *manual mode* driven by a UI slider or the physical knob, kept fully in sync both ways. Turning the knob or moving the slider while in auto mode automatically switches the room to manual mode.
- **Ignore Physical Rotation:** Child-lock synchronization and knob-change reversion capabilities to prevent unwanted manual physical adjustments.
- **Global & Room Overrides:**
  - Global *master switch* (summer mode handling).
  - *Maintenance switch* to safely halt the automation.
  - *Laundry drying mode* override with customizable target temperature offset (`offset_laundry`).
- **Window/Door Integration:** Pauses heating automatically after a configurable delay when windows or doors are left open.
- **Hardware Keep-Alive:** Automatically re-sends the external temperature periodically to prevent the TRV from timing out its remote sensor (~30 min hardware timeout prevention).
- **Boiler Boost Integration:** Optional automated boiler boost features based on room temperature differences.

---

## 🚦 Control Priority Order

The controller evaluates states based on the following priority hierarchy (highest to lowest):

0. **Maintenance switch ON** → Automation is completely halted.
1. **Window / Door open** → Heating paused, TRV dropped to the minimum window temperature.
2. **Laundry override active** → TRV target set to schedule setpoint + `offset_laundry` (defaults to +1.0 °C) & real room temperature passed to external input.
3. **Heating master switch OFF (Summer Mode):**
   - If global laundry switch is ON: Non-laundry valves close (`5 °C`).
   - If global laundry switch is OFF: All valves forced 100% open (prevents valve sticking).
4. **Manual open switch ON** → Valve forced 100% open.
5. **Auto mode ON** → Follows the schedule setpoint (or boiler boost if active).
6. **Auto mode OFF** → Follows the manual target (UI slider or physical knob).

---

## 🔍 Detailed Operational Scenarios

### Priority 0: Maintenance Switch ON (Root Condition)
- **Trigger:** The `heating_maintenance_switch` is turned **ON**.
- **Behavior:** The automation is completely halted at the root condition level. No actions are executed, and the TRV state is left untouched, allowing manual maintenance or physical work on the radiator without the automation fighting back.

### Priority 1: Window / Door Open
- **Trigger:** Any configured window or door binary sensor (or group) turns **ON** and stays open longer than the configured `window_delay` (default: 30 seconds).
- **Behavior:** Heating is immediately paused to prevent energy waste. 
  - **Target Setpoint:** Set to `window_temp` (default: `5 °C`).
  - **External Temperature:** Sent as the real room temperature so the TRV remains responsive.

### Priority 2: Laundry Override Active
- **Trigger:** Both the room's **Laundry override switch** (`laundry_override_enabled`) and the house-wide **Global laundry drying mode switch** (`laundry_mode_switch`) are **ON**.
- **Behavior:** The room enters laundry drying mode.
  - **Target Setpoint:** Set to the scheduled room setpoint plus the laundry offset (`schedule_sp + laundry_offset_val`, defaulting to `+1.0 °C` if no helper is provided).
  - **External Temperature:** The actual room temperature sensor reading (`room_temp`) is fed to the TRV's external temperature input, allowing the TRV to natively modulate and regulate heat dynamically as clothes dry rather than overheating the room unconditionally.

### Priority 3: Heating Master Switch OFF (Summer Mode)
- **Trigger:** The house-wide `heating_master_switch` is turned **OFF** (representing Summer Mode).
- **Behavior:** The system bifurcates based on whether global laundry drying is active:
  - **If Global Laundry Switch is ON:** Non-laundry or standard rooms have their valves closed down to `window_temp` (`5 °C`).
  - **If Global Laundry Switch is OFF:** All valves are forced **100% open** (using `fully_open_temp` and an artificially cold `fully_open_ext_temp` feed). This mechanical anti-sticking routine prevents radiator valves from seizing up during the summer months when heating is not actively in use.

### Priority 4: Manual Open Switch ON
- **Trigger:** The room's `manual_open_switch` helper is turned **ON**.
- **Behavior:** Forces the radiator fully open on demand.
  - **Target Setpoint:** Set to `fully_open_temp` (default: `35 °C`).
  - **External Temperature:** Fed an artificially cold reading (`fully_open_ext_temp`, default: `15 °C`) so the TRV's internal logic forces the physical valve 100% open.

### Priority 5: Auto Mode ON (Schedule & Boiler Boost)
- **Trigger:** The `auto_mode_switch` is **ON** (or omitted, meaning auto mode is permanently enabled).
- **Behavior:** The TRV follows automated schedules, with a built-in sub-scenario for **Boiler Boost**:
  - **Standard Schedule:** The target setpoint mirrors the `room_setpoint_sensor` (driven by calendar schedules or template helpers).
  - **Boiler Boost Active Sub-scenario:** If boiler boost is enabled, the central boiler is running, and the room temperature drops below the calculated threshold (`schedule_sp + desired_accuracy + boost_diff_val`), the TRV temporarily overrides the schedule and forces **100% open** (`fully_open_temp`) until the room temperature reaches the target setpoint (incorporating hysteresis).

### Priority 6: Auto Mode OFF (Manual Target)
- **Trigger:** The `auto_mode_switch` is **OFF**.
- **Behavior:** The room operates in manual mode.
  - **Target Setpoint:** Follows the manual target set via the dashboard UI target slider (`target_slider`) or directly via physical knob rotations.

---

### Additional Underlying System Scenarios

- **User Takeover (Knob or Slider Intervention):** If a user manually turns the physical TRV knob (while `ignore_rotation_switch` is OFF) or moves the UI slider while in Auto mode, the automation automatically switches the room mode from **Auto** to **Manual** (`auto_switch` turned OFF) and adopts the new target.
- **Child Lock & Ignore Rotation Sync:** Bidirectionally syncs the TRV's physical child lock feature with the dashboard's `ignore_rotation_switch` to block unwanted manual adjustments.
- **Hardware Keep-Alive:** Periodically (based on `keep_alive_interval`, e.g., every 10 minutes), the controller re-sends the external temperature to prevent the TRV from timing out its remote sensor (`remote_source_offline`, ~30 min timeout).

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
   `config/blueprints/automation/pof42428/trv.yaml`
3. Go to **Settings > Automations & Scenes > Blueprints** and click **Reload Blueprints**.

---

## ⚙️ Inputs Configuration Overview

### 1. Room Entities
- **Climate entity (TRV):** The physical radiator climate device.
- **Room temperature sensor:** Reference temperature sensor fed to the TRV.
- *Optional:* External temperature number entity, temperature accuracy number entity, child lock switch, and window/door binary sensors.

### 2. Room Controls (Helpers)
- *Optional:* Target temperature UI slider (`input_number`), start offset entity, auto mode switch, manual open switch, ignore physical rotation switch, laundry override boolean, laundry mode temperature offset entity (`offset_laundry`), and boiler boost configuration entities.

### 3. Global System Entities
- *Optional:* House-wide heating master switch, maintenance switch, and laundry drying mode switch.

### 4. Tuning & Offsets
- Adjust parameters like the valve "fully open" temperature, artificial cold external temperature for forced opening, window temperature fallback, window open delay, trigger debounce duration, and the hardware keep-alive interval.

---

## 📄 License

This project is open-source and available under the terms of the [MIT License](LICENSE).
