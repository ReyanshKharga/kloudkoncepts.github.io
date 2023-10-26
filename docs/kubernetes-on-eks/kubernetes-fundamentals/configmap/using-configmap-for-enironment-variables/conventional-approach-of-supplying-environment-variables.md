---
description: Explore the conventional approach for supplying environment variables to containers within a pod. Learn how to configure and manage environment variables effectively for your Kubernetes workloads. Master the art of environment variable handling now!
---

# Conventional Approach of Supplying Environment Variables

Let's look at the conventional approach of supplying environment variables to containers running inside a pod.


## Step 1: Create Pods That Uses Environment Variables

Let's create pods that uses environment variables. We'll use a deployment to create pods:

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
            env:
            - name: key1
              value: value1
            - name: key2
              value: value2
    ```

Observe the following:

- We are using the keyword `env` to supply a list of environment variables to the `nginx` container
- In this example, we have supplied two environment variables; `key1`, and `key2`, with values `value1` and `value2`, respectively.

Apply the manifest to create deployment:

```
kubectl apply -f my-deployment.yml
```


## Step 2: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: Verify Environment Variables

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
# Print value of the environment variable key1
echo $key1

# Print value of the environment variable key2
echo $key2
```

You'll notice the environment variables `key1` and `key2` are set to `value1` and `value2` respectively.

This method of supplying environment variables is acceptable if you have only a few of them. But, what if you have a significant number of environment variables? Adding all of them in the manifest file will needlessly bloat the manifest file.

In that case, you can use `ConfigMap` to supply a list of environment variables to containers in a pod. We'll explore this in the next section.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```