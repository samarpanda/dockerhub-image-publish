FROM node:18-alpine

ENV NODE_PM2_VERSION=0.0.1
ENV NODE_VERSION=18.19.0
ENV NPM_VERSION=10.2.3
ENV PM2_VERSION=5.3.0

# Check https://github.com/nodejs/docker-node/tree/b4117f9333da4138b03a546ec926ef50a31506c3#nodealpine to understand why libc6-compat might be needed.
RUN apk add --no-cache libc6-compat
RUN apk add --no-cache bash curl
#RUN apk update && apk add --no-cache bash curl

WORKDIR /node
ENV NPM_CONFIG_PREFIX=/node/.npm-global
ENV PATH=$PATH:/node/.npm-global/bin
RUN npm install pm2@5.3.0 -g --only=production

#WORKDIR /app
#RUN echo "registry=https://registry.npmjs.org" > .npmrc