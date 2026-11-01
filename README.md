# Advanced TRV Room Controller

A production-grade **Home Assistant Automation Blueprint** for single-room heating control with Thermostatic Radiator Valves (TRVs) that support an external temperature input and start threshold / temperature accuracy settings (e.g., Sonoff TRVZB via Zigbee2MQTT).

---

## Overview

Unlike standard ON/OFF heating blueprints, this controller leverages the **TRV's native internal regulation algorithm**. Instead of constantly overriding valve positions manually, it continuously feeds the TRV with precise setpoints, real-time room temperature readings, and start threshold offsets.

### Features
* **Native TRV Regulation**: Relies on the TRV firmware to modulate valve position accurately based on room temperature, setpoint, and accuracy thresholds.
* **Smart Laundry Drying Mode**: Allows boosting a room's target temperature by a custom offset (`offset_laundry`) when drying clothes, while continuing to pass real room temperature readings to allow natural valve modulation.
* **Dynamic Schedules & UI Controls**: Seamlessly switches between schedule-driven *Auto Mode* and UI/knob-driven *Manual Mode*.
* **Knob Synchronization & Child Lock**: Supports bidirectionally syncing UI sliders with physical TRV knob turns, with an option to lock out physical rotation (`ignore_rotation_switch`).
* **Boiler Boost Support**: Triggers target setpoint boosts when boiler state and room temperature thresholds indicate active heating demand.
* **Window/Door Pause**: Pauses heating (drops TRV setpoint to a low target) when windows or doors remain open past a configurable delay.
* **Hardware Keep-Alive**: Periodically re-sends external temperature measurements to prevent TRVs from timing out their remote sensor connections (`remote_source_offline`).

---

## Priority Stack (Highest to Lowest)

When multiple conditions or overrides occur at the same time, the blueprint evaluates them strictly in the following priority order:

| Priority | Control Mode | Description |
| :--- | :--- | :--- |
| **0** | **Maintenance Switch** | Halts the automation completely while `ON`. |
| **1** | **Window Open** | Drops TRV setpoint to frost protection (`window_temp`, default `5 °C`) after delay. |
| **2** | **Laundry Override** | Boosts target setpoint by `offset_laundry` (default `+1 °C`) above schedule while keeping real external temperature active. |
| **3** | **Summer Mode (Master OFF)** | Closes valves or leaves them fully open based on global laundry switch state. |
| **4** | **Manual Open** | Forces valve 100% open using artificial temperature override. |
| **5** | **Auto Mode** | Follows schedule setpoint (or active Boiler Boost). |
| **6** | **Manual Mode** | Follows UI slider or physical knob input. |

---

## Detailed Feature Breakdown: Laundry Drying Mode

### How it Works

When both the **Global Laundry Drying Mode Switch** (`laundry_mode_switch`) and a room's local **Laundry Override Switch** (`laundry_override_enabled`) are active (`ON`), the room enters **Laundry Mode**.

```
                       +-------------------------------+
                       | Global Laundry Switch = ON    |
                       | Local Room Switch = ON        |
                       +---------------+---------------+
                                       |
                                       v
                        +--------------+--------------+
                        | Control Mode: Laundry Mode  |
                        +--------------+--------------+
                                       |
         +-----------------------------+-----------------------------+
         |                                                           |
         v                                                           v
+-------------------------------+                           +-------------------------------+
| Target Setpoint Calculation  |                           | External Temperature Input    |
|                               |                           |                               |
| Target = Scheduled Setpoint   |                           | TRV receives the REAL         |
|        + offset_laundry       |                           | Room Temperature Sensor state |
| (Default: Schedule + 1.0 °C)  |                           | (No artificial temp trick)    |
+-------------------------------+                           +-------------------------------+
         |                                                           |
         +-----------------------------+-----------------------------+
                                       |
                                       v
                   +-------------------+-------------------+
                   |   TRV Native Modulation Algorithm    |
                   |                                       |
                   | Opens valve if Room <= Target + Acc   |
                   | Modulates down as Room approaches SP  |
                   | Closes valve when target reached      |
                   +---------------------------------------+
```

1. **Target Temperature Calculation**: 
   The blueprint takes the current schedule setpoint (`room_setpoint_sensor`) and adds the value defined in the optional **Laundry Mode Temperature Offset** entity (`offset_laundry`). 
   * *Fallback*: If `offset_laundry` is not defined or unconfigured, it defaults to a **+1.0 °C** offset.
2. **Real-Time Sensor Relay**: 
   Unlike manual 100% forced overrides, Laundry Mode feeds the TRV's external temperature input (`trv_external_temp`) with the **actual room temperature** measured by `room_temp_sensor`.
3. **TRV Native Regulation**: 
   The TRV uses its internal algorithm to modulate the radiator valve. If the room is below the new laundry target, the valve opens. As the room warms up and reaches the boosted laundry setpoint, the TRV automatically scales back or closes the valve, preventing room overheating while ensuring extra drying heat.

---

## Blueprint Inputs Reference

### Room Entities
* `trv_climate` *(Required)*: The main climate entity for the TRV.
* `room_temp_sensor` *(Required)*: Sensor providing actual room temperature.
* `room_setpoint_sensor` *(Required)*: Entity providing the scheduled target temperature.
* `trv_external_temp` *(Optional)*: Number entity on the TRV for remote temperature input.
* `trv_temp_accuracy` *(Optional)*: Number entity for TRV start threshold offset.
* `trv_child_lock` *(Optional)*: Switch entity for hardware child lock.
* `window_sensors` *(Optional)*: Binary sensor or group of window/door contacts.

### Room Controls (Helpers)
* `target_slider` *(Optional)*: `input_number` helper for manual UI adjustment.
* `start_offset_entity` *(Optional)*: `input_number` helper for dynamic start threshold accuracy.
* `auto_mode_switch` *(Optional)*: `input_boolean` helper toggling Auto vs. Manual mode.
* `manual_open_switch` *(Optional)*: `input_boolean` helper to force radiator wide open.
* `ignore_rotation_switch` *(Optional)*: `input_boolean` helper to revert knob adjustments.
* `laundry_override_enabled` *(Optional)*: `input_boolean` helper enabling laundry override for this specific room.
* `offset_laundry` *(Optional)*: `input_number` helper specifying the temperature offset during laundry mode (defaults to `1 °C`).
* `boiler_boost_switch` *(Optional)*: `input_boolean` helper enabling boiler boost logic.
* `boiler_state_switch` *(Optional)*: Binary sensor/switch indicating central boiler status.
* `boost_temp_difference` *(Optional)*: `input_number` helper setting the boost threshold delta.

### Global Entities
* `heating_master_switch` *(Optional)*: Global toggle for heating system (OFF = Summer Mode).
* `heating_maintenance_switch` *(Optional)*: Global toggle to halt automation entirely.
* `laundry_mode_switch` *(Optional)*: House-wide laundry drying mode toggle.

---

## Installation & Setup

1. Copy the blueprint YAML into your Home Assistant installation under `config/blueprints/automation/helryser/trv.yaml`.
2. Reload Home Assistant Automations (`Developer Tools` -> `YAML` -> `Automations`).
3. Create an automation from the blueprint UI.
4. (Optional) Create an `input_number` helper named `Laundry Target Offset` (e.g., Min: `0.5`, Max: `5.0`, Step: `0.5`, Unit: `°C`) and assign it to `offset_laundry`.