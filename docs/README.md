# Juniper JunOS Metrics Integration

This integration collects metrics from Juniper JunOS devices using the REST API. It polls the JunOS RPC endpoints to gather route engine health, CPU, memory, load averages, and interface traffic and error statistics. The integration uses CEL input to call the JunOS REST API and produces TSDB-compatible time series data.

## Compatibility

This integration has been tested with JunOS 21.x and later. It requires a JunOS device with the REST API (HTTPS) enabled. Enable the REST API with `set system services rest https port 3443` in JunOS configuration mode.

## Data Streams

### Route Engine

The `route_engine` data stream collects per-slot route engine metrics including CPU utilization, memory buffer usage, temperature, load averages, and uptime. Each polling interval produces one event per route engine slot. OTel-aligned `system.cpu.utilization` and `system.memory.utilization` fields are computed from the native JunOS values.

An example event for `route_engine` looks as following:

```json
{
    "event": {
        "dataset": "juniper_junos_metrics.route_engine",
        "kind": "metric",
        "module": "juniper_junos_metrics"
    },
    "host": {
        "uptime": 11690
    },
    "juniper": {
        "junos": {
            "route_engine": {
                "cpu": {
                    "background": 0.0,
                    "idle": 89.0,
                    "interrupt": 0.0,
                    "kernel": 5.0,
                    "user": 6.0
                },
                "last_reboot_reason": "Router rebooted after a normal shutdown.",
                "load_average": {
                    "fifteen_minute": 0.58,
                    "five_minute": 0.47,
                    "one_minute": 0.49
                },
                "mastership_state": "master",
                "memory": {
                    "buffer_utilization": 19.0,
                    "total": 1920991232
                },
                "model": "RE-VMX",
                "slot": "0",
                "start_time": "2026-03-10T07:40:35Z",
                "status": "OK",
                "uptime": 11690
            }
        }
    },
    "observer": {
        "product": "JunOS",
        "type": "router",
        "vendor": "Juniper"
    },
    "system": {
        "cpu": {
            "utilization": 0.11
        },
        "memory": {
            "utilization": 0.19
        }
    },
    "data_stream": {
        "type": "metrics",
        "dataset": "juniper_junos_metrics.route_engine",
        "namespace": "default"
    },
    "ecs": {
        "version": "8.0.0"
    },
    "input": {
        "type": "cel"
    }
}
```

**ECS Field Reference**

Refer to the following [document](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) for detailed information on ECS fields.

**Exported fields**

