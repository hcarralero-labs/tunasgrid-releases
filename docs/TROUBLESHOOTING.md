# Troubleshooting

| Symptom | Check |
| --- | --- |
| Connection refused | Service running, correct host/port, bind address and firewall; use HTTPS for new TLS-enabled versions. |
| Certificate trust error / PKIX | Import the correct public certificate through TLS setup and confirm its fingerprint. Restart after changing trust. |
| Hostname or SAN mismatch | Connection URL must match a DNS name or IP in the certificate. Do not disable verification. |
| Worker stays unavailable | Matching registered ID, two correct tokens, HTTPS in both directions, certificate trust, reachable worker URL. Heartbeat alone does not make a worker ready. |
| Authentication rejected | Worker tokens are not user passwords. Central-to-worker and worker-to-central tokens must be distinct and in their correct fields. |
| No models in Chat Client | Download and load a model on an available worker through Server Manager. |
| Hugging Face download denied | Check token permissions and accept required model license/access terms on Hugging Face. |
| GPU memory exhausted | Unload other models, reduce GPU layers/context, use a smaller model, or choose CPU execution. |
| SIGILL on older Linux CPU | Use the legacy CPU package when available for your release. |
| TLS setup requires a terminal | Run setup interactively first, or use the explicit unattended hostname and fingerprint options in the TLS guide. |
| Keystore/certificate expired | Follow the certificate replacement process; generated identities are not renewed automatically. |

Logs are under each application's data directory in `logs/`. Server Manager's Logs tab can retrieve supported central/worker logs. See [logging](server/LOGGING.md). When reporting a problem include version, platform/backend, and relevant errors. Do not publish tokens, passwords, private keys, or sensitive prompt data.
