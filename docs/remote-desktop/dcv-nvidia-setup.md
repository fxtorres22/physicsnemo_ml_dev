# DCV Remote Desktop for NVIDIA VMs

NICE DCV provides a high-performance remote desktop for cloud GPU instances. This setup gives you a lightweight **CPU-rendered desktop** (for VS Code, browsers, and debugging) while keeping the **NVIDIA GPU 100% free** for ML training and inference.

---

## Overview

```
┌─────────────────────────────────────────┐
│  Cloud VM (e.g., AWS g4dn, GCP A2)     │
│                                         │
│  ┌──────────────┐   ┌───────────────┐   │
│  │  Desktop GUI │   │  ML Training  │   │
│  │  (CPU render)│   │  (GPU)        │   │
│  │  via DCV     │   │  via Docker   │   │
│  └──────────────┘   └───────────────┘   │
│                                         │
│  DCV Server (port 8443)                 │
└─────────────────────────────────────────┘
         │
         │ HTTPS / WebSocket
         ▼
┌─────────────────────┐
│  Your Local Machine  │
│  DCV Client/Browser  │
└─────────────────────┘
```

---

## System Preparation

### 1. Update and Install Minimal Desktop

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ubuntu-desktop-minimal
```

### 2. Install NVIDIA Drivers

!!! warning "Skip this on WSL2"
    On WSL2 the Windows-side driver handles the GPU. This step is only for **bare-metal** or **cloud VM** Linux installations.

Install the recommended driver for your GPU:

```bash
# Check recommended driver version
sudo ubuntu-drivers devices

# Install the recommended driver
sudo ubuntu-drivers autoinstall
```

Or install a specific version:

```bash
sudo apt install -y nvidia-driver-535
```

### 3. Install Dummy Display Driver

The dummy driver is required for **virtual sessions** (headless servers with no physical monitor):

```bash
sudo apt install -y xserver-xorg-video-dummy
```

### 4. Reboot

```bash
sudo reboot
```

---

## Install NICE DCV

### 1. Import GPG Key

```bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
gpg --import NICE-GPG-KEY
```

### 2. Download DCV

Download the DCV 2025.0 package for Ubuntu 24.04:

```bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/2025.0/Servers/nice-dcv-2025.0-20103-ubuntu2404-x86_64.tgz
tar -xvzf nice-dcv-2025.0-20103-ubuntu2404-x86_64.tgz
cd nice-dcv-2025.0-20103-ubuntu2404-x86_64
```

!!! tip "Other Ubuntu Versions"
    Replace `ubuntu2404` in the URL with your Ubuntu version (e.g., `ubuntu2204` for 22.04). Check the [DCV downloads page](https://www.nice-dcv.com/) for the latest packages.

### 3. Install All Packages

This installs the DCV Server, Web Viewer, GPU shim, and X drivers:

```bash
sudo apt install -y \
  ./nice-dcv-server_*.deb \
  ./nice-dcv-web-viewer_*.deb \
  ./nice-dcv-gl_*.deb \
  ./nice-xdcv_*.deb
```

### 4. Configure User Access

```bash
# Add your user to the video group
sudo usermod -aG video ubuntu

# Set a password for GUI login (if not already set)
sudo passwd ubuntu
```

---

## Choose a Display Manager

=== ":material-monitor: GDM3 (Default)"

    **Pros:** Already installed with `ubuntu-desktop-minimal`.

    **Cons:** Heavier, can be buggy with NVIDIA drivers.

    You **must** disable Wayland for stability:

    ```bash
    sudo sed -i 's/#WaylandEnable=false/WaylandEnable=false/' /etc/gdm3/custom.conf
    ```

    !!! info "Recommendation"
        GDM3 works well for virtual-mode DCV. Switch to LightDM only if you experience boot or display issues.

=== ":material-feather: LightDM (Alternative)"

    **Pros:** Lightweight, very reliable for headless servers.

    **Cons:** Requires installation.

    ```bash
    sudo apt install lightdm -y
    ```

    When prompted, select `lightdm` as the default display manager.

---

## Configure Automatic Virtual Session

### 1. Edit the DCV Configuration

```bash
sudo nano /etc/dcv/dcv.conf
```

### 2. Update the Configuration

Add or uncomment the following entries:

```ini
[session-management]
create-session = true

[session-management/defaults]
owner = "ubuntu"
type = "virtual"
```

!!! note "Session Type"
    - `virtual` — Creates a virtual framebuffer (no physical display needed). The GPU is **not** used for desktop rendering.
    - `console` — Uses the physical console/GPU for rendering. Use this only if you need GPU-accelerated desktop apps.

### 3. Save and Exit

- Press ++ctrl+o++, ++enter++ to save
- Press ++ctrl+x++ to exit

---

## Start the DCV Server

```bash
# Enable DCV to start on boot
sudo systemctl enable dcvserver

# Start (or restart) the service
sudo systemctl restart dcvserver
```

---

## Connecting to DCV

### From a Web Browser

Navigate to:

```
https://<your-server-ip>:8443
```

Accept the self-signed certificate warning, then log in with your Ubuntu credentials.

### From the DCV Client

Download the [NICE DCV Client](https://www.nice-dcv.com/) for your local OS, then connect to:

```
<your-server-ip>:8443
```

!!! tip "Performance"
    The native DCV client provides better performance and lower latency than the web viewer, especially for large displays.

---

## Firewall Configuration

Ensure port `8443` (TCP) is open:

=== "UFW (Ubuntu)"

    ```bash
    sudo ufw allow 8443/tcp
    sudo ufw reload
    ```

=== "AWS Security Group"

    Add an inbound rule:

    | Type | Protocol | Port | Source |
    |------|----------|------|--------|
    | Custom TCP | TCP | 8443 | Your IP (e.g., `203.0.113.0/32`) |

=== "GCP Firewall"

    ```bash
    gcloud compute firewall-rules create allow-dcv \
      --allow tcp:8443 \
      --source-ranges <your-ip>/32 \
      --target-tags dcv-server
    ```

---

## Troubleshooting

### Diagnostic Commands

| Command | Purpose |
|---------|---------|
| `sudo lshw -c video` | List video hardware and drivers |
| `sudo dcvgldiag` | DCV GPU diagnostics |
| `nvidia-smi` | GPU status and memory usage |
| `dcv list-sessions` | List active DCV sessions |
| `sudo systemctl status dcvserver` | Check DCV service status |
| `journalctl -u dcvserver -f` | Follow DCV server logs |

### Manual Session Creation

If the automatic session doesn't start, create one manually:

```bash
sudo dcv create-session --type virtual --owner ubuntu session1
```

Verify it's running:

```bash
dcv list-sessions
```

??? question "Black screen after connecting"
    - Ensure the display manager is running: `sudo systemctl status gdm3` (or `lightdm`)
    - Check that Wayland is disabled (GDM3): verify `/etc/gdm3/custom.conf` has `WaylandEnable=false`
    - Restart both services:

        ```bash
        sudo systemctl restart gdm3
        sudo systemctl restart dcvserver
        ```

??? question "DCV service fails to start"
    Check the logs:

    ```bash
    journalctl -u dcvserver --no-pager -n 50
    ```

    Common causes:
    - Missing dummy display driver
    - Display manager not running
    - Port 8443 already in use

??? question "GPU shows as busy when it shouldn't be"
    Ensure you're using `type = "virtual"` in `/etc/dcv/dcv.conf`. Console sessions use the GPU for desktop rendering, which competes with ML workloads.
