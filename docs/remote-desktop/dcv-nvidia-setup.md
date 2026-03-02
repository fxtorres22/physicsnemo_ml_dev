# DCV Remote Desktop for NVIDIA VMs

!!! warning "NVIDIA Optimized VMI"

    This section is only for **cloud VM** Linux installations. Refer to the [NVIDIA Optimized VMI](https://developer.nvidia.com/nvidia-optimized-vmi) documentation.

This setup gives you a lightweight desktop (for VS Code, browsers, and debugging).

---

## System Preparation

NVIDIA Optimized VMI does not include a desktop environment. You need to install one.

### 1. Update and Install Minimal Desktop

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ubuntu-desktop-minimal
```

WIP
