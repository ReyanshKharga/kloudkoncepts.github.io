---
description: Explore the fundamentals of Storage in Kubernetes with our introductory guide. Learn the key concepts and benefits of storage for containerized applications.
---


# Introduction to Storage in Kubernetes

Let's explore the fundamentals of storage in kubernetes and the key concepts and benefits of storage for containerized applications.


## What is Volume in Kubernetes?

Kubernetes offers a flexible storage solution for managing data across containers and nodes.

In kubernetes, a volume is an abstraction that represents a storage device that can be mounted to a container in a pod.

At its core, a volume is a directory, possibly with some data in it, which is accessible to the containers in a pod.

Volumes are an essential component of kubernetes, as they provide a way to store and share data between containers in a pod.

Volumes are also an essential part of deploying stateful applications in kubernetes, as they provide a way to persist data beyond the lifecycle of a container.



## Ephemeral vs Persistent Storage

Kubernetes storage can be classified into two main types:

1. Ephemeral storage
2. Persistent storage

Ephemeral storage is short-term storage that is tied to the lifecycle of a container. It is used to store data that is only needed for the duration of a container's life. When the container is deleted, the ephemeral storage is also deleted. Ephemeral storage is often used for caching, temporary files, or as scratch space.

Persistent storage, on the other hand, is used to store data that needs to persist beyond the lifecycle of a container. This type of storage is used to store data that needs to be shared between multiple containers or to support stateful applications.


## Persistent Storage Options in Kubernetes

Kubernetes provides several options for persistent storage, including local storage, network-attached storage, and cloud-based storage solutions, such as Amazon EBS or Google Persistent Disks.

1. **Local Storage**

    Local storage is storage that is directly attached to a node in the cluster. This can be a physical disk or a virtual disk created using a local volume. Local storage is often used for applications that require high-performance storage or for applications that need to store data that is not shared between containers.

2. **Network Attached Storage**

    Network attached storage, or NAS, is a type of storage that is accessed over a network. Kubernetes supports several types of `NAS`, including `NFS`, `GlusterFS`, and `CephFS`. `NAS` is often used for applications that require shared storage, such as databases or file servers.

3. **Cloud Based Storage**

    Cloud based storage solutions, such as `Amazon EBS` or `Google Persistent Disks`, provide scalable and highly available storage that can be used to support stateful applications or to provide shared storage for multiple containers running in a kubernetes environment.

In summary, kubernetes provides a flexible and extensible storage framework that allows you to manage persistent data across multiple containers and nodes in a kubernetes cluster. With kubernetes storage, you can choose the right storage solution for your application's needs, whether it is local storage, network-attached storage, or cloud-based storage solutions.


## Types of Volumes

Kubernetes supports several types of volumes. The most commonly used volume types are as follows:

- `emptyDir`
- `hostPath`
- `AWS EBS CSI`
- `azureDisk CSI`
- `GCE CSI`

This is just a small list of most commonly used volume types. Here's a full list of supported volume types.


This is a small list of the most commonly used volume types. For a complete list of [supported volume types]{:target="_blank"} in kubernetes, please refer to the documentation.


!!! quote "References:"
    !!! quote ""
        * [Types of Volumes in Kubernetes]{:target="_blank"}



<!-- Hyperlinks -->
[supported volume types]: https://kubernetes.io/docs/concepts/storage/volumes/#volume-types
[Types of Volumes in Kubernetes]: https://kubernetes.io/docs/concepts/storage/volumes/#volume-types