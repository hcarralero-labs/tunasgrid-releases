# TunasGrid log management

## Locations and retention

Every app writes structured UTF-8 JSON Lines to `<data-dir>/logs/events.jsonl`.
Older files are `events.1.jsonl`, `events.2.jsonl`, etc. Data defaults are:

- Central: `~/.tunasgrid/server/logs/`
- Worker: `~/.tunasgrid/model-server/logs/` (or that worker's `--data-dir`)
- Manager: `~/.tunasgrid/server-manager/logs/`
- Chat client: `~/.tunasgrid/chat-client/logs/`

Logging starts automatically. Restart the updated applications to enable it.
Existing console output is not imported. A log-directory lock prevents multiple
processes from rotating the same files: use different `--data-dir` values for
concurrent instances, including multiple chat clients or Managers on one machine.
Log files are created with owner-only read/write permissions.

Defaults: INFO minimum severity, 5 MiB per file, 5 files total (active included),
and 14-day retention. Rotation also occurs on the first write after a UTC date
change. File-count limits may retain fewer than 14 days. Files whose last write is
older than retention are deleted during startup, writes, or reads; this does not
require a running background collector. If the app is stopped, cleanup waits for
its next startup. Disk errors produce a console warning instead of aborting a
request; monitor disk capacity so diagnostics are not lost.

Configure each instance using properties, corresponding flags, or environment:

```properties
logLevel=INFO
logMaxBytes=5242880
logFiles=5
logRetentionDays=14
```

Flags: `--log-level`, `--log-max-bytes`, `--log-files`, `--log-retention-days`.
Environment: `TUNASGRID_LOG_LEVEL`, `TUNASGRID_LOG_MAX_BYTES`, `TUNASGRID_LOG_FILES`,
`TUNASGRID_LOG_RETENTION_DAYS`. Normal CLI > environment > config > default precedence
applies. Allowed ranges are 64 KiB–10 MiB, 1–10 files, 1–3650 days, and severity
DEBUG/INFO/WARN/ERROR. DEBUG includes routine health/presence polling; INFO avoids
that polling noise. WARN/ERROR configurations intentionally omit lower-severity
audit records, so retain INFO for the normal administrative audit trail.

## Server Manager

After administrator login, open **Logs**:

1. Select **Central server**, a registered worker, or **This Manager**.
2. Choose minimum severity, optional Since/Until UTC timestamps (ISO 8601, e.g.
   `2026-10-08T12:00:00Z`), exact model ID, or exact request UUID.
3. Click **Fetch logs**. Results are newest first; select a row for full metadata.

The default result limit is 500, maximum 2000. If results are truncated, narrow the
time window or increase the limit. Only retained entries are searchable. Retrieval
is manual and bounded, not a continuously streaming remote log collector. The
worker list updates with central registration state. An offline worker's files
remain locally available, but remote retrieval requires that worker to respond.
Chat-client logs remain local on each client machine; clients do not expose a log
server. Logout clears the Manager's log display and rejects stale fetch results.

## Events and correlation

Central records login/logout and administrative request outcomes, routing target,
HTTP result, duration, worker availability transitions, catalog synchronization,
and observed worker download/verification states. Changes to users and workers
include actor and target identifiers when available. Sessions expiring due to idle
policy produce authentication failures on subsequent requests.

Workers record model load/unload results, GPU-layer settings, file-operation
progress (roughly 10% increments), completion/failure, request results, and safe
exception types and Java stack locations. Files stay local during disconnection.
Central also records worker job-state changes it observes during polling; it is
not a durable replay of every event generated while the worker was offline.

A central request receives a UUID in `X-TunasGrid-Request-Id`. Central forwards it to
the worker; worker events and background file jobs keep that ID. Desktop HTTP
results include the same ID when the response supplies it. Use the Request ID
filter on each source to trace a request. Worker ID, model ID, username, component,
instance, status, and duration appear where applicable. Native runtime callbacks
may run outside a request thread and therefore have instance-level correlation.

Native llama.cpp warnings/errors are captured as diagnostic categories such as
MEMORY_ALLOCATION, GPU_BACKEND, MODEL_ARCHITECTURE and TOKEN_METADATA. Raw native
messages are deliberately not stored: they may include prompt or token content.
This provides safe operational diagnostics, not a verbatim native debug trace.
Exception messages and HTTP request/response bodies are likewise excluded.

Passwords, session tokens, service keys, HF tokens, Authorization headers, prompts,
and generated responses are not recorded by these structured log writers. No
switch enables prompt/body logging. Logs still contain operational identities and
paths in Java stack locations, so treat them as administrator-only information.
Logs are local rotating diagnostics, not a tamper-proof compliance archive.

## HTTP API

- Administrator session: `GET /admin/logs` for central logs.
- Administrator session: `GET /admin/logs?worker=worker-a` proxies worker retrieval.
- Internal worker endpoint: `GET /internal/logs`, authenticated by its service key.

Optional query parameters: `level` (ALL or a minimum severity), `since`, `until`,
`model`, `requestId`, `limit`. Response contains `entries`, `truncated`, and `limit`.
No endpoint accepts filenames or filesystem paths. Central enforces administrator
role before either local or worker log access; workers require the private service
key. Ordinary clients cannot retrieve server logs. Use the existing HTTPS setup
for remote access.
