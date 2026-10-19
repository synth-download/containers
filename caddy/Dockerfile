FROM caddy:builder-alpine AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/porkbun \
    --with github.com/caddyserver/replace-response \
    --with github.com/stardothosting/caddy-altcha

FROM caddy:alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
