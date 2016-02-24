# docker-alpine-node
Minimal Node.js Docker Images built on Alpine Linux with S6 process manager

---------------------------------------------------------

A port of [mhart/alpine-node](https://hub.docker.com/r/mhart/alpine-node/) to
use [janeczku/alpine-kubernetes](https://hub.docker.com/r/janeczku/alpine-kubernetes/)
as its base image
to provide DNS support for  micro-services containers in Kubernetes, Consol,
Tutum or other Docker cluster environments that use DNS-based service discovery
and rely on the containers being able to use the search domains from resolv.conf
for resolving service names.

[![Docker Repository on Quay](https://quay.io/repository/republicwealth/alpine-dnsmasq-node/status "Docker Repository on Quay")](https://quay.io/repository/republicwealth/alpine-dnsmasq-node)

All versions use the one [quay.io/republicwealth/docker-alpine-dnsmasq-node](https://quay.io/repository/republicwealth/alpine-dnsmasq-node/)
repository, but each version aligns with the following tags (ie, `republicwealth/docker-alpine-node-kubernetes:<tag>`, size is full/compressed):

- Full install built with npm (2.14.15 unless specified):
  - `latest`, `5`, `5.5`, `5.5.0` – 37.01 MB (npm 3.5.3)
  - `4`, `4.2`, `4.2.5` – 36.42 MB
  - `0.12`, `0.12.9` – 33.11 MB
  - `0.10`, `0.10.41` – 28.46 MB
- Base install with node built as a static binary with no npm:
  - `base`, `base-5`, `base-5.5`, `base-5.5.0` – 26.01 MB
  - `base-4`, `base-4.2`, `base-4.2.5` – 25.63 MB
  - `base-0.12`, `base-0.12.9` – 22.31 MB
  - `base-0.10`, `base-0.10.41` – 18.5 MB

Example
-------

    $ docker run quay.io/republicwealth/alpine-node-kubernetes node --version
    v5.4.0

    $ docker run quay.io/republicwealth/alpine-node-kubernetes:4 node --version
    v4.2.2

    $ docker run quay.io/republicwealth/alpine-node-kubernetes npm --version
    3.4.0

    $ docker run quay.io/republicwealth/alpine-node-kubernetes:base node --version
    v5.4.0

Example Dockerfile for your own Node.js project
-----------------------------------------------

If you don't have any native dependencies, ie only depend on pure-JS npm
modules, then my suggestion is to run `npm install` locally *before* running
`docker build` (and make sure `node_modules` isn't in your `.dockerignore`) –
then you don't need an `npm install` step in your Dockerfile and you don't need
`npm` installed in your Docker image – so you can use one of the smaller
`base*` images.

It is recommended you setup you process to run using [s6](http://skarnet.org/software/s6/)
as s6 is using `ENTRYPOINT` to run its `init`. Example usage can be found in
[janeczku/alpine-kubernetes](https://hub.docker.com/r/janeczku/alpine-kubernetes/).

    FROM quay.io/republicwealth/alpine-node-kubernetes:base
    # FROM quay.io/republicwealth/alpine-node-kubernetes:base-0.10
    # FROM quay.io/republicwealth/alpine-node-kubernetes

    WORKDIR /src

    # Copy the s6 service.d file(s)
    # S6 requires and executable (755) runfile to be deployed to:
    # /etc/services.d/{APPLICATION NAME}/run
    COPY rootfs /

    # Copy the application
    COPY bin ./bin

    # If you have native dependencies, you'll need extra tools
    # RUN apk add --update make gcc g++ python

    # If you need npm, don't use a base tag
    # RUN npm install

    # If you had native dependencies you can now remove build tools
    # RUN apk del make gcc g++ python && \
    #   rm -rf /tmp/* /var/cache/apk/* /root/.npm /root/.node-gyp

    EXPOSE 3000
    # If you don't want to use s6, you must use CMD
    # CMD ["node", ".bin/index.js"]

Caveats
-------

As Alpine Linux uses musl, you may run into some issues with environments
expecting glibc-like behaviour (for example, Kubernetes). Some of these issues
are documented here:

- http://gliderlabs.viewdocs.io/docker-alpine/caveats/
- https://github.com/gliderlabs/docker-alpine/issues/8

Inspired by:

- https://github.com/mhart/alpine-node
- https://github.com/janeczku/docker-alpine-kubernetes
- https://github.com/alpinelinux/aports/blob/454db196/main/nodejs/APKBUILD
- https://github.com/alpinelinux/aports/blob/454db196/main/libuv/APKBUILD
- https://hub.docker.com/r/ficusio/nodejs-base/~/dockerfile/
