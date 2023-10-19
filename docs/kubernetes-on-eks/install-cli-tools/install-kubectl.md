---
description: Learn how to install kubectl, the Kubernetes command-line tool, with our step-by-step guide for hassle-free setup.
---

# Install kubectl

The Kubernetes command-line tool, `kubectl`, allows you to run commands against Kubernetes clusters.

With `kubectl`, users can create, modify, and delete Kubernetes resources such as pods, deployments, services, and namespaces.

Let's see how you can install `kubectl` on your operating system.

!!! warning
    Given the ever-changing nature of the installation process, it is advisable to rely on the official documentation when installing `kubectl`.

    You can visit the [official documentation]{:target="_blank"} and follow the instructions to install `kubectl` on your operating system.


## Step 1: Install Kubectl

### Install kubectl on MacOS

You can install kubectl with homebrew as follows:

```
brew install kubectl
```


### Install kubectl on Windows

Follow the official documentation on how to [Install and Set Up kubectl on Windows]{:target="_blank"}. This will ensure you have the latest release installed on your system.


### Install kubectl on Linux

1. Download the latest release with the command:

    ```
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

2. Validate the binary (optional):

    ```
    # Download the kubectl checksum file
    curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

    # Validate the kubectl binary against the checksum file
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
    ```

3. Install kubectl:

    ```
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    ```



## Step 2: Verify Kubectl Installation

Run the following command to verify if `kubectl` is installed:

```
kubectl version --client --output=yaml
```

The output should look similar to the below:

```yaml
clientVersion:
  buildDate: "2022-06-27T22:22:16Z"
  compiler: gc
  gitCommit: b77d9473a02fbfa834afa67d677fd12d690b195f
  gitTreeState: clean
  gitVersion: v1.23.7-eks-4721010
  goVersion: go1.17.10
  major: "1"
  minor: 23+
  platform: linux/amd64
```


!!! quote "References:"
    !!! quote ""
        * [Install and Set Up kubectl on Linux]{:target="_blank"}
        * [Install and Set Up kubectl on Windows]{:target="_blank"}
        * [Install and Set Up kubectl on macOS]{:target="_blank"}


<!-- Hyperlinks -->
[official documentation]: https://kubernetes.io/docs/tasks/tools/
[Install and Set Up kubectl on Linux]: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
[Install and Set Up kubectl on Windows]: https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
[Install and Set Up kubectl on macOS]: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/