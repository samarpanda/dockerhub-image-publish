# Node.js DevOps Image: PM2, Bash, and Curl Ready
> Nodejs devops image with essentials bash, curl & pm2 for containers

This Docker image is tailored for Node.js developers, equipped with essential tools like Node.js, PM2 process manager, Bash, and Curl. It streamlines the creation of new images by providing a comprehensive environment, allowing seamless development, testing, and deployment of Node.js applications. Ideal for building scalable and efficient containerized Node.js applications, this image simplifies workflows and empowers developers to craft robust solutions effortlessly.

[hub.docker.com/r/samarpanda/node-pm2](https://hub.docker.com/r/samarpanda/node-pm2)

## Features:

1. **Node.js Environment:**  
   - Includes Node.js for seamless execution and management of Node.js applications.

2. **PM2 Process Manager:**  
   - Facilitates production-grade process management for Node.js apps.
   - Ensures uptime, scaling, and robust monitoring capabilities.

3. **Bash Shell:**  
   - Offers a flexible command-line interface for executing scripts and commands.

4. **Curl Utility:**  
   - Enables performing HTTP requests and interacting with APIs for testing and integration.

5. **Streamlined Image Creation:**  
   - Bundles necessary tools, eliminating dependency installation for quick container setup.

6. **Enhanced Development Workflow:**  
   - Provides a comprehensive environment for coding, testing, and deployment.

7. **Scalability and Robustness:**  
   - PM2 ensures scalability, efficient monitoring, and handling of application crashes.

This Docker image simplifies Node.js development within containers, offering a standardized environment for efficient workflow and application deployment.

### Usage:

Example Dockerfile
```bash
#!/bin/sh

FROM node-pm2:latest AS base

FROM base AS deps
WORKDIR /app
RUN echo "registry=https://registry.npmjs.org" > .npmrc
# Install dependencies based on the preferred package manager
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  elif [ -f package-lock.json ]; then npm ci --only=production; \
  elif [ -f pnpm-lock.yaml ]; then pnpm install --frozen-lockfile --prod; \
  else echo "No lock file found. Please add lock file. Failling the build & Exit." && exit 1; \
  fi

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

FROM base AS runner
ENV NODE_ENV production
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodejs

USER root
RUN mkdir /var/log/app && chown -R nodejs:nodejs /var/log/app
USER nodejs
COPY --from=builder /app .

CMD [ "sh", "init.sh" ]
```

Below is example init.sh using pm2

```bash
#!/bin/sh

export NODE_ENV=development

APP_NAME=app
WORKER=2
pm2-runtime start server.js --name $APP_NAME --instances $WORKER
```

Feel free to use this in various deployments around Nodejs applications

## Quick commands used to build the docker image

1. `docker build -t node-pm2:latest -f Dockerfile .`
1. `docker run --rm -it samarpanda/node-pm2:latest`
1. `docker image rm -f samarpanda/node-pm2:latest`


## Supported tags and respective Dockerfile links

- [samarpanda/node-pm2:0.0.1](https://github.com/samarpanda/dockerhub-image-publish/blob/master/alpine-demo/Dockerfile)

## Github source code folder reference

- [samarpanda/node-pm2:0.0.1](https://github.com/samarpanda/dockerhub-image-publish/tree/master/alpine-demo)

## Where to file issues?

[https://github.com/samarpanda/dockerhub-image-publish/issues](https://github.com/samarpanda/dockerhub-image-publish/issues)

