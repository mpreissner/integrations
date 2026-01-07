# UniFi Integration

An Elastic Agent integration for Ubiquiti UniFi products with automatic device type detection and ECS field mapping.

## Overview

This integration collects and parses logs from different UniFi device types:

- **Access Points** (U6/U7/UAP models)
- **Switches** (USW/US-/USP-PDU models)
- **Gateways** (System/service logs)
- **Firewalls** (Network traffic logs)
- **Consoles** (UniFi OS CEF format logs)

## Features

- **Automatic Device Detection**: Intelligently identifies device types based on log content
- **ECS Compliance**: Properly maps fields to Elastic Common Schema
- **Syslog Ready**: Designed for remote syslog collection (`/var/log/remote/*/*.log`)
- **Built-in Parsing**: No need to configure custom parsers - works out of the box
- **Error Handling**: Graceful fallback for unknown log formats

## Data Streams

### Syslogs

Collects syslog data from UniFi devices via filestream input with automatic device type detection and parsing.

**Fingerprint Configuration**: This integration uses a fingerprint length of 100 bytes (instead of the standard 1024 bytes) for near-real-time detection of new log files, which is important for high-volume logging scenarios.

## Installation

1. Add the UniFi integration to your Elastic Agent policy via Fleet UI
2. Configure log file paths (defaults to `/var/log/remote/*/*.log`)
3. Deploy the policy to your Elastic Agents

## Configuration

### Required Settings
- **UniFi Log Paths**: Paths to your UniFi log files (supports wildcards)

### Optional Settings
- **Preserve Original Event**: Keep a copy of the raw log message for debugging
- **Tags**: Additional tags to apply to all events

## Log Sources

This integration is designed to work with UniFi devices configured to send logs to a remote syslog server. Common syslog-ng configurations store logs in `/var/log/remote/hostname/*.log` format.

### Supported Log Formats

#### Access Point Logs
```
U6-Lite,U6L-6.6.55-ce15d4b4: hostapd[1234]: STA 00:11:22:33:44:55 IEEE 802.11: authenticated
```

#### Switch Logs
```
USW-24-POE,USW-4.3.21: stp[1234]: MSTP: Port 1 entering forwarding state
```

#### Firewall Logs
```
[WAN_IN-4000] DESCR="Block RFC1918" IN=eth0 OUT= MAC=... SRC=192.168.1.100 DST=10.0.0.1 LEN=60 PROTO=TCP SPT=54321 DPT=80
```

#### Console Logs (CEF)
```
CEF:0|Ubiquiti|UniFi OS|3.2.9|login|User Login|5|msg=User admin logged in from 192.168.1.100
```

## Field Mapping

The integration creates structured data in the `unifi.*` namespace:

- `unifi.device_id`: Device identifier
- `unifi.model`: Device model
- `unifi.firmware`: Firmware version
- `unifi.device_type`: Detected device type
- `unifi.access_point.message`: Access point specific content
- `unifi.switch.message`: Switch specific content
- `unifi.gateway.message`: Gateway specific content
- `unifi.console.*`: Console specific fields (vendor, product, version, etc.)
- `unifi.firewall.*`: Firewall specific fields (rule_id, description)

Standard ECS fields are also populated:
- `observer.*`: Device information
- `process.*`: Process details
- `network.*`: Network connection details (for firewall logs)
- `source.*` / `destination.*`: Network endpoints

## Dashboards

### Overview Dashboard

The integration includes a pre-built Overview Dashboard with the following visualizations:

- **Activity Timeline**: Stacked area chart showing log activity over time broken down by device type
- **Event Type Distribution**: Donut chart displaying the distribution of event actions (firewall allows/denies, DNS queries, DHCP events, VPN connections, etc.)
- **Top Firewall Rules**: Table showing the most frequently triggered firewall rules with hit counts
- **Top Active Devices**: Horizontal bar chart of the devices generating the most log activity

Access the dashboard after installation via **Analytics → Dashboard** and search for "[UniFi] Overview Dashboard".

## Requirements

- Elastic Stack 9.0.8+
- Elastic Agent with Fleet
- UniFi devices configured for remote syslog

## Future Enhancements

- API data stream for metrics collection from UniFi Network Application
- Additional dashboards for security analysis and network health monitoring

## License

Apache 2.0

## Changelog

See [changelog.yml](../changelog.yml) for version history.
