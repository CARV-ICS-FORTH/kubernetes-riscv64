# Metrics Server image for RISC-V

This is a simple Dockerfile for building the Metrics Server container for the RISC-V architecture.

Build and push with:
```bash
docker buildx build --platform linux/riscv64 -t carvicsforth/metrics-server:v0.7.2 --push .
```
