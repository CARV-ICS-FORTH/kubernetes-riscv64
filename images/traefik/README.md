# Traefik image for RISC-V

This is a simple Dockerfile for building the Traefik container for the RISC-V architecture.

To build Traefik:
* Comment out all operating systems and architectures except 'linux' and 'riscv64' in `.goreleaser.yml`.
* Add the `--skip-validate` flag in the `goreleaser` command in `Makefile`.
* Run `CODENAME=saintmarcelin make release-packages`.

Then copy over 'dist/traefik_linux_riscv64/traefik' here.

Build and push with:
```bash
docker buildx build --platform linux/riscv64 -t carvicsforth/traefik:2.10.3 --push .
```