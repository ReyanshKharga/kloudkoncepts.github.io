---
description: Ensure the status of worker nodes in your Amazon EKS cluster. Learn how to verify and manage worker nodes effectively for a robust EKS environment.
---

# Verify Worker Nodes

Let's verify if worker nodes were created.

## Step 1: List Node Groups

List the node groups in the EKS cluster we created.

```
# Command template
eksctl get nodegroup --cluster <cluster-name>

# Actual command
eksctl get nodegroup --cluster my-cluster
```


## Step 2: List Worker Nodes

List the worker nodes in the EKS cluster we created.

```
kubectl get nodes
```


## More Useful Commands

Here are some more useful `kubectl` commands:

```
# Display the cluster info
kubectl cluster-info

# Get all worker nodes with expanded (aka "wide") output
kubectl get nodes -o wide

# Describe a node
kubectl describe node <node-name>
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get no 
kubectl get node
kubectl get nodes
```

!!! note
    `node` is abbreviated as `no`.