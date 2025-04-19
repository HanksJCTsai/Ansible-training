# ansi_rhel8

> Customized Red Hat UBI 8 Docker image with SSH server and `docker` user support.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Building the Image](#building-the-image)
- [Running the Container](#running-the-container)
- [Testing & Validation](#testing--validation)
- [Tagging & Pushing](#tagging--pushing)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Features

- Base image: Red Hat UBI 8.8
- SSH server configured (`openssh-server`)
- Root login enabled
- `docker` user with passwordless sudo
- SSH key-based authentication for `docker` user
- Exposes port `22`

## Prerequisites

- Docker ≥ 20.10
- Registry access to `registry.access.redhat.com`
- Optional: Buildx enabled for advanced builds
  ```bash
  docker buildx create --use
  docker buildx inspect --bootstrap
  ```

## Building the Image

```bash
docker buildx build . \
  --load \
  -f Dockerfile.rhel \
  -t ansi_rhel8:1.0.0
```

- `--load`: load the built image into local Docker
- `-f Dockerfile.rhel`: specify the RHEL Dockerfile
- `-t ansi_rhel8:1.0.0`: assign name and tag

## Running the Container

```bash
docker run --rm -it ansi_rhel8:1.0.0 /bin/bash
```

- `--rm`: remove container after exit
- `-it`: interactive TTY

## Testing & Validation

1. **Verify OS and kernel**:
   ```bash
   uname -a
   cat /etc/os-release
   ```
2. **Test SSH access**:
   ```bash
   # On host, get container ID or IP, then:
   ssh docker@<container-ip> -i ~/.ssh/id_rsa
   ```
3. **Check user and sudo**:
   ```bash
   su - docker
   sudo whoami  # should return 'root'
   ```

## Tagging & Pushing

```bash
docker tag ansi_rhel8:1.0.0 yourrepo/ansi_rhel8:1.0.0
docker push yourrepo/ansi_rhel8:1.0.0
```

## Troubleshooting

- **DNF Errors**: ensure network access to Red Hat registry and run `dnf clean all`.
- **SSHD not starting**: verify path `/usr/sbin/sshd` and correct configuration in `/etc/ssh/sshd_config`.
- **PAM sessions issue**: confirm `pam_loginuid.so` line is updated in `/etc/pam.d/sshd`.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

