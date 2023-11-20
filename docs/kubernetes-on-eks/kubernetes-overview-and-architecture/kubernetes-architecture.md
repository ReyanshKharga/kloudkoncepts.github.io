---
description: Dive into the architecture of Kubernetes with our informative guide. Understand the inner workings and components that power container orchestration.
---

# Architecture of Kubernetes

Kubernetes has a master-slave architecture. The "master" controls and manages the cluster, while the "slaves" (also known as nodes) are the worker machines where applications run. The master ensures that applications are deployed, scaled, and maintained as per your specifications, making it a powerful system for container management.


## Components of Kubernetes

<p align="center">
    <img src="../../../assets/eks-course-images/kubernetes-overview-and-architecture/components-of-kubernetes.svg" alt="Components of Kubernetes" />
</p>

### 1. Control Plane

The control plane is the brain of the Kubernetes system, responsible for managing and orchestrating all of the cluster's resources.

It consists of several key components, including the Kubernetes `API server`, `etcd`, `controller manager`, `scheduler`, and `cloud controller manager`.

Control Plane Components:

1. **API server:** API server exposes the Kubernetes API which is used to interact with the cluster.
2. **Etcd:** The `etcd` is a distributed key-value store that stores the state and configuration data.
3. **Controller manager:** The controller manager monitors the state of the cluster and makes required changes.

    The Controller Manager includes several essential components:

    - `Node Controller`: Monitors and manages nodes' health and status.
    - `Replication Controller`: Maintains the desired number of pod replicas.
    - `Endpoint Controller`: Populates endpoint resources with pod IPs for services.
    - `Service Account Controllers`: Manage service accounts and associated tokens.
    - `Service Controller`: Manages services and their load-balanced endpoints.
    - `Job Controller`: Manages batch jobs, ensuring they run to completion.
    - `Namespace Controller`: Handles the lifecycle and policies related to namespaces within the cluster.

    These controllers work to ensure the cluster's resources align with the desired state specified in kubernetes configurations, aiding in self-healing and maintaining stability.

4. **Scheduler:** The `scheduler` schedules the containers to run on the worker nodes.
5. **Cloud controller manager:** The `cloud controller manager` manages the underlying cloud infrastructure.


### 2. Nodes

Nodes are the worker machines that run containerized applications in a Kubernetes cluster. Each node runs a container runtime, such as Docker, and is managed by the control plane. 

Nodes communicate with the control plane via the Kubernetes API and are responsible for running and managing containers. Nodes also run several key components, including the `kubelet`, `kube-proxy`, and `container runtime`.

Node Components:

1. **Kubelet:** The `kubelet` is an agent that runs on each node in the cluster and is responsible for managing the state of the pods running on that node. Its main responsibility is to ensure that containers are running and healthy.

2. **Kube proxy:** The `kube-proxy` is responsible for managing the network connectivity between the pods and services in the cluster.

3. **Container runtime:** The `container runtime` is responsible for starting and stopping containers on the node. Kubernetes supports multiple container runtimes, including Docker, containerd, and CRI-O.


### 3. Cloud Provider API

The cloud provider API is an optional component of Kubernetes that allows Kubernetes to interact with cloud provider-specific services, such as load balancers, storage, and networking. 

Cloud provider APIs are implemented by cloud providers and enable Kubernetes to provision and manage cloud resources directly from within the Kubernetes system.


## Kubernetes Add-ons

Addons are optional components that can be installed to extend the functionality of a Kubernetes cluster.

Addons can provide additional features or services that are not part of the core Kubernetes platform.

Some examples of Kubernetes add-ons include `Dashboard`, `DNS`, `Ingress controller`, and `Metrics server`.

Add-ons are typically installed as Kubernetes resources, such as `deployments` or `daemonsets`, and are managed by Kubernetes controllers.

Add-ons can be installed using various tools, such as `kubectl` or `Helm` charts.

While add-ons are optional, they may be necessary for certain Kubernetes configurations or workloads. For example, all Kubernetes clusters should have cluster DNS Addon.


!!! quote "References:"
    !!! quote ""
        * [Kubernetes Components]{:target="_blank"}



<!-- Hyperlinks -->
[Kubernetes Components]: https://kubernetes.io/docs/concepts/overview/components/