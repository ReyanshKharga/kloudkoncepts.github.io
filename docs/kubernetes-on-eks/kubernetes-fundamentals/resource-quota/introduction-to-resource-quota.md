---
description: Learn the basics of Resource Quota in Kubernetes with our straightforward introduction. Understand how to set limits on resource usage and maintain control over your cluster's capacity.
---

# Introduction to Resource Quota

In a shared cluster with limited nodes, it's important to prevent one team from hogging all the resources. That's where resource quotas come in.

Think of a `ResourceQuota` like a set of rules set by the administrators. These rules determine how much of the cluster's resources each team or user can use.

For example, a resource quota can limit the number of objects (like `pods` or `services`) that can be created in a specific namespace. It can also put a cap on the total amount of computing power (CPU, memory) that can be used by the resources within that namespace.

By setting these limits, administrators ensure that everyone gets their fair share of resources and that the cluster operates smoothly without any team monopolizing all the power.


!!! quote "References:"
    !!! quote ""
        * [Resource Quotas]{:target="_blank"}


<!-- Hyperlinks -->
[Resource Quotas]: https://kubernetes.io/docs/concepts/policy/resource-quotas/