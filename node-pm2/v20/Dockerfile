FROM node:20.18.0-alpine

ENV NODE_PM2_VERSION=20.18.0
ENV NODE_VERSION=20.18.0
ENV NPM_VERSION=10.9.0
ENV PM2_VERSION=5.4.2

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk update && apk add --no-cache libc6-compat bash curl

WORKDIR /node
ENV NPM_CONFIG_PREFIX=/node/.npm-global
ENV PATH=$PATH:/node/.npm-global/bin
RUN npm install pm2@5.4.2 -g --only=production

#WORKDIR /app
#RUN echo "registry=https://registry.npmjs.org" > .npmrc