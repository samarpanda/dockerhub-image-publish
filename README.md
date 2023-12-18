# Publish image to dockerhub

This repository contains docker images for various services & applications. Each folder is configured as an docker image. To know more check details of `Dockerfile` in each folder. Automated publishing to dockerhub using github actions.

## Directory Structure

- `/alpine-demo`: [Alpine bash docker image](https://github.com/samarpanda/dockerhub-image-publish/tree/master/alpine-demo)
- `/node-pm2`: [Node & pm2 docker image](https://github.com/samarpanda/dockerhub-image-publish/tree/master/node-pm2)

## Dockerhub repositories

- [samarpanda/alpine-demo:1.0.3](https://hub.docker.com/r/samarpanda/alpine-demo)
- [samarpanda/alpine-demo:0.0.1](https://hub.docker.com/r/samarpanda/node-pm2)

## Usage

To use the docker images in this repository, go to README.md file of the corresponding directory. Entire details has been shared in it.

```bash
cd alpine-demo/
```

## Quick commands

1. Docker build `docker build -t <DOCKER_IMAGE_NAME> -f Dockerfile .`
1. Docker run & on stop / exit remove container `docker run --rm --name <DOCKER_CONTAINER_NAME> <DOCKER_IMAGE_NAME>`
1. Docker stop container & remove image `docker stop <CONTAINER_NAME>; docker rm -f <CONTAINER_NAME>; docker image rm -f <DOCKER_IMAGE_NAME>;`

## Publish docker image to dockerhub

Github actions used to publish docker image to dockerhub

## Where to file issues?

[https://github.com/samarpanda/dockerhub-image-publish/issues](https://github.com/samarpanda/dockerhub-image-publish/issues)

