---
description: Optimize your pod configurations with Requests and Limits in Kubernetes. Learn how to allocate and control computing resources effectively with our comprehensive guide. Start configuring pods for peak performance today!
---

# Configure Pods With Request and Limit

Let's see how we can configure pods with `requests` and `limits`.


## Step 1: Create Pods With Request and Limit

Let's create pods with `request` and `limt`. We'll use a deployment to create pods:

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
          app: stress
      template:
        metadata:
          labels:
            app: stress
        spec:
          containers:
          - name: my-container
            image: progrium/stress
            command: ['sh', '-c', "sleep 3600"]
            resources:
              requests:
                cpu: "250m"
                memory: "64Mi"
              limits:
                cpu: "500m"
                memory: "128Mi"
    ```

Observe the following:

- We have requested `250m` CPU and `64Mi` Memory
- We have set a limit of `500m` on CPU and `128Mi` on memory

!!! note
    The `progrium/stress` is a Docker image for `stress`, a tool for generating workload. It can produce CPU, memory, I/O, and disk stress.

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

# Describe pod
kubectl describe pod <pod-name>
```


## Step 3: View CPU and Memory Usage of Pod

```
# View resource utilization
kubectl top pod <pod-name>

# View resource utilization of the pod periodically (defaults to every 2 seconds)
watch kubectl top pod <pod-name>

# View resource utilization of the pod every second
watch -n 1 kubectl top pod <pod-name>
```

Due to the absence of any load and the process being idle, you will observe that the pod consumes minimal CPU and memory resources.

Note that `kubectl top` command doesn't have `-w` option. We are using the `watch` command in linux as a workaround.



## Step 4: CPU Stress Test

Let's induce CPU stress using the `stress` command-line utility in Linux and analyze the CPU utilization patterns of the pod.

First, let's open two terminals. One to watch pods and another to monitor the CPU utilization of the pod:

```
# Watch the pod every second
kubectl get pod <pod-name> -w

# View the resource utilization of the pod
watch kubectl top pod <pod-name>
```

Now, induce the CPU stress in the pod:

```
# Start a shell session inside the pod
kubectl exec -it <pod-name> -- bash

# Stress the cpu
stress --cpu 2 --timeout 60s
```

Here's an explanation of the `stress` command:

- `stress`: This is the name of the command-line tool used for generating system stress.
- `--cpu 2`: This option specifies the number of CPU workers to create for inducing stress.
- `--timeout 60s`: This option sets the duration for which the stress will be applied.

You will notice that the CPU utilization begins to spike but does not exceed the cpu value specified in the `limits` field of the deployment manifest.

Please note that kubernetes allows for short bursts of CPU usage that can exceed the specified limit. These bursts allow pods to utilize additional CPU resources temporarily. However, if the sustained usage consistently exceeds the `limit`, kubernetes will throttle the CPU allocation and enforce the `limit`.


## Step 5: Memory Stress Test

Let's induce Memory stress using the `stress` command-line utility in Linux and analyze the Memory utilization patterns of the pod.

First, let's open two terminals. One to watch pods and another to monitor the Memory utilization of the pod:

```
# Watch the pod every second
kubectl get pod <pod-name> -w

# View the resource utilization of the pod
watch kubectl top pod <pod-name>
```


Now, induce the Memory stress in the pod:

```
# Start a shell session inside the pod
kubectl exec -it <pod-name> -- bash

# Stress the memory
stress --vm 1000 --vm-bytes 1048576 --timeout 60s
```


Here's an explanation of the `stress` command:

- `stress`: This is the name of the command-line tool used for generating system stress.
- `--vm 1000`: This option specifies the number of virtual memory workers to create gradually.
- `--vm-bytes 1048576`: This option sets the amount of memory allocated to each virtual memory worker. In this case, it is set to 1048576 bytes, which is equivalent to 1 mebibyte (Mi).
- `--timeout 60s`: This option sets the duration for which the stress test will be active.

In essence, the command attempts to utilize `1000Mi` of memory.

You will notice that the memory utilization begins to spike but does not exceed the memory value specified in the `limits` field of the deployment manifest.

In some cases, you may observe that the container is terminated with the status `OOMKilled`. This occurs when the main process within the container is unable to continue execution due to insufficient available resources, particularly memory.


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