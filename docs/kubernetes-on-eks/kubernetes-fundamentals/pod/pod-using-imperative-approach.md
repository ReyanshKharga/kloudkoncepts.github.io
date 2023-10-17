---
description: Explore the creation and management of kubernetes pods using imperative commands. Learn hands-on techniques for effective control and manipulation of your containerized applications.
---

# Create and Manage Kubernetes Pods Using Imperative Commands

Let's look at the imperative commands that you can use to create and manage Kubernetes Pods.

Here is the Docker Image used in this tutorial: [reyanshkharga/nginx]{:target="_blank"}


## Step 1: Create a Pod

```
# Command template
kubectl run <pod-name> --image=<image-name>

# Actual command
kubectl run my-pod --image=reyanshkharga/nginx:v1
```


## Step 2: List Pods

```
# List all pods
kubectl get pods

# List all pods with expanded (aka "wide") output
kubectl get pods -o wide
```

Resource types are case-insensitive and you can specify the singular, plural, or abbreviated forms.

The following commands produce the same output:

```
kubectl get po 
kubectl get pod
kubectl get pods
```

!!! note
    `pod` is abbreviated as `po`.


## Step 3: Describe a Pod

```
# Command template
kubectl describe pod <pod-name>
{OR}
kubectl describe pod/<pod-name>

# Actual command
kubectl describe pod my-pod
{OR}
kubectl describe pod/my-pod
```


## Step 4: Interact With a Pod

Let's start a shell session inside the container:

```
# Command template
kubectl exec -it <pod-name> -- bash

# Actual command
kubectl exec -it my-pod -- bash
```

Now that you are inside the container, you can run any linux commands.

Try running the below commands:

```
ls
env
curl localhost
```

!!! note
    A pod can have multiple containers. In that case, you need to provide the container name you want to connect to as follows:

    ```
    kubectl exec -it my-pod --container <container-name> -- bash
    ```

## Step 5: Interact With Pods From Outside the Container

You can also run commands inside the container without starting a seperate shell session.

```
# Interact with pod from outside the container
kubectl exec -it my-pod -- ls /
kubectl exec -it my-pod -- env
kubectl exec -it my-pod -- curl localhost

# If the pod has multiple containers
kubectl exec -it my-pod --container <container-name> -- <command>
```


## Step 6: Use Port Forwarding to Access Application

`kubectl port forward` is a command that allows you to access an application or a service running on a Kubernetes cluster from your local machine.

You can use port forwarding to investigate issues with kubernetes applications locally:

```
# Command template
kubectl port-forward <pod-name> <host-port>:<container-port>

# Actual command
kubectl port-forward my-pod 5000:80
```

Let's test it out!

```
# Hit the port 5000 on your host machine
curl localhost:5000/
```

Or you can open any browser and visit `localhost:5000`.


## Step 7: View Logs of a Pod

```
# Command template
kubectl logs <pod-name>

# Actual command
kubectl logs my-pod

# Stream the log
kubectl logs my-pod -f
```

## Step 8: Delete a Pod

```
# Command template
kubectl delete pod <pod-name>

# Actual command
kubectl delete pod my-pod
```

<!-- Hyperlinks -->
[reyanshkharga/nginx]: https://hub.docker.com/r/reyanshkharga/nginx