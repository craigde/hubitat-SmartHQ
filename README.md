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

1. In the *Apps Code* section of Hubitat, add the SmartHQ app from `apps/smartHQ_app`
2. In the *Drivers Code* section of Hubitat, add any drivers that apply to your system from the `devices` subdirectory
3. In the *Bundles* section of Hubitat, import the `hubitat_SmartHQ.zip` bundle (contains shared libraries)
4. Go to *Apps* → *Add User App* → *SmartHQ*
5. Enter your SmartHQ username and password and save
6. Appliance devices will be created automatically

## Supported Appliances

- Refrigerator
- Oven (with TemperatureMeasurement capability)
- Dishwasher
- Laundry (Washer/Dryer)
- Microwave
- Portable AC
- Home Water Filter
- Ice Maker (Opal Nugget)
- Range Hood

## Usage

- Appliance updates will generally come automatically without polling via WebSocket connection
- Each appliance child device supports the `Refresh` command if you want to poll for current status
- Most commands and attributes are self explanatory - try things out!

## Requesting New Appliance Support

If you would like to request support for a new appliance type or additional attributes/commands:

1. Enable debug logging on the SmartHQ app in Hubitat
2. Open the *Logs* view in another browser tab or window
3. Operate the appliance manually and save the logs
4. Open an issue on GitHub with the log and a description of what you were adjusting

## Version History

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
