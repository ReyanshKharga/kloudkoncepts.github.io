---
description: Safely supply environment variables using Kubernetes Secrets. Learn how to securely manage sensitive data for your application configurations with our practical guide.
---

# Use Secret to Supply Environment Variables

Let's see how we can use `Secret` to supply environment variables to containers in a pod:


## Step 1: Create a Secret

Let's create a `Secret` with data that contains the required environment variables:

=== ":octicons-file-code-16: `my-secret.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Secret
    metadata:  
      name: my-secret
    data:
      username: cmV5YW5zaA==
      password: bXlkYnBhc3N3b3Jk
    ```

Apply the manifest to create the Secret:

```
kubectl apply -f my-secret.yml
```


## Step 2: Verify Secret

```
# List secrets
kubectl get secrets

# Describe the secret
kubectl describe secret my-secret
```


## Step 3: Create Pods That Uses Environment Variables

Let's create pods that uses `Secret` to set environment variables for the container. We'll use a deployment to create pods:

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
            - secretRef:
                name: my-secret
    ```

Observe that we are using the keyword `envFrom` to supply a list of environment variables from the Secret `my-secret`.

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
# Print value of the environment variable username
echo $username

# Print value of the environment variable password
echo $password
```

!!! note
    Kubernetes automatically does `base64` decoding for secrets used in the pod.



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-secret.yml
│   |-- my-deployment.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```


!!! tip
    Since `Secret` is similar to `ConfigMap` you can repeat all the examples that we discussed in `ConfigMap` section for `Secret` as well. Just replace `configMapRef` by `secretRef` and `configMapKeyRef` by `secretKeyRef`.