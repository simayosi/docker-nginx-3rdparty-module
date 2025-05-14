# Nginx Docker Image with Third-Party Module Support

Easily build custom Nginx Docker images with your chosen third-party module, using official Nginx images (Alpine or Debian). This project automates module compilation and installation, ensuring compatibility with the official Nginx build process.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Build Arguments](#build-arguments)
- [Usage](#usage)
- [Using Docker Compose](#using-docker-compose)
- [Caveats](#caveats)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Overview

This repository provides Dockerfiles to build Nginx images with third-party modules compiled from source, supporting both Alpine and Debian base images.

## Quick Start

Build an image with the [Echo module](https://github.com/openresty/echo-nginx-module):

```bash
# Alpine
docker build -f Dockerfile.alpine \
  --build-arg MODULE_NAME=echo \
  --build-arg MODULE_SOURCE=https://github.com/openresty/echo-nginx-module/archive/v0.63.tar.gz \
  -t nginx-with-echo:alpine .

# Debian
docker build -f Dockerfile.debian \
  --build-arg MODULE_NAME=echo \
  --build-arg MODULE_SOURCE=https://github.com/openresty/echo-nginx-module/archive/v0.63.tar.gz \
  -t nginx-with-echo:debian .
```

## Usage

You can use these Dockerfiles to build Nginx images with any published or custom third-party module. Just set `MODULE_NAME` and `MODULE_SOURCE` to the appropriate values for your desired module.

> **Note:** `MODULE_SOURCE` can be a URL to a Git repository (ending with `.git`), a `.zip` archive, or a tarball. The underlying `build_module.sh` script determines exact compatibility.

```bash
docker build -f Dockerfile.alpine \
  --build-arg MODULE_NAME=your-module \
  --build-arg MODULE_SOURCE=https://url-to-your-module.tar.gz \
  --build-arg NGINX_IMAGE_TAG=1.28-alpine \
  --build-arg BUILD_DEPS="required-alpine-package1 required-alpine-package2" \
  -t nginx-with-custom-module:alpine .
```

## Build Arguments

-   **`MODULE_NAME`**
    -   Module name (for package naming). If omitted, the build script attempts to guess it from `MODULE_SOURCE`.
    -   **Default:** `(auto-guessed)`
-   **`MODULE_SOURCE`**
    -   URL or path to the module source. This argument is required.
    -   **Default:** `(none)`
-   **`NGINX_IMAGE_TAG`**
    -   Base Nginx image tag.
    -   **Default:** `mainline-alpine` (for `Dockerfile.alpine`), `mainline-bookworm` (for `Dockerfile.debian`)
-   **`BUILD_DEPS`**
    -   Additional build dependencies (space-separated, e.g., `debian-pkg-foo debian-pkg-bar` for Debian, or `alpine-pkg-foo alpine-pkg-bar` for Alpine).
    -   **Default:** `(none)`

### Custom Configuration

Mount your own `nginx.conf` to load and configure the installed module:

```bash
docker run -d -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx-with-echo:alpine
```

## Using Docker Compose

Example `docker-compose.yml` (using the [Echo module](https://github.com/openresty/echo-nginx-module)):

```yaml
version: '3.8'
services:
  nginx:
    build:
      context: .
      dockerfile: Dockerfile.alpine   # or Dockerfile.debian
      args:
        MODULE_NAME: echo
        MODULE_SOURCE: https://github.com/openresty/echo-nginx-module/archive/v0.63.tar.gz
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
```

## Caveats

- Some modules require extra dependencies: use `BUILD_DEPS`.
- Ensure module compatibility with your Nginx version.
- This setup builds dynamic modules. Modules designed exclusively for static compilation into Nginx may not be compatible.
- Rebuild images after Nginx security updates.
- Final image size may be larger than the official image.
- For `MODULE_SOURCE`, use versioned URLs or specific commit hashes. This ensures reproducible builds and helps avoid issues from stale caches or unexpected upstream changes.
- Be cautious with third-party modules as they might introduce security vulnerabilities.

## Troubleshooting

1. Check Nginx logs: `docker logs <container-id>`
2. Confirm the module is loaded in your `nginx.conf`.
3. Ensure module and Nginx versions are compatible.
4. Add missing dependencies via `BUILD_DEPS`.

## License

[MIT LICENSE](LICENSE)
