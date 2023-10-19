---
description: Learn how to install eksctl for easy Amazon EKS management with our step-by-step guide.
---


# Install eksctl

`eksctl` is a simple command line tool for creating and managing Kubernetes clusters on Amazon EKS.

`eksctl` provides the fastest and easiest way to create a new cluster with nodes for Amazon EKS.

Let's see how you can install `eksctl` on your operating system.

!!! warning
    Given the ever-changing nature of the installation process, it is advisable to rely on the official documentation when installing `eksctl`.

    You can visit the [official documentation]{:target="_blank"} and follow the instructions to install `eksctl` on your operating system.



## Step 1: Install eksctl

### Install eksctl on MacOS

You can install or upgrade `eksctl` on MacOS using the following command:

```
brew upgrade eksctl && { brew link --overwrite eksctl; } || { brew tap weaveworks/tap; brew install weaveworks/tap/eksctl; }
```

### Install eksctl on Windows

You can use Chocolatey to install `eksctl` on Windows. If you do not already have Chocolatey installed on your Windows system, see [Installing Chocolatey]{:target="_blank"}.

Install `eksctl`:

```
choco install -y eksctl 
```

If you already have `eksctl` installed, you can upgrade it using the followign command:

```
choco upgrade -y eksctl 
```

### Install eksctl on Unix

1. Download and extract the latest release of `eksctl` with the following command:

    ```
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    ```

2. Move the extracted binary to `/usr/local/bin`:

    ```
    sudo mv /tmp/eksctl /usr/local/bin
    ```


## Step 2: Verify eksctl Installation

```
eksctl version
```

The above command should print the version of the eksctl installed on your system.



!!! quote "References:"
    !!! quote ""
        * [Install eksctl on Unix]{:target="_blank"}
        * [Install eksctl on Windows]{:target="_blank"}
        * [Install eksctl on MacOs]{:target="_blank"}


<!-- Hyperlinks -->
[official documentation]: https://eksctl.io/introduction/#installation
[Install eksctl on Unix]: https://eksctl.io/introduction/#for-unix
[Install eksctl on Windows]: https://eksctl.io/introduction/#for-windows
[Install eksctl on MacOs]: https://eksctl.io/introduction/#for-macos
[Installing Chocolatey]: https://chocolatey.org/install