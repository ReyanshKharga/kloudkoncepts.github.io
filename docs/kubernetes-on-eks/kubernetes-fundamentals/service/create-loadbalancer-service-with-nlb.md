---
description: Create a LoadBalancer Service with NLB (Network Load Balancer) in Kubernetes. Step-by-step guidance on setting up LoadBalancer services using AWS NLB, optimizing your application's performance and ensuring high availability within your Kubernetes cluster.
---

# Create LoadBalancer Service With NLB

A Network Load Balancer (NLB) is a type of load balancer that operates at the Transport layer (Layer 4) of the OSI model, which allows it to handle a large number of requests per second and distribute traffic to targets across multiple Availability Zones.

This makes it a good choice for applications that require high throughput and low latency, such as gaming applications, large-scale database deployments, or DNS resolution.

To configure a kubernetes service to use a Network Load Balancer, add the annotation `service.beta.kubernetes.io/aws-load-balancer-type` and set its value to `nlb`.

Let's see this in action!


## Docker Image

Let's see the examples we discussed in action!

Here is the Docker Image used in this tutorial: [reyanshkharga/nodeapp]{:target="_blank"}


## Step 1. Create a Deployment

First, we need a set of pods that we want to expose using the LoadBalancer Service.

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

Let's create a LoadBalancer service that creates a network load balancer (NLB) as follows:

=== ":octicons-file-code-16: `my-nlb-service.yml`"

    ```yaml linenums="1"
    apiVersion: v1
    kind: Service
    metadata:
      name: my-nlb-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: nlb
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

Let's apply the manifest to create the service:

```
kubectl apply -f my-nlb-service.yml
```


## Step 4: Verify the Service

```
# List services
kubectl get svc
```

Notice the `PORT(S)` field in the output. You'll see the values of service port as well as `nodePort` mentioned there. (e.g. `80:32124`)

You'll also observe that the `EXTERNAL-IP` field is set to the value of the DNS name of the network load balancer that was created.


## Step 5: Verify the Network Load Balancer (NLB) in AWS Console

Visit AWS Console and verify that a network load balancer (NLB) was created.

You'll observe the following:

- The network load balancer (NLB) spans across the same set of AZs as the EKS cluster
- A target group is created and worker nodes are added to the target group
- The load balancer uses the TCP port for health check that kubernetes assigned to the service (`nodePort`)
- Listeners are added with ports as defined in service definition.
- Security group of worker nodes is updated to allow layer 4 traffic. (Notice the ICMP rule.)

!!! note
    Network Load Balancers (NLBs) in AWS don't have a security group because they operate at the network layer (Layer 4) of the OSI model.


## Step 6: Access the Service Using Network Load Balancer (NLB) DNS

Since this is a LoadBalancer service we can use the Load Balancer DNS to access the Kubernetes service.

First, we need to get the LoadBalancer DNS.

The load balancer DNS can be obtained through either of the following methods:

1. Via the AWS Console
2. Through the kubernetes service


Let's get the load balancer DNS using service:

```
kubectl get svc
```

Copy the `EXTERNAL-IP` value. This is the load balancer DNS.

Visit any browser on your local machine and hit `<EXTERNAL-IP>:80`. You'll get the response form the kubernetes service.


## Clean Up

Assuming your folder structure looks like the one below:

```
|-- manifests
│   |-- my-deployment.yml
│   |-- my-nlb-service.yml
```

Let's delete all the resources we created:

```
kubectl delete -f manifests/
```

When you delete a kubernetes service, any AWS resources that were created alongside it, such as the Load Balancer and the security group for the Load Balancer, will also be deleted.



<!-- Hyperlinks -->
[reyanshkharga/nodeapp]: https://hub.docker.com/r/reyanshkharga/nodeapp