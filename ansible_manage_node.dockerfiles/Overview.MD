# SSH-Enabled Ubuntu & RHEL Containers

Efficiently build, deploy, and manage SSH-enabled Ubuntu 24.04 and Red Hat UBI 8.8 containers with Docker Compose and key-based authentication.

---

## Table of Contents
1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Prerequisites](#prerequisites)
4. [Project Structure](#project-structure)
5. [Building Container Images](#building-container-images)
6. [SSH Key Management](#ssh-key-management)
7. [Docker Compose Configuration](#docker-compose-configuration)
8. [Launching and Accessing Containers](#launching-and-accessing-containers)
9. [Security Best Practices](#security-best-practices)
10. [Troubleshooting](#troubleshooting)
11. [Contributing](#contributing)
12. [License](#license)

---

## Overview
This repository automates the creation of two SSH-ready containers:

- **ubuntu22**: Ubuntu 24.04 with OpenSSH server
- **rhel8**: Red Hat UBI 8.8 with OpenSSH server

Each container is configured for secure SSH access using host-managed key pairs, enabling seamless Ansible or manual administration workflows.

## Architecture
- **Dockerfiles** define base image configuration and user setup.
- **Host SSH Keys** are generated and stored outside the containers for auditability.
- **Docker Compose** orchestrates build, network, port mappings, and volume mounts.

## Prerequisites
- **Docker Engine** ≥ 20.10
- **Docker Compose CLI** or standalone `docker-compose` ≥ 1.27
- **ssh-keygen** available on the host OS
- **macOS** (Docker Desktop) or **Linux** host environment

## Project Structure
```
project-root/
├── ubuntu-24.04/
│   └── Dockerfile.ubuntu          # Ubuntu 24.04 container definition
├── redhat-8.8/
│   └── Dockerfile.rhel            # Red Hat UBI 8.8 container definition
├── docker-compose.yml             # Compose orchestration file
└── host_ssh_keys/                 # Host-side SSH key storage
    ├── ubuntu/
    │   ├── id_ed25519            # Private key for ubuntu22
    │   ├── id_ed25519.pub        # Public key for ubuntu22
    │   └── authorized_keys       # Authorized keys file
    └── rhel/
        ├── id_ed25519            # Private key for rhel8
        ├── id_ed25519.pub        # Public key for rhel8
        └── authorized_keys       # Authorized keys file
```

## Building Container Images

### Using Docker Compose
```bash
cd project-root
docker-compose build
```
This command will build both `ubuntu22` and `rhel8` images based on their respective Dockerfiles.

### Manual Build
```bash
# Ubuntu image
docker buildx build . \
  --load \
  -f ubuntu-24.04/Dockerfile.ubuntu \
  -t ansi_ubuntu24.04:1.0.0

# RHEL image
docker buildx build . \
  --load \
  -f redhat-8.8/Dockerfile.rhel \
  -t ansi_rhel8:1.0.0
```

## SSH Key Management
1. **Create directories**
   ```bash
   mkdir -p host_ssh_keys/ubuntu host_ssh_keys/rhel
   ```
2. **Generate ed25519 key pairs**
   ```bash
   ssh-keygen -t ed25519 -f host_ssh_keys/ubuntu/id_ed25519 -N "" -C "docker@ubuntu22"
   ssh-keygen -t ed25519 -f host_ssh_keys/rhel/id_ed25519   -N "" -C "docker@rhel8"
   ```
3. **Prepare `authorized_keys`**
   ```bash
   cp host_ssh_keys/ubuntu/id_ed25519.pub host_ssh_keys/ubuntu/authorized_keys
   cp host_ssh_keys/rhel/id_ed25519.pub   host_ssh_keys/rhel/authorized_keys
   ```
4. **Set secure permissions**
   ```bash
   chmod 700 host_ssh_keys/ubuntu host_ssh_keys/rhel
   chmod 600 host_ssh_keys/ubuntu/authorized_keys host_ssh_keys/rhel/authorized_keys
   ```

## Docker Compose Configuration
The `docker-compose.yml` mounts each `authorized_keys` into `/home/docker/.ssh` and exposes SSH ports:
```yaml
version: '3.8'
services:
  ubuntu22:
    build:
      context: .
      dockerfile: ubuntu-24.04/Dockerfile.ubuntu
    image: ansi_ubuntu24.04:1.0.0
    container_name: ubuntu22
    ports:
      - "2222:22"
    volumes:
      - ./host_ssh_keys/ubuntu/authorized_keys:/home/docker/.ssh/authorized_keys:ro
    restart: unless-stopped

  rhel8:
    build:
      context: .
      dockerfile: redhat-8.8/Dockerfile.rhel
    image: ansi_rhel8:1.0.0
    container_name: rhel8
    ports:
      - "2223:22"
    volumes:
      - ./host_ssh_keys/rhel/authorized_keys:/home/docker/.ssh/authorized_keys:ro
    restart: unless-stopped
```

## Launching and Accessing Containers
```bash
# Start containers
docker-compose up -d

# SSH into Ubuntu 22 container
ssh -i host_ssh_keys/ubuntu/id_ed25519 docker@localhost -p 2222

# SSH into RHEL8 container
ssh -i host_ssh_keys/rhel/id_ed25519   docker@localhost -p 2223
```

## Security Best Practices
- **Use ed25519**: Strong, compact key algorithm.
- **Restrict permissions**: Ensure private keys are only readable by the owner.
- **Read-Only mounts**: Mount `authorized_keys` as read-only (`:ro`).
- **Isolate networks**: Use separate Docker networks for production environments.

## Troubleshooting
- **Mount errors**: Verify `authorized_keys` exists and is a file, not a directory.
- **SSHD failures**: Ensure your Dockerfile includes `mkdir -p /home/docker/.ssh && chown docker:docker /home/docker/.ssh`.
- **Port conflicts**: Change host ports if `2222` or `2223` are occupied.

## Contributing
Contributions welcome! Please submit issues or pull requests with clear descriptions of changes.

## License
MIT © Hanks JC Tsai