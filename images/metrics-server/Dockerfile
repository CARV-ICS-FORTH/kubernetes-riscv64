FROM riscv64/alpine:edge as build

RUN apk add -U bash build-base go git
RUN mkdir -p /go/src/sigs.k8s.io/metrics-server && \
    cd /go/src/sigs.k8s.io && \
    git clone --depth 1 --branch v0.6.3 https://github.com/kubernetes-sigs/metrics-server.git
WORKDIR /go/src/sigs.k8s.io/metrics-server

RUN go mod download

ARG ARCH=riscv64
RUN make metrics-server

FROM ghcr.io/go-riscv/distroless/static-unstable:latest
COPY --from=build /go/src/sigs.k8s.io/metrics-server/metrics-server /
USER 65534
ENTRYPOINT ["/metrics-server"]