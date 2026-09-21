# Laradock's Workspace Base Image

[Contribution Guide](http://laradock.io/contributing/#edit-base-image).

[Workspace Docker Hub Repository](https://hub.docker.com/r/laradock/workspace/)

[Laradock Github Repository](https://github.com/Laradock/laradock).

## Publishing a new image version

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine + `buildx` plugin) installed and running.
- Logged in to Docker Hub with an account that has push access to `firstavenueit/workspace`:
  ```bash
  docker login
  ```
- Build for `linux/amd64` — Bitbucket Pipelines runners are amd64, and each `Dockerfile-<version>` pins its base image with `FROM --platform=linux/amd64 ...`, so always match `--platform=linux/amd64` on the build too.

### Recommended steps (single architecture)

Tag the image at build time with `-t` instead of building untagged and looking up the hash afterwards — it's fewer steps and avoids accidentally grabbing the wrong hash from `docker image ls`.

```bash
# Build and tag in one step (match --platform to the Dockerfile's FROM pin)
docker build --platform=linux/amd64 -f ./Dockerfile-8.5 -t firstavenueit/workspace:latest-85 .

# Push the tagged image
docker push firstavenueit/workspace:latest-85
```

### Previous (working, but avoidable) steps

This works, but has an unnecessary manual lookup step and risks tagging the wrong image if multiple untagged builds exist locally:

```bash
docker build --platform=linux/amd64 -f ./Dockerfile-8.5 .   # builds an untagged image
docker image ls --all                                       # find the new image's hash (no tag)
docker tag <hash> firstavenueit/workspace:latest-85          # tag it manually
docker push firstavenueit/workspace:latest-85                # push
```

### Multi-arch build (optional, if both amd64 and arm64 are needed)

Requires `docker buildx` (bundled with modern Docker Desktop) and a builder that supports multiple platforms:

```bash
docker buildx create --use --name multiarch-builder   # one-time setup
docker buildx build --platform=linux/amd64,linux/arm64 \
    -f ./Dockerfile-8.5 \
    -t firstavenueit/workspace:latest-85 \
    --push .
```

This pushes a single manifest that resolves to the correct architecture per-pull. Note: this still requires removing the hardcoded `--platform=linux/amd64` pin from the Dockerfile's `FROM` line, otherwise every architecture in the build gets the amd64 base.
