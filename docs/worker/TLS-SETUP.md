# First-run TLS wizard

TLS setup runs before connections, login, heartbeat, or model initialization. Server and worker use a terminal wizard; Manager and Chat Client use Swing dialogs. Cancellation stops startup. For background services, complete setup interactively first or use the explicit unattended options below.

## Recommended order

1. In the central project run `./run.sh --config /absolute/path/server.properties --tls-setup true`. Enter comma-separated names/IP addresses used to reach central (for example `central.example.com,192.168.68.140,localhost,127.0.0.1`). Do not enter URLs, ports, or wildcard bind addresses. Leave the trust import path blank for now. Setup exits without starting the server.
2. Central prints its SHA-256 fingerprint. Transfer ONLY `<central-data-dir>/tls/identity/certificate.pem` to workers and desktop machines through your trusted administrator channel.
3. On each worker run the equivalent `--tls-setup true` command with its own `--config` / `--data-dir`. Enter that worker's reachable names/IPs. Import central's public certificate and enter the full expected fingerprint obtained from the central administrator. The worker generates its own identity and exports its public certificate.
4. Run central's `--tls-setup true` again to import each worker certificate and confirm each fingerprint. Existing identities are reused, not overwritten.
5. Configure each worker's ID, central HTTPS URL and two token files as described in WORKER-SECURITY.md. Start central and workers normally. Ports do not change automatically: defaults remain central 8080 and worker 8081, now using HTTPS.
6. Launch Manager/Chat Client. Their first-run trust dialog accepts central's public certificate and requires confirmation of its SHA-256 fingerprint. Connect using an HTTPS URL matching the certificate. Register workers through Manager with their matching IDs, HTTPS URLs and two tokens.

An imported certificate is a trust anchor, not a bypass of hostname verification. Import only certificates supplied by your administrator. Public CA certificates from the JVM remain trusted, so Hugging Face and other public HTTPS services still work. Unknown remote certificates are never fetched and silently trusted. A certificate signed by an already-trusted public CA needs no manual import: choose an empty path to retain public CA trust only.

## Generated files

Under each application's data directory:

- `tls/identity/identity.p12`: server private key and self-signed certificate (servers only).
- `tls/identity/password`: random private keystore password, loaded automatically.
- `tls/identity/certificate.pem`: public certificate for export and trust approval.
- `tls/trusted.p12`: administrator-approved certificates, with no private keys. Its fixed integrity password is not an authentication credential; filesystem permissions protect this file.
- `tls/configured`: completed setup marker.

Private files use owner-only POSIX permissions or Windows ACLs. Two instances must use separate data directories. Never share identity.p12 or its password with peers. Keep secure backups; do not commit TLS state to Git.

## Existing CA-issued certificates

Existing `tlsKeystore` and `tlsPasswordFile` settings take precedence. The wizard reuses that identity rather than generating another. Export the public certificate or issuing CA certificate using your existing certificate-management process. Configure trust through the same wizard. No custom JVM truststore flags are required.

## Unattended setup

Example (substitute paths, addresses and fingerprint):

```bash
./run.sh --data-dir /absolute/path/worker-a --tls-setup true \
  --tls-hosts worker-a.example.com,192.168.68.141 \
  --tls-trust-file /absolute/path/central.pem \
  --tls-trust-sha256 EXPECTED_SHA256_FINGERPRINT
```

When no private certificate needs to be trusted, use `--tls-public-ca-only true` instead of the two trust-import flags. This preserves existing approvals too; it never trusts an unknown certificate. Names and fingerprints are not secrets. `tlsHosts`, `tlsTrustFile`, `tlsTrustSha256`, `tlsPublicCaOnly`, and `tlsSetup` also work in properties files, with corresponding `TUNASGRID_TLS_*` environment variables. Remove `tlsSetup=true` from normal service configuration: it intentionally exits after setup.

The wizard uses keytool from the current Java runtime. New Linux/Windows packages retain it; when running JARs directly, use a JDK 21 containing keytool. Older stripped runtimes need replacement or an explicitly configured keystore.

## Later changes and expiration

Run with `--tls-setup true` to add trusted certificates; then restart normally. Previously created HTTP clients are never switched to insecure verification. Generated identities last 365 days, and servers warn within 30 days of expiration. Expired identities stop startup. Automatic renewal is not implemented.

For replacement, prepare a new keystore with matching SANs, distribute and approve its new public certificate before switching `tlsKeystore` / `tlsPasswordFile`, then restart. Remove obsolete trust entries with keytool (the alias is the SHA-256 fingerprint without separators, lowercase; truststore password is `tunasgrid-public-certificates`) and restart. For CA-issued identities, renew through your CA. Preserve public trust approvals until the replacement is deployed on all affected instances.

Changing the connection hostname/IP requires a certificate covering the new address. Certificate approval does not replace the per-worker tokens or user authentication.
