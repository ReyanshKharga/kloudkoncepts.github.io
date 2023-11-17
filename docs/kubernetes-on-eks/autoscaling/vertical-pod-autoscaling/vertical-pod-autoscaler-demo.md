---
description: Experience the magic of Vertical Pod Autoscaler (VPA) in action with our hands-on demo! Witness dynamic pod scaling, optimize resource utilization, and achieve seamless performance adjustments in real-time.
---

# Vertical Pod Autoscaler (VPA) Demo

Now that you have an understanding of how VPA works, let's see it in action.


## Docker Images

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1]{:target="_blank"} runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Objective

We'll follow these steps to test the Vertical Pod Autoscaler (VPA):

1. We'll create a `Deployment` and a `Service` object.
2. We'll create a `VerticalPodAutoscaler` object for the deployment.
3. We'll generate load on pods managed by the deployment.
3. We'll observe VPA taking autoscaling actions to meet the increased demand.

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
              requests:
                cpu: "10m"
                memory: "10Mi"
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

Note that each pod can requests a minimum of `10m` CPU and `10Mi` memory.


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


## Step 3: Create VPA for the Deployment

Now, let's create a VPA for the deployment as follows:

=== ":octicons-file-code-16: `my-vpa.yml`"

    ```yaml linenums="1"
    apiVersion: "autoscaling.k8s.io/v1"
    kind: VerticalPodAutoscaler
    metadata:
      name: my-vpa
    spec:
      targetRef:
        apiVersion: "apps/v1"
        kind: Deployment
        name: my-deployment
      updatePolicy:
        updateMode: "Auto"
        minReplicas: 2 # Minimal number of replicas which need to be alive for Updater to attempt Pod eviction
      resourcePolicy:
        containerPolicies:
          - containerName: '*' # The name of the container that the policy applies to.
            minAllowed:
              cpu: 10m
              memory: 10Mi
            maxAllowed:
              cpu: 500m
              memory: 500Mi
            controlledResources: ["cpu", "memory"]
    ```

The `minAllowed` field prevents the VPA from recommending or setting resource requests below the specified minimum threshold, ensuring requests won't drop below that limit.

The `maxAllowed` field prevents the VPA from recommending or setting resource requests above the specified maximum threshold, ensuring requests won't exceed that limit.

Apply the manifest to create the VPA:

```
kubectl apply -f my-vpa.yml
```

Verify VPA:

```
# List vpa
kubectl get vpa

# Describe vpa to view the events
kubectl descripe vpa my-vpa
```

!!! warning
    The `.spec.replicas` value in deployment must be greater than or equal to `.spec.updatePolicy.minReplicas` value in VPA for the VPA to manage the pods for autoscaling.


## Step 4: Generate Load

Let's generate load on the pods managed by the deployment. On your local machine run the following command to generate the load:

```
while sleep 1; do seq 1000 | xargs -P100 -I{} curl -s <load-balancer-dns> > /dev/null; done
```

The above command concurrently sends 1000 requests per second to the LoadBalancer service using 100 parallel processes.


## Step 5: Monitor Pods and VPA Events

```
# List pods in watch mode
kubectl get pods -w

# List vpa in watch mode
kubectl get vpa -w

# View vpa events
kubectl describe vpa my-vpa
```

You'll notice that vpa recommender recommends a new value for resource requests and then vpa updater updates the resource requests of pods.

!!! note
    The updater may take some time to apply the recommendation and you might have to wait before you can see the updated resource `requests` in pods. 

You can view the VPA logs as follows:

```
# view logs from admission controller
kubectl get pods -n kube-system | grep vpa-admission | kubectl logs -f `awk '{print $1}'` -n kube-system

# view logs from recommender
kubectl get pods -n kube-system | grep vpa-recommender | kubectl logs -f `awk '{print $1}'` -n kube-system

# view logs from updater
kubectl get pods -n kube-system | grep vpa-updater | kubectl logs -f `awk '{print $1}'` -n kube-system
```



## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
│   |-- my-vpa.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp