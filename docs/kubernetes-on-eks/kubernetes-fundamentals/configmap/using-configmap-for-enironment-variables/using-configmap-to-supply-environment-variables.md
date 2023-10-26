---
description: Leverage the power of ConfigMap to efficiently supply environment variables to your Kubernetes containers. Discover how to simplify and manage environment variables with our practical guide.
---

# Using ConfigMap to Supply Environment Variables

Let's see how we can use ConfigMap to supply environment variables to containers running inside in a pod.


## Step 1: Create a ConfigMap

Let's create a `ConfigMap` with data that contains the required environment variables.

=== ":octicons-file-code-16: `my-configmap.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: my-configmap
    data:
      foo1: bar1
      foo2: bar2
    ```

Apply the manifest to create ConfigMap:

```
kubectl apply -f my-configmap.yml
```


## Step 2: Verify ConfigMap

```
# List configmaps
kubectl get cm

# Describe the configmap
kubectl describe cm my-configmap
```

## Step 3: Create Pods That Uses Environment Variables