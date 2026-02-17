# Setup DCV for NVIDIA Optimized VMI

This setup gives you a lightweight CPU-rendered desktop (perfect for VS Code/Debugging) while leaving your NVIDIA GPU 100% free and dedicated to your Machine Learning tasks.

## System Prep & Drivers

1. Update and Install GUI (Minimal)

``` bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ubuntu-desktop-minimal
```

2. Install NVIDIA Drivers

``` bash
sudo
```

3. Install Dummy Driver (Required for Virtual Sessions)

``` bash
sudo apt install -y xserver-xorg-video-dummy
```

4. Reboot

``` bash
sudo reboot
```

## Install NICE DCV (The Complete Suite)

1. Import GPG Key

``` bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/NICE-GPG-KEY
gpg --import NICE-GPG-KEY
```

2. Download DCV 2025.0 (Ubuntu 24.04 version)

``` bash
wget https://d1uj6qtbmh3dt5.cloudfront.net/2025.0/Servers/nice-dcv-2025.0-20103-ubuntu2404-x86_64.tgz
tar -xvzf nice-dcv-2025.0-20103-ubuntu2404-x86_64.tgz
cd nice-dcv-2025.0-20103-ubuntu2404-x86_64
```

3. Install All Packages

This installs the `Server`, `Web Viewer`, `GPU shim`, and `X drivers` in one go.

``` bash
sudo apt install ./nice-dcv-server_*.deb ./nice-dcv-web-viewer_*.deb ./nice-dcv-gl_*.deb ./nice-xdcv_*.deb -y
```

3. Configure User Access

``` bash
# Add ubuntu user to video group
sudo usermod -aG video ubuntu

# Set a password for the GUI login (if you haven't already)
sudo passwd ubuntu
```

## Choose Your Display Manager

* GDM3 (Default - Easiest)

>Pros: Already installed.

>Cons: Heavy, can be buggy with NVIDIA drivers.

>Configuration: You must disable Wayland for stability.

``` bash
sudo sed -i 's/#WaylandEnable=false/WaylandEnable=false/' /etc/gdm3/custom.conf
```

* LightDM (Recommended - Most Stable)

>Pros: Lightweight, ignores missing monitors, very reliable for headless servers.

>Cons: Requires installation.

``` bash
sudo apt install lightdm -y
# (Select 'lightdm' when the pink screen appears)
```
Stick with (GDM3) first since we are using Virtual Mode. If you ever have boot issues, switch to LightDM.

## Configure Automatic "Virtual" Session

1. Edit the DCV Configuration File

``` bash
sudo nano /etc/dcv/dcv.conf
```
2. Make the following changes:

Section: `[session-management]`

>Uncomment/Add: `create-session = true`

Section: `[session-management/defaults]`

>Uncomment/Add: `owner = "ubuntu"`

>Uncomment/Add: `type = "virtual"`

Your file should look roughly like this (comments removed for clarity):

```
[session-management]
create-session = true

[session-management/defaults]
owner = "ubuntu"
type = "virtual"
```

3. Save and Exit

* Press Ctrl+O, Enter (Save).

* Press Ctrl+X (Exit).

## Final Enable & Start

``` bash
# Enable the service to start on boot
sudo systemctl enable dcvserver

# Restart the service to apply the config changes
sudo systemctl restart dcvserver
```

## Useful commands for debugging

``` bash
sudo lshw -c video
```

``` bash
sudo dcvgldiag
```

``` bash
nvidia-smi
```

``` bash
sudo dcv create-session --type virtual --owner ubuntu session1
```

``` bash
dcv list-sessions
```