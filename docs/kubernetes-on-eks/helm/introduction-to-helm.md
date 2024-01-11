---
description: Unlock the power of Helm with our comprehensive guide. Dive into the world of Kubernetes package management and streamline your deployment process. 
---

# Introduction to Helm

Helm is the package manager for Kubernetes.

This means you can package your kubernetes YAML manifest files and distribute them through public or private helm repositories.

You know that:

- `apt` is a package manager for Debian, and Debian-based Linux distributions, yum
- `homebrew` is a package manager for Apple's operating system, macOS

Helm does the similar job but for Kubernetes packages.

A `Chart` is a Helm package. Think of it like the Kubernetes equivalent of a homebrew formula, an apt dpkg, or a yum RPM file.



## Example Use Case of Helm Chart

Consider you need to deploy prometheus in your EKS cluster. Prometheus is a stateful application and has a lot of components.

### Method 1: Write YAML manifest files for all the componets

This is going to be a cumbersome task since you need to look for the manifest files for all components on the internet. You may need to edit all the manifest to meet your needs. For example you might want to change namespace in all the manifest.

What about updates? What if something changed in prometheus architecture? Incorporating those changes in your manifest is going to be a nightmare.


### Method 2: Use Helm Chart

What if someone has already packaged all the required YAML manifests together and published it as a package (chart in helm) in some repository?

This will make your life easier. You just need to install the chart and you are good to go.

You can also replace values easily. For example, if you want to deploy the prometheus resources in a particular namesapce, you can do that just by setting the required placeholder with the value you desire. (Remember set parameter?)



## Install Helm CLI

See the [installation guide]{target="_blank"} to instlal `Helm CLI` on your operating system.


Verify if helm is installed:

```
helm version
```

The output will be similar to the below:

```yaml
version.BuildInfo{Version:"v3.11.1", GitCommit:"293b50c65d4d56187cd4e2f390f0ada46b4c4737", GitTreeState:"clean", GoVersion:"go1.18.10"}
```


Now, let's see how you can work with Helm CLI to install charts in your kubernetes cluster.


## Prerequisites

You must have the following in place before you can start working with Helm:

- A Kubernetes Cluster
- A local configured copy of `kubectl`


## Initialize a Helm Chart Repository

Once you have `Helm CLI` installed, you can add a chart repository. A chart repository is a server that houses packaged charts.

For example, `Helm hub` is the official public repository for Helm Charts.

Let's add `bitnami` helm chart repository as follows:

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

You can see which repositories are configured using the following command:

```
helm repo list
```

Now that you have added the `bitnami` repo, you can search the charts that are published in that repository using `helm search` command as follows:

```
# Command template
helm search repo <repo-name>

# Example
helm search repo bitnami
```

It will output a list of Helm charts available in the `bitnami` Helm chart repository, along with details such as chart name, version, description, and other relevant information.


## Install a Chart

Now that we have added the repository. We can install charts from that repository.

1. Update the repo to make sure we get the latest list of charts:

    ```
    helm repo update

    ```

2. Install the chart:

    ```
    # Command template to install a chart
    helm install <name> <chart>

    # Example
    helm install mysql bitnami/mysql 
    ```

    The charts will be installed in the `default` namespace. You can use the `--namespace` or `-n` to install it in a particular namespace.

    ```
    # Create namespace mysql
    kubectl create namespace mysql

    # Install mysql in mysql namespace
    helm install mysql bitnami/mysql -n mysql
    ```

    You can get a basic idea of the features of this MySQL chart by running the following command:

    ```
    helm show chart bitnami/mysql -n <namespace>
    ```

Helm does not wait until all of the resources are running before it exits. Many charts require Docker images that are over 600M in size, and may take a long time to install into the cluster.

To keep track of a release's state, or to re-read configuration information, you can use helm status command as follows:

```
helm status mysql -n <namespace>
```

When installing Helm charts with the `helm install` command, the `--set` argument allows you to define values used in a chart’s templates.

For example, when installing prometheus using helm you can set the values as per your need as follows:

```
# Helm install with set argument
helm install prometheus prometheus-community/prometheus --namespace prometheus --set server.persistentVolume.size=20Gi --set server.retention=15d
```

Alternatively, you can also set values using a custom YAML file:

```
helm install prometheus prometheus-community/prometheus --namespace prometheus --values my-values.yml
```

Your` my-values.yml` may look like the below:

```yaml
server:
    persistentVolume:
        size: 20Gi
    retention: 15d
```


## Show Default values.yml for a Given Chart

You can use the following commmand to view the default values for a chart:

```
# Command template
helm show values <chart>

# Example
helm show values prometheus-community/prometheus
```


## Helm Releases

The `helm list` (or `helm ls`) command will show you a list of all deployed releases:

```
# Command template
helm list -n <namespace>

# List all deployed releases in default namespace
helm list

# List all deployed releases in prometheus namespace
helm list -n prometheus
```


## Uninstall a Release

Use the helm uninstall command to uninstall a release:

```
# Command template
helm uninstall <name> -n namespace

# Example
helm uninstall mysql
```


<!-- Hyperlinks -->
[installation guide]: https://helm.sh/docs/intro/install/