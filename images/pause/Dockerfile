FROM ubuntu:latest AS builder

RUN apt-get update && \
    apt-get install -y wget gcc && \
    wget https://raw.githubusercontent.com/kubernetes/kubernetes/v1.31.1/build/pause/linux/pause.c && \
    gcc -Os -Wall -Werror -static -DVERSION=v3.10-v1.31.1 -o pause pause.c

FROM scratch
COPY --from=builder /pause /pause
USER 65535:65535
ENTRYPOINT ["/pause"]
