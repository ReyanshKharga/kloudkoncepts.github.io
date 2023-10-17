---
description: Explore the creation and management of deployments using imperative commands. Learn the art of hands-on control for effective application management.
---

# Create and Manage Deployment Using Imperative Commands

Let's look at the imperative commands that you can use to create and manage Kubernetes `Deployments`.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}

## Step 1: Create a Deployment

```
# Command template
kubectl create deployment <deployment-name> --image=<image-name> --replicas=<replica-count>
{OR}
kubectl create deployment/<deployment-name> --image=<image-name> --replicas=<replica-count>

# Actual command
kubectl create deployment my-deployment --image=reyanshkharga/nginx:v1 --replicas 2
{OR}
kubectl create deployment/my-deployment --image=reyanshkharga/nginx:v1 --replicas 2
```

## Step 2: List Deployments

```
# List all deployments
kubectl get deployments

# List all deployments with expanded (aka "wide") output
kubectl get deployments -o wide
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get deploy 
kubectl get deployment
kubectl get deployments
```

!!! note
    `deployment` is abbreviated as `deploy`.

The `UP-TO-DATE` field shows how many replicas have been updated to the latest version, while the `AVAILABLE` field shows how many replicas are currently available and ready to serve traffic.


## Step 3: View ReplicaSets Created by the Deployment

```
kubectl get rs | grep my-deployment
```

You'll see a `ReplicaSet` created and managed by the Deployment.

The name of the `ReplicaSets` created by `Deployments` starts with the deployment name.


## Step 4: View Pods Created by the Deployment

```
kubectl get pods
```

You'll see Pods created and managed by the `Deployment`.

The name of the pods created by a `Deployment` starts with the deployment name.


## Step 5: Describe a Deployment

```
# Command template
kubectl describe deployment <deployment-name>
{OR}
kubectl describe deployment/<deployment-name>

# Actual command
kubectl describe deployment my-deployment
{OR}
kubectl describe deployment/my-deployment
```


## Step 6: Delete a Deployment

```
# Command template
kubectl delete deployment <deployment-name>
{OR}
kubectl delete deployment/<deployment-name>

# Actual command
kubectl delete deployment my-deployment
{OR}
kubectl delete deployment/my-deployment
```

<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx