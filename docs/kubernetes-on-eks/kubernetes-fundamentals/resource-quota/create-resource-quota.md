---
description: Effortlessly create and manage Resource Quotas in Kubernetes with our step-by-step guide. Discover how to effectively set limits on resource usage, ensuring efficient cluster management.
---

# Create and Manage Resource Quotas

Let's see how we can create and manage Resource Quotas.


## Step 1: Create a Namespace

Let's create a namespace first:

=== ":octicons-file-code-16: `dev-namespace.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
      labels:
        name: dev
    ```

Apply the manifest to create the `dev` namespace:

```
kubectl apply -f dev-namespace.yml
```

Verify the namespace:

```
kubectl get ns
```


## Step 2: Create a Resource Quota

Let's create a Resource Quota for the `dev` namespace:

=== ":octicons-file-code-16: `dev-resource-quota.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: dev-resource-quota
      namespace: dev
    spec:
      hard:
        requests.cpu: "4" # equivalent to "4000m"
        limits.cpu: "8" # equivalent to "8000m"
        requests.memory: 10Gi
        limits.memory: 20Gi
        pods: "2"
    ```

The following resource types can be specified in the `.spec.hard` field:

| <div style="width:120px">Resource Name</div>     | Description |
| ----------------- | ----------- |
| <a>`limits.cpu`</a>      | Across all pods in a non-terminal state, the sum of CPU limits cannot exceed this value. |
| <a>`limits.memory`</a>   | Across all pods in a non-terminal state, the sum of memory limits cannot exceed this value. |
| <a>`requests.cpu`</a>   | Across all pods in a non-terminal state, the sum of CPU requests cannot exceed this value |
| <a>`requests.memory`</a> | Across all pods in a non-terminal state, the sum of memory requests cannot exceed this value. |
| <a>`pods`</a>            | The total number of pods in a non-terminal state that can exist in the namespace. A pod is in a terminal state if `.status.phase` in (`Failed`, `Succeeded`) is true. |

Apply the manifest to create the Resource Quota:

```
kubectl apply -f dev-resource-quota.yml
```


## Step 3: Verify the Resource Quota

List the Resource Quotas:

```
kubectl get quota -n dev
```

Describe the quota:

```
kubectl describe quota dev-resource-quota -n dev
```

When you view the quota details, you will find the specific hard limit set for each resource, as well as the current usage of those resources.


## Step 4: Create Pods and Test the Quota

In our example, we can't have more than `2` pods running in the `dev` namepsace and aggregate `requests.cpu` across all pods in the `dev` namespace can't exceed `8`. The same is the case for other constraints specified in the resource quota.

Let's create a deployment and test this out:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
      namespace: dev
    spec:
      replicas: 3
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
            image: nginx
            resources:
              requests:
                cpu: "110m"
                memory: "128Mi"
              limits:
                cpu: "510m"
                memory: "600Mi"
    ```

!!! warning
    For every pod in the namespace, each container must have a memory `request`, memory `limit`, CPU `request`, and CPU `limit` if there is a Resource Quota object in the namespace that puts a `limit` on the CPU and memory. This is because kubernetes needs to know whether the pod should be allowed to be scheduled based on the resources the pod requests.

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment
```


## Step 5: Verify Deployment and Pods

```
# List deployments
kubectl get deployments -n dev

# List pods
kubectl get pods -n dev
```


You'll notice that only 2 Ppds are created. This is because the resource quota in the `dev` namespace restricts the maximum number of pods that can be created to 2.

You can verify the same by checking the events in the replicaset object as follows:

```
# List replicasets in dev namespace
kubectl get rs -n dev

# Describe the replicaset to view the events
kubectl describe rs <replicaset-name> -n dev
```

You'll observe an event similar to one below:

```
Error creating: pods "my-deployment-5c74fcc755-f4hct" is forbidden: exceeded quota: dev-resource-quota, requested: pods=1, used: pods=2, limited: pods=2
```



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- dev-namespace.yml
│   |-- dev-resource-quota.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```