| Field | Description | Type | Unit | Metric Type |
|---|---|---|---|---|
| @timestamp | Event timestamp. | date |  |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |  |
| data_stream.type | Data stream type. | constant_keyword |  |  |
| ecs.version | ECS version this event conforms to. | keyword |  |  |
| error.message | Error message. | match_only_text |  |  |
| event.dataset | Event dataset. | constant_keyword |  |  |
| event.kind | The kind of event. | keyword |  |  |
| event.module | Event module. | constant_keyword |  |  |
| host.uptime | Device uptime in seconds. | long |  | gauge |
| juniper.junos.route_engine.cpu.background | Percentage of CPU time spent on background (nice) processes. | float | percent | gauge |
| juniper.junos.route_engine.cpu.idle | Percentage of CPU time spent idle with no pending work. | float | percent | gauge |
| juniper.junos.route_engine.cpu.interrupt | Percentage of CPU time spent servicing hardware interrupts. | float | percent | gauge |
| juniper.junos.route_engine.cpu.kernel | Percentage of CPU time spent in kernel space. | float | percent | gauge |
| juniper.junos.route_engine.cpu.user | Percentage of CPU time spent executing user-space processes. | float | percent | gauge |
| juniper.junos.route_engine.last_reboot_reason | Reported reason for the most recent route engine reboot. | keyword |  |  |
| juniper.junos.route_engine.load_average.fifteen_minute | System load average over the last 15 minutes. | float |  | gauge |
| juniper.junos.route_engine.load_average.five_minute | System load average over the last 5 minutes. | float |  | gauge |
| juniper.junos.route_engine.load_average.one_minute | System load average over the last 1 minute. | float |  | gauge |
| juniper.junos.route_engine.mastership_state | Redundancy role of the route engine, either master or backup. | keyword |  |  |
| juniper.junos.route_engine.memory.buffer_utilization | Percentage of route engine memory currently in use by buffers and caches. | float | percent | gauge |
| juniper.junos.route_engine.memory.total | Total installed memory available to the route engine in bytes. | long | byte | gauge |
| juniper.junos.route_engine.model | Route engine hardware model string. Used as a TSDB dimension. | keyword |  |  |
| juniper.junos.route_engine.slot | Route engine slot identifier. Used as a TSDB dimension. | keyword |  |  |
| juniper.junos.route_engine.start_time | Timestamp of the route engine's most recent boot. | date |  |  |
| juniper.junos.route_engine.status | Operational status of the route engine (for example OK, Testing, Failed). | keyword |  |  |
| juniper.junos.route_engine.temperature.cpu | Current CPU die temperature in degrees Celsius. | float |  | gauge |
| juniper.junos.route_engine.temperature.routing_engine | Current route engine board temperature in degrees Celsius. | float |  | gauge |
| juniper.junos.route_engine.uptime | Time elapsed since the route engine last booted, in seconds. | long | s | gauge |
| observer.product | Observer product name. | keyword |  |  |
| observer.type | Observer type such as router, switch, or firewall. | keyword |  |  |
| observer.vendor | Observer vendor name. | keyword |  |  |
| system.cpu.utilization | CPU utilization as a fraction from 0 to 1, derived from (1 - cpu.idle / 100). | float |  | gauge |
| system.memory.utilization | Memory utilization as a fraction from 0 to 1, derived from memory.buffer_utilization / 100. | float |  | gauge |


### Interfaces

The `interfaces` data stream collects per-interface traffic and error metrics. Each polling interval produces one event per physical interface. Fields include byte and packet counters, rate gauges, error counts, and drop counts. OTel-aligned `system.network.*` fields aggregate the per-direction counters.

An example event for `interfaces` looks as following:

```json
{
    "event": {
        "dataset": "juniper_junos_metrics.interfaces",
        "kind": "metric",
        "module": "juniper_junos_metrics"
    },
    "juniper": {
        "junos": {
            "interfaces": {
                "admin_status": "up",
                "link_type": "Full-Duplex",
                "mac_address": "52:54:00:12:bd:fe",
                "mtu": 1514,
                "name": "em1",
                "oper_status": "up",
                "snmp_index": 23,
                "speed": "10Gbps",
                "traffic": {
                    "input": {
                        "packets": 90573
                    },
                    "output": {
                        "packets": 123407
                    }
                },
                "type": "Ethernet"
            }
        }
    },
    "observer": {
        "product": "JunOS",
        "type": "router",
        "vendor": "Juniper"
    },
    "data_stream": {
        "type": "metrics",
        "dataset": "juniper_junos_metrics.interfaces",
        "namespace": "default"
    },
    "ecs": {
        "version": "8.0.0"
    },
    "input": {
        "type": "cel"
    }
}
```

**ECS Field Reference**

Refer to the following [document](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) for detailed information on ECS fields.

**Exported fields**

