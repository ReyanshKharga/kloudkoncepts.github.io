---
description: Discover the fundamentals of LimitRange in Kubernetes with our simple introduction. Learn how to set constraints on resource usage and maintain cluster efficiency.
---

# Introduction to LimitRange

In kubernetes, a `LimitRange` is an object that allows you to define `default` and `maximum` resource `limits` for pods and containers within a namespace.

The `LimitRange` ensures that pods adhere to the specified resource constraints on CPU, memory, and storage resources, helping to manage and allocate resources effectively without manual intervention for every pod.

When you create a `LimitRange` in a namespace, the defined `default` and `maximum` resource `limits` are automatically applied to pods within that namespace, even if you don't explicitly specify them.

Remember that `LimitRange` is applied at the namespace level, so it affects all containers running within that namespace unless overridden by explicit pod resource requests or limits.

it is recommended to avoid having multiple `LimitRange` objects in a namespace to prevent conflicts and confusion.



!!! quote "References:"
    !!! quote ""
        * [LimitRange in Kubernetes]{:target="_blank"}


<!-- Hyperlinks -->
[LimitRange in Kubernetes]: https://kubernetes.io/docs/concepts/policy/limit-range/