# NVIDIA Container Toolkit

The NVIDIA Container Toolkit enables Docker containers to access the host system's NVIDIA GPUs for accelerated computing. It integrates with Docker Engine to automatically mount GPU drivers and libraries into containers at launch — no driver installation needed inside the container.

---

## Prerequisites

Before installing the NVIDIA Container Toolkit, ensure you have:

- [x] [Docker installed](docker-engine.md) and running
- [x] An NVIDIA GPU with compatible drivers installed on the host
- [x] Ubuntu 22.04 or 24.04

!!! tip "Check your GPU driver"
Verify your host NVIDIA driver is working:

    ```bash
    nvidia-smi
    ```

    If this command fails, install the NVIDIA driver on your host first (or on Windows if using WSL2 — see [WSL Setup](wsl-setup.md)).

---

## Install NVIDIA Container Toolkit

### 1. Install prerequisites

```bash
sudo apt-get update && sudo apt-get install -y --no-install-recommends \
  ca-certificates \
  curl \
  gnupg2
```

### 2. Configure the repository

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg \
--dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && curl \
-s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
| sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

### 3. Update the package list

```bash
sudo apt-get update
```

### 4. Install the toolkit

```bash
export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.2-1

sudo apt-get install -y \
nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

!!! note "Version Pinning"
The this example uses `1.18.2-1`. Check the [NVIDIA Container Toolkit releases](https://github.com/NVIDIA/nvidia-container-toolkit/releases) for the latest version.

---

## Configure Docker Runtime

### 1. Register the NVIDIA runtime

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

This modifies `/etc/docker/daemon.json` to add the NVIDIA runtime.

### 2. Restart Docker

```bash
sudo systemctl restart docker
```

### 3. Verify GPU access in containers

```bash
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

You should see output similar to:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.86.10    Driver Version: 535.86.10    CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

!!! success "NVIDIA Container Toolkit is ready"
Your containers can now access the host GPU. Proceed to [PhysicsNeMo Container Setup](../physicsnemo/container-setup.md).

---

## Troubleshooting

??? question "`nvidia-smi` works on host but not in container" - Ensure the NVIDIA runtime is configured: check `/etc/docker/daemon.json` for the `nvidia` runtime entry - Restart Docker: `sudo systemctl restart docker` - Use `--runtime=nvidia --gpus all` flags when running containers

??? question "Permission denied errors"
Make sure your user is in the `docker` group (see [Docker post-install](docker-engine.md#manage-docker-as-a-non-root-user)).

??? question "Toolkit version conflicts"
Remove all existing toolkit packages and reinstall:

    ```bash
    sudo apt-get remove nvidia-container-toolkit*
    sudo apt-get install nvidia-container-toolkit
    ```

---

## References

- [NVIDIA Container Toolkit Documentation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/index.html)
- [NVIDIA Container Toolkit GitHub](https://github.com/NVIDIA/nvidia-container-toolkit)
