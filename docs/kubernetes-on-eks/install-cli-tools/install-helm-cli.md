---
description: Learn how to install the Helm CLI with our simple guide. Get started with Helm for efficient Kubernetes package management.
---


## Install Helm CLI

`Helm CLI` is a command-line tool that is used to interact with Helm, the package manager for Kubernetes.

It is used to install, manage, and upgrade applications and services on Kubernetes clusters using Helm charts.

Let's see how you can install `Helm CLI` on your operating system.

!!! warning
    Given the ever-changing nature of the installation process, it is advisable to rely on the official documentation when installing `Helm CLI`.

    You can visit the [official documentation]{:target="_blank"} and follow the instructions to install `Helm CLI` on your operating system.


## Step 1: Install Helm CLI

### Install Helm CLI on MacOS

You can install `Helm CLI` with Homebrew using the following command:

```
brew install helm
```


### Install Helm CLI on Windows

You can use Chocolatey to install `Helm CLI` on Windows. If you do not already have Chocolatey installed on your Windows system, see [Installing Chocolatey]{:target="_blank"}.

Install `Helm CLI`:

```
choco install kubernetes-helm
```

### Install Helm CLI on Debian/Ubuntu

You can install `Helm CLI` from Apt as follows:

```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## Step 2: Verify Helm Installation

```
helm version
```

If `Helm CLI` is installed on your system, it should produce an output similar to the below:

```json
version.BuildInfo{Version:"v3.12.3", GitCommit:"3a31588ad33fe3b89af5a2a54ee1d25bfe6eaa5e", GitTreeState:"clean", GoVersion:"go1.20.7"}
```


!!! quote "References:"
    !!! quote ""
        * [Install Helm CLI on MacOS]{:target="_blank"}
        * [Install Helm CLI on Windows]{:target="_blank"}
        * [Install Helm CLI on Debina/Ubuntu]{:target="_blank"}


<!-- Hyperlinks -->
[official documentation]: https://helm.sh/docs/intro/install/
[Install Helm CLI on MacOS]: https://helm.sh/docs/intro/install/#from-homebrew-macos
[Install Helm CLI on Windows]: https://helm.sh/docs/intro/install/#from-chocolatey-windows
[Install Helm CLI on Debina/Ubuntu]: https://helm.sh/docs/intro/install/#from-apt-debianubuntu
[Installing Chocolatey]: https://chocolatey.org/install