# WSL2 Setup for ML Development

This guide walks through setting up **Windows Subsystem for Linux 2 (WSL2)** as a development environment for GPU-accelerated machine learning workflows.

---

## Prerequisites

!!! info "Windows Version Requirements"
    - **Windows 10** version 2004 (build 19041) or later
    - **Windows 11** — any version
    - BIOS/UEFI virtualization must be enabled (Intel VT-x or AMD-V)

!!! warning "NVIDIA GPU Driver"
    You must install the **Windows-side** NVIDIA GPU driver (not the Linux one). WSL2 uses GPU paravirtualization, so the Windows driver handles GPU communication.

    Download the latest driver from [NVIDIA Driver Downloads](https://www.nvidia.com/Download/index.aspx).

---

## Install WSL2

### Enable and Install WSL

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This command enables the required features and installs Ubuntu as the default distribution. Restart your machine when prompted.

!!! tip "Choosing a Different Distribution"
    To see available distributions:

    ```powershell
    wsl --list --online
    ```

    To install a specific one (e.g., Ubuntu 24.04):

    ```powershell
    wsl --install -d Ubuntu-24.04
    ```

### Verify WSL2 is Active

After restarting, confirm WSL2 is being used:

```powershell
wsl --list --verbose
```

Expected output:

```
  NAME            STATE           VERSION
* Ubuntu-24.04    Running         2
```

If the `VERSION` column shows `1`, upgrade it:

```powershell
wsl --set-version Ubuntu-24.04 2
```

### Set WSL2 as the Default

```powershell
wsl --set-default-version 2
```

---

## Resource Configuration

### Memory and CPU Limits (`.wslconfig`)

By default, WSL2 can use up to 50% of your system RAM. For ML workloads you'll want to tune this.

Create or edit `%USERPROFILE%\.wslconfig` (e.g., `C:\Users\YourName\.wslconfig`):

```ini
[wsl2]
# Adjust these based on your system
memory=16GB          # Limit WSL2 memory (leave room for Windows)
processors=8         # Number of CPU cores to allocate
swap=8GB             # Swap file size
localhostForwarding=true  # Access WSL2 services from Windows via localhost

[experimental]
autoMemoryReclaim=gradual  # Reclaim unused memory over time
sparseVhd=true             # Compact the virtual disk automatically
```

!!! tip "Applying Changes"
    After editing `.wslconfig`, restart WSL2:

    ```powershell
    wsl --shutdown
    ```

    Then reopen your Ubuntu terminal.

### Disk Space

WSL2 stores its filesystem as a virtual disk (`.vhdx`). To check disk usage from inside WSL:

```bash
df -h /
```

If you need more space, move the WSL distribution to a different drive:

```powershell
# Export the distribution
wsl --export Ubuntu-24.04 D:\Backups\ubuntu-backup.tar

# Unregister the old one
wsl --unregister Ubuntu-24.04

# Import to a new location
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu D:\Backups\ubuntu-backup.tar
```

---

## GPU Passthrough

### Verify GPU Access

WSL2 supports GPU passthrough natively with the Windows NVIDIA driver. Inside your WSL2 terminal:

```bash
nvidia-smi
```

You should see your GPU listed. If not, ensure:

1. You installed the **Windows** NVIDIA driver (not a Linux one inside WSL)
2. Your Windows build is recent enough
3. WSL2 is fully updated: `wsl --update` from PowerShell

### CUDA inside WSL2

The NVIDIA driver provides CUDA support automatically via `/usr/lib/wsl/lib/`. You do **not** need to install CUDA inside WSL — Docker containers bring their own CUDA runtime.

To verify CUDA is accessible:

```bash
ls /usr/lib/wsl/lib/libcuda*
```

!!! warning "Do NOT Install Linux NVIDIA Drivers"
    Installing `nvidia-driver-*` packages inside WSL2 will **conflict** with the Windows driver passthrough and break GPU access. Only install the driver on the Windows side.

---

## Windows Terminal Configuration

Windows Terminal provides a modern tabbed interface for managing multiple WSL sessions.

### Install

Windows Terminal comes pre-installed on Windows 11. On Windows 10, install from the [Microsoft Store](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701).

### Recommended Settings

Open Windows Terminal settings (`Ctrl+,`) and configure:

| Setting | Recommended Value | Why |
|---------|-------------------|-----|
| Default profile | Ubuntu-24.04 | Opens WSL directly |
| Starting directory | `//wsl.localhost/Ubuntu-24.04/home/<user>` | Starts in your Linux home |
| Font | Cascadia Code NF | Supports icons and ligatures |
| Color scheme | One Half Dark | Easy on the eyes for long sessions |

### Useful Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| ++ctrl+shift+t++ | New tab |
| ++ctrl+shift+w++ | Close tab |
| ++ctrl+shift+d++ | Split pane |
| ++ctrl+shift+p++ | Command palette |

---

## Shell Configuration for ML Workflows

### Useful Bash Aliases

Add these to your `~/.bashrc` for common ML development tasks:

```bash
# Docker shortcuts
alias dps='docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"'
alias dlog='docker logs -f'
alias dexec='docker exec -it'

# GPU monitoring
alias gpu='watch -n 1 nvidia-smi'
alias gpumem='nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader'

# PhysicsNeMo container shortcut
alias physicsnemo='docker run --shm-size=1g --ulimit memlock=-1 \
  --ulimit stack=67108864 --runtime nvidia -v ${PWD}:/workspace \
  -it --rm nvcr.io/nvidia/physicsnemo/physicsnemo:25.11 bash'

# Python / ML shortcuts
alias jlab='jupyter lab --ip=0.0.0.0 --no-browser --allow-root'
alias tb='tensorboard --logdir=./outputs --bind_all'
alias act='source .venv/bin/activate'
```

Apply changes:

```bash
source ~/.bashrc
```

---

## VS Code Remote — WSL Integration

### Setup

1. Install [Visual Studio Code](https://code.visualstudio.com/) on **Windows**
2. Install the [WSL extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl)
3. Open a WSL terminal and navigate to your project:

    ```bash
    cd ~/ml_projects/physicsnemo_ml_dev
    code .
    ```

    This opens VS Code connected to WSL2, with full access to the Linux filesystem and GPU.

### Recommended Extensions for ML

| Extension | Purpose |
|-----------|---------|
| Python | Language support and debugging |
| Pylance | Advanced IntelliSense |
| Jupyter | Notebook editing and execution |
| Dev Containers | Open projects inside Docker containers |
| Docker | Container management from the sidebar |
| GitLens | Enhanced Git integration |

---

## Networking and Port Forwarding

### Access WSL2 Services from Windows

With `localhostForwarding=true` in `.wslconfig`, services running on WSL2 (e.g., Jupyter on port `8888`) are accessible from Windows at `localhost:8888`.

### Access WSL2 Services from Other Machines

By default, WSL2 uses a NAT network. To expose services to your LAN:

```powershell
# Run in PowerShell as Administrator
netsh interface portproxy add v4tov4 listenport=8888 listenaddress=0.0.0.0 connectport=8888 connectaddress=$(wsl hostname -I | ForEach-Object { $_.Trim() })
```

!!! note "Firewall"
    You may need to add a Windows Firewall rule to allow incoming connections on the forwarded port.

---

## Troubleshooting

??? question "GPU not detected in WSL2"
    1. Ensure the **Windows** NVIDIA driver is up to date
    2. Run `wsl --update` from PowerShell
    3. Restart WSL: `wsl --shutdown`
    4. Verify with `nvidia-smi` inside WSL

??? question "Out of memory errors during training"
    Increase the `memory` value in `.wslconfig` and restart WSL. Also check that `swap` is configured.

??? question "Slow filesystem performance"
    Store your ML projects on the **Linux filesystem** (`/home/user/`), not on Windows mounts (`/mnt/c/`). Cross-filesystem access is significantly slower.

??? question "Docker fails to start"
    If using Docker Engine (not Desktop), ensure the Docker daemon is running:

    ```bash
    sudo service docker start
    ```
