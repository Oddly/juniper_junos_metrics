# Juniper JunOS Metrics Integration

This integration collects metrics from Juniper JunOS devices using the REST API. It polls the JunOS RPC endpoints to gather route engine health, CPU, memory, load averages, and interface traffic and error statistics. The integration uses CEL input to call the JunOS REST API and produces TSDB-compatible time series data.

## Compatibility

This integration has been tested with JunOS 21.x and later. It requires a JunOS device with the REST API (HTTPS) enabled. Enable the REST API with `set system services rest https port 3443` in JunOS configuration mode.

## Data Streams

### Route Engine

The `route_engine` data stream collects per-slot route engine metrics including CPU utilization, memory buffer usage, temperature, load averages, and uptime. Each polling interval produces one event per route engine slot. OTel-aligned `system.cpu.utilization` and `system.memory.utilization` fields are computed from the native JunOS values.

{{event "route_engine"}}

**ECS Field Reference**

Refer to the following [document](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) for detailed information on ECS fields.

{{fields "route_engine"}}

### Interfaces

The `interfaces` data stream collects per-interface traffic and error metrics. Each polling interval produces one event per physical interface. Fields include byte and packet counters, rate gauges, error counts, and drop counts. OTel-aligned `system.network.*` fields aggregate the per-direction counters.

{{event "interfaces"}}

**ECS Field Reference**

Refer to the following [document](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) for detailed information on ECS fields.

{{fields "interfaces"}}

## Setup

1. Enable the REST API on the JunOS device: `set system services rest https port 3443`.
2. Create a local user account or use an existing one with at least read-only access.
3. In Kibana, navigate to **Integrations** and search for "Juniper JunOS Metrics".
4. Add the integration and configure the device host (for example, `https://192.168.1.1:3443`), username, and password.
5. Set the SSL verification mode to `none` if the device uses a self-signed certificate, or to `full` if a trusted certificate is installed.
6. Deploy the policy to an Elastic Agent that has network access to the JunOS device.
