# WSL2 Setup for ML Development

This guide walks through setting up **Windows Subsystem for Linux 2 (WSL2)** as a development environment for GPU-accelerated machine learning workflows.

!!! info "Setup for windows only"

    This guide is specific to windows and can be skipped if you are using Ubuntu.

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
* Ubuntu          Running         2
```

---

## Resource Configuration

### Memory and CPU Limits (`.wslconfig`)

By default, WSL2 can use up to **50% of your system RAM** and **all CPU cores**. For ML workloads you'll want to tune this to balance between WSL and Windows.

Create or edit `%USERPROFILE%\.wslconfig` (e.g., `C:\Users\YourName\.wslconfig`):

```ini
[wsl2]
memory=24GB
processors=28
swap=4GB
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
sparseVhd=true
```

#### Setting Breakdown

##### **Memory**

Maximum RAM that WSL2 can use. Leave at least **4–6 GB for Windows** to avoid system slowdowns.

| Total System RAM | Recommended WSL2 **memory** |
| ---------------- | --------------------------- |
| 16 GB            | 10–12 GB                    |
| 32 GB            | 24–28 GB                    |
| 64 GB            | 48–56 GB                    |

##### **Processors**

Number of **logical CPU cores** to allocate to WSL2. To check how many you have, run in PowerShell:

```powershell
(Get-CimInstance Win32_Processor).NumberOfLogicalProcessors
```

**Rule of thumb:** Leave **2–4 cores for Windows** and give the rest to WSL2. For example, on a 16-core machine, set `processors=12`.

##### **Swap**

Swap acts as a **safety net** — if your training job temporarily exceeds the **memory** limit, it spills to disk instead of being **killed immediately** (OOM). Setting `swap=0` disables this safety net entirely.

!!! warning "Don't disable swap for ML workloads"

    While you never _want_ training to run on swap (it would be very slow), having a small swap buffer (2–4 GB) prevents your long-running training job from being killed if it briefly exceeds the **memory** limit. Think of it as insurance, not working memory.

##### **LocalhostForwarding**

When set to **true**, any service running inside WSL2 (e.g., Jupyter on port `8888`, TensorBoard on `6006`) is automatically accessible from **Windows** at `localhost:<port>`.
Without this, you'd need to find WSL2's internal virtual IP (`hostname -I` inside WSL) and use that address instead, which changes every time WSL restarts.

##### **AutoMemoryReclaim**

Set to **gradual** so WSL2 **gradually returns unused memory** to Windows over time. Without this, WSL2 holds onto all memory it has ever used, even after processes finish.

##### **SparseVhd**

WSL2 stores its entire Linux filesystem in a single virtual hard disk file (`.vhdx`). By default, this file **only grows** — if you download 10 GB of data and then delete it, the `.vhdx` file remains 10 GB on your Windows drive.

With **sparseVhd=true**, the `.vhdx` file **automatically shrinks** when space is freed inside WSL, so it only uses the actual disk space needed.

!!! tip "Applying Changes"

    After editing `.wslconfig`, restart WSL2:

    ```powershell
    wsl --shutdown
    ```

    Then reopen your Ubuntu terminal.

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

!!! warning "Do NOT Install Linux NVIDIA Drivers"

    Installing `nvidia-driver-*` packages inside WSL2 will **conflict** with the Windows driver passthrough and break GPU access. Only install the driver on the Windows side.

---

## VS Code Remote — WSL Integration

!!! warning "Do NOT open WSL paths directly from Windows"

    Opening VS Code on Windows and navigating to a `\\wsl$\Ubuntu\...` path **does not** use the WSL extension. VS Code will run entirely on the **Windows side**, accessing Linux files over the network — this makes file operations, IntelliSense, and builds **significantly slower**. Always connect to WSL using one of the methods below.

### Clone the Repository inside WSL

Open your **WSL terminal** (Ubuntu) and clone the project into the Linux filesystem:

```bash
# cd to the desired clone location
git clone <repository-url>
```

!!! tip "Always store projects on the Linux filesystem"

    Keep your code under `/home/<user>/` inside WSL — **not** on `/mnt/c/` or any Windows mount. Cross-filesystem access is significantly slower.

### Connect VS Code to WSL

1. Open VS Code on Windows normally
2. Press ++ctrl+shift+p++ to open the **Command Palette**
3. Type **`WSL: Connect to WSL`** and select it — VS Code will reload connected to your WSL instance
4. Once connected (you'll see **WSL: Ubuntu** in the bottom-left corner), use **File → Open Folder** and navigate to your project:

!!! tip "Open Folder directly in WSL"

    You can also use **`WSL: Open Folder in WSL...`** from the Command Palette to connect and open a folder in a single step.

### Recommended Extensions

Install these extensions **inside the WSL session** (they need to run on the Linux side):

| Extension       | Purpose                                |
| --------------- | -------------------------------------- |
| Dev Containers  | Open projects inside Docker containers |
| Container Tools | Container management from the sidebar  |

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
