# VS Code Dev Container

This project includes a pre-configured [Dev Container](https://containers.dev/) that launches the PhysicsNeMo NGC container directly in VS Code with GPU support, debugging tools, and Jupyter integration.

---

## Prerequisites

- [x] [Docker installed](../environment/docker-engine.md) and running
- [x] [NVIDIA Container Toolkit](../environment/nvidia-container-toolkit.md) configured
- [x] [VS Code](https://code.visualstudio.com/) with the [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

---

## Getting Started

### Open in Dev Container

1. Open the project folder in VS Code
2. Press ++ctrl+shift+p++ and select **Dev Containers: Reopen in Container**
3. VS Code will pull the PhysicsNeMo image (if needed) and launch the container
4. Wait for the post-creation setup to complete (installs `pip` and `jupyter`)

!!! info "First Launch"
    The first time you open the Dev Container, it needs to pull the ~15 GB PhysicsNeMo image. Subsequent launches are much faster since the image is cached locally.

---

## Configuration Explained

The Dev Container is defined in [`.devcontainer/devcontainer.json`](file:///home/fxtorres22/ml_projects/physicsnemo_ml_dev/.devcontainer/devcontainer.json). Here's what each section does:

### Container Image

```json
"image": "nvcr.io/nvidia/physicsnemo/physicsnemo:25.11"
```

Uses the official PhysicsNeMo container from NVIDIA NGC. Change the tag to use a different version.

### GPU and Runtime Arguments

```json
"runArgs": [
    "--gpus=all",
    "--shm-size=8g",
    "--ulimit", "memlock=-1",
    "--ulimit", "stack=67108864",
    "--runtime=nvidia"
]
```

| Argument | Purpose |
|----------|---------|
| `--gpus=all` | Exposes all NVIDIA GPUs to the container |
| `--shm-size=8g` | 8 GB shared memory for multi-worker data loading |
| `--ulimit memlock=-1` | Unlimited memory locking (required by CUDA) |
| `--ulimit stack=67108864` | 64 MB stack size (required by some CUDA ops) |
| `--runtime=nvidia` | Enables the NVIDIA container runtime |

### Workspace Mount

```json
"workspaceMount": "source=${localWorkspaceFolder},target=/workspace,type=bind,consistency=cached",
"workspaceFolder": "/workspace"
```

Mounts your local project folder to `/workspace` inside the container. The `cached` consistency mode improves filesystem performance on macOS/WSL2.

### Port Forwarding

```json
"forwardPorts": [8888],
"portsAttributes": {
    "8888": {
        "label": "Jupyter",
        "onAutoForward": "notify"
    }
}
```

Automatically forwards port `8888` for Jupyter Lab. VS Code will notify you when the port becomes available.

### Post-Creation Command

```json
"postCreateCommand": "pip install --upgrade pip jupyter"
```

Runs after the container is created to ensure `pip` and `jupyter` are up to date.

### Extensions

The Dev Container automatically installs these VS Code extensions inside the container:

| Extension | Purpose |
|-----------|---------|
| Remote Containers | Container management |
| Docker | Docker sidebar integration |
| GitHub Copilot + Chat | AI code assistance |
| Jupyter (suite) | Notebook editing, rendering, cell tags |
| Python + Pylance | Python language support and IntelliSense |
| Python Debugger (debugpy) | Interactive debugging |
| Python Environments | Virtual environment management |
| YAML | YAML file editing |

---

## Customization

### Adding Python Packages

To install additional packages that persist across container rebuilds, modify the `postCreateCommand`:

```json
"postCreateCommand": "pip install --upgrade pip jupyter tensorboard wandb"
```

### Changing the PhysicsNeMo Version

Update the `image` field with a different tag from the [NGC Container Registry](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/physicsnemo/containers/physicsnemo/tags):

```json
"image": "nvcr.io/nvidia/physicsnemo/physicsnemo:<new-tag>"
```

Then rebuild the container: ++ctrl+shift+p++ → **Dev Containers: Rebuild Container**.

### Adding More Port Forwards

To expose additional services (e.g., TensorBoard on port `6006`):

```json
"forwardPorts": [8888, 6006],
"portsAttributes": {
    "8888": { "label": "Jupyter", "onAutoForward": "notify" },
    "6006": { "label": "TensorBoard", "onAutoForward": "notify" }
}
```

---

## Troubleshooting

??? question "Container fails to start with GPU errors"
    Ensure the NVIDIA Container Toolkit is properly configured and Docker can access the GPU:

    ```bash
    docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
    ```

??? question "Extensions not installing"
    Rebuild the container: ++ctrl+shift+p++ → **Dev Containers: Rebuild Container**

??? question "Slow filesystem performance"
    If using WSL2, ensure your project files are on the Linux filesystem (`/home/user/`), not on a Windows mount (`/mnt/c/`).
