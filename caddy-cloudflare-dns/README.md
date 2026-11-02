# Caddy with Cloudflare DNS

A custom Caddy image with the [`dns.providers.cloudflare`](https://github.com/caddy-dns/cloudflare) module for automatic HTTPS using Cloudflare DNS challenges, including wildcard certificates.

The [Dockerfile](v2/Dockerfile) builds Caddy with `xcaddy` and runs it as a non-root user:

| Setting | Value |
| --- | --- |
| Image | `samarpanda/caddy-cloudflare-dns:2.11.7` |
| Container user | `caddyuser` |
| UID / GID | `10001:10001` |
| Persistent data | `/data` (certificates, private keys, and ACME state) |
| Runtime configuration | `/config` |
| Caddyfile | `/etc/caddy/Caddyfile` |

The publishing workflow also tags the image as `latest`. The examples use the version tag configured in this repository.

## Prerequisites

- Docker with the Docker Compose plugin (`docker compose`).
- A domain whose DNS is managed by Cloudflare.
- A Cloudflare API token with **Zone / Zone / Read** and **Zone / DNS / Edit**, restricted to the relevant zone. See the [module's token requirements](https://github.com/caddy-dns/cloudflare#configuration).
- Available host ports 80 and 443. For public access, point your domain's DNS records at the server and allow the appropriate incoming traffic.

## 1. Create the deployment folder

From the repository root:

```bash
cd caddy-cloudflare-dns
mkdir -p deployment
cd deployment
```

Run the remaining deployment commands from this directory. Keep `deployment/` outside version control, or add it to your local Git excludes: it will contain your API token and certificate data.

### Create writable data folders on Linux

For a standard Linux Docker installation without rootless mode or user namespace remapping:

```bash
mkdir -p caddy_data caddy_config
sudo chown 10001:10001 caddy_data caddy_config
sudo chmod 700 caddy_data caddy_config
```

The numeric owner must match the container's UID/GID; a host account named `caddyuser` is unnecessary. Mode `700` lets that owner read, write, and enter these directories while keeping certificate keys private. Your regular host account may need `sudo` to inspect them afterward.

Create these folders **before** starting Caddy. A [bind mount hides the image's original directory](https://docs.docker.com/engine/storage/bind-mounts/#bind-mounting-over-existing-data), including its ownership. Setting permissions in the Dockerfile alone therefore does not fix host folder permissions.

### Repair existing data folders

If the folders already contain data from a previous deployment, stop its service before changing permissions. From the directory containing its Compose file:

```bash
docker compose stop caddy
sudo chown -R 10001:10001 caddy_data caddy_config
sudo chmod -R u+rwX,go-rwx caddy_data caddy_config
```

Use these recursive commands only on the dedicated Caddy storage directories. They preserve existing certificates and give the owner access to existing files and subdirectories without making every file executable. Do not use `chmod 777`.

## 2. Create the configuration files

Create a regular file named `Caddyfile`:

```caddyfile
example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }

    respond "Caddy with Cloudflare DNS is running."
}
```

Replace `example.com` with your domain. After verifying the setup, replace `respond` with your application configuration, such as `reverse_proxy app:3000` when the application shares a Docker network with Caddy.

Create a `.env` file and enter your token:

```dotenv
CLOUDFLARE_API_TOKEN=replace_with_your_cloudflare_api_token
```

Set permissions on these files:

```bash
chmod 644 Caddyfile
chmod 600 .env
```

Caddy needs to read the Caddyfile but does not need to modify it. Compose reads `.env` on the host and passes the token into the container. Do not commit `.env` or put the token directly in the Caddyfile.

Create `compose.yaml`:

```yaml
services:
  caddy:
    image: samarpanda/caddy-cloudflare-dns:2.11.7
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    environment:
      CLOUDFLARE_API_TOKEN: ${CLOUDFLARE_API_TOKEN:?Set CLOUDFLARE_API_TOKEN in .env}
    volumes:
      - type: bind
        source: ./Caddyfile
        target: /etc/caddy/Caddyfile
        read_only: true
        bind:
          create_host_path: false
      - type: bind
        source: ./caddy_data
        target: /data
        bind:
          create_host_path: false
      - type: bind
        source: ./caddy_config
        target: /config
        bind:
          create_host_path: false
```

The [Compose option `create_host_path: false`](https://docs.docker.com/engine/storage/bind-mounts/#use-a-bind-mount-with-docker-compose) makes missing paths fail explicitly instead of creating unexpected directories. Keep `/data` and `/config` writable; only the Caddyfile mount is read-only.

## 3. Verify permissions before starting

Run this check using the image's default non-root user and the actual deployment mounts:

```bash
docker compose run --rm --no-deps --entrypoint sh caddy -ec '
  id
  test -f /etc/caddy/Caddyfile
  test -r /etc/caddy/Caddyfile
  for dir in /data /config; do
    probe=$(mktemp "$dir/.caddy-write-check.XXXXXX")
    rm "$probe"
  done
  echo "Caddy configuration is readable and storage is writable."
'
```

Expect UID and GID `10001` and the success message. Creating and removing temporary files verifies actual write access. If the check fails, fix the permissions before continuing.

Verify the module and validate the Caddyfile:

```bash
docker compose run --rm --no-deps caddy caddy list-modules
docker compose run --rm --no-deps caddy caddy validate --config /etc/caddy/Caddyfile --adapter caddyfile
```

The module list should include `dns.providers.cloudflare`. Validation does not prove Cloudflare accepts the token; certificate issuance will exercise it.

## 4. Start and manage Caddy

```bash
docker compose up -d
docker compose logs --tail=100 -f caddy
```

After certificate issuance succeeds, visit `https://example.com`. Press Ctrl+C to leave the log viewer; Caddy keeps running.

To reload after editing the Caddyfile:

```bash
docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile --adapter caddyfile
```

After changing `.env`, run `docker compose up -d` to recreate the service with the updated environment. To stop and remove the container, run `docker compose down`. The bind-mounted data remains on the host. Back up your storage and configuration securely; losing `/data` loses the stored certificates and ACME state.

## Troubleshooting permissions

- **`permission denied` under `/data` or `/config`:** check `docker compose exec caddy id` and `docker compose exec caddy ls -ldn /data /config`. On standard Linux Docker, storage should be owned by `10001:10001`. Apply the repair commands above and repeat the write check.
- **Missing mount path or Caddyfile reported as a directory:** create the files and folders beside `compose.yaml`. If an older deployment created a directory named `Caddyfile`, move it aside and create a regular configuration file in its place.
- **Compose `user:` override:** remove it to use the image's default user, or deliberately match storage ownership to the overridden UID/GID.
- **Docker Desktop on macOS or Windows:** host ownership behaves differently from native Linux. Use a directory shared with Docker Desktop and run the container write check. Host `chown` alone is not proof of container access.
- **Rootless Docker or user namespace remapping:** container UID `10001` maps to a different host UID. The Linux ownership commands above do not apply directly; use your daemon's UID mapping when preparing mounts, then repeat the write check. See [Docker's user namespace documentation](https://docs.docker.com/engine/security/userns-remap/).
- **SELinux hosts:** Unix ownership may be correct while labels still prevent access. Apply an appropriate container label to the dedicated mounts; see [Docker's SELinux guidance](https://docs.docker.com/engine/storage/bind-mounts/#configure-the-selinux-label). Do not disable SELinux or broaden file permissions to fix a label problem.
- **`bind: permission denied` on ports 80 or 443:** this is a networking restriction, separate from storage permissions. The Dockerfile installs `libcap` but does not set a port-binding capability. Check the runtime's unprivileged-port settings; environments that require it may need the binary granted `cap_net_bind_service`, or Caddy configured to listen on higher container ports with matching host mappings.

## Build locally

From `caddy-cloudflare-dns`:

```bash
docker build -t caddy-cloudflare-dns:local ./v2
docker run --rm caddy-cloudflare-dns:local caddy list-modules
```

To use the local build, change the Compose image to `caddy-cloudflare-dns:local` and repeat validation and the write check.
