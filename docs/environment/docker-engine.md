# Docker Installation

Docker is the core runtime for building and running containers. This guide covers two installation paths: **Docker Desktop** (GUI-based) and **Docker Engine** (CLI-only).

---

## Which One Should I Choose?

=== ":material-monitor: Docker Desktop"

    **Best for:** Windows/macOS users who want a GUI, WSL2 integration, and automatic updates.

    | Pros | Cons |
    |------|------|
    | Simple GUI for managing containers | Heavier resource usage |
    | Built-in WSL2 backend on Windows | Requires license for large organizations |
    | Automatic updates | Runs as a background service |
    | Includes Docker Compose v2 | |

=== ":material-console: Docker Engine"

    **Best for:** Linux servers, headless machines, or users who prefer CLI-only workflows.

    | Pros | Cons |
    |------|------|
    | Lightweight, no GUI overhead | Manual updates |
    | Full control over daemon configuration | CLI-only interface |
    | Free for all use cases | Requires manual setup |
    | Standard for production environments | |

---

## Docker Desktop

!!! info "WSL2 Backend"
    On Windows, Docker Desktop uses WSL2 as its backend. Make sure you have [WSL2 set up](wsl-setup.md) before proceeding.

### Install on Windows

1. Download the installer from [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. Run the installer and ensure **"Use WSL 2 instead of Hyper-V"** is checked
3. Follow the prompts and restart when asked
4. Open Docker Desktop and complete the initial setup

### Install on Ubuntu (with GUI)

```bash
# Download the .deb package
wget https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb

# Install
sudo apt install ./docker-desktop-amd64.deb
```

### Verify Installation

Open a terminal (or WSL2) and run:

```bash
docker --version
docker run hello-world
```

---

## Docker Engine (CLI Only)

### Supported Ubuntu Versions

| Version | Codename |
|---------|----------|
| Ubuntu 25.10 | Kinetic |
| Ubuntu 24.04 LTS | Noble |
| Ubuntu 22.04 LTS | Jammy |

### Uninstall Conflicting Packages

Remove any unofficial Docker packages that may interfere:

```bash
sudo apt remove $(dpkg --get-selections \
  docker.io docker-compose docker-compose-v2 \
  docker-doc podman-docker containerd runc 2>/dev/null | cut -f1) 2>/dev/null
```

### Install Using the `apt` Repository

#### 1. Set up Docker's `apt` repository

Add Docker's official GPG key:

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
```

Add the repository to `apt` sources:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update the package index:

```bash
sudo apt update
```

#### 2. Install Docker packages

```bash
sudo apt install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

#### 3. Verify Docker is running

```bash
sudo systemctl status docker
```

If the service is not active, start it:

```bash
sudo systemctl start docker
```

#### 4. Test the installation

```bash
sudo docker run hello-world
```

!!! success "You have successfully installed Docker Engine"
    The `hello-world` container downloads a test image, runs it, prints a confirmation message, and exits.

---

## Post-Installation Steps

These steps apply to **both** Docker Desktop and Docker Engine installations.

### Manage Docker as a Non-Root User

!!! warning "Security Notice"
    The `docker` group grants root-level privileges. Only add trusted users to this group.

1. Create the `docker` group:

    ```bash
    sudo groupadd docker
    ```

2. Add your user to the group:

    ```bash
    sudo usermod -aG docker $USER
    ```

3. Activate the group changes:

    ```bash
    newgrp docker
    ```

4. Verify you can run Docker without `sudo`:

    ```bash
    docker run hello-world
    ```

### Configure Docker to Start on Boot

!!! note "Ubuntu Default"
    On Ubuntu, Docker starts automatically on boot. Use these commands only to change this behavior.

=== "Enable auto-start"

    ```bash
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service
    ```

=== "Disable auto-start"

    ```bash
    sudo systemctl disable docker.service
    sudo systemctl disable containerd.service
    ```

---

## Useful Docker Commands

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker images` | List downloaded images |
| `docker system prune` | Remove unused data (stopped containers, dangling images) |
| `docker logs -f <name>` | Follow container logs |
| `docker exec -it <name> bash` | Open a shell in a running container |

---

## Next Steps

With Docker installed, proceed to set up the [NVIDIA Container Toolkit](nvidia-container-toolkit.md) for GPU support.
