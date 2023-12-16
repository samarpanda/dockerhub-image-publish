FROM alpine:3.19.0

ENV ALPINE_DEMO_VERSION=1.0.3

RUN apk add --no-cache bash

WORKDIR /app

COPY entrypoint.sh .
RUN chmod +x entrypoint.sh

ENTRYPOINT [ "/app/entrypoint.sh" ]