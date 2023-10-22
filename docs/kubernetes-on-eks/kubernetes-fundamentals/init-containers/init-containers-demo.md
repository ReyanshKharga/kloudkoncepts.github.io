---
description: Explore the power of Kubernetes Init Containers with our hands-on demo. Learn how to enhance container orchestration for seamless app deployment.
---

# Init Containers Demo

Now, let's see init containers in action and observe how these specialized containers, as discussed in the previous section, perform their crucial tasks within the kubernetes deployment.


## Docker Images

Here are the Docker Images used in this tutorial:

- [reyanshkharga/nodeapp]{:target="_blank"}
- [reyanshkharga/alpine]{:target="_blank"}

!!! note
    [reyanshkharga/alpine]{:target="_blank"} is the modified version of famous `alpine` Docker image which includes `curl`, `ping`, and `nslookup` linux utilities.


## Step 1: Create a Deployment

Let's use a deployment to create pods with init containers as follows: 

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
          initContainers:
          - name: init-database
            image: reyanshkharga/alpine
            command: ['sh', '-c', "until nslookup my-database-service; do echo waiting for database service to be available...; sleep 2; done"]
          - name: init-cache
            image: reyanshkharga/alpine
            command: ['sh', '-c', "until nslookup my-cache-service; do echo waiting for cache service to be available...; sleep 2; done"]
    ```

!!! note
    - `until` command in Linux is used to execute a set of commands as long as the final command in the `until` commands has an exit status which is not zero.

    - `nslookup` is a command-line tool for querying DNS to retrieve information about domain names and their associated IP addresses. It is used for network troubleshooting and DNS resolution.


Here's what each init container does in the above deployment:

1. `init-database` initializes by checking the availability of the `my-database-service` through DNS resolution, waiting until it's reachable before moving on.

2. `init-cache` performs a similar initialization, ensuring the `my-cache-service` is accessible through DNS resolution before proceeding with the main container.


## Step 2: Verify Pods

!!! tip
    List the pods in watch mode so that we can observe how the status changes once the init containers finish their tasks.

```
# List pods
kubectl get pods -w
```

You'll notice that the pods haven't finished their initialization and are currently in a non-running state. This is because the init containers are still pending completion, as the services `my-database-service` and `my-cache-service` haven't been created and are consequently unavailable.


## Step 3: Create Services

Let's start creating services that init containers are waiting for:

=== ":octicons-file-code-16: `my-database-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-database-service
    spec:
      selector:
        name: my-database
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 3306
    ```

=== ":octicons-file-code-16: `my-cache-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-cache-service
    spec:
      selector:
        name: my-cache
    spec:
      ports:
      - protocol: TCP
        port: 80
        targetPort: 6379
    ```


Remember that `kubelet` runs each init container sequentially. So, even if you create the `my-cache-service` first, the `init-cache` init container won't run until the preceding init container `init-database` has completed successfully.

1. Create `my-database-service`: 

    ```
    kubectl apply -f my-database-service.yml
    ```

    Once `my-database-service` is operational, you will observe that the `init-database` init container has successfully completed its tasks.


2. Create `my-cache-service`:

    ```
    kubectl apply -f my-cache-service.yml
    ```

    Now that `my-cache-service` is also operations, `init-cache` init container would also complete its tasks successfully.

At this stage, both init containers have successfully finished their tasks. The pods in the deployment will now be initialized and the status will change to `PodInitializing`, and then eventually to `Running`.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-database-service.yml
│   |-- my-cache-service.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```



<!-- Hyperlinks -->
[reyanshkharga/nodeapp]: https://hub.docker.com/r/reyanshkharga/nodeapp
[reyanshkharga/alpine]: https://hub.docker.com/r/reyanshkharga/alpine