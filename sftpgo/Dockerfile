# syntax=docker/dockerfile:1.4

# 1. For build React app
# FROM node:lts AS development

# # Set working directory
# WORKDIR /app

# # 
# COPY package.json /app/package.json
# COPY package-lock.json /app/package-lock.json

# # Same as npm install
# RUN npm ci

# COPY . /app

# ENV CI=true
# ENV PORT=3000

# CMD [ "npm", "start" ]

# FROM development AS build

# RUN npm run build


# FROM development as dev-envs
# RUN <<EOF
# apt-get update
# apt-get install -y --no-install-recommends git
# EOF

# RUN <<EOF
# useradd -s /bin/bash -m vscode
# groupadd docker
# usermod -aG docker vscode
# EOF
# # install Docker tools (cli, buildx, compose)
# COPY --from=gloursdocker/docker / /
# CMD [ "npm", "start" ]

# # 2. For Nginx setup
# FROM nginx:alpine

# # Copy config nginx
# COPY --from=build /app/.nginx/nginx.conf /etc/nginx/conf.d/default.conf

# WORKDIR /usr/share/nginx/html

# # Remove default nginx static assets
# RUN rm -rf ./*

# # Copy static assets from builder stage
# COPY --from=build /app/build .

# # Containers run nginx with global directives and daemon off
# ENTRYPOINT ["nginx", "-g", "daemon off;"]


FROM ubuntu:latest

WORKDIR /app
ARG UID
ARG GID
ARG UNAME

RUN apt update && \
    apt install -y sudo && \
    addgroup --gid $GID $UNAME && \
    adduser --uid $UID --gid $GID --disabled-password --gecos "" $UNAME && \
    echo "$UNAME ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN chage -I -1 -m 0 -M 99999 -E -1 $UNAME

RUN apt update
RUN apt install -y software-properties-common
RUN add-apt-repository -y ppa:sftpgo/sftpgo
RUN apt update
RUN apt install -y sftpgo

ENV SFTPGO_DEFAULT_ADMIN_USERNAME="cx"
ENV SFTPGO_DEFAULT_ADMIN_PASSWORD="ok"


# RUN jq '.data_provider.create_default_admin = true' /etc/sftpgo/sftpgo.json > temp.json; \
#     mv temp.json /etc/sftpgo/sftpgo.json

COPY ./sftpgo.json /etc/sftpgo/sftpgo.json

RUN mkdir -p /app/shared
RUN chmod -R a+rw /app/shared
RUN chown -R $UNAME:$UNAME /app/
RUN chmod -R a+rw /app/shared

USER $UNAME

#ENTRYPOINT ["sftpgo", "serve"]
CMD ["sftpgo", "serve"]

# required if using CMD instead of entrypoint
STOPSIGNAL SIGINT

# https://stackoverflow.com/questions/45405212/safe-way-to-use-build-time-argument-in-docker/51921954#51921954
# 
# docker build --no-cache --progress=plain --secret id=mysecret,src=$(pwd)/mysecret.txt -f - . <<EOF
# syntax = tonistiigi/dockerfile:secrets20180808
# FROM busybox
# RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
# RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar
# EOF

# docker build --build-arg UID=2000 --build-arg GID=1000 --build-arg UNAcME=user -t sftpgo .
# docker run --name sftp --rm -it -p 8090:8080 -p 2022:2022 -v ./app:/app/ sftpgo


# https://blog.zartre.com/2022/set-up-a-local-sftp-server-in-minutes-with-sftpgo/

# sftp -P 2022 user@127.0.0.1