FROM rust:1-alpine as builder

ARG TAG='master'

WORKDIR /src
RUN apk add git ca-certificates alpine-sdk cmake perl pkgconf clang mold
ENV RUSTFLAGS="-C linker=clang -C link-arg=-fuse-ld=mold"
COPY ./src .
RUN cargo build --release --package knot-server --package knot-migrate
RUN strip target/release/knot-server target/release/knot-migrate

FROM alpine:latest
ENV KNOT_SCAN_PATH=/data/repos
ENV KNOT_SEALED_KEY_FILE=/data/sealed-keys
ENV KNOT_SSH_HOST_KEY_FILE=/data/ssh_host_key
ENV KNOT_MASTER_KEY_ENV=KNOT_MASTER_KEY
EXPOSE 5555 2222

LABEL org.opencontainers.image.title='knot'
LABEL org.opencontainers.image.description='data server for tangled'
LABEL org.opencontainers.image.source='https://tangled.org/did:plc:f5s5la5wlofsxidb3zemdune'
LABEL org.opencontainers.image.url='https://tangled.org'
LABEL org.opencontainers.image.vendor='tangled.org'
LABEL org.opencontainers.image.licenses='MIT'

ARG UID=1000
ARG GID=1000

RUN apk add ca-certificates curl
RUN addgroup -g $GID knot && \
    adduser -D -u $UID -G knot -h /data knot
RUN mkdir -p /data/repos && \
    chown -R $UID:$GID /data
COPY --from=builder /src/target/release/knot-server /usr/bin/knot-server
COPY --from=builder /src/target/release/knot-migrate /usr/bin/knot-migrate

HEALTHCHECK --interval=60s --timeout=30s --start-period=600s --retries=3 \
    cmd curl -f http://localhost:5555/xrpc/_health || exit 1

USER knot
ENTRYPOINT ["/usr/bin/knot-server"]
