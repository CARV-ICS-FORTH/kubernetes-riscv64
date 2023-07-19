FROM riscv64/alpine:edge
RUN apk --no-cache add ca-certificates tzdata
COPY traefik /usr/local/bin/
RUN wget --quiet -O /entrypoint.sh "https://raw.githubusercontent.com/traefik/traefik-library-image/45ebd782ec472b47b90058b27253874297242a26/alpine/entrypoint.sh"; \
    chmod +x /entrypoint.sh
EXPOSE 80
ENTRYPOINT ["/entrypoint.sh"]
CMD ["traefik"]

# Metadata
LABEL org.opencontainers.image.vendor="Traefik Labs" \
    org.opencontainers.image.url="https://traefik.io" \
    org.opencontainers.image.source="https://github.com/traefik/traefik" \
    org.opencontainers.image.title="Traefik" \
    org.opencontainers.image.description="A modern reverse-proxy" \
    org.opencontainers.image.version="v2.10.3" \
    org.opencontainers.image.documentation="https://docs.traefik.io"