# TunasGrid releases

Run AI models on your own machines, coordinated by a central server.

## Downloads

[**Download Linux and Windows packages**](https://github.com/hcarralero-labs/tunasgrid-releases/releases)

Archives include Java 21. Extract the complete archive and verify its accompanying SHA-256 checksum. Worker variants support CPU, legacy Linux CPU, or CUDA; GPU use requires a compatible NVIDIA driver. Model weights download separately from Hugging Face and are subject to their own licenses and access requirements.

## Documentation

- [**Documentation index**](docs/README.md)
- [Quick start and startup order](docs/QUICKSTART.md)
- [First-run TLS wizard and certificate trust](docs/server/TLS-SETUP.md)
- [Worker registration and two-token authentication](docs/server/WORKER-SECURITY.md)
- [Configuration reference](docs/README.md#configuration-by-application)
- [Troubleshooting](docs/TROUBLESHOOTING.md)

**Version notice:** Repository guides currently describe the upcoming TLS-wizard/two-token release. Older published packages may behave differently. Use the documentation included with your installed package. Future releases also attach `tunasgrid-docs-VERSION-PLATFORM.zip` with matching instructions and package checksums.

Application source code is maintained privately. All documentation linked here is public; no source repository access is required.
