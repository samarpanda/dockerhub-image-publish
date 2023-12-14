# Github action & dockerhub integration
> Publish docker image to dockerhub using github actions

## Build docker image

1. Docker build `docker build -t <DOCKER_IMAGE_NAME> -f <DOCKERFILE_NAME> .`
1. Example command `docker build -t alpine-demo -f Dockerfile .`

## Docker run & remove the container

1. Docker run & remove `docker run --rm --name <DOCKER_IMAGE_NAME> <DOCKER_IMAGE_NAME>`
1. Example command `docker run --rm --name alpine-demo alpine-demo`

## Docker run without removing the container

1. Docker run `docker run -it --name <DOCKER_IMAGE_NAME> <DOCKER_IMAGE_NAME>`
1. Example command `docker run -it --name alpine-demo alpine-demo`

## Docker remove the container & image

1. Docker run `docker stop <DOCKER_IMAGE_NAME>; docker rm -f <DOCKER_IMAGE_NAME>; docker image rm -f <DOCKER_IMAGE_NAME>;`
1. Example command `docker stop alpine-demo; docker rm -f alpine-demo; docker image rm -f alpine-demo;`
