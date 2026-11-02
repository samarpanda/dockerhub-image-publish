# Stage 1: Build the custom Caddy binary with the Cloudflare DNS module
FROM caddy:2.11.7-builder-alpine AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare

# Stage 2: Create the final lightweight production image
FROM caddy:2.11.7-alpine

# Install capability management utility
RUN apk add --no-cache libcap

# Copy the compiled binary from the builder stage
COPY --from=builder /usr/bin/caddy /usr/bin/caddy

# Create non-root user, grant port-binding capability,
# and configure writable directories
RUN addgroup -S -g 10001 caddygroup && \
    adduser -S -D -u 10001 -G caddygroup caddyuser && \
    mkdir -p /data /config && \
    chown -R caddyuser:caddygroup /data /config && \

# Run as non-root
USER caddyuser

EXPOSE 80 443 443/udp