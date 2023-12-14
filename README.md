# Publish docker image to dockerhub
> Automate publishing docker image to dockerhub using github actions

## Quick docker commands

1. Docker build `docker build -t alpine-demo -f Dockerfile`
1. Docker start & remove after used `docker run --rm --name alpine-demo alpine-demo`
1. Docker start `docker run -it --name alpine-demo alpine-demo`
1. Docker stop, remove container & remove the image `docker stop alpine-demo; docker rm -f alpine-demo; docker image rm -f alpine-demo;`