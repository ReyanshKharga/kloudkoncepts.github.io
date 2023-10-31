---
description: Get started with an easy introduction to Service Accounts in Kubernetes. Learn how to manage identity and access control for your workloads.
---

# Introduction to Service Account

A `Service Account` in kubernetes provides an identity for processes that run in a pod.

You use kubernetes `Service Accounts` when you need to manage kubernetes resources (nodes, pods, deployments etc.) from inside a pod.

To access kubernetes `API Server` you need an `authentication token`. The processes that are running inside your containers use a `Service Account` to authenticate with the `API server`.

Just like a user account represents a human, a `Service Account` represents and provides an identity to your pods.

Each pod you create has a `Service Account` assigned to it even if you don't explicitly provide a `Service Account` name. If you don't provide the `Service Account` name ubernetes will assign a default `Service Account` for the pod.


!!! quote "References:"
    !!! quote ""
        * [Kubernetes Service Accounts]{:target="_blank"}


<!-- Hyperlinks -->
[Kubernetes Service Accounts]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/