# ansi_ubuntu24.04

[![Docker Image Size](https://img.shields.io/docker/image-size/yourrepo/ansi_ubuntu24.04/latest)](https://hub.docker.com/r/yourrepo/ansi_ubuntu24.04)
[![Build Status](https://img.shields.io/github/actions/workflow/status/yourrepo/ansi_ubuntu24.04/docker-publish.yml?branch=main)](https://github.com/yourrepo/ansi_ubuntu24.04/actions)

> Customized Ubuntu 22.10 Docker image for rapid development and testing.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Testing & Validation](#testing--validation)
- [Pushing to Registry](#pushing-to-registry)
- [Multi-Platform Build](#multi-platform-build)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Features

- Based on Ubuntu 22.10
- Pre-configured SSH server and sudo user
- Supports custom tags: `ansi_ubuntu24.04:1.0.0`
- Interactive container entry for debugging

## Prerequisites

- Docker ≥ 20.10
- Buildx enabled:
  ```bash
  docker buildx version
  docker buildx create --use
  docker buildx inspect --bootstrap
  ```

## Installation

1. Clone or navigate to the project root containing the Dockerfile.
2. Build and load the image locally:
   ```bash
   sudo docker buildx build . \
     --load \
     -t yourrepo/ansi_ubuntu24.04:1.0.0
   ```

## Usage

Launch an interactive shell in the container:

```bash
docker run --rm -it yourrepo/ansi_ubuntu24.04:1.0.0 /bin/bash
```

- `--rm`: Remove container after exit
- `-it`: Interactive TTY

## Testing & Validation

Inside the running container, verify OS and kernel:

```bash
uname -a
cat /etc/os-release
```

Install additional packages:

```bash
apt-get update
apt-get install -y <package-name>
```

Check custom environment variables:

```bash
echo $YOUR_ENV_VAR
```

## Pushing to Registry

Tag and push to Docker Hub or private registry:

```bash
docker tag yourrepo/ansi_ubuntu24.04:1.0.0 registry.example.com/yourrepo/ansi_ubuntu24.04:1.0.0
docker push registry.example.com/yourrepo/ansi_ubuntu24.04:1.0.0
```

## Multi-Platform Build

Build for AMD64 and ARM64 and push directly:

```bash
docker buildx build . \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t yourrepo/ansi_ubuntu24.04:1.0.0
```

## Troubleshooting

- **ENV syntax warning**: Use `ENV KEY=VALUE` instead of `ENV KEY VALUE` in Dockerfile.
- **Image not loading**: Ensure `--load` is used, or switch to `--push` and pull from registry.

## Contributing

Contributions, issues, and feature requests are welcome!
Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

