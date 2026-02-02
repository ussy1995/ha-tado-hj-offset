# Blueprint: TADO Hijack automatic update Entity Offset with Switch 🔥🌡️

This blueprint allows you to dynamically update the temperature offset of a Tado climate entity (specifically via a Home Assistant `number` entity representing the offset) based on a Zigbee sensor's readings. It includes controls for time range, checking heating status, and customizable update intervals.

## Use Case:
You may want to adjust the temperature offset on your climate entity based on a more accurate or strategically placed sensor. This blueprint enables you to automate that adjustment within a specified time range and only when the heating system is on.

## How It Works:
- **Sensor Monitoring:** You specify a Zigbee temperature sensor whose state will be monitored.
- **Climate Entity:** You select the Tado climate entity to read the current temperature from.
- **Offset Entity:** You select the `number` entity that controls the Tado offset (e.g., exposed via HomeKit or Tado integration).
- **Threshold Control:** You can set a maximum allowed temperature difference between the monitored sensor and the climate entity before triggering an update.
- **Time Range:** You can define start and stop times for when the automation should be active.
- **Interval:** You can customize how often the check runs (in minutes).
- **Heat Mode Control:** There's an optional toggle to enable or disable this automation based on whether the climate entity is in HEAT mode (specifically checking `hvac_action: heating`).

## Inputs:
- `Zigbee Sensor to Monitor:` The temperature sensor whose readings will determine the true temperature.
- `Tado Homekit Entity:` The climate entity (Tado) to read the current internal temperature from.
- `Tado Hijack offset entity:` The `number` entity controlling the offset that will be updated.
- `Threshold:` The temperature difference required to trigger an update (default: 0-10°C).
- `Start Time:` Time when the automation starts.
- `End Time:` Time when the automation ends.
- `Time Interval (minutes):` How frequently to check and update the calibration (default: 15 minutes).
- `Enable Automation Only When Heating:` Boolean to control whether the automation only runs when the climate entity is actually heating.

## Example:
You have a Zigbee sensor in your living room and want to adjust the offset on your Tado thermostat. The automation calculates the difference between the Zigbee sensor and the Tado's internal reading. If the difference exceeds the threshold, it updates the specific "offset" number entity. This happens every 15 minutes (or your configured interval) between your set start and end times, and optionally only if the heating is active.

## Trigger & Condition:
The automation triggers every minute but processes logic based on the configured `Time Interval`. It checks:
1. Is it within the Start/End time?
2. Is the current minute a multiple of the configured interval?
3. If "Enable Automation only when Heating" is on, is the Tado entity currently heating?

If these conditions are met and the temperature difference exceeds the threshold, the offset is updated.

## Action:
The action calculates the new offset:
`New Offset = (Zigbee Temp - Tado Temp)`
It then updates the target `number` entity with this value.