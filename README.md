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
| qryn | `CLICKHOUSE_SERVER` | `hostname` | ClickHouse host (no protocol or port) |
| qryn | `CLICKHOUSE_PORT` | `number` | ClickHouse port (e.g., `8443`) |
| qryn | `CLICKHOUSE_PROTO` | `http` or `https` | ClickHouse protocol |
| qryn | `CLICKHOUSE_AUTH` | `username:password` | ClickHouse credentials |
| qryn | `CLICKHOUSE_DB` | `string` | ClickHouse database name |
| grafana | `GF_SECURITY_ADMIN_PASSWORD` | `string` | Grafana admin password |

Example for `https://clickhouse-db.example.cloud:8443`:
```
CLICKHOUSE_SERVER=clickhouse-db.example.cloud
CLICKHOUSE_PORT=8443
CLICKHOUSE_PROTO=https
CLICKHOUSE_AUTH=user:password
CLICKHOUSE_DB=qryn
```

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

## Span Metrics

The OTEL collector generates R.E.D (Request, Error, Duration) metrics from traces via the spanmetrics connector. These are exported as Prometheus metrics:

| Metric | Description |
|--------|-------------|
| `traces_spanmetrics_calls_total` | Total span count by service/operation |
| `traces_spanmetrics_duration_milliseconds_*` | Span duration histogram |

Query examples in Grafana (Prometheus data source):
```promql
# Span rate by operation
rate(traces_spanmetrics_calls_total{span_name="compute-properties-incremental"}[5m])

# Error rate
rate(traces_spanmetrics_calls_total{status_code="STATUS_CODE_ERROR"}[5m])

# P95 latency
histogram_quantile(0.95, rate(traces_spanmetrics_duration_milliseconds_bucket[5m]))
```
