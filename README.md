# Render Telemetry Stack

Render Blueprint for deploying a telemetry stack with qryn, Grafana, and OpenTelemetry Collector.

## Services

| Service | Type | Description |
|---------|------|-------------|
| qryn | Private | Polyglot observability backend (Loki/Prometheus/Tempo compatible) |
| otel-collector | Private | Receives OTLP and forwards to qryn |
| grafana | Web | Visualization dashboard with persistent storage |

## Configuration

### Required Environment Variables

Set these in the Render Dashboard after deploying:

| Service | Variable | Format | Description |
|---------|----------|--------|-------------|
| qryn | `CLICKHOUSE_SERVER` | `hostname` | ClickHouse server address |
| qryn | `CLICKHOUSE_AUTH` | `username:password` | ClickHouse credentials |
| grafana | `GF_SECURITY_ADMIN_PASSWORD` | `string` | Grafana admin password |

### Default Values

| Service | Variable | Default |
|---------|----------|---------|
| qryn | `CLICKHOUSE_PORT` | `8123` |
| qryn | `CLICKHOUSE_PROTO` | `https` |
| qryn | `CLICKHOUSE_DB` | `qryn` |

## Endpoints

Once deployed:

- **OTLP gRPC**: `otel-collector:4317` (internal)
- **OTLP HTTP**: `otel-collector:4318` (internal)
- **qryn API**: `qryn:3100` (internal)
- **Grafana**: Public URL provided by Render

## Grafana Data Sources

Configure these data sources in Grafana to query qryn:

| Type | URL |
|------|-----|
| Loki (logs) | `http://qryn:3100` |
| Tempo (traces) | `http://qryn:3100` |
| Prometheus (metrics) | `http://qryn:3100` |
