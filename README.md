# TADO Hijack Automatic Update Entity Offset with Manual Push 🔥🌡️

An optimized fork of the original TADO Hijack blueprint that adds manual push button functionality and improves efficiency by eliminating unnecessary every-minute triggers.

## What's New in This

- ✅ **Manual Push Button** - Trigger calibration immediately at any time
- ✅ **Optimized Performance** - Only runs at your chosen interval (not every minute)
- ✅ **Flexible Intervals** - Choose from 5 to 60 minutes in 5-minute increments
- ✅ **Bypass Time Restrictions** - Manual push works outside of scheduled hours

## Use Case

Automatically adjust your Tado thermostat's temperature offset based on a more accurate Zigbee temperature sensor. This ensures your heating system works with precise temperature readings from a strategically placed sensor rather than the Tado's built-in sensor.

## How It Works

**Automatic Mode:**
- Monitors a Zigbee temperature sensor
- Compares it to your Tado climate entity's current temperature
- If the difference exceeds your threshold, updates the offset
- Runs at your chosen interval (5-60 minutes) during specified hours
- Optionally only runs when actively heating

**Manual Mode:**
- Press a button to force immediate calibration
- Bypasses time window and interval restrictions
- Still respects threshold and heating status checks

## Setup Instructions

### Step 1: Install the Blueprint

1. Save the blueprint YAML as a file in `/config/blueprints/automation/tado_hijack_manual.yaml`
2. Or go to **Settings → Automations & Scenes → Blueprints → Import Blueprint**
3. Restart Home Assistant or reload automations

### Step 2: Create Helper Buttons (One Per Room)

For each room where you want manual control:

1. Go to **Settings → Devices & Services → Helpers**
2. Click **"+ Create Helper"**
3. Select **Button**
4. Configure:
   - **Name**: `[Room Name] TADO Manual Calibrate` (e.g., "Living Room TADO Manual Calibrate")
   - **Icon**: `mdi:thermometer-auto` (optional)
5. Click **Create**

Example helper entities you'll create:
- `button.living_room_tado_manual_calibrate`
- `button.bedroom_tado_manual_calibrate`
- `button.kitchen_tado_manual_calibrate`
- `button.office_tado_manual_calibrate`

### Step 3: Create Automations (One Per Room)

For each room:

1. Go to **Settings → Automations & Scenes**
2. Click **"+ Create Automation"**
3. Select **"Use a blueprint"** and choose **"TADO Hijack automatic update Entity Offset with Switch + Manual Push"**
4. Configure the inputs:

| Input | Description | Example |
|-------|-------------|---------|
| **Zigbee Sensor to Monitor** | Your accurate temperature sensor | `sensor.living_room_temperature` |
| **Tado Homekit Entity** | Your Tado climate entity | `climate.living_room_tado` |
| **Tado Hijack offset entity** | The number entity controlling offset | `number.living_room_offset` |
| **Manual Push Button** | The helper button you created | `button.living_room_tado_manual_calibrate` |
| **Threshold** | Minimum temp difference to trigger update | `0.5°C` |
| **Start Time** | When automatic calibration starts | `06:00` |
| **End Time** | When automatic calibration stops | `23:00` |
| **Time Interval** | How often to auto-calibrate | `Every 15 minutes` |
| **Enable only when Heating** | Only calibrate when actively heating | `true` |

5. Name it: `TADO Calibration - [Room Name]`
6. Click **Save**

### Step 4: Add Buttons to Your Dashboard

#### Option A: Individual Buttons Per Room

```yaml
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
