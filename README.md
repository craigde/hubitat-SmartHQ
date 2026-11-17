# hubitat-SmartHQ

GE SmartHQ Appliance Integration for Hubitat Elevation.

This provides various driver capabilities for SmartHQ appliances. Note that all operations have a cloud (internet) interaction.

## Acknowledgments

This project is based on the original work by [tomw](https://github.com/tomwpublic/hubitat_SmartHQ). Thank you to tomw for creating and maintaining the original SmartHQ integration that made this fork possible.

The SmartHQ API implementation is substantially based on this work: [https://github.com/simbaja/gehome](https://github.com/simbaja/gehome)

## Installation

Installing with Hubitat Package Manager (HPM) is recommended.

### HPM Installation

1. Open Hubitat Package Manager
2. Search for "SmartHQ"
3. Install the package

### Manual Installation

If you must install manually, follow these steps:

1. In the *Bundles* section of Hubitat, import the `SmartHQHelpersLibrary.zip` bundle (contains the shared `smarthqHelpers` library that the app and drivers `#include`). **Do this first** — the app and drivers will not compile without it.
2. In the *Drivers Code* section of Hubitat, add the **SmartHQ Connector** driver from `devices/smartHQ_connector`. **Required** — the app spawns a child device using this driver to own the WebSocket connection.
3. In the *Apps Code* section of Hubitat, add the SmartHQ app from `apps/smartHQ_app`
4. In the *Drivers Code* section of Hubitat, add any appliance drivers that apply to your system from the `devices` subdirectory
5. Go to *Apps* → *Add User App* → *SmartHQ*
6. Enter your SmartHQ username and password and save
7. Appliance devices will be created automatically

## Architecture

- **SmartHQ** (app) — credentials UI, OAuth flow, dispatches incoming messages to appliance child devices.
- **SmartHQ Connector** (child device, required) — owns the WebSocket connection to the GE Brillion cloud. Hubitat does not expose `interfaces.webSocket` to apps, so this driver is required for the WebSocket to work. The app creates one connector child automatically; you don't add it manually.
- **Appliance drivers** (refrigerator, oven, etc.) — children of the app, one per discovered appliance. Each parses ERD events and exposes Hubitat capabilities.

### Regenerating the Bundle

Hubitat Package Manager (HPM) does not install libraries directly from a manifest — they have to be delivered via a bundle ZIP. The repo includes a pre-built `SmartHQHelpersLibrary.zip` containing the `smarthqHelpers` library. Regenerate it whenever `libraries/smarthqHelpers` changes:

1. In Hubitat: *Libraries Code* → import `libraries/smarthqHelpers` if it isn't already loaded
2. Go to *Bundles* → create a new bundle named `SmartHQ Helpers Library` (namespace `craigde`)
3. Add the `smarthqHelpers` library to the bundle
4. Export the bundle — Hubitat produces a ZIP
5. Replace `SmartHQHelpersLibrary.zip` at the repo root with the exported ZIP
6. Commit and push

The bundle only contains the library, so it rarely needs updating — most ongoing changes are in the app and individual drivers, which HPM installs as plain files.

## Supported Appliances

- Refrigerator
- Oven (with TemperatureMeasurement capability)
- Dishwasher
- Laundry (Washer/Dryer)
- Microwave
- Portable AC
- Home Water Filter
- Ice Maker (Opal Nugget)
- Toaster Oven (new: status, light and control lock; reported by Café Couture ovens as "Toaster Oven")
- Range Hood
- Dehumidifier (driver contributed by Alan_F)

## Usage

- Appliance updates will generally come automatically without polling via WebSocket connection
- Each appliance child device supports the `Refresh` command if you want to poll for current status
- Each appliance has a `healthStatus` attribute (`online`/`offline`) showing whether the appliance itself is connected to GE's cloud, updated every 5 minutes
- The app watches the cloud connection and reconnects by itself if it stops responding for 12 minutes. This can be turned off in the SmartHQ app under *Connection*. The SmartHQ Connector device's `connectionStatus` shows whether the connection is currently open
- Most commands and attributes are self explanatory - try things out!

## Requesting New Appliance Support

If you would like to request support for a new appliance type or additional attributes/commands:

1. Enable debug logging on the SmartHQ app in Hubitat
2. Open the *Logs* view in another browser tab or window
3. Operate the appliance manually and save the logs
4. Open an issue on GitHub with the log and a description of what you were adjusting

Before posting a log, check it for anything you'd rather keep private. Versions before 1.0.14 logged GE login tokens with debug logging on; if you're on an older version, update first or remove the lines starting `refresh =`, `auth =` and `wssInfo =`.

## Version History

Newest first. The full notes for each release are in HPM's release notes (`packageManifest.json`).

### 1.0.14
- New: **Toaster Oven** driver. Shows state, cook mode, temperature, cook time, time remaining and convection, and controls the light and control lock. Confirmed on a Café Couture toaster oven (C9OAAAS2RS3). Requested in issue #24
- Fix: the control lock showed "locked" when the panel was unlocked. This affected the dishwasher too
- Security: debug logs no longer include GE login tokens, the WebSocket URL's access token or login authorization codes. If you have shared logs from an older version, change your GE password
- New: connection watchdog. If the cloud connection stops responding for 12 minutes, the app reconnects by itself. Can be turned off in the SmartHQ app under *Connection*
- New: `healthStatus` (online/offline) on every appliance, showing whether the appliance itself is connected to GE's cloud
- After every connect or reconnect, the app fetches each appliance's full state, so changes made while disconnected show up straight away

### 1.0.13
- Ice maker: new `needsDescaling` attribute (yes/no). Confirmed on an Opal 2.0
- Ice maker: `currentStatus` now reports "missing water source" and "lid open" instead of "unknown"

### 1.0.12
- Dishwasher: `rinseAgentStatus` no longer drops to "unknown" when the drain filter needs cleaning
- Dishwasher: new `filterStatus` (clean the drain filter) and `sanitized` (yes/no: whether the last cycle met GE's sanitize standard). Thanks terminal3

### 1.0.11
- Dishwasher: setting the pod count works again. GE retired the old command, so the driver now writes the count directly, as Home Assistant does. "Add" and "subtract" need a current reading, so press Refresh first if they log a warning. Thanks terminal3

### 1.0.10
- Requests the cloud rejects are now logged as warnings with GE's reason, instead of failing silently
- Dishwasher: `adjustNumberOfPods` now works from Rule Machine
- Dehumidifier: bucket-full and clean-filter status work again on newer firmware. Contributed by youzer-name (PR #18)

### 1.0.9
- Dishwasher: rinse aid status

### 1.0.8
- Login works again after Hubitat changed its HTTP handling in hub firmware 2.5.1.143. Fix from Alan_F

### 1.0.7
- New `reconnect` command on every appliance, so a rule can rebuild the cloud connection

### 1.0.6
- Login: fixed a crash that stopped the integration recovering when it needed a full re-login
- Login: GE's optional "test MFA" prompt is now skipped automatically. Contributed by youzer-name (PR #12)
- If login fails, the app now says why, on its settings page and in the log

### 1.0.4
- New: **Dehumidifier** driver, contributed by Alan_F

### 1.0.3
- Standard Hubitat capabilities on the drivers, so dashboards and rules work natively: temperature and filter status on the refrigerator; an on/off `switch` on the oven, dishwasher, laundry and microwave (read-only except the dishwasher, where it starts and stops a cycle); thermostat mode and filter status on the portable AC; filter status on the ice maker

### 1.0.2
- Fixed the cloud connection failing to start. Hubitat only lets drivers open WebSockets, so the connection now lives in a SmartHQ Connector device that the app creates automatically

### 1.0.1
- The login code moved into the app, so device drivers no longer carry GE's app credentials
- More reliable reconnects
- Setting a portable AC's cooling setpoint no longer forces it into cool mode
- Fixes for several other driver bugs

### 1.0.0 (craigde)
- Forked from tomw's original repository
- Refactored from device driver to app-based architecture (Hubitat best practice)
- Added TemperatureMeasurement capability to oven driver
- Added F/C temperature unit support for oven

### Previous Versions (tomw)
- 0.9.15 - Bugfixes for connection reliability
- 0.9.14 - Bugfix for token failure cases
- 0.9.13 - Bugfix for websocket failure case
- 0.9.12 - Hood support
- 0.9.11 - Ice Maker support
- 0.9.10 - Added selectable region for login
- 0.9.9 - Flow rate and daily usage for Home Water Filter
- 0.9.8 - Home Water Filter support
- 0.9.7 - Dishwasher commands for start, stop, and pod count management
- 0.9.6 - Additional attributes for dishwasher
- 0.9.5 - Improved Washer/Dryer support
- 0.9.4 - Improved air conditioner support
- 0.9.3 - Added Microwave support. Added Sabbath mode setting
- 0.9.2 - Make child logging follow system device setting
- 0.9.1 - Added support for Portable AC
- 0.9.0 - Initial release

## Disclaimer

I have no affiliation with any of the companies mentioned in this readme or in the code. This is an unofficial integration.

## License

Apache License 2.0
