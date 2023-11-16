---
description: Experience the magic of Horizontal Pod Autoscaler (HPA) in action with our hands-on demo! Witness dynamic pod scaling, optimize resource utilization, and achieve seamless performance adjustments in real-time.
---

# Horizontal Pod Autoscaler (HPA) Demo

Now that you have an understanding of how HPA works, let's see it in action.


## Docker Images

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1]{:target="_blank"} runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Objective

We'll follow these steps to test the Horizontal Pod Autoscaler (HPA):

1. We'll create a `Deployment` and a `Service` object.
2. We'll create `HorizontalPodAutoscaler` object for the deployment.
3. We'll generate load on pods managed by the deployment.
3. We'll observe HPA taking autoscaling actions to meet the increased demand.

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
      replicas: 1
      selector:
        matchLabels:
          app: demo
      template:
        metadata:
          labels:
            app: demo
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 5000
            resources:
              limits:
                cpu: "100m"
                memory: "128Mi"
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

Note that each pod can consume a maximum of `100m` CPU and `128Mi` memory.


## Step 2: Create a Service

Next, let's create a `LoadBalancer` service as follows:

=== ":octicons-file-code-16: `my-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      type: LoadBalancer
      selector:
        app: demo
      ports:
        - port: 80
          targetPort: 5000
    ```

Apply the manifest to create the service:

```
kubectl apply -f my-service.yml
```

Verify service:

```
kubectl get svc
```


## Step 3: Create HPA for the Deployment

Now, let's create a HPA for the deployment as follows:

=== ":octicons-file-code-16: `my-hpa.yml`"

    ```yaml linenums="1"
    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: my-hpa
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: my-deployment
      minReplicas: 1
      maxReplicas: 10
      metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 50
      - type: Resource
        resource:
          name: memory
          target:
            type: Utilization
            averageUtilization: 50
    ```

Apply the manifest to create the HPA:

```
kubectl apply -f my-hpa.yml
```

Verify HPA:

```
# List hpa
kubectl get hpa

# Describe hpa to view the events
kubectl descripe hpa my-hpa
```


## Step 4: Generate Load

Let's generate load on the pods managed by the deployment. On your local machine run the following command to generate the load:

```
while sleep 1; do seq 1000 | xargs -P100 -I{} curl -s <load-balancer-dns> > /dev/null; done
```

The above command concurrently sends 1000 requests per second to the LoadBalancer service using 100 parallel processes.


## Step 5: Monitor Pods and HPA Events

```
# List pods in watch mode
kubectl get pods -w

# List hpa in watch mode
kubectl get hpa -w

# View hpa events
kubectl describe hpa my-hpa
```

You'll notice that as soon as any of the defined threshold is crossed the hpa scales the number of replicas to ensure the resource utilization is within the defined threshold.

Here's a sample event from the hpa:

```yaml
Conditions:
  Type            Status  Reason              Message
  ----            ------  ------              -------
  AbleToScale     True    SucceededRescale    the HPA controller was able to update the target scale to 2
  ScalingActive   True    ValidMetricFound    the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange  the desired count is within the acceptable range
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  ----                       -------
  Normal  SuccessfulRescale  5s    horizontal-pod-autoscaler  New size: 2; reason: cpu resource utilization (percentage of request) above target
```



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
│   |-- my-hpa.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp