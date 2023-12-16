FROM alpine:3.19.0

ENV ALPINE_DEMO_VERSION=1.0.2

RUN apk update && apk add --no-cache bash

WORKDIR /app

COPY README.md .
COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

ENTRYPOINT [ "/app/entrypoint.sh" ]