# PhysicsNeMo Container Setup

NVIDIA PhysicsNeMo is an open-source framework for developing physics-informed machine learning models that blend physical laws with observational data. It enables the creation of high-speed AI surrogates to simulate complex engineering problems thousands of times faster than traditional numerical solvers.

---

## Prerequisites

Before pulling the container, ensure you have:

- [x] [Docker installed](../environment/docker-engine.md)
- [x] [NVIDIA Container Toolkit configured](../environment/nvidia-container-toolkit.md)
- [x] An [NVIDIA NGC account](https://ngc.nvidia.com/) (free registration)

---

## Pull the Container

Download the PhysicsNeMo Docker container from NGC:

```bash
docker pull nvcr.io/nvidia/physicsnemo/physicsnemo:<tag>
```

!!! tip "Available Tags"
    Browse all available tags on the [PhysicsNeMo NGC Container page](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/physicsnemo/containers/physicsnemo/tags).

    As of this writing, the latest tag is `25.11`.

---

## Launching Container Sessions

!!! note "Replace `<tag>`"
    In all commands below, replace `<tag>` with your installed version (e.g., `25.11`).

### Basic Shell Session

Start an interactive shell with GPU access:

```bash
docker run \
  --shm-size=1g \
  --ulimit memlock=-1 \
  --ulimit stack=67108864 \
  --runtime nvidia \
  -it --rm \
  nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> bash
```

### Shell Session with Workspace Mounted

Mount your current directory into the container for accessing your project files:

```bash
docker run \
  --shm-size=1g \
  --ulimit memlock=-1 \
  --ulimit stack=67108864 \
  --runtime nvidia \
  -v ${PWD}:/workspace \
  -it --rm \
  nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> bash
```

!!! tip "Mounting Directories"
    The `-v ${PWD}:/workspace` flag mounts your current directory to `/workspace` inside the container. Changes to files in `/workspace` persist on your host machine.

### Session with Jupyter Lab

Launch a Jupyter Lab server accessible from your browser:

```bash
docker run \
  --shm-size=1g \
  --ulimit memlock=-1 \
  --ulimit stack=67108864 \
  --runtime nvidia \
  -v ${PWD}:/workspace \
  -p 8888:8888 \
  -it --rm \
  nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> \
  jupyter lab --ip=0.0.0.0 --no-browser --allow-root
```

Then open `http://localhost:8888` in your browser and paste the token from the terminal output.

### Multi-GPU Session

To use all available GPUs:

```bash
docker run \
  --shm-size=8g \
  --ulimit memlock=-1 \
  --ulimit stack=67108864 \
  --gpus all \
  --runtime nvidia \
  -v ${PWD}:/workspace \
  -it --rm \
  nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> bash
```

---

## Understanding the Docker Run Flags

| Flag | Purpose |
|------|---------|
| `--shm-size=1g` | Sets shared memory size — increase for multi-process data loading |
| `--ulimit memlock=-1` | Removes memory locking limits (required for GPU pinned memory) |
| `--ulimit stack=67108864` | Sets stack size to 64MB (required by some CUDA operations) |
| `--runtime nvidia` | Uses the NVIDIA container runtime for GPU access |
| `--gpus all` | Exposes all GPUs to the container |
| `-v host:container` | Bind-mounts a host directory into the container |
| `-p host:container` | Maps a container port to the host (e.g., for Jupyter) |
| `-it` | Interactive mode with a pseudo-TTY |
| `--rm` | Automatically remove the container when it exits |

---

## Next Steps

- Set up a [VS Code Dev Container](devcontainer.md) for a fully integrated development experience
- Explore the [PhysicsNeMo documentation](https://docs.nvidia.com/physicsnemo/) for tutorials and API reference
- Check out [example models](https://github.com/NVIDIA/PhysicsNeMo/tree/main/examples) on GitHub
