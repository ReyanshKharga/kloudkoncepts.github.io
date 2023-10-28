---
description: Efficiently create and manage LimitRange settings in Kubernetes with our step-by-step guide. Learn how to enforce resource constraints and maintain a well-organized cluster environment.
---

# Create and Manage LimitRange

Let's see how we can create and manage Limit Ranges.


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

Apply the manifest to create the namespace:

```
kubectl apply -f dev-namespace.yml
```

Verify the namespace:

```
kubectl get ns
```


## Step 2: Create a LimitRange Object in the Namespace

Create a `LimitRange` object in the `dev` namespace:

=== ":octicons-file-code-16: `dev-limitrange.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: dev-limitrange
      namespace: dev
    spec:
      limits:
      - default: # this section defines default limits
          cpu: 500m
          memory: 512Mi
        defaultRequest: # this section defines default requests
          cpu: 100m
          memory: 64Mi
        max: # maximum resource limits
          cpu: 1000m
          memory: 1000Mi
        min: # minumum resource limits
          cpu: 100m
          memory: 64Mi
        type: Container
    ```

Here's a breakdown of the purpose of each field defined in `.spec.limits`:

- `default`: This field sets the default resource `limits` for containers. If a container does not specify its own resource `limits`, it will inherit these default values.

- `defaultRequest`: This field sets the default resource `requests` for containers. Similar to defaults, if a container does not specify its own resource `requests`, it will use these default values.

- `min`: The `min` field specifies the minimum resource `request` that a container must set. It enforces a lower boundary, ensuring that containers are allocated at least the specified amount of resources. This prevents resource starvation and ensures containers have the necessary resources to function properly.

- `max`: The `max` field sets the maximum resource `limits` that a container can specify. It imposes an upper boundary, preventing containers from consuming excessive resources. This helps in maintaining fairness, preventing resource overutilization, and mitigating the impact of misbehaving or compromised containers.


The `type` field in the `LimitRange` object specifies the scope to which the resource `limits` apply. It can have two possible values:

1. `Container`: This type indicates that the resource `limits` defined in the `LimitRange` object apply to individual containers within pods. It means that the specified limits are enforced for each container running in the namespace.

2. `Pod`: If the type is set to `Pod`, the resource limits defined in the `LimitRange` object apply at the pod level. This means that the limits specified will be shared by all containers within the same pod.


Apply the manifest to create the `LimitRange` object:

```
kubectl apply -f dev-limitrange.yml
```


## Step 3: Verify the LimitRange

```
# List limitranges
kubectl get limitrage -n dev

# Describe the limitrange
kubectl describe limitrange dev-limitrange -n dev
```

Note that `LimitRange` can also be used to enforce a ratio between `request` and `limit` for a resource in a namespace. And, that's why you would notice a field `Max Limit/Request Ratio` as well when you describe the limitrange.



## Step 4: Create Pod in the Dev Namespace

Let's create pods in the `dev` namespace and observe how the default `request` and `limit` values are applied to the pods. We'll use deployment to create pods:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
      namespace: dev
    spec:
      replicas: 1
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
    ```

You can observe that we have not specified any `request` or `limit` for cpu or memory in the pod template.

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 5: Verify Deployment and Pods

```
# List deployments
kubectl get deployments -n dev

# List pods
kubectl get pods -n dev
```


## Step 6: Describe Pod

Let's describe one of the pods in the deployment and observe the `request` and `limit` values:

```
kubectl describe pod <pod-name> -n dev
```

You'll observe that the pod has inherited the default `request` and `limit` values defined in the `LimitRange` object in the `dev` namespace.

Note that the default `request` and `limit` values for resources can be overriden if we explicitly specify it in the pod definition. Let's see that in the next step.


## Step 7: Override the Default Values Explicitly

Let's override the default `request` and `limit` values by explicitly providing the desired `request` and `limit` values in the pod template:

Update the deployment as follows:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
      namespace: dev
    spec:
      replicas: 1
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
            # Override the default requests and limits inherited from limitrange
            resources:
              requests:
                cpu: "110m"
                memory: "128Mi"
              limits:
                cpu: "510m"
                memory: "600Mi"
    ```

Apply the manifest to update the deployment:

```
kubectl apply -f my-deployment-with-override.yml
```

Verify pods:

```
kubectl get pods -n dev
```

Describe one of the pods:

```
kubectl describe pod <pod-name> -n dev
```

You'll observe that the pod has the resource `request` and `limit` values set as specified in the pod template.

Pleast note that when you override the `request` and `limit` values of a resource the following conditions must be satisfied:

1. The `request` value of the resource can't be lesser than the `min` value specified in the limitrange.
2. The `limit` value of the resource can't be greater than the `max` value specified in the limitrange.

In our example, the `requests` value for the `cpu` resource can't be lesser than `100m` (the `min` value in limitrange definition) and the `limits` value of the `cpu` resource can't be greater than `1000m` (the `max` value in the limitrange definition).

Similarly, the `requests` value for the `memory` resource can't be lesser than `64Mi` (the `min` value in limitrange definition) and the `limits` value of the `memory` resource can't be greater than `1000Mi` (the `max` value in the limitrange definition).

Let's test this out in the next step.



## Step 8: Override the Default Values With Forbidden Values

Let's see what happens when we try to override the default values but do not adhere to the `min` and `max` limit values specified in the `LimitRange` object.

Update the deployment as follows:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
      namespace: dev
    spec:
      replicas: 1
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
            # Override the default requests and limits inherited from limitrange
            resources:
              requests:
                cpu: "110m"
                memory: "128Mi"
              limits:
                cpu: "510m"
                memory: "1200Mi"
    ```

Notice that the cpu `request` and `limit` values are well within the specified `min` and `max` values defined in the `LimitRange` object (`100m <= 110m & 510m <= 1000m`) but the memory `limit` value exceeds the ma`x value defined in the `LimitRange` object (`1024Mi > 1000Mi`).

Let's update the deployment and observe the output:

```
kubectl apply -f my-deployment.yml
```

Describe the deployment:

```
kubectl get deploy/my-deployment -n dev
```

You'll notice that no new pods come up because the min/max constraint is not satisfied.

You can verify the same by checking the events in the replicaset object as follows:

```
# List replicasets in dev namespace
kubectl get rs -n dev

# Describe the replicaset to view the events
kubectl describe rs <replicaset-name> -n dev
```

You'll observe an event similar to one below:

```
Error creating: pods "my-deployment-5ddffd4d5d-5drmh" is forbidden: maximum memory usage per Container is 1000Mi, but limit is 1200Mi
```


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- dev-limitrange.yml
│   |-- dev-namespace.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```