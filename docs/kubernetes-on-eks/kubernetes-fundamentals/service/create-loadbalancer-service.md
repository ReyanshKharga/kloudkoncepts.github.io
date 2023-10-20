---
description: Learn How to Create a LoadBalancer Service. Step-by-step instructions and insights on setting up LoadBalancer services in your Kubernetes environment, ensuring high availability and efficient traffic distribution for your applications.
---

# Create LoadBalancer Service

A `LoadBalancer` service in kubernetes is a way to distribute incoming network traffic across multiple instances of your application running in a kubernetes cluster.

It acts as a middleman between the client and the servers and helps distribute requests evenly across multiple pods or nodes.

Think of it like a traffic cop at a busy intersection. The traffic cop helps distribute the cars evenly across all lanes so that no lane becomes overwhelmed with traffic and causes a traffic jam.

Similarly, the load balancer service helps distribute incoming network traffic across all the pods or nodes in the cluster, ensuring that no one pod or node becomes overwhelmed with traffic and causes a bottleneck.


## What Happens When You Create a LoadBalancer Service?

When you create a `LoadBalancer` service in kubernetes, the following happens:

1. The cloud provider (such as AWS or GCP) provisions a new load balancer resource in the cloud environment.
2. The load balancer is configured to forward traffic to the service's endpoints.

Let's see this in action!


## Docker Image

Let's see the examples we discussed in action!

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp]{:target="_blank"}


## Step 1. Create a Deployment

First, we need a set of pods that we want to expose using the `LoadBalancer` Service.

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


## Step 3: Create LoadBalancer Service

Let's create a LoadBalancer service as follows:

=== ":octicons-file-code-16: `my-loadbalancer-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-loadbalancer-service
    spec:
      type: LoadBalancer
      selector:
        app: nodeapp
      ports:
        - port: 80
          targetPort: 5000
    ```

Note that you can also specify a `nodePort` field with desired value.

If you don't specify `nodePort` field, kubernetes will automatically allocate a port within the valid range (`30000-32767`) for you.

Apply the manifest to create the service:

```
kubectl apply -f my-loadbalancer-service.yml
```


## Step 4: Verify the Service

```
# List services
kubectl get svc
```

Notice the `PORT(S)` field in the output. You'll see the values of service port as well as `nodePort` mentioned there. (e.g. `80:32124`)

You'll also observe that the `EXTERNAL-IP` field is set to the value of the DNS name of the classic load balancer that was created.

!!! note
    By default Kubernetes creates a classic load balancer (CLB) but we can configure it to create a network load balancer or an application load balancer.
    
    We'll see that later in this course.


## Step 5: Verify the Classic Load Balancer in AWS Console

Visit AWS Console and verify that a classic load balancer was created.

You'll observe the following:

- The classic load balancer spans across the same set of AZs as the EKS cluster
- The worker nodes (instances) are added to the load balancer
- The load balancer uses the TCP port for health check that kubernetes assigned to the service (`nodePort`)
- Listeners are added with ports as defined in the service definition.
- Security group of worker nodes is updated to allow traffic from the load balancer


## Step 6: Access the Service Using Load Balancer DNS

Since this is a `LoadBalancer` service we can use the Load Balancer DNS to access the kubernetes service.

First, we need to get the LoadBalancer DNS.

The load balancer DNS can be obtained using either of the following methods:

1. Via the AWS Console
2. Through the kubernetes service

Let's get the load balancer DNS using kubernetes service:

```
kubectl get svc
```

Copy the `EXTERNAL-IP` value. This is the load balancer DNS.

Visit any browser on your local machine and hit `<EXTERNAL-IP>:80`. You'll get the response form the Kubernetes Service.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-loadbalancer-service.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

When you delete a kubernetes service, any AWS resources that were created alongside it, such as the Load Balancer and the security group for the Load Balancer, will also be deleted.



<!-- Hyperlinks -->
[reyanshkharga/nodeapp]: https://hub.docker.com/r/reyanshkharga/nodeapp