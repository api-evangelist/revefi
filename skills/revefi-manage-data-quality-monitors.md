---
name: Manage Revefi custom data quality monitors
description: Create, inspect, update, run, and delete custom data quality monitors on a warehouse table via the Revefi Public API.
api: openapi/revefi-data-quality-monitors-openapi-original.yml
operations: [getMonitorByArtifact, createMonitor, getMonitors, updateMonitor, runMonitors, deleteMonitor]
---

# Manage Revefi custom data quality monitors

Operate on Revefi custom data quality monitors for a warehouse table (an
"artifact" = databaseName + schemaName + tableName). Base URL:
`https://gateway.revefi.com/api/v1/`.

## Authentication
- Mint an API access token in the Revefi app: enable "API access" in settings and
  hold the **API User** role, then create a token (save it — shown once).
- Send it on every request: `Authorization: Bearer <token>`.
- Writes require a **READ_WRITE** token; a READ token gets `403`.

## Steps
1. **Find existing monitors for a table** — call `getMonitorByArtifact` with
   `databaseName`, `schemaName`, `tableName` (plus optional `limit`/`offset`).
2. **Create a monitor** — call `createMonitor` with a `Monitor` body: the target
   artifact, the `SqlSpecName` (e.g. `COLUMN_NULL_COUNT`, `CUSTOM_SQL`), the
   `SqlMetricEvaluationType` (static threshold vs automatic anomaly detection),
   `CollectionFrequency` (`HOURLY`/`DAILY`), and — for static thresholds — a
   `StaticThreshold` (comparison op + value type + limits). Returns `201`.
3. **Look up monitors by id** — call `getMonitors` with `monitorIds[]`
   (`limit` max 50, default 10).
4. **Update a monitor** — call `updateMonitor` with a `MonitorUpdate` body.
5. **Run monitors on demand** — call `runMonitors` with a `RunMonitorsRequest`
   (artifacts + optional monitor ids). **Rate-limited to 10 requests/hour** —
   back off on `429`.
6. **Delete a monitor** — call `deleteMonitor` with the `monitorId` query param.

## Conventions & error handling
- Pagination is `limit`/`offset` on the GET operations.
- No idempotency key — do not blindly retry writes.
- Errors are HTTP-status only: `400` validation, `401` bad/missing token,
  `403` insufficient permission, `404` artifact/monitor not found, `429` rate
  limit. See `errors/revefi-problem-types.yml`.
