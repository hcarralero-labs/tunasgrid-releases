# Quick start

## Choose packages

Download the central server, Server Manager, Chat Client, and a model worker for each inference machine. Select the correct operating system and worker backend: CPU, Linux legacy CPU for older processors, or CUDA for a compatible NVIDIA GPU/driver. Archives include Java; model weights download separately from Hugging Face and have their own access and license requirements.

Verify the accompanying SHA-256 checksum and extract each complete archive. Keep its runtime and supporting libraries together. On Linux run the executable named after the application; on Windows run its `.exe`. Pass `--config /absolute/path/app.properties` to load a configuration file. Each application includes a config example.

## Configure TLS before starting the grid

For versions with the TLS wizard:

1. Run the central executable with `--tls-setup true`. Enter its reachable DNS names/IP addresses and export the displayed public certificate.
2. Run each worker with `--tls-setup true` and its own data directory. Import the central certificate and confirm its fingerprint using information from the central administrator.
3. Run central's setup again to import each worker's public certificate and confirm its fingerprint.
4. Configure each worker's ID, central HTTPS URL, and two distinct token files. Use Server Manager to register the same ID, HTTPS URL, and tokens after starting the services.
5. Import central's public certificate during Manager and Chat Client setup.

Read the [complete TLS guide](server/TLS-SETUP.md) before setup. Keep private keys and password files on their owning machines. Only public certificates are exchanged.

## Startup order

1. Start the central server. On a new installation, use its initial administrator credential according to the server startup instructions; protect the generated `admin-initial-password.txt` in its data directory.
2. Start workers. They retry authenticated heartbeats while waiting for registration.
3. Open Server Manager, connect to the central HTTPS URL, log in as administrator, and register workers. See [worker registration](server/WORKER-SECURITY.md).
4. Select a worker in Catalog, download a model, and load it. Complete Hugging Face access/license requirements when applicable.
5. Open Chat Client, connect to central, log in, and select a loaded model.

Clients and Manager connect to central, not directly to workers. Central routes inference and administration requests. Every worker owns its model repository; the catalog and users are centrally managed.

Defaults: central port 8080, worker port 8081. Separate workers on one machine need distinct ports, IDs, tokens and data directories. Services bind to loopback by default; configure a reachable bind address for remote machines and permit the required traffic through the firewall. Never use `0.0.0.0` as a client connection URL.

## Upgrade

Back up configuration, credentials, and application data. Stop applications before replacing binaries. Retain each instance's data directory and model repository. Upgrade central, workers, Manager, and Client together when the release changes authentication. Existing single-key deployments require migration before using the two-token versions.
