# Pause image for RISC-V

This is a simple Dockerfile for building the Kubernetes pause container for the RISC-V architecture.

Build and push with:
```bash
docker buildx build --platform linux/riscv64 -t carvicsforth/pause:v3.10-v1.31.1 --push .
```
