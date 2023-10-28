---
description: Install Metrics Server on your EKS cluster seamlessly with our step-by-step guide. Monitor and manage your Kubernetes environment more effectively. Get started with Metrics Server on EKS now!
---

# Install Metrics Server on EKS Cluster

Metrics Server is a tool in kubernetes that tracks and provides information about how much `CPU` and `memory` resources are being used by nodes and pods in the cluster.


## Step 1: Deploy the Metrics Server

Deploy the latest or specific release of metrics server as follows:

```
# Deploy the latest metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

{OR}

# Deploy a specific version of the metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.3/components.yaml
```

You can check all the releases here: [metrics server releases]{:target="_blank"}



## Step 2: Verify the Metrics Server

Lets' verify the status of the `metrics-server` APIService. It could take a few minutes to be available.

```
# Verify the status of metrics server
kubectl get apiservice v1beta1.metrics.k8s.io -o json | jq '.status'
```

Once the `metrics server` is available, you should see an output similar to the one below:

```json
{
  "conditions": [
    {
      "lastTransitionTime": "2023-06-04T03:44:08Z",
      "message": "all checks passed",
      "reason": "Passed",
      "status": "True",
      "type": "Available"
    }
  ]
}
```

## Step 3: View CPU and Memory Usage of Nodes

```
# View CPU and Memory usage of all the nodes
kubectl top nodes

# View CPU and Memory usage of a particular node
kubectl top node <node-name>
```

## Step 4: View CPU and Memory Usage of Pods

```
# View CPU and Memory usage of all the pods
kubectl top pods

# View CPU and Memory usage of a particular pod
kubectl top pod <pod-name>
```




<!-- Hyperlinks -->
[metrics server releases]: https://github.com/kubernetes-sigs/metrics-server/releases