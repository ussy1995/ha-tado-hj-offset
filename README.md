# TADO Hijack Automatic Update Entity Offset with Manual Push 🔥🌡️

An optimized fork of the original **TADO Hijack** blueprint that adds manual push button functionality, improves efficiency by eliminating unnecessary every-minute triggers, and includes failsafe protection against stale sensor data.

## What's New in This Fork

- ✅ **Manual Push Button** — Trigger calibration immediately at any time.
- ✅ **Optimized Performance** — Runs only at your chosen interval (not every minute).
- ✅ **Flexible Intervals** — Choose from 5 to 60 minutes in 5-minute increments.
- ✅ **Bypass Time Restrictions** — Manual push works outside scheduled hours.
- ✅ **Stale Sensor Protection** — Won’t run if the Zigbee sensor reading is too old.
- ✅ **Sensor Validity Checks** — Blocks execution if the sensor is `unknown` / `unavailable`.

## Use Case

Automatically adjust your Tado thermostat’s temperature offset based on a more accurate Zigbee temperature sensor. This helps when the Tado device is positioned in a spot that doesn’t represent the true room temperature (e.g., hallway placement, near a radiator, drafts, etc.).

## How It Works

### Automatic Mode

- Runs on a fixed schedule (every **5–60 minutes**).
- Only runs within the configured **Start Time** / **End Time** window.
- Optional: only runs while the Tado entity reports `hvac_action: heating`.
- Failsafe: only runs if the Zigbee sensor has updated recently.
- If the temperature difference exceeds the **Threshold**, the blueprint updates the offset entity.

### Manual Mode

- Press a helper **button** to trigger calibration immediately.
- Manual push bypasses **time window** and **interval** restrictions.
- Still respects heating-only mode (if enabled), threshold, and failsafe checks.

### Failsafe Protection (Anti-stale)

The automation will **NOT** update the offset if:

- The Zigbee sensor state is `unknown`, `unavailable`, or `none`.
- The Zigbee sensor hasn’t updated within **Maximum Sensor Age** minutes.

This prevents offset updates based on stale readings (e.g., Zigbee mesh issues or a dead sensor battery).

## Setup Instructions

### Step 1: Install the Blueprint

**Option A (File):**
1. Save the blueprint YAML in: `/config/blueprints/automation/`
2. Example filename: `tado_hijack_offset_manual_push.yaml`
3. Reload automations (or restart Home Assistant).

**Option B (Import):**
1. Go to **Settings → Automations & Scenes → Blueprints**.
2. Click **Import Blueprint** and paste the blueprint URL/YAML.

### Step 2: Create Helper Buttons (One Per Room)

If you create one automation per room, you typically want **one manual button per room**.

1. Go to **Settings → Devices & Services → Helpers**.
2. Click **+ Create Helper**.
3. Select **Button**.
4. Name it like: `Living Room TADO Manual Calibrate`.
5. (Optional) Choose an icon like `mdi:thermometer-auto`.

Example entities you might end up with:
- `button.living_room_tado_manual_calibrate`
- `button.bedroom_tado_manual_calibrate`
- `button.kitchen_tado_manual_calibrate`
- `button.office_tado_manual_calibrate`

### Step 3: Create Automations (One Per Room)

1. Go to **Settings → Automations & Scenes → + Create Automation**.
2. Choose **Use a blueprint**.
3. Pick: **TADO Hijack automatic update Entity Offset with Switch + Manual Push**.
4. Fill in the inputs for that room.

#### Example Room Configuration

| Input | What it is | Example |
|------|------------|---------|
| Zigbee Sensor to Monitor | Your accurate temperature sensor | `sensor.living_room_temperature` |
| Tado Homekit Entity | The Tado climate entity | `climate.living_room_tado` |
| Tado Hijack offset entity | The number entity controlling the offset | `number.living_room_offset` |
| Manual Push Button | Room helper button | `button.living_room_tado_manual_calibrate` |
| Threshold | Min delta before change | `0.5°C` |
| Start Time | Auto calibration starts | `06:00` |
| End Time | Auto calibration stops | `23:00` |
| Time Interval | Auto run frequency | `Every 15 minutes` |
| Maximum Sensor Age | Stale-data cutoff | `10 minutes` |
| Enable only when Heating | Run only when heating | `true` |

5. Name the automation like: `TADO Calibration - Living Room`.

### Step 4: Add Manual Buttons to Dashboard

#### Option A: Grid of Room Buttons

type: grid
columns: 2
cards:
  - type: button
    name: Living Room
    icon: mdi:sofa
    entity: button.living_room_tado_manual_calibrate
    tap_action:
      action: toggle
  - type: button
    name: Bedroom
    icon: mdi:bed
    entity: button.bedroom_tado_manual_calibrate
    tap_action:
      action: toggle
  - type: button
    name: Kitchen
    icon: mdi:fridge
    entity: button.kitchen_tado_manual_calibrate
    tap_action:
      action: toggle
  - type: button
    name: Office
    icon: mdi:desk
    entity: button.office_tado_manual_calibrate
    tap_action:
      action: toggle

#### Option B: Compact List

type: entities
title: TADO Manual Calibration
entities:
  - entity: button.living_room_tado_manual_calibrate
    name: Living Room
    icon: mdi:sofa
  - entity: button.bedroom_tado_manual_calibrate
    name: Bedroom
    icon: mdi:bed
  - entity: button.kitchen_tado_manual_calibrate
    name: Kitchen
    icon: mdi:fridge
  - entity: button.office_tado_manual_calibrate
    name: Office
    icon: mdi:desk

## Blueprint Inputs Reference
| Input | Type |  Default | Description |
|------|-------|-----|---------|
|sensor_to_monitor |	Entity (sensor) |	Required	|Temperature sensor with accurate readings
|climate_entity |	Entity (climate) |	Required	|Tado climate entity
|entity_to_update|	Entity (number)|	Required	|Offset number entity to update
|manual_trigger|	Entity (button / input_button)|	Optional	|Manual calibration trigger
|threshold|	Number|	Required	|Minimum temp difference to update offset
|start_time|	Time|	Required	|Auto mode start time
|end_time|	Time|	Required	|Auto mode end time
|time_interval_minutes|	Select|	/15	|Auto run interval (5–60 min)
|sensor_max_age_minutes	|Number	|10|	Max sensor age before blocking run
|enable_switch|	Boolean|	true	|Only run when heating (hvac_action: heating)

### Logic 
#### Automatic Calibration
Every interval:
Failsafe: sensor updated within max age?
Failsafe: sensor state valid?
Within time window?
Heating-only enabled? If yes, is hvac_action = heating?
Delta exceeds threshold?
If yes → set new offset.

#### Manual Push
On button press:
Failsafe: sensor updated within max age?
Failsafe: sensor state valid?
Heating-only enabled? If yes, is hvac_action = heating?
Delta exceeds threshold?
If yes → set new offset.

### Troubleshooting
#### Manual button does nothing
Confirm the helper button is selected in the automation.
Check the automation trace: it will show which condition failed.
Check the sensor age (failsafe may be blocking due to stale data).

#### Automation runs but offset doesn't change
The delta might not exceed the threshold.
Tado may not be heating (if heating-only is enabled).

#### Automation seems to skip scheduled runs
Increase Maximum Sensor Age if your Zigbee sensor updates slowly.

