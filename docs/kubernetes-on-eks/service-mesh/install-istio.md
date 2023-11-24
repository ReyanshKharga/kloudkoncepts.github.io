---
description: Learn how to effortlessly install Istio on your EKS Kubernetes cluster with our step-by-step guide. Enhance your cluster's capabilities and streamline management.
---

# Install Istio in EKS Kubernetes Cluster

Now that we know what Istio is and how it works, let's install it in our EKS kubernetes cluster. Later on, we'll explore its features more deeply in the following sections.


## Step 1: Download Istio

1. Download installation file:

    ```
    curl -L https://istio.io/downloadIstio | sh -
    ```

    !!! tip
        The command above downloads the latest release (numerically) of Istio. You can pass variables on the command line to download a specific version or to override the processor architecture. For example, to download Istio 1.20.0 for the x86_64 architecture, run:

        ```
        curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.20.0 TARGET_ARCH=x86_64 sh -
        ```

2. Change directory to the Istio package directory. For example, if the package is `istio-1.20.0`:

    ```
    cd istio-1.20.0
    ```

    Verify that the directory contains `bin/` folder. This contains the `istioctl` client binary.

    !!! note
        `istioctl` is a command-line utility used to configure and manage Istio service mesh within kubernetes environments.


3. Add `istioctl` client to your path:

    ```
    # Add to path temporarily
    export PATH=$PWD/bin:$PATH

    {OR}

    # Add to path permanently
    sudo cp bin/istioctl /usr/bin/
    ```

4. Verfy `istioctl` version:

    ```
    istioctl version --remote=false
    ```

    If everything is fine it should print out the `istioctl` version.



## Step 2: Install Istio

1. Check if your EKS cluster is ready to install Istio:

    ```
    istioctl x precheck
    ```

    If your EKS cluster passes the pre-installation verification check you should see the following output:

    ```
    ✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
    To get started, check out https://istio.io/latest/docs/setup/getting-started/
    ```

2. List available istio configuration profiles:

    Istio [configuration profiles]{:target="_blank"} are predefined sets of configuration settings that allow users to easily customize and fine-tune Istio's behavior within their service mesh.

    ```
    # List configuration profiles
    istioctl profile list
    ```

    This lists all the configuration profiles that can be used while installing istio.

    In this course, we will use the `default` profile, which is recommended for production deployment.


3. Install Istio:

    Let's install istio with configuration from `default` profile.

    ```
    istioctl install --set profile=default -y
    ```

    If istio is installed successfully, you should see the output similar to below:


    ```
    ✔ Istio core installed
    ✔ Istiod installed
    ✔ Egress gateways installed
    ✔ Ingress gateways installed
    ✔ Installation complete
    ```

4. Verify Istio installation:

    Istio components are deployed in `istio-system` namespace. Let's list pods in `istio-system` namespace.

    ```
    kubectl get pods -n istio-system
    ```

    Verify if all the pods are running.

    !!! note
        `Istiod` makes up the control plane and is responsible for injecting the sidecar proxies into our services that are part of the mesh.


## Uninstall Istio

If you no longer need Istio in your EKS kubernetes cluster, you can uninstall it using following command:

```
# Uninstall istio
istioctl uninstall --purge
```

This will remove all istio components from your kubernetes cluster.



!!! quote "References:"
    !!! quote ""
        * [Download Istio]{:target="_blank"}
        * [Istio Configuration Profiles]{:target="_blank"}


<!-- Hyperlinks -->
[configuration profiles]: https://istio.io/latest/docs/setup/additional-setup/config-profiles/
[Download Istio]: https://istio.io/latest/docs/setup/getting-started/
[Istio Configuration Profiles]: https://istio.io/latest/docs/setup/additional-setup/config-profiles/