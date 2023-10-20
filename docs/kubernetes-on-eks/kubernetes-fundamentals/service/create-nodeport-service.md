---
description: Discover how to Create a NodePort Service in Kubernetes. Step-by-step guidance and insights on setting up NodePort services, ensuring accessibility to your applications. Harness the power of NodePort services for external communication in your Kubernetes cluster.
---

# Create NodePort Service

A `NodePort` service is a type of kubernetes service that exposes a set of pods to the outside network.

When you create a `NodePort` service, kubernetes allocates a `static port` on each node in the cluster, and then forwards traffic sent to that port to the corresponding pods.

This allows external clients to access the service by connecting to any node's IP address and the allocated static port.

`NodePort` services are often used to expose a service to the outside world for testing or development purposes, or for services that need to be accessible from outside the cluster.


## Docker Image

Let's see the examples we discussed in action!

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp]{:target="_blank"}



## Step 1. Create a Deployment

First, we need a set of pods that we want to expose using the `NodePort` service.

So, let's create a deployment as follows:

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
          app: nodeapp
      template:
        metadata:
          labels:
            app: nodeapp
        spec:
          containers:
          - name: nodeapp
            image: reyanshkharga/nodeapp:v1
            imagePullPolicy: Always
            ports:
              - containerPort: 5000
    ```

```
# Create deployment
kubectl apply -f my-deployment.yml
```


## Step 2: Verify Deployment and Pods

```
# List deployments
kubectl get deployments

# List pods
kubectl get pods
```


## Step 3: Create NodePort Service

Let's create a `NodePort` service as follows:

=== ":octicons-file-code-16: `my-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nodeport-service
    spec:
      type: NodePort
      selector:
        app: nodeapp
      ports:
        - port: 80
          targetPort: 5000
          nodePort: 30000
    ```

Note that we have also specified a `nodePort` of `30000`, which means that traffic sent to this port on any node in the cluster will be forwarded to the service.

The `nodePort` value can be any valid TCP or UDP port number between `30000` and `32767`.

!!! note
    If you don't specify `nodePort` field, kubernetes will automatically allocate a port within the valid range (`30000-32767`) for you.

Let's apply the manifest to create the service:

```
kubectl apply -f my-nodeport-service.yml
```


## Step 4: Verify the Service

```
# List services
kubectl get svc
```

Notice the `PORT(S)` field. You'll see the values of service `port` as well as `nodePort` mentioned there. (e.g. `80:30000`).


## Step 5: Access the Service Using NodePort

Since this is a `NodePort` service we can use any woker node to access the service using the `nodePort` that we specified.

First, we need to get IP address of the worker nodes:

```
kubectl get nodes -o wide
```

Copy the `EXTERNAL-IP` value (public IP) of any node you want to connect the service through.

Visit any browser on your local machine and hit `<EXTERNAL-IP>:30000`. You'll get the response form the kubernetes service.

!!! note
    You must whitelist port `30000` in the security group of the worker nodes or else you won't be able to connect to it.

You can also use the `INTERNAL-IP` value of any node to connect through the service. But internal IP is accessible only from within the VPC where this node is.

Let's test this out!

1. Create a simple pod:

    ```
    kubectl run nginx --image=nginx
    ```

2. Start a shell session inside the nginx container:

    ```
    kubectl exec -it nginx -- bash
    ```

3. Access the service through node port:

    ```
    curl <INTERNAL-IP>:30000
    ```

    You'll get the response from the kubernetes service.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-service.yml
```

Let's delete all the resources we created:

```
# Delete deployment and services
kubectl delete -f manifests/

# Delete nginx pod
kubectl delete pod nginx
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp]: https://hub.docker.com/r/reyanshkharga/nodeapp