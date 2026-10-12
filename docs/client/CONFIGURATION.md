> **First-run TLS setup:** See [TLS-SETUP.md](TLS-SETUP.md). The wizard creates server identities and manages approved trust automatically; manual keystore/JVM settings below are optional alternatives.

# Configuration files and worker authentication

Every app accepts `--config FILE` or `--config=FILE`; `TUNASGRID_CONFIG` is an alternative.
Files use UTF-8 Java `.properties` syntax. No config file is loaded implicitly.
Launch from the relevant project, for example:

```bash
./run.sh --config config/server.properties
./run.sh --config config/worker.properties --port 8082
./run.sh --config config/manager.properties
./run.sh --config config/client.properties
```

Each project includes its corresponding example. Copy and edit it for each instance.
Order: **command line → environment → config file → default**. Desktop connection
preferences supply fallback URL/username when none is explicitly configured.
Paths in both config files and flags resolve against the launch working directory;
use absolute paths for deployments. `~` and environment variables inside properties
values are not expanded. Quote paths with spaces on the command line, not inside
the properties file.

## Settings

Config uses camelCase; flags use kebab-case. Examples: `dataDir` → `--data-dir`,
`contextTokens` → `--context-tokens`. Existing flags and environment variables still
work. Both `--name value` and `--name=value` are accepted; options require a value.

| App | Supported configuration properties |
| --- | --- |
| All | `dataDir` |
| Central | `host`, `port`, `modelId`, `catalog`, `usersFile`, `sessionIdleMinutes`, `adminPasswordFile`, `tlsKeystore`, `tlsPasswordFile` |
| Worker | `host`, `port`, `modelId`, `modelsDir`, `catalog`, `gpuLayers`, `contextTokens`, `maxModels`, `centralTokenFile`, `workerTokenFile`, `tlsKeystore`, `tlsPasswordFile` |
| Manager/client | `serverUrl`, `username`, `passwordFile`, `idleMinutes` |
| Worker identity | `centralUrl`, `workerId` |

Use each app's existing data environment variable, or `TUNASGRID_DATA_DIR`. Ordinary
settings use their corresponding `TUNASGRID_*` environment variable, e.g.
`TUNASGRID_CONTEXT_TOKENS`, `TUNASGRID_SERVER_URL`, `TUNASGRID_SESSION_IDLE_MINUTES`.
Worker variables are `TUNASGRID_CENTRAL_URL`, `TUNASGRID_WORKER_ID`, `TUNASGRID_CENTRAL_TOKEN_FILE`, and `TUNASGRID_WORKER_TOKEN_FILE`.

Secrets must be referenced by file in properties files. Plain token properties are rejected.
Secret files contain the literal secret (one trailing newline is removed).
Protect them with owner-only permissions, for example `chmod 600 /path/to/secret`.
The old direct-secret environment variables remain supported, as do corresponding
`*_FILE` variables. Within one source, a file reference wins over a direct secret;
command-line settings win over environment settings, which win over config settings.
Avoid direct secrets on the command line because process listings can expose them.
Configuration and startup messages never print resolved secret values.

## Worker registration and authentication

See [two-token worker setup](WORKER-SECURITY.md). Registration is administrator-only; auto-registration has been removed.

## Log management

Rotating logs are stored in `<data-dir>/logs/`. See [log management](LOGGING.md) for retention settings, request correlation, and the administrator Logs tab.
