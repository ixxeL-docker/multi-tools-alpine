# multi-tools-alpine
[![multi-tools-img-CI](https://github.com/ixxeL-docker/multi-tools-alpine/actions/workflows/multi-tools-alpine-image.yml/badge.svg)](https://github.com/ixxeL-docker/multi-tools-alpine/actions/workflows/multi-tools-alpine-image.yml)

Repository dedicated to build Multi Tools Alpine image, which is a multi purpose image include most commonly used CLI tools

- `multi-tools-alpine` image : Container aiming at providing Ops tools to handle servers and CD actions (kubectl, oc, helm, vault...)

This image includes `kaniko`. Kaniko requires following environment variables :
```Dockerfile
ENV PATH="/kaniko:$PATH"
ENV SSL_CERT_DIR="/kaniko/ssl/certs"
ENV DOCKER_CONFIG="/kaniko/.docker/"
```
