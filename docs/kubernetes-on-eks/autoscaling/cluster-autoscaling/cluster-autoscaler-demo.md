---
description: Explore the power of Cluster Autoscaler with our interactive demo! Optimize resource utilization, enhance scalability, and streamline your Kubernetes cluster management effortlessly. Witness seamless auto-scaling in action. 
---

# Cluster Autoscaler Demo

Now that we have the Cluster Autoscaler deployed in our EKS cluster, let's see it in action.


## Docker Images

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1]{:target="_blank"} runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Objective

Currently, we have 2 `t3.medium` nodes in `private-nodegroup`. We also have `min(2)` and `max(4)` nodes set for the nodegroup.

```
# List nodes
kubectl get nodes
```

!!! note
    Each `t3.medium` node has 2 core CPU and 4GB memory.

We'll follow these steps to test the Cluster Autoscaler:

1. We'll create a deployment with 2 replicas, each requesting 1 core CPU.
2. We'll scale the deployment to add more replicas.
3. We'll observe Cluster Autoscaler taking autoscaling actions to accommodate pending pods that couldn't be scheduled due to insufficient resources.

Let's see this in action!


## Step 1: Create a Deployment

First, let's create a deployment as follows:

=== ":octicons-file-code-16: `my-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-deployment
    spec:
      replicas: 2
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
            resources:
              requests:
                cpu: "1"
    ```

Apply the manifest to create the deployment:

```
kubectl apply -f my-deployment.yml
```

Verify deployment and pods:

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```

Note that each pod requests 1 core CPU. Also, we already have some workload running in the `kube-system` namespace. In the best case, the two nodes can accommodate two pods created by `my-deployment`.

You will observe that both pods are scheduled and running without any issues.


## Step 2: Update the Deployment

Now, let's update the deployment by setting the replicas to 4.

```
# Apply the modified deployment
kubectl apply -f deployment.yml

# List pods
kubectl get pods
```

You might notice the new pods getting stuck in the `Pending` state due to the lack of CPU resources.

Describe the pod to determine why it is still in a `Pending` state:

```
kubectl describe pod <pod-name>
```

At this point, the Cluster Autoscaler will assess the autoscaling needs and launch additional instances in the node group.

Once the new nodes are up and running, the new pods can be scheduled.

List nodes and pods in watch mode:

```
# List nodes
kubectl get nodes -w

# List pods
kubectl get pods -w
```

You can view the autoscaling events in the `cluster-autoscaler` log:

```
# View autoscaler logs
kubectl logs -f deployment.apps/cluster-autoscaler -n kube-system
```

You can also use the following command to see kubernetes events:

```
kubectl events
```

!!! note
    When you run `kubectl events` command, it retrieves and displays a list of events from the cluster's Event API. These events can include information about pod scheduling, container lifecycle events, resource creation, deletion, errors, and much more.


## Step 3: Delete the Deployment

Let's delete the deployment:

```
kubectl delete -f deployment.yml
```

When the resources are freed you will notice that the autoscaling scales down the instances in the autoscaling group.

You can view the autoscaling events in the `cluster-autoscaler` log:

```
kubectl logs -f deployment.apps/cluster-autoscaler -n kube-system
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp