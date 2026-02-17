# PhysicsNeMo Docker Setup

***

## OS Requirements

* Ubuntu Kinetic 25.10
* Ubuntu Noble 24.04 (LTS)
* Ubuntu Jammy 22.04 (LTS)

***

## Docker

Docker Engine is the core runtime software responsible for building and executing containers by directly sharing the host operating system's kernel. It isolates applications into lightweight, portable units that bundle all necessary code and dependencies, ensuring they run consistently in any environment without the heavy overhead of a full virtual machine.

---

### Setup Docker Engine

#### Uninstall old/conflicting versions

The unofficial packages to uninstall are:

    docker.io
    docker-compose
    docker-compose-v2
    docker-doc
    podman-docker

Run the following command to uninstall all conflicting packages if any:

``` bash
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 /
docker-doc podman-docker containerd runc | cut -f1)
```

#### Install using the `apt` repository

1. Set up Docker's `apt` repository:

    Add Docker's official GPG key:

    ``` bash
    sudo apt update
    sudo apt install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    ```

    Add the repository to `apt` sources:

    ``` bash
    sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
    Types: deb
    URIs: https://download.docker.com/linux/ubuntu
    Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
    Components: stable
    Signed-By: /etc/apt/keyrings/docker.asc
    EOF
    ```

    Update `apt`:

    ``` bash
    sudo apt update
    ```

2. Install the Docker packages:

    ``` bash
    sudo apt install docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin
    ```

3. Verify that Docker is running:

    ``` bash
    sudo systemctl status docker
    ```

    If the service is not active, start it:

    ``` bash
    sudo systemctl start docker
    ```

4. Verify that the installation is successful by running the `hello-world` image:

    ``` bash
    sudo docker run hello-world
    ```

    This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

    *SUCCESS: You have now successfully installed and started Docker Engine.*

### Post-installation steps for Docker Engine

#### Manage Docker as a non-root user

*WARNING: The docker group grants root-level privileges to the user. Only add trusted users to this group.*

If you don't want to preface the docker command with sudo, create a Unix group called docker and add users to it.

1. Create the `docker` group.

    ``` bash
    sudo groupadd docker
    ```

2. Add your user to the docker group.

    ``` bash
    sudo usermod -aG docker $USER
    ```

3. Activate the changes to groups.

    ``` bash
    newgrp docker
    ```

4. Verify that you can run `docker` commands without `sudo`.

    ``` bash
    docker run hello-world
    ```

    This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

#### Configure Docker to start on boot with systemd

*INFO: On Ubuntu, the Docker service will start by default. Use these commands only if you need to manually enable or disable this behavior.*

To enable Docker to start on boot:

``` bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

To disable this behavior:

``` bash
sudo systemctl disable docker.service
sudo systemctl disable containerd.service
```

***

## NVIDIA Container Toolkit

The NVIDIA Container Toolkit is a runtime extension that enables containers to directly utilize the host system's NVIDIA GPUs for accelerated computing. It integrates with the Docker Engine to automatically mount the necessary GPU drivers and libraries into the container at launch, eliminating the need to install specific drivers inside the container itself

---

### Install NVIDIA Container Toolkit

1. Install the prerequisites:

    ``` bash
    sudo apt-get update && sudo apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2
    ```

2. Configure the production repository:

    ``` bash
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg \
    --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && curl \
    -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
    | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    ```

3. Update the packages list from the repository:

    ``` bash
    sudo apt-get update
    ```

4. Install the NVIDIA Container Toolkit packages:

    ``` bash
    export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.18.2-1
    sudo apt-get install -y \
    nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
    ```

### Configure with Docker

1. Configure the container runtime by using the `nvidia-ctk` command:

    ``` bash
    sudo nvidia-ctk runtime configure --runtime=docker
    ```

2. Restart the Docker daemon:

    ``` bash
    sudo systemctl restart docker
    ```

3. Verify your installation by running a sample workload:

    ``` bash
    docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
    ```

    Your output should resemble the following:

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

***

## PhysicsNeMo

NVIDIA PhysicsNeMo is an open-source framework for developing physics-informed machine learning models that blend physical laws with observational data. It enables the creation of high-speed AI surrogates to simulate complex engineering problems thousands of times faster than traditional numerical solvers.

---

### Get the container

Download the PhysicsNeMo Docker container from NGC using:

``` bash
docker pull nvcr.io/nvidia/physicsnemo/physicsnemo:<tag>
```

* All the available tags can be found on [PhysicsNeMo NGC Container](https://catalog.ngc.nvidia.com/orgs/nvidia/teams/physicsnemo/containers/physicsnemo/tags).

### Launching shell sessions using the container

*NOTE: Replace `<tag>` with the tag installed in the previous step.*

#### Start a shell session

``` bash
docker run --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 \
--runtime nvidia -it --rm nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> bash
```

#### Start shell session with the current directory mounted

*TIP: Mounting your current directory is useful for accessing your work files from within the container.*

``` bash
docker run --shm-size=1g --ulimit memlock=-1 --ulimit stack=67108864 \
--runtime nvidia -v ${PWD}:/workspace \
-it --rm nvcr.io/nvidia/physicsnemo/physicsnemo:<tag> bash
```
