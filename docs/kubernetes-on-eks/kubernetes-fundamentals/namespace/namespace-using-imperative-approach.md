---
description: Learn how to create and manage Kubernetes namespaces using imperative commands. Master the essential skills for efficient container organization.
---


# Create and Manage Namespaces Using Imperative Commands

Let's look at the imperative commands that you can use to create and manage kubernetes namespaces.


## Step 1: Create a Namespace

```
# Command template
kubectl create namespace <namespace-name>
```

Let's create a namespace named `dev` follows:

```
# Create namespace dev
kubectl create namespace dev
```


## Step 2: List Namespaces

```
kubectl get namespaces
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get ns 
kubectl get namespace
kubectl get namespaces
```

!!! note
    `namespace` is abbreviated as `ns`.


## Step 3: Describe a Namespace

```
# Command template
kubectl describe namespace <namespace-name>

# Actual command
kubectl describe namespace dev
```

## Step 4: Create Kubernetes Resources in a Namespace

Let's create a pod in the `dev` namespace that we created:

```
kubectl run nginx --image=nginx -n dev
```

You can either use `-n` or `--namespace` to specify the namespace where the resource should be created.


## Step 5: List Resources in a Namespace

Let's list all the pods in the `dev` namespace:

```
kubectl get pods -n dev
{OR}
kubectl get pods --namespace dev
```

Let's start a shell session inside the nginx container to verify if the application in the pod is running as expected:

```
kubectl exec -it nginx -n dev -- bash
```

Get the default nginx page:

```
curl localhost
```


## Step 6: Delete a Resource in a Namespace

Let's delete the `nginx` pod in the `dev` namespace:

```
kubectl delete pod nginx -n dev
```


## Step 7: Delete a Namespace

```
# Command template
kubectl delete namespace <namespace-name>

# Actual command
kubectl delete namespace dev
```