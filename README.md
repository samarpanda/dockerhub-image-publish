# Docker Hub image publishing

Dockerfiles and GitHub Actions workflows for building and publishing images under the `samarpanda` Docker Hub namespace. Each image has its own usage documentation.

## Images

The versions below match the Dockerfiles and publishing workflows in this repository; they are not a live inventory of Docker Hub tags.

| Image | Configured version tags | Purpose and usage |
| --- | --- | --- |
| [samarpanda/alpine-demo](https://hub.docker.com/r/samarpanda/alpine-demo) | `1.0.3` | [Alpine image with Bash and a demo entrypoint](alpine-demo/README.md) |
| [samarpanda/node-pm2](https://hub.docker.com/r/samarpanda/node-pm2) | `14.21.3`, `16.20.2`, `18.20.4`, `20.18.0` | [Node.js with PM2, Bash, and Curl](node-pm2/README.md) |
| [samarpanda/caddy-cloudflare-dns](https://hub.docker.com/r/samarpanda/caddy-cloudflare-dns) | `2.11.7` | [Non-root Caddy with the Cloudflare DNS module](caddy-cloudflare-dns/README.md) |

The Alpine, Node 20, and Caddy publishing workflows also update `latest` for their respective repositories. Prefer explicit version tags when deploying.

The Node.js release lines currently included here have reached end of life as of November 2026; see the [Node.js release schedule](https://github.com/nodejs/Release#release-schedule) and the [Node image documentation](node-pm2/README.md#image-versions) before choosing a base image.

## Repository layout

```text
.
├── alpine-demo/
│   ├── Dockerfile
│   ├── entrypoint.sh
│   └── README.md
├── node-pm2/
│   ├── v14/Dockerfile
│   ├── v16/Dockerfile
│   ├── v18/Dockerfile
│   ├── v20/Dockerfile
│   └── README.md
├── caddy-cloudflare-dns/
│   ├── v2/Dockerfile
│   └── README.md
└── .github/workflows/
```

## Run or build locally

Install Docker first. The Node and Caddy deployment examples also describe their application and configuration requirements.

Run the Alpine demo and remove its container when it exits:

```bash
docker run --rm samarpanda/alpine-demo:1.0.3
```

The demo prints its version and exits. For an interactive Bash session, override its entrypoint:

```bash
docker run --rm -it --entrypoint /bin/bash samarpanda/alpine-demo:1.0.3
```

Build any of the images from the repository root:

```bash
docker build -t alpine-demo:local ./alpine-demo
docker build -t node-pm2:local ./node-pm2/v20
docker build -t caddy-cloudflare-dns:local ./caddy-cloudflare-dns/v2
```

Check the Node runtime or the Caddy module list:

```bash
docker run --rm node-pm2:local node --version
docker run --rm caddy-cloudflare-dns:local caddy list-modules
```

Use the image-specific READMEs for complete deployments. In particular, [prepare Caddy's storage permissions](caddy-cloudflare-dns/README.md#1-create-the-deployment-folder) before mounting host directories: its container runs as UID/GID `10001:10001`.

## Publishing to Docker Hub

Image publishing is manually triggered through GitHub Actions using `workflow_dispatch`.

1. Configure the repository's GitHub Actions secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`. The Docker Hub account must have permission to push to the workflow's configured image repository.
2. Update the relevant Dockerfile and the publishing workflow's `DOCKER_IMAGE_TAG` together. For Node images, also keep the version-related environment variables consistent with the base image.
3. Build and verify the image locally, then commit and push the changes to the branch you intend to publish.
4. In GitHub, open **Actions**, select the corresponding publishing workflow, choose **Run workflow**, select that branch, and run it.
5. Check the workflow logs and the resulting Docker Hub tags before deploying.

| Image variant | Publishing workflow | Tags pushed |
| --- | --- | --- |
| Alpine demo | [alpine-demo.yaml](.github/workflows/alpine-demo.yaml) | Version and `latest` |
| Node 14 | [v14-node-pm2.yaml](.github/workflows/v14-node-pm2.yaml) | Version only |
| Node 16 | [v16-node-pm2.yaml](.github/workflows/v16-node-pm2.yaml) | Version only |
| Node 18 | [v18-node-pm2.yaml](.github/workflows/v18-node-pm2.yaml) | Version only |
| Node 20 | [v20-node-pm2.yaml](.github/workflows/v20-node-pm2.yaml) | Version and `latest` |
| Caddy | [v2-caddy-cloudflare-dns.yaml](.github/workflows/v2-caddy-cloudflare-dns.yaml) | Version and `latest` |

The Caddy workflow checks that `dns.providers.cloudflare` is present before publishing. The separate [Build workflow](.github/workflows/build.yml) runs SonarQube analysis on pushes to `master` and on pull request events; it does not build or publish Docker images. Its analysis uses the `SONAR_TOKEN` repository secret.

## Issues

Report problems in [the GitHub issue tracker](https://github.com/samarpanda/dockerhub-image-publish/issues). Include the image name and tag, relevant Docker or Compose configuration, and logs with credentials removed.
