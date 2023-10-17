---
description: Explore how to scale a Kubernetes deployment with our practical guide. Learn to efficiently adjust the size of your application deployments.
---

# Scale a Kubernetes Deployment

You can use `kubectl scale` command to scale a Kubernetes `Deployment` to a desired value.


## Step 1: Scale a Deployment

```
# Command template
kubectl scale deployment <deployment-name> --replicas <desired-count>
{OR}
kubectl scale deployment/<deployment-name> --replicas <desired-count>

# Actual command
kubectl scale deployment my-deployment --replicas 3
{OR}
kubectl scale deployment/my-deployment --replicas 3
```


## Step 2: Verify if the Deployment Was Scaled

Verfify the Deployment and Pods:

```
# Describe deployment
kubectl describe deploy/my-deployment

# List pods
kubectl get pods | grep my-deployment
```

You'll notice the following:

- New pods come up if the updated value of replicas is greater than the previous value (Scale up)
- Some of the old pods are terminated if the updated value of replicas is less than the previous value (Scale down)



## Use kubectl edit Command to Scale the Deployment

You can also use `kubectl edit` command to scale the Deployment as follows:

1. Set the editor you want to use (Default is `vim`):

    ```
    export KUBE_EDITOR=nano
    ```

2. Edit and update the Deployment:

    ```
    # Command template
    kubectl edit deployment <deployment-name>
    {OR}
    kubectl edit deployment/<deployment-name>

    # Actual command
    kubectl edit deployment my-deployment
    {OR}
    kubectl edit deployment/my-deployment
    ```

    Change the replicas to a desired value (For example, 2) and save the file. The Deployment will be updated.

3. Verfify the Deployment and Pods again:

    ```
    # Describe deployment
    kubectl describe deploy/my-deployment

    # List pods
    kubectl get pods | grep my-deployment
    ```