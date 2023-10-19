---
description: Learn how to create and manage kubernetes ReplicaSets with our comprehensive guide. Master the art of efficient replication in containerized applications.
---

# Create and Manage Kubernetes ReplicaSets

Let's see how you can create and manage `ReplicaSets`.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}

!!! note
    You cannot create a `ReplicaSet` using imperative command. The only way to create ReplicaSets is by using the declarative approach.

## Step 1. Create ReplicaSet Manifest

First, we need to write the `ReplicaSet` manifest as follows:

=== ":octicons-file-code-16: `my-replicaset.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: my-replicaset
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
            tier: backend
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 80
    ```

Required fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object.
- `kind` - What kind of object you want to create.
- `metadata` - Data that helps uniquely identify the object, including a name string, UID , and - optional namespace.
- `spec` - What state you desire for the object.


## Step 2: Create ReplicaSet

Let's use `kubectl apply` to apply the manifest and create the ReplicaSet.

```
# Create replicaset
kubectl apply -f my-replicaset.yml
```


## Step 3: List ReplicaSets

```
# List all replicasets
kubectl get replicasets

# List all replicasets with expanded (aka "wide") output
kubectl get replicasets -o wide
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get rs 
kubectl get replicaset
kubectl get replicasets
```

!!! note
    `replicaset` is abbreviated as `rs`.


## Step 4: Describe a ReplicaSet

```
# Command template
kubectl describe rs <replicaset-name>
{OR}
kubectl describe rs/<replicaset-name>

# Actual command
kubectl describe rs my-replicaset
{OR}
kubectl describe rs/my-replicaset
```


## Step 5: Scale a ReplicaSet

You can use `kubectl scale` command to scale a ReplicaSet up or down by changing the desired number of replicas.

```
# Command template
kubectl scale rs <replicaset-name> --replicas <desired-count>
{OR}
kubectl scale rs/<replicaset-name> --replicas <desired-count>

# Actual command - scale up
kubectl scale rs my-replicaset --replicas 3
{OR}
kubectl scale rs/my-replicaset --replicas 3

# Actual command - scale down
kubectl scale rs my-replicaset --replicas 1
{OR}
kubectl scale rs my-replicaset --replicas 1
```

Or, you can change the value of `replicas` field to a desired value in the manifest and then apply the manifest again. This method is desirable and recommended.

```
# Update the replicaset
kubectl apply -f my-replicaset.yml
```


## Step 6: ReplicaSet in Action

Let's delete a pod in the `ReplicaSet` and notice what happens.

```
kubectl delete pod <pod-name>
```

You will notice that as soon as the pod terminates, the `ReplicaSet` will spawn a new pod using the template to ensure that the desired number of pods is always running.

```
# Watch and continuously update the status of pods in real-time
kubectl get pods -w
```

## Step 7: Delete a ReplicaSet

```
# Command template
kubectl delete rs <replicaset-name>
{OR}
kubectl delete rs/<replicaset-name>

# Actual command
kubectl delete rs my-replicaset
{OR}
kubectl delete rs/my-replicaset
```



!!! quote "References:"
    !!! quote ""
        * [ReplicaSet Concept]{:target="_blank"}
        * [ReplicaSet v1 apps]{:target="_blank"}
        * [Workload Resources - ReplicaSet]{:target="_blank"}
        * [ObjectMeta]{:target="_blank"}
        * [ReplicaSetSpec]{:target="_blank"}



<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx
[ReplicaSet v1 apps]: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#replicaset-v1-apps
[Workload Resources - ReplicaSet]: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replica-set-v1/
[ObjectMeta]: https://kubernetes.io/docs/reference/kubernetes-api/common-definitions/object-meta/#ObjectMeta
[ReplicaSetSpec]: https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/replica-set-v1/#ReplicaSetSpec
[ReplicaSet Concept]: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/