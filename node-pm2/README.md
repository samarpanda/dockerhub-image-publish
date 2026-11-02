# Node.js with PM2

Alpine-based Node.js images with PM2, Bash, Curl, and `libc6-compat`. Use them as a base for application images that run with `pm2-runtime`.

[Docker Hub](https://hub.docker.com/r/samarpanda/node-pm2) · [Repository overview](../README.md)

## Image versions

These tags are configured in this repository's Dockerfiles and publishing workflows. All variants install PM2 `5.4.2`.

| Image tag | Node.js base image | Dockerfile |
| --- | --- | --- |
| `20.18.0` | `node:20.18.0-alpine` | [v20/Dockerfile](v20/Dockerfile) |
| `18.20.4` | `node:18.20.4-alpine` | [v18/Dockerfile](v18/Dockerfile) |
| `16.20.2` | `node:16.20.2-alpine` | [v16/Dockerfile](v16/Dockerfile) |
| `14.21.3` | `node:14.21.3-alpine` | [v14/Dockerfile](v14/Dockerfile) |

The Node 20 publishing workflow also updates `latest`; the other Node workflows publish only their version tags. Pin a version in your application Dockerfile to avoid unexpected changes when `latest` moves.

As of November 2026, all Node.js release lines listed here have reached end of life. See the [Node.js release schedule](https://github.com/nodejs/Release#release-schedule). These images document the existing builds; upgrade the base images to a supported Node.js release before adopting them for a new production deployment.

## What the base image provides

- PM2 installed globally under `/node/.npm-global`, with its executables on `PATH`.
- Bash, Curl, and `libc6-compat` installed through Alpine's package manager.
- Working directory `/node`.

The Dockerfiles do not select a non-root user or configure an application startup command. Set both in your application image. They also declare `NPM_VERSION` as an environment variable without installing that npm version; use `npm --version` to check the actual installed version.

## Build an application image

This example assumes an npm application with `package.json`, `package-lock.json`, and an entry point named `server.js`. It installs production dependencies only and does not run a compilation step. Applications that need TypeScript compilation or another build step should build with development dependencies in a separate stage and copy the generated output into the runtime image.

Create this `Dockerfile` in your application's directory:

```dockerfile
FROM samarpanda/node-pm2:20.18.0

ENV NODE_ENV=production
ENV PM2_HOME=/home/node/.pm2
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci --omit=dev

COPY --chown=node:node . .
RUN mkdir -p /home/node/.pm2 /app/storage \
    && chown -R node:node /home/node/.pm2 /app/storage

USER node
EXPOSE 3000
CMD ["pm2-runtime", "start", "server.js", "--name", "app"]
```

The inherited `node` user runs the application. Its PM2 state directory and the example application's `/app/storage` directory are writable. Change the entry point and port to match your application, and make the server listen on `0.0.0.0` so it can receive traffic through Docker's published port.

Add a `.dockerignore` beside the Dockerfile:

```text
node_modules
.git
.env
.env.*
npm-debug.log*
storage
```

Build and start the application:

```bash
docker build -t my-node-app:local .
docker run -d --name node-app --restart unless-stopped -p 3000:3000 my-node-app:local
docker logs --tail=100 -f node-app
```

[PM2 recommends `pm2-runtime` for containers](https://pm2.keymetrics.io/docs/usage/docker-pm2-nodejs/). It keeps the application in the foreground and forwards shutdown signals. If you use a startup script, end it with `exec pm2-runtime start server.js --name app` so the shell hands control to PM2.

Stop and remove the application container:

```bash
docker stop node-app
docker rm node-app
```

## Persistent storage and permissions

Only mount directories your application actually needs to persist. A host mount replaces the image directory, so the Dockerfile's ownership does not apply to that host folder.

For standard Linux Docker without rootless mode or user namespace remapping, first check the application's runtime identity:

```bash
docker run --rm my-node-app:local id
```

For the example image, expect the inherited `node` user with UID/GID `1000:1000`. If your image reports different numbers, use those numbers below.

Create a dedicated host directory before starting a container with the mount:

```bash
mkdir -p app_storage
sudo chown 1000:1000 app_storage
sudo chmod 700 app_storage
```

Check actual write access as the application's default user:

```bash
docker run --rm \
  --mount type=bind,src="$(pwd)/app_storage",dst=/app/storage \
  my-node-app:local sh -ec '
    probe=$(mktemp /app/storage/.write-check.XXXXXX)
    rm "$probe"
    echo "Application storage is writable."
  '
```

After that succeeds, add the same `--mount` option to the application start command. The `--mount` syntax fails if the source folder is missing instead of creating one implicitly.

For existing storage, stop every container writing to it, then repair only that dedicated directory:

```bash
sudo chown -R 1000:1000 app_storage
sudo chmod -R u+rwX,go-rwx app_storage
```

Do not use `chmod 777`. Your host account may need `sudo` to inspect the private storage afterward. Docker Desktop, rootless Docker, user namespace remapping, and SELinux can require different host ownership or labeling; verify access with the container write check rather than relying on host permissions alone.

## Build the base image locally

From the repository root:

```bash
docker build -t node-pm2:local ./node-pm2/v20
docker run --rm node-pm2:local node --version
docker run --rm node-pm2:local npm --version
docker run --rm node-pm2:local pm2 --version
docker run --rm -it node-pm2:local bash
```

Replace `v20` with `v18`, `v16`, or `v14` to build another variant. To use your local build in the application example, change its `FROM` line to `FROM node-pm2:local`.

## Publishing and issues

See the [root README](../README.md#publishing-to-docker-hub) for GitHub Actions publishing instructions.

Report issues in [the GitHub issue tracker](https://github.com/samarpanda/dockerhub-image-publish/issues). Include the image tag, application startup command, and relevant logs without credentials.
