FROM riscv64/ubuntu:latest AS builder

RUN apt-get update && \
    apt-get install -y wget gcc && \
    wget https://raw.githubusercontent.com/kubernetes/kubernetes/v1.27.2/build/pause/linux/pause.c && \
    gcc -Os -Wall -Werror -static -DVERSION=v3.9-v1.27.2 -o pause pause.c

FROM scratch
COPY --from=builder /pause /pause
USER 65535:65535
ENTRYPOINT ["/pause"]
