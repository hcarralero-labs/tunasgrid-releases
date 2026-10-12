> **First-run TLS setup:** See [TLS-SETUP.md](TLS-SETUP.md). The wizard creates server identities and manages approved trust automatically; manual keystore/JVM settings below are optional alternatives.

# Worker authentication: two tokens per worker

Worker registration is administrator-only through Server Manager. Workers never hold central administrator passwords and cannot register themselves.

## Generate credentials

Run once for each worker, in a private directory. Do not overwrite existing token files during routine startup:

```bash
umask 077
mkdir -p "$HOME/.tunasgrid/secrets"
openssl rand -hex 32 > "$HOME/.tunasgrid/secrets/worker-a-central.token"
openssl rand -hex 32 > "$HOME/.tunasgrid/secrets/worker-a-worker.token"
```

Use two different tokens, unique to this worker. The central token authorizes central → worker requests. The worker token authorizes only worker → central heartbeats, not user/admin APIs. Tokens are case-sensitive; copy them exactly.

## Configure the worker

```properties
workerId=worker-a
centralUrl=https://central.example:8443
centralTokenFile=/absolute/path/secrets/worker-a-central.token
workerTokenFile=/absolute/path/secrets/worker-a-worker.token
tlsKeystore=/absolute/path/worker.p12
tlsPasswordFile=/absolute/path/secrets/tls-password
host=0.0.0.0
port=8444
```

Flags: `--central-token-file`, `--worker-token-file`, `--worker-id`, `--central-url`.
Environment: `TUNASGRID_CENTRAL_TOKEN_FILE`, `TUNASGRID_WORKER_TOKEN_FILE`, `TUNASGRID_WORKER_ID`, `TUNASGRID_CENTRAL_URL`. Direct token environment variables are supported but protected files are preferred. Plain token properties are rejected.

Verified HTTPS is mandatory for inter-server traffic, including loopback, and for worker administration. Configure the central TLS keystore too. Each JVM must trust the remote endpoint's certificate, using its JVM truststore (for private CAs, configure `javax.net.ssl.trustStore` with the trusted CA certificates). Certificates must match the endpoint hostname/IP. Do not disable verification. No mTLS/client certificates are needed. TLS certificates and the application tokens serve different purposes.

## Register and activate

Connect Server Manager to central over HTTPS, log in as administrator, and choose Model servers → Register worker. Enter the worker ID, HTTPS worker URL, and both tokens matching its configuration. A new record remains unavailable until an authenticated heartbeat and a successful central → worker health/identity check occur. Central then synchronizes the catalog and enables routing.

Heartbeats run every 10 seconds; routing stops when the last authenticated heartbeat is 45 seconds old, or existing health freshness checks fail. A heartbeat never creates a record or changes its URL. Wrong, missing, removed, or direction-swapped credentials are rejected. Embedded model endpoints remain loopback-only.

Edit worker allows changing URL and either token; blank token fields preserve current values. Restart the worker with matching updated token files. Edits reset readiness until both directions authenticate again. Removing the registration rejects subsequent heartbeats and prevents new routing; it does not erase the worker's configured inbound credential. Stop the worker or rotate its central token to revoke that credential. Already executing requests are not retroactively cancelled.

Central stores the outbound central token in its owner-restricted workers.json and only a SHA-256 digest of the inbound worker token. Hashing is appropriate here because tokens have 256 bits of random entropy. Public status and logs do not include credentials. Back up credential files securely.

## Existing installations

Upgrade central, worker, and manager together. Old registry rows load for migration but remain unavailable without the new token and heartbeat. Configure HTTPS, generate two new tokens, update worker configuration, and edit the registration in Manager. Remove old `workerKeyFile`, `centralUsername`, `centralPasswordFile`, and `workerUrl` enrollment settings; they no longer enroll workers. Existing model files and user accounts are retained.
