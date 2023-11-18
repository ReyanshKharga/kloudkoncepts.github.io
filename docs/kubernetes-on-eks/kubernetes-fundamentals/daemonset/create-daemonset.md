---
description: Learn how to efficiently deploy and oversee Kubernetes DaemonSets. Master pod distribution across your cluster nodes with our comprehensive guide on creating and managing DaemonSets.
---

# Create and Manage Kubernetes DaemonSets

Let's see how you can create and manage kubernetes Daemonsets. For this demonstration, let's assume there is a need to run an nginx server on each node. In the real world, your requirements might differ depending on your use case.


## Step 1: Create DaemonSet Manifest

First, we need to write the DaemonSet manifest as follows:

=== ":octicons-file-code-16: `my-daemonset.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: nginx-daemonset
    spec:
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80

    ```

Required fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a name string, UID , and - optional namespace.
- `spec` - What state you desire for the object.


## Step 2: Create DaemonSet

Let's use `kubectl apply` to apply the manifest and create the DaemonSet:

```
# Create daemonset
kubectl apply -f my-daemonset.yml
```

## Step 3: List DaemonSets and Pods

```
kubectl get daemonsets
{OR}
kubectl get daemonset
{OR}
kubectl get ds
```

!!! note
    `daemonset` is abbreviated as `ds`.

Also, list pods and verify if they are running on all the nodes in the cluster:

```
kubectl get pods -o wide
```

You'll that a copy of the pod is running on all the nodes in the cluster.


## Step 4: Delete DaemonSet

```
# Delete the daemonset
kubectl delete -f my-daemonset.yml
```

This will delete the DaemonSet and pods managed by the DaemonSet.



!!! quote "References:"
    !!! quote ""
        * [DaemonSet]{:target="_blank"}



<!-- Hyperlinks -->
[DaemonSet]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/