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
- Know the target platform/architecture of the machines that will run this image (e.g. `linux/amd64` for most CI/prod servers, `linux/arm64` for Apple Silicon Macs).

### ⚠️ Platform mismatch to fix first

Each `Dockerfile-<version>` pins its base image with `FROM --platform=linux/amd64 ...`. If you build with `--platform=linux/arm64` while that pin is in place, Docker forces the amd64 base anyway (silently emulated) — the resulting image is **not actually arm64**, even though you tag/push it as one.

Before publishing, pick one:
- Remove the `--platform=linux/amd64` pin from the `FROM` line in the Dockerfile if you want to build natively for the host architecture (e.g. arm64 on Apple Silicon), or
- Keep the pin and always build/tag with `--platform=linux/amd64` to match it, or
- Build a real multi-arch image (see below) if both architectures are actually needed.

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
docker build --platform=linux/arm64 -f ./Dockerfile-8.5 .   # builds an untagged image
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
