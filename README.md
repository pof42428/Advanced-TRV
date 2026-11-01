# Advanced Single-Room TRV Controller (Home Assistant Blueprint)

[![GitHub Release](https://img.shields.io/github/v/release/pof42428/Advanced-TRV?style=flat-square&color=blue)](https://github.com/pof42428/Advanced-TRV/releases)
[![Home Assistant Blueprint](https://img.shields.io/badge/Home%20Assistant-Blueprint-blue?logo=homeassistant&style=flat-square)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fpof42428%2FAdvanced-TRV%2Fblob%2Fmain%2Ftrv.yaml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)

A production-grade, highly reliable **Home Assistant** blueprint for controlling a single room's Thermostatic Radiator Valve (TRV) using native temperature regulation, schedule setpoints, physical knob synchronization, window sensor pauses, boiler boost features, and global override modes.

Specifically optimized for TRVs that support an **external temperature input** and a **temperature accuracy (start offset)** setting (such as the *Sonoff TRVZB* running firmware ≥ 1.3 via Zigbee2MQTT).

---

## 🚀 Quick Setup

Import this blueprint directly into your Home Assistant instance:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fpof42428%2FAdvanced-TRV%2Fblob%2Fmain%2Ftrv.yaml)

---

## ⚡ Features

- **🌡️ Native TRV Regulation:** Pushes the target setpoint, room temperature, and accuracy threshold (`-1.0 °C` to `-0.2 °C`) directly to the TRV so internal firmware regulates heat without automation jitter.
- **🔄 Bidirectional Sync:** Physical TRV knob adjustments and dashboard UI sliders reflect each other in real-time. Manual rotation automatically switches the room from auto to manual mode unless locked.
- **🔒 Child Lock & Physical Lockout:** Syncs `trv_child_lock` bidirectionally with an `ignore_rotation_switch` helper to ignore unwanted manual knob adjustments.
- **🪟 Window & Door Pause:** Automatically drops target setpoint to `5 °C` (configurable) when open windows or doors are detected beyond the set debounce delay.
- **🧺 Laundry Drying Mode:** Boosts room temperature via `offset_laundry` (or forces full heat) when local and global laundry drying toggles are ON, while continuing to feed real-time room temperature to allow native modulation.
- **🔥 Boiler Boost Support:** Forces TRV open when central heating boiler turns ON and room temperature drops below the configured boost threshold.
- **📡 External Sensor Keep-Alive:** Periodically re-sends external temperature values to prevent hardware offline timeouts (`remote_source_offline`, ~30 min).

---

## 🎛️ Control Priority Order

The controller evaluates actions in strict priority order (highest priority first):

| Priority | Condition | Resulting TRV Action |
|---|---|---|
| **0. Root** | Maintenance switch `ON` | Automation halted completely. |
| **1. Pause** | Window / Door open | Heating paused; TRV set to `window_temp` (`5 °C`). |
| **2. Override** | Laundry override active | TRV target set to `Room setpoint + offset_laundry` (default `+1 °C`). Real room temperature sent to external sensor input. |
| **3. Master** | Master heating switch `OFF` | **Summer mode:** If global laundry switch is `ON`, non-laundry valves close (`5 °C`). Otherwise, valves stay forced 100 % open to prevent valve sticking. |
| **4. Force** | Manual open switch `ON` | Valve forced 100 % open (`fully_open_temp` setpoint + artificial cold external temp feed). |
| **5. Auto** | Auto mode switch `ON` | Follows scheduled room setpoint (or boiler boost setpoint if active). |
| **6. Manual** | Auto mode switch `OFF` | Follows manual UI target slider or physical knob rotation. |

---

## 🧺 How Laundry Drying Mode Works

When both the **Local Laundry Override Switch** (`laundry_override_enabled`) and the **Global Laundry Drying Mode Switch** (`laundry_mode_switch`) are turned `ON`:

1. **Target Temperature Calculation:**
   The TRV target setpoint is set to:
   $$\text{TRV Target} = \text{Room Setpoint Sensor} + \text{Laundry Offset}$$
   *(If `offset_laundry` is omitted, it defaults to `+1.0 °C`).*

2. **Real-Time Temperature Feedback:**
   The actual room temperature reading (`room_temp_sensor`) is continuously pushed to the TRV's external temperature input (`trv_external_temp`).

3. **Native Modulation:**
   Instead of forcing the valve open unconditionally, the TRV natively modulates its valve state—heating aggressively while the room is cool and closing down as the room reaches the elevated laundry target.

---

## 📦 Blueprint Entities Overview

### Required Inputs
- **Climate entity (TRV):** Main `climate` entity of your thermostatic valve.
- **Room temperature sensor:** External room thermometer (`sensor` with `device_class: temperature`).
- **Room setpoint sensor:** Schedule target temperature helper (`sensor` or `input_number`).

### Optional Inputs & Helpers
- **TRV External Temperature Input:** `number` entity of TRV (e.g., `number.xxx_external_temperature_input`).
- **TRV Temperature Accuracy:** `number` entity of TRV start threshold (e.g., `number.xxx_temperature_accuracy`).
- **TRV Child Lock Switch:** `switch` entity for locking physical knob.
- **Target Temperature UI Slider:** `input_number` helper mirroring target setpoint.
- **Auto Mode Switch:** `input_boolean` helper (`ON` = follow schedule, `OFF` = manual).
- **Manual Open Switch:** `input_boolean` helper to force valve 100 % open.
- **Ignore Physical Rotation Switch:** `input_boolean` helper to ignore knob turns.
- **Laundry Override Switch:** `input_boolean` helper (per room).
- **Laundry Mode Temperature Offset:** `input_number` helper (`offset_laundry`).
- **Global Laundry Drying Switch:** `input_boolean` helper (house-wide).
- **Heating Master Switch:** `input_boolean` helper (`OFF` = Summer Mode).
- **Heating Maintenance Switch:** `input_boolean` helper (`ON` = halt automation).
- **Boiler State Switch:** `input_boolean` or `binary_sensor` representing boiler state.
- **Boiler Boost Feature Switch:** `input_boolean` to enable/disable boiler boost.
- **Boost Temp Difference Entity:** `input_number` threshold value for boost.

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).
