---
description: Learn how you can access pods without kubernetes service. simplify container management for your apps.
---

# Access Pods Without Kubernetes Service

Consider a deployment with 2 replicas of a pod. Now, let's assume there is no Kubernetes Service. How does other pods in the cluster access these pods?

They do so through IP addresses of these pods. Let's see this in action!

## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/nginx:v1]{:target="_blank"}
- [reyanshkharga/nodeapp:v1]{:target="_blank"}

!!! note
    [reyanshkharga/nodeapp:v1] runs on port `5000` and has the following routes:

    - `GET /` Returns host info and app version
    - `GET /health` Returns health status of the app
    - `GET /random` Returns a randomly generated number between 1 and 10


## Step 1: Create Pods Using Deployments

Let's create two deployments as follows:

=== ":octicons-file-code-16: `frontend-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: frontend-deployment
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: nginx
          tier: frontend
      template:
        metadata:
          labels:
            app: nginx
            tier: frontend
        spec:
          containers:
          - name: nginx
            image: reyanshkharga/nginx:v1
            ports:
              - containerPort: 80
    ```

=== ":octicons-file-code-16: `backend-deployment.yml`"

    ```yaml linenums="1"
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: backend-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nodeapp
          tier: backend
      template:
        metadata:
          labels:
            app: nodeapp
            tier: backend
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            ports:
              - containerPort: 5000
    ```

Here's what your folder structure should look like:

```
|-- manifests
│   |-- frontend-deployment.yml
│   |-- backend-deployment.yml
```

Apply the manifest to create the deployments:

```
# Create frontend deployment
kubectl apply -f frontend-deployment.yml

# Create backend deployment
kubectl apply -f backend-deployment.yml
```

Or, you can also apply them together as follows:

```
kubectl apply -f manifests/
```


## Step 2: Verify Deployments and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```

Also, let's list out the pods with wide option to view the IP addresses of the pods.

```
kubectl get pods -o wide
```


## Step 3: Access Backend Pod From a Frontend Pod

Let's try to access one of the backend pods from a frontend pod.

1. Start a shell session inside the conainer of one of the frontend pods:

    ```
    kubectl exec -it <frontend-pod-name> -- bash
    ```

2. Access backend pods:

    ```
    # Access root endpoint
    curl <backend-pod-IP>:5000

    # Access /health route
    curl <backend-pod-IP>:5000/health

    # Access /random route
    curl <backend-pod-IP>:5000/random
    ```

You can verify the same with other backend pod.


## Problems With This Approach of Accessing Pods

1. What happens if let's say one of the pods go down? The Kubernetes deployment controller brings up another pod. Now the IP address list of these pods change and all the other pods need to keep track of the same.

2. The same is the case when there is auto scaling enabled. The number of the pods increases or decreases based on demand.

3. What if you want to enable load balancing for these pods? How do you do that? Implementing load balancing could be a tedious job.

And that's where kubernetes `Service` comes into the picture. It solves all of the above problems.


## Clean Up

Delete the deployments:

```
# Delete frontend deployment
kubectl delete -f frontend-deployment.yml

# Delete backend deployment
kubectl delete -f backend-deployment.yml
```

Or, you can also delete them together as follows:

```
kubectl delete -f manifests/
```




<!-- Hyperlinks -->
[reyanshkharga/nginx:v1]: https://hub.docker.com/r/reyanshkharga/nginx
[reyanshkharga/nodeapp:v1]: https://hub.docker.com/r/reyanshkharga/nodeapp