[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge)](https://github.com/hacs/integration)

# hass-opnsense

Join `OPNsense` with `Home Assistant`!

`hass-opnsense` uses the built-in APIs in `OPNsense` for all
interactions.

Initial development was done against `OPNsense` `23.1` and `Home Assistant` `2023.5`.

# Overview

- [Installation](#Installation)
  - [OPNsense plugin](#OPNsense_plugin)
  - [Home Assistant integration](#HomeAssistant_integration)
    - [HACS installation](#HACS_installation)
    - [Manual installation](#Manual_installation)
- [Configuration](#Configuration)
  - [OPNsense](#OPNsense_plugin)
  - [HA Config](#config)
  - [Options](#options)
- [Entities](#entities)
  - [Binary Sensor](#binary_sensor)
  - [Device Tracker](#device_tracker)
  - [Sensor](#sensor)
  - [Switch](#switch)
  - [Services](#services)
- [Known Issues](#known-issues)
  - [AdGuardHome](#AdGuardHome)

# installation

# Installation

This integration currently **replaces** the built-in `OPNsense` integration
which only provides `device_tracker` functionality, be sure to remove any
associated configuration for the built-in integration before installing this
replacement.

## HomeAssistant_integration

In `Home Assistant`, add this repository to your `HACS` installation or clone the directory manually.

### HACS_installation

In HACS, add this as a custom repository: https://github.com/11harveyj/hass-opnsense then go to the
HACS integrations page, search for `OPNsense integration for Home Assistant` and install it. Once the
integration is installed be sure to restart `Home Assistant`.

### Manual_installation

Copy the contents of the custom_components folder to your `Home Assistant` config/custom_components
folder and restart `Home Assistant`.

# Configuration

Configuration is managed entirely from the UI using `config_flow` semantics.
Simply go to `Configuration -> Integrations -> Add Integration` and search for `OPNsense`
in the search box. If you can't find it in the list (well-known HA issue) you need to do
a 'hard-refresh' of the browser (ctrl-F5) then open the list again, you'll find it there.

## OPNsense

- Create a new user or choose an existing user, and create an API key associated to
  to that user. When creating the API key, `OPNsense` will push the API file containing
  the API key and API secret to your browser, you'll find it in the download folder.
- If using a non `admin` user account ensure the user has the following privileges:
  - `Diagnostics: ARP Table`
  - `Diagnostics: NDP Table`
  - `Diagnostics: Netstat`
  - `Status: Traffic Graph`
  - `System:Firmware`

## config

- `Host` - The FQDN or IP to your `OPNsense` installation (ie: `192.168.1.1`)
- `Use SSL` - Use SSL to communicate with `OPNsense`
- `Verify SSL Certificate` - If the SSL certificate should be verified or not
  (if you get an SSL error try unchecking this)
- `API Key` - The API key created previously
- `API Secret` - The API secret of the API key
- `Firewall Name` - A custom name to be used for `entity` naming (default: use
  the `OPNsense` `hostname`)
- `Disable SSL Warnings` - Flag that will disable all SSL warnings if set

## options

- `Scan Interval (seconds)` - scan interval to use for state polling (default:
  `30`)
- `Enable Device Tracker` - turn on the device tracker integration using
  `OPNsense` arp table (default: `false`)
- `Device Tracker Scan Interval (seconds)` - scan interval to use for arp
  updates (default: `60`)
- `Device Tracker Consider Home (seconds)` - seconds to wait until marking
  a device as not home after not being seen.
  (default: `0`)
  - `0` - disabled (if device is not present during any given scan interval it
    is considered away)
  - `> 0` - generally should be a multiple of the configured scan interval

# entities

Many `entities` are created by `hass-opnsense` for stats etc. Due to to volume
of entities many are disabled by default. If something is missing be sure to
review the disabled entities as what you're looking for is probably there.

## binary_sensor

- carp status (enabled/disabled)
- system notices present (the bell icon in the upper right of the UI)
- firmware updates available

## device_tracker

`ScannerEntity` entries are created for the `OPNsense` arp table. Disabled by
default. Not only is the feature disabled by default but created entities are
currently disabled by default as well. Search the disabled entity list for the
relevant mac addresses and enable as desired.

Note that by default `FreeBSD`/`OPNsense` use a max age of 20 minutes for arp
entries (sysctl `net.link.ether.inet.max_age`). You may lower that using
`System -> Advanced -> System Tunables` if desired.

Also note that if you are running `AdGuardHome` DNS queries may get throttled
causing issues with the tracker. See #22 for details.

## sensor

- system details (name, version, boottime, etc)
- pfstate details (used, max, etc)
- cpu details (average load, frequency, etc)
- mbuf details
- memory details
- filesystem usage
- interface details (status, stats, pps, kbs (time samples are based on the
  `Scan Interval (seconds)` config option))
- gateways details (status, delay, stddev, loss)

## switch

Below has not yet been implemented in this version

All of the switches below are disabled by default.

- filter rules - enable/disable rules
- nat port forward rules - enable/disable rules
- nat outbound rules - enable/disable rules
- services - start/stop services (note that services must be enabled before they can be started)

# update

Update entities for:

 - System
 - Packages

These are polled at the defined interval during config *10 (eg 30*10 = 300 seconds)

# services

```
service: opnsense.close_notice
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  # default is to clear all notices
  # id: <some id>

service: opnsense.file_notice
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  notice: "hello world"

service: opnsense.system_halt
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present

service: opnsense.system_reboot
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present

service: opnsense.start_service
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  service_name: "dpinger"

service: opnsense.stop_service
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  service_name: "dpinger"

service: opnsense.restart_service
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  service_name: "dpinger"
  # only_if_running: false

service: opnsense.send_wol
data:
  entity_id: binary_sensor.opnsense_localdomain_pending_notices_present
  interface: lan
  mac: "B9:7B:A6:46:B3:8B"
```

# Known Issues
