---
hide:
  - navigation
---

# PhysicsNeMo ML Dev

**A personal learning journal for deploying, training, and using NVIDIA PhysicsNeMo machine learning models.**

---

## :material-rocket-launch: About This Project

This documentation tracks every step of my journey learning [NVIDIA PhysicsNeMo](https://docs.nvidia.com/physicsnemo/) — from setting up a development environment on Windows/WSL to training physics-informed neural networks with GPU-accelerated containers.

Whether you're starting from scratch or looking for a quick reference, this site covers the full workflow:

<div class="grid cards" markdown>

-   :material-microsoft-windows:{ .lg .middle } **WSL2 Environment**

    ---

    Set up Windows Subsystem for Linux with GPU passthrough, memory tuning, and VS Code integration for ML workloads.

    [:octicons-arrow-right-24: WSL Setup Guide](environment/wsl-setup.md)

-   :material-docker:{ .lg .middle } **Docker & NVIDIA Runtime**

    ---

    Install Docker (Engine or Desktop) and the NVIDIA Container Toolkit to run GPU-accelerated containers.

    [:octicons-arrow-right-24: Docker Installation](environment/docker-engine.md)

-   :material-atom:{ .lg .middle } **PhysicsNeMo Container**

    ---

    Pull the official NGC container, launch interactive sessions, and mount your workspace for development.

    [:octicons-arrow-right-24: Container Setup](physicsnemo/container-setup.md)

-   :material-microsoft-visual-studio-code:{ .lg .middle } **VS Code Dev Container**

    ---

    Open the project in a fully configured development container with GPU support, debugger, and Jupyter.

    [:octicons-arrow-right-24: Dev Container Config](physicsnemo/devcontainer.md)

-   :material-monitor:{ .lg .middle } **Remote Desktop (DCV)**

    ---

    Set up NICE DCV on NVIDIA cloud VMs for a GPU-free desktop session while keeping the GPU for ML.

    [:octicons-arrow-right-24: DCV Setup](remote-desktop/dcv-nvidia-setup.md)

</div>

---

## :material-book-open-variant: Official NVIDIA Resources

| Resource | Description |
|----------|-------------|
| [:material-file-document: PhysicsNeMo Documentation](https://docs.nvidia.com/physicsnemo/) | Official user guide, API reference, and tutorials |
| [:material-github: PhysicsNeMo GitHub](https://github.com/NVIDIA/PhysicsNeMo) | Source code, examples, and issue tracker |
| [:material-package-variant: NGC Container Registry](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/physicsnemo/containers/physicsnemo) | Pre-built Docker images with all dependencies |
| [:material-school: NVIDIA Deep Learning Institute](https://www.nvidia.com/en-us/training/) | Free and paid courses on GPU computing and AI |
| [:material-flask: Deep Learning Examples](https://github.com/NVIDIA/DeepLearningExamples) | Reference implementations for training and inference |

---

## :material-list-status: Prerequisites

Before getting started, make sure you have:

- [x] A machine with an **NVIDIA GPU** (compute capability 7.0+)
- [x] **Windows 10/11** (build 19041+) with WSL2, or native **Ubuntu 22.04/24.04**
- [x] At least **16 GB RAM** and **50 GB free disk space**
- [x] An [NVIDIA NGC account](https://ngc.nvidia.com/) (free) for pulling containers

---

*Last updated: March 2025*