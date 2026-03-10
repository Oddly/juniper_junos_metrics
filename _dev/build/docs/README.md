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
