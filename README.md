# TADO Hijack Automatic Update Entity Offset with Manual Push 🔥🌡️

An optimized blueprint for automatically calibrating Tado TRV temperature offsets using external Zigbee temperature sensors. Features manual push button functionality and mathematically correct cumulative offset calculation.

## Table of Contents

- [What's New](#whats-new)
- [Use Case](#use-case)
- [How It Works](#how-it-works)
- [Setup Instructions](#setup-instructions)
- [Dashboard Examples](#dashboard-examples)
- [Monitoring & Debugging](#monitoring--debugging)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)
- [Technical Details](#technical-details)
- [Credits](#credits)

## What's New

- ✅ **Cumulative Offset Calculation** - Correctly handles Tado's offset-adjusted temperature readings
- ✅ **Manual Push Button** - Trigger calibration immediately at any time
- ✅ **No Heating Restrictions** - Runs anytime (heating-only mode removed)
- ✅ **Optimized Performance** - Only runs at your chosen interval (5-60 minutes)
- ✅ **Stale Sensor Protection** - Blocks updates if Zigbee sensor hasn't updated recently
- ✅ **Debug Logging** - Track Zigbee temp, Tado temp, current offset, delta, and new offset
- ✅ **Bypass Time Restrictions** - Manual push works outside of scheduled hours

## Use Case

Automatically adjust your Tado thermostat's temperature offset based on a more accurate Zigbee temperature sensor. This ensures your heating system works with precise temperature readings from a strategically placed sensor rather than the Tado's built-in sensor (which suffers from self-heating when the radiator is active).

## How It Works

### Offset Calculation Logic

The blueprint uses **cumulative offset calculation** to correctly handle Tado's already-offset temperature readings:

```
new_offset = current_offset + (zigbee_temp - tado_display_temp)
```

This is mathematically correct because:
- Home Assistant sees the **displayed temperature** (already adjusted by current offset)
- To calculate the new absolute offset, we must add the delta to the existing offset
- This prevents the offset from "fighting itself" and causing temperature jumps

**Example:**
- Tado internal sensor: 23.5°C (affected by radiator heat)
- Current offset: -1.5°C
- Tado displays: 22.0°C
- Zigbee sensor: 22.1°C (accurate room temperature)
- Delta: 22.1 - 22.0 = +0.1°C
- New offset: -1.5 + 0.1 = **-1.4°C**
- Result: Tado now displays 22.1°C (matches Zigbee)

### Automatic Mode

- Monitors a Zigbee temperature sensor for staleness and updates
- Compares it to your Tado climate entity's current temperature
- If the difference exceeds your threshold, calculates and applies new offset
- Runs at your chosen interval (5-60 minutes) during specified hours
- Respects sensor age limits (blocks if sensor hasn't updated recently)
- Checks if sensor changed in last hour (prevents stuck sensor issues)

### Manual Mode

- Press a button to force immediate calibration
- Bypasses time window, interval, and threshold restrictions
- Respects sensor staleness checks (unless override enabled)
- Useful for immediate adjustment after room changes or manual verification

## Setup Instructions

### Step 1: Install the Blueprint

1. Copy the blueprint YAML to `/config/blueprints/automation/tado/tado_hijack_manual.yaml`
2. Or go to **Settings → Automations & Scenes → Blueprints → Import Blueprint**
3. Paste the blueprint URL or YAML content
4. Restart Home Assistant or reload automations

### Step 2: Create Helper Buttons (One Per Room)

For each room where you want manual control:

1. Go to **Settings → Devices & Services → Helpers**
2. Click **"+ Create Helper"**
3. Select **Input Button**
4. Configure:
   - **Name**: `[Room Name] Offset Manual` (e.g., "Backroom Offset Manual")
   - **Icon**: `mdi:sync` or `mdi:thermometer-auto`
5. Click **Create**

Example helper entities:
- `input_button.backroom_offset_manual`
- `input_button.main_bedroom_offset_manual`
- `input_button.office_offset_manual`

### Step 3: Create Automations (One Per Room)

For each room:

1. Go to **Settings → Automations & Scenes**
2. Click **"+ Create Automation"**
3. Select **"Use a blueprint"** and choose **"TADO Hijack automatic update Entity Offset with Manual Push"**
4. Configure the inputs:

| Input | Description | Example |
|-------|-------------|---------|
| **Zigbee Sensor to Monitor** | Your accurate temperature sensor | `sensor.backroom_temp_sensor_temperature` |
| **Tado Homekit Entity** | Your Tado climate entity (HomeKit Controller) | `climate.tado_smart_radiator_thermostat_va0908283136` |
| **Tado Hijack offset entity** | The number entity controlling offset | `number.tado_backroom_trv_temperature_offset` |
| **Manual Push Button** | The helper button you created | `input_button.backroom_offset_manual` |
| **Manual push ignores failsafe** | Allow manual push even if sensor is stale | `false` |
| **Maximum Sensor Age** | Block updates if sensor older than X minutes | `15 minutes` |
| **Threshold** | Minimum temp difference to trigger auto update | `0.5°C` |
| **Debug logging** | Log calibration details to logbook | `true` |
| **Start Time** | When automatic calibration starts | `06:00` |
| **End Time** | When automatic calibration stops | `23:00` |
| **Time Interval** | How often to auto-calibrate | `Every 15 minutes` |

5. Name it: `TADO: [Room Name] Offset` (e.g., "TADO: Backroom Offset")
6. Click **Save**

## Dashboard Examples

### Option A: Compact Horizontal Layout with Last Run Info

```yaml
type: horizontal-stack
cards:
  - type: custom:button-card
    entity: input_button.backroom_offset_manual
    name: Push
    icon: mdi:sync
    tap_action:
      action: toggle
    styles:
      card:
        - height: 50px
  - type: custom:button-card
    entity: automation.tado_backroom_offset
    name: Last Triggered
    icon: mdi:clock-outline
    show_state: false
    show_name: true
    show_label: true
    label: >
      [[[
        const last = entity.attributes.last_triggered;
        if (!last) return 'Never';
        const date = new Date(last);
        const now = new Date();
        const mins = Math.floor((now - date) / 60000);
        if (mins < 1) return 'Just now';
        if (mins < 60) return mins + ' min ago';
        const hours = Math.floor(mins / 60);
        if (hours < 24) return hours + 'h ago';
        return Math.floor(hours / 24) + 'd ago';
      ]]]
    tap_action:
      action: more-info
    styles:
      card:
        - height: 50px
      label:
        - font-size: 11px
        - color: white
```

### Option B: Individual Room Buttons

```yaml
type: grid
columns: 2
cards:
  - type: button
    name: Backroom
    icon: mdi:sofa
    entity: input_button.backroom_offset_manual
    tap_action:
      action: toggle
  - type: button
    name: Main Bedroom
    icon: mdi:bed
    entity: input_button.main_bedroom_offset_manual
    tap_action:
      action: toggle
  - type: button
    name: Office
    icon: mdi:desk
    entity: input_button.office_offset_manual
    tap_action:
      action: toggle
```

### Option C: Compact Entity List

```yaml
type: entities
title: TADO Manual Calibration
entities:
  - entity: input_button.backroom_offset_manual
    name: Backroom
    icon: mdi:sofa
  - entity: input_button.main_bedroom_offset_manual
    name: Main Bedroom
    icon: mdi:bed
  - entity: input_button.office_offset_manual
    name: Office
    icon: mdi:desk
```

## Monitoring & Debugging

### Enable Debug Logging

Set **Debug logging** to `true` in the blueprint configuration. This logs to **History → Logbook**:

**Manual Push:**
```
Manual push. Zigbee=22.1, Tado=22.5, CurrentOffset=0.2, Delta=-0.4, NewOffset=-0.2
```

**Auto Run:**
```
Auto run. Zigbee=21.6, Tado=20, CurrentOffset=0, Delta=1.6, NewOffset=1.6
```

### Check Automation Traces

1. Go to **Settings → Automations & Scenes**
2. Find your TADO automation
3. Click **Traces** tab
4. Review recent executions to see which conditions passed/failed

### Verify Offset is Applied

1. **Developer Tools → States**
2. Find `number.tado_[room]_trv_temperature_offset`
3. Check the state matches expected offset
4. Find `climate.tado_smart_radiator_thermostat_[id]`
5. Check `current_temperature` attribute reflects the offset

## Troubleshooting

### Temperature Jumps After Offset Applied

**Cause:** Using absolute offset calculation instead of cumulative.

**Solution:** This blueprint already uses cumulative calculation. If you're seeing jumps, check that you're using the latest version.

### Offset Not Applied / Automation Not Running

**Possible causes:**
1. **Sensor is stale** - Check sensor `last_updated` in Developer Tools → States
2. **Outside time window** - Check start/end times match when you expect it to run
3. **Below threshold** - Temperature difference might be less than threshold setting
4. **Sensor stuck** - Sensor hasn't changed in last hour (failsafe triggered)

**Solution:** 
- Use manual push to force update
- Check automation traces for which condition failed
- Reduce threshold to 0.3°C for more frequent updates

### Manual Button Shows "Off" / No Time Display

**Cause:** Automation state is "on"/"off", not a timestamp.

**Solution:** Use the `custom:button-card` configuration from Option A above, which extracts `last_triggered` attribute and formats it.

## Advanced Configuration

### Recommended Settings for Different Scenarios

**Active Heating Period (high fluctuation):**
- Threshold: `0.3°C`
- Interval: `Every 5 minutes`
- Time window: `06:00 - 22:00`

**Overnight (stable temps):**
- Threshold: `0.5°C`
- Interval: `Every 15 minutes`
- Time window: `22:00 - 06:00`

**Testing/Initial Setup:**
- Threshold: `0.1°C`
- Interval: `Every 5 minutes`
- Debug logging: `true`
- Sensor max age: `10 minutes`

### Multiple Sensors per Room

If you have multiple Zigbee sensors in one room, create a **Template Sensor** averaging them:

```yaml
template:
  - sensor:
      - name: "Living Room Average Temperature"
        unit_of_measurement: "°C"
        device_class: temperature
        state: >
          {{ (
            (states('sensor.living_room_sensor_1_temperature') | float(0) +
             states('sensor.living_room_sensor_2_temperature') | float(0)) / 2
          ) | round(1) }}
```

Then use `sensor.living_room_average_temperature` as the Zigbee sensor input.

## Technical Details

### Why Cumulative Offset?

The Tado Hijack integration (via HomeKit Controller) provides `current_temperature` as the **already-offset display value**, not the raw internal sensor reading.

To calculate the correct new absolute offset:
```
T = raw internal TRV sensor
O = current offset
D = displayed temperature (what we see in HA)
Z = Zigbee (true room) temperature

Tado applies: D = T + O
We want: Z = T + O_new
We know: T = D - O
Therefore: O_new = Z - T = Z - (D - O) = O + (Z - D)
```

This is the **cumulative formula**: `new_offset = current_offset + (zigbee - display)`

### Performance Optimization

Original blueprint triggered every minute with modulo check. This version uses direct `time_pattern` minutes input, reducing trigger overhead by 3-12x depending on interval chosen.

### Blueprint Variables

| Variable | Source | Purpose |
|----------|--------|---------|
| `zigbee_temp` | Zigbee sensor state | Reference temperature |
| `tado_display_temp` | Climate `current_temperature` attribute | Tado's offset-adjusted display |
| `current_offset` | Number entity state | Existing offset value |
| `delta` | Calculated | Temperature difference |
| `new_offset` | Calculated | New absolute offset to apply |

## Credits

- Original blueprint: [davidepanato/ha-bp-tado-hj-control](https://github.com/davidepanato/ha-bp-tado-hj-control)
- Cumulative offset calculation: Community research on Tado offset behavior
- Manual push functionality: Custom enhancement

## License

MIT License - Free to use and modify

## Support

For issues, questions, or improvements, please open an issue on the GitHub repository.
