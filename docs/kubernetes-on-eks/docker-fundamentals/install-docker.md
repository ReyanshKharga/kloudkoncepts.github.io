---
description: Learn how to install Docker on your operating system. A step-by-step guide for seamless Docker setup.
---


# Install Docker

Docker is an open platform for developing, shipping, and running applications.

You can download and install Docker on the platform of your choice, including Mac, Linux, or Windows by following the platform-specific instructions provided below.

!!! warning
    Given the ever-changing nature of the installation process, it is advisable to rely on the official documentation when installing Docker.


## Docker Engine vs Docker Desktop

`Docker Engine` is the core software that allows you to create and run Docker containers, while `Docker Desktop` is a complete development environment that includes Docker Engine and several other tools and services for building and testing Docker applications on your local machine.

!!! note
    Docker Desktop is the only way to install the Docker Engine on `Windows` and `macOS` operating systems.

Docker Desktop is also available for `Linux`, although Linux users are free to install the Docker Engine separately.


## Install Docker on Mac

Docker Desktop Download URL: [Install Docker Desktop on Mac]{:target="_blank"}

You'll see two options:

1. Docker Desktop for Mac with Intel chip
2. Docker Desktop for Mac with Apple silicon

To find out which option to choose, click on the Apple logo on top left corner of the Mac and click on `About This Mac`. Search for a line that mentions either `Chip` or `Processor`. 

If you find `M1` or `M2` in that line, it means the computer is powered by Apple Silicon. Conversely, if you see the word `Intel`, it indicates the machine is equipped with an Intel-based Core series processor.


## Install Docker on Windows

Docker Desktop Download URL: [Install Docker Desktop on Windows]{:target="_blank"}

You may see a message that says: `WSL2 installation is incomplete`. Make sure `WSL2` is insatlled.

If during installation you see a checkbox that says Use `WSL2` instead of `Hyper-V` (recommended), make sure you check this checkbox.

Windows Subsystem for Linux (WSL) is a feature of Windows that allows developers to run a Linux environment without the need for a separate virtual machine or dual booting.

## Install Docker on Ubuntu (Linux)

### Method 1: Docker Desktop

Download Deb Package: [Install Docker Desktop on Ubuntu]{:target="_blank"}

* For non-Gnome Desktop environments, gnome-terminal must be installed:

    ```
    sudo apt install gnome-terminal
    ```

* Install the package with apt as follows:

    ```
    sudo apt-get update
    sudo apt-get install ./docker-desktop-<version>-<arch>.deb
    ```

### Method 2: Docker Engine

Docker Engine comes bundled with Docker Desktop for Linux. But if you want to install only Docker Engine and not the Docker Desktop, you can do so by following the instructions below.

Official Documentation URL: [Install Docker Engine on Ubuntu]{:target="_blank"}

Let's install Docker Engine using the repository.

**Step 1: Set up the repository**

1. Update the apt package index and install packages to allow apt to use a repository over HTTPS:

    ```
    sudo apt-get update

    sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```

2. Add Docker’s official GPG key:

    ```
    sudo mkdir -m 0755 -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    ```

3. Use the following command to set up the repository:

    ```
    echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

**Step 2: Install Docker Engine**

1. Update the apt package index:

    ```
    sudo apt-get update
    ```

2. Install latest version of Docker Engine, containerd, and Docker Compose:

    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

3. Verify that the Docker Engine installation is successful by running the hello-world image:

    ```
    sudo docker run hello-world
    ```

!!! quote "References:"
    !!! quote ""
        * [Install Docker Desktop on Mac]{:target="_blank"}
        * [Install Docker Desktop on Windows]{:target="_blank"}
        * [Install Docker Desktop on Ubuntu]{:target="_blank"}
        * [Install Docker Engine on Ubuntu]{:target="_blank"}


<!-- Hyperlinks -->
[Install Docker Desktop on Mac]: https://docs.docker.com/desktop/install/mac-install/
[Install Docker Desktop on Windows]: https://docs.docker.com/desktop/install/windows-install/
[Install Docker Desktop on Ubuntu]: https://docs.docker.com/desktop/install/ubuntu/
[Install Docker Engine on Ubuntu]: https://docs.docker.com/engine/install/ubuntu/