| Field | Description | Type | Unit | Metric Type |
|---|---|---|---|---|
| @timestamp | Event timestamp. | date |  |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |  |
| data_stream.type | Data stream type. | constant_keyword |  |  |
| ecs.version | ECS version this event conforms to. | keyword |  |  |
| error.message | Error message. | match_only_text |  |  |
| event.dataset | Event dataset. | constant_keyword |  |  |
| event.kind | The kind of event. | keyword |  |  |
| event.module | Event module. | constant_keyword |  |  |
| juniper.junos.interfaces.admin_status | Administrative status of the interface (up or down), reflecting the configured state. | keyword |  |  |
| juniper.junos.interfaces.description | Operator-configured description string for the interface. | keyword |  |  |
| juniper.junos.interfaces.errors.input | Cumulative number of inbound packets that contained errors (for example CRC, framing). | long |  | counter |
| juniper.junos.interfaces.errors.input_drops | Cumulative number of inbound packets dropped due to resource constraints or policy. | long |  | counter |
| juniper.junos.interfaces.errors.output | Cumulative number of outbound packets that failed to transmit due to errors. | long |  | counter |
| juniper.junos.interfaces.errors.output_drops | Cumulative number of outbound packets dropped due to queue overflow or policy. | long |  | counter |
| juniper.junos.interfaces.link_type | Physical link mode (for example Full-Duplex, Half-Duplex). | keyword |  |  |
| juniper.junos.interfaces.mac_address | Hardware MAC address assigned to the interface. | keyword |  |  |
| juniper.junos.interfaces.mtu | Maximum transmission unit size configured on the interface, in bytes. | long | byte | gauge |
| juniper.junos.interfaces.name | Interface name (for example ge-0/0/0, ae0, lo0). Used as a TSDB dimension. | keyword |  |  |
| juniper.junos.interfaces.oper_status | Operational status of the interface (up or down), reflecting the actual link state. | keyword |  |  |
| juniper.junos.interfaces.snmp_index | SNMP ifIndex value assigned to this interface for SNMP polling. | long |  | gauge |
| juniper.junos.interfaces.speed | Negotiated or configured interface speed (for example 1000mbps, 10Gbps, Auto). | keyword |  |  |
| juniper.junos.interfaces.traffic.input.bps | Current inbound traffic rate in bits per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.input.bytes | Cumulative number of bytes received on the interface. | long | byte | counter |
| juniper.junos.interfaces.traffic.input.packets | Cumulative number of packets received on the interface. | long |  | counter |
| juniper.junos.interfaces.traffic.input.pps | Current inbound traffic rate in packets per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.output.bps | Current outbound traffic rate in bits per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.output.bytes | Cumulative number of bytes transmitted on the interface. | long | byte | counter |
| juniper.junos.interfaces.traffic.output.packets | Cumulative number of packets transmitted on the interface. | long |  | counter |
| juniper.junos.interfaces.traffic.output.pps | Current outbound traffic rate in packets per second. | long |  | gauge |
| juniper.junos.interfaces.type | Interface media type (for example Ethernet, SONET, Loopback). | keyword |  |  |
| observer.product | Observer product name. | keyword |  |  |
| observer.type | Observer type such as router, switch, or firewall. | keyword |  |  |
| observer.vendor | Observer vendor name. | keyword |  |  |


## Setup

### JunOS device configuration

Create a read-only login class and user with the minimum permissions needed for this integration:

```
# Create a login class with only view permissions and REST API access
set system login class elastic-monitor permissions view
set system login class elastic-monitor allow-commands "show.*"
set system login class elastic-monitor deny-commands "configure|edit|set|delete|rollback|commit|request|start|restart|clear|file"

# Create the user
set system login user elastic-agent class elastic-monitor
set system login user elastic-agent authentication plain-text-password
# (enter password at prompt)

# Enable the REST API
set system services rest http port 8080
# Or for HTTPS:
# set system services rest https port 3443
# set system services rest https default-certificate

# Commit
commit
```

The `view` permission is sufficient because all 8 RPC endpoints this integration calls (`get-route-engine-information`, `get-interface-information`, `get-bgp-summary-information`, `get-ospf-overview-information`, `get-route-summary-information`, `get-system-storage`, `get-environment-information`, `get-alarm-information`) are read-only `show` equivalents. No `configure`, `edit`, or `request` permissions are needed.

### Elastic Agent configuration

1. In Kibana, navigate to **Integrations** and search for "Juniper JunOS Metrics".
2. Add the integration and configure the device host (for example, `192.168.1.1:3443`), username, and password.
3. Enable the **Use HTTPS** toggle if the device serves the REST API over TLS.
4. Set the SSL verification mode to `none` if the device uses a self-signed certificate, or to `full` if a trusted certificate is installed.
5. Deploy the policy to an Elastic Agent that has network access to the JunOS device.
