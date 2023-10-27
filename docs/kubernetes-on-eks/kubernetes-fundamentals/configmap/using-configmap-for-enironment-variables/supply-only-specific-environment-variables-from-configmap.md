---
description: Efficiently select and supply specific environment variables from a ConfigMap in your Kubernetes environment. Learn how to fine-tune your configuration data for precise variable provisioning with our step-by-step guide. Enhance your environment variable management now!
---

# Supply Only Specific Environment Variables From ConfigMap

Consider a case where you want to use only a specific items from `ConfigMap` as environment variables instead of supplying entire `ConfigMap` as environment variables.

Let's see how we can do that.


## Step 1: Create a ConfigMap

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

Let's create pods that uses specific items from the `ConfigMap` as environment variables for the container. We'll use a deployment to create pods:

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
            - name: foo1
              valueFrom:
                configMapKeyRef:
                  name: my-configmap
                  key: foo1
    ```

Observe the following:

- We are using `env` keyword to supply a list of environment variables.
- We are also using `valueFrom` keyword to get the value of the key `foo1` in the ConfigMap `my-configmap` and then setting it as the value of an environment variable named `foo1`.

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

You'll see a list of environment variables available to the container. This includes both `system-provided` as well as `user-provided` (using env and `valueFrom` keyword) environment variables.


Print values of the environment variables we set:

```
# Print value of the environment variable key1
echo $key1

# Print value of the environment variable key2
echo $key2

# Print value of the environment variable foo1
echo $foo1
```

You'll notice the following:

- Environment variables `key1` and `key2` are set to `value1` and `value2` respectively.
- Environment variables `foo1` is set to `bar1`.


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