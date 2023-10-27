---
description: Leverage the power of ConfigMap to efficiently supply environment variables to your Kubernetes containers. Discover how to simplify and manage environment variables with our practical guide.
---

# Use ConfigMap to Supply Environment Variables

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

Let's create pods that uses `ConfigMap` to set environment variables for the container. We'll use a deployment to create pods:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
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
            envFrom:
            - configMapRef:
                name: my-configmap
    ```

Observe that we are using the keyword `envFrom` to supply a list of environment variables from the ConfigMap `my-configmap`.

Apply the manifest to create deployment:

```
kubectl apply -f my-deployment.yml
```

## Step 4: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 5: Verify Environment Variables

Start a shell session inside the container:

```
kubectl exec -it <pod-name> -- bash
```

List environment variables available to the container:

```
env
```

You'll see a list of environment variables available to the container. This includes both `system-provided` as well as `user-provided` environment variables.

Print values of the environment variables we set:

```
# Print value of the environment variable foo1
echo $foo1

# Print value of the environment variable foo2
echo $foo2
```

You'll notice the environment variables `foo1` and `foo2` are set to `bar1` and `bar2` respectively.

Now, what if you want to use `ConfigMap` for the environemnt variables but also want to supply additional environment variables directly in the pod definition?

In that case, you can use both the `env` and `envFrom` field. We'll explore this in the next section.

## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-configmap.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```