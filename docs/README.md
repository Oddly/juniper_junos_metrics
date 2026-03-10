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
    "system": {
        "cpu": {
            "utilization": 0.02
        },
        "memory": {
            "utilization": 0.56
        }
    },
    "event": {
        "ingested": "2026-03-10T08:15:55.440911974Z",
        "dataset": "juniper_junos_metrics.route_engine",
        "kind": "metric",
        "module": "juniper_junos_metrics"
    },
    "juniper": {
        "junos": {
            "route_engine": {
                "start_time": "2026-03-10T07:40:35Z",
                "memory": {
                    "buffer_utilization": 56.0,
                    "total": 1920991232
                },
                "load_average": {
                    "five_minute": 0.65,
                    "fifteen_minute": 0.44,
                    "one_minute": 0.47
                },
                "last_reboot_reason": "Router rebooted after a normal shutdown.",
                "mastership_state": "master",
                "cpu": {
                    "interrupt": 0.0,
                    "idle": 98.0,
                    "user": 1.0,
                    "background": 0.0,
                    "kernel": 1.0
                },
                "model": "RE-VMX",
                "slot": "0",
                "status": "OK",
                "uptime": 533
            }
        }
    },
    "@timestamp": "2026-03-10T08:00:00.000Z",
    "data_stream": {
        "type": "metrics",
        "dataset": "juniper_junos_metrics.route_engine",
        "namespace": "default"
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
| juniper.junos.route_engine.cpu.background | CPU background utilization percentage. | float | percent | gauge |
| juniper.junos.route_engine.cpu.idle | CPU idle percentage. | float | percent | gauge |
| juniper.junos.route_engine.cpu.interrupt | CPU interrupt utilization percentage. | float | percent | gauge |
| juniper.junos.route_engine.cpu.kernel | CPU kernel utilization percentage. | float | percent | gauge |
| juniper.junos.route_engine.cpu.user | CPU user utilization percentage. | float | percent | gauge |
| juniper.junos.route_engine.last_reboot_reason | Reason for the last reboot. | keyword |  |  |
| juniper.junos.route_engine.load_average.fifteen_minute | Fifteen-minute load average. | float |  | gauge |
| juniper.junos.route_engine.load_average.five_minute | Five-minute load average. | float |  | gauge |
| juniper.junos.route_engine.load_average.one_minute | One-minute load average. | float |  | gauge |
| juniper.junos.route_engine.mastership_state | Route engine mastership state (master or backup). | keyword |  |  |
| juniper.junos.route_engine.memory.buffer_utilization | Memory buffer utilization percentage. | float | percent | gauge |
| juniper.junos.route_engine.memory.total | Total memory in bytes. | long | byte | gauge |
| juniper.junos.route_engine.model | Route engine model. | keyword |  |  |
| juniper.junos.route_engine.slot | Route engine slot number. | keyword |  |  |
| juniper.junos.route_engine.start_time | Route engine start time. | date |  |  |
| juniper.junos.route_engine.status | Route engine status. | keyword |  |  |
| juniper.junos.route_engine.temperature.cpu | CPU temperature in Celsius. | float |  | gauge |
| juniper.junos.route_engine.temperature.routing_engine | Route engine temperature in Celsius. | float |  | gauge |
| juniper.junos.route_engine.uptime | Route engine uptime in seconds. | long | s | gauge |
| system.cpu.utilization | CPU utilization as a fraction from 0 to 1, mapped from 1 minus cpu.idle divided by 100. | float |  | gauge |
| system.memory.utilization | Memory utilization as a fraction from 0 to 1, mapped from memory.buffer_utilization divided by 100. | float |  | gauge |


### Interfaces

The `interfaces` data stream collects per-interface traffic and error metrics. Each polling interval produces one event per physical interface. Fields include byte and packet counters, rate gauges, error counts, and drop counts. OTel-aligned `system.network.*` fields aggregate the per-direction counters.

An example event for `interfaces` looks as following:

```json
{
    "system": {
        "network": {
            "io": 0
        }
    },
    "event": {
        "ingested": "2026-03-10T08:15:55.668245718Z",
        "dataset": "juniper_junos_metrics.interfaces",
        "kind": "metric",
        "module": "juniper_junos_metrics"
    },
    "juniper": {
        "junos": {
            "interfaces": {
                "link_type": "Full-Duplex",
                "mac_address": "bc:24:11:c4:0f:18",
                "snmp_index": 1,
                "name": "fxp0",
                "admin_status": "up",
                "type": "Ethernet",
                "oper_status": "up",
                "speed": "10Gbps",
                "mtu": 1514,
                "traffic": {
                    "output": {
                        "packets": 379
                    },
                    "input": {
                        "packets": 140
                    }
                }
            }
        }
    },
    "@timestamp": "2026-03-10T08:00:00.000Z",
    "data_stream": {
        "type": "metrics",
        "dataset": "juniper_junos_metrics.interfaces",
        "namespace": "default"
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
| juniper.junos.interfaces.admin_status | Administrative status of the interface. | keyword |  |  |
| juniper.junos.interfaces.description | Interface description. | keyword |  |  |
| juniper.junos.interfaces.errors.input | Total input errors on the interface. | long |  | counter |
| juniper.junos.interfaces.errors.input_drops | Total input drops on the interface. | long |  | counter |
| juniper.junos.interfaces.errors.output | Total output errors on the interface. | long |  | counter |
| juniper.junos.interfaces.errors.output_drops | Total output drops on the interface. | long |  | counter |
| juniper.junos.interfaces.link_type | Link type (for example, Full-Duplex). | keyword |  |  |
| juniper.junos.interfaces.mac_address | MAC address of the interface. | keyword |  |  |
| juniper.junos.interfaces.mtu | Maximum transmission unit in bytes. | long | byte | gauge |
| juniper.junos.interfaces.name | Interface name (for example, ge-0/0/0). | keyword |  |  |
| juniper.junos.interfaces.oper_status | Operational status of the interface. | keyword |  |  |
| juniper.junos.interfaces.snmp_index | SNMP interface index. | long |  | gauge |
| juniper.junos.interfaces.speed | Interface speed string. | keyword |  |  |
| juniper.junos.interfaces.traffic.input.bps | Input rate in bits per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.input.bytes | Total bytes received on the interface. | long | byte | counter |
| juniper.junos.interfaces.traffic.input.packets | Total packets received on the interface. | long |  | counter |
| juniper.junos.interfaces.traffic.input.pps | Input rate in packets per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.output.bps | Output rate in bits per second. | long |  | gauge |
| juniper.junos.interfaces.traffic.output.bytes | Total bytes transmitted on the interface. | long | byte | counter |
| juniper.junos.interfaces.traffic.output.packets | Total packets transmitted on the interface. | long |  | counter |
| juniper.junos.interfaces.traffic.output.pps | Output rate in packets per second. | long |  | gauge |
| juniper.junos.interfaces.type | Interface type (for example, Ethernet). | keyword |  |  |
| system.network.errors | Network errors, split by direction. | long |  | counter |
| system.network.io | Network I/O in bytes, split by network.io.direction attribute. | long | byte | counter |
| system.network.packet.dropped | Dropped packets, split by direction. | long |  | counter |


## Setup

1. Enable the REST API on the JunOS device: `set system services rest https port 3443`.
2. Create a local user account or use an existing one with at least read-only access.
3. In Kibana, navigate to **Integrations** and search for "Juniper JunOS Metrics".
4. Add the integration and configure the device host (for example, `https://192.168.1.1:3443`), username, and password.
5. Set the SSL verification mode to `none` if the device uses a self-signed certificate, or to `full` if a trusted certificate is installed.
6. Deploy the policy to an Elastic Agent that has network access to the JunOS device.
