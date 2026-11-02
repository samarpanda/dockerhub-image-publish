# Stage 1: Build the custom Caddy binary with the Cloudflare DNS module
FROM caddy:2.11.7-builder-alpine AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare

# Stage 2: Create the final lightweight production image
FROM caddy:2.11.7-alpine

# Copy the compiled binary from the builder stage
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
